我们都知道对`UITableViewCell`的重用，但是却很少人知道，对`UITableView`的header和footer，也是可以重用的。下面就来看看`UITableView`的header和footer的重用例子：

- 首先声明一个全局静态变量：

```objc
static NSString *headerIdentifier = @"tableViewHeaderIdentifier";
```

* 然后在`viewDidLoad`里面注册`UITableViewHeaderFooterView`或者其子类：

```objc
[self.tableView registerClass:[UITableViewCell class] forCellReuseIdentifier:cellIdentifier];
    [self.tableView registerClass:[UITableViewHeaderFooterView class] forHeaderFooterViewReuseIdentifier:headerIdentifier];
```

* 接下来就在`UITableViewDelegate`的方法`-tableView:viewForHeaderInSection:`或者`-tableView:viewForFooterInSection:`里面用identifer来或者可重用的header或者footer：

```objc
-(UIView *)tableView:(UITableView *)tableView viewForHeaderInSection:(NSInteger)section {
    UITableViewHeaderFooterView *headerView = [tableView dequeueReusableHeaderFooterViewWithIdentifier:headerIdentifier];
    headerView.textLabel.text = self.yearArray[section];
    headerView.contentView.backgroundColor = [UIColor colorWithRed:1 green:0 blue:0 alpha:0.3];
    headerView.detailTextLabel.text = @"detailTextLabel";
    return headerView;
}
```

上面的例子用的是类`UITableViewHeaderFooterView`本身，我们可以通过继承`UITableViewHeaderFooterView`来达到自定义`UITableView`的header和footer的目的。



---

【完】