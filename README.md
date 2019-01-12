# XRouter

A simple routing library for iOS projects.

[![Codacy Badge](https://api.codacy.com/project/badge/Grade/d0ef88b70fc843adb2944ce0d956269d)](https://app.codacy.com/app/reececomo/XRouter?utm_source=github.com&utm_medium=referral&utm_content=reececomo/XRouter&utm_campaign=Badge_Grade_Dashboard)
[![CodeCov Badge](https://codecov.io/gh/reececomo/XRouter/branch/master/graph/badge.svg)](https://codecov.io/gh/reececomo/XRouter)
[![Build Status](https://travis-ci.org/reececomo/XRouter.svg?branch=master)](https://travis-ci.org/reececomo/XRouter)
[![Docs Badge](https://raw.githubusercontent.com/reececomo/XRouter/master/docs/badge.svg?sanitize=true)](https://reececomo.github.io/XRouter)
[![Version](https://img.shields.io/cocoapods/v/XRouter.svg?style=flat)](https://cocoapods.org/pods/XRouter)
[![License](https://img.shields.io/cocoapods/l/XRouter.svg?style=flat)](https://cocoapods.org/pods/XRouter)
[![Platform](https://img.shields.io/cocoapods/p/XRouter.svg?style=flat)](https://cocoapods.org/pods/XRouter)

<p align="center">
<img src="https://raw.githubusercontent.com/reececomo/XRouter/master/XRouter.jpg" alt="XRouter" width="625" style="max-width:625px;width:auto;height:auto;"/>
</p>

## Usage
### Basic Usage

#### Creating a Router
```swift
// Create a router
let router = Router<MyRoutes>()

// Navigate to a route
try? router.navigate(to: .loginFlow)
```

#### Configuring Routes

Define your routes, like so:

```swift
enum AppRoute {
    case home
    case profile(withID: Int)
}
```

- Note: By default, enum properties dont factor into equality checks/comparisons. You can provide your own
        implemention of `var name: String` or `static func == (_:_:)` if you would like to override this.

Implement the protocol stubs:
```swift
extension AppRoute: RouteProvider {

    /// Configure the transitions
    var transition: RouteTransition {
        switch self {
        case .home:
            return .push
        case .profile:
            return .modal
        }
    }

    /// Prepare the view controller for the route
    /// - You can use this to configure entry points into flow coordinators
    /// - You can throw errors here to cancel the navigation
    func prepareForTransition(from currentViewController: UIViewController) throws {
        switch self {
        case .home:
            return HomeCoordinator.shared.navigateHome().navigationController
        case .profile(let profileID):
            let myProfile = try Profile.load(withID: profileID)
            return ProfileViewController(profile: myProfile)
        }
    }

}
```

### Advanced Usage

#### Wrapping Try/Catch

It can be messy trying to manage do/catch blocks in your app, and it is important to handle errors.

It is highly recommended to wrap the navigate function if you handle
all navigation errors in the same way. For example, something like this:
```swift
extension BaseCoordinator {

    /// Open the route, or report an error
    @discardableResult func navigate(to: Route) -> Bool {
        do {
            try router.navigate(to: route, animated: true)
            return true
        } catch {
            // Handle error here! Log, etc.
            assertionFailure("An error occured while navigating to \(route.name). Error: \(error)")
            return false
        }
    }

}
```

#### Custom Transitions
Here is an example using the popular [Hero Transitions](https://github.com/HeroTransitions/Hero) library.

Set the `customTransitionDelegate` for the `Router`:
```swift
router.customTransitionDelegate = self
```

(Optional) Define your custom transitions in an extension so you can refer to them statically
```swift
extension RouteTransition {
    static var myHeroFade: RouteTransition {
        return .custom(identifier: "heroFade")
    }
}
```

Implement the delegate method `performTransition(...)`:
```swift

extension AppDelegate: RouterCustomTransitionDelegate {
    
    /// Perform a custom transition
    func performTransition(to newViewController: UIViewController,
                           from oldViewController: UIViewController,
                           transition: RouteTransition,
                           animated: Bool) {
        if transition == .myHeroFade {
            oldViewController.hero.isEnabled = true
            newViewController.hero.isEnabled = true
            newViewController.hero.modalAnimationType = .fade
            
            // Creates a container nav stack
            let containerNavController = UINavigationController()
            containerNavController.hero.isEnabled = true
            containerNavController.setViewControllers([newViewController], animated: false)
            
            // Present the hero animation
            oldViewController.present(containerNavController, animated: animated)
        }
    }
    
}
```

And set the transition to `.custom` in your `Routes.swift` file:
```swift
    var transition: RouteTransition {
        switch self {
            case .myRoute:
                return .myHeroFade
        }
    }
```

## Documentation

Complete [documentation is available here](https://reececomo.github.io/XRouter).

## Example

To run the example project, clone the repo, and run it in Xcode 10.

## Requirements

## Installation

### CocoaPods

XRouter is available through [CocoaPods](https://cocoapods.org). To install
it, simply add the following line to your Podfile:

```ruby
pod 'XRouter'
```

## Author

Reece Como, reece.como@gmail.com

## License

XRouter is available under the MIT license. See the LICENSE file for more info.
