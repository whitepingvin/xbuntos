---
layout: post
title: "Про компоновку, dependency hell и обратную совместимость"
permalink: about-linking-dependency-hell-and-backward-compatibility
tags: dll linking dependency-hell C C++
---

В статье речь пойдет о высокоуровневом взгляде на компоновку. Где ищутся разделяемые библиотеки на Linux, BSD*, Mac OS X, Windows, от которых зависят приложения? Что делать с обратной совместимостью? Как бороться с адом зависимостей? На основе этой [статьи](http://habrahabr.ru/post/220961/).

---

## Содержание
- [Проблемы статической загрузки динамических библиотек](#problems)
- [Побег из ада](#escape_from_hell)
- [ELF & GNU ld (Linux, *BSD, etc)](#linux)
- [Как обстоят дела на Маке?](#macos)
- [И наконец, Windows!](#windows)
- [Дополнительные замечания о С++](#cpp_lang)
- [Заключение](#summary)
- [Ссылки](#refs)



## Проблемы статической загрузки динамических библиотек:     {#problems}

1. <img src="/assets/2015-01-18-about-linking-dependency-hell-and-backward-compatibility/1.png" width="300" align="right"/>main.exe зависит от version-0.3.dll и bar.dll. bar в свою очередь, зависит от version-0.2.dll, которая бинарно не совместима с версией 0.3 (не просто символы отсутствуют, а совпадают имена, но различное число аргументов, или создают объекты разной природы и т. п.). Затрут ли символы из version-0.2.dll оные из version-0.3.dll? Тот же вопрос стоит тогда, когда используется одна статическая версия библиотеки (скажем, version-0.2.lib) и динамическая (version-0.3.dll);
2. создание перемещаемых приложений: где динамический загрузчик будет искать version-0.?.dll и bar.dll для приложения из предыдущего пункта? Найдёт ли он зависимости main.exe, если тот будет перемещен в другую папку? Как нужно собрать main.exe, чтобы зависимости искались относительно исполняемого файла?
3. dependency hell: две версии одной библиотеки `/opt/kde3/lib/libkdecore.so` и `/opt/kde4/lib/libkdecore.so` (с которой плазма уже не падает), половина программ требуют первую, другая половина программ — вторую. Обе библиотеки нельзя поместить в одну область видимости (один каталог). Эта же проблема есть и в п. 1, т. к. надо поместить две версии библиотеки version в один каталог.

После прочтения первого пункта читатель может воскликнуть: "Да это извращение! Так не делают! Надо использовать одну версию библиотеки!" Да, это так, но в жизни всякое бывает. Простейший пример: ваше приложение использует библиотеку `а`, и стороннюю закрытую библиотеку (я подчеркиваю это, **чужой платный продукт**), которая ну вот никак не может использовать ту же версию `а`, что и вы или наоборот.

Другим примером в рамках огромного проекта служит тот факт, что выпуск различных частей имеет разный период. Таким образом, главный продукт может время от времени выходить в конфигурации, описанной в пункте 1.

[Dependency hell](https://ru.wikipedia.org/wiki/Dependency_hell) больше актуален для разработчиков системных библиотек и операционных систем, но и в прикладной области может возникнуть. Опять же предположим, что имеется огромный проект, в котором несколько исполняемых программ. Все они зависят от одной библиотеки, но разных версий. (Это не та же ситуация, что и в п. 1: там в один процесс загружается две версии одной библиотеки, а здесь в каждый процесс загружается только одна, но каждый использует свою версию).

## Побег из ада  {#escape_from_hell}

Ответ прост: надо добавить версию в имя файла библиотеки. Это позволит размещать файлы библиотек в одном каталоге. При этом рекомендуется добавлять версию ABI, а не API, порождая тем самым две параллельных ветки версий и соответствующие трудности.

Контроль версии – очень рутинная работа. Рассмотрим схему x.y.z:

- x – мажорный выпуск. Ни о какой совместимости речи не идет;
- y – минорный выпуск. Библиотека совместима на уровне исходных текстов, но двоичная совместимость может быть сломана;
- z – исправление ошибки. Библиотека совместима в обе стороны.

Тогда в имя файла разумно включить x.y. Если при увеличении минорной версии совместимость сохранили, то достаточно сделать соответствующий симлинк:

version-1.1.dll  
version-1.0.dll → version-1.1.dll  

Будут работать и приложения, использующие version-1.1.0, и те, кто использует version-1.0.x.

Если совместимость сломали, то в системе будет два файла и снова всё будет работать.

Если по каким-то причинам совместимость сломали при багфиксе, то должна быть увеличена минорная версия (и нечего не ломается, как это сделала команда Qt <sup><a name="cite_ref-qtmultimedia501"></a><a href="#cite_note-qtmultimedia501">[1]</a></sup>).

Кстати говоря, никто не запрещает вообще включить версию API – тогда символических ссылок будет больше, т.к. совместимость чаще сохраняется. Зато в этом случае упомянутый баг в Qt решился бы легко и не заставил увеличивать минорную версию.

Это справедливо для всех платформ.

Решение оставшихся двух вопросов отличается в зависимости от ОС.

## ELF & GNU ld (Linux, *BSD, etc)   {#linux}

В разделяемой библиотеке формата ELF присутствует так называемое SONAME <sup><a name="cite_ref-soname-wiki"></a><a href="#cite_note-soname-wiki">[2]</a></sup><sup><a name="cite_ref-soname-so-howto"></a><a href="#cite_note-soname-so-howto">[3]</a></sup>. Это &ndash; строка символов, которая прописывается в двоичный файл в секцию DT_SONAME. Просмотреть SONAME для библиотеки можно, например, так:

{% highlight bash %}
$ objdump -p /path/to/file | grep SONAME
{% endhighlight %}

Если программа/библиотека faz связывается с библиотекой baz, которая имеет SONAME = <u>baz-0.dll</u>, то строка <u>baz-0.dll</u> будет жестко прописана в двоичном файле faz в секции DT_NEEDED, и при его запуске динамический загрузчик будет искать файл с именем <u>baz-0.dll</u>. При этом никто не запрещает назвать файл по-другому!

Просмотреть SONAME'ы, от которых зависит исполняемый файл можно так:

{% highlight bash %}
$ objdump -x /path/to/file | grep NEEDED
{% endhighlight %}

Динамический загрузчик ищет библиотеки из секции DT_NEEDED в следующих местах в данном порядке <sup><a name="cite_ref-man-ld-linux_so"></a><a href="#cite_note-man-ld-linux_so">[4]</a></sup><sup><a name="cite_ref-rpath-wiki"></a><a href="#cite_note-rpath-wiki">[5]</a></sup>:

1. список каталогов в секции DT_RPATH, которая жёстко прописана в исполняемом файле. Поддерживается большинством *nix-систем. Игнорируется, если присутствует секция DT_RUNPATH;
2. LD_LIBRARY_PATH &ndash; переменная окружения, также содержит список каталогов;
3. DT_RUNPATH &ndash; тоже самое, что и DT_RPATH, только просматривается после LD_LIBRARY_PATH. Поддерживается только на самых свежих Unix-подобных системах;
4. /etc/ld.so.conf &ndash; файл настроек динамического загрузчика ld.so, который содержит список папок с библиотеками;
5. жёстко зашитые пути &ndash; обычно /lib и /usr/lib.

Формат данных для RPATH, LD_LIBRARY_PATH и RUNPATH такой же, как и для PATH: список путей, разделенных двоеточием. Просмотреть RUNPATH'ы можно, например, так:

{% highlight bash %}
$ objdump -x /path/to/file | egrep 'R(|UN)PATH'
{% endhighlight %}

R[UN]PATH может содержать специальную метку `$ORIGIN`, которую динамический загрузчик развернет в полный абсолютный путь до загружаемой сущности. Здесь стоит отметить, что некоторые разработчики добавляют в RUNPATH `.` (точку). Это не тоже самое, что `$ORIGIN`! Точка развернется в текущий рабочий каталог, который естественно не обязан совпадать с путем до сущности!

Для демонстрации написанного, разработаем приложение по схеме из п. 1 (ссылка на хранилище в [гитхабе](https://github.com/gshep/linking-sample) ). Чтобы собрать всю систему достаточно перейти в корень папки и вызвать `./linux_make_good.sh`, результат будет в папке `result`. Ниже будут разобраны некоторые этапы сборки.

На этапе компоновки библиотек version-0.x задаются SONAME:
{% highlight bash %}
$ gcc -shared -Wl,-soname,version-0.3.dll -o version-0.3.dll version.o
{% endhighlight %}

Они зависят только от системных библиотек и поэтому не требуют наличия секций R[UN]PATH.

Библиотека bar уже зависит от version-0.2, поэтому нужно указать RPATH:

{% highlight bash %}
$ gcc -shared -Wl,-rpath-link,/path/to/version-0.2/ -L/path/to/version-0.2/ -l:version-0.2.dll -Wl,-rpath,\$ORIGIN/ -Wl,--enable-new-dtags -Wl,-soname,bar.dll -o bar.dll bar.o
{% endhighlight %}

Параметр `--enable-new-dtags` указывает компоновщику заполнить секцию DT_RUNPATH.

Параметр `-Wl,-rpath,...` позволяет заполнить секцию R[UN]PATH. Для задания списка путей можно указать параметр несколько раз, либо перечислить все пути через двоеточие:

{% highlight bash %}
$ gcc -Wl,-rpath,/path/1/ -Wl,-rpath,/path/2 ...
$ gcc -Wl,-rpath,/path/1:/path/2 ...
{% endhighlight %}

Теперь всё содержимое папки result целиком или саму папку можно перемещать по файловой системе как угодно, но при запуске динамический загрузчик найдет все зависимости и программа исполнится:

{% highlight bash %}
$ ./result/main.exe
Hello World!
bar library uses libversion 0.3.0, number = 3
version::get_version result = 0
But I uses liversion 0.3.0
number = 3
{% endhighlight %}

Вот мы и подошли к проблеме затирания символов! Bar использует version-0.2.dll, в которой `get_number()` возвращает 2, а само приложение version-0.3.dll, где та же функция возвращает уже 3. По выводу приложения видно, что одна версия функции `get_number()` затирается другой.

Дело в том <sup><a name="cite_ref-linkers-loaders"></a><a href="#cite_note-linkers-loaders">[6; Dynamic Linking and Loading, Comparison of dynamic linking approaches]</a></sup>, что GNU ld & ELF не использует SONAME или имя файла в качестве пространства имен для импортируемых символов: если разные библиотеки экспортируют сущности с одними и теми же именами, то одни из них будут перетирать другие и в лучшем случае программа упадет.

Случай, когда одна из библиотек суть статическая, решается просто: все символы статической библиотеки должны быть скрыты <sup><a name="cite_ref-dsohowto"></a><a href="#cite_note-dsohowto">[7, 2.2.2 Define Global Visibility]</a></sup>.

К сожалению, в случае динамических библиотек не всё так просто. У компоновщика/загрузчика GNU отсутствует такая функциональность, как прямое связывание <sup><a name="cite_ref-direct-binding"></a><a href="#cite_note-direct-binding">[8]</a></sup>. Кто-то разрабатывал эту возможность в Генту <sup><a name="cite_ref-gentoo-direct-binding"></a><a href="#cite_note-gentoo-direct-binding">[9]</a></sup>, но кажется, всё заглохло. В Solaris она есть <sup><a name="cite_ref-solaris-direct-binding"></a><a href="#cite_note-solaris-direct-binding">[10]</a></sup><sup><a name="cite_ref-solaris-no-dll-hell"></a><a href="#cite_note-solaris-no-dll-hell">[11]</a></sup>, но сама Solaris перестала развиваться.

Одним из возможных вариантов является версионирование самих символов <sup><a name="cite_ref-dsohowto-1"></a><a href="#cite_note-dsohowto">[7, 2.2.5 Use Export Maps]</a></sup>. На самом деле это больше похоже на декорирование символов. (Можно только представлять, что сейчас кричит читатель, программирующий на Си++...)

Данный способ заключается в том, чтобы создать так называемый версионный сценарий, в котором перечислить все экспортируемые и скрытые сущности <sup><a name="cite_ref-ld-version-0"></a><a href="#cite_note-ld-version-0">[12]</a></sup><sup><a name="cite_ref-ld-version-1"></a><a href="#cite_note-ld-version-1">[13]</a></sup>. Пример сценария из version-0.3:

    VERSION_0.3 {
        global:
            get_version;
            get_version2;
            get_number;
        local:
            *;
    };

На этапе компоновки указать данный файл с помощью параметра `--version-script=/path/to/version.script`. После этого приложение, которое будет связано с такой библиотекой, получит в NEEDED version-0.3.dll, а в таблице импорта неопределенный символ `get_number@@VERSION_0.3`, хотя в заголовочных файлах по-прежнему будет просто `int get_number()`.

Натравите nm на любую программу, которая использует glibc, и вы прозреете!

Чтобы собрать пример с использованием версионирования символов в библиотеках version-0.x запустите корневой сценарий `linux_make_good.sh` с параметром:

{% highlight bash %}
$ ./linux_make_good.sh use_version_script
$ ./result/main.exe
Hello World!
bar library uses libversion 0.2.0, number = 2
version::get_version result = 0
But I uses liversion 0.3.0
number = 3

$ nm ./result/main.exe
// …
                 U get_number@@VERSION_0.3
                 U get_version@@VERSION_0.3
0000000000401008 T main
                 U memset@@GLIBC_2.2.5

$ nm ./result/bar.dll
// …
                 U get_number@@VERSION_0.2
                 U get_version2@@VERSION_0.2
0000000000000800 t register_tm_clones
                 U strcat@@GLIBC_2.2.5
{% endhighlight %}

\- Эй, Дарт! Наша libX будет поддерживать Линукс!  
\- Noooooooooooooooooooooooooo!  
![](/assets/2015-01-18-about-linking-dependency-hell-and-backward-compatibility/dart.jpg)

После неудачи, с которым наша команда столкнулась, было принято волевое решение использовать только одну версию библиотеки (именно из-за Линукса).

## Как обстоят дела на Маке? {#macos}

Мак Ось использует формат Mach-o для исполняемых файлов, а для поиска символов двухуровневое пространство имен <sup><a name="cite_ref-macosx-man-ld"></a><a href="#cite_note-macosx-man-ld">[14, Two-level namespace]</a></sup><sup><a name="cite_ref-macho-layout"></a><a href="#cite_note-macho-layout">[16]</a></sup>. Это по-умолчанию сейчас, но можно собрать с плоским пространством имен или вообще отключить его при запуске программы <sup><a name="cite_ref-macosx-man-dyld-0"></a><a href="#cite_note-macosx-man-dyld">[15, FORCE_FLAT_NAMESPACE]</a></sup>. Узнать, собран ли исполняемый файл с поддержкой пространства имен поможет команда:

{% highlight bash %}
$ otool -hv /path/to/binary/file
{% endhighlight %}

То есть не надо париться с каким-то дополнительным декорированием имен – просто включить версию в имя файла!

А что же с поиском зависимостей?

В макоси почти всё аналогично, только называется по-другому.

Вместо SONAME есть id библиотеки или install name. Просмотреть можно, например, так:

{% highlight bash %}
$ otool -D /usr/lib/libstdc++.dylib
{% endhighlight %}

Изменить можно с помощью install_name_tool.
При связывании с библиотекой её id прописывается в исполняемом файле.

Просмотреть зависимости исполняемого файла можно так:

{% highlight bash %}
$ otool -L /path/to/main.exe
{% endhighlight %}

или

{% highlight bash %}
$ dyldinfo -dylibs /path/to/main.exe
{% endhighlight %}

При запуске dyld пытается открыть файл с именем «id» <sup><a name="cite_ref-macosx-man-dyld-1"></a><a href="#cite_note-macosx-man-dyld">[15, DYNAMIC LIBRARY LOADING]</a></sup>, т. е. рассматривает install name как абсолютный путь к зависимости. Если потерпел неудачу &ndash; то ищет файл с именем/суффиксом «id» в каталогах, перечисленных в переменной окружения DYLD_LIBRARY_PATH (полный аналог LD_LIBRARY_PATH).

Если поиск в DYLD_LIBRARY_PATH не дал результатов, то dyld аналогично просматривает ещё парочку переменных окружения <sup><a name="cite_ref-macosx-man-dyld-2"></a><a href="#cite_note-macosx-man-dyld">[15]</a></sup>, после чего поищет библиотеку в стандартных каталогах.

Такая схема не позволяет собирать перемещаемые приложения, поэтому была введена специальная метка, которую можно прописывать в `id: @executable_path/`. Эта метка во время загрузки будет развернута в абсолютный путь до исполняемого файла. 

Далее, можно поменять зависимости у готового исполняемого файла:

{% highlight bash %}
$ install_name_tool -change /usr/lib/libstdc++.dylib @executable_path/libstdc++.dylib main.exe
{% endhighlight %}

Теперь загрузчик сначала будет искать эту библиотеку в той же папке, где и main.exe. Чтобы не менять в готовом исполняемом файле, надо во время компоновки подсунуть библиотеку libstdc++.dylib, у которой `id = @executable_path/libstdc++.dylib`.

Далее, возникает одна проблема, а точнее две. Пусть есть такая иерархия:

- main.bin
- tools/
    - auxiliary.bin
- library.dll

`main.bin` зависит от `library.dll`, но и `tools/auxiliary.bin` зависит от нее же. При этом id библиотеки = `@executable_path/library.dll`, и оба бинарника были просто с ней скомпонованы. Тогда при запуске `auxiliary.bin` загрузчик будет искать `/path/to/tools/library.dll` и естественно не найдет! Конечно можно ручками после компоновки подправить `tools/auxiliary.bin` или кинуть мягкую ссылку, но опять неудобства!

Еще лучше проблема проявляет себя, когда речь заходит о подключаемых модулях (plugins):

- main.bin
- plugin/
    - 1.plugin
    - helper.dylib

`1.plugin` имеет запись `@executable_path/helper.dylib`, но во время запуска она развернется в абсолютный путь до `main.bin`, а не `1.plugin`!

Для решения этой проблемы яблочники с версии Оси 10.4 ввели новый маркер: `@loader_path/`. Во время загрузки зависимости, этот маркер развернется в абсолютный путь к исполняемому файлу, который дергает зависимость.

Последняя сложность заключается в том, что надо две версии связываемых библиотек: одни будут установлены в систему, и иметь `id = /usr/lib/libfoo.dylib`, а другие использованы для сборки проектов, и их `id = @loader_path/libfoo.dylib`. Это легко решить с помощью install_name_tool, но утомительно; поэтому с версии 10.5 ввели метку `@rpath/`. Библиотека собирается с `id = @rpath/libfoo.dylib` и копируется куда угодно. Бинарник собирается со списком путей для поиска зависимостей, в котором разрешено использовать `@{executable, loader}_path/`:

{% highlight bash %}
$ gcc ... -Xlinker -rpath -Xlinker '@executable_path/libs' -Xlinker -rpath -Xlinker '/usr/lib' ...
{% endhighlight %}

Это аналогично RPATH/RUNPATH для ELF. При запуске бинарника строка `@rpath/libfoo.dylib` будет развёрнута в `@executable_path/libs/libfoo.dylib`, которая уже развернется в абсолютный путь. Либо в `/usr/lib/libfoo.dylib`.

Просмотреть зашитые в бинарник rpath'ы можно так:<pre><code class="bash">

{% highlight bash %}
$ otool -l main.bin | grep -A 2 -i lc_rpath
{% endhighlight %}

Удалить, изменить или добавить rpath'ы можно с помощью install_name_tool.

Проверяем на примере:

{% highlight bash %}
$ ./macosx_make_good.sh
Building version-0.2
Building version-0.3
Building bar
Building fooapp
$ ./result/main.exe
Hello World!
bar library uses libversion 0.2.0, number = 2
version::get_version result = 0
But I uses liversion 0.3.0
number = 3
{% endhighlight %}

На iOS всё так же.

Как видно из примера, Mac OS X в плане динамических библиотек лучше Linux & Co.

## И наконец, Windows!   {#windows}

Тут тоже всё хорошо <sup><a name="cite_ref-linkers-loaders-1"></a><a href="#cite_note-linkers-loaders">[6; Dynamic Linking and Loading, Comparison of dynamic linking approaches]</a></sup>. Надо только добавить версию в имя файла и… симлинков нет! То есть они есть, но на них многие жалуются и работают они только на NTFS <s>(Windows XP точно можно установить на FAT раздел)</s>. Следовательно, обратная совместимость может стоить приличного места на диске… Ну и ладно. )

Чтобы собрать пример на Windows потребуется запустить консоль Visual Studio, в которой уже будет настроено окружение. Далее сборка и запуск:

    > .\windows_make_good.bat
    // ...
    >.\result\main.exe
    Hello World!
    bar library uses libversion 0.2.0, number = 2
    version::get_version result = 0
    But I uses liversion 0.3.0
    number = 3

Библиотеки ищутся только так <sup><a name="cite_ref-windows-search-order"></a><a href="#cite_note-windows-search-order">[17]</a></sup>. Одним из возможных способов смены алгоритма поиска зависимостей является использование файла настроек приложения (application configuration file) и свойства privatePath у probing <sup><a name="cite_ref-windows-change-order"></a><a href="#cite_note-windows-change-order">[18]</a></sup>. Однако данный способ применим только начиная с Windows 7/Server 2008 R2.

А ещё есть WinSxS и так называемые сборки (assemblies) <sup><a name="cite_ref-winsxs-wiki"></a><a href="#cite_note-winsxs-wiki">[19]</a></sup>. Это &ndash; тема отдельной статьи. Однако пока писалась эта статья, снизошло озарение и понимание, что эти самые сборки нужны лишь для того (по крайней мере, Сишникам и Си++никам) чтобы все приложения компоновались, скажем, с comdlg32.dll, но все использовали разные версии.

## C++   {#cpp_lang}

До сих пор сказанное было справедливо для С. В случае с С++ есть особенности.

Есть несколько вариантов:

1. version экспортирует только шаблоны, т.е. никаких сгенерированных классов наружу не торчит;
2. version экспортирует шаблоны, а также некоторые классы, сгенерированные с помощью этих шаблонов.

Сначала второй случай. Проблема здесь заключается в том, что когда `liba` включит version-1.0.h

{% highlight cpp %}
// ...
typedef version::list<version::window> windows_list;
// ...
{% endhighlight %}

к себе в исходный текст, то дальнейшее развитие событий зависит от того, как оформлен шаблон `version::list`. Если он – header-only, то всё плохо:

1. в самой version.dll будет присутствовать сгенерированный класс `version::list<version::window>`;
2. когда будет компилироваться сама библиотека `а`, то в той единице трансляции, в которую был включен `version.h`, также будет сгенерирован класс `version::list<version::window>`. Он будет скомпилирован, в лучшем случае бинарно совместим с классом в `version.dll`, и все ссылки на него будут не undefined (что заставило бы загрузчик искать эти символы во время загрузки), а определены и указывать на сущности внутри `а` (в терминах nm – T либо t).

Даже если класс `version::list<version::window>` и был версионирован в `version.dll`, он окажется в непонятном состоянии в `liba`. (Тут, кстати, стоит отметить, что именно по этой причине некоторые люди используют AddRef/Release семантику для экспортируемых классов, а также, не экспортируют stl-классы: нагенерированные классы внутри самой библиотеки и ее пользователи отличаются бинарно даже в случае Debug/Release).

Чтобы решить эту проблему для C++03 и выше, шаблон надо разделить на объявление и определение: `version_list.h`, `version_list_impl.h`. Внутри самой библиотеки в файле реализации `version_windows_list.cpp` будет примерно так:

{% highlight cpp %}
#include "version_windows_list.hpp"

// подключаем тело шаблона
#include "version_list_impl.h"

// явно инстанцируем шаблон
template class version::list<version::window>;
{% endhighlight %}

Теперь, при сборке самой `version.dll` будет использован `version.script` для символов `vesion::list<version::window>` внутри самой библиотеки. `liba` и другие при компиляции не получат тело шаблона `version::list`, что заставит компоновщик требовать его при связывании. Таким образом, все получат в зависимостях версионированный шаблонный класс. Техника разделения шаблона описана [тут](http://www.parashift.com/c++-faq/templates-defn-vs-decl.html).

Начиная с Си++11 можно заставить компилятор не инстанцировать шаблон даже для хедер-онли шаблонов с помощью extern. Например, для std::unique_ptr:

{% highlight cpp %}
// MyTypePtr.hpp
#include <memory>

namespace mylib
{

class MyType;

} // namespace mylib

extern template class std::unique_ptr<mylib::MyType>;

namespace mylib
{

using MyTypePtr = std::unique_ptr<mylib::MyType>;

} // namespace mylib


// MyTypePtr.cpp

#include "MyTypePtr.hpp"

#include "MyType.hpp"

template class std::unique_ptr<mylib::MyType>;
{% endhighlight %}

Такая техника позволит экспортировать даже классы, сгенеренные из хедер-онли шаблонов, и применить для этих классов version.script. Стоит отметить, что extern уже давно использовался как расширение Си++ в msvc: <http://support.microsoft.com/kb/168958/en-us>.

К сожалению, с помощью extern не получится экспортировать контейнер перемещаемых объектов: <http://stackoverflow.com/questions/22381685/extern-template-class-stdcontainer-of-movable-objects>.

Теперь вернемся к первому случаю, когда экспортируются только шаблоны. Как видим, эти шаблоны должны быть грамотно оформлены. С точки зрения version, это проблема других, как они используют эти шаблоны. Если представляют их наружу – это первый случай, только с позиции уже другой библиотеки. Если используют для внутренних нужд – то с помощью того же version.script они должны быть помечены как local.

Также на счет С++ ABI.

В MacOS и в Linux'е, [c++filt](https://developer.apple.com/library/mac/documentation/Darwin/Reference/ManPages/man1/c++filt.1.html) входит в состав системы.
Также есть стандартный [ABI](http://mentorembedded.github.io/cxx-abi/). Есть, конечно, проблемы связанные с тем, что программисты соответствующие инструменты неправильно используют, ну так от этого и COM не защищает.

## Заключение    {#summary}

Все основные платформы позволяют относительно просто создавать приложения, которые могут быть установлены обычным копированием. Однако проблемы dependency hell, обратной совместимости и затирания символов разработчики должны решать самостоятельно.

Основным решением является выбор правильного версионирования и контроля за ним.

В то время, как <s>Curiosity бороздит марсианские просторы</s>автор пытался здесь поведать о том, как избежать затирания символов, на хабре уже давно есть статьи, где рассказано, как специально добиться обратного: <a href="http://habrahabr.ru/post/106107/">habrahabr.ru/post/106107/</a>, <a href="http://habrahabr.ru/post/115558/">habrahabr.ru/post/115558/</a>.

## Ссылки    {#refs}

1. <a name="cite_note-qtmultimedia501"></a><a href="#cite_ref-qtmultimedia501">^ </a>&nbsp;<a href="https://qt.gitorious.org/qt/qtmultimedia/source/b088962950dbc4c6f0219de30b0d9a8cf47a3376:dist/changes-5.0.1">QtMultimedia changes-5.0.1</a>
2. <a name="cite_note-soname-wiki"></a><a href="#cite_ref-soname-wiki">^ </a>&nbsp;<a href="http://en.wikipedia.org/wiki/Soname">http://en.wikipedia.org/wiki/Soname</a>.
3. <a name="cite_note-soname-so-howto"></a><a href="#cite_ref-soname-so-howto">^ </a>&nbsp;Program Library HOWTO, <a href="http://tldp.org/HOWTO/Program-Library-HOWTO/shared-libraries.html">3.1.1. Shared Library Names</a>.
4. <a name="cite_note-man-ld-linux_so"></a><a href="#cite_ref-man-ld-linux_so">^ </a>&nbsp;man ld-linux.so.
5. <a name="cite_note-rpath-wiki"></a><a href="#cite_ref-rpath-wiki">^ </a>&nbsp;<a href="http://en.wikipedia.org/wiki/Rpath">http://en.wikipedia.org/wiki/Rpath</a>.
6. <a name="cite_note-linkers-loaders"></a>^ <a href="#cite_ref-linkers-loaders">1</a> <a href="#cite_ref-linkers-loaders-1">2</a>&nbsp;Linkers and Loaders by John R. Levine, <a href="http://www.iecc.com/linker/">http://www.iecc.com/linker/</a>.
7. <a name="cite_note-dsohowto"></a>^ <a href="#cite_ref-dsohowto">1 </a><a href="#cite_ref-dsohowto-1">2 </a>&nbsp;How To Write Shared Libraries by Ulrich Drepper, <a href="http://www.akkadia.org/drepper/dsohowto.pdf">http://www.akkadia.org/drepper/dsohowto.pdf</a>(pdf).
8. <a name="cite_note-direct-binding"></a><a href="#cite_ref-direct-binding">^ </a>&nbsp;<a href="http://en.wikipedia.org/wiki/Direct_binding">http://en.wikipedia.org/wiki/Direct_binding</a>.
9. <a name="cite_note-gentoo-direct-binding"></a><a href="#cite_ref-gentoo-direct-binding">^ </a>&nbsp;<a href="https://bugs.gentoo.org/show_bug.cgi?id=114008">https://bugs.gentoo.org/show_bug.cgi?id=114008</a>.
10. <a name="cite_note-solaris-direct-binding"></a><a href="#cite_ref-solaris-direct-binding">^ </a>&nbsp;<a href="https://blogs.oracle.com/msw/date/20050614">https://blogs.oracle.com/msw/date/20050614</a>.
11. <a name="cite_note-solaris-no-dll-hell"></a><a href="#cite_ref-solaris-no-dll-hell">^ </a>&nbsp;<a href="http://cryptonector.com/2012/02/dll-hell-on-linux-but-not-solaris/">http://cryptonector.com/2012/02/dll-hell-on-linux-but-not-solaris/</a>.
12. <a name="cite_note-ld-version-0"></a><a href="#cite_ref-ld-version-0">^ </a>&nbsp;<a href="https://sourceware.org/binutils/docs/ld/VERSION.html">https://sourceware.org/binutils/docs/ld/VERSION.html</a>.
13. <a name="cite_note-ld-version-1"></a><a href="#cite_ref-ld-version-1">^ </a>&nbsp;<a href="http://www.tux.org/pub/tux/eric/elf/docs/GNUvers.txt">http://www.tux.org/pub/tux/eric/elf/docs/GNUvers.txt</a>.
14. <a name="cite_note-macosx-man-ld"></a><a href="#cite_ref-macosx-man-ld">^ </a>&nbsp;man ld.
15. <a name="cite_note-macosx-man-dyld"></a>^ <a href="#cite_ref-macosx-man-dyld-0">1</a> <a href="#cite_ref-macosx-man-dyld-1">2</a> <a href="#cite_ref-macosx-man-dyld-2">3</a>&nbsp;man dyld.
16. <a name="cite_note-macho-layout"></a><a href="#cite_ref-macho-layout">^ </a>&nbsp;<a href="http://en.wikipedia.org/wiki/Mach-O#Mach-O_file_layout">http://en.wikipedia.org/wiki/Mach-O#Mach-O_file_layout</a>.
17. <a name="cite_note-windows-search-order"></a><a href="#cite_ref-windows-search-order">^ </a>&nbsp;MSDN, Dynamic-Link Library Search Order, <a href="http://msdn.microsoft.com/en-us/library/windows/desktop/ms682586%28v=vs.85%29.aspx">http://msdn.microsoft.com/en-us/library/windows/desktop/ms682586%28v=vs.85%29.aspx</a>.
18. <a name="cite_note-windows-change-order"></a><a href="#cite_ref-windows-change-order">^ </a>&nbsp;<a href="http://stackoverflow.com/a/10390305/1758733">http://stackoverflow.com/a/10390305/1758733</a>.
19. <a name="cite_note-winsxs-wiki"></a><a href="#cite_ref-winsxs-wiki">^ </a>&nbsp;<a href="http://en.wikipedia.org/wiki/WinSXS">http://en.wikipedia.org/wiki/WinSXS</a>.
20. <a name="cite_note-oracle-linker-libraries"></a>&nbsp;Oracle, Linker and libraries Guide, <a href="http://docs.oracle.com/cd/E19683-01/817-1983/index.html">http://docs.oracle.com/cd/E19683-01/817-1983/index.html</a>.
21. [Руководство новичка по эксплуатации компоновщика](http://habrahabr.ru/post/150327/).
22. <a href="http://semver.org">semver.org</a>
