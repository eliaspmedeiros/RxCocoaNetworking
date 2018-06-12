# RxCocoaNetworking

An extremely lightweight networking framework built on top of `RxCocoa`, tailored for testing with `RxTest` and inspired by `Moya`.

[![Swift 4.1+](https://img.shields.io/badge/Swift-4.1-orange.svg?style=flat)](https://swift.org) [![codecov.io](http://codecov.io/github/gobetti/RxCocoaNetworking/coverage.svg?branch=master)](http://codecov.io/github/gobetti/RxCocoaNetworking?branch=master) [![Platforms](https://img.shields.io/cocoapods/p/RxCocoaNetworking.svg)](https://cocoapods.org/pods/RxCocoaNetworking)

[![CocoaPods compatible](https://img.shields.io/cocoapods/v/RxCocoaNetworking.svg)](https://cocoapods.org/pods/RxCocoaNetworking)
[![Carthage compatible](https://img.shields.io/badge/Carthage-compatible-4BC51D.svg?style=flat)](https://github.com/Carthage/Carthage)
[![Swift Package Manager](https://img.shields.io/badge/Swift%20Package%20Manager-compatible-brightgreen.svg)](https://github.com/apple/swift-package-manager)

- [Requirements](#requirements)
- [Usage](#usage)
- [Installation](#installation)
- [Contributing](#contributing)
- [License](#license)

While [Moya](https://github.com/Moya/Moya) is built on top of [Alamofire](https://github.com/Alamofire/Alamofire) and provides an Rx extension with `Observable` signatures, **RxCocoaNetworking** is built on top of [RxCocoa](https://github.com/ReactiveX/RxSwift), which already provides extensions to `NSURLSession` with `Observable` signatures. **RxCocoaNetworking** was made for you if:

- ✅ you love how `Moya` keeps your network layer clean, readable and testable;
- ✅ the power of Alamofire+Moya is too much for your needs;
- ✅ your project already depends on `RxCocoa`.

... and extra points if:

- ✅ you'd like to use `RxTest` for testing `delayed` stubs;
- ✅ you don't like to implement `sampleData` in your production target;
- ✅ you feel you could have less code for the simple APIs you use.

Note though that this is **by no means a full replacement** to Alamofire+Moya. For example, you may miss features which will only ever be supported if **RxCocoaNetworking** is kept extremely lightweight, e.g.:

- ❌ anything other than `Data` on successful responses;
- ❌ file upload;
- ❌ plugins (interceptors).

... in which case Alamofire+Moya would be a better bet.

This framework was made possible thanks to the network request handling already embedded in `RxCocoa`, together with Swift 4.1's conditional conformance (see [ReactiveURLSessionProtocol](https://github.com/gobetti/RxCocoaNetworking/blob/master/Sources/Core/ReactiveURLSessionProtocol.swift)).

## Requirements

- 📱 iOS 9.0+ / Mac OS X 10.11+ / tvOS 10.0+ / watchOS 3.0+
- 🛠 Xcode 9.3+
- ✈️ Swift 4.1+
- ⚠️ RxCocoa
- 🔥 Does not require Alamofire

## Usage

If you're already used to [Moya](https://github.com/Moya/Moya), the good news is that **RxCocoaNetworking** (intentionally) has a very similar architecture! All you need is to create a structure to represent your API - an `enum` is recommended - and have it implement one of the [`TargetType`](https://github.com/gobetti/RxCocoaNetworking/blob/master/Sources/Core/TargetType.swift) protocols. Requests to your API are managed by a [`Provider`](https://github.com/gobetti/RxCocoaNetworking/blob/master/Sources/Core/Provider.swift) which is typed to your concrete `TargetType`.

Both if you're used to Moya or not, the other good news is that you can base off the example [`ExampleAPI`](https://github.com/gobetti/RxCocoaNetworking/blob/master/Tests/Example/ExampleAPI.swift) and its [spec](https://github.com/gobetti/RxCocoaNetworking/blob/master/Tests/ExampleAPISpec.swift).

<details>
  <summary><strong>Summarized `ExampleAPI`</strong></summary><p>
  
```swift
enum ExampleAPI {
  // Endpoints as cases:
  case rate(movieID: String, rating: Float)
  case reviews(movieID: String, page: Int)
}

extension ExampleAPI: ProductionTargetType {
  // Your API's base URL is usually what determines an API enum.
  var baseURL: URL { return URL(string: "...")! }
  
  var path: String {
    switch self {
    case .rate(let movieID, _):
      return "/movie/\(movieID)/rating"
    case .reviews(let movieID, _):
      return "/movie/\(movieID)/reviews"
    }
  }
  
  var task: Task {
    // Specify GET/POST/etc., body and query parameters:
    switch self {
    case .rate(_, let rating):
      return Task(method: .post, dictionaryBody: ["value": rating])
    case .reviews(_, let page):
      return Task(parameters: parameters)
    }
  }
  
  var headers: [String : String]? { return nil }
}

extension ExampleAPI: TargetType {
  var sampleData: Data {
    ...
  }
}
```
  </p></details>

### Regular network requests (no stubbing):
```swift
let provider = Provider<ExampleAPI>()
```
The default `Provider` parameters is most often what you'll use in production code.

### Immediately stubbed network responses:
```swift
let provider = Provider<ExampleAPI>(stubBehavior: .immediate(stub: .default))
```
`stub: .default` means the `sampleData` from your API will be used. Other `Stub` types allow you to specify different responses inline.

**Note:** if you implement `ProductionTargetType`, then `stub: .default` is not available, because `sampleData` is not required by that protocol 👌 You can always implement `TargetType` separately, for example, in the Tests target.

### `RxTest`-testable delayed stubbed network responses:
```swift
let testScheduler = TestScheduler(initialClock: 0)
let provider = Provider<ExampleAPI>(stubBehavior: .delayed(time: 3,
                                                           stub: .error(SomeError.anError)),
                                    scheduler: testScheduler)
```
An `error` will be emitted 3 virtual time units after the subscription occurs.

## Installation

### Dependency Managers
<details>
  <summary><strong>CocoaPods</strong></summary>

[CocoaPods](http://cocoapods.org) is a dependency manager for Cocoa projects. You can install it with the following command:

```bash
$ gem install cocoapods
```

To integrate RxCocoaNetworking into your Xcode project using CocoaPods, specify it in your `Podfile`:

```ruby
source 'https://github.com/CocoaPods/Specs.git'
platform :ios, '9.0'
use_frameworks!

pod 'RxCocoaNetworking', '~> 0.2.0'
```

Then, run the following command:

```bash
$ pod install
```

</details>

<details>
  <summary><strong>Carthage</strong></summary>

[Carthage](https://github.com/Carthage/Carthage) is a decentralized dependency manager that automates the process of adding frameworks to your Cocoa application.

You can install Carthage with [Homebrew](http://brew.sh/) using the following command:

```bash
$ brew update
$ brew install carthage
```

To integrate RxCocoaNetworking into your Xcode project using Carthage, specify it in your `Cartfile`:

```ogdl
github "gobetti/RxCocoaNetworking" ~> 0.2.0
```

</details>

<details>
  <summary><strong>Swift Package Manager</strong></summary>

To use RxCocoaNetworking as a [Swift Package Manager](https://swift.org/package-manager/) package just add the following in your Package.swift file.

```swift
// swift-tools-version:4.1
import PackageDescription

let package = Package(
    name: "HelloRxCocoaNetworking",
    dependencies: [
        .package(url: "https://github.com/gobetti/RxCocoaNetworking.git", .upToNextMajor(from: "0.2.0"))
    ],
    targets: [
        .target(name: "HelloRxCocoaNetworking", dependencies: ["RxCocoaNetworking"])
    ]
)
```
</details>

### Manually

If you prefer not to use either of the aforementioned dependency managers, you can integrate RxCocoaNetworking into your project manually.

<details>
  <summary><strong>Git Submodules</strong></summary><p>

- Open up Terminal, `cd` into your top-level project directory, and run the following command "if" your project is not initialized as a git repository:

```bash
$ git init
```

- Add RxCocoaNetworking as a git [submodule](http://git-scm.com/docs/git-submodule) by running the following command:

```bash
$ git submodule add https://github.com/gobetti/RxCocoaNetworking.git
$ git submodule update --init --recursive
```

- Open the new `RxCocoaNetworking` folder, and drag the `RxCocoaNetworking.xcodeproj` into the Project Navigator of your application's Xcode project.

    > It should appear nested underneath your application's blue project icon. Whether it is above or below all the other Xcode groups does not matter.

- Select the `RxCocoaNetworking.xcodeproj` in the Project Navigator and verify the deployment target matches that of your application target.
- Next, select your application project in the Project Navigator (blue project icon) to navigate to the target configuration window and select the application target under the "Targets" heading in the sidebar.
- In the tab bar at the top of that window, open the "General" panel.
- Click on the `+` button under the "Embedded Binaries" section.
- You will see two different `RxCocoaNetworking.xcodeproj` folders each with two different versions of the `RxCocoaNetworking.framework` nested inside a `Products` folder.

    > It does not matter which `Products` folder you choose from.

- Select the `RxCocoaNetworking.framework`.

- And that's it!

> The `RxCocoaNetworking.framework` is automagically added as a target dependency, linked framework and embedded framework in a copy files build phase which is all you need to build on the simulator and a device.

</p></details>

<details>
  <summary><strong>Embedded Binaries</strong></summary><p>

- Download the latest release from https://github.com/gobetti/RxCocoaNetworking/releases
- Next, select your application project in the Project Navigator (blue project icon) to navigate to the target configuration window and select the application target under the "Targets" heading in the sidebar.
- In the tab bar at the top of that window, open the "General" panel.
- Click on the `+` button under the "Embedded Binaries" section.
- Add the downloaded `RxCocoaNetworking.framework`.
- And that's it!

</p></details>

## Contributing

Issues and pull requests are welcome!

## Author

Marcelo Gobetti [@mwgobetti](https://twitter.com/mwgobetti)

## License

RxCocoaNetworking is released under the MIT license. See [LICENSE](https://github.com/gobetti/RxCocoaNetworking/blob/master/LICENSE) for details.
