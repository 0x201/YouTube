# Как обойти DPI

В данном гайде я расскажу как исправить проблему зависающего YouTube с помощью различных программ обхода блокировок DPI (Deep Packet Inspection / Глубокое исследование пакетов). В работу DPI погружать не буду, а сразу перейду к делу. Но если есть желание узнать об этой технологии, вам [сюда](https://web.archive.org/web/20230331233644/https://habr.com/ru/post/335436/), или [туда](https://ru.wikipedia.org/wiki/Deep_packet_inspection), или [here](https://geneva.cs.umd.edu/papers/geneva_ccs19.pdf).

|_[Как обойти DPI](#как-обойти-dpi)<br />
  &nbsp;&nbsp;&nbsp;&nbsp;| [GoodByeDPI (Windows)](#goodbyedpi-windows)<br />
  &nbsp;&nbsp;&nbsp;&nbsp;| [Zapret для YouTube (Linux / Роутеры)](#zapret-для-youtube-подходит-для-linux-и-роутеров)<br />
  &nbsp;&nbsp;&nbsp;&nbsp;| [Zapret для всех блокировок](#zapret-для-всех-блокировок)<br />
  &nbsp;&nbsp;&nbsp;&nbsp;| [SpoofDPI (MacOS / Linux)](#spoofdpi-linux--macos)<br />
  &nbsp;&nbsp;&nbsp;&nbsp;| [ByeDPI (Android)](#byedpi-на-android)<br />
  &nbsp;&nbsp;&nbsp;&nbsp;| _ [Решение возникших пробелем](#решение-возникших-проблем)<br />
  &nbsp;&nbsp;&nbsp;&nbsp; &nbsp;&nbsp;&nbsp;&nbsp;| [Скрипты не являются исполняемыми](#скрипты-не-являются-исполняемыми)<br />
  &nbsp;&nbsp;&nbsp;&nbsp; &nbsp;&nbsp;&nbsp;&nbsp;| [Нет Bash](#нет-bash)<br />
  &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; | [ByeDPI жрет много батереи](#byedpi-жрет-много-батареи)<br />
  &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; | [ByeDPI не работает (Не работает обход блокировок)](#byedpi-не-работает-не-работает-обход-блокировок)<br />

## GoodByeDPI (Windows)
1. Скачайте архив с [GoodByeDPI](https://github.com/ValdikSS/GoodbyeDPI/releases)
2. Распаковываем в любую удобную для нас папку.
3. Заходим в неё и редактируем файл:
  
   - Если вам нужен YouTube рекомендую пользоваться
    ```
    1_russia_blacklist_YOUTUBE.cmd
    1_russia_blacklist_YOUTUBE_ALT.cmd
    ```
   - Ну а если нужна раблокировка всех сайтов, но необязательно YouTube, то открываем 
    ```1_russia_blacklist.cmd```

   - Не рекомендуется использовать скрипты 

     ```
     service_install_russia_blacklist.cmd
     service_install_russia_blacklist_dnsredir.cmd
     service_install_russia_blacklist_YOUTUBE.cmd
     service_install_russia_blacklist_YOUTUBE_ALT.cmd
     ```

Так как они устанавливают службу в Windows, а потом их сложно будет выкорчевывать из системы. 
    

1. После строчки ```goodbyedpi.exe``` вместо ```-9``` вставим цифру в соответствии от вашего [провайдера](https://github.com/ValdikSS/GoodbyeDPI/issues/378).
2. Ну а после ```--fake-from-hex``` всталяем рандомный hex, с помощью [hex генератора](https://www.browserling.com/tools/random-hex). В графе "How many digits?" вводим 120 и более, а после нажимаем "Generate Hex", копируем и вставляем в файл.
3. Сохраняем и выходим.
4. И запускаем нужный нам скрипт.
5. Готово.

## Zapret для YouTube (Linux / Роутеры)
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
  
    Всё что за # - комменатрии, их не трогаем. Нужны нам лишь "NFQWS_OPT_DESYNC" и *(опционально, в зависимости от того будете вы использовать протокол QUIC или нет) "NFQWS_OPT_DESYNC_QUIC"*
    <br />

    Выбираем способ обхода замедления. Теперь вписываем в "NFQWS_OPT_DESYNC" после ="
    ```NFQWS_OPT_DESYNC="--dpi-desync=split2```
    или же
    ```NFQWS_OPT_DESYNC="--dpi-desync=disorder2```
    или же
    ```NFQWS_OPT_DESYNC="--dpi-desync=fake --dpi-desync-repeats=6```
    
    Для протокола QUIC
    
    ```NFQWS_OPT_DESYNC_QUIC="--dpi-desync=fake,split2 --dpi-desync-ttl=3"```
    Цифру 3 можно увеличить на 1 или уменьшить на 1, если не сработает. Увы, придется перебрать множество вариантов чтобы найти рабочий.

- После настройки опций сохраняемся и выходим.
- WAN interface - Рекомендую выбрать ANY
- enable http support - Включаем
- enable https support - Включаем
- enable quic support - Можно не включать, опять же в зависимости от того будете ли вы пользоваться протоколом QUIC или нет
- select filtering - Рекомендую выбирать hostlist
- do you want to auto download ip/host list - Скачиваем сам hostlist
- your choice - Рекомендую оставить по умолчанию
  
  **Готово!**
1. Далее вставим ссылки на сервера Google 
   ```
    play.google.ru
    play.google.com
    account.youtube.com
    youtube.com
    www.youtube.com
    i.ytimg.com
    studio.youtube.com
    googlevideo.com
    yt3.ggpht.com
    yt4.ggpht.com
    youtu.be
    yt.be
    ytimg.com
    ggpht.com
    gvt1.com
    youtube.googleapis.com
    youtubeembeddedplayer.googleapis.com
    ytimg.l.google.com
    nhacmp3youtube.com
    jnn-pa.googleapis.com
    youtube-nocookie.com
    youtube-ui.l.google.com
    yt-video-upload.l.google.com
    wide-youtube.l.google.com
    ```
    В файл zapret-hosts-user.txt
    ```nano /opt/zapret/ipser/zapret-hosts-user.txt```
    Сохраняемся и выходим 
    
2. **(Если хотите QUIC)** В вашем браузере зайдите в доп. настройки
- Chrome - ```chrome://flags/#enable-quic```
- Vivaldi - ```vivaldi://flags/#enable-quic```
- Opera - ```opera://flags/#enable-quic```
- Yandex - ```browser://flags/#enable-quic```
- FireFox - зайдите в ```about:config``` и в поиске введите ```network.http.http3.enable``` , переставьте значение на true

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
- После окончания проверки появится итоговый результат. Выглядит он примерно так **(ЭТО НЕ ГОТОВЫЙ КОНФИГ)**:
  ```ipv4 rutracker.org curl_test_https_tls12:nfqws --dpi-desync=fake,split2 --dpi-desync-ttl=3```
  Важна нам часть после :
  - nfqws это MODE, который вы выбираете в скрипте install_easy.sh, у вас может быть и tpws
  - Строку ```--dpi-desync=fake,split2 --dpi-desync-ttl=3``` нам необходимо вставить в опцию ```NFQWS_OPT_DESYNC``` или ```NFQWS_OPT_DESYNC_QUIC``` 


2. Теперь после найденной стратегии запускаем скрипт ```./install_easy.sh```, доходим до "do you want to edit the options" и редактируем конфиг.

## SpoofDPI (Linux / MacOS)
1. Скачиваем [SpoofDPI](https://github.com/xvzc/SpoofDPI) с помощью комманд:
   - MacOS Intel
   ```curl -fsSL https://raw.githubusercontent.com/xvzc/SpoofDPI/main/install.sh | bash -s darwin-amd64```
   - MacOS Apple Silicon
   ```curl -fsSL https://raw.githubusercontent.com/xvzc/SpoofDPI/main/install.sh | bash -s darwin-arm64```
   - Linux amd64 (x86/x64)
   ```curl -fsSL https://raw.githubusercontent.com/xvzc/SpoofDPI/main/install.sh | bash -s linux-amd64```
   - Linux arm
   ```curl -fsSL https://raw.githubusercontent.com/xvzc/SpoofDPI/main/install.sh | bash -s linux-arm```
   - Linux arm64
   ```curl -fsSL https://raw.githubusercontent.com/xvzc/SpoofDPI/main/install.sh | bash -s linux-arm64```
2. Далее в зависимости от того какая у вас оболочка терминал Bash или Zsh, редактируем файл 
   - Zsh - ```nano ~/.zshrc```
   - Bash - ```nano ~/.bashrc```

Проверить какая у вас стоит оболочка можно с помощью комманды ```echo $SHELL```

3. В конфиге оболочки вставляем строку 
  ```export PATH=$PATH:~/.spoofdpi/bin```

  Чтобы вводя ```spoofdpi``` у вас открывалась эта программа. Мы создали некоторую ассоциацию с программой.

4. Необходимо перезапустить терминал, а лучше перезагрузить компьютер.
5. Теперь запускаем программу (можем также добавить пару [параметров](https://github.com/xvzc/SpoofDPI/blob/main/_docs/README_ru.md#%D0%B8%D1%81%D0%BF%D0%BE%D0%BB%D1%8C%D0%B7%D0%BE%D0%B2%D0%B0%D0%BD%D0%B8%D0%B5))
   ```spoofdpi```

6. Программа запущена, но она не будет работать, т.к. мы должны запустить браузер с параметрами прокси на loopback
   ```google-chrome --proxy-server="http://127.0.0.1:8080"```

   Вместо "google-chrome" вводим свой браузер будь-то

   ```chromium --proxy-server="http://127.0.0.1:8080"```
   ```firefox --proxy-server="http://127.0.0.1:8080"```
   ```vivaldi --proxy-server="http://127.0.0.1:8080"```

7. Готово. Всё должно заработать.

Могу посоветовать пару параметров для этой программы
```spoofdpi```

## ByeDPI на Android
1. Скачиваем программу с официального [репозитория](https://github.com/dovecoteescapee/ByeDPIAndroid). В релизах скачиваем [.apk](https://github.com/dovecoteescapee/ByeDPIAndroid/releases)
2. Устанавливаем на ваш телефон и запускаем.
3. Сразу не запускаем, а заходим в настройки с помощью шестерёнки в правом верхнем углу.
4. **(Необязательно, т.к. сервера Google медленее (либо и их замедляют))** В графе "DNS" вводим ```8.8.8.8``` вместо ```1.1.1.1``` (то есть вводим DNS Google вместо Cloudflare)
5. Далее выбираем пункт UI editor
6. Тут уже можно поиграться с настройками и выявить стратегию обхода блокировок.
7. После всех мохинаций выходим из настроек программы и нажимаем кнопку Connect / Подключиться
8. **Готово**

Или же можно вписать параметры в консоли. Найти их можно на этом [форуме](https://4pda.to/forum/index.php?showtopic=1092092)

## Решение возникших проблем
В данном разделе вы скорее всего найдёте решение вашей проблемы.
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
Выход один, тыкаться и подбирать консольные параметры в настройках с надеждой "авось заработает". В конце концов цензоры на месте не сидят и занимаются своей задачей, мешая обходить их блокировки. Вот [здесь](https://4pda.to/forum/index.php?showtopic=1092092) или [здесь](https://github.com/dovecoteescapee/ByeDPIAndroid/discussions/64) можно найти всё готовенькое, но не факт.

**Автор: 0x201 :3**
