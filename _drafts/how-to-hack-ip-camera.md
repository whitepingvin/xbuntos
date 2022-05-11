---
layout: post
title: "Как я ip камеру настраивал"
permalink: ip-camera-hack
tags: 
---



---

rtsp://login:passwd@<ip>:<port>/tcp/av0_0
mplayer rtsp://login:passwd@<ip>:<port>/tcp/av0_0
web page: http://<ip>:<port>/admin.htm
webview: http://192.168.1.179:28843

Как похакать ip camera Vstarcam c7837wip

Берем web ui отсюда(http://4pda.ru/forum/lofiversion/index.php?t782299-340.html), архив CH-app-EN53.8.1.14_VSTARCAM

	(Для камер с прошивкой 48.53.72.80 и выше: Начиная с прошивки 48.53.72.80 в камере отключен telnet. Запускаем следующим образом: Зайти в вебморду через браузер, перейти к вкладке "Настройка FTP" и в поле сервера ввести: $(telnetd), сохранить и нажать кнопочку "Тест". telnet запущен.
	В последних вебмордах отсутствуют Mail Settings и FTP Settings. Решается заменой Веб интерфейса на CH-app-EN53.8.1.14_VSTARCAM из шапки. Замена через стандартный "Upgrade Web UI".


https://github.com/linkingvision/rapidonvif
http://veyesys.com/
https://sourceforge.net/projects/onvifcpplib/
http://4pda.ru/forum/index.php?showtopic=782299

https://www.fontenay-ronan.fr/c7824wip-security-review/

http://avreg.net/manual_applications_cam-sniff.html
http://www.electronicsfaq.com/2016/11/automatically-rebooting-vstarcam-hd.html
https://www.ispyconnect.com/man.aspx?n=Vstarcam#

Playing H264 from vstarcam:
https://ubuntuforums.org/showthread.php?t=2334252&page=2

http://www.ekzorchik.ru/2017/03/how-to-install-motioneye-on-raspberry/



[Веб интерфейс для motion](https://github.com/ccrisan/motioneye/wiki/Install-On-Ubuntu)
