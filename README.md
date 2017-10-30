# µFeatures Manifesto

# What

uFeatures is a architectural approach to structure iOS applications to enable scalability, optimizing build and testing cycles, and good practices in your team. Its core idea is to build your apps by building independent features that are interconnected using clear and concise APIS.

This manifesto introduces the principles of the architecture, helping you identify and organize your application features in different layers. It also introduces tips, tools and advices worth to be considered.

> The name uFeatures (Microfeatures)* is inspired by the backend Microservices architecture, where different "backend features" run as different services with defined APIS to enable communication between them.

# Context

Apps are made of features. Typically these features are part of the same module, or target where the whole application is defined. The natural inclination in the team is to continue building features and its tests in the same targets. As a result, the application and its tests target grows in complexity which manifests in bugs, bad compilation times, and team performance. What seemed to be a good architecture, doesn't work out that well in large codebases or teams.

This is frequently a big source of frustration when it comes to work on those projects. The time we spend goes into compiling rather than building and experimenting with the platform.

# Motivation