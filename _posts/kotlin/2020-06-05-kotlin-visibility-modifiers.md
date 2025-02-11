---
author: melissa
categories:
- Kotlin
- Basics
date: "2020-06-05T14:10:00Z"
tags:
- kotlin
- theory
- basics
- modifiers
title: Kotlin. Модификаторы доступа - private, protected, internal, public
---

Любой класс, а также его конструкторы, свойства и функции имеют **модификаторы
доступа** - это такие ключевые слова, с помощью которых можно задать область
действия данных. Или иначе - они позволяют регулировать уровень доступа к
различным частям кода.

В Kotlin есть четыре модификатора доступа:
- private
- protected
- internal
- public

Если модификатор явно не указан, то присваивается значение по умолчанию -
**public**.

***

## Модификатор private

**Private** - самый строгий модификатор доступа. При его использовании данные
будут доступны только в пределах конкретного класса или файла.

```
// Переменная видима внутри данного файла
private const val a = 20

// Класс доступен только внутри данного файла
private class Person() {

    // Переменную можно использовать только внутри класса Person
    private val b = a
}

// ERROR: переменную b нельзя использовать за пределами класса Person
private const val c = b
```

По сути главное предназначение данного модификатора - реализация инкапсуляции
в программе.

***

## Модификатор protected

Данные, отмеченные модификатором **protected** будут видны:
- внутри класса, в котором они объявлены.
- в дочерних классах.

При этом нельзя отметить модификатором **protected** данные _высокого уровня_.
К таким данным относятся классы, а также переменные или функции, объявленные
вне класса.

```
// ERROR: Нельзя использовать protected для переменных вне класса
protected const val a = 20

// ERROR: Нельзя использовать protected для класса
protected class Person() {

  // Переменная видима внутри класса Person
  protected val b = a
}
```

Если в дочернем классе мы переопределим метод с модификатором **protected**,
то он унаследует модификатор доступа от родителя и будет виден только внутри
дочернего класса. Несмотря на то, что модификатор не будет указан явно.

```
open class Person() {
    protected open fun getAge() = 20
}

private class Student : Person() {
    // модификатор явно не указан, но он такой же, как и в родительском классе
    override fun getAge() = 25
}
```

Помимо модификатора **protected** такому методу можно задать модификатор
**public**. При использовании остальных модификаторов Kotlin ругается.

***

## Модификатор internal

Как правило при разработке проекта мы делим его на независимые модули. Каждый
модуль состоит из файлов, компилируемых вместе. Так вот модификатор **internal**
позволяет сделать данные видимыми для всего модуля.

Данный модификатор можно применять ко всем типам данных. Однако он полезен
только в том случае, если в проекте есть более одного модуля. Иначе используется
модификатор **public**.

Например, в проекте есть два модуля - **Module1** и **Module2**. В первом
модуле есть класс `Person()`.

```
// Module1

// Переменная видима для всего Module1
internal const val a = 20

// Класс доступен для всего Module1
internal open class Person() {
    // Переменная видима для всего Module1
    internal val b = a
}
```

И еще в первом модуле есть такой файл:

```
// Module1

private const val c = a + b
```

Так как этот файл тоже находится в **Module1**, то мы можем получить доступ к
переменным `a` и `b`. Но если попытаться к ним обратиться из **Module2** -
получим ошибку.

```
// Module2

// ERROR: переменные a и b недоступны для данного модуля
private const val d = a + b
```

***

## Модификатор public

Если при объявлении каких-либо данных использовать модификатор **public**, то
они будут видны всем (даже в космосе). Еще **public** является модификатором по
умолчанию для тех данных, которым модификатор явно не был указан.

```
// Переменные доступны из любого места

public const val a = 20

public open class Person() {

    public val b = a
}

public class Student() {

    public val с = a + b
}
```

***

## Полезные ссылки

Официальная документация - [Visibility Modifiers](https://kotlinlang.org/docs/reference/visibility-modifiers.html "kotlinlang.org"). <br>
Документация в переводе на русский - [Модификаторы доступа](https://kotlinlang.ru/docs/reference/visibility-modifiers.html "kotlinlang.ru"). <br>
