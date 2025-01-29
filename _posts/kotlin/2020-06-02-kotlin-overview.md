---
author: dev3java
categories:
- Kotlin
- Basics
date: "2020-06-02T01:40:00Z"
tags:
- kotlin
- theory
- basics
title: Kotlin. Общий обзор
---

Начало разработки Kotlin было анонсировано JetBrains в 2011 году.
Планировался он как альтернатива языкам Java и Scala, так как тоже выполняется
под управлением Java Virtual Machine. И спустя 6 лет компания Google
анонсировала начало официальной поддержки Kotlin, как языка для разработки под
операционную систему Android.

Почему именно Kotlin? Потому что Kotlin - это язык со свежим взглядом на
старые вещи. Он дает нам простые и удобные инструменты, которые позволяют
писать лаконичный код.

Какую бы статью про Kotlin я не читала, в них всегда присутствует сравнение
того, как один и тот же код выглядит на Java и Kotlin. После чего следует
вывод, что на Kotlin все кратко и красиво. По факту так и есть. На своем
небольшом опыте работы с обоими языками могу сказать, что как только я начала
использовать Kotlin в своих проектах, то у меня сразу отпало желание
возвращаться к Java. И это не потому что Java какой-то плохой язык. Просто
Kotlin удобнее и к этому быстро привыкаешь.

К тому же Android Studio полностью поддерживает Kotlin, позволяя создавать новые
проекты с файлами Kotlin, добавлять файлы Kotlin в существующий проект и даже
конвертировать Java в Kotlin. Абсолютно все инструменты, доступные в Android
Studio, можно использовать при разработке на Kotlin.

Так как Kotlin полностью совместим с Java, то при работе с Android API можно
заметить, что очень часто код выглядит почти также как и соответствующий код
на Java. С единственной поправкой - вызовы методов можно комбинировать с
особенностями языка Kotlin.

***

## Релизы

На данный момент последняя версия Kotlin вышла 16 ноября 2021 года - 1.6.0. [Подробнее][kotlin-1.6.0].

Версия 1.5.30, дата выхода - 24 августа 2021 года. [Подробнее][kotlin-1.5.30]

Версия 1.5.20, дата выхода - 24 июня 2021 года. [Подробнее][kotlin-1.5.20]

Версия 1.5.0, дата выхода - 5 мая 2021 года. [Подробнее][kotlin-1.5.0]

Версия 1.4.30, дата выхода - 3 февраля 2021 года. [Подробнее][kotlin-1.4.30]

Версия 1.4.20, дата выхода - 23 ноября 2020 года. [Подробнее][kotlin-1.4.20]

Версия 1.4.0, дата выхода - 17 августа 2020 года. [Подробнее][kotlin-1.4.0]

Версия 1.3.70, дата выхода - 3 марта 2020 года. [Подробнее][kotlin-1.3.70-blog].

***

## Статьи

В данном разделе будут ссылки на мои статьи о Kotlin.

[Основной синтаксис][dev3java-basic-syntax].  
[Null-безопасность. Операторы "?.", "!!.", "?:"][dev3java-null-safety].  
[Модификаторы доступа][dev3java-visibility-modifiers].  
[Модификатор const][dev3java-const-modifier].  
[Отложенная и ленивая инициализация свойств][dev3java-lateinit-and-lazy].  
[Лямбда-выражения и анонимные функции][dev3java-lambdas-expressions-and-anonymous-functions].  
[Функции области видимости (Scope Functions)][dev3java-scope-functions].  
[Перегрузка операторов][dev3java-operator-overloading].  
[Коллекции][dev3java-collections].


### Классы и объекты

[Ключевое слово open][dev3java-open-keyword].  
[Классы данных (Data classes)][dev3java-data-classes].  
[Вложенные и внутренние классы][dev3java-nested-and-inner-clesses].  
[Классы перечислений (enum)][dev3java-enum-classes].  
[Изолированные классы (sealed classes)][dev3java-sealed-classes].  
[Основной и вторичный конструкторы. Init блок][dev3java-constructors-and-init-block].  
[Абстрактные классы и интерфейсы][dev3java-abstract-classes-and-interfaces].  
[Ключевое слово object][dev3java-object-keyword].  


### Библиотеки

[Нюансы при использовании библиотеки Gson][dev3java-gson].

***

## Полезные ссылки

[Официальная документация][doc-kotlin-official-eng] - на английском языке.  
[Learn Kotlin by Example][doc-learn-by-example-eng] - изучай Kotlin на примерах. Понятно и коротко объясняются основные фичи Котлина с такими же доступными примерами, которые можно здесь же запустить и посмотреть результат выполнения.  
[Неофициальный сайт][doc-kotlin-ru] с частичным переводом документации на русский язык.  
[Kotlin blog][kotlin-blog-official-eng].  
[Kotlin Christmas][kotlin-christmas] - ресурс, где вы найдете множество интересных статей по Kotlin, библиотеках, фреймворках и лучших практик.  
[Kotlin Cheat Sheet][kotlin-cheat-sheet] - шпаргалка для новичков по основному синтаксису.  


<!-- Ссылки на сторонние ресурсы -->
[doc-kotlin-official-eng]: https://kotlinlang.org/docs/reference/ "kotlinlang.org"
[doc-learn-by-example-eng]: https://play.kotlinlang.org/byExample/overview "play.kotlinlang.org"
[doc-kotlin-ru]: https://kotlinlang.ru/ "kotlinlang.ru"
[kotlin-blog-official-eng]: https://blog.jetbrains.com/kotlin/ "blog.jetbrains.com"
[kotlin-christmas]: https://kotlin.christmas/2020 "kotlin.christmas"
[kotlin-cheat-sheet]: https://drive.google.com/file/d/1G-waZ_omN6nwA63O31odBX3nOB_CaPbO/view?usp=sharing

<!-- Ссылки на версии Kotlin -->
[kotlin-1.3.70-blog]: https://blog.jetbrains.com/kotlin/2020/03/kotlin-1-3-70-released/ "blog.jetbrains.com"
[kotlin-1.4.0]: https://kotlinlang.org/docs/whatsnew14.html "kotlinlang.org"
[kotlin-1.4.20]: https://kotlinlang.org/docs/whatsnew1420.html "kotlinlang.org"
[kotlin-1.4.30]: https://kotlinlang.org/docs/whatsnew1430.html "kotlinlang.org"
[kotlin-1.5.0]: https://kotlinlang.org/docs/whatsnew15.html "kotlinlang.org"
[kotlin-1.5.20]: https://kotlinlang.org/docs/whatsnew1520.html "kotlinlang.org"
[kotlin-1.5.30]: https://kotlinlang.org/docs/whatsnew1530.html "kotlinlang.org"
[kotlin-1.6.0]: https://kotlinlang.org/docs/whatsnew16.html "kotlinlang.org"

<!-- Внутренние ссылки -->
[dev3java-basic-syntax]: /posts/kotlin-basic-syntax/ "dev3java.github.io"
[dev3java-null-safety]: /posts/kotlin-null-safety/ "dev3java.github.io"
[dev3java-visibility-modifiers]: /posts/kotlin-visibility-modifiers/ "dev3java.github.io"
[dev3java-const-modifier]: /posts/kotlin-const-modifier/ "dev3java.github.io"
[dev3java-lateinit-and-lazy]: /posts/kotlin-lateinit-and-lazy/ "dev3java.github.io"
[dev3java-lambdas-expressions-and-anonymous-functions]: /posts/kotlin-lambdas-expressions-and-anonymous-functions/ "dev3java.github.io"
[dev3java-scope-functions]: /posts/kotlin-scope-functions/ "dev3java.github.io"
[dev3java-operator-overloading]: /posts/kotlin-operator-overloading/ "dev3java.github.io"
[dev3java-collections]: /posts/kotlin-collections/ "dev3java.github.io"

[dev3java-open-keyword]: /posts/kotlin-open-keyword/ "dev3java.github.io"
[dev3java-data-classes]: /posts/kotlin-data-classes/ "dev3java.github.io"
[dev3java-nested-and-inner-clesses]: /posts/kotlin-nested-and-inner-clesses/ "dev3java.github.io"
[dev3java-enum-classes]: /posts/kotlin-enum-classes/ "dev3java.github.io"
[dev3java-sealed-classes]: /posts/kotlin-sealed-classes/ "dev3java.github.io"
[dev3java-constructors-and-init-block]: /posts/kotlin-constructors-and-init-block/ "dev3java.github.io"
[dev3java-abstract-classes-and-interfaces]: /posts/kotlin-abstract-classes-and-interfaces/ "dev3java.github.io"
[dev3java-object-keyword]: /posts/kotlin-object-keyword/ "dev3java.github.io"

[dev3java-gson]: /posts/kotlin-gson/ "dev3java.github.io"
