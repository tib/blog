---
title: Custom @Environment value for share actions
description: Showing share actions in SwiftUI style
image: "./assets/cover.png"
publication: '2021-02-13'
authors:
  - artem-novichkov
---

SwiftUI has a lot of modern and useful features. One of my favourite is @Environment property wrapper. It allows you to get system-wide settings, for instance, current locale or color scheme. Since iOS 14.0 you can use `openURL` value to open URLs from apps easily.

```swift
@available(iOS 14.0, macOS 11.0, tvOS 14.0, watchOS 7.0, *)
extension EnvironmentValues {

/// Opens a URL using the appropriate system service.

    public var openURL: OpenURLAction { get }
}
```

With just a one line of code you can extend behaviour of you views:

```swift
import SwiftUI

struct ContentView: View {

    @Environment(\.openURL) var openURL
    
    var body: some View {
        Button("Share") {
            openURL(URL(string: "https://www.artemnovichkov.com")!)
        }
    }
}
```

I was wondering how it works under the hood and tried to implement the same trick for sharing activity items with `UIActivityViewController`. Let's check the result!

## Keys and values

At first we should create a custom key, conform to `EnvironmentKey` protocol and set a default value. It will be an empty struct for now:

```swift
struct ShareAction {}

struct ShareActionEnvironmentKey: EnvironmentKey {

    static let defaultValue: ShareAction = .init()
}
```

Next, we should extend `EnvironmentValues` to make `ourShareAction` be available from the environment:

```swift
extension EnvironmentValues {

    var share: ShareAction {
        self[ShareActionEnvironmentKey]
    }
}
```

The last thing is adding `callAsFunction` in `ShareAction` struct to use the same syntax:

```swift
struct ShareAction {

    func callAsFunction(_ activityItems: [Any]) {
        let vc = UIActivityViewController(activityItems: activityItems, applicationActivities: nil)
        UIApplication.shared.windows.first?.rootViewController?.present(vc, animated: true, completion: nil)
    }
}
```

> Unfortunately, there is no SwiftUI-way to open this controller. If you use a better way, let me know!

To read more about callable values of user-defined nominal types, check [SE-0253](https://github.com/apple/swift-evolution/blob/master/proposals/0253-callable.md) proposal on Github.

That's it, now we can use `.share` value and share activities from any view:

```swift
import SwiftUI

struct ContentView: View {

    @Environment(\.share) var share
    
    var body: some View {
        Button("Share") {
            share([URL(string: "https://www.artemnovichkov.com")!])
        }
    }
}
```

## Related Resources

The final code is available [here](https://github.com/artemnovichkov/ShareEnvironmentExample), and there is a list of related articles:

- [Environment Documentation](https://developer.apple.com/documentation/swiftui/environment)
- [What is @Environment in SwiftUI](https://sarunw.com/posts/what-is-environment-in-swiftui) by [Sarun W.](https://twitter.com/sarunw)
- [The power of Environment in SwiftUI](https://swiftwithmajid.com/2019/08/21/the-power-of-environment-in-swiftui) by [Majid Jabrayilov](https://twitter.com/mecid)
- [How to use @EnvironmentObject to share data between views](https://www.hackingwithswift.com/quick-start/swiftui/how-to-use-environmentobject-to-share-data-between-views) by [Paul Hudson](https://twitter.com/twostraws)
