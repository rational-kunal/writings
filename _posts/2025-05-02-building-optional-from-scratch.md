---
layout: post
title: "Building Optional from scratch"
date: 2025-05-02 10:00:00 +0000
categories: swift ios
hero_image: /assets/images/building-optional.jpg
hero_caption: "De-mystifying Swift Optionals with Enums"
---

From this article [2 Minute Tips: The Dark Secret of Optionals](https://blog.jacobstechtavern.com/p/2-minute-tips-the-dark-secret-of?utm_source=publication-search) I got to know that the Optionals are nothing but an **Enum**. I started experimenting: How can we build Optional from scratch?

### Enter Schrödinger's Cat or … a box

You might have heard of Schrödinger’s Cat — the thought experiment where a cat in a box is both alive and dead until you check. That's basically an Optional, right? A value that might exist... or not.

So, let's turn this idea into code:

```swift
enum 📦<Value> { 
    case 😺(Value) 
    case 😵 
    
    init(_ from: Value) { 
        self = .😺(from) 
    } 
}
```

Now we can do:

```swift
let x: 📦 = .😺(123) 
let y: 📦 = .😵
```

But that's not quite as convenient as Swift's native syntax: `let x: Int? = 123`.

Can we make ours feel just as nice?

### Literal support

Swift lets types conform to `ExpressibleByNilLiteral`. Let’s do that:

```swift
extension 📦: ExpressibleByNilLiteral { 
    init(nilLiteral: ()) { 
        self = .😵 
    } 
}
```

Now we can say:

```swift
let y = nil as 📦<Int>
```

Note: we still need to add `as` since `nil` has no type context.

Nice! But we can't do `let x: 📦<Int> = 123` directly — Swift won't let us. As [this StackOverflow post](https://stackoverflow.com/questions/77243489/optional-like-initialization-of-custom-type) explains, that’s a compiler-level limitation.

But hey, we are creative. Let’s define a custom postfix operator to return the optional:

```swift
postfix operator ❓ 

postfix func ❓<T>(value: T) -> 📦<T> { 
    .😺(value) 
}
```

Now the usage becomes:

```swift
let x = 123❓            // same as .😺(123) 
let y = nil as 📦<Int>   // same as .😵
```

### Lets implement force unwrapping

Again we can use a custom postfix operator for this:

```swift
postfix operator ❗ 

postfix func ❗<T>(value: 📦<T>) -> T { 
    switch value { 
    case .😺(let v): 
        return v 
    case .😵: 
        fatalError("Cat is dead Bruh!") 
    } 
}
```

### Nil Coalescing

Lets add `??`-like operator:

```swift
infix operator ❓➡️: NilCoalescingPrecedence 

func ❓➡️<T>(lhs: 📦<T>, rhs: @autoclosure () -> T) -> T { 
    switch lhs { 
    case .😺(let value): 
        return value 
    case .😵: 
        return rhs() 
    } 
}
```

```swift
let value1 = x ❓➡️ 999 // 123 
let value2 = y ❓➡️ 999 // 999
```

### Optional Chaining

This was the complex part that I had to workout. For this I created a wrapper Optional type which has a `@dynamicMemberLookup` on it.

```swift
@dynamicMemberLookup 
struct _📦<Value> { 
    private let storage: 📦<Value> 
    
    init(from: 📦<Value>) { 
        storage = from 
    } 
    
    subscript<T>(dynamicMember keyPath: KeyPath<Value, 📦<T>>) -> 📦<T> { 
        switch storage { 
        case .😺(let v): 
            return v[keyPath: keyPath] 
        case .😵: 
            return .😵 
        } 
    } 
} 

postfix operator ❔ 

postfix func ❔<T>(value: 📦<T>) -> _📦<T> { 
    _📦(from: value) 
}
```

Let's try it:

```swift
struct TheInteger { 
    let x: 📦<Int> 
} 

struct Service { 
    let integer: 📦<TheInteger> 
} 

let service = Service(integer: TheInteger(x: 27❓)❓) 
let s = service.integer❔.x // 😺(27) 

print(s)
```

Why bother? — Because its fun and I got to play with under the hood stuff that I will probably not use in a production environment.

---

### References
- [Jacob's Tech Tavern: The Dark Secret of Optionals](https://blog.jacobstechtavern.com/p/2-minute-tips-the-dark-secret-of?utm_source=publication-search)
- [Swift by Sundell: Custom operators in Swift](https://www.swiftbysundell.com/articles/custom-operators-in-swift/)
- [Swift with Majid: Dynamic member lookup in Swift](https://swiftwithmajid.com/2023/05/23/dynamic-member-lookup-in-swift)
