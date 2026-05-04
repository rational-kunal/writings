---
layout: post
title: "Xcode debugging cheatsheet"
date: 2024-02-04 10:00:00 +0000
categories: ios
hero_image: /assets/images/xcode-debugging.png
hero_caption: "Xcode LLDB Debugging Cheatsheet"
---

This is a cheatsheet - starting point to look for things that you can do. Avoided complex / 3rd party things as they require some additional setup that may not be allowed in certain projects.

### Breakpoints
- Breakpoint is user enabled point in code where the application pauses
- Line and column breakpoints
- Pauses the program before the statement on line/column is executed
- Conditions: Execute walk method just for LadyBug: `self.bugType == 1`
- Actions: Sound, Log message, Debug command
- Symbolic breakpoints: Add breakpoint on a method without source file `[UIImage systemImageNamed:]`

```
po $arg1 -> Receiver object
po $arg2 -> po (SEL)$arg2 -> Method name
po $arg3 -> First argument, and goes
```

- Watch points: Pauses when the value changes

### LLDB commands
- `help`: Information about commands
- `v`: Does not compile the expression instead it just dumps value from the stack
- `p`: Evaluates and prints the output
- `po`: Evaluates the print debug output
  - description method
  - Use `[]` syntax if `.` is failing due to typecasting
- `expression`: Evaluates and prints, also stores the output in a variable
  - Expressions in other languages: `expression -l swift/objc -O -- <code>`
- `alias`: Creates a shortcut for complex commands
  - `command alias greet p @"Hello World"`

- Inject code without building it: `expression <sample fix/feature>`
  - `expr [bugs addObjectsFromArray:[[self makeAntViews:20] mutableCopy]]`
- Short circuit / skip code execution:
  - Move the debugger handle
  - `thread jump --by 1`
- Early return through a method: `thread return <return value>`
  - NOTE: This does not always work and can be dangerous 
- Print the view hierarchy: `po [self.view recursiveDescription]`
- Highlight a view:
  - `(UIColor *)[bug setBackgroundColor:UIColor]`
  - `(void)[CATransition flush]`: Refreshes UI
- Add delay in code: `expr [NSThread sleepForTimeInterval:2]`
- Auto layout debugging: `po [self.view _autolayoutTrace]`

### References
1. [https://www.wwdcnotes.com/notes/wwdc18/412/](https://www.wwdcnotes.com/notes/wwdc18/412/)
2. [Advanced lldb tricks for Swift - Injecting and changing code on the fly](https://swiftrocks.com/using-lldb-manually-xcode-console-tricks.html?utm_campaign=iOS%2BDev%2BWeekly&utm_medium=web&utm_source=iOS%2BDev%2BWeekly%2BIssue%2B416)
3. [https://bryce.co/lldb-method-stubbing/](https://bryce.co/lldb-method-stubbing/)
4. [https://www.objc.io/issues/19-debugging/lldb-debugging/](https://www.objc.io/issues/19-debugging/lldb-debugging/)
5. [https://github.com/facebook/chisel](https://github.com/facebook/chisel)
6. [Leveraging Swift in Xcode’s LLDB](https://itnext.io/leveraging-swift-in-xcodes-lldb-d75e2adc5741)
