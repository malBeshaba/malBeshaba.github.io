---
title: iOS开发——UITableView（一）
date: 2020-04-28 15:57:21
tags: [UITableView, iOS]
---

<!-- more -->	

### UITableView是一种应用面极广的控件

UITableView有几个个基本属性：
1. frame （当然是设置位置和大小的啦）
2. style （.plain和.grouped)   [区别可以看这个](https://www.jianshu.com/p/764ed5aa46cf)

#### 当我们准备开始使用UITableView时应该先声明一个对象

```
var yourTableView: UITableView!
```
#### 当然也可以直接初始化

```
var yourTableView = UITableView(frame: CGRect , style: UITableView.Style)
```
- 根据需求决定CGRect和style

## 我们知道当我们使用UITableView的时候必须要使用代理
### 所以这句话一定不要忘了写！！！(最初我自己都忘了好多次）

```
yourTableView.delegate = self
yourTableView.dataSource = self
```
这里的self就是要用此类中的代理方法，当我们想直接调用其他类中已经写好的代理方法时可以这样

```
yourTableView.delegate = 类名.self
```
### 以下是必须的代理方法(由于编辑的时候想到什么写什么，所以有些混乱，先做一些标注，之后再进一步整理）
*标data的是UITableViewDataSource的方法*
*标del的是UITableViewDelegate的方法*

```
func tableView(_ tableView: UITableView, numberOfRowsInSection section: Int) -> Int {
//***data***
//在这里我们可以设置我们想要返回几个cell
//也可以根据不同的section、不同的tableView设置不同的返回值
}
```

```
func tableView(_ tableView: UITableView, heightForRowAt indexPath: IndexPath) -> CGFloat {
//***del***
        //在这里我们可以设置每个单元格的高度
        //当然可以根据不同的tableView分别设置
    }
```
**下面这个方法非常重要**
```
func tableView(_ tableView: UITableView, cellForRowAt indexPath: IndexPath) -> UITableViewCell {
//***data***
//在这里我们可以自己设置单元格内想要显示的东西
//可以直接声明一个UITableViewCell的对象，也可以自定义一个cell
}
```
**UITableViewCell的具体用法可以看下面**
****
### 之后是不必须的但也常用的方法
*这些方法不分什么顺序*

```
yourTableView.frame = view.bounces // 紧贴屏幕
```

```
func tableView(_ tableView: UITableView, didSelectRowAt indexPath: IndexPath)  {
//***del***
//选中cell时执行此方法
}
```

```
func tableView(_ tableView: UITableView, viewForHeaderInSection section: Int) -> UIView? {
//***del***
//TableView的每个section都可以又一个header view
//可以自定义
}
func tableView(_ tableView: UITableView, heightForHeaderInSection section: Int) -> CGFloat {
//***del***
//返回header view的高度
}
```

```
func tableView(_ tableView: UITableView, viewForFooterInSection section: Int) -> UIView?  {
//***del***
//footer view的设置方法和header view相似
}
func tableView(_ tableView: UITableView, heightForFooterInSection section: Int) -> CGFloat {
//***del***
}
```

```
tableView!.register(MyTableViewCell.self, forCellReuseIdentifier: "cell")
//创建重用单元格
```

### 一些不常用的东西：
- 取消行间分割线`yourTableView?.separatorStyle = UITableViewCell.SeparatorStyle.none`
*之后再补吧现在不想写了*