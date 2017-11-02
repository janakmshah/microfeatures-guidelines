![Logo](assets/logo.png)
# µFeatures Manifesto

## Index 📝
- [What](#what-)
- [Contex](#context-)
- [Motivation](#motivation-)
- [Before reading](#before-reading-)
- [Core Principle](#core-principle-)
- [What is a µFeature](#what-is-a-µfeature-)
- [Why a µFeature](#why-a-µfeature-)
- [Dependencies](#dependencies-)

## What 🤔

uFeatures is a architectural approach to structure iOS applications to enable scalability, optimizing build and testing cycles, and ensuring good practices in your team. Its core idea is to build your apps by building independent features that are interconnected using clear and concise APIS.

This manifesto introduces the principles of the architecture, helping you identify and organize your application features in different layers. It also introduces tips, tools and advices if you decide for this architecture.

> The name uFeatures (Microfeatures)* is inspired by the backend Microservices architecture, where different "backend features" run as different services with defined APIS to enable communication between them.

## Context 🥑

Apps are made of features. Typically these features are part of the same module, or target where the whole application is defined. The natural inclination in the team is to continue building features and its tests in the same targets. As a result, the application and its tests target grows in complexity which manifests in bugs, bad compilation times, and team performance. What seemed to be a good architecture, doesn't work out that well in large codebases or teams.

**This is frequently a big source of frustration when it comes to work on those projects. The time we spend goes into compiling rather than building and experimenting with the platform.**

![Xcode compiling](assets/compiling.gif)

## Motivation 🍎

The µFeatures's main motivation is to support the scalability of large iOS codebases leveraging platform features and tools. There are other solutions out there that could be also be considered to overcome those issues. A very popular one nowadays is [React Native](https://facebook.github.io/react-native/) that leverages the Javascript dynamism to offer developers a pleasant experience working in the code base, but at the same time a native experience from the user point of view.

**We believe that the usage of native tools and technologies can be optimized to overcome scalability challenges that sooner or later show up in our projects**

## Before reading 🍒

- Don't expect this to be a silver-bullet to your problems. You should take the core ideas, process them, and apply the principles to your project/s.
- Each project is different so are the needs. With the ideas in the manifesto, and your needs, you should figure out what might work out for you.
- Since everything this architecture depends on is evolving *(tools, languages, concepts)*, the manifesto might get outdated very quickly. If that happens, don't hesitate to open a PR and contribute with keeping this manifesto up to date.

## Core principle 🐝

Developers should be able to **build, test and try** their features fast, with independence of the main app. 

## What is a µFeature 📱
A µFeature represents an application feature and is a combination of the following 4 targets *(referring with target to a Xcode target)*:

- **Source:** Contains the feature source code *(Swift, Objective-C, C++, React Native...)* and its resources *(images, fonts, storyboards, xibs)*.
- **Tests:** Contains the feature unit and integration tests.
- **Testing:** Provides testing data that can be used for the tests and from the example app. It also provide mocks for uFeature classes and protocols that can be used by other features as we'll see later.
- **Example:** Contains an example app that developers can use to try out the feature under certain conditions *(different languages, screen sizes, settings)*.

The diagram below shows the 4 targets and the dependencies between them:

## Why a µFeature 🐛

#### Clear and concise APIs
When all the app source code lives in the same target is very easy to build implicit dependencies in code, and end up with the so well-known spaguetti code. Everything is strongly coupled, the state is sometimes unpredictable, and introducing new changes become a nightmare. When we define features in independent targets we need to design public APIs as part of our feature implementation. We need to decide what should be public, how our feature should be consumed, what should remain private. We have more control over how we want our feature *"clients"* to use the feature and we can enforce good practises by designing safe APIs.

#### Small modules
[Divide and conquer](https://en.wikipedia.org/wiki/Divide_and_conquer). Working in small modules allows you to have more focus and test and try the feature in isolation. Moreover, development cycles are much faster since we have a more selective compilation, compiling only the components that are necessary to get our feature working. The compilation of the whole app is only necessary at the very end of our work, when we need to integrate the feature into the app.

#### Reusability
Reusing code across apps and other products like extensions is encouraged using frameworks or libraries. By building µFeatures reusing them is pretty straightforward. We can build an iMessage extension, a Today Extension, or a watchOS application by just combining existing µFeatures and adding *(when necessary)* platform-specific UI layers.

## Layers



## Dependencies ⚙️


## Ideas
- Features have APIs



## Resources 📚

- [Building µFeatures](https://speakerdeck.com/pepibumur/building-ufeatures)
- [Framework Oriented Programming](https://speakerdeck.com/pepibumur/framework-oriented-programming-mobilization-dot-pl)
- [A Journey into frameworks and Swift](https://speakerdeck.com/pepibumur/a-journey-into-frameworks-and-swift)
- [Leveraging frameworks to speed up our development on iOS - Part 1](https://developers.soundcloud.com/blog/leveraging-frameworks-to-speed-up-our-development-on-ios-part-1)
- [Library Oriented Programming](https://academy.realm.io/posts/justin-spahr-summers-library-oriented-programming/)
- [Building Modern Frameworks](https://developer.apple.com/videos/play/wwdc2014/416/)
- [The Unofficial Guide to xcconfig files](https://pewpewthespells.com/blog/xcconfig_guide.html)
- [Static and Dynamic Libraries](https://pewpewthespells.com/blog/static_and_dynamic_libraries.html)
