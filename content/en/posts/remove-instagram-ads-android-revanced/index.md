---
title: 'How to Remove Instagram Ads on Android with ReVanced (2026)'
date: '2026-10-07'
lastmod: '2026-10-10'
description: Instagram ad patch broken in the ReVanced release? Build ReVanced patches from the dev branch on Linux, macOS or Windows and patch Instagram in Termux, no root.
tags:
- Revanced
- Android
- Instagram
- Termux
- guide
categories:
- Software and Services
translationKey: revanced-build-patches-from-dev-branch
---

I use [ReVanced Manager](https://revanced.app/download), an app that patches APK files on Android and adds all kinds of improvements to them. No root required.
What patched apps can do:
- Block ads
- Tweak the feed, for example hide live streams in TikTok
- Add features like SponsorBlock in YouTube

I use a lot of these apps: Instagram, YouTube, Reddit and TikTok. Recently I wanted a newer Instagram, but it turned out that the Instagram ad patch in the current ReVanced release doesn't work. Sad. The fix is in the dev branch of the patches. So you have to build them from dev yourself... Below I explain how.

The process has two stages:

1. Build the `.rvp` patches file from source on a computer.
2. Patch the APK with this file using ReVanced CLI in Termux on your phone.

## What you need

- A [GitHub](https://github.com/) account and a Personal Access Token (PAT). Create it [here](https://github.com/settings/tokens/new) and select only the `read:packages` scope.
- The [Termux terminal app](https://f-droid.org/en/packages/com.termux/) installed from F-Droid.
- The APK of the app you want to patch. For Instagram you need version 425.0.0.47.61, you can [download it from APKMirror](https://www.apkmirror.com/apk/instagram/instagram-instagram/instagram-425-0-0-47-61-release/instagram-425-0-0-47-61-20-android-apk-download/).

## Stage 1. Build ReVanced patches from source

There are 2 ways:
1. A script I made:

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

   The code is open and available on GitHub.
2. Running the commands manually.

Both ways need JDK 17 and Git:

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
Install [JDK 17](https://adoptium.net/temurin/releases/?version=17) and [Git for Windows](https://git-scm.com/download/win).
{{< /tab >}}
{{< /tabs >}}

### 1. Build with the script

I put the commands into a script that does most of the work. Its [code is on GitHub](https://raw.githubusercontent.com/Avonae/Scripts/main/revanced/build-revanced.sh), it's just a set of commands. Before it starts, the script only asks for your GitHub username and token (the PAT we created earlier).

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
Run in PowerShell:
```powershell
irm https://raw.githubusercontent.com/Avonae/Scripts/main/revanced/build-revanced.ps1 | iex
```
{{< /tab >}}
{{< /tabs >}}

The `.rvp` patches file will be in `~/revanced-patches/patches/build/libs/`. Running the script again updates the repository and rebuilds.

### 2. Build manually

**1. GitHub username and token.** The project dependencies are hosted in GitHub Packages, so the build needs a token to read them. Put your username and token into `gradle.properties` in your home folder:

```
githubPackagesUsername=your_github_username
githubPackagesPassword=ghp_...
```

{{< tabs group="os" default="macOS" >}}
{{< tab label="Linux" >}}
File path: `~/.gradle/gradle.properties`.
{{< /tab >}}
{{< tab label="macOS" >}}
File path: `~/.gradle/gradle.properties`.
{{< /tab >}}
{{< tab label="Windows" >}}
File path: `C:\Users\<name>\.gradle\gradle.properties`.
{{< /tab >}}
{{< /tabs >}}

**2. Clone the dev branch** from GitLab:

```bash
git clone -b dev https://gitlab.com/ReVanced/revanced-patches.git
cd revanced-patches
```

**3. Install the Android SDK**

1. Download the Android SDK from the [Android Studio page](https://developer.android.com/studio#command-line-tools-only), section "Command line tools only".
2. Put the `licenses` folder into `~/Android/dependency/`. It must contain the `licenses/android-sdk-license` file.
3. Create a `local.properties` file in the repository root and set the path to the SDK folder in it. Replace `user` with your username.

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

Escape the colon and backslashes with a backslash, otherwise Gradle reads the path incorrectly.
{{< /tab >}}
{{< /tabs >}}

**4. Fix minSdk for Twitter.** Right now (as of October 7, 2026) the dev branch has a small bug: Call requires API level 33 (current min is 26): java.io.InputStream#readAllBytes [NewApi]

![ReVanced patches build error: Call requires API level 33 in the Twitter extension](twitter-minsdk-error.png)

Because of it, the build fails at compile time. To fix it, open `extensions/twitter/build.gradle.kts` and change `minSdk = 26` to `minSdk = 33`.

**5. Build.** From the `revanced-patches` folder, run:

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

The result appears at `patches/build/libs/patches-<version>.rvp`. You don't need the file with the `-sources.rvp` suffix.

## Stage 2. Patch the APK with ReVanced CLI in Termux

Now we build the app itself from the patches. We'll do it on the phone.

### Preparation

1. Install [Termux from F-Droid](https://f-droid.org/en/packages/com.termux/). The Google Play version won't work.
2. In a file manager, create a `revanced` folder inside `Download`.
3. Put three files into it:
   - the `.jar` from [ReVanced CLI](https://github.com/revanced/revanced-cli/);
   - the `.rvp` patches file built in stage 1;
   - the app APK. I patched Instagram 425.0.0.47.61, you can [download it from APKMirror](https://www.apkmirror.com/apk/instagram/instagram-instagram/instagram-425-0-0-47-61-release/instagram-425-0-0-47-61-20-android-apk-download/).

In the end, `/storage/emulated/0/Download/revanced/` should contain 3 files.

### Set up Termux

1. Give Termux access to your files and accept the permission request:

   ```bash
   termux-setup-storage
   ```

2. Save the folder path to a variable:

   ```bash
   revanced=/storage/emulated/0/Download/revanced
   ```

3. Pick a package repository mirror:

   ```bash
   termux-change-repo
   ```

4. Install wget and the JDK:

   ```bash
   pkg install wget openjdk-17 -y
   ```

### Download aapt2

ReVanced CLI needs `aapt2` built for your phone's CPU architecture. Check it with this command in Termux:

```bash
getprop ro.product.cpu.abi
```

The command prints `arm64-v8a` or `armeabi-v7a`. Most modern phones are `arm64-v8a`.

Download the library for your architecture:

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

### Patch the app

```bash
java -jar $revanced/*.jar patch -bp $revanced/*.rvp --custom-aapt2-binary ./libaapt2.so $revanced/*.apk
```

The command creates a file with the `-patched` suffix. Move it to the `revanced` folder:

```bash
mv *-patched.apk $revanced
```

### Install

The patched app is signed with a different key, so it usually won't install over the original. Uninstall the current app and install the `-patched.apk` from scratch.

Done. Now you have Instagram without ads, you're awesome.

## Build on the phone

You can also compile the patches on the phone. It takes a bit longer, but you don't need a computer. This works on the aarch64 architecture.

There are 2 ways here too:
1. A [shell script](https://github.com/Avonae/Scripts/blob/main/revanced/build-revanced-termux.sh) I made. The code is open and available on GitHub.
2. Running the commands manually.

Both ways need:

- A [GitHub](https://github.com/) account and a Personal Access Token (PAT). Create it [here](https://github.com/settings/tokens/new) and select only the `read:packages` scope.
- The [Termux terminal app](https://f-droid.org/en/packages/com.termux/) installed from F-Droid.

### 1. Build with the script

There is a script for compiling on the phone too. Its [code is on GitHub](https://raw.githubusercontent.com/Avonae/Scripts/main/revanced/build-revanced-termux.sh), it's just a set of commands. Before it starts, the script only asks for your GitHub username and token (the PAT we created earlier).

Before running the script, run:
```bash
termux-setup-storage
```

Give the terminal access to your files and run the script:
```bash
bash <(curl -fsSL https://raw.githubusercontent.com/Avonae/Scripts/main/revanced/build-revanced-termux.sh)
```

The `.rvp` patches file will be in `~/revanced-patches/patches/build/libs/` and in Downloads. Running the script again updates the repository and rebuilds.

### 2. Build manually

**1. Packages**
```bash
pkg upgrade -y
pkg install -y git openjdk-17 aapt2 aidl protobuf
```

**2. Repository** (temporarily hosted on GitLab). Clone the `dev` branch and fix minSdk for Twitter right away, as in step 4 of the computer build:
```bash
git clone -b dev https://gitlab.com/revanced/revanced-patches
cd revanced-patches && chmod +x gradlew
sed -i 's/minSdk = 26/minSdk = 33/' extensions/twitter/build.gradle.kts
```

**3. SDK and licenses.** Without the license file, Gradle refuses to download Platform and Build-Tools.
```bash
mkdir -p ~/android-sdk/licenses
printf '\n24333f8a63b6825ea9c5514f83c2829b004d1fee\nd56f5187479451eabf01fb78af6dfcb131a6481e\n84831b9409646a918e30573bab4c9c91346d8abd\n' \
  > ~/android-sdk/licenses/android-sdk-license
echo "sdk.dir=$HOME/android-sdk" > local.properties
echo 'export ANDROID_HOME=$HOME/android-sdk' >> ~/.bashrc
export ANDROID_HOME=$HOME/android-sdk
```
By creating this file, you accept the [Android SDK License](https://developer.android.com/studio/terms).

**4. `~/.gradle/gradle.properties`.** Use exactly these key names, `gpr.user`/`gpr.key` don't work.
```properties
githubPackagesUsername=your_username
githubPackagesPassword=ghp_xxx
android.aapt2FromMavenOverride=/data/data/com.termux/files/usr/bin/aapt2
org.gradle.jvmargs=-Xmx2g
```

**5. Use protoc from Termux instead of the downloaded one**
```bash
sed -i 's|artifact = .*protoc.*|path = "/data/data/com.termux/files/usr/bin/protoc"|' \
  extensions/shared/protobuf/build.gradle.kts
```

**6. protobuf-javalite version = protoc version.** Otherwise you get `cannot find symbol throwCannotGetNumberOfUnrecognized`. Rule: `libprotoc 35.1` → `4.35.1`.
```bash
protoc --version
sed -i 's|^protoc = ".*"|protoc = "4.35.1"|' gradle/libs.versions.toml
```
Don't touch the `protobuf = "master-SNAPSHOT"` key (the plugin version).

**7. First build** (downloads the SDK, fails on `aidl`)
```bash
./gradlew build --no-daemon
```

**8. Replace aidl in build-tools and build again**
```bash
BT=~/android-sdk/build-tools/36.0.0
mv $BT/aidl $BT/aidl.x86 && ln -s $PREFIX/bin/aidl $BT/aidl
./gradlew build --no-daemon
```
Result: `patches/build/libs/patches-*.rvp`.

**9. Apply the patches**
All that's left is to patch the APK with the `rvp` file you got:

```bash
java -jar revanced-cli.jar patch -bp patches.rvp \
  --custom-aapt2-binary $PREFIX/bin/aapt2 app.apk
```

## Troubleshooting

### When building the patches

- **`missing ... githubPackagesUsername / Password`.** Gradle can't find your GitHub username and token. Check `gradle.properties`: step 1 on the computer or step 4 on the phone.
- **`SDK location not found`.** There is no `local.properties` in the project root. Do step 3.
- **`licences have not been accepted`.** The `licenses` folder is empty. Do step 3.
- **`Call requires API level 33`.** In the dev branch, the Twitter extension requires API 33. Fix minSdk: step 4 on the computer or step 2 on the phone.
- **`generateProto ... protoc: stdout: . stderr:`.** Phone only: Gradle downloaded an x86/glibc protoc. Do step 5.
- **`cannot find symbol throwCannot...`.** Phone only: the protobuf-javalite version is older than protoc. Do step 6.
- **`compileReleaseAidl ... Error while executing process .../aidl`.** Phone only: build-tools contains an x86 aidl. Do step 8.
- **Termux closes silently.** Not enough memory. Set `-Xmx1536m` and disable battery optimization for Termux.

### When patching and installing

- **`termux-change-repo` doesn't exist.** You have Termux from Google Play. Install it from F-Droid.
- **aapt2 error during patching.** The `libaapt2.so` doesn't match your CPU. Check the architecture with `getprop ro.product.cpu.abi` and download the right library.
- **"App not installed" when installing the patched APK.** Uninstall the original app and install from scratch.
