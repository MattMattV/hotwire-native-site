---
permalink: /ios/bridge-components.html
order: 03
title: "Bridge Components"
description: "Bridge the gap with native bridge components driven by the web on iOS."
---

# Bridge Components

Hotwire Native abstracts the integration between the OS and JavaScript using Bridge Components, making it even faster to get started. Let's walk through how to create a new component on iOS. This assumes you've already installed [the Strada JavaScript package](https://strada.hotwired.dev/handbook/installing) on your server.

The component will add a native bar button item to the right side of the navigation bar. Tapping it will "click" the associated link in the HTML.

<figure>
    <img src="/assets/bridge-ios-button.png" width="500" alt="Native button component on iOS">
    Native button component on iOS
</figure>

Components are made of three parts: the HTML markup, a Stimulus controller, and the native code. The HTML configures Stimulus which passes messages to Swift.

## Stimulus Controller

On your server, add the data attributes needed to wire up the Stimulus controller.

```html
<a href="/profile" data-controller="button" data-bridge-title="Profile">
  View profile
</a>
```

Then, create a new JavaScript component/controller with the following.

```javascript
import { BridgeComponent } from "@hotwired/strada"

export default class extends BridgeComponent {
  static component = "button"

  connect() {
    super.connect()

    const element = this.bridgeElement
    const title = element.bridgeAttribute("title")
    this.send("connect", {title}, () => {
      this.element.click()
    })
  }
}
```

This component identifies itself as `"button"` via the static `component` property. It will pass all messages to a bridge component identified with the same name.

When `data-controller="button"` is found in the DOM then `connect()` is fired. This function calls `send()` which passes the title of the button to the bridge component.

The third parameter of send, the callback block, is executed when the bridge component replies back to the message, which is explained below. Here, the button is clicked.

## Swift Component

In Xcode, create a new Swift file with the following.

```swift
import HotwireNative
import UIKit

final class ButtonComponent: BridgeComponent {
    override class var name: String { "button" }

    override func onReceive(message: Message) {
        guard let viewController else { return }
        addButton(via: message, to: viewController)
    }

    private var viewController: UIViewController? {
        delegate.destination as? UIViewController
    }

    private func addButton(via message: Message, to viewController: UIViewController) {
        guard let data: MessageData = message.data() else { return }

        let action = UIAction { [unowned self] _ in
            self.reply(to: "connect")
        }
        let item = UIBarButtonItem(title: data.title, primaryAction: action)
        viewController.navigationItem.rightBarButtonItem = item
    }
}

private extension ButtonComponent {
    struct MessageData: Decodable {
        let title: String
    }
}
```

First, the component identifies itself as `"button"` via `name` to match the Stimulus controller.

`onReceive(message:)` is called when a message is received from Stimulus. Here, the `{title}` object is unpacked to add a native button to the right side of the screen. When it's tapped, the `UIAction` is fired, replying to the message and calling the callback block, clicking the button.

Finally, register the component. If you followed the [getting started steps](/overview/getting-started-ios) then this will go in `SceneDelegate.swift` before routing your first URL.

```swift
Hotwire.registerBridgeComponents([
    ButtonComponent.self
])
```
