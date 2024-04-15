# .NET MAUIをLinuxでビルドする環境構築

## 1. 目的

.NET MAUIの環境をLinuxで開発、ビルドするために環境を構築する

### 1-1. 参考サイト

[Ubuntu(CLI)上での.NET MAUI androidアプリ用ビルド環境構築方法](https://qiita.com/teiron/items/78869833b024c31bdfc3)  
[Develop .NET MAUI Apps on Linux with VS Code: Complete Guide](https://www.youtube.com/watch?v=4D2vUYUIqFU)  
[Ubuntu20.04にAndroidのNDK,SDKをコマンドラインでインストール](https://blog.123abcsoft.com/2021/01/20/765/)  
[Android Studioを入れずにAndroid SDKをインストールする方法(Mac)](https://qiita.com/uhooi/items/e94575b18e785c747386)  

## 2. 準備

ubuntu server 22.04を最小構成でインストール

## 3. インストール

### 3-1. はじめに

元々はディストリビューション(今回の場合はubuntu server 22.04)のパッケージマネージャだけでやろうとしたのですが、javaやAndroid SDKの細かいバージョンがあったりして、思ったようにいかなかったため、dotnet、java、android-sdkはすべて手動でインストールしました  
おそらく、この方法が一番いいんじゃないかと  
Windowsでやれ!!  
って声が聞こえて来そうですが、多分その通りかとw  
ただ、私の環境下にWindowsがありませんので…ww

### 3-2. 構成

<table>
    <tr>
        <th>dotnet</th>
        <td>dotnet8</th>
    </th>
    <tr>
        <th>Android SDK</th>
        <td>34</th>
    </tr>
    <tr>
        <th>OpenJDK</th>
        <td>17</th>
    </tr>
</table>

上記構成はdotnet8以外意図していたものではなく、dotnet8で進める以上これが最善ということになったためです

### 3-1. 必要なパッケージのインストール

```console
$ sudo apt update
$ sudo apt upgrade
$ sudo apt install vim
```

テキストエディタはお好みのものを

### 3-2. .NET8のインストール

[.NET インストール スクリプト](https://dotnet.microsoft.com/ja-jp/download/dotnet/scripts)  
より今回はbash用のスクリプトをダウンロードしてインストールする

```console
$ curl -L -O https://dotnet.microsoft.com/download/dotnet/scripts/v1/dotnet-install.sh
$ chmod +x dotnet-install.sh
$ ./dotnet-install.sh  --channel 8.0
```
2024-04-13時点、--channel 8.0を付けなくても最新は8.0がインストールされる  

~/.dotnetにPATHを通す
```console
$ cd
$ cp -p .bashrc dot.bashrc
$ echo 'export DOTNET_ROOT=$HOME/.dotnet' >> .bashrc
$ echo 'export PATH=$PATH:$DOTNET_ROOT:$DOTNET_ROOT/tools' >> .bashrc
$ exec $SHELL -l
$ dotnet --version
8.0.204
```
dotnetコマンドでバージョンを確認できたらOK.

## 4. ビルド

以降はビルドしながら必要なものを入れていく

### 4-1. maui-androidのworkloadのインストール

まずは、maui-androidのworkloadをインストール
```console
$ dotnet workload install maui-android
```

### 4-2. MAUIプロジェクト作成

MAUI用の新規プロジェクトを作成
```console
$ dotnet new maui --name MauiSample
```

Linuxでは、androidしか対応していないので、.csprojファイルを以下のように変更する
```console
$ cd MauiSample
$ vi .MauiSample.csproj

<TargetFrameworks>net8.0-android;net8.0-ios;net8.0-maccatalyst</TargetFrameworks>
<TargetFrameworks Condition="$([MSBuild]::IsOSPlatform('windows'))">$(TargetFrameworks);net8.0-windows10.0.19041.0<TargetFrameworks>
↓
<TargetFrameworks>net8.0-android</TargetFrameworks>
<TargetFrameworks Condition="!$([MSBuild]::IsOSPlatform('linux'))">$(TargetFrameworks);net8.0-ios;net8.0-maccatalyst<TargetFrameworks>
<TargetFrameworks Condition="$([MSBuild]::IsOSPlatform('windows'))">$(TargetFrameworks);net8.0-windows10.0.19041.0<TargetFrameworks>
```

ビルドコマンドを実行
```console
$ dotnet build
MSBuild version 17.9.8+b34f75857 for .NET
  Determining projects to restore...
  Restored /home/gari/MauiSample/MauiSample.csproj (in 1.91 min).
/home/gari/.dotnet/packs/Microsoft.Android.Sdk.Linux/34.0.95/tools/Xamarin.Android.Tooling.targets(70,5): error XA5300: The Android SDK directory could not be found. Install the Android SDK by following the instructions at: https://aka.ms/dotnet-android-install-sdk [/home/gari/MauiSample/MauiSample.csproj::TargetFramework=net8.0-android]
/home/gari/.dotnet/packs/Microsoft.Android.Sdk.Linux/34.0.95/tools/Xamarin.Android.Tooling.targets(70,5): error XA5300: To use a custom SDK path for a command line build, set the 'AndroidSdkDirectory' MSBuild property to the custom path. [/home/gari/MauiSample/MauiSample.csproj::TargetFramework=net8.0-android]

Build FAILED.

/home/gari/.dotnet/packs/Microsoft.Android.Sdk.Linux/34.0.95/tools/Xamarin.Android.Tooling.targets(70,5): error XA5300: The Android SDK directory could not be found. Install the Android SDK by following the instructions at: https://aka.ms/dotnet-android-install-sdk [/home/gari/MauiSample/MauiSample.csproj::TargetFramework=net8.0-android]
/home/gari/.dotnet/packs/Microsoft.Android.Sdk.Linux/34.0.95/tools/Xamarin.Android.Tooling.targets(70,5): error XA5300: To use a custom SDK path for a command line build, set the 'AndroidSdkDirectory' MSBuild property to the custom path. [/home/gari/MauiSample/MauiSample.csproj::TargetFramework=net8.0-android]
    0 Warning(s)
    1 Error(s)

Time Elapsed 00:01:58.10
```

error XA5300: To use a custom SDK path for a command line build, set the 'AndroidSdkDirectory' MSBuild property to the custom path. [/home/gari/MauiSample/MauiSample.csproj::TargetFramework=net8.0-android]  

とあるので、AndroidSdkDirectoryをセットしろと言われるが、まだAndroid SDKをインストールしていないので、Android SDKをインストールする

### 4-3. Android SDKのインストール

#### 4-3-1. OpenJDKのインストール

パッケージマネージャからもOpenJDKを入れられるが、  
[OpenJDK の Microsoft Build をインストールする](https://learn.microsoft.com/ja-jp/java/openjdk/install#install-on-ubuntu)  
より、Microsoftが提示している方法でOpenJDKをインストールする

```console
$ cd $HOME
$ wget https://packages.microsoft.com/config/ubuntu/`lsb_release -rs`/packages-microsoft-prod.deb -O packages-microsoft-prod.deb
$ sudo apt install ./packages-microsoft-prod.deb
$ sudo apt update
$ sudo apt install msopenjdk-17
```
(ms)openjdk-21もあるが、結論からいえばdotnetのビルドでopenjdk17を求められるので

#### 4-3-2. Android SDKのインストール

Android SDKをコマンドラインツールで入れていく  
[Android Developers](https://developer.android.com/studio?hl=ja)  
のコマンドラインツールのみのLinuxの部分からインストール先ずはコマンドラインツールをインストールする  
Android系のインストールは今回は、$HOME/Androidとする(別にどこでもいいですが)

```console
$ curl -L -o cmdlinetool.zip https://dl.google.com/android/repository/commandlinetools-linux-11076708_latest.zip?hl=ja
$ sudo apt install unzip
$ unzip cmdlinetool.zip
$ mkdir -p $HOME/Android/SDK
```

ここまでできたら、$HOME/Android/SDK配下に各パッケージを入れていく  
platformは34を入れる(これもdotnet build時に求められるパッケージバージョンがそうなので)
```console
$ cd cmdline-tool/bin
$ ./sdkmanager --sdk_root=$HOME/Android/SDK/ "cmdline-tools;latest"
$ ./sdkmanager --sdk_root=$HOME/Android/SDK/ "platform-tools" "platforms;android-34"
$ ./sdkmanager --sdk_root=$HOME/Android/SDK/ "build-tools;34.0.0"
```

### 4-4. 再ビルド

```console
$ cd $HOME/MauiSample
$ dotnet build -p:AndroidSdkDirectory=$HOME/Android/SDK
MSBuild version 17.9.8+b34f75857 for .NET
  Determining projects to restore...
  All projects are up-to-date for restore.
  MauiSample -> /home/gari/MauiSample/bin/Debug/net8.0-android/MauiSample.dll

Build succeeded.
    0 Warning(s)
    0 Error(s)

Time Elapsed 00:04:03.78
```

エラーなくビルドできました  
お疲れ様でした  

### 5. 後始末

```console
$ cd $HOME
$ rm -rf cmdline* packages-microsoft-prod.deb
```

