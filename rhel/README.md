# .NET MAUIをRHEL系でビルドする環境構築

## 1. 目的

.NET MAUIの環境をRHEL系で開発、ビルドするために環境を構築する

## 2. 準備

[Rocky Linux](https://rockylinux.org/ja/)  
で今回は行います  
バージョンは9です  

## 3. 構成

[ubuntu](../ubuntu/README.md)  
と同じなので割愛

### 3-1. 必要なパッケージのインストール

```console
# dnf install tar vim
```

最近のRHELディストリビューションはデフォルトで、tarが入ってないのよね  
テキストエディタはお好みのものを

### 3-2. .NET8のインストール

ubuntuの場合と同じ

```console
$ curl -L -O https://dotnet.microsoft.com/download/dotnet/scripts/v1/dotnet-install.sh
$ chmod +x dotnet-install.sh
$ ./dotnet-install.sh  --channel 8.0
```

~/.dotnetにPATHを通す
```console
$ cd
$ cp -p .bashrc dot.bashrc
$ echo 'export DOTNET_ROOT=$HOME/.dotnet' >> .bashrc
$ echo 'export PATH=$PATH:$DOTNET_ROOT:$DOTNET_ROOT/tools' >> .bashrc
$ exec $SHELL -l
$ dotnet --version
Process terminated. Couldn't find a valid ICU package installed on the system. Please install libicu (or icu-libs) using your package manager and try again. Alternatively you can set the configuration flag System.Globalization.Invariant to true if you want to run with no globalization support. Please see https://aka.ms/dotnet-missing-libicu for more information.
   at System.Environment.FailFast(System.String)
   at System.Globalization.GlobalizationMode+Settings..cctor()
   at System.Globalization.CultureData.CreateCultureWithInvariantData()
   at System.Globalization.CultureData.get_Invariant()
   at System.Globalization.TextInfo..cctor()
   at System.String.ToLowerInvariant()
   at System.Text.EncodingHelper.GetEncodingFromCharset()
   at System.ConsolePal.GetConsoleEncoding()
   at System.Console.get_OutputEncoding()
   at Microsoft.DotNet.Cli.AutomaticEncodingRestorer..ctor()
   at Microsoft.DotNet.Cli.Program.Main(System.String[])
Aborted (core dumped)
```
Process terminated. Couldn't find a valid ICU package installed on the system. Please install libicu (or icu-libs) using your package manager and try again.  
と言われているので、ICU関連パッケージが入ってないので入れる

```console
# dnf install libicu
$ dotnet --version
8.0.204
```
libicuだけで問題ない模様。dotnetコマンドでバージョンを確認できたらOK.

## 4. ビルド

ここもubuntuの場合と同様

```console
$ dotnet workload install maui-android
$ dotnet new maui --name MauiSample
$ cd MauiSample
$ vi MauiSample.csproj
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
```

結果やっぱりAndroidSdkDirectory言われるので、Android SDKをインストール

### 4-1. Android SDKのインストール

#### 4-1-1. OpenJDKのインストール

[CentOS 7 へのインストール (RPM)](https://learn.microsoft.com/ja-jp/java/openjdk/install#install-on-centos-7-rpm)  
よりCentOS 7系のrpmをインストールする

```console
# dnf install https://packages.microsoft.com/config/centos/7/packages-microsoft-prod.rpm
# dnf install msopenjdk-17
```

https://packages.microsoft.com/config/rhel/9/packages-microsoft-prod.rpm  
とか  
https://packages.microsoft.com/config/rocky/9/packages-microsoft-prod.rpm  
とかでも良さそうに思えるが、msopenjdkパッケージは含まれないので、CentOS 7用のものを使用する

#### 4-1-2. Android SDKのインストール

ここもubuntuの場合と同じ

```console
# dnf install unzip
$ cd $HOME
$ mkdir -p Android/SDK
$ curl -L -o cmdlinetool.zip https://dl.google.com/android/repository/commandlinetools-linux-11076708_latest.zip?hl=ja
$ unzip cmdlinetool.zip
$ cd cmdline-tools/bin
$ ./sdkmanager --sdk_root=$HOME/Android/SDK/ "cmdline-tools;latest"
$ ./sdkmanager --sdk_root=$HOME/Android/SDK/ "platform-tools" "platforms;android-34"
$ ./sdkmanager --sdk_root=$HOME/Android/SDK/ "build-tools;34.0.0"
```

### 4-2. 再ビルド

```console
dotnet build -p:AndroidSdkDirectory=$HOME/Android/SDK
MSBuild version 17.9.8+b34f75857 for .NET
  Determining projects to restore...
  All projects are up-to-date for restore.
  MauiSample -> /home/gari/dotnet-maui-on-linux-develop/rhel/MauiSample/bin/Debug/net8.0-android/MauiSample.dll

Build succeeded.
    0 Warning(s)
    0 Error(s)

Time Elapsed 00:04:05.41
```

エラーなくビルド完了

## 5. 後始末

```console
$ cd $HOME
$ rm -rf cmdline* packages-microsoft-prod.rpm
```
