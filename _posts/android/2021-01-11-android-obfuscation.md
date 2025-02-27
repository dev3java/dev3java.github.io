---
author: melissa
categories:
- Android
- Theory
date: "2021-01-11T01:14:00Z"
tags:
- android
- theory
- obfuscation
- proguard
title: Обфускация кода
---

Защита кода приложения это то, о чем не всегда задумывается разработчик. Но  ведь обфускация при сборке – это не только защита. Это также эффективный способ уменьшить размер конечного APK, а иногда и оптимизировать исполнение программы. В этом нам поможет **Proguard**.

С учетом того, на сколько сейчас просто настроить Proguard для разработки под Android, не использовать эту возможность крайне не логично.

***

## Что делает Proguard?

**Proguard** – наверное самый часто используемый инструмент для обфускации кода при разработке под Андроид. Он бесплатный, с открытым исходным кодом. Есть еще **DexGuard** для более продвинутой защиты вашего приложения.

### Удаление неиспользуемого кода

Неиспользуемый код – лишние байты информации. Обфускация и Proguard удаляет его, экономя на финальном размере Apk.

Надо быть осторожным, код может использовать через рефлексию, в таком случае обфусцированное приложение сломается у пользователя на устройстве.

### Оптимизация байт кода

Proguard умеет работать с исходниками на уровне байт-кода, применяя оптимизации, например такие как:
- Замена умножение и деление на степень двойки на сдвиг,
- Inline интерфейсов с единственной реализацией,
- Подмена по возможности enum на численные константы,
- и [другие полезные вещи][proguard-optimizations].

Надо понимать, что Proguard работает на уровне Java байт кода, в то время как Андроид приложения исполняются на Dalvik Virtual Machine (DVM) используя Dalvik байт код. Оптимизации одного кода могут плохо отразиться на втором. А это значит надо внимательно относится к данному процессу.

***

## Как настроить Proguard для Android?

Максимально подробную инструкцию от первоисточника можно найти в [официальной документации Андроида][android-obfuscate-doc], пройдемся по порядку:

### Изменить build.gradle

```
android {
    ...
    buildTypes {
        release {
            shrinkResources true
            minifyEnabled true
            proguardFiles getDefaultProguardFile('proguard-android.txt'),
                    'proguard-rules.pro'
        }
    }
}
```

- `getDefaultProguardFile('proguard-android.txt')` – базовый конфиг для Андроид приложения из Android SDK, по умолчанию в нем отключена оптимизация, т.к. Dex может ее не пережить, попробовать ее включить можно, использовав вместо этого `getDefaultProguardFile('proguard-android-optimize.txt')`
- `proguard-rules.pro` – конфигурация непосредственно вашего модуля

Не следует включать обфускацию для Debug сборок, потому как этот процесс занимает некоторое время.

### Добавить config для используемых библиотек

Редко когда приложение не использует сторонние библиотеки. А внешние модули часто требуют дополнительных настроек для Proguard. Но! Существует [проект на GitHub][android-proguard-snippets] – сборник сниппетов для популярных библиотек, в котором уже есть большинство нужных конфигов.

Достаточно положить все файлы настроек в папку `proguard`  вашего модуля (по умолчанию `app`) и добавить в `build.gradle` строку, чтобы блок, отвечающий за `proguard` выглядел следующим способом:

```
minifyEnabled true
shrinkResources true
proguardFiles getDefaultProguardFile('proguard-android.txt')
proguardFiles(file('./proguard').listFiles())
```

`proguardFiles(file('./proguard').listFiles())` возьмет все файлы конфигов из папки `proguard` и применит их при обфускации вашего приложения.

### Проверить, что ваше приложение работает

Нужно проверять не Debug сборку без обфускации, а именно “оптимизированное” приложение. Лично я часто натыкался на то, что у меня все работает, а у тестировщиков сломано. Далеко не сразу соображаешь, что дело именно в ProGuard.

### Проверить сериализацию данных после обфускации

Большинство приложений сегодня работает с бэкендом. Запрос и ответ зачастую описан примерно таким классом:

```
public final class BookResponse {
    private String author;
    private String title;
}
```

И все корректно работает. Мы получаем данные. Все хорошо.

Приложение уходит тестировщику, у которого опять все сломано. Дело в том, что при обфускации меняются в том числе имена приватных методов, полей и другой “служебной” информации. Вследствие чего при автоматическом парсинге JSON сериализатор будет искать поле не `author` , а не пойми какое.

Решений по сути есть 2
- Запретить обфускацию классов, участвующих в сетевом обмене на уровне конфигурации модуля.
- Добавить имя для сериализации.

Я выбираю второе по нескольким причинам:
- Имя параметра не зависит от имени параметра на сервере.
- Руководствуясь [AOSP Java Code Style for Contributors][code-style] все не публичные члены класса у меня имеют префикс m.
- Смена имени на бэкенде при рефакторинге приведет к изменению ровно одного значения.

В итоге данный класс будет выглядеть следующий образом:

```
public final class BookResponse {
    @SerializedName("author")
    private String mAuthor;
    @SerializedName("title")
    private String mTitle;
}
```

### Все, обфускация готова!

Описанного выше достаточно, чтобы настроить ProGuard для защиты вашего проекта. Как видите, ничего сложного нет, обычно 5-10 минут работы и ваше приложение как-никак, но защищено от статического анализа кода. А защитить приложение от несанкционированного доступа поможет экран пин кода на старте.

***

## DexGuard

Если вам нужна только базовая обфускация и оптимизация, то ProGuard – то что вам нужно. Ваше приложение или библиотека требуют более серьезной защиты? Тогда следует посмотреть в сторону коммерческого DexGuard. Основные отличия между двумя инструментами:

ProGuard  |  DexGuard
--|--
Оптимизация Java байт кода  |  Комплексная защита Андроид приложения, включающая в себя тюнингованные версии часто используемых библиотек (Google play service, Dagger, Realm, SQLCipher и др.)
Базовая защита от [статического анализа кода][static-program-analysis]  |  Защита как от статического так и от [динамического анализа приложения][dynamic-program-analysis]
Фокус на байт коде приложения  |  Оптимизация, обфускация и шифрование файлов манифеста, нативных библиотек, файлов ресурсов и тд.
Усложнение реверс инжиниринга путем переименования классов методов и тд.  |  Помимо базовой, обфускация арифметический и логических операция, контроль порядка выполнения методов, шифрования строк и классов, добавление рефлексии для доступа к критическим данным
Инструмент с открытым исходным кодом  |  Коммерческий инструмент, требующий лицензии

***

## Ссылки

Оригинал статьи - [Обфускация кода под Android][dimlix].



[proguard-optimizations]: https://www.guardsquare.com/en/products/proguard/manual/usage/optimizations "guardsquare.com"
[android-obfuscate-doc]: https://developer.android.com/studio/build/shrink-code "developer.android.com"
[android-proguard-snippets]: https://github.com/krschultz/android-proguard-snippets "github.com"
[code-style]: https://source.android.com/setup/contribute/code-style#follow-field-naming-conventions "source.android.com"
[static-program-analysis]: https://ru.wikipedia.org/wiki/%D0%A1%D1%82%D0%B0%D1%82%D0%B8%D1%87%D0%B5%D1%81%D0%BA%D0%B8%D0%B9_%D0%B0%D0%BD%D0%B0%D0%BB%D0%B8%D0%B7_%D0%BA%D0%BE%D0%B4%D0%B0 "ru.wikipedia.org"
[dynamic-program-analysis]: https://ru.wikipedia.org/wiki/%D0%94%D0%B8%D0%BD%D0%B0%D0%BC%D0%B8%D1%87%D0%B5%D1%81%D0%BA%D0%B8%D0%B9_%D0%B0%D0%BD%D0%B0%D0%BB%D0%B8%D0%B7_%D0%BA%D0%BE%D0%B4%D0%B0 "ru.wikipedia.org"
[dimlix]: https://dimlix.com/obfuscation-android-proguard/ "dimlix.com"
