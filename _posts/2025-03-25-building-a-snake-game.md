---
layout: post
title: "Building a Snake Game (on terminal with Swift)"
date: 2025-03-25 10:00:00 +0000
categories: swift terminal gamedev
hero_image: /assets/images/snake-game-hero.png
hero_caption: "Classic Snake Game in the Terminal"
---

Please checkout the repository at [https://github.com/rational-kunal/SSSnake](https://github.com/rational-kunal/SSSnake)

## Basic terminal configuration

To create a smooth terminal-based game, we need to control the terminal behavior by:
1. Hiding the cursor to prevent flickering
2. Enabling raw mode to capture key presses without requiring the user to hit Enter.

```swift
static func hideCursor() { 
    print("\u{001B}[?25l", terminator: "") 
} 

static func enableRawMode() { 
    var raw = termios() 
    tcgetattr(STDIN_FILENO, &raw) 
    raw.c_lflag &= ~tcflag_t(ECHO | ICANON) // Disable echo & line buffering 
    raw.c_cc.0 = 1 // Min character read 
    raw.c_cc.1 = 0 // No timeout 
    tcsetattr(STDIN_FILENO, TCSAFLUSH, &raw) 
}
```

## Rendering the Game

The game is displayed using a 2D character array (canvas). To render a frame, we print this array. Also, instead of printing each frame anew (which causes flickering), we move the cursor to the top and update the existing buffer.

```swift
struct Terminal { 
    private(set) var canvas: [[Character]] = TerminalHelper.makeCanvas() 
    
    // Renders the canvas / frame on the terminal 
    mutating func render() { 
        TerminalHelper.moveCursor(x: 0, y: 0) 
        for row in canvas { 
            print(String(row)) 
        } 
        fflush(stdout) 
    } 
    
    // Draws the symbol at the given position 
    mutating func draw(x: Int, y: Int, symbol: Character) { 
        canvas[y][x] = symbol 
    } 
}
```

To compute the size of the canvas, i.e. terminal size, we can use the `ioctl` function.

```swift
private struct TerminalHelper { 
    static func windowSize() -> (width: Int, height: Int) { 
        var ws = winsize() 
        _ = ioctl(STDOUT_FILENO, TIOCGWINSZ, &ws) 
        return (Int(ws.ws_col), Int(ws.ws_row) - 1) 
    } 
    
    // Create blank 2d buffer of blank characters 
    static func makeCanvas() -> [[Character]] { 
        let (width, height) = windowSize() 
        return Array(repeating: Array(repeating: " ", count: width), count: height) 
    } 
}
```

## Get the user input

We need a non-blocking way to listen for user input. Running a background thread ensures that key presses are read continuously while the game logic runs separately.

```swift
DispatchQueue.global(qos: .userInteractive).async { [weak self] in 
    while let self, self.isRunning, let key = Terminal.getKeyPress() { 
        self.processInput(key) 
    } 
} 

extension Terminal { 
    static func getKeyPress() -> String? { 
        var buffer = [UInt8](repeating: 0, count: 3) 
        return read(STDIN_FILENO, &buffer, 3) == 1 ? String(UnicodeScalar(buffer[0])) : nil 
    } 
}
```

## Game loop

There should be a loop where we will ask the game to update its state and then update the canvas and then render it.

```swift
class Game { 
    lazy var terminal = Terminal() 
    
    public func start() { 
        // Start the game loop 
        while isRunning { 
            terminal.update() // Refresh canvas 
            loop()           // Update game state 
            draw()           // Draw updated elements 
            terminal.render() // Render the frame 
            usleep(100_000)   // Control game speed (10 FPS) 
        } 
    } 
    
    func processInput(_ key: String) {} 
    func loop() {} 
    func draw() {} 
}
```

## Snake game

Now, we can override the `Game` class to create our own Snake game.

```swift
class SnakeGame: Game { 
    override func draw() { 
        // Draw border, snake and its food 
        // Like: terminal.at(x: 12, y: 33, symbol: "#") 
    } 
    
    override func loop() { 
        // Update game state like move the snake spawn food if needed etc 
    } 
    
    override func processInput(_ key: String) { 
        // Process inputs turn the direction of snake as needed 
    } 
}
```

Check out its implementation at [SnakeGame.swift](https://github.com/rational-kunal/SSSnake/blob/main/Sources/SnakeGame.swift)

<img src="{{ '/assets/images/snake-game-demo.gif' | relative_url }}" alt="Snake Game Demo" style="display: block; margin: 20px auto; max-height: 500px; width: auto;" />

Thanks for reading.
