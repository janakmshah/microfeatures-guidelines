![Logo](assets/logo.png)
# µFeatures Guidelines

[![Say Thanks!](https://img.shields.io/badge/Say%20Thanks-!-1EAEDB.svg)](https://saythanks.io/to/pepibumur)

## Index 📝
- [What](#what-)
- [Context](#context-)
- [Motivation](#motivation-)
- [Before reading](#before-reading-)
- [Core Principle](#core-principle-)
- [What is a µFeature](#what-is-a-µfeature-)
- [Why a µFeature](#why-a-µfeature-)
- [Types of µFeatures](#types-of-µfeatures-)
- [Hooking µFeatures](#hooking-µfeatures-)
- [Layers](#layers-)
- [Dependencies](#dependencies-)
- [Cross-platform µFeatures](#cross-platform-µfeatures-)
- [Shortcomings](#shortcomings-)
- [FAQ](#faq-)

## What 🤔

uFeatures is an architectural approach to structure iOS applications to enable scalability, optimizing build and testing cycles, and ensuring good practices in your team. Its core idea is to build your apps by building independent features that are interconnected using clear and concise APIS.

This guidelines introduces the principles of the architecture, helping you identify and organize your application features in different layers. It also introduces tips, tools and advices if you decide for this architecture.

> The name uFeatures *(Microfeatures)* comes from the Microservices architecture, where different "backend features" run as different services with defined APIS to enable communication between them.

## Context 🥑

Apps are made of features. Typically these features are part of the same module, or target where the whole application is defined. The natural inclination in the team is to continue building features and its tests in the same targets. As a result, the application and its tests target grows in complexity which manifests in bugs, bad compilation times, and team performance. What seemed to be a good architecture, doesn't work out that well in large codebases or teams.

**This is frequently a big source of frustration when it comes to work on those projects. The time we spend goes into compiling rather than building and experimenting with the platform.**

![Xcode compiling](assets/compiling.gif)

## Motivation 🍎

The µFeatures's main motivation is to support the scalability of large iOS codebases leveraging platform features and tools. There are other solutions out there that could be also be considered to overcome those issues. A very popular one nowadays is [React Native](https://facebook.github.io/react-native/) that leverages the Javascript dynamism to offer developers a pleasant experience working in the code base, but at the same time a native experience from the user point of view.

**We believe that the usage of native tools and technologies can be optimized to overcome scalability challenges that sooner or later show up in our projects**

## Before reading 🍒

- Don't expect this to be a silver-bullet solution to your problems. You should take the core ideas, process them, and apply the principles to your projects.
- Each project is different so are the needs. With the ideas in the guidelines, and your needs, you should figure out what might work out for you.
- Since everything this architecture depends on is evolving *(tools, languages, concepts)*, the guidelines might get outdated very quickly. If that happens, don't hesitate to open a PR and contribute with keeping this guidelines up to date.
- It can very tempting to scale your app architecture before it actually needs it. If your app needs it, you'll notice it, and only at that point, you should consider start tackling the issue. 

## Example app 🍷
If you are eager to see an example of an app with a µFeatures architecture you can check out the [example app](https://github.com/pepibumur/microfeatures-example). It's structured following the principles described in the following sections.

## Core principle 🐝

Developers should be able to **build, test and try** their features fast, with independence of the main app. 

## What is a µFeature 📱
A µFeature represents an application feature and is a combination of the following 4 targets *(referring with target to a Xcode target)*:

- **Source:** Contains the feature source code *(Swift, Objective-C, C++, React Native...)* and its resources *(images, fonts, storyboards, xibs)*.
- **Tests:** Contains the feature unit and integration tests.
- **Testing:** Provides testing data that can be used for the tests and from the example app. It also provide mocks for uFeature classes and protocols that can be used by other features as we'll see later.
- **Example:** Contains an example app that developers can use to try out the feature under certain conditions *(different languages, screen sizes, settings)*.

The diagram below shows the 4 targets and the dependencies between them:

<p align="center">
    <img src="/assets/diagram.png" width="200" max-width="50%" alt="µFeature components" />
</p>

## Why a µFeature 🐛

#### Clear and concise APIs
When all the app source code lives in the same target is very easy to build implicit dependencies in code, and end up with the so well-known spaguetti code. Everything is strongly coupled, the state is sometimes unpredictable, and introducing new changes become a nightmare. When we define features in independent targets we need to design public APIs as part of our feature implementation. We need to decide what should be public, how our feature should be consumed, what should remain private. We have more control over how we want our feature *"clients"* to use the feature and we can enforce good practises by designing safe APIs.

#### Small modules
[Divide and conquer](https://en.wikipedia.org/wiki/Divide_and_conquer). Working in small modules allows you to have more focus and test and try the feature in isolation. Moreover, development cycles are much faster since we have a more selective compilation, compiling only the components that are necessary to get our feature working. The compilation of the whole app is only necessary at the very end of our work, when we need to integrate the feature into the app.

#### Reusability
Reusing code across apps and other products like extensions is encouraged using frameworks or libraries. By building µFeatures reusing them is pretty straightforward. We can build an iMessage extension, a Today Extension, or a watchOS application by just combining existing µFeatures and adding *(when necessary)* platform-specific UI layers.

## Types of µFeatures 🐤

### Foundation
Foundation µFeatures contain foundational tools *(wrappers, extensions, ...)* that are combined to build other µFeatures. Thus other µFeatures have access to the foundation ones. Some examples of foundations µFeatures are:

- **µUI:** Provides custom views, UIKit extensions, fonts, and colors that are used to build user-facing layouts.
- **µTesting:** Facilitates testing by providing XCTest extensions as well as custom assertions.
- **µCore:** It can be seen as the `Foundation` of your app, providing tools such as analytics reporter, logger, API client or a storage class.

In practice, foundation µFeatures expose **Interfaces (Structs, Classes, Enums)** and **extensions** of platform frameworks such as `XCTest`, `Foundation` or `UIKit`. 

> :warning: Note: Foundation µFeatures shouldn't expose static instances that are globally accessed. As we'll see later, it's up to the app to control the lifecycle of those foundation dependencies, and pass them to other µFeatures using dependency injection.

### Product
Product µFeatures contain features that the user can feel and interact with. They are built by combining foundation µFeatures. Some examples of product µFeatures are:

- **µSearch:** Contains your product search feature that allows users searching content on the platform.
- **µPayments:** Contains the business logic to handle payment flows and upsell screens to upgrade users to premium plans.
- **µHome:** Contains the product home screen with the most recent platform content.

> Product µFeatures usually represent your product's features.

In practice, product µFeatures expose **views** and **services**. In the following sections we'll see how the app target uses those views and services to build up the app.

## Hooking µFeatures 🦊

As we mentioned earlier, µFeatures **don't expose instances** and it's the app responsibility to create instances and use them. How we instantiate and hook µFeatures depends on the type of µFeature.

### Services

 Apps usually have services or utils whose state is tied to the application lifecycle. Those instances are global and the majority of the features will need to access them. 

```swift
// Services.swift in the main application
import uCore
import uPlayback

class Services {
    static let playback = PlaybackService() // From uPlayback
    static let client = Client(baseUrl: "https://api.shakira.io") // From uCore
    static let analytics = Analytics(firebaseKey: "xxx") // From uCore
}
```

In the example above, `Services.swift` works a a static container, initializing all the services and tools with their initial state. Some of these services might need to know about the application lifecycle. We could subscribe to those notifications internally but we'd be coupling the service to the `NotificationCenter` and the platform-specific lifecycle notifications. What we could do instead is explicitly notifying them about the lifecycle events from the app delegate.

```swift
// AppDelegate.swift
@UIApplicationMain
class AppDelegate: UIResponder, UIApplicationDelegate {

    func applicationDidBecomeActive(_ application: UIApplication) {
        Services.playback.restoreState()    
    }
    
}
```

### Views/ViewControllers

On the other side, product µFeatures can also expose views and view controllers. These views usually encapsulate the logic to update themselves according to internal state changes, and react to user actions, turning those actions into state updates *(e.g. synchronizing data with the API)*.

```swift
// Home.swift in uHome
import UIKit
import uCore

public class Home {
    let client: Client
    public init(client: Client) {
        self.client = client
    }
    public func makeViewController(delegate: HomeDelegate) -> UIViewController {
        return HomeViewController(client: client, delegate: delegate)
    }
}
```
The example above shows how the home µFeature looks like. It gets initialized with its dependencies and expose a method that instantiates and returns a view controller to be used from the app. Notice that the method returns a `UIViewController` instead of a `HomeViewController`. By doing that we abstract the app from any implementation detail.

#### Delegating navigation
You might have noticed that we pass a delegate when we instantiate the view controller. The delegate responds to actions that trigger a navigation to a different µFeature. It's up to the app to define the navigation between different µFeatures. A pattern that works very well here is the [Coordinator Pattern](https://vimeo.com/144116310) that allows you represent your navigation as a tree of coordinators. These coordinators would be in the app, responding to µFeatures actions, and triggering the navigation to other coordinators.

Delegating the navigation to the app gives us the flexibility to change the navigation based on the product where we are consuming the µFeature from. Let's take an hypothetical search µFeature that exposes a search view controller. When use that view controller from the app, we want to navigate to another µFeature when the user taps in one of the search results. However, if we use that view controller from an iMessage extension, we want the action to be different, and instead, share the search result with one of your contacts.

## Layers 🐬

We talked about µFeatures, types, and how to use them from the app, but we missed  out a very important point, how to organize them to prevent ending up with a messy dependency graph or eventually with circular dependencies impossible to resolve for the compiler.

It's recommended to organize the µFeatures in three layers below the product layer as shown in the image below.

<p align="center">
    <img src="/assets/layers.png" width="300" max-width="50%" alt="µFeatures layers" />
</p>

- **Product µFeatures:** Contains all the product µFeatures.
- **Dependency inversion:** Contains the product µFeatures public interfaces.
- **Foundation µFeatures:** Contains all the foundation µFeatures.

Product µFeatures don't depend on each other, instead we use the **dependency inversion principle** to expose their interfaces in a layer underneath them. There are a couple of advantages that support it:

- It delegates to the app the decission about how different features are connected. We could decide in a product-basis.
- It decouples the targets in the same layer, removing the need to compile all the dependencie when we try to compile a single product µFeature *(faster compilation)*.

## Dependencies ⚙️

As soon as you start building µFeatures you'll realize that most of the features need dependencies that are injected from the app. We could inject those dependencies in the constructor but we'd end up with constructors with a long list of parameters being passed. Instead, we could leverage protocols to represent the µFeatures dependencies and pass them easily *(credits to [@andreacipriani](https://github.com/andreacipriani) for coming up with this approach)*:

```swift
public protocol BaseDependencies {
    func makeClient() -> Client
    func makeLogger() -> Logger
}
```

A protocol defines the base dependencies that are the most common dependencies across all the features. Dependencies are exposed through methods that return the dependency as a return parameter of those methods.

```swift
class AppDependencies: BaseDependencies {
    func makeClient() -> Client {
        return Services.client
    }
    func makeLogger() -> Logger {
        return Services.logger
    }
}
```

From the app we conform the `BaseDependencies` protocol, defining a class, `AppDependencies` that represents our application base dependencies.

```swift
public protocol SearchDependencies: BaseDependencies {
    func makeAnalytics() -> Analytics
}
```

For some particular µFeatures, we might need some extra dependencies. We can define those in a new protocol that conforms the `BaseDependencies` protocol, adding the extra dependencies. In the example below `SearchDependencies` exposes also an `Analytics` dependency.

```swift
public final class SearchBuilder {

    private let dependenciesSolver: SearchDependencies

    public init(dependenciesSolver: SearchDependencies) {
        self.dependenciesSolver = dependenciesSolver
    }

    public func makeViewController() -> UIViewController {
        let client = dependenciesSolver.makeClient()
        let analytics = dependenciesSolver.makeLogger()
        return SearchViewController(client: client, logger: logger)
    }
}

// From the app
let searchBuilder = SearchBuilder(dependenciesSolver: AppDependencies())
```

The example above shows how we can inject dependencies in a builder that builds the µFeature instance, in this case a `UIViewController`.

> Note: This is just an example of how we can simplify dependency injection. There are other alternatives out there. It's up to you to pick up the one that works best for your project and setup.


## Cross-platform µFeatures ⌚️📱💻📺
Your product might be available in different platforms or from different products. In that case it's very important that we reuse as much code as possible. `Foundation` APIs are very similar across plaftorms, with just some subtle differences, but UI frameworks like `UIKit` or `AppKit` are not. We could have multiplatform µFeatures by using [Swift macros](http://ericasadun.com/2014/06/06/swift-cross-platform-code/) that conditionally compile UI for each of the platforms. However, Xcode is very bad dealing with those macros and the syntax highlghting and the autocompletion don't work very well.

There is something we can do though to enable cross-platform µFeatures. Splitting business logic and UI in two different layers, one that contains the business layer and is cross-platform, and another one that contains the UI and that is platform/product specific.

<p align="center">
    <img src="/assets/cross-platform.png" width="300" max-width="50%" alt="Cross platform µFeatures" />
</p>

The image above shows how the Search µFeature is split into the business logic target, `µSearch` and the platform specific ones `µSearchiOS` and `µSearchmacOS`. Both UI frameworks depend on `µSearch`. Each of those targets would have their corresponding tests target. 

> You can read more about how to setup frameworks to be cross-platform no the [following link](http://ilya.puchka.me/xcode-cross-platform-frameworks/).

## Shortcomings 🙈
#### Maintenance
µFeatures are Xcode projects with multiple targets that share the same base configuration. Although the configuration defined in build settings can be extracted into `.xcconfig` file, there are other settings that can't be extracted to be reused:

- The targets per µFeature project.
- The schemes available.
- The configurations available in each project.

Maintaining the setup is very cumbersome. For example, if you want to introduce a new configuration, you need to manually add the configuration in each project. Moreover, if you want to add a new µFeature, the process is very manual since there's no official API that allows you to automate some steps.


## FAQ 🐿

#### One or multiple Git repositories?
If you are working with git branches, we recommend you to keep everything in the same repository for convenience reasons. Facebook is a good example of a huge company keeping all the project in a single repositories and Uber [wrote about it](https://eng.uber.com/ios-monorepo/) a year ago.

#### How do you version µFeatures?
If µFeatures are part of the same repository, they are versioned with the app. If you have them in different repositories you can use Git Submodules, Carthage, or your own dependency resolver to fetch specific versions of your µFeatures to link from the app.

#### External dependencies?
This architecture doesn't limit you from using external dependencies. There's one thing to keep in mind though, you won't be able to use [CocoaPods](https://cocoapods.org) dependencies with your µFeatures. CocoaPods is not able to analyze your dependencies tree, and setup all the targets in the stack accordingly. If you want to use an external dependency from a µFeature framework we recommend you to use [Carthage](https://github.com/carthage)

#### Can I include resources?
You can, but remember that you can only do it if your µFeature is a framework and not a library.


## Resources 📚

- [Building µFeatures](https://speakerdeck.com/pepibumur/building-ufeatures)
- [Framework Oriented Programming](https://speakerdeck.com/pepibumur/framework-oriented-programming-mobilization-dot-pl)
- [A Journey into frameworks and Swift](https://speakerdeck.com/pepibumur/a-journey-into-frameworks-and-swift)
- [Leveraging frameworks to speed up our development on iOS - Part 1](https://developers.soundcloud.com/blog/leveraging-frameworks-to-speed-up-our-development-on-ios-part-1)
- [Library Oriented Programming](https://academy.realm.io/posts/justin-spahr-summers-library-oriented-programming/)
- [Building Modern Frameworks](https://developer.apple.com/videos/play/wwdc2014/416/)
- [The Unofficial Guide to xcconfig files](https://pewpewthespells.com/blog/xcconfig_guide.html)
- [Static and Dynamic Libraries](https://pewpewthespells.com/blog/static_and_dynamic_libraries.html)
