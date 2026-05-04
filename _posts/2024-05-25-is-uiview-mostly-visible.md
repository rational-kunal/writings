---
layout: post
title: "Is UIView (mostly) visible"
date: 2024-05-25 10:00:00 +0000
categories: ios
hero_image: /assets/images/uiview-visible.jpg
hero_caption: "Checking UIView visibility beyond isHidden"
---

If you've ever needed to check whether a view is visible to the user, you might have used the `isHidden` property. However, this method falls short in scenarios where a view can appear hidden due to overlapping by other views. To address this, I created a simple extension to provide a more comprehensive visibility check.

```swift
extension UIView { 
    /// Checkes if the view is (mostly) visible to user or not. 
    /// Internaly it checks following things 
    /// - Should NOT be hidden 
    /// - Should NOT be completely transparent 
    /// - Bound should NOT be empty 
    /// - Should be in some window i.e. in view heirarchy 
    /// - Center should be directly visible to user i.e. NOT overlapped with other views 
    var isMostlyVisible: Bool { 
        guard !isHidden, 
              alpha > 0, 
              !bounds.isEmpty, 
              let window, 
              window.hitTest(window.convert(center, from: self.superview), with: nil) == self else { 
            return false 
        } 
        return true 
    } 
}
```

Here is the extension in action:

<img src="{{ '/assets/images/uiview-visible-demo.gif' | relative_url }}" alt="UIView isMostlyVisible Demo" style="display: block; margin: 0 auto; max-height: 500px; width: auto;" />

Thank you for reading.
