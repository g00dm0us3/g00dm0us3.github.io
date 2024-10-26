# Short notes on how SwiftUI works from WWDC

[WWDC 2021 - Demystify SwiftUI](https://developer.apple.com/videos/play/wwdc2021/10022)

1. View's lifetime is the duration of its identity.
    1. Identity is either explicitly defined for a view, or is structural - based on the type and position of the view in the hierarchy.
    See: 
    ```swift 
    if(condition) { ViewA() } else { ViewB() } -> some View = _ConditionalView<ViewA, ViewB> TrueView vs False view - identity.
    ```
    2. Same view can be in different states, but it has the same identity. Views are short lived.
2. Persistence of state is tied to a life time - basically, as soon as body in
```swift
    var body: some View { ... }
```
resolves to a type different from whatever it was (there is an apparent dynamics here, does it tie into animations?) the old view is destroyed.
3. Use ```Identifiable``` protocol for better performance, when structural identity is not an option. A brain-teaser here - why is if-else condition a stable enough way to provide identity information, but loop is not? Compile time vs Runtime?

Dependencies - data on which view depends. When the dependency (say a @Binding var changes) body of a view is reevaluated, potentially destroying and creating views. This data form a dependency graph, all the views which depend on a data. 

[WWDC 2019 - Data Essentials in SwiftUI](https://developer.apple.com/videos/play/wwdc2020/10040)
