---
title: Интегрируем Rust в Android-приложение (Финальная часть)
author: dev3java
date: 2025-02-17 17:00:00 +0300
categories: [Android, Rust]
tags: [android, compose, rust, kotlin, java]
pin: false
image:
  path: /assets/img/posts/android-rust/rust-android-scaled.webp
---

Это вторая и заключительная часть серии, в которой объясняется, 
как добавить код Rust в ваш Android-проект без необходимости изучать или 
использовать JNI или C в вашем проекте. В [первой части серии][integration-rust-into-android-app] мы создали простой 
проект Android Studio и простую библиотеку, написанную на Rust, для регистрации информации. 
Во второй и заключительной части мы рассмотрим практический пример применения 
функций Rust в нашей работе.

В этом пошаговом руководстве мы создадим простое приложение-калькулятор, которое вычисляет результат, 
когда пользователь нажимает кнопку "равно". Приложение будет принимать 2 входных сигнала, 
а затем выполнять 3 операции над ними: сложение, вычитание и умножение. 
Эти простые знания были бы полезны в реальных случаях, когда вам, возможно, 
придется выполнять трудоемкие или низкоуровневые операции на Android. 
Не мудрствуя лукаво, давайте создадим серверную часть нашего приложения 
(изначально вы могли бы создать U.I, но для этого пошагового руководства я считаю, 
что сначала удобнее написать серверную часть Rust).

## Проектирование Серверной части ⚙️

Как уже говорилось ранее, мы бы создали структуру, содержащую 2 значения, 
а также функции, которые возвращали бы соответствующие значения сложения, 
вычитания и умножения. Мы бы добавили эту структуру в `lib.rs`.

[//]: # (<script src="https://gist.github.com/Kofituo/8b1dd003c24465d41ad13a5a1e951c66.js"></script>)

```rust
pub struct Inputs {
    first: i64,
    second: i64,
}

impl Inputs {
    #[generate_interface(constructor)]
    pub fn new(first: i64, second: i64) -> Inputs {
        Self {
            first,
            second,
        }
    }
    #[generate_interface]
    pub fn addition(&self) -> i64 {
        self.first + self.second
    }
    #[generate_interface]
    pub fn subtraction(&self) -> i64 {
        self.first - self.second
    }
    #[generate_interface]
    pub fn multiplication(&self) -> i64 {
        self.first * self.second
    }
}
```

`Self` в аргументах метода - это синтаксический сахар для принимающего типа метода, то есть типа, 
в котором используется этот метод, и в данном случае UIit - это `Inputs`. 
Вы могли бы заменить `Self` на `Inputs`, и код работал бы нормально.

Помните, что `#[generate_interface]` происходит от rifgen, как описано в [первой части серии][integration-rust-into-android-app].

Теперь давайте реализуем U.I.

## Проектирование U.I. 📱

Как было описано ранее, U.I. будет иметь 2 текстовых поля для ввода и 3 текстовых поля для вывода. 
Это будет простой макет для иллюстрации:

[//]: # (<script src="https://gist.github.com/Kofituo/2f0cb830eb039744b3e67e61298f5c83.js"></script>)

```kotlin
package com.example.rustapplication

import android.os.Bundle
import androidx.activity.ComponentActivity
import androidx.activity.compose.setContent
import androidx.compose.foundation.layout.*
import androidx.compose.foundation.text.KeyboardOptions
import androidx.compose.material.*
import androidx.compose.runtime.*
import androidx.compose.ui.Alignment
import androidx.compose.ui.Modifier
import androidx.compose.ui.text.input.KeyboardType
import androidx.compose.ui.unit.dp
import com.example.rustapplication.lib.Inputs // note the imports
import com.example.rustapplication.lib.RustLog
import com.example.rustapplication.ui.theme.RustApplicationTheme

class MainActivity : ComponentActivity() {

    companion object {
        init {
            System.loadLibrary("rust_lib")
            RustLog.initialiseLogging()
        }
    }

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContent {
            RustApplicationTheme {
                Surface(
                    modifier = Modifier.fillMaxSize(),
                    color = MaterialTheme.colors.background
                ) {
                    Homepage()
                }
            }
        }
    }
}

@Composable
fun Homepage() {
    var firstInput by remember { mutableStateOf("") }
    var secondInput by remember { mutableStateOf("") }
    var addition by remember { mutableStateOf("Add") }
    var subtraction by remember { mutableStateOf("Subtract") }
    var multiplication by remember { mutableStateOf("Multiply") }
    var showError by remember { mutableStateOf(false) } // Indicate error if the wrong input is received

    Column {
        Column(horizontalAlignment = Alignment.CenterHorizontally) {
            Row {
                OutlinedTextField(
                    modifier = Modifier.weight(1f),
                    value = firstInput,
                    onValueChange = { firstInput = it },
                    label = { Text("Enter any number") },
                    keyboardOptions = KeyboardOptions.Default.copy(
                        keyboardType = KeyboardType.Number // accept only numbers
                    ),
                    isError = showError
                )
                OutlinedTextField(
                    modifier = Modifier.weight(1f),
                    value = secondInput,
                    onValueChange = { secondInput = it },
                    label = { Text("Enter any number") },
                    keyboardOptions = KeyboardOptions.Default.copy(
                        keyboardType = KeyboardType.Number // accept only numbers
                    ),
                    isError = showError
                )
            }
            Spacer(modifier = Modifier.size(14.dp))
            Button(onClick = {
                val first = firstInput.toLongOrNull()
                val second = secondInput.toLongOrNull()
                if (first == null || second == null) {
                    showError = true
                    return@Button
                }
                val inputs = Inputs(first, second)
                addition = "${inputs.addition()}"
                subtraction = "${inputs.subtraction()}"
                multiplication = "${inputs.multiplication()}"
                showError = false
            }) {
                Text(text = "=")
            }
            Spacer(modifier = Modifier.size(14.dp))
        }
        Column(horizontalAlignment = Alignment.CenterHorizontally) {
            Row {
                OutlinedTextField(
                    modifier = Modifier.weight(1f),
                    value = addition,
                    onValueChange = { },
                    label = { Text("Addition") },
                    readOnly = true
                )
                OutlinedTextField(
                    modifier = Modifier.weight(1f),
                    value = subtraction,
                    onValueChange = { },
                    label = { Text("Subtraction") },
                    readOnly = true
                )
            }
            OutlinedTextField(
                value = multiplication,
                onValueChange = { },
                label = { Text("Multiplication") },
                readOnly = true
            )
        }
    }
}
```
![github-gif][1-github-gif]{: .right}
И это все! Вот как должно выглядеть приложение.

Всякий раз, когда пользователь нажимает кнопку "равно", числа из текстовых полей передаются 
в структуру, написанную на `Rust`. Затем значения вычисляются, и результат отображается 
в текстовом поле. Не стесняйтесь изменять код и пробовать другие способы 
добавления разнообразия `Rust` в ваш Android-проект.

## Дальнейшее чтение 📖

Узнайте больше о зависимостях, используемых в этом пошаговом руководстве:

- [Dushistov/flapigen-rs][2-github-flapigen-rs]: Инструмент для подключения программ или библиотек, написанных на `Rust`, к другим языкам
- [Kofituo/rifken][3-github-rifgen]: Программа для перевода библиотек, написанных на `Rust`, в файлы интерфейса. Работает с `flapigen`.
- [sfackler/rust-jni-sys][4-github-sfackler]

[Репозиторий][5-repository]

<!-- Ссылки по тексту -->
[1-github-gif]: https://github.com/dev3java/RustApplication/raw/master/20220821_122458.gif "github.com"
[2-github-flapigen-rs]: https://github.com/Dushistov/flapigen-rs "github.io/flapigen-rs"
[3-github-rifgen]: https://github.com/Kofituo/rifgen "github.io/rifgen"
[4-github-sfackler]: https://github.com/sfackler/rust-jni-sys "github.io/sfackler"
[5-repository]: https://github.com/dev3java/RustApplication "github.com"

[integration-rust-into-android-app]: /posts/integration-rust-into-android-app "Первая часть"
