---
title: Xamarin アプリケーションのビルドとテスト
intro: GitHub Actions で継続的インテグレーション (CI) ワークフローを作成して、Xamarin アプリケーションをビルドおよびテストできます。
redirect_from:
  - /actions/guides/building-and-testing-xamarin-applications
versions:
  fpt: '*'
  ghes: '*'
  ghae: '*'
  ghec: '*'
type: tutorial
topics:
  - CI
  - Xamarin
  - Xamarin.iOS
  - Xamarin.Android
  - Android
  - iOS
shortTitle: Build & test Xamarin apps
---

{% data reusables.actions.enterprise-beta %}
{% data reusables.actions.enterprise-github-hosted-runners %}

## はじめに

このガイドでは、Xamarin プロジェクトの継続的インテグレーション (CI) を実行するワークフローを作成する方法を説明します。 作成するワークフローによって、Pull Requestに対するコミットがデフォルトブランチに対してビルドあるいはテストの失敗を引き起こしたことを見ることができるようになります。このアプローチは、コードが常に健全であることを保証するための役に立ちます。

For a full list of available Xamarin SDK versions on the {% data variables.product.prodname_actions %}-hosted macOS runners, see the documentation:

* [macOS 10.15](https://github.com/actions/runner-images/blob/main/images/macos/macos-10.15-Readme.md#xamarin-bundles)
* [macOS 11](https://github.com/actions/runner-images/blob/main/images/macos/macos-11-Readme.md#xamarin-bundles)

{% data reusables.actions.macos-runner-preview %}

## 必要な環境

Xamarin、.NET Core SDK、YAML、ワークフロー設定オプション、およびワークフローファイルの作成方法の基本を理解しておくことをお勧めします。 詳しい情報については、以下を参照してください。

- [{% data variables.product.prodname_actions %}のワークフロー構文](/actions/automating-your-workflow-with-github-actions/workflow-syntax-for-github-actions)
- 「[.NET を使ってみる](https://dotnet.microsoft.com/learn)」
- "[Xamarin について学ぶ](https://dotnet.microsoft.com/learn/xamarin)"

## Building Xamarin.iOS apps

The example below demonstrates how to change the default Xamarin SDK versions and build a Xamarin.iOS application.

```yaml
name: Build Xamarin.iOS app

on: [push]

jobs:
  build:

    runs-on: macos-latest

    steps:
    - uses: {% data reusables.actions.action-checkout %}
    - name: Set default Xamarin SDK versions
      run: |
        $VM_ASSETS/select-xamarin-sdk-v2.sh --mono=6.12 --ios=14.10

    - name: Set default Xcode 12.3
      run: |
        XCODE_ROOT=/Applications/Xcode_12.3.0.app
        echo "MD_APPLE_SDK_ROOT=$XCODE_ROOT" >> $GITHUB_ENV
        sudo xcode-select -s $XCODE_ROOT

    - name: Setup .NET Core SDK 5.0.x
      uses: {% data reusables.actions.action-setup-dotnet %}
      with:
        dotnet-version: '5.0.x'

    - name: Install dependencies
      run: nuget restore <sln_file_path>

    - name: Build
      run: msbuild <csproj_file_path> /p:Configuration=Debug /p:Platform=iPhoneSimulator /t:Rebuild
```

## Building Xamarin.Android apps

The example below demonstrates how to change default Xamarin SDK versions and build a Xamarin.Android application.

```yaml
name: Build Xamarin.Android app

on: [push]

jobs:
  build:

    runs-on: macos-latest

    steps:
    - uses: {% data reusables.actions.action-checkout %}
    - name: Set default Xamarin SDK versions
      run: |
        $VM_ASSETS/select-xamarin-sdk-v2.sh --mono=6.10 --android=10.2

    - name: Setup .NET Core SDK 5.0.x
      uses: {% data reusables.actions.action-setup-dotnet %}
      with:
        dotnet-version: '5.0.x'

    - name: Install dependencies
      run: nuget restore <sln_file_path>

    - name: Build
      run: msbuild <csproj_file_path> /t:PackageForAndroid /p:Configuration=Debug
```

## .NETのバージョンの指定

{% data variables.product.prodname_dotcom %}ホストランナーにプリインストールされたバージョンの.NET Core SDKを使うには、`setup-dotnet`アクションを使ってください。 このアクションは各ランナーのツールキャッシュから指定されたバージョンの.NETを見つけ、必要なバイナリを`PATH`に追加します。 これらの変更は、ジョブの残りの部分で保持されます。

`setup-dotnet`アクションは、{% data variables.product.prodname_actions %}で.NETを使うための推奨される方法です。これは、それによって様々なランナーや様々なバージョンの.NETに渡って一貫した振る舞いが保証されるためです。 セルフホストランナーを使っている場合は、.NETをインストールして`PATH`に追加しなければなりません。 詳しい情報については[`setup-dotnet`](https://github.com/marketplace/actions/setup-net-core-sdk)アクションを参照してください。
