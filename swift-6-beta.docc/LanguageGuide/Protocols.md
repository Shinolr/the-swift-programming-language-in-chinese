# 协议

定义所遵循的类型必须实现的方法、属性和其他要求。

*协议（Protocol）*定义了满足特定任务或功能所需的方法、属性和其他要求的蓝图。类、结构体或枚举可以*遵循（adopt）*该协议，并提供协议要求的具体实现。任何满足协议要求的类型都被称为*遵循（conform）*该协议。

除了声明所遵循类型必须实现的要求之外，你还可以对协议进行扩展，通过扩展来实现一部分要求或者实现一些附加功能，这样遵循协议的类型就能够使用这些功能。

<!--
  FIXME: Protocols should also be able to support initializers,
  and indeed you can currently write them,
  but they don't work due to
  <rdar://problem/13695680> Constructor requirements in protocols (needed for NSCoding).
  I'll need to write about them once this is fixed.
  UPDATE: actually, they *can* be used right now,
  but only in a generic function, and not more generally with the protocol type.
  I'm not sure I should mention them in this chapter until they work more generally.
-->

## 协议语法

协议的定义方式与类、结构体和枚举的定义非常相似：

```swift
protocol SomeProtocol {
    // 这里是协议的定义部分
}
```

<!--
  - test: `protocolSyntax`

  ```swifttest
  -> protocol SomeProtocol {
        // protocol definition goes here
     }
  ```
-->

要让自定义类型遵循某个协议，在定义类型时，需要在类型名称后加上协议名称，中间以冒号（`:`）分隔。遵循多个协议时，各协议之间用逗号（`,`）分隔：

```swift
struct SomeStructure: FirstProtocol, AnotherProtocol {
    // 这里是结构体的定义部分
}
```

<!--
  - test: `protocolSyntax`

  ```swifttest
  >> protocol FirstProtocol {}
  >> protocol AnotherProtocol {}
  -> struct SomeStructure: FirstProtocol, AnotherProtocol {
        // structure definition goes here
     }
  ```
-->

如果一个类拥有父类，应该将父类名放在其他遵循的协议名之前，以逗号分隔：

```swift
class SomeClass: SomeSuperclass, FirstProtocol, AnotherProtocol {
    // 这里是类的定义部分
}
```

<!--
  - test: `protocolSyntax`

  ```swifttest
  >> class SomeSuperclass {}
  -> class SomeClass: SomeSuperclass, FirstProtocol, AnotherProtocol {
        // class definition goes here
     }
  ```
-->

> 注意：由于协议是类型，它们的名称应以大写字母开头（如 `FullyNamed` 和 `RandomNumberGenerator`），以与 Swift 中其他类型的命名规范（如 `Int`、`String` 和 `Double`）保持一致。

## 属性要求

协议可以要求遵循协议的类型提供特定名称和类型的实例属性或类型属性。协议不指定属性是存储属性还是计算属性，它只指定属性的名称和类型。此外，协议还指定属性是*可读*的还是*可读可写的*。

如果协议要求属性是可读可写的，那么该属性不能是常量属性或只读的计算属性。如果协议只要求属性是可读的，那么该属性不仅可以是可读的，如果你自己的代码需要的话，还可以是可写的。

协议总是用 `var` 关键字来声明变量属性，在类型声明后加上 `{ set get }`来表示属性是可读可写的，可读属性则用 `{ get }` 来表示：

```swift
protocol SomeProtocol {
    var mustBeSettable: Int { get set }
    var doesNotNeedToBeSettable: Int { get }
}
```

<!--
  - test: `instanceProperties`

  ```swifttest
  -> protocol SomeProtocol {
        var mustBeSettable: Int { get set }
        var doesNotNeedToBeSettable: Int { get }
     }
  ```
-->

在协议中定义类型属性时，总是使用 `static` 关键字作为前缀。当一个类遵循协议时，除了 `static` 关键字，还可以使用 `class` 关键字来声明类型属性：

```swift
protocol AnotherProtocol {
    static var someTypeProperty: Int { get set }
}
```

<!--
  - test: `instanceProperties`

  ```swifttest
  -> protocol AnotherProtocol {
        static var someTypeProperty: Int { get set }
     }
  ```
-->

如下所示，这是一个只含有一个实例属性要求的协议：

```swift
protocol FullyNamed {
    var fullName: String { get }
}
```

<!--
  - test: `instanceProperties`

  ```swifttest
  -> protocol FullyNamed {
        var fullName: String { get }
     }
  ```
-->

`FullyNamed` 协议除了要求遵循协议的类型提供 `fullName` 属性外，并没有其他特别的要求。这个协议表示，任何遵循 `FullyNamed` 的类型，都必须有一个可读的 `String` 类型的实例属性 `fullName`。

下面是一个遵循 `FullyNamed` 协议的简单结构体：

```swift
struct Person: FullyNamed {
    var fullName: String
}
let john = Person(fullName: "John Appleseed")
// john.fullName 为 "John Appleseed"
```

<!--
  - test: `instanceProperties`

  ```swifttest
  -> struct Person: FullyNamed {
        var fullName: String
     }
  -> let john = Person(fullName: "John Appleseed")
  /> john.fullName is \"\(john.fullName)\"
  </ john.fullName is "John Appleseed"
  ```
-->

这个例子中定义了一个叫做 `Person` 的结构体，用来表示一个具有名字的人。它在定义的第一行声明了它遵循 `FullyNamed` 协议。

每个 `Person` 实例都有一个 `String` 类型的存储属性 `fullName`。这正好满足了 `FullyNamed` 协议的要求，也就意味着 `Person` 结构体正确地遵循了协议。
（如果协议要求未被完全满足，Swift 在编译时会报错。）

下面是一个更为复杂的类，它遵循并符合了 `FullyNamed` 协议：

```swift
class Starship: FullyNamed {
    var prefix: String?
    var name: String
    init(name: String, prefix: String? = nil) {
        self.name = name
        self.prefix = prefix
    }
    var fullName: String {
        return (prefix != nil ? prefix! + " " : "") + name
    }
}
var ncc1701 = Starship(name: "Enterprise", prefix: "USS")
// ncc1701.fullName 为 "USS Enterprise"
```

<!--
  - test: `instanceProperties`

  ```swifttest
  -> class Starship: FullyNamed {
        var prefix: String?
        var name: String
        init(name: String, prefix: String? = nil) {
           self.name = name
           self.prefix = prefix
        }
        var fullName: String {
           return (prefix != nil ? prefix! + " " : "") + name
        }
     }
  -> var ncc1701 = Starship(name: "Enterprise", prefix: "USS")
  /> ncc1701.fullName is \"\(ncc1701.fullName)\"
  </ ncc1701.fullName is "USS Enterprise"
  ```
-->

`Starship` 类把 `fullName` 作为只读的计算属性来实现。每一个 `Starship` 类的实例都有一个名为 `name` 的非可选属性和一个名为 `prefix` 的可选属性。当 `prefix` 存在时，计算属性 `fullName` 会将 `prefix` 插入到 `name` 之前，从而得到一个带有 `prefix` 的 `fullName`。

<!--
  TODO: add some advice on how protocols should be named
-->

## 方法要求

协议可以要求遵循协议的类型实现某些指定的实例方法和类方法。这些方法的编写方式与普通实例方法和类型方法完全相同，都写在协议定义的一部分中，但没有大括号或方法主体。协议允许使用可变参数，和普通方法的定义方式相同。但是，不能在协议定义中为方法参数指定默认值。

正如属性要求中所述，在协议中定义类方法的时候，总是使用 `static` 关键字作为前缀。即使在类中实现时，类方法要求使用 `class` 或 `static` 作为关键字前缀，这条规则仍然适用：

```swift
protocol SomeProtocol {
    static func someTypeMethod()
}
```

<!--
  - test: `typeMethods`

  ```swifttest
  -> protocol SomeProtocol {
        static func someTypeMethod()
     }
  ```
-->

下面的例子定义了一个只含有一个实例方法的协议：

```swift
protocol RandomNumberGenerator {
    func random() -> Double
}
```

<!--
  - test: `protocols`

  ```swifttest
  -> protocol RandomNumberGenerator {
        func random() -> Double
     }
  ```
-->

`RandomNumberGenerator` 协议要求遵循协议的类型必须拥有一个名为 `random`，返回值类型为 `Double` 的实例方法。尽管这里并未指明，但是我们假设返回值是从 `0.0` 到（但不包括）`1.0`。

`RandomNumberGenerator` 协议并没有对如何生成每个随机数做任何假设 —— 它只要求生成器提供一种标准的方式来生成新的随机数。

这里有一个遵循并符合 `RandomNumberGenerator` 协议的类。该类实现了一个叫做 *线性同余生成器（linear congruential generator）* 的伪随机数算法。

```swift
class LinearCongruentialGenerator: RandomNumberGenerator {
    var lastRandom = 42.0
    let m = 139968.0
    let a = 3877.0
    let c = 29573.0
    func random() -> Double {
        lastRandom = ((lastRandom * a + c)
            .truncatingRemainder(dividingBy:m))
        return lastRandom / m
    }
}
let generator = LinearCongruentialGenerator()
print("Here's a random number: \(generator.random())")
// 打印 “Here's a random number: 0.37464991998171”
print("And another one: \(generator.random())")
// 打印 “And another one: 0.729023776863283”
```

<!--
  - test: `protocols`

  ```swifttest
  -> class LinearCongruentialGenerator: RandomNumberGenerator {
        var lastRandom = 42.0
        let m = 139968.0
        let a = 3877.0
        let c = 29573.0
        func random() -> Double {
           lastRandom = ((lastRandom * a + c)
               .truncatingRemainder(dividingBy:m))
           return lastRandom / m
        }
     }
  -> let generator = LinearCongruentialGenerator()
  -> print("Here's a random number: \(generator.random())")
  <- Here's a random number: 0.3746499199817101
  -> print("And another one: \(generator.random())")
  <- And another one: 0.729023776863283
  ```
-->

## 变值方法要求

有时需要在方法中改变（或*变值（mutate）*）方法所属的实例。例如，在值类型（即结构体和枚举）的实例方法中，将 `mutating` 关键字作为方法的前缀，写在 `func` 关键字之前，表示可以在该方法中修改它所属的实例以及实例的任意属性的值。这一过程在 <doc:Methods#Modifying-Value-Types-from-Within-Instance-Methods> 章节中有详细描述。

如果你在协议中定义了一个实例方法，该方法会改变遵循该协议的类型的实例，那么在定义协议时需要在方法前加 `mutating` 关键字。这使得结构体和枚举能够遵循此协议并满足此方法要求。

> 注意: 实现协议中的 `mutating` 方法时，若是类类型，则不用写 `mutating` 关键字。`mutating` 关键字只用于结构体和枚举。

如下所示，`Togglable` 协议只定义了一个名为 `toggle` 的实例方法。顾名思义，`toggle()` 方法将改变实例属性，从而切换遵循该协议类型的实例的状态。

`toggle()` 方法在定义的时候，使用 `mutating` 关键字标记，这表明当它被调用时，该方法将会改变遵循协议的类型的实例：

```swift
protocol Togglable {
    mutating func toggle()
}
```

<!--
  - test: `mutatingRequirements`

  ```swifttest
  -> protocol Togglable {
        mutating func toggle()
     }
  ```
-->

当使用枚举或结构体来实现 `Togglable` 协议时，需要提供一个被标记为 `mutating` 的 `toggle()` 方法。

下面定义了一个名为 `OnOffSwitch` 的枚举。这个枚举在两种状态之间进行切换，用枚举成员 `On` 和 `Off` 表示。枚举的 `toggle()` 方法被标记为 `mutating`，以满足 `Togglable` 协议的要求：

```swift
enum OnOffSwitch: Togglable {
    case off, on
    mutating func toggle() {
        switch self {
        case .off:
            self = .on
        case .on:
            self = .off
        }
    }
}
var lightSwitch = OnOffSwitch.off
lightSwitch.toggle()
// lightSwitch 现在的值为 .on
```

<!--
  - test: `mutatingRequirements`

  ```swifttest
  -> enum OnOffSwitch: Togglable {
        case off, on
        mutating func toggle() {
           switch self {
              case .off:
                 self = .on
              case .on:
                 self = .off
           }
        }
     }
  -> var lightSwitch = OnOffSwitch.off
  -> lightSwitch.toggle()
  // lightSwitch is now equal to .on
  ```
-->

## 构造器要求

协议可以要求遵循协议的类型实现指定的构造器。你可以像编写普通构造器那样，在协议的定义里写下构造器的声明，但不需要写花括号和构造器的实体：

```swift
protocol SomeProtocol {
    init(someParameter: Int)
}
```

<!--
  - test: `initializers`

  ```swifttest
  -> protocol SomeProtocol {
        init(someParameter: Int)
     }
  ```
-->

### 协议构造器要求的类实现

你可以在遵循协议的类中实现构造器，无论是作为指定构造器，还是作为便利构造器。无论哪种情况，你都必须为构造器实现标上 `required` 修饰符：

```swift
class SomeClass: SomeProtocol {
    required init(someParameter: Int) {
        // 这里是构造器的实现部分
    }
}
```

<!--
  - test: `initializers`

  ```swifttest
  -> class SomeClass: SomeProtocol {
        required init(someParameter: Int) {
           // initializer implementation goes here
        }
     }
  ```
-->

<!--
  - test: `protocolInitializerRequirementsCanBeImplementedAsDesignatedOrConvenience`

  ```swifttest
  -> protocol P {
        init(x: Int)
     }
  -> class C1: P {
        required init(x: Int) {}
     }
  -> class C2: P {
        init() {}
        required convenience init(x: Int) {
           self.init()
        }
     }
  ```
-->

使用 `required` 修饰符可以确保所有子类也必须提供此构造器实现，从而也能遵循协议。

关于 `required` 构造器的更多内容，参考 <doc:Initialization#Required-Initializers>。

<!--
  - test: `protocolInitializerRequirementsRequireTheRequiredModifierOnTheImplementingClass`

  ```swifttest
  -> protocol P {
        init(s: String)
     }
  -> class C1: P {
        required init(s: String) {}
     }
  -> class C2: P {
        init(s: String) {}
     }
  !$ error: initializer requirement 'init(s:)' can only be satisfied by a 'required' initializer in non-final class 'C2'
  !! init(s: String) {}
  !! ^
  !! required
  ```
-->

<!--
  - test: `protocolInitializerRequirementsRequireTheRequiredModifierOnSubclasses`

  ```swifttest
  -> protocol P {
        init(s: String)
     }
  -> class C: P {
        required init(s: String) {}
     }
  -> class D1: C {
        required init(s: String) { super.init(s: s) }
     }
  -> class D2: C {
        init(s: String) { super.init(s: s) }
     }
  !$ error: 'required' modifier must be present on all overrides of a required initializer
  !! init(s: String) { super.init(s: s) }
  !! ^
  !! required
  !$ note: overridden required initializer is here
  !! required init(s: String) {}
  !! ^
  ```
-->

> 注意: 如果类已经被标记为 `final`，那么不需要在协议构造器的实现中使用 `required` 修饰符，因为 `final` 类不能有子类。关于 `final` 修饰符的更多内容，参见 <doc:Inheritance#Preventing-Overrides>。

<!--
  - test: `finalClassesDoNotNeedTheRequiredModifierForProtocolInitializerRequirements`

  ```swifttest
  -> protocol P {
        init(s: String)
     }
  -> final class C1: P {
        required init(s: String) {}
     }
  -> final class C2: P {
        init(s: String) {}
     }
  ```
-->

如果一个子类重写了父类的指定构造器，并且该构造器满足了某个协议的要求，那么该构造器的实现需要同时标注 `required` 和 `override` 修饰符：

```swift
protocol SomeProtocol {
    init()
}

class SomeSuperClass {
    init() {
        // 这里是构造器的实现部分
    }
}

class SomeSubClass: SomeSuperClass, SomeProtocol {
    // 因为遵循协议，需要加上 required；因为继承自父类，需要加上 override
    required override init() {
        // 这里是构造器的实现部分
    }
}
```

<!--
  - test: `requiredOverrideInitializers`

  ```swifttest
  -> protocol SomeProtocol {
        init()
     }
  ---
  -> class SomeSuperClass {
        init() {
           // initializer implementation goes here
        }
     }
  ---
  -> class SomeSubClass: SomeSuperClass, SomeProtocol {
        // "required" from SomeProtocol conformance; "override" from SomeSuperClass
        required override init() {
           // initializer implementation goes here
        }
     }
  ```
-->

### 可失败构造器要求

协议还可以为遵循协议的类型定义可失败构造器要求，详见 <doc:Initialization#Failable-Initializers>。

遵循协议的类型可以通过可失败构造器（`init?`）或非可失败构造器（`init`）来满足协议中定义的可失败构造器要求。协议中定义的非可失败构造器要求可以通过非可失败构造器（`init`）或隐式解包可失败构造器（`init!`）来满足。

<!--
  - test: `failableRequirementCanBeSatisfiedByFailableInitializer`

  ```swifttest
  -> protocol P { init?(i: Int) }
  -> class C: P { required init?(i: Int) {} }
  -> struct S: P { init?(i: Int) {} }
  ```
-->

<!--
  - test: `failableRequirementCanBeSatisfiedByIUOInitializer`

  ```swifttest
  -> protocol P { init?(i: Int) }
  -> class C: P { required init!(i: Int) {} }
  -> struct S: P { init!(i: Int) {} }
  ```
-->

<!--
  - test: `iuoRequirementCanBeSatisfiedByFailableInitializer`

  ```swifttest
  -> protocol P { init!(i: Int) }
  -> class C: P { required init?(i: Int) {} }
  -> struct S: P { init?(i: Int) {} }
  ```
-->

<!--
  - test: `iuoRequirementCanBeSatisfiedByIUOInitializer`

  ```swifttest
  -> protocol P { init!(i: Int) }
  -> class C: P { required init!(i: Int) {} }
  -> struct S: P { init!(i: Int) {} }
  ```
-->

<!--
  - test: `failableRequirementCanBeSatisfiedByNonFailableInitializer`

  ```swifttest
  -> protocol P { init?(i: Int) }
  -> class C: P { required init(i: Int) {} }
  -> struct S: P { init(i: Int) {} }
  ```
-->

<!--
  - test: `iuoRequirementCanBeSatisfiedByNonFailableInitializer`

  ```swifttest
  -> protocol P { init!(i: Int) }
  -> class C: P { required init(i: Int) {} }
  -> struct S: P { init(i: Int) {} }
  ```
-->

<!--
  - test: `nonFailableRequirementCanBeSatisfiedByNonFailableInitializer`

  ```swifttest
  -> protocol P { init(i: Int) }
  -> class C: P { required init(i: Int) {} }
  -> struct S: P { init(i: Int) {} }
  ```
-->

<!--
  - test: `nonFailableRequirementCanBeSatisfiedByIUOInitializer`

  ```swifttest
  -> protocol P { init(i: Int) }
  -> class C: P { required init!(i: Int) {} }
  -> struct S: P { init!(i: Int) {} }
  ```
-->

## 协议作为类型

协议本身并不实现任何功能。尽管如此，你仍然可以在代码中将协议用作类型。

最常见的将协议用作类型的方式是将其用作泛型约束。具有泛型约束的代码可以与任何符合该协议的类型一起工作，具体的类型由使用该 API 的代码选择。例如，当你调用一个函数并传入一个参数，而该参数的类型是泛型时，调用者会选择具体的类型。

在代码中使用不透明类型时，可以与某个符合该协议的类型一起工作。底层类型在编译时是已知的，API 实现会选择该类型，但该类型的身份对 API 的使用方是隐藏的。使用不透明类型可以防止 API 的实现细节泄露到抽象层之外 —— 例如，通过隐藏函数的具体返回类型，并仅保证该值符合给定的协议。

代码使用装箱（boxed）的协议类型时，可以与任何在运行时选择的、符合该协议的类型一起工作。为了支持这种运行时的灵活性，Swift 在必要时会添加一个间接层 —— 称为*箱子（box）*，这会带来性能开销。由于这种灵活性，Swift 在编译时无法知道底层类型，这意味着你只能访问协议所要求的成员。要访问底层类型的任何其他 API，都需要在运行时进行强制转换。

关于使用协议作为泛型约束的信息，参考 <doc:Generics>。关于不透明类型和装箱协议类型的信息，参考 <doc:OpaqueTypes>。
<!--
Performance impact from SE-0335:

Existential types are also significantly more expensive than using concrete types.
Because they can store any value whose type conforms to the protocol,
and the type of value stored can change dynamically,
existential types require dynamic memory
unless the value is small enough to fit within an inline 3-word buffer.
In addition to heap allocation and reference counting,
code using existential types incurs pointer indirection and dynamic method dispatch
that cannot be optimized away.
-->

## 代理

*代理（Delegate）*是一种设计模式，它允许类或结构体将一些需要它们负责的功能代理给其他类型的实例。代理模式的实现很简单：定义协议来封装那些需要被代理的功能，这样就能确保遵循协议的类型能提供这些功能。代理模式可以用来响应特定的动作，或者接收外部数据源提供的数据，而无需关心外部数据源的类型。

下面的例子定义了两个基于骰子游戏的协议：
以下示例定义了一个骰子游戏，以及一个用于跟踪游戏进度的嵌套协议：

```swift
class DiceGame {
    let sides: Int
    let generator = LinearCongruentialGenerator()
    weak var delegate: Delegate?

    init(sides: Int) {
        self.sides = sides
    }

    func roll() -> Int {
        return Int(generator.random() * Double(sides)) + 1
    }

    func play(rounds: Int) {
        delegate?.gameDidStart(self)
        for round in 1...rounds {
            let player1 = roll()
            let player2 = roll()
            if player1 == player2 {
                delegate?.game(self, didEndRound: round, winner: nil)
            } else if player1 > player2 {
                delegate?.game(self, didEndRound: round, winner: 1)
            } else {
                delegate?.game(self, didEndRound: round, winner: 2)
            }
        }
        delegate?.gameDidEnd(self)
    }

    protocol Delegate: AnyObject {
        func gameDidStart(_ game: DiceGame)
        func game(_ game: DiceGame, didEndRound round: Int, winner: Int?)
        func gameDidEnd(_ game: DiceGame)
    }
}
```

`DiceGame` 类实现了一个游戏，每个玩家轮流掷骰子，掷出最高点数的玩家赢得该轮。它使用前文中示例中的线性同余生成器来生成骰子掷出的随机数。

`DiceGame.Delegate` 协议可用于跟踪骰子游戏的进度。由于 `DiceGame.Delegate` 协议总是在骰子游戏的上下文中使用，因此它被嵌套在 `DiceGame` 类内部。协议可以嵌套在类型声明（如结构体和类）内部，只要外部声明不是泛型。关于嵌套类型的更多信息，参见 <doc:NestedTypes>。

为了防止强引用循环，代理被声明为弱引用。关于弱引用的更多信息，参见 <doc:AutomaticReferenceCounting#Strong-Reference-Cycles-Between-Class-Instances>。将协议标记为 class-only 允许 `DiceGame` 类声明其代理必须使用弱引用。一个 class-only 协议通过继承自 `AnyObject` 来标记，如 <doc:Protocols#Class-Only-Protocols> 中所述。

`DiceGame.Delegate` 提供了三个方法来跟踪游戏的进度。这三个方法被整合到上面的 `play(rounds:)` 方法的游戏逻辑中。当新游戏开始、新回合开始或游戏结束时，`DiceGame` 类会调用它的代理方法。

因为 `delegate` 属性是 *可选的* `DiceGame.Delegate`，所以 `play(rounds:)` 方法在每次调用代理方法时都使用可选链，如 <doc:OptionalChaining> 中所述。如果 `delegate` 属性为 nil，这些代理调用将被忽略。如果 `delegate` 属性不为 nil，则会调用代理方法，并将 `DiceGame` 实例作为参数传递。

下一个示例展示了一个名为 `DiceGameTracker` 的类，它遵循了 `DiceGame.Delegate` 协议：

```swift
class DiceGameTracker: DiceGame.Delegate {
    var playerScore1 = 0
    var playerScore2 = 0
    func gameDidStart(_ game: DiceGame) {
        print("Started a new game")
        playerScore1 = 0
        playerScore2 = 0
    }
    func game(_ game: DiceGame, didEndRound round: Int, winner: Int?) {
        switch winner {
            case 1:
                playerScore1 += 1
                print("Player 1 won round \(round)")
            case 2: playerScore2 += 1
                print("Player 2 won round \(round)")
            default:
                print("The round was a draw")
        }
    }
    func gameDidEnd(_ game: DiceGame) {
        if playerScore1 == playerScore2 {
            print("The game ended in a draw.")
        } else if playerScore1 > playerScore2 {
            print("Player 1 won!")
        } else {
            print("Player 2 won!")
        }
    }
}
```

`DiceGameTracker` 类实现了 `DiceGame.Delegate` 协议要求的所有三个方法。它通过这些方法在新游戏开始时将两个玩家的分数清零，在每轮结束时更新他们的分数，以及在游戏结束时宣布获胜者。

以下是 `DiceGame` 和 `DiceGameTracker` 的实际运行情况:

```swift
let tracker = DiceGameTracker()
let game = DiceGame(sides: 6)
game.delegate = tracker
game.play(rounds: 3)
// 开始新游戏
// Player 2 won round 1
// Player 2 won round 2
// Player 1 won round 3
// Player 2 won!
```

## 在扩展里添加协议遵循

即便无法修改源代码，你依然可以通过扩展令已有类型遵循并符合协议。扩展可以为已有类型添加属性、方法、下标以及构造器，因此可以符合协议中可能需要的任意要求。关于扩展的更多详情，参见 <doc:Extensions>。

> 注意: 当一个协议的遵循被添加到实例类型的扩展中时，现有的实例会自动遵循并符合该协议。

例如下面这个 `TextRepresentable` 协议，任何想要通过文本表示一些内容的类型都可以实现该协议。这些想要表示的内容可以是实例本身的描述，也可以是实例当前状态的文本描述：

```swift
protocol TextRepresentable {
    var textualDescription: String { get }
}
```

<!--
  - test: `protocols`

  ```swifttest
  -> protocol TextRepresentable {
        var textualDescription: String { get }
     }
  ```
-->

上面提到的 `Dice` 类可以被扩展以遵循并符合 `TextRepresentable` 协议：

<!--
  No "from above" xref because
  even though Dice isn't defined in the section immediately previous
  it's part of a running example and Dice is used in that section.
-->

```swift
extension Dice: TextRepresentable {
    var textualDescription: String {
        return "A \(sides)-sided dice"
    }
}
```

<!--
  - test: `protocols`

  ```swifttest
  -> extension Dice: TextRepresentable {
        var textualDescription: String {
           return "A \(sides)-sided dice"
        }
     }
  ```
-->

通过扩展遵循并适配协议，和在原始定义中遵循并符合协议的效果完全相同。协议名称写在类型名之后，以冒号隔开，然后在扩展的大括号内实现协议要求的内容。

现在所有 `Dice` 的实例都可以被看做 `TextRepresentable` 类型：

```swift
let d12 = Dice(sides: 12, generator: LinearCongruentialGenerator())
print(d12.textualDescription)
// 打印 “A 12-sided dice”
```

<!--
  - test: `protocols`

  ```swifttest
  -> let d12 = Dice(sides: 12, generator: LinearCongruentialGenerator())
  -> print(d12.textualDescription)
  <- A 12-sided dice
  ```
-->

同样，`SnakesAndLadders` 类也可以通过扩展来适配和遵循 `TextRepresentable` 协议：

```swift
extension SnakesAndLadders: TextRepresentable {
    var textualDescription: String {
        return "A game of Snakes and Ladders with \(finalSquare) squares"
    }
}
print(game.textualDescription)
// 打印 “A game of Snakes and Ladders with 25 squares”
```

<!--
  - test: `protocols`

  ```swifttest
  -> extension SnakesAndLadders: TextRepresentable {
        var textualDescription: String {
           return "A game of Snakes and Ladders with \(finalSquare) squares"
        }
     }
  -> print(game.textualDescription)
  <- A game of Snakes and Ladders with 25 squares
  ```
-->

### 有条件地遵循协议

泛型类型可能只在某些情况下满足一个协议的要求，比如当类型的泛型形式参数遵循对应协议时。你可以通过在扩展类型时列出限制让泛型类型有条件地遵循某协议。在你采纳协议的名字后面写泛型 `where` 分句。更多关于泛型 `where` 分句，参见 <doc:Generics#Generic-Where-Clauses>。

下面的扩展让 `Array` 类型只要在存储遵循 `TextRepresentable` 协议的元素时，就遵循 `TextRepresentable` 协议。

```swift
extension Array: TextRepresentable where Element: TextRepresentable {
    var textualDescription: String {
        let itemsAsText = self.map { $0.textualDescription }
        return "[" + itemsAsText.joined(separator: ", ") + "]"
    }
}
let myDice = [d6, d12]
print(myDice.textualDescription)
// 打印 "[A 6-sided dice, A 12-sided dice]"
```

<!--
  - test: `protocols`

  ```swifttest
  -> extension Array: TextRepresentable where Element: TextRepresentable {
        var textualDescription: String {
           let itemsAsText = self.map { $0.textualDescription }
           return "[" + itemsAsText.joined(separator: ", ") + "]"
        }
     }
     let myDice = [d6, d12]
  -> print(myDice.textualDescription)
  <- [A 6-sided dice, A 12-sided dice]
  ```
-->

### 在扩展里声明协议遵循

当一个类型已经遵循了某个协议中的所有要求，却还没有声明遵循该协议时，可以通过空的扩展来让它遵循该协议：

```swift
struct Hamster {
    var name: String
    var textualDescription: String {
        return "A hamster named \(name)"
    }
}
extension Hamster: TextRepresentable {}
```

<!--
  - test: `protocols`

  ```swifttest
  -> struct Hamster {
        var name: String
        var textualDescription: String {
           return "A hamster named \(name)"
        }
     }
  -> extension Hamster: TextRepresentable {}
  ```
-->

从现在起，`Hamster` 的实例可以作为 `TextRepresentable` 类型使用：

```swift
let simonTheHamster = Hamster(name: "Simon")
let somethingTextRepresentable: TextRepresentable = simonTheHamster
print(somethingTextRepresentable.textualDescription)
// 打印 “A hamster named Simon”
```

<!--
  - test: `protocols`

  ```swifttest
  -> let simonTheHamster = Hamster(name: "Simon")
  -> let somethingTextRepresentable: TextRepresentable = simonTheHamster
  -> print(somethingTextRepresentable.textualDescription)
  <- A hamster named Simon
  ```
-->

> 注意: 即使满足了协议的所有要求，类型也不会自动遵循协议，必须显式地遵循协议。

## 使用合成实现来遵循协议

Swift 可以在很多简单场景下自动提供遵循 `Equatable`、`Hashable` 和 `Comparable` 协议的实现。在使用这些合成实现之后，无需再编写重复的样板代码来实现这些协议所要求的方法。

<!--
  Linking directly to a section of an article like the URLs below do
  is expected to be stable --
  as long as the section stays around, that topic ID will be there too.

  Conforming to the Equatable Protocol
  https://developer.apple.com/documentation/swift/equatable#2847780

  Conforming to the Hashable Protocol
  https://developer.apple.com/documentation/swift/hashable#2849490

  Conforming to the Comparable Protocol
  https://developer.apple.com/documentation/swift/comparable#2845320

  ^-- Need to add discussion of synthesized implementation
  to the reference for Comparable, since that's new

  Some of the information in the type references above
  is also repeated in the "Conform Automatically to Equatable and Hashable" section
  of the article "Adopting Common Protocols".
  https://developer.apple.com/documentation/swift/adopting_common_protocols#2991123
-->

Swift 为以下几种自定义类型提供了 `Equatable` 协议的合成实现：

- 只包含遵循 `Equatable` 协议的存储属性的结构体
- 只包含遵循 `Equatable` 协议的关联类型的枚举
- 没有任何关联类型的枚举

在包含类型原始声明的文件中声明对 `Equatable` 协议的遵循，可以得到 `==` 操作符的合成实现，且无需自己编写任何关于 `==` 的实现代码。`Equatable` 协议同时包含 `!=` 操作符的默认实现。

下面的例子中定义了一个 `Vector3D` 结构体来表示一个类似 `Vector2D` 的三维向量 `(x, y, z)`。由于 `x`、`y` 和 `z` 都是满足 `Equatable` 的类型，`Vector3D` 可以得到等价运算符的合成实现。

```swift
struct Vector3D: Equatable {
    var x = 0.0, y = 0.0, z = 0.0
}

let twoThreeFour = Vector3D(x: 2.0, y: 3.0, z: 4.0)
let anotherTwoThreeFour = Vector3D(x: 2.0, y: 3.0, z: 4.0)
if twoThreeFour == anotherTwoThreeFour {
    print("These two vectors are also equivalent.")
}
// 打印 "These two vectors are also equivalent."
```

<!--
  - test: `equatable_synthesis`

  ```swifttest
  -> struct Vector3D: Equatable {
        var x = 0.0, y = 0.0, z = 0.0
     }
  ---
  -> let twoThreeFour = Vector3D(x: 2.0, y: 3.0, z: 4.0)
  -> let anotherTwoThreeFour = Vector3D(x: 2.0, y: 3.0, z: 4.0)
  -> if twoThreeFour == anotherTwoThreeFour {
         print("These two vectors are also equivalent.")
     }
  <- These two vectors are also equivalent.
  ```
-->

<!--
  Need to cross reference here from "Adopting Common Protocols"
  https://developer.apple.com/documentation/swift/adopting_common_protocols

  Discussion in the article calls out that
  enums without associated values are Equatable & Hashable
  even if you don't declare the protocol conformance.
-->

Swift 为以下几种自定义类型提供了 `Hashable` 协议的合成实现：

- 只包含遵循 `Hashable` 协议的存储属性的结构体
- 只包含遵循 `Hashable` 协议的关联类型的枚举
- 没有任何关联类型的枚举

在包含类型原始声明的文件中声明对 `Hashable` 协议的遵循，可以得到 `hash(into:)` 的合成实现，且无需自己编写任何关于 `hash(into:)` 的实现代码。

Swift 为没有原始值的枚举类型提供了 `Comparable` 协议的合成实现。如果枚举类型包含关联类型，那这些关联类型也必须同时遵循 `Comparable` 协议。在包含原始枚举类型声明的文件中声明其对 `Comparable` 协议的遵循，可以得到 `<` 操作符的合成实现，且无需自己编写任何关于 `<` 的实现代码。`Comparable` 协议同时包含 `<=`、`>` 和 `>=` 操作符的默认实现。

下面的例子中定义了 `SkillLevel` 枚举类型，其中定义了 beginner（初学者）、intermediate（中级）和 expert（专家）三种类型，专家类型会由额外的 stars（星级）数量来进行排名。

```swift
enum SkillLevel: Comparable {
    case beginner
    case intermediate
    case expert(stars: Int)
}
var levels = [SkillLevel.intermediate, SkillLevel.beginner,
              SkillLevel.expert(stars: 5), SkillLevel.expert(stars: 3)]
for level in levels.sorted() {
    print(level)
}
// 打印 “beginner”
// 打印 “intermediate”
// 打印 “expert(stars: 3)”
// 打印 “expert(stars: 5)”
```

<!--
  - test: `comparable-enum-synthesis`

  ```swifttest
  -> enum SkillLevel: Comparable {
         case beginner
         case intermediate
         case expert(stars: Int)
     }
  -> var levels = [SkillLevel.intermediate, SkillLevel.beginner,
                   SkillLevel.expert(stars: 5), SkillLevel.expert(stars: 3)]
  -> for level in levels.sorted() {
         print(level)
     }
  <- beginner
  <- intermediate
  <- expert(stars: 3)
  <- expert(stars: 5)
  ```
-->

<!--
  The example above iterates and prints instead of printing the whole array
  because printing an array gives you the debug description of each element,
  which looks like temp123908.SkillLevel.expert(5) -- not nice to read.
-->

<!--
  - test: `no-synthesized-comparable-for-raw-value-enum`

  ```swifttest
  >> enum E: Int, Comparable {
  >>     case ten = 10
  >>     case twelve = 12
  >> }
  !$ error: type 'E' does not conform to protocol 'Comparable'
  !! enum E: Int, Comparable {
  !!      ^
  !$ note: enum declares raw type 'Int', preventing synthesized conformance of 'E' to 'Comparable'
  !! enum E: Int, Comparable {
  !!         ^
  !$ note: candidate would match if 'E' conformed to 'FloatingPoint'
  !! public static func < (lhs: Self, rhs: Self) -> Bool
  !!                        ^
  !$ note: candidate has non-matching type '<Self, Other> (Self, Other) -> Bool'
  !! public static func < <Other>(lhs: Self, rhs: Other) -> Bool where Other : BinaryInteger
  !!                        ^
  !$ note: candidate would match if 'E' conformed to '_Pointer'
  !! public static func < (lhs: Self, rhs: Self) -> Bool
  !!                        ^
  !$ note: candidate would match if 'E' conformed to '_Pointer'
  !! @inlinable public static func < <Other>(lhs: Self, rhs: Other) -> Bool where Other : _Pointer
  !!                                   ^
  !$ note: candidate has non-matching type '<Self> (Self, Self) -> Bool'
  !! @inlinable public static func < (x: Self, y: Self) -> Bool
  !!                                   ^
  !$ note: candidate would match if 'E' conformed to 'StringProtocol'
  !! @inlinable public static func < <RHS>(lhs: Self, rhs: RHS) -> Bool where RHS : StringProtocol
  !!                                   ^
  !$ note: protocol requires function '<' with type '(E, E) -> Bool'
  !! static func < (lhs: Self, rhs: Self) -> Bool
  !!                 ^
  ```
-->

## 协议类型的集合

协议类型可以在数组或者字典这样的集合中使用，在 <doc:Protocols#Protocols-as-Types> 提到了这样的用法。下面的例子创建了一个元素类型为 `TextRepresentable` 的数组：

```swift
let things: [TextRepresentable] = [game, d12, simonTheHamster]
```

<!--
  - test: `protocols`

  ```swifttest
  -> let things: [TextRepresentable] = [game, d12, simonTheHamster]
  ```
-->

现在可以遍历 `things` 数组，并打印每个元素的文本表示：

```swift
for thing in things {
    print(thing.textualDescription)
}
// A game of Snakes and Ladders with 25 squares
// A 12-sided dice
// A hamster named Simon
```

<!--
  - test: `protocols`

  ```swifttest
  -> for thing in things {
        print(thing.textualDescription)
     }
  </ A game of Snakes and Ladders with 25 squares
  </ A 12-sided dice
  </ A hamster named Simon
  ```
-->

注意 `thing` 常量是 `TextRepresentable` 类型而不是 `Dice`，`DiceGame`，`Hamster` 等类型，即使实例在幕后确实是这些类型中的一种。由于 `thing` 是 `TextRepresentable` 类型，任何 `TextRepresentable` 的实例都有一个 `textualDescription` 属性，所以在每次循环中可以安全地访问 `thing.textualDescription`。

## 协议的继承

协议能够*继承（inherit）*一个或多个其他协议，可以在继承的协议的基础上增加新的要求。协议的继承语法与类的继承相似，多个被继承的协议间用逗号分隔：

```swift
protocol InheritingProtocol: SomeProtocol, AnotherProtocol {
    // 这里是协议的定义部分
}
```

<!--
  - test: `protocols`

  ```swifttest
  >> protocol SomeProtocol {}
  >> protocol AnotherProtocol {}
  -> protocol InheritingProtocol: SomeProtocol, AnotherProtocol {
        // protocol definition goes here
     }
  ```
-->

如下所示，`PrettyTextRepresentable` 协议继承了上面提到的 `TextRepresentable` 协议：

```swift
protocol PrettyTextRepresentable: TextRepresentable {
    var prettyTextualDescription: String { get }
}
```

<!--
  - test: `protocols`

  ```swifttest
  -> protocol PrettyTextRepresentable: TextRepresentable {
        var prettyTextualDescription: String { get }
     }
  ```
-->

例子中定义了一个新的协议 `PrettyTextRepresentable`，它继承自 `TextRepresentable` 协议。任何遵循 `PrettyTextRepresentable` 协议的类型，除了必须满足 `TextRepresentable` 协议的要求，*还要*额外满足 `PrettyTextRepresentable` 协议的要求。在这个例子中，`PrettyTextRepresentable` 协议额外要求遵循协议的类型提供一个返回值为 `String` 类型的 `prettyTextualDescription` 属性。

如下所示，扩展 `SnakesAndLadders`，使其遵循并符合 `PrettyTextRepresentable` 协议：

```swift
extension SnakesAndLadders: PrettyTextRepresentable {
    var prettyTextualDescription: String {
        var output = textualDescription + ":\n"
        for index in 1...finalSquare {
            switch board[index] {
            case let ladder where ladder > 0:
                output += "▲ "
            case let snake where snake < 0:
                output += "▼ "
            default:
                output += "○ "
            }
        }
        return output
    }
}
```

<!--
  - test: `protocols`

  ```swifttest
  -> extension SnakesAndLadders: PrettyTextRepresentable {
        var prettyTextualDescription: String {
           var output = textualDescription + ":\n"
           for index in 1...finalSquare {
              switch board[index] {
                 case let ladder where ladder > 0:
                    output += "▲ "
                 case let snake where snake < 0:
                    output += "▼ "
                 default:
                    output += "○ "
              }
           }
           return output
        }
     }
  ```
-->

上述扩展令 `SnakesAndLadders` 遵循了 `PrettyTextRepresentable` 协议，并提供了协议要求的 `prettyTextualDescription` 属性。每个 `PrettyTextRepresentable` 类型同时也是 `TextRepresentable` 类型，所以在 `prettyTextualDescription` 的实现中，可以访问 `textualDescription` 属性，然后拼接上冒号和换行符，接着遍历数组中的元素，拼接一个几何图形来表示每个棋盘方格的内容：

- 当从数组中取出的元素的值大于 `0` 时，用 `▲` 表示。
- 当从数组中取出的元素的值小于 `0` 时，用 `▼` 表示。
- 当从数组中取出的元素的值等于 `0` 时，用 `○` 表示。

任意 `SankesAndLadders` 的实例都可以使用 `prettyTextualDescription` 属性来打印一个漂亮的文本描述：

```swift
print(game.prettyTextualDescription)
// 一个有 25 个方格的蛇梯棋（Snakes and Ladders）游戏:
// ○ ○ ▲ ○ ○ ▲ ○ ○ ▲ ▲ ○ ○ ○ ▼ ○ ○ ○ ○ ▼ ○ ○ ▼ ○ ▼ ○
```

<!--
  - test: `protocols`

  ```swifttest
  -> print(game.prettyTextualDescription)
  </ A game of Snakes and Ladders with 25 squares:
  </ ○ ○ ▲ ○ ○ ▲ ○ ○ ▲ ▲ ○ ○ ○ ▼ ○ ○ ○ ○ ▼ ○ ○ ▼ ○ ▼ ○
  ```
-->

## Class-Only Protocols

You can limit protocol adoption to class types (and not structures or enumerations)
by adding the `AnyObject` protocol to a protocol's inheritance list.

```swift
protocol SomeClassOnlyProtocol: AnyObject, SomeInheritedProtocol {
    // class-only protocol definition goes here
}
```

<!--
  - test: `classOnlyProtocols`

  ```swifttest
  >> protocol SomeInheritedProtocol {}
  -> protocol SomeClassOnlyProtocol: AnyObject, SomeInheritedProtocol {
        // class-only protocol definition goes here
     }
  ```
-->

In the example above, `SomeClassOnlyProtocol` can only be adopted by class types.
It's a compile-time error to write a structure or enumeration definition
that tries to adopt `SomeClassOnlyProtocol`.

> Note: Use a class-only protocol when the behavior defined by that protocol's requirements
> assumes or requires that a conforming type has
> reference semantics rather than value semantics.
> For more about reference and value semantics,
> see <doc:ClassesAndStructures#Structures-and-Enumerations-Are-Value-Types>
> and <doc:ClassesAndStructures#Classes-Are-Reference-Types>.

<!--
  - test: `anyobject-doesn't-have-to-be-first`

  ```swifttest
  >> protocol SomeInheritedProtocol {}
  -> protocol SomeClassOnlyProtocol: SomeInheritedProtocol, AnyObject {
        // class-only protocol definition goes here
     }
  ```
-->

<!--
  TODO: a Cacheable protocol might make a good example here?
-->

## Protocol Composition

It can be useful to require a type to conform to multiple protocols at the same time.
You can combine multiple protocols into a single requirement
with a *protocol composition*.
Protocol compositions behave as if you
defined a temporary local protocol that has the combined requirements
of all protocols in the composition.
Protocol compositions don't define any new protocol types.

Protocol compositions have the form `SomeProtocol & AnotherProtocol`.
You can list as many protocols as you need,
separating them with ampersands (`&`).
In addition to its list of protocols,
a protocol composition can also contain one class type,
which you can use to specify a required superclass.

Here's an example that combines two protocols called `Named` and `Aged`
into a single protocol composition requirement on a function parameter:

```swift
protocol Named {
    var name: String { get }
}
protocol Aged {
    var age: Int { get }
}
struct Person: Named, Aged {
    var name: String
    var age: Int
}
func wishHappyBirthday(to celebrator: Named & Aged) {
    print("Happy birthday, \(celebrator.name), you're \(celebrator.age)!")
}
let birthdayPerson = Person(name: "Malcolm", age: 21)
wishHappyBirthday(to: birthdayPerson)
// Prints "Happy birthday, Malcolm, you're 21!"
```

<!--
  - test: `protocolComposition`

  ```swifttest
  -> protocol Named {
        var name: String { get }
     }
  -> protocol Aged {
        var age: Int { get }
     }
  -> struct Person: Named, Aged {
        var name: String
        var age: Int
     }
  -> func wishHappyBirthday(to celebrator: Named & Aged) {
        print("Happy birthday, \(celebrator.name), you're \(celebrator.age)!")
     }
  -> let birthdayPerson = Person(name: "Malcolm", age: 21)
  -> wishHappyBirthday(to: birthdayPerson)
  <- Happy birthday, Malcolm, you're 21!
  ```
-->

In this example,
the `Named` protocol
has a single requirement for a gettable `String` property called `name`.
The `Aged` protocol
has a single requirement for a gettable `Int` property called `age`.
Both protocols are adopted by a structure called `Person`.

The example also defines a `wishHappyBirthday(to:)` function.
The type of the `celebrator` parameter is `Named & Aged`,
which means “any type that conforms to both the `Named` and `Aged` protocols.”
It doesn't matter which specific type is passed to the function,
as long as it conforms to both of the required protocols.

The example then creates a new `Person` instance called `birthdayPerson`
and passes this new instance to the `wishHappyBirthday(to:)` function.
Because `Person` conforms to both protocols, this call is valid,
and the `wishHappyBirthday(to:)` function can print its birthday greeting.

Here's an example that combines
the `Named` protocol from the previous example
with a `Location` class:

```swift
class Location {
    var latitude: Double
    var longitude: Double
    init(latitude: Double, longitude: Double) {
        self.latitude = latitude
        self.longitude = longitude
    }
}
class City: Location, Named {
    var name: String
    init(name: String, latitude: Double, longitude: Double) {
        self.name = name
        super.init(latitude: latitude, longitude: longitude)
    }
}
func beginConcert(in location: Location & Named) {
    print("Hello, \(location.name)!")
}

let seattle = City(name: "Seattle", latitude: 47.6, longitude: -122.3)
beginConcert(in: seattle)
// Prints "Hello, Seattle!"
```

<!--
  - test: `protocolComposition`

  ```swifttest
  -> class Location {
         var latitude: Double
         var longitude: Double
         init(latitude: Double, longitude: Double) {
             self.latitude = latitude
             self.longitude = longitude
         }
     }
  -> class City: Location, Named {
         var name: String
         init(name: String, latitude: Double, longitude: Double) {
             self.name = name
             super.init(latitude: latitude, longitude: longitude)
         }
     }
  -> func beginConcert(in location: Location & Named) {
         print("Hello, \(location.name)!")
     }
  ---
  -> let seattle = City(name: "Seattle", latitude: 47.6, longitude: -122.3)
  -> beginConcert(in: seattle)
  <- Hello, Seattle!
  ```
-->

The `beginConcert(in:)` function takes
a parameter of type `Location & Named`,
which means "any type that's a subclass of `Location`
and that conforms to the `Named` protocol."
In this case, `City` satisfies both requirements.

Passing `birthdayPerson` to the `beginConcert(in:)` function
is invalid because `Person` isn't a subclass of `Location`.
Likewise,
if you made a subclass of `Location`
that didn't conform to the `Named` protocol,
calling `beginConcert(in:)` with an instance of that type
is also invalid.

## Checking for Protocol Conformance

You can use the `is` and `as` operators described in <doc:TypeCasting>
to check for protocol conformance, and to cast to a specific protocol.
Checking for and casting to a protocol
follows exactly the same syntax as checking for and casting to a type:

- The `is` operator returns `true` if an instance conforms to a protocol
  and returns `false` if it doesn't.
- The `as?` version of the downcast operator returns
  an optional value of the protocol's type,
  and this value is `nil` if the instance doesn't conform to that protocol.
- The `as!` version of the downcast operator forces the downcast to the protocol type
  and triggers a runtime error if the downcast doesn't succeed.

This example defines a protocol called `HasArea`,
with a single property requirement of a gettable `Double` property called `area`:

```swift
protocol HasArea {
    var area: Double { get }
}
```

<!--
  - test: `protocolConformance`

  ```swifttest
  -> protocol HasArea {
        var area: Double { get }
     }
  ```
-->

Here are two classes, `Circle` and `Country`,
both of which conform to the `HasArea` protocol:

```swift
class Circle: HasArea {
    let pi = 3.1415927
    var radius: Double
    var area: Double { return pi * radius * radius }
    init(radius: Double) { self.radius = radius }
}
class Country: HasArea {
    var area: Double
    init(area: Double) { self.area = area }
}
```

<!--
  - test: `protocolConformance`

  ```swifttest
  -> class Circle: HasArea {
        let pi = 3.1415927
        var radius: Double
        var area: Double { return pi * radius * radius }
        init(radius: Double) { self.radius = radius }
     }
  -> class Country: HasArea {
        var area: Double
        init(area: Double) { self.area = area }
     }
  ```
-->

The `Circle` class implements the `area` property requirement
as a computed property, based on a stored `radius` property.
The `Country` class implements the `area` requirement directly as a stored property.
Both classes correctly conform to the `HasArea` protocol.

Here's a class called `Animal`, which doesn't conform to the `HasArea` protocol:

```swift
class Animal {
    var legs: Int
    init(legs: Int) { self.legs = legs }
}
```

<!--
  - test: `protocolConformance`

  ```swifttest
  -> class Animal {
        var legs: Int
        init(legs: Int) { self.legs = legs }
     }
  ```
-->

The `Circle`, `Country` and `Animal` classes don't have a shared base class.
Nonetheless, they're all classes, and so instances of all three types
can be used to initialize an array that stores values of type `AnyObject`:

```swift
let objects: [AnyObject] = [
    Circle(radius: 2.0),
    Country(area: 243_610),
    Animal(legs: 4)
]
```

<!--
  - test: `protocolConformance`

  ```swifttest
  -> let objects: [AnyObject] = [
        Circle(radius: 2.0),
        Country(area: 243_610),
        Animal(legs: 4)
     ]
  ```
-->

The `objects` array is initialized with an array literal containing
a `Circle` instance with a radius of 2 units;
a `Country` instance initialized with
the surface area of the United Kingdom in square kilometers;
and an `Animal` instance with four legs.

The `objects` array can now be iterated,
and each object in the array can be checked to see if
it conforms to the `HasArea` protocol:

```swift
for object in objects {
    if let objectWithArea = object as? HasArea {
        print("Area is \(objectWithArea.area)")
    } else {
        print("Something that doesn't have an area")
    }
}
// Area is 12.5663708
// Area is 243610.0
// Something that doesn't have an area
```

<!--
  - test: `protocolConformance`

  ```swifttest
  -> for object in objects {
        if let objectWithArea = object as? HasArea {
           print("Area is \(objectWithArea.area)")
        } else {
           print("Something that doesn't have an area")
        }
     }
  </ Area is 12.5663708
  </ Area is 243610.0
  </ Something that doesn't have an area
  ```
-->

Whenever an object in the array conforms to the `HasArea` protocol,
the optional value returned by the `as?` operator is unwrapped with optional binding
into a constant called `objectWithArea`.
The `objectWithArea` constant is known to be of type `HasArea`,
and so its `area` property can be accessed and printed in a type-safe way.

Note that the underlying objects aren't changed by the casting process.
They continue to be a `Circle`, a `Country` and an `Animal`.
However, at the point that they're stored in the `objectWithArea` constant,
they're only known to be of type `HasArea`,
and so only their `area` property can be accessed.

<!--
  TODO: This is an *extremely* contrived example.
  Also, it's not particularly useful to be able to get the area of these two objects,
  because there's no shared unit system.
  Also also, I'd say that a circle should probably be a structure, not a class.
  Plus, I'm having to write lots of boilerplate initializers,
  which make the example far less focused than I'd like.
  The problem is, I can't use strings within an @objc protocol
  without also having to import Foundation, so it's numbers or bust, I'm afraid.
-->

<!--
  TODO: Since the restrictions on @objc of the previous TODO are now lifted,
  Should the previous examples be revisited?
-->

## Optional Protocol Requirements

<!--
  TODO: split this section into several subsections as per [Contributor 7746]'s feedback,
  and cover the missing alternative approaches that he mentioned.
-->

<!--
  TODO: you can specify optional subscripts,
  and the way you check for them / work with them is a bit esoteric.
  You have to try and access a value from the subscript,
  and see if the value you get back (which will be an optional)
  has a value or is nil.
-->

You can define *optional requirements* for protocols.
These requirements don't have to be implemented by types that conform to the protocol.
Optional requirements are prefixed by the `optional` modifier
as part of the protocol's definition.
Optional requirements are available so that you can write code
that interoperates with Objective-C.
Both the protocol and the optional requirement
must be marked with the `@objc` attribute.
Note that `@objc` protocols can be adopted only by classes,
not by structures or enumerations.

When you use a method or property in an optional requirement,
its type automatically becomes an optional.
For example,
a method of type `(Int) -> String` becomes `((Int) -> String)?`.
Note that the entire function type
is wrapped in the optional,
not the method's return value.

An optional protocol requirement can be called with optional chaining,
to account for the possibility that the requirement was not implemented
by a type that conforms to the protocol.
You check for an implementation of an optional method
by writing a question mark after the name of the method when it's called,
such as `someOptionalMethod?(someArgument)`.
For information on optional chaining, see <doc:OptionalChaining>.

The following example defines an integer-counting class called `Counter`,
which uses an external data source to provide its increment amount.
This data source is defined by the `CounterDataSource` protocol,
which has two optional requirements:

```swift
@objc protocol CounterDataSource {
    @objc optional func increment(forCount count: Int) -> Int
    @objc optional var fixedIncrement: Int { get }
}
```

<!--
  - test: `protocolConformance`

  ```swifttest
  >> import Foundation
  -> @objc protocol CounterDataSource {
  ->    @objc optional func increment(forCount count: Int) -> Int
  ->    @objc optional var fixedIncrement: Int { get }
  -> }
  ```
-->

The `CounterDataSource` protocol defines
an optional method requirement called `increment(forCount:)`
and an optional property requirement called `fixedIncrement`.
These requirements define two different ways for data sources to provide
an appropriate increment amount for a `Counter` instance.

> Note: Strictly speaking, you can write a custom class
> that conforms to `CounterDataSource` without implementing
> *either* protocol requirement.
> They're both optional, after all.
> Although technically allowed, this wouldn't make for a very good data source.

The `Counter` class, defined below,
has an optional `dataSource` property of type `CounterDataSource?`:

```swift
class Counter {
    var count = 0
    var dataSource: CounterDataSource?
    func increment() {
        if let amount = dataSource?.increment?(forCount: count) {
            count += amount
        } else if let amount = dataSource?.fixedIncrement {
            count += amount
        }
    }
}
```

<!--
  - test: `protocolConformance`

  ```swifttest
  -> class Counter {
        var count = 0
        var dataSource: CounterDataSource?
        func increment() {
           if let amount = dataSource?.increment?(forCount: count) {
              count += amount
           } else if let amount = dataSource?.fixedIncrement {
              count += amount
           }
        }
     }
  ```
-->

The `Counter` class stores its current value in a variable property called `count`.
The `Counter` class also defines a method called `increment`,
which increments the `count` property every time the method is called.

The `increment()` method first tries to retrieve an increment amount
by looking for an implementation of the `increment(forCount:)` method on its data source.
The `increment()` method uses optional chaining to try to call `increment(forCount:)`,
and passes the current `count` value as the method's single argument.

Note that *two* levels of optional chaining are at play here.
First, it's possible that `dataSource` may be `nil`,
and so `dataSource` has a question mark after its name to indicate that
`increment(forCount:)` should be called only if `dataSource` isn't `nil`.
Second, even if `dataSource` *does* exist,
there's no guarantee that it implements `increment(forCount:)`,
because it's an optional requirement.
Here, the possibility that `increment(forCount:)` might not be implemented
is also handled by optional chaining.
The call to `increment(forCount:)` happens
only if `increment(forCount:)` exists ---
that is, if it isn't `nil`.
This is why `increment(forCount:)` is also written with a question mark after its name.

Because the call to `increment(forCount:)` can fail for either of these two reasons,
the call returns an *optional* `Int` value.
This is true even though `increment(forCount:)` is defined as returning
a non-optional `Int` value in the definition of `CounterDataSource`.
Even though there are two optional chaining operations,
one after another,
the result is still wrapped in a single optional.
For more information about using multiple optional chaining operations,
see <doc:OptionalChaining#Linking-Multiple-Levels-of-Chaining>.

After calling `increment(forCount:)`, the optional `Int` that it returns
is unwrapped into a constant called `amount`, using optional binding.
If the optional `Int` does contain a value ---
that is, if the delegate and method both exist,
and the method returned a value ---
the unwrapped `amount` is added onto the stored `count` property,
and incrementation is complete.

If it's *not* possible to retrieve a value from the `increment(forCount:)` method ---
either because `dataSource` is nil,
or because the data source doesn't implement `increment(forCount:)` ---
then the `increment()` method tries to retrieve a value
from the data source's `fixedIncrement` property instead.
The `fixedIncrement` property is also an optional requirement,
so its value is an optional `Int` value,
even though `fixedIncrement` is defined as a non-optional `Int` property
as part of the `CounterDataSource` protocol definition.

Here's a simple `CounterDataSource` implementation where the data source
returns a constant value of `3` every time it's queried.
It does this by implementing the optional `fixedIncrement` property requirement:

```swift
class ThreeSource: NSObject, CounterDataSource {
    let fixedIncrement = 3
}
```

<!--
  - test: `protocolConformance`

  ```swifttest
  -> class ThreeSource: NSObject, CounterDataSource {
        let fixedIncrement = 3
     }
  ```
-->

You can use an instance of `ThreeSource` as the data source for a new `Counter` instance:

```swift
var counter = Counter()
counter.dataSource = ThreeSource()
for _ in 1...4 {
    counter.increment()
    print(counter.count)
}
// 3
// 6
// 9
// 12
```

<!--
  - test: `protocolConformance`

  ```swifttest
  -> var counter = Counter()
  -> counter.dataSource = ThreeSource()
  -> for _ in 1...4 {
        counter.increment()
        print(counter.count)
     }
  </ 3
  </ 6
  </ 9
  </ 12
  ```
-->

The code above creates a new `Counter` instance;
sets its data source to be a new `ThreeSource` instance;
and calls the counter's `increment()` method four times.
As expected, the counter's `count` property increases by three
each time `increment()` is called.

Here's a more complex data source called `TowardsZeroSource`,
which makes a `Counter` instance count up or down towards zero
from its current `count` value:

```swift
class TowardsZeroSource: NSObject, CounterDataSource {
    func increment(forCount count: Int) -> Int {
        if count == 0 {
            return 0
        } else if count < 0 {
            return 1
        } else {
            return -1
        }
    }
}
```

<!--
  - test: `protocolConformance`

  ```swifttest
  -> class TowardsZeroSource: NSObject, CounterDataSource {
        func increment(forCount count: Int) -> Int {
           if count == 0 {
              return 0
           } else if count < 0 {
              return 1
           } else {
              return -1
           }
        }
     }
  ```
-->

The `TowardsZeroSource` class implements
the optional `increment(forCount:)` method from the `CounterDataSource` protocol
and uses the `count` argument value to work out which direction to count in.
If `count` is already zero, the method returns `0`
to indicate that no further counting should take place.

You can use an instance of `TowardsZeroSource` with the existing `Counter` instance
to count from `-4` to zero.
Once the counter reaches zero, no more counting takes place:

```swift
counter.count = -4
counter.dataSource = TowardsZeroSource()
for _ in 1...5 {
    counter.increment()
    print(counter.count)
}
// -3
// -2
// -1
// 0
// 0
```

<!--
  - test: `protocolConformance`

  ```swifttest
  -> counter.count = -4
  -> counter.dataSource = TowardsZeroSource()
  -> for _ in 1...5 {
        counter.increment()
        print(counter.count)
     }
  </ -3
  </ -2
  </ -1
  </ 0
  </ 0
  ```
-->

## Protocol Extensions

Protocols can be extended to provide method,
initializer, subscript, and computed property implementations
to conforming types.
This allows you to define behavior on protocols themselves,
rather than in each type's individual conformance or in a global function.

For example, the `RandomNumberGenerator` protocol can be extended
to provide a `randomBool()` method,
which uses the result of the required `random()` method
to return a random `Bool` value:

```swift
extension RandomNumberGenerator {
    func randomBool() -> Bool {
        return random() > 0.5
    }
}
```

<!--
  - test: `protocols`

  ```swifttest
  -> extension RandomNumberGenerator {
        func randomBool() -> Bool {
           return random() > 0.5
        }
     }
  ```
-->

By creating an extension on the protocol,
all conforming types automatically gain this method implementation
without any additional modification.

```swift
let generator = LinearCongruentialGenerator()
print("Here's a random number: \(generator.random())")
// Prints "Here's a random number: 0.3746499199817101"
print("And here's a random Boolean: \(generator.randomBool())")
// Prints "And here's a random Boolean: true"
```

<!--
  - test: `protocols`

  ```swifttest
  >> do {
  -> let generator = LinearCongruentialGenerator()
  -> print("Here's a random number: \(generator.random())")
  <- Here's a random number: 0.3746499199817101
  -> print("And here's a random Boolean: \(generator.randomBool())")
  <- And here's a random Boolean: true
  >> }
  ```
-->

<!--
  The extra scope in the above test code allows this 'generator' variable to shadow
  the variable that already exists from a previous testcode block.
-->

Protocol extensions can add implementations to conforming types
but can't make a protocol extend or inherit from another protocol.
Protocol inheritance is always specified in the protocol declaration itself.

### Providing Default Implementations

You can use protocol extensions to provide a default implementation
to any method or computed property requirement of that protocol.
If a conforming type provides its own implementation of a required method or property,
that implementation will be used instead of the one provided by the extension.

> Note: Protocol requirements with default implementations provided by extensions
> are distinct from optional protocol requirements.
> Although conforming types don't have to provide their own implementation of either,
> requirements with default implementations can be called without optional chaining.

For example, the `PrettyTextRepresentable` protocol,
which inherits the `TextRepresentable` protocol
can provide a default implementation of its required `prettyTextualDescription` property
to simply return the result of accessing the `textualDescription` property:

```swift
extension PrettyTextRepresentable  {
    var prettyTextualDescription: String {
        return textualDescription
    }
}
```

<!--
  - test: `protocols`

  ```swifttest
  -> extension PrettyTextRepresentable  {
        var prettyTextualDescription: String {
           return textualDescription
        }
     }
  ```
-->

<!--
  TODO <rdar://problem/32211512> TSPL: Explain when you can/can't override a protocol default implementation
-->

<!--
  If something is a protocol requirement,
  types that conform to the protocol can override the default implementation.
-->

<!--
  If something isn't a requirement,
  you get wonky behavior when you try to override the default implementation.
-->

<!--
  If the static type is the conforming type,
  your override is used.
-->

<!--
  If the static type is the protocol type,
  the default implementation is used.
-->

<!--
  You can't write ``final`` on a default implementation
  to prevent someone from overriding it in a conforming type.
-->

### Adding Constraints to Protocol Extensions

When you define a protocol extension,
you can specify constraints that conforming types
must satisfy before the methods and properties of the extension are available.
You write these constraints after the name of the protocol you're extending
by writing a generic `where` clause.
For more about generic `where` clauses, see <doc:Generics#Generic-Where-Clauses>.

For example,
you can define an extension to the `Collection` protocol
that applies to any collection whose elements conform
to the `Equatable` protocol.
By constraining a collection's elements to the `Equatable` protocol,
a part of the Swift standard library,
you can use the `==` and `!=` operators to check for equality and inequality between two elements.

```swift
extension Collection where Element: Equatable {
    func allEqual() -> Bool {
        for element in self {
            if element != self.first {
                return false
            }
        }
        return true
    }
}
```

<!--
  - test: `protocols`

  ```swifttest
  -> extension Collection where Element: Equatable {
         func allEqual() -> Bool {
             for element in self {
                 if element != self.first {
                     return false
                 }
             }
             return true
         }
     }
  ```
-->

The `allEqual()` method returns `true`
only if all the elements in the collection are equal.

Consider two arrays of integers,
one where all the elements are the same,
and one where they aren't:

```swift
let equalNumbers = [100, 100, 100, 100, 100]
let differentNumbers = [100, 100, 200, 100, 200]
```

<!--
  - test: `protocols`

  ```swifttest
  -> let equalNumbers = [100, 100, 100, 100, 100]
  -> let differentNumbers = [100, 100, 200, 100, 200]
  ```
-->

Because arrays conform to `Collection`
and integers conform to `Equatable`,
`equalNumbers` and `differentNumbers` can use the `allEqual()` method:

```swift
print(equalNumbers.allEqual())
// Prints "true"
print(differentNumbers.allEqual())
// Prints "false"
```

<!--
  - test: `protocols`

  ```swifttest
  -> print(equalNumbers.allEqual())
  <- true
  -> print(differentNumbers.allEqual())
  <- false
  ```
-->

> Note: If a conforming type satisfies the requirements for multiple constrained extensions
> that provide implementations for the same method or property,
> Swift uses the implementation corresponding to the most specialized constraints.

<!--
  TODO: It would be great to pull this out of a note,
  but we should wait until we have a better narrative that shows how this
  works with some examples.
-->

<!--
  TODO: Other things to be included
  ---------------------------------
  Class-only protocols
  Protocols marked @objc
  Standard-library protocols such as Sequence, Equatable etc.?
  Show how to make a custom type conform to Boolean or some other protocol
  Show a protocol being used by an enumeration
  accessing protocol methods, properties etc.  through a constant or variable that's *just* of protocol type
  Protocols can't be nested, but nested types can implement protocols
  Protocol requirements can be marked as @unavailable, but this currently only works if they're also marked as @objc.
  Checking for (and calling) optional implementations via optional binding and closures
-->

> Beta Software:
>
> This documentation contains preliminary information about an API or technology in development. This information is subject to change, and software implemented according to this documentation should be tested with final operating system software.
>
> Learn more about using [Apple's beta software](https://developer.apple.com/support/beta-software/).

<!--
This source file is part of the Swift.org open source project

Copyright (c) 2014 - 2022 Apple Inc. and the Swift project authors
Licensed under Apache License v2.0 with Runtime Library Exception

See https://swift.org/LICENSE.txt for license information
See https://swift.org/CONTRIBUTORS.txt for the list of Swift project authors
-->
