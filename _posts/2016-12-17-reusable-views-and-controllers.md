---
title: Reusable UIViews and UIViewControllers
---

The simplest way to use `UIView`s and `UIViewController`s is to drop them directly into storyboard files in Xcode's Interface Builder. But in more advanced apps you'll want to be able to reuse `UIView`s and `UIViewController`s. How can you do that while still taking advantage of Interface Builder for layout?

## UIViewControllers

First, set up the `UIViewController` in its own storyboard file, making it the initial view controller. Then, from within another `UIViewController`, you can instantiate it from the storyboard, pass any necessary data to it, and push it.

```swift
let storyboard = UIStoryboard.init(name: "MyStoryboard", bundle: nil)
let vc = storyboard.instantiateInitialViewController() as! MyViewController
vc.myCustomProperty = "Hi."
pushViewController(vc, animated: true)
```

This approach also works if you want to wrap your custom view controller in a `UINavigationController`:

```swift
let storyboard = UIStoryboard.init(name: "MyStoryboard", bundle: nil)
let navVC = storyboard.instantiateInitialViewController() as! UINavigationController
let vc = navVC.topViewController as! MyViewController
vc.myCustomProperty = "Hi."
pushViewController(vc, animated: true)
```

## UIViews

`UIView`s are a bit more complex. First, set up the `UIView` in its own nib (`.xib`) file. Then, instantiate it from the nib. Some looping is required to find the view itself:

```swift
let nib = UINib(nibName: "MyNib", bundle: nil)
let nibContents = nib.instantiate(withOwner: nil, options: nil)
var myView: MyCustomView? = nil
for nibView in nibContents {
    if let view = nibView as? MediaDetailView {
        myView = view
    }
}
```

It can be convenient to wrap this into a static method in your view:

```swift
class MyCustomView: UIView {

    static func view() -> MyCustomView? {
        let nib = UINib(nibName: "MyNib", bundle: nil)
        let nibContents = nib.instantiate(withOwner: nil, options: nil)
        for nibView in nibContents {
            if let view = nibView as? MediaDetailView {
                return view
            }
        }
        return nil
    }
    //...
}
```

The easiest way to place it is to add a plain `UIView` to your layout where you want it to go, and then set up auto layout constraints to pin your custom view to that container view:

```swift
parentView.addSubview(myView)

myView.translatesAutoresizingMaskIntoConstraints = false

NSLayoutConstraint.activate([
    myView.topAnchor.constraint(equalTo: parentView.topAnchor),
    myView.bottomAnchor.constraint(equalTo: parentView.bottomAnchor),
    myView.leadingAnchor.constraint(equalTo: parentView.leadingAnchor),
    myView.trailingAnchor.constraint(equalTo: parentView.trailingAnchor),
])
```

A few notes:

- You have to add `myView` as a subview of `parentView` before setting up any constraints, or your app will crash.
- Views instantiated from nib files automatically have `translatesAutoresizingMaskIntoConstraints` set to true. If you want to use auto layout constraints to place it (and you should), you need to set it to `false`, or the view won't resize properly.

It can be good to wrap the following into an extension method:

```swift
extension UIView {
    func pinToSuperviewEdges() {
        if let parentView = self.superview {
            self.translatesAutoresizingMaskIntoConstraints = false
            
            NSLayoutConstraint.activate([
                self.topAnchor.constraint(equalTo: parentView.topAnchor),
                self.bottomAnchor.constraint(equalTo: parentView.bottomAnchor),
                self.leadingAnchor.constraint(equalTo: parentView.leadingAnchor),
                self.trailingAnchor.constraint(equalTo: parentView.trailingAnchor),
            ])
        }
    }
}
```