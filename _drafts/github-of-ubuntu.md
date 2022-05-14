---
layout: post
title: "Установка GitHubDesktop на Ubuntu"
permalink: github-of-ubuntu
tags: github
---

![top_img](./assets/github-of-ubuntu/githubdesktop.png)   
Шаг 1 :       
Используйте команду wget:   
> sudo wget https://github.com/shiftkey/desktop/releases/download/release-2.9.0-linux2/GitHubDesktop-linux-2.9.0-linux2.deb

Шаг 2 :   
Установите gdebi с помощью следующей команды:

> sudo apt-get install gdebi-core



Шаг 3 :   
После установки gdebi с помощью gdebi установите клиент GitHubDesktop

> sudo gdebi GitHubDesktop-linux-2.9.0-linux2.deb

---

Альтернативные способы установки после шага 1:
Вы также можете использовать команды dpkg или apt.

Способ 1:

> sudo dpkg -i GitHubDesktop-linux-2.9.0-linux2.deb

> sudo apt-get install -f

Способ 2:

> sudo apt install ./GitHubDesktop-linux-2.9.0-linux2.deb
