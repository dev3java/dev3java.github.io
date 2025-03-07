---
author: melissa
categories:
- Android
- Theory
date: "2020-07-04T01:30:00Z"
tags:
- android
- theory
- activity
title: Activity (Активность, Операция)
---

**Activity** - это компонент приложения, который является одним из его фундаментальных строительных блоков. Его основное предназначение заключается в том, что оно служит точкой входа для взаимодействия приложения с пользователем, а также отвечает за то, как пользователь перемещается внутри приложения или между приложениями.

По сути активити - это окно, с которым пользователь взаимодействует. И в этом окне можно "нарисовать" какой-либо пользовательский интерфейс. При этом окно может быть как на весь экран, так и занимать только заданную его часть и даже размещаться поверх других окон. Но, как правило, каждая активити реализует только одно окно. Однако, у каждого приложения может быть несколько активити.

Одна из активити может быть отмечена как **основная** (или главная) и тогда она будет появляться первой при запуске приложения. А уже из нее можно запустить другие активити. Причем не только те, которые принадлежат текущему приложению, но и активити из других приложений. Может показаться, что все запускаемые активити являются частями одного приложения. На самом же деле они могут быть определены в разных приложениях и работать в разных процессах.

Мне встречалась интересная аналогия, что активити - это как страницы разных сайтов, открываемых в браузере по ссылке.

***

## Создание новой активити

Первым делом следует создать новый класс, который должен наследоваться от класса `Activity` и переопределять метод `onCreate()`.

```
class MainActivity : AppCompatActivity() {

  override fun onCreate(savedInstanceState: Bundle?) {
    super.onCreate(savedInstanceState)
    ...
  }
}
```

Таким образом мы получим пустое окно. Чтобы в этом окне хоть что-то отображалось нужно создать разметку - **layout**. Добавляется она в ресурсах (папка **res**) в подпапке **layout**.

```
// res/layout/activity_main.xml

<LinearLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical"
    tools:context=".MainActivity">
    <TextView
        android:id="@+id/tv_info"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:text="Hello World"/>
</LinearLayout>
```

Теперь эту разметку нужно "подключить" к активити в методе `onCreate()`.

```
class MainActivity : AppCompatActivity() {

  override fun onCreate(savedInstanceState: Bundle?) {
    super.onCreate(savedInstanceState)
    setContentView(R.layout.activity_main)  // подключаем разметку
  }
}
```

Ну и наконец, чтобы приложение могло использовать только что созданную активити её нужно объявить в [манифесте](https://dev3java.github.io/posts/manifest-file/) приложения при помощи элемента `<activity>`.

```
// src/main/AndroidManifest.xml

<?xml version="1.0" encoding="utf-8"?>
<manifest ... >
    <application ... >
        <activity android:name=".MainActivity" />
    </application>

</manifest>
```

Все вышеперечисленные шаги можно сократить до одного благодаря наличию в Android Studio стандартных шаблонов: правой кнопкой мыши вызываем контекстное меню и в нём выбираем **New -> Activity -> "выбрать нужный шаблон"**.

![new-activity-patterns](/assets/img/posts/android/activity/new-activity-patterns.jpg)

После этого класс, разметка, а также запись в манифесте будут добавлены автоматически.

***

## Настройка активити в манифесте

Рассмотрим более подробно, какую информацию нужно или можно добавлять для элемента `<activity>` в манифесте.

### Объявление активити

Как упоминалось выше, все активити приложения обязательно нужно регистрировать в манифесте. Для этого предназначен элемент `<activity>`. У этого элемента есть только один **обязательный** атрибут - `android:name`, который ссылается на имя класса активити.

При этом, если приложение уже опубликовано, то не следует менять имя класса, так как это может привести к негативным последствиям. Для часто используемого приложения пользователь, как правило, создает ярлык на главном экране устройства. Ярлык представляет из себя `Intent`, который, используя имя класса, указывает какой компонент должен быть запущен. Поэтому при смене имени класса сломаются все ярлыки, а пользователь будет недоволен.

Помимо обязательного атрибута существуют и другие, с помощью которых каждой активити можно задать уникальные заголовок, иконку, тему и многие другие характеристики. Подробнее с ними можно ознакомиться в моей статье про [манифест](https://dev3java.github.io/posts/manifest-file/#activity) или в официальной [документации](https://developer.android.com/guide/topics/manifest/activity-element "developer.android.com").

### Объявление intent-фильтров

Скорее всего данная тема будет не особо понятна новичкам, поэтому сначала рекомендую ознакомиться с тем, что такое [Intent](https://developer.android.com/guide/components/intents-filters "developer.android.com").

Intent-фильтры - это выражение в файле манифеста, которое указывает, какие объекты `Intent` может обработать текущее приложение. Т.е. они позволяют настроить, на что будет реагировать активити.

Если у приложения отсутствуют intent-фильтры, то запустить его можно будет только с помощью явного `Intent` (по имени класса).

Объявляются они при помощи атрибута [`<intent-filter>`](https://dev3java.github.io/posts/manifest-file/#intent-filter) внутри элемента `<activity>`. При этом `<intent-filter>` должен **обязательно** содержать атрибут [`<action>`](https://dev3java.github.io/posts/manifest-file/#action) и может содержать необязательные атрибуты [`<category>`](https://dev3java.github.io/posts/manifest-file/#category) и [`<data>`](https://dev3java.github.io/posts/manifest-file/#data). Все вместе эти элементы указывают на тип объекта `Intent`, на который текущая активити сможет реагировать.

Например, активити можно добавить intent-фильтр, который будет говорить, что она умеет отправлять данные.

```
<activity android:name=".MainActivity">
    <intent-filter>
        // указывает, что активити умеет отправлять данные
        <action android:name="android.intent.action.SEND" />

        // позволяет активити получать запросы на запуск
        <category android:name="android.intent.category.DEFAULT" />

        // тип данных, которые активити умеет отправлять
        <data android:mimeType="text/plain" />
    </intent-filter>
</activity>
```

Выше упоминалось, что активити можно отметить как **основную**. Это тоже осуществляется с помощью intent-фильтра.

```
<activity android:name=".MainActivity">
    <intent-filter>
        <action android:name="android.intent.action.MAIN" />
        <category android:name="android.intent.category.LAUNCHER" />
    </intent-filter>
</activity>
```

### Объявление разрешений

С помощью разрешений можно контролировать, какие приложения могут запускать активити. При этом одна активити не сможет запустить другую, если они не имеют одинаковых разрешений в манифесте.

Например, ваше приложение хочет использовать приложение **SocialApp** для публикации какой-либо информации в социальных сетях. Приложение **SocialApp** должно определить разрешение:

```
<manifest>
<activity android:name="...."
   android:permission=”com.google.socialapp.permission.SHARE_POST”

/>
```

И чтобы иметь возможность запускать **SocialApp** вы должны в своем приложении добавить идентичное разрешение:

```
<manifest>
   <uses-permission android:name="com.google.socialapp.permission.SHARE_POST" />
</manifest>
```

***

## Жизненный цикл

При использовании приложения мы постоянно перемещаемся от одного экрана к другому и обратно. Поэтому все активити, с которыми мы взаимодействуем постоянно меняют состояние своего жизненного цикла. А чтобы узнать о смене состояния существуют методы обратного вызова. Т.е. как только активити перешла в другое состояние, сразу же вызывается соответствующий метод обратного вызова. Таким образом можно отслеживать смену состояния и реагировать на него.

Для чего это делать? Чтобы избежать следующих ситуаций:
- Сбой приложения, когда пользователю поступает звонок или когда он переключается на другое приложение.
- Потребление ценных ресурсов, даже если пользователь активно не пользуется приложением в данный момент.
- Потеря прогресса пользователя, если он переключился на другое приложение или временно из него вышел.
- Сбой или потеря прогресса при смене ориентации экрана с портретной на альбомную и обратно.

Это вовсе не означает, что каждый раз нужно реализовывать все методы обратного вызова. Требуется лишь понять для чего каждый из них предназначен, чтобы использовать их в нужный момент.


### Методы обратного вызова

#### **onCreate()**

Вызывается при создании и перезапуске активити. После создания активити она переходит в состояние "Создана" (Created state), но существует в нём недолго. Как только метод `onCreate()` будет выполнен, активити переходит в статус "Запущена" (Started state).

Этому методу передается объект `Bundle`, который содержит в себе сохранённое состояние активити (если она перезапустилась), либо `null` (если активити ранее не существовала).

**Что в нём должно происходить:**  
Выполнение базовой логики запуска приложения, которая должна происходить только один раз за весь период действия. Например, создание пользовательского интерфейса, привязка данных, создание сервисов (service) и потоков.

**Следующий метод:**   
`onStart()`.


#### **onStart()**

Вызывается при переходе активити в состояние "Запущена" (Started state), т.е. сразу после выполнения метода `onCreate()`. В этом состоянии активити существует пока метод `onStart()` не будет выполнен. После чего переходит в состояние "Возобновлена" (Resumed state).

Также этот метод делает активити видимой для пользователя, но с ней еще нельзя взаимодействовать.

**Что в нём должно происходить:**  
Выполнение кода, который поддерживает пользовательский интерфейс

**Следующий метод:**  
`onResume()`


#### **onResume()**

Вызывается при переходе активити в состояние "Возобновлена" (Resumed state), т.е. сразу после выполнения метода `onStart()`.
В этом состоянии активити взаимодействует с пользователем. И продолжается это до тех пор пока пользователя что-то не отвлечёт, например, телефонный звонок, переход к другому приложению или отключение экрана устройства. В этом случае активити перейдёт в состояние "Приостановлена" (Paused state).

**Что в нём должно происходить:**  
Включение функциональности, которая должна работать, пока активити видна и находится на переднем плане.
Возобновить работу того, что было приостановлено в методе `onPause()`.

**Следующий метод:**  
`onPause()`


#### **onPause()**

Вызывается, когда активити теряет фокус и переходит в состояние "Приостановлена" (Paused state). Т.е. активити больше не находится на переднем плане, хотя может быть всё ещё видна пользователю.

Завершение работы метода `onPause()` не означает, что активити перейдёт в другое состояние. Она будет оставаться в состоянии "Приостановлена" до тех пор, пока либо не перейдёт обратно на передний план (вызовется метод `onResume()`), либо пока полностью не станет невидимой (вызовется метод `onStop()`).

**Что в нём должно происходить:**  
Приостановка или настройка операций, которые не должны продолжаться пока активити находится в состоянии "Приостановлена", но ожидается, что вы вскоре их возобновите.  
Код должен быть легковесным, так как нет гарантии, что он успеет выполнится.

**Следующий метод:**  
`onResume()` или `onStop()`.


#### **onStop()**

Вызывается, когда активити больше не видна пользователю и переходит в состояние "Остановлена". Это может произойти, если активити уничтожается, запускается новая активити или возобновляет работу существующая активити, закрывая собой текущую. Несмотря на это, активити остаётся в памяти, но она больше не привязана к диспетчеру окон.

Из состояния "Остановлена" активити либо возвращается к взаимодействию с пользователем (вызывается метод `onRestart()`), либо завершается (вызывается метод `onDestroy()`).

**Что в нём должно происходить:**  
Остановка функций, работа которых не нужна пока активити невидима для пользователя.
Например, остановка анимаций, сохранение информации в базе данных.

**Следующий метод:**  
`onRestart()` или `onDestroy()`.


#### **onRestart()**
Вызывается, когда активити в состоянии "Остановлена" повторно отображается пользователю.

**Что в нём должно происходить:**  
Выполнение действий, которые должны выполняться при повторном запуске активити в рамках одного жизненного цикла приложения.

**Следующий метод:**  
`onStart()`.


#### **onDestroy()**

Вызывается перед тем, как активити будет уничтожена и переходит в состояние "Уничтожена" (Destroyed state). Это происходит в следующих случаях: если пользователь её закрыл; был вызван метод `finish()`; произошла смена конфигурации активити (например, поворот экрана). При этом в первых двух случаях метод `onDestroy()` будет последним обратным вызовом в жизненном цикле активити. Если же `onDestroy()` был вызван из-за смены конфигурации, система немедленно создаст новый экземпляр активити и вызовет для него метод `onCreate()`.

**Что в нём должно происходить:**  
Освобождение всех оставшихся ресурсов, которые не были освобождены в методах `onPause()` и `onStop()`.  
Не использовать для какой-либо критически важной логики, так как данный метод может быть и не вызван.

**Следующий метод:**  
Последний метод жизненного цикла.


### Схема состояний активити

Исходя из вышеописанного можно сделать простенькую схему для лучшего понимания того, как активити переходит из одного состояния своего жизненного цикла в другое. Я делаю акцент именно на состояниях (а не на методах обратного вызова), потому что на мой взгляд это легче для восприятия (особенно новичкам).

![activity-state-scheme](/assets/img/posts/android/activity/activity-state-scheme.png)

***

## Полезные ссылки
[Introduction to Activities](https://developer.android.com/guide/components/activities/intro-activities#ctm "developer.android.com") - официальная документация.
