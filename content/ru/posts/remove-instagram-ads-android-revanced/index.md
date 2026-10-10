---
title: 'Убираем рекламу в инстаграме на андроид в 2026'
date: '2026-10-07'
lastmod: '2026-10-10'
description: Как собрать патчи для ReVanced из dev-ветки на Linux, macOS или Windows и пропатчить Instagram через ReVanced CLI в Termux прямо на телефоне без root.
tags:
- Revanced
- Android
- Instagram
- Termux
- инструкция
categories:
- Софт и сервисы
translationKey: revanced-build-patches-from-dev-branch
aliases:
- /posts/revanced-build-patches-from-dev-branch/
---

Я использую [Revanced Manager](https://revanced.app/download) - приложение, позволяющее пропатчить APK файл на андроиде и применить к нему всякие улучшения. Рут для него не требуется. 

Что умеют патченные приложения:
- Отключать рекламу
- Настраивать ленту, например, отключить стримы в тиктоке
- Добавлять функции вроде sponsorblock на ютубе

Я использую много таких приложений: инстаграм, ютуб, реддит и тикток. И недавно решил поставить себе инстаграм поновее, но выяснилось, что в актуальном релизе ReVanced патч на рекламу для инсты не работает. Печально. Исправление нашлось в dev-ветке патчей. Поэтому их придётся собрать из dev самому... Ниже рассказываю как.

Процесс состоит из двух этапов:

1. Собрать файл патчей `.rvp` из исходников на компьютере.
2. Пропатчить APK этим файлом через ReVanced CLI в Termux на телефоне.

## Подготовка
Вам понадобится:

- Аккаунт на [гитхабе](https://github.com/) и Personal Access Token (PAT). Сделайте его [по ссылке](https://github.com/settings/tokens/new), scope укажите только `read:packages`
- Приложение [терминала Termux](https://f-droid.org/en/packages/com.termux/), установленное через F-Droid.
- APK приложения, которое хотите пропатчить. Для инстаграма нужна версия 425.0.0.47.61, [скачать ее можно с APK Mirror](https://www.apkmirror.com/apk/instagram/instagram-instagram/instagram-425-0-0-47-61-release/instagram-425-0-0-47-61-20-android-apk-download/) 

## Этап 1. Сборка патчей ReVanced из исходников

Способа 2:
1. Скрипт, который я сделал:

   {{< tabs group="os" default="macOS" >}}
   {{< tab label="Linux" >}}
   [build-revanced.sh](https://github.com/Avonae/Scripts/blob/main/revanced/build-revanced.sh)
   {{< /tab >}}
   {{< tab label="macOS" >}}
   [build-revanced.sh](https://github.com/Avonae/Scripts/blob/main/revanced/build-revanced.sh)
   {{< /tab >}}
   {{< tab label="Windows" >}}
   [build-revanced.ps1](https://github.com/Avonae/Scripts/blob/main/revanced/build-revanced.ps1)
   {{< /tab >}}
   {{< /tabs >}}

   Код открыт и доступен на гитхабе.
2. Выполнение команд вручную.

В обоих случаях нужны JDK 17 и git:

{{< tabs group="os" default="macOS" >}}
{{< tab label="Linux" >}}
```bash
sudo apt install openjdk-17-jdk git
```
{{< /tab >}}
{{< tab label="macOS" >}}
```bash
brew install openjdk@17 git
```
{{< /tab >}}
{{< tab label="Windows" >}}
Установите [JDK 17](https://adoptium.net/temurin/releases/?version=17) и [Git for Windows](https://git-scm.com/download/win).
{{< /tab >}}
{{< /tabs >}}

### 1. Установка скриптом

Я собрал команды в скрипт, который сделает большую часть работы сам. Его [код доступен на гитхабе](https://raw.githubusercontent.com/Avonae/Scripts/main/revanced/build-revanced.sh), это просто набор команд. Перед началом работы скрипт только спросит логин от гитхаба и токен (PAT, который мы сгенерировали ранее).

{{< tabs group="os" default="macOS" >}}
{{< tab label="Linux" >}}
```bash
bash <(curl -fsSL https://raw.githubusercontent.com/Avonae/Scripts/main/revanced/build-revanced.sh)
```
{{< /tab >}}
{{< tab label="macOS" >}}
```bash
bash <(curl -fsSL https://raw.githubusercontent.com/Avonae/Scripts/main/revanced/build-revanced.sh)
```
{{< /tab >}}
{{< tab label="Windows" >}}
Запустите в PowerShell:
```powershell
irm https://raw.githubusercontent.com/Avonae/Scripts/main/revanced/build-revanced.ps1 | iex
```
{{< /tab >}}
{{< /tabs >}}

Готовый файл с патчами `.rvp` будет в `~/revanced-patches/patches/build/libs/`. Повторный запуск обновляет репозиторий и пересобирает.

### 2. Установка вручную

**1. Логин и токен GitHub.** Зависимости проекта лежат в GitHub Packages, поэтому компилятору нужен токен для их чтения. Пропишите логин и токен в `gradle.properties` в домашней папке:

```
githubPackagesUsername=ваш_логин_на_github
githubPackagesPassword=ghp_...
```

{{< tabs group="os" default="macOS" >}}
{{< tab label="Linux" >}}
Путь к файлу: `~/.gradle/gradle.properties`.
{{< /tab >}}
{{< tab label="macOS" >}}
Путь к файлу: `~/.gradle/gradle.properties`.
{{< /tab >}}
{{< tab label="Windows" >}}
Путь к файлу: `C:\Users\<имя>\.gradle\gradle.properties`.
{{< /tab >}}
{{< /tabs >}}

**2. Клонирование dev-ветки** с гитлаба:

```bash
git clone -b dev https://gitlab.com/ReVanced/revanced-patches.git
cd revanced-patches
```

**3. Установка Android SDK**

1. Скачайте Android SDK со [страницы Android Studio](https://developer.android.com/studio#command-line-tools-only), раздел «Command line tools only».
2. Положите папку `licenses` в `~/Android/dependency/`. Внутри должен быть файл `licenses/android-sdk-license`.
3. Создайте в корне репозитория файл `local.properties` и укажите в нём путь к папке с SDK. Замените `user` на имя своего пользователя.

{{< tabs group="os" default="macOS" >}}
{{< tab label="Linux" >}}
```
sdk.dir=/home/user/Android/dependency
```
{{< /tab >}}
{{< tab label="macOS" >}}
```
sdk.dir=/Users/user/Android/dependency
```
{{< /tab >}}
{{< tab label="Windows" >}}
```
sdk.dir=C\:\\Users\\user\\Documents\\dependency
```

Двоеточие и обратные слеши нужно экранировать обратным слешем, иначе Gradle прочитает путь неправильно.
{{< /tab >}}
{{< /tabs >}}

**4. Правка minSdk для Twitter.** В дев сборке сейчас (на 07.10.26) есть небольшая ошибка: Call requires API level 33 (current min is 26): java.io.InputStream#readAllBytes [NewApi]

![Ошибка сборки патчей ReVanced: Call requires API level 33 в расширении Twitter](twitter-minsdk-error.png)

из-за которой сборка падает при компиляции. Чтобы её исправить, в файле `extensions/twitter/build.gradle.kts` замените `minSdk = 26` на `minSdk = 33`.

**5. Сборка.** Находясь в папке `revanced-patches`, запустите:

{{< tabs group="os" default="macOS" >}}
{{< tab label="Linux" >}}
```bash
./gradlew build
```
{{< /tab >}}
{{< tab label="macOS" >}}
```bash
./gradlew build
```
{{< /tab >}}
{{< tab label="Windows" >}}
```
.\gradlew.bat build
```
{{< /tab >}}
{{< /tabs >}}

Готовый файл появится здесь: `patches/build/libs/patches-<версия>.rvp`. Файл с суффиксом `-sources.rvp` нам не нужен.

## Этап 2. Патчим APK через Termux

Теперь из патчей собираем само приложение. Делать это будем на телефоне.

### Подготовка

1. Установите [Termux из F-Droid](https://f-droid.org/en/packages/com.termux/). Версия из Google Play не подойдёт.
2. В файловом менеджере создайте папку `revanced` внутри `Download`.
3. Положите в неё три файла:
   - `.jar` из [ReVanced CLI](https://github.com/revanced/revanced-cli/);
   - файл патчей `.rvp`, собранный на первом этапе;
   - APK приложения. Я патчил Instagram версии 425.0.0.47.61, его можно [скачать с APKMirror](https://www.apkmirror.com/apk/instagram/instagram-instagram/instagram-425-0-0-47-61-release/instagram-425-0-0-47-61-20-android-apk-download/).

В итоге в папке `/storage/emulated/0/Download/revanced/` должно получиться 3 файла.

### Настройка Termux

1. Дайте Termux доступ к файлам и согласитесь с запросом прав:

   ```bash
   termux-setup-storage
   ```

2. Сохраните путь к папке в переменную:

   ```bash
   revanced=/storage/emulated/0/Download/revanced
   ```

3. Выберите зеркало репозитория пакетов:

   ```bash
   termux-change-repo
   ```

4. Установите wget и JDK:

   ```bash
   pkg install wget openjdk-17 -y
   ```

### Загрузка компилятора андроид aapt2

ReVanced CLI нужен `aapt2`, собранный под архитектуру процессора телефона. Узнайте её командой в Termux:

```bash
getprop ro.product.cpu.abi
```

Команда выведет `arm64-v8a` или `armeabi-v7a`. Большинство современных телефонов — `arm64-v8a`.

Скачайте библиотеку для своей архитектуры:

{{< tabs group="abi" >}}
{{< tab label="arm64-v8a" >}}
```bash
wget https://github.com/ReVanced/revanced-manager/raw/refs/heads/main/app/src/main/jniLibs/arm64-v8a/libaapt2.so && chmod +x libaapt2.so
```
{{< /tab >}}
{{< tab label="armeabi-v7a" >}}
```bash
wget https://raw.githubusercontent.com/ReVanced/revanced-manager/refs/heads/main/app/src/main/jniLibs/armeabi-v7a/libaapt2.so && chmod +x libaapt2.so
```
{{< /tab >}}
{{< /tabs >}}

### Патчим APK файл

```bash
java -jar $revanced/*.jar patch -bp $revanced/*.rvp --custom-aapt2-binary ./libaapt2.so $revanced/*.apk
```

Команда создаст файл с суффиксом `-patched`. Переносим его в папку `revanced`:

```bash
mv *-patched.apk $revanced
```

### Установка патченного приложения

Пропатченное приложение подписано другим ключом, поэтому поверх оригинала оно обычно не ставится. Удалите установленное приложение и поставьте собранное приложение `-patched.apk` с нуля.

Готово. Теперь у вас инста без рекламы, вы великолепны.

## Сборка на телефоне

Патчи можно откомпилить и на телефоне. Это займет чуть больше времени, но зато компьютер не нужен. Способ работает на архитектуре aarch64.

Способа тоже 2:
1. [Шелл-скрипт](https://github.com/Avonae/Scripts/blob/main/revanced/build-revanced-termux.sh), который я сделал. Код открыт и доступен на гитхабе.
2. Выполнение команд вручную

В обоих случаях вам понадобится:

- Аккаунт на [гитхабе](https://github.com/) и Personal Access Token (PAT). Сделайте его [по ссылке](https://github.com/settings/tokens/new), scope укажите только `read:packages`
- Приложение [терминала Termux](https://f-droid.org/en/packages/com.termux/), установленное через F-Droid.

### 1. Установка скриптом

Для компиляции на телефоне тоже есть скрипт. Его [код доступен на гитхабе](https://raw.githubusercontent.com/Avonae/Scripts/main/revanced/build-revanced-termux.sh), это просто набор команд. Перед началом работы скрипт только спросит логин от гитхаба и токен (PAT, который мы сгенерировали ранее).

Перед запуском скрипта выполните:
```bash
termux-setup-storage
```

Дайте доступ терминалу к файлам и запускайте скрипт:
```bash
bash <(curl -fsSL https://raw.githubusercontent.com/Avonae/Scripts/main/revanced/build-revanced-termux.sh)
```

Готовый файл с патчами `.rvp` будет в `~/revanced-patches/patches/build/libs/` и в Downloads. Повторный запуск обновляет репозиторий и пересобирает.

### 2. Установка вручную

**1. Пакеты**
```bash
pkg upgrade -y
pkg install -y git openjdk-17 aapt2 aidl protobuf
```

**2. Репозиторий** (временно живёт на GitLab). Клонируем ветку `dev` и сразу правим minSdk для Twitter, как в шаге 4 сборки на компьютере:
```bash
git clone -b dev https://gitlab.com/revanced/revanced-patches
cd revanced-patches && chmod +x gradlew
sed -i 's/minSdk = 26/minSdk = 33/' extensions/twitter/build.gradle.kts
```

**3. SDK и лицензии.** Без файла лицензии Gradle откажется скачивать Platform и Build-Tools.
```bash
mkdir -p ~/android-sdk/licenses
printf '\n24333f8a63b6825ea9c5514f83c2829b004d1fee\nd56f5187479451eabf01fb78af6dfcb131a6481e\n84831b9409646a918e30573bab4c9c91346d8abd\n' \
  > ~/android-sdk/licenses/android-sdk-license
echo "sdk.dir=$HOME/android-sdk" > local.properties
echo 'export ANDROID_HOME=$HOME/android-sdk' >> ~/.bashrc
export ANDROID_HOME=$HOME/android-sdk
```
Создавая файл, вы принимаете [Android SDK License](https://developer.android.com/studio/terms).

**4. `~/.gradle/gradle.properties`.** Именно такие имена ключей, `gpr.user`/`gpr.key` не работают.
```properties
githubPackagesUsername=ваш_ник
githubPackagesPassword=ghp_xxx
android.aapt2FromMavenOverride=/data/data/com.termux/files/usr/bin/aapt2
org.gradle.jvmargs=-Xmx2g
```

**5. protoc из Termux вместо скачиваемого**
```bash
sed -i 's|artifact = .*protoc.*|path = "/data/data/com.termux/files/usr/bin/protoc"|' \
  extensions/shared/protobuf/build.gradle.kts
```

**6. Версия protobuf-javalite = версии protoc.** Иначе ошибка `cannot find symbol throwCannotGetNumberOfUnrecognized`. Правило: `libprotoc 35.1` → `4.35.1`.
```bash
protoc --version
sed -i 's|^protoc = ".*"|protoc = "4.35.1"|' gradle/libs.versions.toml
```
Ключ `protobuf = "master-SNAPSHOT"` (версия плагина) не трогать.

**7. Первая сборка** (скачает SDK, упадёт на `aidl`)
```bash
./gradlew build --no-daemon
```

**8. Подменить aidl из build-tools и собрать снова**
```bash
BT=~/android-sdk/build-tools/36.0.0
mv $BT/aidl $BT/aidl.x86 && ln -s $PREFIX/bin/aidl $BT/aidl
./gradlew build --no-daemon
```
Результат: `patches/build/libs/patches-*.rvp`.

**9. Применение патчей**
Осталось пропатчить наш APK с помощью полученного `rvp` файла:  

```bash
java -jar revanced-cli.jar patch -bp patches.rvp \
  --custom-aapt2-binary $PREFIX/bin/aapt2 app.apk
```

## Решение проблем

### При сборке патчей

- **`missing ... githubPackagesUsername / Password`.** Gradle не нашёл логин и токен GitHub. Проверьте `gradle.properties`: шаг 1 на компьютере или шаг 4 на телефоне.
- **`SDK location not found`.** В корне проекта нет `local.properties`. Выполните шаг 3.
- **`licences have not been accepted`.** Папка `licenses` пустая. Выполните шаг 3.
- **`Call requires API level 33`.** В dev-ветке расширение Twitter требует API 33. Поправьте minSdk: шаг 4 на компьютере или шаг 2 на телефоне.
- **`generateProto ... protoc: stdout: . stderr:`.** Только на телефоне: Gradle скачал protoc под x86/glibc. Выполните шаг 5.
- **`cannot find symbol throwCannot...`.** Только на телефоне: версия protobuf-javalite старше protoc. Выполните шаг 6.
- **`compileReleaseAidl ... Error while executing process .../aidl`.** Только на телефоне: в build-tools лежит aidl под x86. Выполните шаг 8.
- **Termux молча закрывается.** Не хватает памяти. Укажите `-Xmx1536m` и отключите оптимизацию батареи для Termux.

### При патчинге и установке

- **Команды `termux-change-repo` не существует.** У вас Termux из Google Play. Установите его из F-Droid.
- **Ошибка aapt2 при патчинге.** `libaapt2.so` не подходит к процессору. Проверьте архитектуру командой `getprop ro.product.cpu.abi` и скачайте нужную библиотеку.
- **«Приложение не установлено» при установке пропатченного APK.** Удалите оригинальное приложение и установите с нуля.
