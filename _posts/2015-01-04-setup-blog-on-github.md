---
layout: post
title: Как поднять блог на Github Pages
permalink: setup-blog-on-github/
tags: github github-pages jekyll
---

Первый пост блога, инструкция.

---

[comment]: <> (This is a comment, it will not be included)
[comment]: <> (in  the output file unless you use it in)
[comment]: <> (a reference style link.)
[comment]:<--# Как поднять сайт на Github Pages. блог распрацоўніка-->

## Содержание

- [Лирика](#lirics)
- [Установка и настройка окружения](#setup)
- [Кастомизация](#customizations)
- [Итоги](#summaries)
- [Ссылки](#refs)

## Лирика   {#lirics}

Давно зрела идея завести личный сайт для того, что бы туда скидывать разные решения, которые нашел в процессе программирования и хотелось бы запомнить. Да и просто иметь место, где мыслью по древу иногда можно растекаться.

Останавливало два фактора: лень и незнание веб-технологий(и отсутсвие большого желания изучать их). Использовать же современные готовые движки типа [Wordpress](https://wordpress.org/), [Blogger](https://www.blogger.com/) или [LiveJournal](https://www.livejournal.com) и иже с ними не хотелось из-за их перегруженности, "вебдванольности", не подконтрольности мне. Да и большинство функционала, который предлагают современные блоги/дневники, мне не нужно.

И вот тут в очередной раз гуляя по [Github](https://github.com) я наталкиваюсь на [Github Pages](https://pages.github.com). Полчаса чтения и я понимаю &mdash; это то что нужно. Суть в чем? Сервис предлагает пользователям Github бесплатный хостинг статических html-страниц для специально созданного репозитория. Простые страницы все же не блог — так какой же от сервиса толк? Там есть генератор удобный, который из пачки шаблонов и простого текста сгенерирует сайт пользователю на радость командой одной — git push — прелесть какая:smile:.

#### Если коротко, то вот что меня привлекло завести блог на Github Pages:

- отсутствие желания заниматься самому настройкой сервера, следить за обновлениями, безопасностью и все такое. На Github Pages это происходит без моего участия;
- набор статических html-страниц, а значит быстро будет отображаться в браузере;
- есть генератор этих самых статических страниц &mdash; [Jekyll](http://jekyllrb.com/), а значит не надо будет самому заниматься версткой постов;
- раз статические страницы, то не надо никаких баз данных, php и тому подобных вещей;
- посты пишутся в простом [markdown](https://en.wikipedia.org/wiki/Markdown)-формате, а значит подойдет тем кто не хочет вообще заморачиваться с html;
- git:smile:;  это значит многое: версионность, локальное хранение, безопасность(шифрованные данные же пересылаем :smile: ) и еще очень много хорошего;
- возможность привязать к своему домену блог;
- возможность кастомизации по самое не хочу ... для тех кто хочет:wink:.

## Установка и настройка окружения  {#setup}

Вместо чтения кучи документации по Jekyll, я нашел крутую репу [poole](https://github.com/poole/poole). Как пишет ее автор:

> "a clear and concise foundational setup for any Jekyll site. And it has a super minimal look... It does so by furnishing a full vanilla Jekyll install with example templates, pages, posts, and styles."

Т.е. минимально необходимый набор для запуска блога: форкнул, поменял настройки под себя и пользуешься на здоровье. Правило "Все уже написано до нас!" работает безотказно :) .

[demo.getpoole.com](http://demo.getpoole.com/) &mdash; рабочее демо блога:

![The demo pool website](https://f.cloud.github.com/assets/98681/1834359/71ae4048-73db-11e3-9a3c-df38eb170537.png)

#### Приступим к созданию своего блога:

- устанавливаем `Ruby`;
- устанавливаем `Ruby DevKit`;
- устанавливаем `Jekyll`. Я сначала установил генератор так:  `gem install jekyll`. И это правильно в общем случае. Но не в случае с Github. Сайт использует не последнюю версию. И возможна такая ситуация, когда вы случайно задействуете новую фичу, которая не поддерживается той версией Jekyll, которая установлена на Github. Узнать текущую версию можно [здесь](https://pages.github.com/versions/). Установить требуемую версию `Jekyll` можно так: `gem install jekyll -v x.y.z`.
Но гораздо проще воспользоваться готовым пакетом от самых ребят с Github: `gem install github-pages`, он разворачивает текущее окружение, используемое Github, у вас на компьютере;  
**Update**: с докером все намного проще:
    
>docker run -ti \-\-rm -v "$PWD":/usr/src/app -p "4000:4000" starefossen/github-pages
    
- клонируем репозиторий [poole](https://github.com/poole/poole);
- запускаем локально сайт, для этого в консоли переходим в папку с `poole` и вводим `jekyll serve`. После того как Jekyll запуститься, мы сможем узреть сайт по адресу <http://localhost:4000>. Когда вы запускаете Jekyll, он создает папку `_site`, в которую записывает сгенерированный блог. Каждый файл в репозитории будет скопирован внутрь папки, за исключением тех файлов/папок, которые начинаются с подчеркивания. Markdown-файлы будут автоматически конвертированы в соотв. страницы. В папке `_posts` должны находится посты в markdown-формате. При чем имя файла должно быть [вида](http://jekyllrb.com/docs/posts/#creating-post-files) `год-месяц-день-название_поста.md`. Иначе для него не сгенерируется соответствующая html-страница. `_config.yml` &mdash; конфигурационный файл Jekyll, меняем его под свои надобности, подробно почитать о настройке [здесь](http://jekyllrb.com/docs/configuration/). **Важное примечание:** по умолчанию Jekyll при любом изменении файлов перезапишет сайт, но если вы поменяли что-то в `_config.yml`, то надо перезапустить Jekyll;
- добавляем посты, меняем шаблоны и вообще делаем что желаем;
- создаем на Github новый репозиторий с именем вида `github_username.github.io`;
- заливаем/пушим в него наш блог;
- спустя небольшое время(до получаса) блог становиться доступен по адресу <http://github_username.github.io>
- если есть свой домен, то Github легко позволяет привязать его к блогу. [Руководство](https://help.github.com/articles/setting-up-a-custom-domain-with-github-pages/) от Github, [больше инфы](http://davidensinger.com/2013/03/setting-the-dns-for-github-pages-on-namecheap/) для тех у кого регистратор домена Namecheap.

## Кастомизация     {#customizations}

В принципе дальнейшие шаги уже не обязательны, если вы дошли сюда, то скорее всего у вас уже успешно работает блог. В этой секции я описываю изменения, которые я захотел сделать.

Дефолтная тема poole мне подошла практически идеально. Но захотелось изменить еще некоторые вещи.

Добавил три ссылки вверху:

- [О блоге](/about/)
- [Архив записей](/archive/) - динамически формирующийся список записей, для быстрой навигации;
- [Лента](/atom.xml) - для желающих подписаться на обновления блога.

Для этого я добавил в `_config.yml`:

{% highlight yaml %}
pages_list:
  О блоге: '/about'
  Архив записей: '/archive'
  Лента: '/atom.xml'
{% endhighlight %}

И модифицировал шаблон [_layouts/default.html](https://github.com/alexprivalov/alexprivalov.github.io/blob/master/_layouts/default.html):

{% highlight html %}
{% raw %}
<h3 class="masthead-title">
    <a href="/" title="Home">{{ site.title }}</a>

    {% for page in site.pages_list %}
      &nbsp;&nbsp;&nbsp;
      <small><a href="{{ page[1]  }}">{{ page[0] }}</a></small>
    {% endfor %}

    <p> <small>{{ site.tagline }}</small> </p>
</h3>
{% endraw %}
{% endhighlight %}

Еще оказалось что с нуля нет поддержки тегов, что есть не классно:open_mouth:. Для Jekyll есть отдельный плагин, который реализует теги, но в нашем случае это не подходит, так как тот Jekyll, который крутится на серверах Github, нельзя кастомизировать плагинами. Памятуя о правиле "Все написано до нас", я полез в Google и спустя 5-ть минут [решение](https://cdlaap.github.io/2014/06/29/add-tags-page-to-jekyll-blog.html) найдено. Расписывать много не хочется, решение полностью скопировано, для наглядности будет удобно посмотреть [diff](https://github.com/alexprivalov/alexprivalov.github.io/commit/89dae334c91831def6577744d1cb38599e9a174b). А именно файлы [poole.css](https://github.com/alexprivalov/alexprivalov.github.io/blob/master/public/css/poole.css), [index.html](https://github.com/alexprivalov/alexprivalov.github.io/blob/master/index.html), [post.html](https://github.com/alexprivalov/alexprivalov.github.io/blob/master/_layouts/post.html) и [tags.html](https://github.com/alexprivalov/alexprivalov.github.io/blob/master/tags.html).

Честно говоря я ни грамма не понял, что я за магию сделал, для того что бы теги заработали(что взять с С++ разработчика?:stuck_out_tongue_winking_eye:), но все получилось и работает так как мне нужно. Вот, что <s>крест животворящий</s> свободный софт делает!:smile:

## Итоги    {#summaries}

Я получил личный блог/сайт, не особо окунаясь в мир веба, могу писать разной степени полезности посты и продолжать программировать на своих любимых С/С++.

## Ссылки   {#refs}

- официальный сайт [Jekyll](http://jekyllrb.com/) с очень хорошей [документацией](http://jekyllrb.com/docs/home);
- документация [Github Pages](https://pages.github.com);
- [poole](https://github.com/poole/poole) репозиторий.
- [руководство](https://guides.github.com/features/mastering-markdown/) по markdown;
- [пост](http://joshualande.com/jekyll-github-pages-poole/) чувака, на который я ориентировался при настройке блога;
- [пост](http://sylvaindurand.org/making-jekyll-multilingual/) чувака для настройки мультилингвальности блога;
- [настройка](http://davidensinger.com/2013/03/setting-the-dns-for-github-pages-on-namecheap/) DNS для Namecheap домена для  блога на Github;
- [пост](https://cdlaap.github.io/2014/06/29/add-tags-page-to-jekyll-blog.html) чувака, который рассказывает как добавить поддержку тегов без плагина Jekyll;
- [репозиторий](https://github.com/alexprivalov/alexprivalov.github.io) моего блога.
