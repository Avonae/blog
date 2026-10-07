---
title: 'How to Remove Instagram Ads on Android with ReVanced (2026)'
date: '2026-10-07'
description: Instagram ad patch broken in the ReVanced release? Build ReVanced patches from the dev branch on Linux, macOS or Windows and patch Instagram in Termux, no root.
tags:
- ReVanced
- Android
- Instagram
- Termux
- Guide
translationKey: revanced-build-patches-from-dev-branch
---

I use [ReVanced Manager](https://revanced.app/download), an app that patches APK files on Android and adds all kinds of improvements to them. No root required.
What patched apps can do:
- Block ads
- Tweak the feed, for example hide live streams in TikTok
- Add features like SponsorBlock in YouTube

I use a lot of patched apps: Instagram, YouTube, Reddit and TikTok. Recently I decided to update Instagram.

It turned out that the Instagram ad patch in the current ReVanced release doesn't work. Luckily, the fix is already in the dev branch. So you have to build the patches from dev yourself...
It took me about 30 minutes of tinkering, but now I have Instagram without ads.

The process has two stages:

1. Build the `.rvp` patches file from source on a computer.
2. Patch the APK with this file using ReVanced CLI in Termux on your phone.

## What you need

- A [GitHub](https://github.com/) account.
- The [Termux terminal app](https://f-droid.org/en/packages/com.termux/) installed from F-Droid.
- The APK of the app you want to patch. For Instagram you need version 425.0.0.47.61, you can [download it from APKMirror](https://www.apkmirror.com/apk/instagram/instagram-instagram/instagram-425-0-0-47-61-release/instagram-425-0-0-47-61-20-android-apk-download/).

## Stage 1. Build ReVanced patches from source

### Install dependencies

You need JDK 17 and Git.

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

### GitHub token

The project dependencies are hosted in GitHub Packages, so you need a token that can read `packages`.

Create a **classic** token at https://github.com/settings/tokens/new with the `read:packages` scope. Then put it into `gradle.properties` in your home folder:

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

### Clone the dev branch

The ReVanced patches repository now lives on GitLab. Clone the `dev` branch from there:

```bash
git clone -b dev https://gitlab.com/ReVanced/revanced-patches.git
cd revanced-patches
```

### Android SDK

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

### Fix minSdk for Twitter

Right now (as of October 7, 2026) the dev branch has a small bug: Call requires API level 33 (current min is 26): java.io.InputStream#readAllBytes [NewApi]

![ReVanced patches build error: Call requires API level 33 in the Twitter extension](twitter-minsdk-error.png)

Because of it, the build fails at compile time. To fix it, open `extensions/twitter/build.gradle.kts` and change `minSdk = 26` to `minSdk = 33`.

### Build

From the `revanced-patches` folder, run:

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

## Common problems

- **"App not installed" when installing the patched APK.** The signature differs from the original. Uninstall the original app first.
- **`termux-change-repo` or `pkg` doesn't work.** You have Termux from Google Play. Install it from F-Droid.
- **aapt2 error during patching.** The `libaapt2.so` doesn't match your CPU. Check `getprop ro.product.cpu.abi` and download the right one.
- **Build fails with `Call requires API level 33`.** Apply the minSdk fix for Twitter from stage 1.

Done. Now you have Instagram without ads, you're awesome.
