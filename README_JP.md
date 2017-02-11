# ReSwift

[![Build Status](https://img.shields.io/travis/ReSwift/ReSwift/master.svg?style=flat-square)](https://travis-ci.org/ReSwift/ReSwift) [![Code coverage status](https://img.shields.io/codecov/c/github/ReSwift/ReSwift.svg?style=flat-square)](http://codecov.io/github/ReSwift/ReSwift) [![CocoaPods Compatible](https://img.shields.io/cocoapods/v/ReSwift.svg?style=flat-square)](https://cocoapods.org/pods/ReSwift) [![Platform support](https://img.shields.io/badge/platform-ios%20%7C%20osx%20%7C%20tvos%20%7C%20watchos-lightgrey.svg?style=flat-square)](https://github.com/ReSwift/ReSwift/blob/master/LICENSE.md) [![License MIT](https://img.shields.io/badge/license-MIT-blue.svg?style=flat-square)](https://github.com/ReSwift/ReSwift/blob/master/LICENSE.md)

**サポートしているSwiftバージョン:**: Swift 3.0.1
Swift 2.2については[Release 2.0.0](https://github.com/ReSwift/ReSwift/releases/tag/2.0.0)かもしくはそれ以前のバージョンがサポートしています。

# Introduction

ReSwiftは[Redux](https://github.com/reactjs/redux)ライクな単方向データフローアーキテクチャをSwiftで実装したものです。ReSwift はアプリを三つの原則に基いて要素に分割します:

- **State**: ReSwiftを使ったアプリではアプリの状態を明示的にデータ構造に保存します。これによってコードが複雑になることを避け、デバッグを容易にし、さらにより多くの恩恵を受けることが出来ます。
- **Views**: ReSwiftをつかったアプリではViewはアプリの状態が変わった時に更新されます。つまりViewはアプリの状態をシンプルに表すものになります。
- **State Changes**: ReSwiftを使ったアプリではその状態はActionを通じてしか変えることができません。Actionは状態の変化を記述した小さなデータの塊です。厳しく状態の変化を制限することによって、アプリはより理解しやすく複数で開発することも容易になります。

ReSwiftライブラリは小さいためユーザーがコードを読み、全てを理解することが可能です。[コントリビュート](#contributing)も歓迎しています。

ReSwift is quickly growing beyond the core library, providing experimental extensions for routing and time traveling through past app states!

Excited? So are we 🎉

Check out our [public gitter chat!](https://gitter.im/ReSwift/public)

# Table of Contents

- [About ReSwift](#about-reswift)
- [Why ReSwift?](#why-reswift)
- [Getting Started Guide](#getting-started-guide)
- [Installation](#installation)
- [Testing](#testing)
- [Checking Out Source Code](#checking-out-source-code)
- [Demo](#demo)
- [Extensions](#extensions)
- [Example Projects](#example-projects)
- [Contributing](#contributing)
- [Credits](#credits)
- [Get in touch](#get-in-touch)

# About ReSwift

ReSwiftはいくつかの原則に基いています:
- **Store** はアプリ全体の状態を一つのデータ構造の形で保存しています。この状態はStoreに対するActionの適用でしか変えることができません。Store内で状態の変化があれば必ず全てのObserver（下図ではView）へ通知します。
- **Actions**は宣言的に状態の変化を記述したものです。Actionには処理は書かれず、Storeとこの次に述べるReducerによって利用されます。ReducerはそれぞれのActionに対して違う状態変化を実装することでActionを扱います。
- **Reducers**は純粋な現在のActionとアプリの状態に基いて新しいアプリの状態をつくる関数を提供します。

![](Docs/img/reswift_concept.png)

とても単純な例として、足し引きのみできるカウンターを取ると、このアプリの状態は次のように定義できます:

```swift
struct AppState: StateType {
    var counter: Int = 0
}
```

カウンターの足すと引くの二つのActionも定義できます。もっと複雑なActionについては[チュートリアル](http://reswift.github.io/ReSwift/master/getting-started-guide.html)をみてください。この例での単純なActionは空のActionを継承したstructにより定義できます:

```swift
struct CounterActionIncrease: Action {}
struct CounterActionDecrease: Action {}
```

Reducerはこの二つの異なるActionに応える必要があり、これはActionの型で条件分岐することで実現できます:

```swift
struct CounterReducer: Reducer {

    func handleAction(action: Action, state: AppState?) -> AppState {
        var state = state ?? AppState()

        switch action {
        case _ as CounterActionIncrease:
            state.counter += 1
        case _ as CounterActionDecrease:
            state.counter -= 1
        default:
            break
        }

        return state
    }

}
```
アプリの状態が予想できるように、Reducerは副作用が無く現在のアプリの状態とActionを受け取ったら決まった新しいアプリの状態を返せることが重要です。

To maintain our state and delegate the actions to the reducers, we need a store. Let's call it `mainStore` and define it as a global constant, for example in the app delegate file:

```swift
let mainStore = Store<AppState>(
	reducer: CounterReducer(),
	state: nil
)

@UIApplicationMain
class AppDelegate: UIResponder, UIApplicationDelegate {
	[...]
}
```


最後にViewの部分、今回の場合ViewControllerではStoreの更新をsubscribe(監視)してアプリの状態が変わる時は必ずActionを送信する必要があります:

```swift
class CounterViewController: UIViewController, StoreSubscriber {

    @IBOutlet var counterLabel: UILabel!

    override func viewWillAppear(animated: Bool) {
        mainStore.subscribe(self)
    }

    override func viewWillDisappear(animated: Bool) {
        mainStore.unsubscribe(self)
    }

    func newState(state: AppState) {
        counterLabel.text = "\(state.counter)"
    }

    @IBAction func increaseButtonTapped(sender: UIButton) {
        mainStore.dispatch(
            CounterActionIncrease()
        )
    }

    @IBAction func decreaseButtonTapped(sender: UIButton) {
        mainStore.dispatch(
            CounterActionDecrease()
        )
    }

}
```

`newState`関数は`Store`から新しいアプリの状態が利用可能になると必ず呼ばれるもので、ここに最新のアプリの状態をViewに反映する処理を書かなければいけません。

ボタンがタップされるとStoreとそのReducerで扱われるActionを送り、結果として新しいアプリの状態を得ます。

これはとても基本的な例でReSwiftの一部の機能しか紹介できていません。チュートリアルを読んでどうやってアプリ全体にこのアーキテクチャを適用するのかを学んでください。この例の完全な実装例は[CounterExample](https://github.com/ReSwift/CounterExample)にあります。

[You can also watch this talk on the motivation behind ReSwift](https://realm.io/news/benji-encz-unidirectional-data-flow-swift/).

# Why ReSwift?

Model-View-Controller(MVC)モデルは総体的なアプリケーションアーキテクチャではありません。典型的なCocoaアプリはMVCが状態管理に他の解決法を要請しないがためにControllerがとても複雑になってしまい、これはアプリ開発においてもっとも複雑な問題となっています。

MVCでつくられたアプリは状態の管理や伝達周りの複雑さによって破綻してしまいます。アプリの中で情報を関連するViewに渡して状態を最新のものに保っておくためにはコールバックやデリゲーション、KVOや通知を使わなければなりません。

このやり方だと多くの手動の操作が必要で、結果エラーが出る傾向があり上手く調整できず複雑なコードになってしまいます。

また依存関係がViewControllerの中に隠れてしまっているので一目でみて理解することが難しいコードになってしまいます。そして最終的にそれぞれの開発者がそれぞれ好きな状態遷移の処理を書いてしまって多くの場合一貫性の無いコードになってしまいます。この問題はスタイルガイドをつくりコードレビューすることで解決することはできますが、自動でこれらのガイドラインに従わせることはできません。

ReSwiftはこれらの問題をアプリケーションの書き方に強い制約を設けることで解決しようとしています。アプリケーションの状態をデータ構造、Action、そしてReducerで検査することでプログラマーの生むエラーの余地を減らしより簡単に理解できるアプリケーションになるようにしているのです。

この構造はコードを良くするだけでなくさらなる利点もあります:

- Stores, Reducers, Actions and extensions such as ReSwift Router are entirely platform independent - you can easily use the same business logic and share it between apps for multiple platforms (iOS, tvOS, etc.)
- Want to collaborate with a co-worker on fixing an app crash? Use [ReSwift Recorder](https://github.com/ReSwift/ReSwift-Recorder) to record the actions that lead up to the crash and send them the JSON file so that they can replay the actions and reproduce the issue right away.
- Maybe recorded actions can be used to build UI and integration tests?

The ReSwift tooling is still in a very early stage, but aforementioned prospects excite me and hopefully others in the community as well!

# Getting Started Guide

[A Getting Started Guide that describes the core components of apps built with ReSwift lives here](http://reswift.github.io/ReSwift/master/getting-started-guide.html). It will be expanded in the next few weeks. To get an understanding of the core principles we recommend reading the brilliant [redux documentation](http://redux.js.org/).

# Installation

## CocoaPods

You can install ReSwift via CocoaPods by adding it to your `Podfile`:
```
use_frameworks!

source 'https://github.com/CocoaPods/Specs.git'
platform :ios, '8.0'

pod 'ReSwift'
```

And run `pod install`.

## Carthage

You can install ReSwift via [Carthage](https://github.com/Carthage/Carthage) by adding the following line to your `Cartfile`:

```
github "ReSwift/ReSwift"
```

## Swift Package Manager

You can install ReSwift via [Swift Package Manager](https://swift.org/package-manager/) by adding the following line to your `Package.swift`:

```swift
import PackageDescription

let package = Package(
    [...]
    dependencies: [
        .Package(url: "https://github.com/ReSwift/ReSwift.git", majorVersion: XYZ)
    ]
)
```

# Checking out Source Code

ReSwift no longer has any carthage dependencies for development. Just checkout the project and run.

# Demo

Using this library you can implement apps that have an explicit, reproducible state, allowing you, among many other things, to replay and rewind the app state, as shown below:

![](Docs/img/timetravel.gif)

# Extensions

This repository contains the core component for ReSwift, the following extensions are available:

- [ReSwift-Router](https://github.com/ReSwift/ReSwift-Router): Provides a ReSwift compatible Router that allows declarative routing in iOS applications
- [ReSwift-Recorder](https://github.com/ReSwift/ReSwift-Recorder): Provides a `Store` implementation that records all `Action`s and allows for hot-reloading and time travel

# Example Projects

- [CounterExample](https://github.com/ReSwift/CounterExample): A very simple counter app implemented with ReSwift.
- [CounterExample-Navigation-TimeTravel](https://github.com/ReSwift/CounterExample-Navigation-TimeTravel): This example builds on the simple CounterExample app, adding time travel with [ReSwiftRecorder](https://github.com/ReSwift/ReSwift-Recorder) and routing with [ReSwiftRouter](https://github.com/ReSwift/ReSwift-Router).
- [GitHubBrowserExample](https://github.com/ReSwift/GitHubBrowserExample): A real world example, involving authentication, network requests and navigation. Still WIP but should be the best resource for starting to adapt `ReSwift` in your own app.
- [Meet](https://github.com/Ben-G/Meet): A real world application being built with ReSwift - currently still very early on. It is not up to date with the latest version of ReSwift, but is the best project for demonstrating time travel.

##Production Apps with Open Source Code

- [Product Hunt for OS X](https://github.com/producthunt/producthunt-osx) Official Product Hunt client for OS X.

# Contributing

There's still a lot of work to do here! We would love to see you involved! You can find all the details on how to get started in the [Contributing Guide](/CONTRIBUTING.md).

# Credits

- Thanks a lot to [Dan Abramov](https://github.com/gaearon) for building [Redux](https://github.com/reactjs/redux) - all ideas in here and many implementation details were provided by his library.

# Get in touch

If you have any questions, you can find the core team on twitter:

- [@benjaminencz](https://twitter.com/benjaminencz)
- [@karlbowden](https://twitter.com/karlbowden)
- [@ARendtslev](https://twitter.com/ARendtslev)
- [@ctietze](https://twitter.com/ctietze)

We also have a [public gitter chat!](https://gitter.im/ReSwift/public)
