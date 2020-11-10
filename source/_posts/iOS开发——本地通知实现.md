---
title: iOS开发——本地通知实现
date: 2020-04-28 16:10:30
tags: [iOS, Notiication]
catalogue: iOS开发	
---
## 什么是本地通知？

本地通知是简短的，基于文本的消息，显示在iPhone屏幕顶部。

也许最常见的通知类型是聊天或文本消息，例如来自iMessage的消息。通知还会显示在iOS的“ *通知中心”中*，您可以在其中查看和回复最新的通知。

在iOS上，您可以发送和接收两种类型的通知：

- **推送通知：**推送通知是通过Internet发送的，例如聊天消息。它们通过基于Web的Apple Push Notification Service（APNS）中继到您的iPhone。

- **本地通知：**本地通知是安排好的，并在您的iPhone上本地“发送”，因此不会通过Internet发送。典型的用例是日历提醒或iPhone闹钟。

  

> 在本篇文章中我们尝试实现一个本地通知管理器

<!-- more -->

## 本地通知管理器

通过构建一个独立的组件，使其在ViewController里的使用更加优雅。该`LocalNotificationManager`类将帮助我们管理本地通知。

这是`LocalNotificationManager`该类的起始定义：

```
import UserNotifications

class LocalNotificationManager
{
    var notifications = [Notification]()

}
```

容易吧？该类具有一个实例属性，称为`notifications`，类型或通知数组。使用上面的语法，该属性用一个空数组初始化。`[Notification]``notifications`

接下来，我们将定义该`Notification`类型：

```
struct Notification {
    var id:String
    var title:String
    var datetime:DateComponents
}
```

您很快就会看到，该`Notification` 结构将帮助我们更好地组织通知。我们没有将不同的值传递给`UNUserNotificationCenter`（用于计划本地通知的iOS SDK组件），而是将信息包装在我们自己制作的方便类型中。

安排本地通知的工作流程如下：

1. 检查用户是否已授予计划/处理本地通知的权限
2. 如果之前未请求任何权限，请询问用户
3. 如果已获得许可，请安排本地通知
4. 如果之前已请求许可，但用户拒绝，则不执行任何操作

为了适应上述功能，我们将在`LocalNotificationManager`类中定义一些功能：

- `listScheduledNotifications()`，因此我们可以调试已安排的通知
- `schedule()`，将启动通知权限和计划的公共功能
- `requestAuthorization()`，这是私人功能，将提示应用程序用户授予发送本地通知的权限
- `scheduleNotifications()`，该私有函数将遍历`notifications`属性以安排本地通知

> 这种3层权限检查在iOS开发中很常见。许多组件使用常量来指示是否以及如何授权使用。良好地处理权限（包括失败/被拒绝）是重要的最佳实践。

您可以使用以下功能来检查已计划哪些本地通知：

```
func listScheduledNotifications()
{
    UNUserNotificationCenter.current().getPendingNotificationRequests { notifications in

        for notification in notifications {
            print(notification)
        }
    }
}
```

上面的代码`getPendingNotificationRequests(completionHandler:)`在共享`UNUserNotificationCenter`实例上调用该函数，该实例接收一个[UNNotificationRequest](https://developer.apple.com/documentation/usernotifications/unnotificationrequest)对象数组。非常适合调试！

## 要求发送本地通知的权限

在我们计划本地通知以将其本地发送给用户之前，我们需要征得许可。大多数具有安全或隐私风险的iOS组件（例如iPhone的相机或照片库）均受基于权限的系统保护。

要求发送本地通知的权限涉及`requestAuthorization(options:completionHandler:)`共享`UNUserNotificationCenter`实例的功能。此单例实例用于管理iOS应用中与通知相关的所有内容。

在`LocalNotificationManager`该类中编写以下函数的代码：

```
private func requestAuthorization()
{
    UNUserNotificationCenter.current().requestAuthorization(options: [.alert, .badge, .sound]) { granted, error in

        if granted == true && error == nil {
            self.scheduleNotifications()
        }
    }
}
```

函数中发生的事情如下：

- 首先，我们`UNUserNotificationCenter`通过调用`current()`函数来访问共享实例。这样，我们就可以访问在iOS上管理通知的单个对象。（在其他API中，通常通过`shared`属性访问此对象。）
- 然后，该函数`requestAuthorization(options:completionHandler:)`被调用。这将提示iOS上的权限对话框。该选项参数是常数组。当前，我们仅在显示警报，更改应用程序图标的数字徽标以及在通知警报弹出时播放声音的权限。
- 然后，将完成处理程序作为函数的最后一个参数传递。它使用结尾闭包语法。授予许可后，将执行此关闭。它有两个参数，分别`granted`是type `Bool`和`error`type `Error`。基于这些参数，我们可以检查是否已实际授予许可。
- 与条件，我们正在检查其`granted`平等`true`和`error`是`nil`。这样，我们可以确定没有错误，并且已经授予了权限。随后，该`scheduleNotifications()`函数被调用（见下文）。

简而言之，此功能向iPhone用户请求发送本地通知的权限。如果允许，`scheduleNotifications()`则调用该函数。如果没有给予许可，或者发生错误，则什么也不会发生。

这里有一些注意事项：

- 该`requestAuthorization()`函数是私有的，因此只能在类*内部*使用`LocalNotificationManager`。
- 在您自己的应用程序中，始终要考虑并处理失败的情况，即，如果用户*未*授予权限会发生什么？
- 您是否对完成处理程序感到厌倦，还是经常使用嵌套的完成处理程序？退房[的承诺](https://learnappmaking.com/promises-swift-how-to/) -它会积极地打击你的心！

## 检查本地通知权限状态

现在我们有了一个要求用户许可的功能，我们可以在授权状态下进行特殊的秘密舞蹈。

该`schedule()`函数是`LocalNotificationManager`该类的面向公众的API的一部分。您可以使用它来启动本地通知权限以及随后的通知调度。

在`LocalNotificationManager`该类中编写以下函数的代码：

```
func schedule()
{
    UNUserNotificationCenter.current().getNotificationSettings { settings in

        switch settings.authorizationStatus {
        case .notDetermined:
            self.requestAuthorization()
        case .authorized, .provisional:
            self.scheduleNotifications()
        default:
            break // Do nothing
        }
    }
}
```

该`schedule()`功能如何工作？

- 首先，我们`UNUserNotificationCenter`再次访问该共享库，然后调用该`getNotificationSettings(completionHandler:)`函数。同样，为了简洁起见，我们使用结尾闭包语法。并且，和以前一样，在收到设置后将执行此完成处理程序。

- 然后，在switch块内部，我们正在检查

  ```
  authorizationStatus
  ```

  枚举类型

  UNAuthorizationStatus

  的属性。此值可以准确地告诉我们是否以及已授予什么权限。

  1. *未确定*。如果未确定授权，则意味着我们之前没有请求过许可。因此，我们将通过调用该`requestAuthorization()`函数来请求许可！
  2. *授权的或临时的*。如果值`authorizationStatus`是`.authorized`或`.provisional`，这意味着我们必须（临时）许可计划的通知。因此，我们将通过致电安排通知`scheduleNotifications()`（请参见下文）。
  3. *还有什么*。如果没有其他切换条件匹配，即在的情况下`.denied`，我们什么都不做。我们无法再次请求许可，也无法安排本地通知。

在iOS上，您只能请求*一次*许可。如果明确拒绝了权限，则用户必须通过iPhone的“设置”应用手动将其重置。这就是为什么您总是想弄清楚应用程序为何要求许可的原因。许多应用程序还使用两步方法，即首先请求软许可，然后使用iOS提供的许可对话框。

这里要理解的一件事很重要，就是我们可以在类中定义两条路径，两条路径都以结尾`scheduleNotifications()`。

1. 尚未授予许可，因此被提出，并`scheduleNotifications()`从中调用`requestAuthorization()`
2. 之前已授予权限，因此`scheduleNotifications()`直接从`schedule()`函数中调用

我们在`LocalNotificationManager`该类中编码的函数是*幂等的*。幂等是一种软件开发原则，该原则指出可以多次调用一个函数，而不会在初始调用后改变结果。

换句话说，无论您调用该`schedule()`函数1次还是100次，它总是将同一组通知安排一次。相反的意思是您两次调用该函数，并且得到的通知（重复）通知次数是原来的两倍！

一些例子：

- 幂等函数是该函数。调用一次或多次都没有关系，该函数将始终舍入到最接近的整数。`round(_:)`
- 非幂等函数是`moveUp()`假设的棋盘游戏中的函数。每次调用该功能时，棋子都会在板上上升一个位置。调用此函数多少次很重要。

重要的是我们的`schedule()`函数是幂等的，因为这样我们就可以调用它而不必担心意外的副作用。我们可以肯定地知道，该功能将始终计划本地通知（如果已授予权限），并且在权限待处理时，也会计划本地通知。

将此与您必须调用两次的函数进行比较：一次获得许可，然后在获得许可时再次调用。或者，假设有一个函数在您每次调用它时安排*新的*通知。这肯定是导致错误的原因！

**快速提示：**知道*幂等之后*，随处可见。人行横道按钮。一个“开”按钮。电梯楼层按钮。iPhone的主屏幕按钮。好的Web表单*提交*按钮。该*“呼叫乘务员”*按钮。紧急刹车。按下多少次都没关系！（这样做的不确定性为零，没有副作用，即*非常安全*。）

## 安排本地通知

现在，我们终于可以编写该`scheduleNotifications()`函数了。此函数遍历数组中的`Notification`对象，`notifications`并计划它们在将来交付。

在该类中编写以下函数的代码：

```
private func scheduleNotifications()
{
    for notification in notifications
    {
        let content      = UNMutableNotificationContent()
        content.title    = notification.title
        content.sound    = .default

        let trigger = UNCalendarNotificationTrigger(dateMatching: notification.datetime, repeats: false)

        let request = UNNotificationRequest(identifier: notification.id, content: content, trigger: trigger)

        UNUserNotificationCenter.current().add(request) { error in

            guard error == nil else { return }

            print("Notification scheduled! --- ID = \(notification.id)")
        }
    }
}
```

在顶层，该函数`notifications`使用for循环遍历数组（一个实例属性）。在循环内部，对于数组中的每个项目，都会发生：

- 首先，创建一个类型为的对象`UNMutableNotificationContent`。该对象包含通知的*内容*，例如标题，正文和声音。
- 然后，创建一个类型为的对象`UNCalendarNotificationTrigger`。该对象包含通知的*触发器*，例如日期和时间。我们正在使用DateComponents类型`datetime`的`Notification`结构属性来指示*何时*发送通知。
- 然后，我们创建一个类型为的对象`UNNotificationRequest`。该对象将内容和触发器与唯一ID结合在一起。每个通知都需要一个唯一的ID，您可以方便地使用它来重新安排本地通知。
- 最后，将`request`对象传递给`add(request:completionHandler:)`共享`UNUserNotificationCenter`实例的功能。这将调度本地通知，然后执行完成处理程序。在此完成处理程序中，我们正在检查是否未发生错误，并向*控制台*打印一条消息。

随便吧，对吧？首先构造内容，然后构造触发器，然后将它们组合为“请求”，然后将请求传递给`UNUserNotificationCenter`。

您还可以根据时间间隔触发本地通知：

```
let trigger = UNTimeIntervalNotificationTrigger(timeInterval: 120, repeats: false)
```

您还`CLRegion`可以在进入或退出该区域时将GPS区域用作类型的触发器。像这样：

```
let trigger = UNLocationNotificationTrigger(triggerWithRegion: region, repeats: false)
```

该`DateComponents`结构对于创建具有特定日期，月份，年份，小时，分钟等参数的基于日期时间的对象很有用。的实例`DateComponents`将使用iPhone的日历语言环境，这几乎使您可以创建零失误的原始日期时间对象。

既然`LocalNotificationManager`课程已经完成，我们可以安排一些通知，如下所示：

```
let manager = LocalNotificationManager()
manager.notifications = [
    Notification(id: "reminder-1", title: "Remember the milk!", datetime: DateComponents(calendar: Calendar.current, year: 2019, month: 4, day: 22, hour: 17, minute: 0)),
    Notification(id: "reminder-2", title: "Ask Bob from accounting", datetime: DateComponents(calendar: Calendar.current, year: 2019, month: 4, day: 22, hour: 17, minute: 1)),
    Notification(id: "reminder-3", title: "Send postcard to mom", datetime: DateComponents(calendar: Calendar.current, year: 2019, month: 4, day: 22, hour: 17, minute: 2))
]

manager.schedule()
```

一次或多次运行此代码都没有关系，它将始终计划相同的三个本地通知。它将一次请求许可，并在以后的调用中考虑授权状态。

该结构的`id`属性`Notification`用于标识唯一通知。如果您使用相同的`id`，或为的`identifier`参数提供相同的标识符`UNNotificationRequest`，则将重新安排关联的通知。如果您为本地通知使用了良好的命名结构，则不必担心意外地计划了许多重复的通知。

`Notification`上面的代码中的struct使用了合成的初始化函数。此函数包括该结构的所有属性作为参数。如果您自己不提供初始化程序，它将自动添加到结构和类中。

在这里值得指出的是`LocalNotificationManager` *必须*使用属性`notifications`，该属性`Notification`在调用之前需要用对象填充`schedule()`。这是为什么？

由于的异步特性`UNUserNotificationCenter`及其对完成处理程序的依赖，我们正在从内部的闭包内部调度通知`schedule()`。如果该`schedule()`函数接受了一个`notifications`参数，我们将需要一些意大利面条式代码将该数组传递给`requestAuthorization()`and `scheduleNotifications()`。不幸的是，这意味着`schedule()`依赖于`notifications`属性的状态才能正常运行。

一个很好的选择是使用诺言。双方`requestAuthorization()`并`scheduleNotifications()`然后函数可以返回一个承诺，我们可以使用一个承诺块`schedule()`解析基于所述许可状况链条。



##完整代码

```swift
import UserNotifications

struct Notification {
    var id: String
    var title: String
    var datetime: DateComponents
}

class LocalNotificationManager {
    var notifications = [Notification]()
    
    /// 检查已计划的本地通知
    func listScheduledNotification() {
        UNUserNotificationCenter.current().getPendingNotificationRequests { notifications in
            for notification in notifications {
                print(notification)
            }
        }
    }
    
    /// 请求发送通知的权限
    private func requestAuthorization() {
        UNUserNotificationCenter.current().requestAuthorization(options: [.alert, .badge, .sound]) { (granted, error) in
            if granted == true && error == nil {
                
            }
        }
    }
    
    func schedule() {
        UNUserNotificationCenter.current().getNotificationSettings { (setting) in
            switch setting.authorizationStatus {
            case .notDetermined:
                self.requestAuthorization()
            case .authorized, .provisional:
                self.scheduleNotifications()
            default:
                break
            }
        }
    }
    
    private func scheduleNotifications() {
        for notification in notifications {
            let content = UNMutableNotificationContent()
            content.title = notification.title
            content.sound = .default
            
            let trigger = UNCalendarNotificationTrigger(dateMatching: notification.datetime, repeats: false)
            let request = UNNotificationRequest(identifier: notification.id, content: content, trigger: trigger)
            
            UNUserNotificationCenter.current().add(request) { (error) in
                guard error == nil else {return}
                
                print("Notification scheduled! --- ID = \(notification.id)")
            }
        }
    }
}

```

