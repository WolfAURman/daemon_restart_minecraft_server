# Автоматическая перезагрузка вашего Minecraft сервера #

## Предисловие ##

Все мы видели подобные плагины как [тут](https://www.spigotmc.org/resources/restartcountdown.82460/) или [тут](https://dev.bukkit.org/projects/ultimateautorestart-only-autorestart-plugin-you-will-ever-need). У них есть огромный недостаток - перезапуск в "пустоту".

У меня сервера запускаются в виртуальных консолях, по этой причине нужно было что б был перезапуск прямо там же, данные плагины увы не смогли это сделать.

Если вы используете ***Linux***, и у вас есть ***systemd***, можете продолжить чтение. Если у вас windows server, можете закрыть эту инструкцию, и принять мои соболезнования. xD


### Почему же systemd а не cron? ###

Крон валенок, и при рестарте вашего сервера пошлёт вас куда подальше, и скажет я не я, это не мой скрипт, я не помню когда его запускать. Если говорить без рофлов - крон собъётся при рестарте, в то время как systemd с его демонами - нет.

И так, нам понадобится:
- Прямые руки
- Немного времени
- Linux сервер
- SystemD
- Git

## Описание ##
Все скрипты похожи. Но в одном используются команды голого сервера для оповещения (say) в другом же команда (bc или broadcast) из essentialsX. Файл без ```ess``` не имеет команды ```bc``` и использует стандартную команду ```say```. 

Так же в одном используется без автозапуска скрипт, и демон поднимает его сам, во втором же убрано это, и скрипт запуска самого сервера должен будет поднять сервер. Тот что поднимает сам, имеет приписку auto в конце, для него нужен скрипт без автоподнятия в запускаторе (ЭТО ВАЖНО!), тот которому нужно поднимать непосредственно из скрипта запуска, не имеет данной приписки.

## Разберём что вам нужно будет настроить и что вам выбрать. ##
### Возьму в пример essrestart.service: ###
```bash
[Unit]
Description=Daemon systemd for restart server Minecraft

[Service]
Type=oneshot
ExecStart=tmux send -t main.0 'bc &c&lПерезагрузка будет произведена через 1 минуту' ENTER
ExecStart=sleep 30
ExecStart=tmux send -t main.0 'bc &c&lПерезагрузка будет произведена через 30 секунд' ENTER
ExecStart=sleep 15
ExecStart=tmux send -t main.0 'bc &c&lПерезагрузка будет произведена через 15 секунд' ENTER
ExecStart=sleep 5
ExecStart=tmux send -t main.0 'bc &c&lПерезагрузка будет произведена через 10 секунд' ENTER
ExecStart=sleep 5
ExecStart=tmux send -t main.0 'bc &c&lПерезагрузка будет произведена через 5 секунд' ENTER
ExecStart=sleep 5
ExecStart=tmux send -t main.0 'bc &c&lПерезагрузка будет произведена сейчас!' ENTER
ExecStart=sleep 4
ExecStart=tmux send -t main.0 'stop' ENTER

[Install]
WantedBy=multi-user.target
```
И так, что у нас тут что:
- ```tmux send -t``` передаёт значение в сессию ```main.0```. ```main``` вы можете заменить по своему усмотрению, это то как называется ваша сессия.
- Всё что написано тут ```'bc &c&lПерезагрузка будет произведена через 5 секунд'``` и в похожих строках кроме последней, вы так же можете отредачить, никто не запрещает.
- Последнюю строку ни в коем случае не трогаем, она передаёт команду ```stop``` для остановки сервера.
- ```ENTER``` передаёт нажатие кнопки ввод.
- Команда ```sleep``` замораживает выполнение следующей команды на кол-во секунд указанных после команды.

Это всё что вам нужно знать. Не важно как называется ваш файл, все ```.service``` файлы настраиваются одинаково.

### В essrestart.timer вы можете изменить значение строки с таймером: ###
```bash
[Unit]
Description=Daemon systemd for restart server Minecraft timer

[Timer]
OnCalendar=*-*-* 3:00:00

[Install]
WantedBy=timers.target
```
в строке ```OnCalendar=*-*-* 3:00:00``` вы можете заменить ```3:00:00``` на любое другое число, в 24-х часовом формате. Я поставил на 3 часа ночи. Это говорит о том что сервер будет каждые сутки в 3 часа ночи перезапускаться.

Не важно как называется ваш файл, суть у всех одна, файлы с расширением ```.timer``` настраиваются одинаково. Вроде как всё, больше вам ничего не нужно знать про редактирование данных файлов. Если вам станет интересно, вы можете почитать [тут](https://interface31.ru/tech_it/2021/04/nastraivaem-taymery-systemd-vmesto-zadaniy-cron.html), [тут](https://wiki.archlinux.org/title/Systemd_(%D0%A0%D1%83%D1%81%D1%81%D0%BA%D0%B8%D0%B9)/Timers_(%D0%A0%D1%83%D1%81%D1%81%D0%BA%D0%B8%D0%B9)) а так же [тут](https://wiki.archlinux.org/title/Systemd/Timers).

Если вы используете скрипт без автоподнятия (рекомендую его использовать) то вам нужно создать вот такой запускатор в папке с ядром:
```bash
#!/bin/bash

while [ true ]; do
java -jar -server -Xms6G -Xmx6G -XX:+UseLargePages -XX:LargePageSizeInBytes=2M -XX:+UnlockExperimentalVMOptions -XX:+UseShenandoahGC -XX:ShenandoahGCMode=iu -XX:+UseNUMA -XX:+AlwaysPreTouch -XX:-UseBiasedLocking -XX:+DisableExplicitGC -Dfile.encoding=UTF-8 название_ядра.jar --nogui

    echo Сервер поднимается...
    echo Нажмите CTRL + C для остановки.

done
```
Почему именно его рекомендую? Нет задержки, запускатор моментально поднимает сервер обратно, если же автоподнятие сделано в демоне, то там есть задержка, и не всегда есть уверенность что оно точно поднимется вовремя, ведь таймера в 30 секунд может не хватить на выключение сервера.

## Использование ##

Склонируйте весь репозиторий командой:
```bash
git clone https://github.com/WolfAURman/daemon_restart_minecraft_server
```

Либо если вам нужно получить конкретные скрипты, то сделайте это через curl:
```bash
curl -o https://raw.githubusercontent.com/WolfAURman/daemon_restart_minecraft_server/master/имена_нужных_файлов
```

После чего скопируйте нужные вам файлы по данному пути - ```/etc/systemd/system```

Проще всего это сделать командой cp:
```bash
sudo cp название_вашего_файла /etc/systemd/system
```

**ВАЖНО!** Вам нужно использовать два файла для переноса, с расширением ```.service``` и ```.timer```

после того как вы перенесли нужные вам файлы, а это таймер (.timer) и сам сервис (.service) вам необходимо их запустить. Сделаем это с помощью systemd:
```bash
systemctl enable название_вашего_таймера.timer && systemctl start название_вашего_таймера.timer
```
Всё, мы добавили нашего демона в автозагрузку. Не нужно добавлять самого демона сервиса в автозагрузку а уж тем более его запускать! Ничего страшного не будет, но не советую так делать.

Почему? Если мы запустим демона через systemctl start мы получим перезагрузку прямо сейчас, а если ещё после закинем его в автозагрузку через systemctl enable то у нас будут пуки в логи, что у нас не запущенно такой сессии и рестарт окончился ошибкой.

# Наведение марафета в конфигах для красоты #

## Изменение сообщения при выключении сервера ##

Откройте ваш ```bukkit.yml```. Там вы увидите такое содержание:
```bash
settings:
  allow-end: true
  warn-on-overload: true
  permissions-file: permissions.yml
  update-folder: update
  plugin-profiling: false
  connection-throttle: 4000
  query-plugins: true
  deprecated-verbose: default
  shutdown-message: §aСервер перезагружается...
  minimum-api: none
  use-map-color-cache: true
spawn-limits:
  monsters: 70
  animals: 10
  water-animals: 5
  water-ambient: 20
  water-underground-creature: 5
  axolotls: 5
  ambient: 15
chunk-gc:
  period-in-ticks: 600
ticks-per:
  animal-spawns: 400
  monster-spawns: 1
  water-spawns: 1
  water-ambient-spawns: 1
  water-underground-creature-spawns: 1
  axolotl-spawns: 1
  ambient-spawns: 1
  autosave: 6000
aliases: now-in-commands.yml
```

Измените значение в строке ```shutdown-message:``` на какое-то красивое предложение. Я использовал ```§aСервер перезагружается...``` что вполне выглядит красиво и очень даже к месту:

![Screenshot from 2022-08-15 15-27-14](https://user-images.githubusercontent.com/93985232/184945955-d7b42bee-ab8e-4d52-a41b-20bfd94a7064.png)

## Изменение внешнего вида от кого пишется сообщение при использовании bc ##

Откройте essentialsX.jar любым архиватором, и найдите файл ```messages_ru.properties```. Там же найдите такие строки:
```bash
broadcast=§b§lSERVER: {0}
broadcastCommandDescription=Объявляет сообщение на весь сервер.
broadcastCommandUsage=/<command> <сообщение>
broadcastCommandUsage1=/<command> <сообщение>
broadcastCommandUsage1Description=Объявляет указанное сообщение на весь сервер
broadcastworldCommandDescription=Объявляет сообщение в мире.
broadcastworldCommandUsage=/<command> <мир> <сообщение>
broadcastworldCommandUsage1=/<command> <мир> <сообщение>
broadcastworldCommandUsage1Description=Объявляет указанное сообщение в указанном мире
```
Вам нужно заменить значение в первой строке - ```broadcast=``` я лично поставил ```§b§lSERVER: {0}``` и заметно, и вроде не так ущербно выглядит:

![Screenshot from 2022-08-15 15-27-09](https://user-images.githubusercontent.com/93985232/184945785-85f2fa75-b085-48bb-9e64-e1ba1a6ef47e.png)
