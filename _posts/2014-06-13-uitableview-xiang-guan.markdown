---
layout: post
title: "UITableView 相关"
date: 2014-06-13 13:48:02 +0800
comments: true
categories: iOS
---

![](/images/4902731144446386506.jpg)
#### 问题描述：


 UITableView deleteRowsAtIndexPaths:withRowAnimation:在使用的时候，当你删除该行的时候，tableView并不会reload，所以该	行的下一行的indexPath并没有改变，当下一行为最后一行的时候，你的Model里的数据在删除的时候调用[arrayremoveObjectAtIndex:indexPath.row] 时候会报错：数组越界。

#### 解决办法:

假如你直接使用[tableView reloadData]，那么就看不到  tableView的动画效果了，所以不能用[tableView reloadData];代替的方法是使用 reloadSections:withRowAnimation:方法。

***


#### 问题描述：


如何设置UITableViewCell accessoryType的颜色？


#### 解决办法:

```objc
	cell.tintColor = [UIColor greenColor];
```

注意：类型为UITableViewCellAccessoryDisclosureIndicator 的不能通过这种方法更改，其颜色只能灰色。

***

#### 问题描述：


如何获取UITableViewCell 在TableView中的frame？

#### 解决办法:

```objc
 CGRect cellFrame = [_tableView rectForRowAtIndexPath:[NSIndexPath indexPathForRow:index inSection:0]];
```
***

#### 问题描述：


在使用Grouped UITableView的时候，如何去掉上面默认的空白


#### 解决办法:
```objc
_tableView.tableHeaderView = [[UIView alloc] initWithFrame:CGRectMake(0.0f, 0.0f, _tableView.bounds.size.width, 0.01f)];
```


***

#### 问题描述

iOS8 如何如何使 tableView cell 自适应高度 ？

#### 解决办法

对自定义的Cell 进行约束布局（Autolayout）,

在`viewDidLoad`里添加

```objc
   self.tableView.rowHeight = UITableViewAutomaticDimension;
   self.tableView.estimatedRowHeight = 100;
```
