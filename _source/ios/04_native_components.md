---
permalink: /ios/native-components.html
order: 04
title: "Native Components"
description: "Bridge the gap with native components driven by the web on iOS."
---

# Native Components

Hotwire Native abstracts the integration with [Strada](https://strada.hotwired.dev), making it even faster to get started. Let's walk through how to create a new component on iOS. This assumes you've already installed [the Strada JavaScript package](https://strada.hotwired.dev/handbook/installing).

Our component will add a native bar button item to the right side of the navigation bar. Tapping it will "click" the associated link in the HTML that drove it.

<img src="/assets/strada-ios-button.png" width="500" alt="Native button component on iOS">

Native button component on iOS

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

When `data-controller="button"` is found in the DOM `connect()` will be fired. This then calls `send()` which passes a message to a native component identified by `"button"`.

## Swift Component

In Xcode, create a new Swift file with the following.

```swift
import Strada
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

This unpacks the `{title}` object from the fired message and adds a native button to the right side of the screen. When it's tapped, the `UIAction` is fired, replying to the Strada message and calling `this.element.click()` in JavaScript.

Finally, register the component. If you followed the [getting started steps](/overview/getting-started-ios) then this will go in `SceneDelegate.swift`.

```swift
Hotwire.registerStradaComponents([
    ButtonComponent.self
])
```
