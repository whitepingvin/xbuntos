---
layout: post
title: Release и Debug сборки при использовании Сmake в Qt Creator
permalink: release-and-debug-with-cmake-and-qtcreator/
tags: cmake qtcreator
---

Долгое время использовал `qmake`, который по умолчанию используется в `Qt Creator`, для сборки С/С++ проектов. "Из коробки" сразу доступны и отладочная и релизная сборки, что хорошо и приятно:smile:.

Также `Qt Creator` может использовать для сборки `сmake`, что позволяет обходится вообще без кутешного инструментария. Но по умолчанию сборка проекта доступна лишь в релизном режиме. Что бы добавить возможность отладочной сборки необходимо добавить в `CMakeLists.txt` следующий код:

{% highlight cmake %}
if(CMAKE_BUILD_TYPE STREQUAL "Debug")
set(CMAKE_LIBRARY_OUTPUT_DIRECTORY ${CMAKE_SOURCE_DIR}/lib-debug)
set(LIB_INSTALL_DIR ${CMAKE_SOURCE_DIR}/lib-debug)
set(EXECUTABLE_OUTPUT_PATH ${CMAKE_SOURCE_DIR}/bin-dbg)
else()
set(CMAKE_LIBRARY_OUTPUT_DIRECTORY ${CMAKE_SOURCE_DIR}/lib)
set(LIB_INSTALL_DIR ${CMAKE_SOURCE_DIR}/lib)
set(EXECUTABLE_OUTPUT_PATH ${CMAKE_SOURCE_DIR}/bin)
endif()
{% endhighlight %}

И потом в `QtCreator` при открытии(или перегенерации) проекта в появившемся окне в "Параметрах" ввести `-DCMAKE_BUILD_TYPE=Debug`, после чего запустить генерацию проекта нажатием на кнопку "Запустить СMake".
После успешной генерации появится два режима сборки:

- all - релизная сборка;
- debug-mode - отладочная сборка.
