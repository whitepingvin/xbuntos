---
layout: post
title: Шпаргалка для работы с GDB
permalink: gdb-notes
tags: gdb
---

Чтобы было куда подсмотреть.

---

## Как запустить на отладку программу с параметрами в gdb?

Два варианта:

- через командную строку:

        + gdb ./stbapp
        GNU gdb (GDB) 7.8.1
        ...........................
        Reading symbols from ./stbapp...done.
        (gdb) set args -platform eglfs:width=1920:height=1080:xpos=0:ypos=0 -plugin EvdevKeyboard::repeat-delay=200:repeat-rate=30 -plugin EvdevMouse: http://192.168.1.61:1111/client.html
        (gdb) show args
        Argument list to give program being debugged when it is started is "-platform eglfs:width=1920:height=1080:xpos=0:ypos=0 -plugin EvdevKeyboard::repeat-delay=200:repeat-rate=30 -plugin EvdevMouse: http://192.168.1.61:1111/client.html".

    Т.е. *set args arg1 arg2 argn*. По сути можно просто скопировать и вставить всю строку с параметрами, которая в "обычном" режиме передается приложению

- через файл .gdbinit. Его можно расположить рядом с бинарником, который надо отлаживать. Также надо разрешить его загрузку; для этого в $HOME/.gdbinit добавить следующее:

        add-auto-load-safe-path / #можно грузить .gdbinit'ы со всего корня, может быть конкретный путь

    Ну и в .gdbinit для конкретного приложения:
    
        set args arg1 arg2 argn  
        show args

    В принципе, при помощи .gdbinit можно автоматизировать отладчик, например:

        set args --disable-web-security --user-data-dir --allow-file-access-from-files --single-process http://localhost:1111/client.html
        show args
        set follow-fork-mode child
        set breakpoint pending on
        break WebChannelIPCTransport::installWebChannel(uint)
        break WebChannelIPCTransport::OnMessageReceived(const IPC::Message &)
        break WebChannelIPCTransport::dispatchWebChannelMessage(const std::vector<char> &, uint)


## Общие полезные штуки

- Список тредов:

        info threads

- переключиться на конкретный 2-й тред:

        thread 2

- колстек текущего треда:

        bt

## Документация

Годный туториал: <http://www.dirac.org/linux/gdb/>  
Убергодный туториал на все случаи жизни: <https://software.intel.com/sites/default/files/article/365160/gdb.pdf>
