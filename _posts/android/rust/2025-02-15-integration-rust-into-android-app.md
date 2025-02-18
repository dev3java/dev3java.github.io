---
title: Интегрируем Rust в Android-приложение (Первая часть)
author: dev3java
date: 2025-02-15 20:40:00 +0300
categories: [Android, Rust]
tags: [android, jetpack compose, rust, kotlin, java]
pin: false
image:
  path: /assets/img/posts/android/rust/rust-android-scaled.webp
  lqip: data:image/webp;
---

Rust — это язык системного программирования общего назначения, который существует уже довольно давно. 
Его можно использовать для выполнения задач, которые реализуются сейчас на C и C++, но с гораздо большей безопасностью памяти. 
Это позволяет использовать Rust для написания программ или скриптов для многих операционных систем, включая Android. 
Вы можете задаться вопросом, как это возможно и есть ли простой способ сделать это. Вот об этом эта статья!

В настоящее время не так много информации о том, как писать код на Rust в Android-приложениях. 
Есть некоторая информация, предоставленная [Google][1-building-rust-modules], но она сложна для понимания новичком. 
Цель этого пошагового руководства — предоставить простое, но эффективное руководство по интеграции кода Rust в разработку для Android. 
Никаких предварительных знаний C или C++ или JNI не требуется.


## Настройка ⚙️

Мы будем использовать [Android Studio][2-android-studio]. Начнем с его настройки.

### Плагин Rust 🌱

Во-первых, нам нужен плагин Rust. Откройте Android Studio. Откроется диалоговое окно, подобное показанному ниже. 
Выберите вкладку `Плагины`, найдите `Rust`, а затем установите официальный плагин Rust от JetBrains. 
В качестве альтернативы, если у вас уже открыт проект, нажмите `Файл` в верхнем левом углу Android Studio и выберите `Настройки`. 
Нажмите `Плагины`.

>"Плагин Rust требует, чтобы Rust был установлен в вашей системе.
> Вы можете перейти по [этой ссылке][3-rust-tools-install], чтобы установить Rust в вашей системе."
{: .prompt-info }

![1-4.webp](/assets/img/posts/android/rust/1-4.webp)


### Создаем пустое приложение 📱

Теперь давайте создадим новое пустое приложение. Начните со стандартной настройки нового проекта.

![2-2.webp](/assets/img/posts/android/rust/2-2.webp)

Назовем наш проект Rust Application и нажмем «Готово».

Появится окно, похожее на показанное ниже.

![3-4.webp](/assets/img/posts/android/rust/3-4.webp)

### Включаем Cargo Project 💼

Давайте теперь добавим [проект Cargo][4-rust-cargo] (менеджер пакетов для Rust) следуя [этому руководству][5-rust-cargo-guide]. 
Чтобы внедрить его в проект Rust, просто щелкните Терминал в Android Studio и введите это:


`C:\Users\USER1\AndroidStudioProjects\RustApplication>cargo new rust_lib --lib`{: .filepath}.

Это создает новую библиотеку, чтобы ее можно было использовать из нашего приложения. 
Мы назвали библиотеку `rust_lib`. Обратите внимание, где создается папка.

Чтобы просмотреть папку в Android Studio, перейдите на панель Project и выберите Project  в раскрывающемся меню вместо Android.

![4-2.webp](/assets/img/posts/android/rust/4-2.webp)

Вы увидите новую папку с именем `rust_lib`. Эта папка по умолчанию содержит новый репозиторий `git`, 
файл `Cargo.toml` и папку `src`, содержащую `lib.rs`. Теперь давайте изменим [тип библиотеки][6-rust-cargo-reference], 
которую мы только что создали, используя файл `Cargo.toml`. 
Для разработки мобильных приложений библиотека должна быть динамической.

Добавьте следующее к содержимому файла `Cargo.toml`.

```toml
[lib]
name = "rust_lib"
crate-type = ["cdylib"]
```

[//]: # (Посмотреть суть [здесь][1-gist-cargo.toml].)

### Добавление зависимостей 🧶

Давайте добавим зависимости, которые создадут необходимые файлы, чтобы сделать связывание нашего Rust кода с Android гладким процессом.

[//]: # (<script src="https://gist.github.com/Kofituo/5ffbc5a67d89db3ec65c36c988a37cac.js"></script>)

```toml
[package]
name = "rust_lib"
version = "0.1.0"
edition = "2021"

# Смотрите больше ключей и их определений по адресу https://doc.rust-lang.org/cargo/reference/manifest.html
[lib]
name = "rust_lib"
crate-type = ["cdylib"]

[dependencies]
rifgen = "*"
jni-sys = "*"
#логирование
log = "*"
log-panics="*"
android_logger = "*"

[build-dependencies]
flapigen = { git = "https://github.com/Dushistov/flapigen-rs", rev = "d79a34f22e73d90fe9f2423148a7421d39b8ed69" }
rifgen = "*"
```

Используйте `*`, чтобы получить последнюю версию зависимости.

[Flapigen][1-crates-flapigen] — это основная зависимость сборки для генерации соответствующего кода из нашего Rust кода и использования в Android-приложении. 
`Flapigen` работает с файлом интерфейса, но необходимость изменять файл интерфейса каждый раз при изменении кода может стать утомительной. 
Вот тут-то и появляется [rifgen][2-crates-rifgen]. Это упрощает создание файла интерфейса.

Дополнительные зависимости предназначены для ведения логов.

### Создание файла сборки 📄

Как упоминалось ранее, `flapigen` и `rifgen` являются зависимостями сборки и взаимодействуют с файлом `build.rs`. 
Щелкните правой кнопкой мыши папку `rust_lib` и выберите New > Rust File.

![5-2.webp](/assets/img/posts/android/rust/5-2.webp)

Назовите файл `build.rs`.

Следуя руководству, предоставленному [flapigen][4-github-flapigen-rs] и [rifgen][3-docs-rifgen], 
наш файл `build.rs` должен выглядеть примерно так:

[//]: # (<script src="https://gist.github.com/Kofituo/7ee81f890123f9f6348102906e05fce2.js"></script>)

```rust
use std::{env, fs};
use std::path::Path;
use flapigen::{JavaConfig, LanguageConfig};
use rifgen::{Generator, Language, TypeCases};

fn main() {
    let out_dir = env::var("OUT_DIR").unwrap();
    let in_src = "src/java_glue.rs.in";
    let out_src = Path::new(&out_dir).join("java_glue.rs");
    Generator::new(TypeCases::CamelCase, Language::Java, "src")
        .generate_interface(in_src);
    //delete the lib folder then create it again to prevent obsolete files from staying
    let java_folder = Path::new("../app/src/main/java/com/example/rustapplication/lib");
    if java_folder.exists() {
        fs::remove_dir_all(java_folder);
    }
    fs::create_dir(java_folder).unwrap();
    let swig_gen = flapigen::Generator::new(LanguageConfig::JavaConfig(
        JavaConfig::new(java_folder.into(), "com.example.rustapplication.lib".into())
            .use_null_annotation_from_package("androidx.annotation".into()),
    )).rustfmt_bindings(true);
    swig_gen.expand("android bindings", &in_src, &out_src);
    println!("cargo:rerun-if-changed=src");
}
```

Вам может быть интересно, что происходит в файле `build.rs`. `Flapigen` преобразует то, 
что находится в файле интерфейса, в Rust файл `java_glue`. 
Поэтому мы сначала указываем исходный файл для файла интерфейса (`in_src`), 
а затем выходной файл (`java_glue.rs`). Каталог `java_glue.rs` должен 
совпадать с переменной среды `OUT_DIR`. После этого используется `rifgen` 
для генерации файла интерфейса, с нашими предпочтениями в зависимости от языка, 
к которому мы добавляем проект Rust. 
Последний параметр `Generator::new` указывает начало папки, 
содержащей наш Rust код, а функция `generate_interface` принимает путь 
к файлу интерфейса, то есть `in_scr`. Папка `java_folder` указывает, 
куда должны помещаться созданные java-файлы. Вызов swig_gen.expand 
указывает `flapigen` сгенерировать соответствующие файлы. 
Обратите внимание на эти параметры, поскольку мы используем их в сборке Gradle.

Наконец, создайте файл с именем `java_glue.rs` в папке `rust_lib/src` и поместите в него следующее содержимое:

[//]: # (<script src="https://gist.github.com/Kofituo/3cc45830466f98b449951a795592d783.js"></script>)

```rust
#![allow(
clippy::enum_variant_names,
clippy::unused_unit,
clippy::let_and_return,
clippy::not_unsafe_ptr_arg_deref,
clippy::cast_lossless,
clippy::blacklisted_name,
clippy::too_many_arguments,
clippy::trivially_copy_pass_by_ref,
clippy::let_unit_value,
clippy::clone_on_copy
)]

include!(concat!(env!("OUT_DIR"), "/java_glue.rs"));
```

И добавьте:

```rust
mod java_glue;
pub use crate::java_glue::*;
```

в ваш файл `lib.rs`. Это соединит ваш Rust код со сгенерированным.

### Добавляем Android Toolchain и линкеры ⛓

Чтобы скомпилировать Rust для Android, нам нужно добавить инструменты Android в `rustup`. Для этого просто запустите в Терминале следующее:

```terminal
>rustup default nightly 
>rustup target add aarch64-linux-android armv7-linux-androideabi
```

Поскольку `rifgen` крейт работает с nightly, вам нужно сначала установить Rust nightly.

Затем вы добавляете наборы инструментов Rust и стандартную библиотеку для 64-битной и 
32-битной версий Android соответственно. Теперь давайте добавим компоновщики компилятора. 
Его нужно добавить в файл `rust_lib/.cargo/config.toml`.

Щелкните правой кнопкой мыши папку `rust_lib`, создайте новый каталог с именем `.cargo`, 
затем создайте новый файл в каталоге `.cargo` с именем `config.toml`.

Соответствующие компоновщики поставляются с Android NDK, поэтому их следует загрузить, 
прежде чем продолжить.

Добавьте следующее в созданный файл конфигурации.

[//]: # (<script src="https://gist.github.com/Kofituo/271dff7c2175ca6e0f28cfb2c73e7dd5.js"></script>)

```toml
[target.aarch64-linux-android]
linker = "{ANDROID SDK PATH}//ndk-bundle/toolchains/llvm/prebuilt/{OS VERSION}/bin/aarch64-linux-android21-clang++"

[target.armv7-linux-androideabi]
linker = "{ANDROID SDK PATH}/ndk-bundle/toolchains/llvm/prebuilt/{OS VERSION}/bin/armv7a-linux-androideabi21-clang++"
```

Замените ANDROID SDK на путь к Android SDK (или папку, содержащую пакет NDK). 
Кроме того, замените OS VERSION на вашу версию ОС. 
Например, на моем компьютере с Windows полный путь:

```toml
[target.aarch64-linux-android]
linker = "C:\\Users\\dev3java\\AppData\\Local\\Android\\Sdk\\ndk-bundle\\toolchains\\llvm\\prebuilt\\windows-x86_64\\bin\\aarch64-linux-android21-clang++.cmd"
```

Обратите внимание на `.cmd` в конце сборки Windows.

>Если вы используете Mac OS с NDK v25, вы можете столкнуться с проблемой. 
>Если это так — попробуйте предыдущие версии NDK.
{: .prompt-info }

## Gradle для автоматизации сборки 🐘

Gradle можно использовать для автоматического запуска сборки cargo всякий раз, когда мы хотим протестировать приложение. 
Это приведет к тому, что изменения, которые мы сделали на стороне Rust, будут автоматически обновлены в нашем приложении.

[//]: # (<script src="https://gist.github.com/Kofituo/58647a68d5807afe0ae97c12ff8addc4.js"></script>)

```groovy
plugins {
    id 'com.android.application'
    id 'org.jetbrains.kotlin.android'
}

android {
    compileSdk 31

    defaultConfig {
        applicationId "com.example.rustapplication"
        minSdk 21
        targetSdk 31
        versionCode 1
        versionName "1.0"

        testInstrumentationRunner "androidx.test.runner.AndroidJUnitRunner"
        vectorDrawables {
            useSupportLibrary true
        }
        ndk.abiFilters 'armeabi-v7a','arm64-v8a','x86','x86_64' // add this line
    }
    
    sourceSets {
        main {
            jniLibs.srcDirs = ['src/main/libs']
        }
    }
    
    buildTypes {
        release {
            minifyEnabled false
            proguardFiles getDefaultProguardFile('proguard-android-optimize.txt'), 'proguard-rules.pro'
        }
    }
    compileOptions {
        sourceCompatibility JavaVersion.VERSION_1_8
        targetCompatibility JavaVersion.VERSION_1_8
    }
    kotlinOptions {
        jvmTarget = '1.8'
    }
    buildFeatures {
        compose true
    }
    composeOptions {
        kotlinCompilerExtensionVersion compose_version
    }
    packagingOptions {
        resources {
            excludes += '/META-INF/{AL2.0,LGPL2.1}'
        }
    }
}

dependencies {
  
    implementation 'androidx.core:core-ktx:1.7.0'
    implementation "androidx.compose.ui:ui:$compose_version"
    implementation "androidx.compose.material:material:$compose_version"
    implementation "androidx.compose.ui:ui-tooling-preview:$compose_version"
    implementation 'androidx.lifecycle:lifecycle-runtime-ktx:2.3.1'
    implementation 'androidx.activity:activity-compose:1.3.1'
    testImplementation 'junit:junit:4.13.2'
    androidTestImplementation 'androidx.test.ext:junit:1.1.3'
    androidTestImplementation 'androidx.test.espresso:espresso-core:3.4.0'
    androidTestImplementation "androidx.compose.ui:ui-test-junit4:$compose_version"
    debugImplementation "androidx.compose.ui:ui-tooling:$compose_version"
}

// add the following
def rustBasePath = "../rust_lib"
def archTriplets = [
        'armeabi-v7a': 'armv7-linux-androideabi',
        'arm64-v8a': 'aarch64-linux-android',
]

archTriplets.each { arch, target ->
    project.ext.cargo_target_directory = rustBasePath + "/target"
    // Build with cargo
    tasks.create(name: "cargo-build-${arch}", type: Exec, description: "Building core for ${arch}") {
        workingDir rustBasePath
        commandLine 'cargo', 'build', "--target=${target}", '--release'
    }
    // Sync shared native dependencies
    tasks.create(name: "sync-rust-deps-${arch}", type: Sync, dependsOn: "cargo-build-${arch}") {
        from "${rustBasePath}/src/libs/${arch}"
        include "*.so"
        into "src/main/libs/${arch}"
    }
    // Copy build libs into this app's libs directory
    tasks.create(name: "rust-deploy-${arch}", type: Copy, dependsOn: "sync-rust-deps-${arch}", description: "Copy rust libs for (${arch}) to jniLibs") {
        from "${project.ext.cargo_target_directory}/${target}/release"
        include "*.so"
        into "src/main/libs/${arch}"
    }

    // Hook up tasks to execute before building java
    tasks.withType(JavaCompile) {
        compileTask -> compileTask.dependsOn "rust-deploy-${arch}"
    }
    preBuild.dependsOn "rust-deploy-${arch}"

    // Hook up clean tasks
    tasks.create(name: "clean-${arch}", type: Delete, description: "Deleting built libs for ${arch}", dependsOn: "cargo-output-dir-${arch}") {
        delete fileTree("${project.ext.cargo_target_directory}/${target}/release") {
            include '*.so'
        }
    }
    clean.dependsOn "clean-${arch}"
}
```

Обратите внимание, что это файл Gradle уровня приложения или модуля, 
а не файл Gradle уровня проекта. Добавьте строки с 20 по 27 и строку 65 и далее. 
Строка 65 и далее просто указывает Gradle запускать сборку cargo всякий раз, 
когда мы запускаем приложение. После запуска команды сборки cargo мы копируем 
созданные java-файлы и файл динамической библиотеки (`.so`) и вставляем их в каталог, 
который может быть прочитан Android и использован в нашем коде Kotlin.

Теперь приложение должно работать 😄.

## Бонус: ведение логов 📃

Давайте быстро реализуем ведение журнала, чтобы облегчить отладку нашего кода на Rust. 
В файл `lib.rs` добавьте следующее содержимое:

[//]: # (<script src="https://gist.github.com/Kofituo/d44d8443441cade4655dea852fe60c6e.js"></script>)

```rust
mod java_glue;
pub use crate::java_glue::*;

use android_logger::Config;
use log::Level;
use rifgen::rifgen_attr::*;

pub struct RustLog;

impl RustLog {
    //set up logging
    #[generate_interface]
    pub fn initialise_logging() {
        #[cfg(target_os = "android")]
            android_logger::init_once(
            Config::default()
                .with_min_level(Level::Trace)
                .with_tag("Rust"),
        );
        log_panics::init();
        log::info!("Logging initialised from Rust");
    }
}
```

Мы используем крейт `android_logger` для создания журналов, а затем вызываем `log_panics::init()`, 
чтобы перенаправить все паники для регистрации, а не для вывода стандартной ошибки. 
Обратите внимание на атрибут `#[generate_interface]`. 
Это сообщает `rifgen`, что мы будем вызывать эту функцию из Kotlin, поэтому он должен 
добавить этот вызов метода в файл интерфейса.

### Загрузка библиотеки из Kotlin 📲

Теперь, когда мы настроили сторону Rust, давайте перейдем к стороне Kotlin. 
Теперь мы загрузим библиотеку из синглтона:

[//]: # (<script src="https://gist.github.com/Kofituo/865606f5c700d5b64f4ea5bab9baf25b.js"></script>)

```kotlin
package com.example.rustapplication

import android.os.Bundle
import androidx.activity.ComponentActivity
import androidx.activity.compose.setContent
import androidx.compose.foundation.layout.fillMaxSize
import androidx.compose.material.MaterialTheme
import androidx.compose.material.Surface
import androidx.compose.material.Text
import androidx.compose.runtime.Composable
import androidx.compose.ui.Modifier
import androidx.compose.ui.tooling.preview.Preview
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
                // A surface container using the 'background' color from the theme
                Surface(
                    modifier = Modifier.fillMaxSize(),
                    color = MaterialTheme.colors.background
                ) {
                    Greeting("Android With Rust")
                }
            }
        }
    }
}


@Composable
fun Greeting(name: String) {
    Text(text = "Hello $name!")
}
```

Добавьте различный импорт для созданного нами класса Logs. 
Обратите внимание, хотя мы и назвали функцию `initialise_logging`, 
мы можем использовать ее как `initialiseLogging`. 
Это потому, что мы указали CamelCase при настройке `rifgen`.

Запустите программу, чтобы увидеть информацию из логов кода Rust.

[Источник][1-gitconnected.com]

[Репозиторий][2-repository]

## Следующие шаги 🍀

На этом мы завершаем первую часть этой серии. В следующей [части][integrating-rust-with-android-app-dev-final-part] мы обсудим наш код, чтобы завершить это пошаговое руководство.

<!-- Ссылки -->
[1-gitconnected.com]: https://levelup.gitconnected.com/integrating-rust-with-android-development-ef341c2f9cca "gitconnected.com"
[2-repository]: https://github.com/dev3java/RustApplication "github.com"

<!-- Ссылки по тексту -->
[1-building-rust-modules]: https://source.android.com/setup/build/rust/building-rust-modules/overview "android.com"
[2-android-studio]: https://developer.android.com/studio "android.com"
[3-rust-tools-install]: https://www.rust-lang.org/tools/install "rust-lang.org"
[4-rust-cargo]: https://doc.rust-lang.org/cargo/ "rust-lang.org/cargo"
[5-rust-cargo-guide]: https://doc.rust-lang.org/cargo/guide/creating-a-new-project.html "rust-lang.org/cargo/guide"
[6-rust-cargo-reference]: https://doc.rust-lang.org/cargo/reference/cargo-targets.html#configuring-a-target "rust-lang.org/cargo/reference"

[1-gist-cargo.toml]: https://gist.github.com/dev3java/e02bd971b75866103438961e98e3711f "gist.github.com"

[1-crates-flapigen]: https://crates.io/crates/flapigen "crates.io/crates/flapigen"
[2-crates-rifgen]: https://crates.io/crates/rifgen "crates.io/crates/rifgen"
[3-docs-rifgen]: https://docs.rs/rifgen/latest/rifgen/ "docs.rs/rifgen"
[4-github-flapigen-rs]: https://dushistov.github.io/flapigen-rs/java-android-example.html "github.io/flapigen-rs"

[integrating-rust-with-android-app-dev-final-part]: /posts/integrating-rust-with-android-app-dev-final-part "Финальная часть"
