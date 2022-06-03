# Factory

A new approach to Container-Based Dependency Injection for Swift and SwiftUI.

## Why do something new?

Resolver was my first Dependency Injection system. While quite powerful and still in use in many of my applications, Resolver suffers from a few drawbacks.

1. Resolver requires pre-registration of all service factories up front. 
2. Resolver uses type inference to dynamically find and return registered services in a container.

The first issue can lead to a performance hit on application launch. Then again, the registration process is usually quick and not normally noticable. No, it's the second item that's somewhat more problematic. 

 Failure to find a matching type *could* lead to an application crash if we attempt to resolve a given type and if a matching registration is not found. In practice that isn't really a problem as such a thing tends to be noticed and fixed rather quickly the very first time you run a unit test or when you run the application to see if your newest feature works.
 
 But... could we do better? That question lead me on a quest for compile-time type safety. Several other systems have attempted to solve this, but I didn't want to have to add a source code scanning and generation step to my build process, nor did I want to give up a lot of the control and flexibility inherent in a run-time-based system.
 
 Could I have my cake and eat it too?
 
 ## Features
 
 Factory is strongly influenced by SwiftUI, and in my opinion is highly suited for use in that environment. Factory is...
 
 * **Safe:** Factory is compile-time safe; a dependency for a given type *must* exist or the code simply will not compile.
 * **Flexible:** It's easy to override dependencies at runtime and for use in SwiftUI Previews. And, like Resolver, Factory supports application, cached, shared, and custom scopes.
 * **Lightweight:** Factory is slim and trim, coming in at about 200 lines of code.
 * **Performant:** Little to no setup time is needed for the vast majority of your services, resolutions are extremely fast, and no compile-time scripts or build phases are needed.
 * **Concise:** Defining a given registration usually takes but a single line of code.
 
 Sound too good to be true? Let's take a look.
 
 ## A simple example
 
 Most container-based dependency injection systems require you to define in some way that a given service type is injectable and many reqire some sort of factory or mechanism that will provide a new instance of the service when needed.
 
 Factory is no exception. Here's a simple registraion and its associated factory.
 
```
extension Factory {
    static let myService = Factory { MySimpleService() }
}
```
Unlike Resolver which often requires defining a plethora of registration functions, or SwiftUI, where defining a new environment variable requires creating a new EnvironmentKey and adding additional getters and setters, in Factory an injection relies simply on exposing a new factory on the provided Factory container. That's it.

Injecting and using the service where needed is equally straightforward.

```
class ContentViewModel: ObservableObject {
    @Injected(Factory.myService) var myService
    ...
}
```
Here our view model uses an `@Injected` property wrapper to request the desired dependency. Similar to `@EnvironmentObject` in SwiftUI, you simply provide the property wrapper with a reference to a factory of the desired type and it handles the rest.

And that's the core mechanism. In order to use the property wrapper you *must* provide the predefined factory. That factory that *must* return the desired type. Fail to do either one and the code will simply not compile. As such, Factory is compile-time safe.