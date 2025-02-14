---
title: Как использовать FlowLayout в Jetpack Compose
author: dev3java
date: 2025-02-11 20:20:00 +0300
categories: [Android, Compose]
tags: [android, compose]
pin: false
image:
  path: /assets/img/posts/android-compose/flow-layout/flowLayout-example-cover.png
---

`FlowRow` и `FlowColumn` очень просты в использовании — они являются мощным и гибким инструментом
компоновки для динамических или неизвестных размеров.

Если вы следите за последними обновлениями Jetpack Compose, вы, возможно, слышали о новых
композитах `FlowRow` и `FlowColumn`, недавно добавленных в Jetpack Compose 1.4.1. Конечно, они уже
некоторое время были опубликованы
в `Accompanist`[^footnote] (группа библиотек, призванных
дополнить Jetpack Compose функциями, часто необходимыми разработчикам, но пока недоступными в
основной версии). Но теперь они доступны в базовом фреймворке, и было бы здорово посмотреть, для
чего их можно использовать.

Самая важная идея в этой компоновке заключается в том, что мы можем выкладывать элементы в
контейнер, когда размер элементов или контейнера неизвестен или динамичен. Например, здесь у нас
есть чипсы (теги), которые мы размещаем в случайном порядке. Фишка `Plant-based food` не помещается
и переносится на следующую строку.

![example-1](/assets/img/posts/android-compose/flow-layout/flowLayout-example-1.png)

```kotlin
fun FlowRow{
  chipsInRow.forEach {
    Chip(text = it.text, pointColor = it.pointColor)
  }
}

```

`FlowColumn` может выглядеть следующим образом:

![example-2](/assets/img/posts/android-compose/flow-layout/flowLayout-example-2.png)

```kotlin
fun FlowColumn{
  cardsInColumn.forEach { cardData ->
    Card(cardData)
  }
}
```

Давайте посмотрим на API:

- `horizontalArrangement` помогает расположить элементы по горизонтали в определенном порядке.
- `verticalAlignment` помогает выровнять элементы, если они имеют разную высоту (хотя это не должно
  быть распространенным случаем).
- `maxItemsInEachRow` указывает максимальное количество элементов, которые мы можем разместить в
  строке.

Это очень удобно использовать в сочетании с модификаторами веса.

```kotlin
fun FlowRow(
  modifier: Modifier = Modifier,
  horizontalArrangement: Arrangement.Horizontal = Arrangement.Start,
  verticalAlignment: Alignment.Vertical = Alignment.Top,
  maxItemsInEachRow: Int = Int.MAX_VALUE,
  content: @Composable RowScope.() -> Unit
) {}
```

Для `FlowColumn` параметры работают так же, только для вертикали.

`VerticalAlignment` предлагает 3 различных варианта, которые очевидны:

![example-3](/assets/img/posts/android-compose/flow-layout/flowLayout-example-3.png)

`HorizontalArrangement`, с другой стороны, имеет больше возможностей работы с некоторыми неочевидными вариантами:

![example-4](/assets/img/posts/android-compose/flow-layout/flowLayout-example-4.png)

Различие между `SpaceEvenly` и `SpaceAround` может быть неочевидным. Основное различие заключается в
том, что `SpaceEvenly`, как следует из названия, обеспечивает равномерный интервал, включая начальные
и конечные поля. `SpaceAround` работает так же, но поля занимают в два раза меньше места, чем
расстояние между внутренними элементами. В качестве альтернативы можно использовать абсолютное
значение, чтобы сохранить постоянство интерфейса при выборе RTL-макета: `Arrangement.Absolute`

Если вы хотите добавить расстояние между элементами, вы можете использовать такую конструкцию:

`Arrangement.spacedBy(5.dp, Alignment.CenterHorizontally)`

`maxItemsInEachRow` можно использовать вместе с модификаторами веса, чтобы сделать что-то вроде этого:

![example-5](/assets/img/posts/android-compose/flow-layout/flowLayout-example-5.png)

Вес всех кнопок установлен на 1, кроме кнопки «ноль», для которой вес установлен на 2.

```kotlin
FlowRow(maxItemsInEachRow = 4) {
  buttons.forEach {
    FlowButton(
      modifier = Modifier
        .aspectRatio(1 * it.weight)
        .clip(CircleShape)
        .background(it.color)
        .weight(it.weight),
      text = it.text,
      textColor = it.textColor,
    )
  }
}
```

Как видите, в простых примерах композабл `FlowRow` и `FlowColumn` очень просты в использовании — они
являются мощным и гибким инструментом компоновки для динамических или неизвестных размеров. Это
делает их особенно полезными для динамических и отзывчивых интерфейсов, которые адаптируются к
различным размерам и ориентации экрана. Мы ожидаем, что приведенные выше примеры покроют потребности
большинства разработчиков, но мы все еще хотим создать более сложный пример с использованием этих
новых макетов и анимацией переходов, которые могут происходить при добавлении, удалении или
изменении положения элементов.

***

[Источник](https://exyte.com/blog/android-flow-layout)

***

[^footnote]: [Accompanist](https://github.com/google/accompanist) — это группа библиотек, целью которых является дополнение Jetpack Compose функциями, которые обычно требуются разработчикам, но пока недоступны. Сейчас в Accompanist, например, есть Insets, System UI Controller, Pager, Placeholder, Navigation Animation, Swipe to Refresh и т.п.

