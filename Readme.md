# Как починить сервера Google чтобы YouTube снова заработал

В данном гайде я расскажу как исправить проблему зависающего YouTube с помощью различных программ обхода блокировок DPI (Deep Packet Inspection / Глубокое исследование пакетов). В работу DPI погружать не буду, а сразу перейду к делу.

## Zapret для YouTube (Подходит для Linux и Роутеров)

1. Необходимо скачать программу и необходимые компоненты. Сделаем мы это с помощью Git. 
Установка на [Linux](https://git-scm.com/downloads/linux):
- Debian / Ubuntu / Debian подобные
  ```sudo apt-get install git curl ip6tables ipset iptables```

  Либо для Ubuntu
  ```add-apt-repository ppa:git-core/ppa```
  ```sudo apt update && apt install git curl ip6tables ipset iptables```

- Arch / Arch подобные
  ```sudo pacman -S git dnsutils curl ip6tables ipset iptables```

-  Red Hat / Fedora / Red Hat подобные
  ```sudo dnf install git curl iptables ipset```

- NixOs
  ```sudo nix-env -i git curl ipset iptables```

- Gentoo
  ```
  emerge --ask --verbose dev-vcs/git
  emerge --ask --verbose net-misc/curl
  emerge --ask --verbose net-firewall/ipset
  emerge --ask --verbose net-firewall/iptables
  ```

- [OpenWrt](https://gist.github.com/kolyanok/6e93f3ed5f3aefb4d482df8c4463f196) / [Keenetic](https://telegra.ph/Nastrojka-zapret-ot-bol-van-na-Keenetic-08-18)
  ```
  opkg update
  opkg install iptables-mod-extra iptables-mod-nfqueue iptables-mod-filter iptables-mod-ipopt iptables-mod-conntrack-extra ipset curl ip6tables-mod-nat grep git-http curl gzip ipset iptables nano ca-certificates
  mkdir /opt # Если директории нет
  cd /opt
  git clone --depth 1 https://github.com/bol-van/zapret
  cd zapret/
  ```

После скачивания всего на OpenWrt и Keenetic переходим на пункт 4.

1. Клонируем архив с программой
   ```git clone https://github.com/bol-van/zapret```

2. Клонируем папку с программой в /opt
   ```sudo cp -r ~/"Директория с программой" /opt```

3.  Переходим в папку с Zapret из под рута
    ```su -```
    Вводим пароль root`а
   ```cd /opt/zapret```

4. Далее запускаем скрипты установки всех нужных компонентов
   ```./install_bin.sh```
   ```./install_prereq.sh```
   
**! Если нужна разблокировка других сервисов, то смотрим на этот [пункт](#zapret-для-всех-блокировок) !**

5. Запускаем скрипт установки службы
   ```./install_easy.sh```

    И делаем следующее
    - Select firewall type - Выбираем на свое усмотрение
    - enable IPV6 suppor - Выбираем отталкиваясь от того какой версии IP вы пользуетесь
    - select MODE - рекомендую выбирать между tpws и nfqws. Я буду показывать напримере nfqws, т.к. работает она лучше.
    - do you want to edit the options - Да, мы хотим изменить, а потому вводим Y и изменяем опции
    ```
    NFQWS_OPT_DESYNC=""
    #NFQWS_OPT_DESYNC_SUFFIX=
    #NFQWS_OPT_DESYNC_HTTP=
    #NFQWS_OPT_DESYNC_HTTP_SUFFIX=
    #NFQWS_OPT_DESYNC_HTTPS=
    #NFQWS_OPT_DESYNC_HTTPS_SUFFIX=
    #NFQWS_OPT_DESYNC_HTTP6=
    #NFQWS_OPT_DESYNC_HTTP6_SUFFIX=
    #NFQWS_OPT_DESYNC_HTTPS6=
    #NFQWS_OPT_DESYNC_HTTPS6_SUFFIX=
    NFQWS_OPT_DESYNC_QUIC=""
    #NFQWS_OPT_DESYNC_QUIC_SUFFIX=
    #NFQWS_OPT_DESYNC_QUIC6=
    #NFQWS_OPT_DESYNC_QUIC6_SUFFIX=
    ```

    Всё что за # это комменатрии, их не трогаем. Нужны нам лишь "NFQWS_OPT_DESYNC" и *(опционально, в зависимости от того будете вы использовать протокол QUIC или нет)"NFQWS_OPT_DESYNC_QUIC"*
    >

    Выбираем способ обхода замедления: **split2**, **disorder2**, fake
    Я рекоменудую выбирать выделенные варианты. Теперь вписываем в "NFQWS_OPT_DESYNC" после ="
    ```NFQWS_OPT_DESYNC="--dpi-desync=split2```
    или же
    ```NFQWS_OPT_DESYNC="--dpi-desync=disorder2```
    или же
    ```NFQWS_OPT_DESYNC="--dpi-desync=fake --dpi-desync-repeats=6```
    </br>
    Для протокола QUIC
    
    ```NFQWS_OPT_DESYNC_QUIC="--dpi-desync=fake,split2 --dpi-desync-ttl=3"```
    Цифру 3 можно увеличить на 1 или уменьшить на 1, если не сработает. Увы, придется перебрать множество вариантов чтобы найти рабочий.

- После настройки опций сохраняемся и выходим.
- WAN interface - Рекомендую выбрать ANY
- enable http support - Если вы хотите разблокировать только YouTube, могу порекомендовать отключить, т.к. запросы на сервера Google происходят протоколу https. Вот примерная ссылка на сервер с видео Google:
  ```https://r2---sn-8vap5-3c2l.googlevideo.com```
- enable https support - Включаем
- enable quic support - Можно не включать, опять же в зависимости от того будете ли вы пользоваться протоколом QUIC или нет
- select filtering - Рекомендую выбирать hostlist
- do you want to auto download ip/host list - Скачиваем сам hostlist
- your choice - Рекомендую оставить по умолчанию
  **Готово!**
6. Далее вставим ссылки на сервера Google 
   ```
    googlevideo.com
    youtubei.googleapis.com
    i.ytimg.com
    yt3.ggpht.com
    ```
    В файл zapret-hosts-user.txt
    ```nano /opt/zapret/ipser/zapret-hosts-user.txt```
    Сохраняемся и выходим

7. **(Если хотите QUIC)** В вашем браузере зайдите в доп. настройки
- Chrome - ```chrome://flags/#enable-quic```
- Vivaldi - ```vivaldi://flags/#enable-quic```
- Opera - ```opera://flags/#enable-quic```
- Yandex - ```browser://flags/#enable-quic```
- FireFox - зайдите в ```about:config``` и в поиске введите ```network.http.http3.enable``` , Переставьте значение на true

 Для лучшего эффекта можно перезагрузить компьютер или же перезапустить браузер

## Zapret для всех блокировок
1. Запускаем скрипт для выявления стратегии обхода блокировок
   ```./blockcheck.sh```
-   specify domain(s) for test - По умолчанию выставлен rutracker.org, можно оставить. Но не выставляйте как тестовый домены YouTube, т.к. они замедлены, но не заблокированы полностью
-   ip protocol version(s) - Выбираем отталкиваясь от того какой версии IP вы пользуетесь
-   check http - Оставляем
-   check https tls 1.2 - Оставляем
-   check https tls 1.3 - Отключаем
-   do not verify server certificate - Отключаем
-   how many times to repeat each test - Выставляем кол-во запросов на сайт с одной стратегией
-   Выбор между quick standart force
  Quick - Быстрая проверка (15-30 минут)
  Standart - Классическая проверка (1-1.5 часов)
  Force - Максимальная проверка (более 2-ух часов)
- Теперь ожидаем окончание проверки.
- После окончания проверки появится итоговый результат. Выглядит он примерно так
  ```ipv4 rutracker.org curl_test_https_tls12:nfqws --dpi-desync=fake,split2 --dpi-desync-ttl=3```
  Важна нам часть после :
  - nfqws это MODE, который вы выбираете в скрипте install_easy.sh, у вас может быть и tpws
  - Строку ```--dpi-desync=fake,split2 --dpi-desync-ttl=3``` нам необходимо вставить в опцию ```NFQWS_OPT_DESYNC``` или ```NFQWS_OPT_DESYNC_QUIC```

2. Теперь после найденной стратегии запускаем скрипт ```./install_easy.sh```, доходим до "do you want to edit the options" и редактируем конфиг.

## ByeDPI на Android
1. Скачиваем программу с официального [репозитория](https://github.com/dovecoteescapee/ByeDPIAndroid). В релизах скачиваем [.apk](https://github.com/dovecoteescapee/ByeDPIAndroid/releases)
2. Устанавливаем на ваш телефон и запускаем.
3. Сразу не запускаем, а заходим в настройки с помощью шестерёнки в правом верхнем углу.
4. В графе "DNS" вводим ```8.8.8.8``` вместо ```1.1.1.1``` (то есть вводим DNS Google вместо Cloudflare)
5. Далее выбираем пункт UI editor
6. Тут уже можно поиграться с настройками и выявить стратегию обхода блокировок. Я например выставил такие параметры 
    **Desync**
    - Hosts - Disabled
    - Default TTL - 0
    - Desync method - Out-of-band
    - Split position - 1
    - Split at host - Выключено
    - Drop SACK - Выключено
    - OOB Data - a
   
   **Protocols**
   - Desync HTTP
   - Desync HTTPS
   - Desync UDP
  
    **HTTP**
   - Host mixed case - Включено
   - Domain mixed case - Включено
   - Host remove spaces - Выключено
  
    **HTTPS**
    - Split TLS record - Включено
    - TLS record split position - 1
    - Split TLS record at SNI - Включено

7. После всех мохинаций выходим из настроек программы и нажимаем кнопку Connect / Подключиться
8. **Готово**

## Решение возникших проблем

### Скрипты не являются исполняемыми
Для того чтобы дать скриптам права на исполнение вводим комманду
```chmod u+x "путь до скрипта"```

Если нужно наделить других пользователей полномочиями запускать скрипт, то вводим следующее
```chmod ugo+x "путь до скрипта"```

### Нет Bash
- Debian / Ubuntu / Debian подобные
  ```sudo apt-get install bash```

- Arch / Arch подобные
  ```sudo pacman -S base base-devel```

- Red Hat / Fedora / Red Hat подобные
  ```sudo dnf install bash```

- Gentoo
  ```emerge --ask --verbose app-shells/bash```

- NixOs
  ```sudo nix-env -u '*' ```

- OpenWrt / Keenetic
  ```opkg install bash```

### Zapret не запущен
Если служба не запущена, то запускаем её этой коммандой
```sudo systemctl start --now zapret.service```

Для остановки
```sudo systemctl stop --now zapret.service```

Для отключения автозагрузки
```sudo systemctl disable --now zapret.service```

Для добавления в автозагрузку
``````sudo systemctl enable --now zapret.service``````

Если же такой службы нет, то переходим к [пункту](#zapret-для-youtube-подходит-для-linux-и-роутеров) 5 и запускаем скрипт, вчитываясь в логи. Так можно найти ошибку.

### ByeDPI жрет много батареи
Решить это почти никак, т.к. по сути программа висит в фоне и весь ваш траффик пропускает через себя, модифицируя его. Но пару действий можно предпринять
- Можно включить оптимизацию батареи для данного приложения 
  1. Переходим в настройки телефона, ищем пункт "Приложения", там ищем "ByeDPI", нажимаем на приложение.
  2. Здесь есть пункт "Расход батареи / Экономия батареи / Использование батареи / App battery usage"
  3. И выбираем пункт "Optimized / Оптимизированый" и в таком духе. 
  4. Готово
- Могу дать совет, не используйте приложение в фоне постоянно. Посмотрели YouTube, условно говоря, выключили и всё.

### ByeDPI не работает (Не работает обход блокировок)
Выход один, тыкаться в настройках с надеждой "авось заработает". В конце концов цензоры на месте не сидят и занимаются своей задачей, мешая обходить их блокировки. Если хочется готовый ответ, то выбирайте в Desync method что-то из этого
  - Split
  - Disorder
  - Out-of-band

Fake не рекомендую для YouTube, т.к. он работает так чтобы пропускать траффик через "сайт прокладку", а это медленнее чем варианты указанные выше. У меня по крайней мере работал медленно.