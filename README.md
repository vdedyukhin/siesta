# Siesta

**iOS REST Client Framework**

[![CocoaPods](https://img.shields.io/cocoapods/v/Siesta.svg)]() [![Carthage compatible](https://img.shields.io/badge/Carthage-compatible-4BC51D.svg?style=flat)](https://github.com/Carthage/Carthage) [![GitHub license](https://img.shields.io/github/license/mashape/apistatus.svg)]()

Drastically simplifies app code by providing a client-side cache of observable models for RESTful resources.

* **OS:** iOS 8+
* **Languages:** Written in Swift, supports Swift and Objective-C
* **Build requirements:** Xcode 7 beta 6, Swift 2.0, Carthage
* **License:** MIT
* **Status:** 1.0 release now in beta. Seeking feedback. Please experiment!

###Contents

- [What’s It For?](#what’s-it-for)
- [Features](#features)
- [Origin](#origin)
- [Design Philosophy](#design-philosophy)
- [Installation](#installation)
- [Basic Usage](#basic-usage)
- [Comparison With Other Frameworks](#comparison-with-other-frameworks)
- Documentation
  - [User Guide](https://bustoutsolutions.github.io/siesta/guide/)
  - [API Docs](https://bustoutsolutions.github.io/siesta/api/)
  - [Specs](https://bustoutsolutions.github.io/siesta/specs/)
- [Examples](#examples)
- [Contributing & Getting Help](#contributing-amp-getting-help)

## What’s It For?

### The Problem

Want your app to talk to an API? Welcome to your state nightmare!

You need to display response data whenever it arrives, unless the requesting ViewController is no longer visible, unless some other currently visible ViewController happens to want the same data. You should show a loading indicator (but watch out for race conditions that leave it stuck spinning forever), display user-friendly errors (but not redundantly — no modal alert dogpiles!), give users a retry mechanism … and hide all of that when a subsequent request succeeds. Be sure to avoid redundant requests. Oh, and remember not to retain your ViewController by accident in your callback closures. Unless you're supposed to.

What could possibly go wrong?

### The Solution

Siesta ends this headache by providing an observable model of a RESTful resource’s state. The model answers three basic questions:

* What is the latest data for this resource, if any?
* Did the latest request result in an error?
* Is there a request in progress?

…then provides notifications whenever the answers to these questions change.

Siesta handles all the transitions and corner cases to deliver these answers wrapped up with a pretty bow on top, letting you focus on your UI.

## Features

- Decouples UI element lifecycle from network request lifecycle
- Eliminates error-prone state tracking logic
- Eliminates redundant network requests
- Unified handling for all errors: encoding, network, server-side, and parsing
- Transparent Etag / If-Modified-Since handling
- Painless handling for JSON and plain text, plus customizable response transformation
- Prebaked UI for loading & error handling
- Debug-friendly, customizable logging
- Written in Swift with a great [Swift-centric API](https://bustoutsolutions.github.io/siesta/api/), but…
- …also works great from Objective-C thanks to a compatibility layer.
- Lightweight. Won’t achieve sentience and attempt to destroy you.
- [Robust regression tests](https://bustoutsolutions.github.io/siesta/specs/)
- [Documentation](https://bustoutsolutions.github.io/siesta/guide/)

### What it doesn’t do

- It **doesn’t reinvent networking.** Siesta delegates network operations to your library of choice (`NSURLSession` by default, or [Alamofire](https://github.com/Alamofire/Alamofire), or inject your own [custom adapter](http://bustoutsolutions.github.io/siesta/api/Protocols/NetworkingProvider.html).
- It **doesn’t hide HTTP**. On the contrary, Siesta strives to expose the full richness of HTTP while providing conveniences to simplify common usage patterns. You can devise an abstraction layer to suit your own particular needs, or work directly with Siesta’s nice APIs for requests and response entities.
- It **doesn’t do response ⟷ model mapping.** This means that Siesta doesn’t constrain your response models, or force you to have any at all. Add a repsonse transformer to work with your model library of choice, or work directly with parsed JSON.

### What sets it apart

It’s not just the features. Siesta **solves a different problem** than other REST frameworks.

Other frameworks treat HTTP as a form of [RPC](https://en.wikipedia.org/wiki/Remote_procedure_call). New information arrives only in responses that are coupled to requests — essentially the return values of asynchronous functions.

Siesta puts the the “ST” back in “REST”, and embraces the notion of _state transfer_, decoupling the act of _observering_ state from the act of _transferring_ it.

## Origin

This project started as helper code we wrote out of practical need on several [Bust Out Solutions](http://bustoutsolutions.com) projects. When we found ourselves copying the code between projects, we knew it was time to open source it.

For the open source transition, we took the time to rewrite our code in Swift — and _rethink_ it in Swift, embracing the language to make the API as clean as the concepts.

Siesta’s code is therefore both old and new: battle-tasted on the App Store, then reincarnated in a Swifty green field.

## Design Philosophy

Make the default thing the right thing most of the time.

Make the right thing easy all of the time.

Build from need. Don’t invent solutions in search of problems.

Design the API with these goals:

1. Make client code easy to **read**.
2. Make client code easy to **write**.
3. Keep the API clean.
4. Keep the implementation tidy.

_…in that order of priority._

---

## Installation

Siesta requires Swift 2.0, so install the latest [Xcode 7 beta](https://developer.apple.com/xcode/downloads/), then point the command line tools at it:

    sudo xcode-select -s /Applications/Xcode-beta.app/Contents/Developer

### CocoaPods

In your `Podfile`:

    pod 'Siesta'

If you want to use Alamofire as your networking provider instead of `NSURLSession`:

    pod 'Siesta/Alamofire'

(You’ll also need to pass an `Alamofire.Manager` when you configure your `Siesta.Service`. See the [API docs](http://bustoutsolutions.github.io/siesta/api/Classes/Service.html#/s:FC6Siesta7ServicecFMS0_FT4baseGSqSS_22useDefaultTransformersSb18networkingProviderPS_18NetworkingProvider__S0_) for more info.)

### Carthage

In your `Cartfile`:

    github "bustoutsolutions/siesta" "1.0-beta.1"

Follow the [Carthage instructions](https://github.com/Carthage/Carthage#adding-frameworks-to-an-application) to add `Siesta.framework` to your project.

As of this writing, there is one additional step you need to follow for Xcode 7 beta that isn’t in the Carthage docs:

* Build settings → Framework search paths → `$(PROJECT_DIR)/Carthage/Build/iOS/`

(In-depth discussion of Carthage on XC7 is [here](https://github.com/Carthage/Carthage/issues/536).)

---

## Basic Usage

Make a singleton for the REST API you want to use:

```swift
let MyAPI = Service(base: "https://api.example.com")
```

Now register your view controller — or view, or anything you like — to receive notifications whenever a particular resource’s state changes:

```swift
override func viewDidLoad() {
    super.viewDidLoad()

    MyAPI.resource("/profile").addObserver(self)
}
```

Use those notifications to populate your UI:

```swift
@IBOutlet weak var nameLabel, colorLabel, errorLabel: UILabel!

func resourceChanged(resource: Resource, event: ResourceEvent) {
    nameLabel.text = resource.jsonDict["name"] as? String
    colorLabel.text = resource.jsonDict["favoriteColor"] as? String

    errorLabel.text = resource.latestError?.userMessage
}
```

Trigger a staleness-aware load when the view appears:

```swift
override func viewWillAppear(animated: Bool) {
    MyAPI.resource("/profile").loadIfNeeded()
}
```

…and you have a networked UI.

Add a loading indicator:

```swift
MyAPI.resource("/profile").addObserver(self) {
    [weak self] resource, event in

    self?.activityIndicator.hidden = !resource.loading
}
```

…or better yet, use Siesta’s prebaked `ResourceStatusOverlay` view to get an activity indicator, a nicely formatted error message, and a retry button for free:

```swift
class ProfileViewController: UIViewController, ResourceObserver {
    @IBOutlet weak var nameLabel, colorLabel: UILabel!

    @IBOutlet weak var statusOverlay: ResourceStatusOverlay

    override func viewDidLoad() {
        super.viewDidLoad()

        MyAPI.resource("/profile")
            .addObserver(self)
            .addObserver(statusOverlay)
    }
    
    override func viewWillAppear(animated: Bool) {
        super.viewWillAppear(animated)
        MyAPI.resource("/profile").loadIfNeeded()
    }

    func resourceChanged(resource: Resource, event: ResourceEvent) {
        nameLabel.text  = resource.jsonDict["name"] as? String
        colorLabel.text = resource.jsonDict["favoriteColor"] as? String
    }
}
```

Note that this example is not toy code. Together with its storyboard, **this small class is a fully armed and operational REST-backed user interface**.

### Your socks still on?

Take a look at AFNetworking’s venerable `UIImageView` extension for asynchronously loading and caching remote images on demand. Seriously, go [skim that code](https://github.com/AFNetworking/AFNetworking/blob/master/UIKit%2BAFNetworking/UIImageView%2BAFNetworking.m) and digest all the cool things it does. Take a few minutes. I’ll wait. I’m a README. I’m not going anywhere.

Got it? Good.

Here’s how you implement the same functionality using Siesta:

```swift
class RemoteImageView: UIImageView {
  static var imageCache: Service = Service()
  
  var placeholderImage: UIImage?
  
  var imageURL: NSURL? {
    get { return imageResource?.url }
    set { imageResource = RemoteImageView.imageCache.resource(url: newValue) }
  }
  
  var imageResource: Resource? {
    willSet {
      imageResource?.removeObservers(ownedBy: self)
      imageResource?.cancelLoadIfUnobserved(afterDelay: 0.05)
    }
    
    didSet {
      imageResource?.loadIfNeeded()
      imageResource?.addObserver(owner: self) { [weak self] _ in
        self?.image = imageResource?.contentAsType(ifNone: placeholderImage)
      }
    }
  }
}
```

The same functionality. Yes, really.

<small>(Well, OK, they’re not quite identical. The Siesta version above has more robust caching & updating behavior.)</small>

There’s a more featureful version of `RemoteImageView` [already included with Siesta](http://bustoutsolutions.github.io/siesta/api/Classes/RemoteImageView.html) — but the UI freebies aren’t the point. The point is that Siesta gives you an **elegant abstraction** that **solves problems you actually have**.

## Comparison With Other Frameworks

Popular REST frameworks all have different primary goals:

- [Siesta](http://bustoutsolutions.github.io/siesta/) untangles state problems with an observable resource cache.
- [Alamofire](https://github.com/Alamofire/Alamofire) provides a Swifty, modern-feeling wrapper for Apple’s network APIs.
- [Moya](https://github.com/Moya/Moya) wraps Alamofire to hide HTTP URLs and parameters.
- [RestKit](https://github.com/RestKit/RestKit) couples HTTP with JSON ⟷ object model ⟷ Core Data mapping.
- [AFNetworking](https://github.com/AFNetworking/AFNetworking) is a modern-feeling Obj-C wrapper for Apple’s network APIs, plus a suite of related utilities.

Siesta has robust functionality, but does not attempt to solve everything. In particular, Moya and RestKit address complementary / alternative concerns, while Alamofire and AFNetworking provide more robust low-level HTTP support.

Further complicating a comparison, some frameworks are built on top of others. This capabilities table may help clear the confusion:

|                             | Siesta             | Alamofire      | RestKit       | Moya      | AFNetworking    | NSURLSession   |
|:----------------------------|:------------------:|:--------------:|:-------------:|:---------:|:---------------:|:--------------:|
| HTTP requests               | ✓                  | ✓              | ✓             | ✓         | ✓               | ✓              |
| Async response callbacks    | ✓                  | ✓              | ✓             | ✓         | ✓               | ✓              |
| Observable in-memory cache  | ✓                  |                |               |           |                 |                |
| Prevents redundant requests | ✓                  |                |               |           |                 |                |
| Prevents redundant parsing  | ✓                  |                |               |           |                 |                |
| Parsing for common formats  | ✓                  | ✓              |               |           | ✓               |                |
| Route-based parsing         | ✓                  |                | ✓             |           |                 |                |
| Content-type-based parsing  | ✓                  |                |               |           |                 |                |
| File upload/download tasks  |                    | ✓              | ~             |           | ✓               |                |
| Object model mapping        |                    |                | ✓             |           |                 |                |
| Core data integration       |                    |                | ✓             |           |                 |                |
| Hides HTTP                  |                    |                |               | ✓         |                 |                |
| UI helpers                  | ✓                  |                |               |           | ✓               |                |
| Primary langauge            | Swift              | Swift          | Obj-C         | Swift     | Obj-C           | Obj-C          |
| Built on top of | <small>any (injectable)</small>| <small>NSURLSession</small> | <small>AFNetworking</small> | <small>Alamofire</small> | <small>NSURLSession / NSURLConnection</small>| <small>Apple guts</small>

<small>Disclaimer: table above compiled by Siesta’s non-omniscient author. Corrections / additions? Please [submit a PR](https://github.com/bustoutsolutions/siesta/edit/master/README.md).</small>

Despite this capabilities list, Siesta is a relatively small codebase — almost exactly the same size as Alamofire, and 5.5x smaller than RestKit.

---

## Documentation

- **[User Guide](https://bustoutsolutions.github.io/siesta/guide/)**
- **[API documentation](https://bustoutsolutions.github.io/siesta/api/)**
- **[Specs](https://bustoutsolutions.github.io/siesta/specs/)**

## Examples

This repo includes a [simple example project](https://github.com/bustoutsolutions/siesta/tree/master/Examples/GithubBrowser). Use Carthage to build its dependencies.

## Contributing & Getting Help

To report a bug, [file an issue](https://github.com/bustoutsolutions/siesta/issues/new).

To point out anything incorrect or confusing in the documentation, [file an issue](https://github.com/bustoutsolutions/siesta/issues/new).

To submit a feature request / cool idea, [file an issue](https://github.com/bustoutsolutions/siesta/issues/new).

To get help, post your question to [Stack Overflow](https://stackoverflow.com) and tag it with **siesta-swift**. (Be sure to include the tag. It triggers a notification.)
