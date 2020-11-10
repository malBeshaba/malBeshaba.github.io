---
title: SwiftUI(一)——第一个项目
date: 2020-06-24 16:43:25
tags: [iOS,swift,SwiftUI]
---

## 创建项目

<img src="https://ae01.alicdn.com/kf/H70fd90f02eaf4f10a313dd9b96fc6b66W.png" alt="image.png" style="zoom:50%;" />

唯一与我们从前使用storyboard不同的是，用户界面我们要选择用SwiftUI框架实现



## 初始文件

- AppDelegate.swift包含用于管理您的应用程序的代码。在这里添加代码曾经很常见，但如今却很少见。
- SceneDelegate.swift包含用于在应用程序中启动一个窗口的代码。这在iPhone上没有多大作用，但在iPad上（用户可以同时打开您的应用的多个实例），这很重要。
- ContentView.swift包含您程序的初始用户界面（UI），并且我们将在该项目中完成所有工作。
- Assets.xcassets是*资产目录* –您要在应用程序中使用的图片的集合。您还可以在此处添加颜色，以及应用程序图标，iMessage贴纸等。
- LaunchScreen.storyboard是一个可视化编辑器，用于创建一小段UI，以在应用程序启动时显示。
- Info.plist是一个特殊值的集合，这些特殊值向系统描述了您的应用程序的工作方式–它是哪个版本，您支持的设备方向等等。不是代码，但仍然很重要的事情。
- 预览内容是一个黄色的组，其中包含Preview Assets.xcassets –这是另一个资产目录，这一次专门针对例如您在设计用户界面时要使用的图像，以使您了解它们在外观时的外观该程序正在运行。

<!-- more -->

## 创建表单

许多应用程序要求用户输入某种输入-可能是要求他们设置一些首选项，可能是要求他们确认想要汽车接载的位置，或者是从菜单中订购食物，或者执行其他任何操作类似。

SwiftUI为此提供了专用的视图类型，称为`Form`。表单是静态控件（如文本和图像）的滚动列表，但也可以包括用户交互控件（如文本字段，切换开关，按钮等）。

我们可以通过将默认文本视图包装在内部来创建基本表单`Form`，如下所示：

```swift
var body: some View {
    Form {
        Text("Hello World")
    }
}
```

实际上，您可以在表单中拥有任意数量的内容，尽管如果您打算添加10个以上的SwiftUI，则需要将它们成组放置以避免出现问题。

例如，此代码显示十行文本就好了：

```swift
Form {
    Text("Hello World")
    Text("Hello World")
    Text("Hello World")
    Text("Hello World")
    Text("Hello World")
    Text("Hello World")
    Text("Hello World")
    Text("Hello World")
    Text("Hello World")
    Text("Hello World")
}
```

但这尝试显示11，这是不允许的：

```swift
Form {
    Text("Hello World")
    Text("Hello World")
    Text("Hello World")
    Text("Hello World")
    Text("Hello World")
    Text("Hello World")
    Text("Hello World")
    Text("Hello World")
    Text("Hello World")
    Text("Hello World")
    Text("Hello World")
}
```

**提示：**如果您好奇为什么允许10行但不允许11行，则这是SwiftUI的局限性：它的编码方式是了解如何向表单添加一件事，如何向表单添加两件事，如何添加三件事，四件事，五件事等等，一直到十个，但不能超过–他们需要在某处画一条线。父级中的10个子成员的限制实际上适用于SwiftUI中的所有位置。

如果您想在表单中包含11件事，则应在内放置一些行`Group`：

```swift
Form {
    Group {
        Text("Hello World")
        Text("Hello World")
        Text("Hello World")
        Text("Hello World")
        Text("Hello World")
        Text("Hello World")
    }

    Group {
        Text("Hello World")
        Text("Hello World")
        Text("Hello World")
        Text("Hello World")
        Text("Hello World")
    }
}
```

组实际上并没有改变用户界面的外观，它们只是让我们解决了SwiftUI在父级内部限制了十个子视图的局限性–在这种情况下，即表单内的文本视图。

如果*希望*将表单项拆分为大块时看起来不同，则应改用`Section`视图。就像“设置”应用程序一样，这会将您的表单分成离散的可视组：

```swift
Form {
    Section {
        Text("Hello World")
    }

    Section {
        Text("Hello World")
        Text("Hello World")
    }
}
```

将表单划分为多个部分时，没有硬性规定，只是在视觉上将相关项目分组。

## 添加导航栏

您可以通过如下所示的用户界面看到它的运行情况：

```swift
struct ContentView: View {
    var body: some View {
        Form {
            Section {
                Text("Hello World")
            }
        }
    }
}
```

您会看到该表单从槽口下方开始，因此默认情况下，表单中的行是完全可见的。但是，表单也可以滚动，因此，如果您在模拟器中四处滑动，您会发现您可以将行向上移动，使其在时钟下运行，这使得它们都难以阅读。

解决此问题的常用方法是将导航栏放在屏幕顶部。导航栏可以具有标题和按钮，并且在SwiftUI中，它们还使我们能够在用户执行操作时显示新视图。

我们将在以后的项目中使用按钮和新视图，但我至少想向您展示如何添加导航栏并为其命名，因为它使我们的表单在滚动时看起来更好。

您已经看到，可以通过`Section`在文本视图周围添加文本视图来将文本视图放置在部分中，并且可以`Form`通过类似的方式将文本视图放置在区域中。嗯，我们以相同的方式添加了一个导航栏，只是在这里称为`NavigationView`。

```swift
var body: some View {
    NavigationView {
        Form {
            Section {
                Text("Hello World")
            }
        }
    }
}
```

通常，您需要在导航栏中放置某种标题，并且可以通过将*修饰符*附加到放置在其中的任何内容来实现。修饰符是常规方法，但有一点点不同：它们总是返回一个新的实例，无论您使用它们是什么。

让我们尝试添加一个修饰符来设置表单的导航栏标题：

```swift
NavigationView {
    Form {
        Section {
            Text("Hello World")
        }
    }
    .navigationBarTitle(Text("SwiftUI"))
}
```

当我们将`.navigationBarTitle()`修饰符附加到表单时，Swift实际上会创建一个新表单，该表单具有导航栏标题以及您提供的所有现有内容。

将标题添加到导航栏时，您会注意到它为该标题使用了大字体。您可以通过调用以下命令获得小字体`navigationBarTitle()`：

```swift
.navigationBarTitle("SwiftUI", displayMode: .inline)
```

您可以在“设置”应用程序中看到Apple如何使用这些大小标题：第一个屏幕以大文本显示“设置”，随后的屏幕以小文本显示其标题。

因为使用大标题非常普遍，所以可以使用一个快捷方式版本，该版本允许您使用纯字符串而不是文本视图：

```swift
.navigationBarTitle("SwiftUI")
```

## 修改程序状态

在SwiftUI开发人员中，有一种说法是“视图是他们状态的函数”，但这虽然只是少数几个单词，但一开始对您可能毫无意义。

如果您正在玩格斗游戏，则可能会丧命，得一些分，收集一些财宝，甚至可能会捡起一些强大的武器。在编程中，我们称这些为*状态* -描述游戏当前*状态*的有效设置集合。

当您退出游戏时，该状态将被保存，并且当您稍后返回游戏时，您可以重新加载游戏以回到原来的状态。但是，*在*您玩游戏时，这全都叫做*state*：所有整数，字符串，布尔值等等，都存储在RAM中以描述您现在正在做什么。

当我们说SwiftUI的视图是其状态的函数时，是指用户界面的外观（人们可以看到的东西以及可以与之交互的东西）取决于程序的状态。例如，他们只有在文本字段中输入名称后才能点击继续。

这本身听起来似乎很明显，但这实际上与之前使用的替代方案有很大不同：您的用户界面由一系列事件决定。因此，用户现在看到的是因为他们已经使用您的应用程序一段时间了，利用了各种功能，可能已经登录或刷新了数据，等等。

“事件顺序”方法意味着很难存储应用程序的状态，因为获取完美状态的唯一方法是回放用户执行的事件的确切顺序。这就是为什么如此多的应用程序甚至根本不尝试保存状态的原因-您的新闻应用程序将不会返回到您正在阅读的上一篇文章，Twitter不会记住您是否正在键入内容。回复某人，Photoshop会忘记您堆积的任何撤消状态。

让我们通过一个按钮将其付诸实践，该按钮可以在SwiftUI中使用标题字符串和一个动作闭合来创建，该闭合在点击按钮时运行：

```swift
struct ContentView: View {
    var tapCount = 0

    var body: some View {
        Button("Tap Count: \(tapCount)") {
            self.tapCount += 1
        }
    }
}
```

该代码看起来足够合理：创建一个显示“点击计数”的按钮，再加上点击该按钮的次数，然后在`tapCount`每次点击该按钮时加1 。

但是，它不会建立。那不是有效的Swift代码。您看到的`ContentView`是一个结构，可以将其创建为常量。如果回想起当您了解结构时，这意味着它是*不可变的* –我们不能随意更改其值。

例如，在创建要更改属性的结构方法时，我们需要添加`mutating`关键字：`mutating func doSomeWork()`。但是，Swift不允许我们对计算的属性进行突变，这意味着我们不能编写`mutating var body: some View`-只是不允许这样做。

似乎我们陷入了僵局：我们希望能够在程序运行时更改值，但是Swift不允许我们这样做，因为我们的视图是结构。

幸运的是，Swift为我们提供了一个称为*属性包装器*的特殊解决方案：我们可以在属性之前放置一个特殊属性，以有效地赋予它们超能力。在存储简单的程序状态（如点击按钮的次数）的情况下，我们可以使用来自SwiftUI的名为的属性包装器`@State`，如下所示：

```swift
struct ContentView: View {
    @State var tapCount = 0

    var body: some View {
        Button("Tap Count: \(tapCount)") {
            self.tapCount += 1
        }
    }
}
```

这个小小的变化足以使我们的程序正常工作，因此您现在可以构建它并进行尝试。

`@State`允许我们解决结构的局限性：我们知道由于结构是固定的，我们无法更改其属性，但`@State`允许该值由SwiftUI单独存储在*可以*修改的位置。

是的，感觉有点像作弊，您可能想知道为什么我们不使用类，而是*可以*自由修改它们。但是请相信我，这是值得的：随着您的进步，您将了解到SwiftUI经常破坏并重新创建您的结构，因此，保持它们的小巧和简单对结构的性能至关重要。

**提示：**有几种方法可以在SwiftUI中存储程序状态，您将学习所有这些方法。`@State`专为存储在一个视图中的简单属性而设计。因此，Apple建议我们`private`向这些属性添加访问控制，如下所示：`@State private var tapCount = 0`。

## 状态绑定控件

SwiftUI的`@State`属性包装器使我们可以自由地修改视图结构，这意味着随着程序的更改，我们可以更新视图属性以进行匹配。

但是，用户界面控件的操作要复杂一些。例如，如果您想创建一个用户可以键入的可编辑文本框，则可以创建一个SwiftUI视图，如下所示：

```swift
struct ContentView: View {
    var body: some View {
        Form {
            TextField("Enter your name")
            Text("Hello World")
        }
    }
}
```

试图创建一个包含文本字段和文本视图的表单。但是，该代码无法编译，因为SwiftUI想要知道将文本存储在文本字段中的位置。

请记住，视图是状态的函数-文本字段仅在反映了存储在程序中的值的情况下才能显示内容。SwiftUI想要的是结构中的字符串属性，该属性可以显示在文本字段中，并且还可以存储用户在文本字段中键入的内容。

因此，我们可以将代码更改为此：

```swift
struct ContentView: View {
    var name = ""

    var body: some View {
        Form {
            TextField("Enter your name", text: name)
            Text("Hello World")
        }
    }
}
```

添加一个`name`属性，然后使用它创建文本字段。但是，该代码*仍然*无法使用，因为Swift需要能够更新`name`属性以匹配用户在文本字段中键入的内容，因此您可以这样使用`@State`：

```swift
@State private var name = ""
```

但这还不够，我们的代码仍然无法编译。

问题在于，Swift区分“只读”和“读写”*。

对于我们的文本字段，Swift需要确保文本中的任何内容也位于该`name`属性中，以便它可以履行其承诺，即我们的视图是其状态的函数-用户看到的一切仅仅是代码中结构和属性的可见表示。

这就是所谓的*双向绑定*：我们绑定文本字段，以便它显示属性的值，但是我们也绑定它，以便对文本字段的任何更改也可以更新属性。

在Swift中，我们用特殊符号标记这些双向绑定，以便它们脱颖而出：我们在它们之前写一个美元符号。这告诉Swift，它应该读取该属性的值，但还应在发生任何更改时将其写回。

因此，我们的结构的正确版本是这样的：

```swift
struct ContentView: View {
    @State private var name = ""

    var body: some View {
        Form {
            TextField("Enter your name", text: $name)
            Text("Hello World")
        }
    }
}
```

立即尝试运行该代码-您应该会发现可以点击文本字段并输入您的姓名，这与预期的一样。

在继续之前，让我们修改文本视图，以便在其文本字段正下方显示用户名：

```swift
Text("Your name is \(name)")
```

请注意，它`name`是如何使用而不是`$name`？那是因为我们不想在这里进行双向绑定–我们想*读取*值，是的，但是我们不想以某种方式写回它，因为文本视图不会改变。

因此，当您在属性名称之前看到美元符号时，请记住，它会创建双向绑定：既读取属性值，又写入属性值。

## 循环创建视图

想要在一个循环中创建几个SwiftUI视图是很常见的。例如，我们可能想要遍历一系列名称，并让每个名称成为文本视图，或者遍历一系列菜单项，并将每个名称显示为图像。

SwiftUI为此提供了专用的视图类型，称为`ForEach`。这可以遍历数组和范围，并根据需要创建任意数量的视图。更好的是，`ForEach`如果我们手动输入视图，不会受到会影响我们的10视图限制的影响。

`ForEach`将为它循环的每个项目运行一次闭包，并传入当前循环项目。例如，如果我们从0循环到100，它将传入0，然后是1，然后是2，依此类推。

例如，这将创建一个包含100行的表单：

```swift
Form {
    ForEach(0 ..< 100) { number in
        Text("Row \(number)")
    }
}
```

因为`ForEach`传入了闭包，所以我们可以对参数名称使用简写语法，如下所示：

```swift
Form {
    ForEach(0 ..< 100) {
        Text("Row \($0)")
    }
}
```

`ForEach`在使用SwiftUI的`Picker`视图时特别有用，它使我们可以显示各种选项供用户选择。

为了证明这一点，我们将定义一个视图：

1. 有可能的学生姓名数组。
2. 具有`@State`存储当前所选学生的属性。
3. 创建一个`Picker`视图，要求用户使用对该`@State`属性的双向绑定来选择自己喜欢的视图。
4. 用于`ForEach`遍历所有可能的学生姓名，将其转换为文本视图。

这是代码：

```swift
struct ContentView: View {
    let students = ["Harry", "Hermione", "Ron"]
    @State private var selectedStudent = 0

    var body: some View {
        VStack {
            Picker("Select your student", selection: $selectedStudent) {
                ForEach(0 ..< students.count) {
                    Text(self.students[$0])
                }
            }
            Text("You chose: Student # \(students[selectedStudent])")
        }
    }
}
```

那里没有很多代码，但是值得澄清一些事情：

1. 该`students`阵列不需要标明`@State`，因为它是一个常数; 它不会改变。
2. 该`selectedStudent`属性以值0开头，但可以更改，这就是为什么将其标记为的原因`@State`。
3. 该`Picker`有一个标签，“选择你的学生”，它告诉用户它做什么，并且还提供了一些描述性的屏幕阅读器大声朗读。
4. 与`Picker`具有双向绑定`selectedStudent`，这意味着它将开始显示选择0，但随着用户移动选择器而更新属性。
5. 在内部，`ForEach`我们从0到（但不包括）数组中的学生数量进行计数。
6. 我们为每个学生创建一个文本视图，以显示该学生的姓名。

`ForEach`将来我们将探讨其他使用方式，但这对于该项目就足够了。

## TextField读取文本

```swift
Form{
    Section{
        TextField("Amount", text: $checkAmount)
            .keyboardType(.decimalPad)
    }
    Section{
        Text("$\(checkAmount)")
    }
}
```

**提示：**在`.numberPad`和`.decimalPad`键盘类型告诉SwiftUI 0到9以及可选的小数点，显示的数字，但这并不能阻止从用户*进入*其它值。例如，如果他们有一个硬件键盘，他们可以键入自己喜欢的东西，如果他们从其他地方复制一些文本，则无论该文本内部是什么，都可以将其粘贴到文本字段中。没关系-我们稍后将处理这种情况。

## 列表形式创建选择器

SwiftUI的选择器有多种用途，它们的外观确切取决于您所使用的设备以及选择器所处的上下文。

在我们的项目中，我们有一个表格，要求用户输入支票的金额，我们想在其中添加一个选择器，以便他们可以选择共享支票的人数。

选择器（如文本字段）需要对属性进行双向绑定，以便它们可以跟踪其值。`@State`为此，我们已经创建了一个名为的属性`numberOfPeople`，因此我们的下一个工作是遍历2到99之间的所有数字，并将它们显示在选择器中。

修改表单的第一部分以包含一个选择器，如下所示：

```swift
Section {
    TextField("Amount", text: $checkAmount)
        .keyboardType(.decimalPad)

    Picker("Number of people", selection: $numberOfPeople) {
        ForEach(2 ..< 100) {
            Text("\($0) people")
        }
    }
}
```

现在，在模拟器中运行该程序并尝试一下-您注意到了什么？

希望您能发现几件事：

1. 会有一个新行，左边显示“ Number of people”，右边显示“ 4 people”。
2. 右边缘有一个灰色的披露指示器，这是iOS发出信号的信号，表示点击该行会显示另一个屏幕。
3. 点击该行*不会*显示其他屏幕。
4. 该行显示“ 4个人”，但是我们为我们的`numberOfPeople`媒体资源设置了默认值2。

因此，这有点“向前两步，向后两步” –我们取得了不错的结果，但是它不起作用，并且没有显示正确的信息！

我们将修复这两个问题，从简单的一个开始：我们给`numberOfPeople`默认值2 时为什么说4个人？好吧，在创建选择器时，我们使用了如下`ForEach`视图：

```swift
ForEach(2 ..< 100) {
```

从2到100计数，创建行。这意味着我们的第0行（即创建的第一行）包含“ 2人”，因此，当我们赋予`numberOfPeople`值2时，实际上是将其设置为*第三*行，即“ 4人”。

因此，尽管有点弯曲，但我们的UI显示“ 4个人”而不是“ 2个人”并不是一个错误。但是我们的代码中仍然存在一个大错误：为什么在该行上点击不执行任何操作？

如果您是在表单外部自行创建选择器，则iOS将显示旋转的选项。不过，在这里，我们告诉SwiftUI，这是一种供用户输入的表单，因此它自动更改了选择器的外观，从而不会占用太多空间。

SwiftUI *想要*做的事情-这也是为什么在行的右边缘添加了灰色显示指示器的原因-展示了一个带有我们选择器选项的新视图。为此，我们需要添加一个导航视图，该导航视图执行以下两项操作：在顶部留出一定空间以放置标题，并允许iOS根据需要在新视图中滑动。

因此，直接在表单add之前`NavigationView {`，以及在表单的右括号之后添加另一个右括号。如果操作正确，则代码应如下所示：

```swift
var body: some View {
    NavigationView {
        Form {
            // everything inside your form
        }
    }
}
```

如果再次运行该程序，则会在顶部看到一个较大的灰色空间，这是iOS为我们提供放置标题的空间。我们稍后会做，但是首先尝试点击“人数”行，您应该会看到一个新的屏幕幻灯片，其中包含所有其他可能的选项供您选择。

您应该看到“ 4人”旁边有一个复选标记，因为它是选定的值，但是您也可以点击另一个数字-屏幕将自动再次滑开，使用户以他们的新选择回到上一个屏幕。

您在这里看到的是所谓的*声明性用户界面设计*的重要性。这意味着我们说我们想要*什么*，而不是说应该*怎么*做。我们说过我们想要一个内部带有一些值的选择器，但是要由SwiftUI来决定轮式选择器还是滑动视图方法更好。之所以选择滑动视图方法，是因为选择器位于表单内部，但是在其他平台和环境上，它可以选择其他内容。

在完成此步骤之前，让我们为该新导航栏添加标题。给表单添加以下修饰符：

```swift
.navigationBarTitle("WeSplit")
```

**提示：**很容易想到修饰符应附加到的末尾`NavigationView`，但需要将其附加到末尾`Form`。其原因是，导航视图能够显示你的程序运行很多意见，所以由标题附加到的东西*里面*我们允许iOS系统改变标题自由导航视图。

## 添加分段控件

现在，我们将第二个选择器视图添加到我们的应用程序中，但是这次我们希望有所不同：我们需要一个*分段控件*。这是一种特殊的选择器，可在水平列表中显示一些选项，当您只有很少的选择时，它会很好用。

我们的表单已经有两个部分：一个是关于人数和人数的，另一个是我们将显示最终结果的部分–目前只是显示`checkAmount`，但我们将尽快对其进行修复。

在这两个部分的中间，我希望您添加第三个部分以显示小费百分比：

```swift
Section {
    Picker("Tip percentage", selection: $tipPercentage) {
        ForEach(0 ..< tipPercentages.count) {
            Text("\(self.tipPercentages[$0])%")
        }
    }
}
```

这将遍历`tipPercentages`数组中的所有选项，并将每个选项转换为文本视图。就像以前的选择器一样，SwiftUI会将其转换为列表中的一行，并在点击时在其中滑动新的选项屏幕。

不过，在这里，我想向您展示如何使用分段控件，因为它看起来要好得多。因此，请将此修饰符添加到提示选择器中：

```swift
.pickerStyle(SegmentedPickerStyle())
```

这应该在选择器的右括号结尾处进行，如下所示：

```swift
Section {
    Picker("Tip percentage", selection: $tipPercentage) {
        ForEach(0 ..< tipPercentages.count) {
            Text("\(self.tipPercentages[$0])%")
        }
    }
    .pickerStyle(SegmentedPickerStyle())
}
```

如果现在运行该程序，您会发现事情开始融合起来：用户现在可以在支票上输入金额，选择人数，然后选择要留下多少小费–不错！

是的，*我们*知道他们要选择留下多少小费，但这在屏幕上并不明显。我们可以而且*应该*做得更好。

一种选择是在分段控件之前直接添加另一个文本视图，我们可以这样做：

```swift
Section {
    Text("How much tip do you want to leave?")

    Picker("Tip percentage", selection: $tipPercentage) {
        ForEach(0 ..< tipPercentages.count) {
            Text("\(self.tipPercentages[$0])%")
        }
    }
    .pickerStyle(SegmentedPickerStyle())
}
```

可以，但是看起来并不好–看起来它本身就是一个项目，而不是分段控件的标签。

一个更好的主意是修改该部分本身：SwiftUI允许我们向部分的页眉和页脚添加视图，在这种情况下，我们可以使用它添加一个小的说明性提示。实际上，我们可以使用刚创建的相同文本视图，只是将其向上移为section标题，而不是其中的松散标签。

这就是代码中的样子：

```swift
Section(header: Text("How much tip do you want to leave?")) {
    Picker("Tip percentage", selection: $tipPercentage) {
        ForEach(0 ..< tipPercentages.count) {
            Text("\(self.tipPercentages[$0])%")
        }
    }
    .pickerStyle(SegmentedPickerStyle())
}
```

这是对代码的很小的改动，但是我认为最终结果看起来要好得多-文本现在看起来像是在其正下方的分段控件的提示。