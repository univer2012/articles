本文来自：[ iOS亲测UITableView重用机制，用事实说话。](http://blog.csdn.net/xietao3/article/details/43527165)

`UITableView`重用机制主要是依靠reuseIdentifier来辨别，以此来建立一个队列，将建好的Cell放入队列中，之后直接使用队列中的Cell,不再新建，极大的提升了`UITableView`的重用性，同时使列表滑动时不会出现卡顿现象。`UITableView`基本上是新手必学，我第一个熟练掌握的控件，及使用得最多的就是`UITableVIew`,重用机制有很多地方很多人都讲过，不过我还是要从自己的角度来再次BB下。由于列表内容太长了，不方便直接在界面上截图，所以只把打印的数据拿出来。

### 1、重用机制Cell是如何新建的？

很多人以为Cell在新建的时候只新建一个，其他的都重复使用那一个，但事实上并不是这样的，新建多少个Cell,并加入重用队列是取决于初始化时屏幕中能容纳多少个Cell,（iPhone 5s模拟器测试）经测试Cell高44时，新建14个Cell。Cell高120时，count:6，新建6个Cell。

```objc
-(CGFloat)tableView:(UITableView *)tableView heightForRowAtIndexPath:(NSIndexPath *)indexPath {
    return 44;
}
-(UITableViewCell *)tableView:(UITableView *)tableView cellForRowAtIndexPath:(NSIndexPath *)indexPath {
    static int count = 0;
    static NSString *cellIndentifier = @"Cell";
    UITableViewCell *cell=[tableView dequeueReusableCellWithIdentifier:cellIndentifier];
    if (cell == nil) {
        cell=[[UITableViewCell alloc]initWithStyle:UITableViewCellStyleDefault reuseIdentifier:cellIndentifier];
        cell.selectionStyle = UITableViewCellSelectionStyleGray;
        count ++;
    }
    
    if (indexPath.row < 10) {
        cell.textLabel.text = [NSString stringWithFormat:@"%ld", indexPath.row];
    }
    NSLog(@"count: %d, cellHeight: %f", count, cell.frame.size.height);
    
    return cell;
}
```

[![cell高为44时，新建了16个cell](https://univer2012.github.io/2017/05/18/17Implementation-of-UITableviewCell-reuse-mechanism/pic1.png)](https://univer2012.github.io/2017/05/18/17Implementation-of-UITableviewCell-reuse-mechanism/pic1.png)cell高为44时，新建了16个cell

[![cell高为120时，新建了6个cell](https://univer2012.github.io/2017/05/18/17Implementation-of-UITableviewCell-reuse-mechanism/pic2.png)](https://univer2012.github.io/2017/05/18/17Implementation-of-UITableviewCell-reuse-mechanism/pic2.png)cell高为120时，新建了6个cell

### 2、Cell重用时是如何取出来的？

在使用120高度的Cell时，重用队列中将只有6个存储的Cell,在下方代码中到第6个Cell,即textLabel == 6时 已用开始使用重用队列中的Cell,由打印的数据可知，init  textLabel为0，这使用的是第一个存入的队列Cell，textLabel == 7时， init  textLabel为1，使用的是存入的第二个Cell，一直到使用第六个完成一轮后再重新使用第一个存入的Cell，由此可以推断，Cell重用时是秉承先进先出的原则，循环使用。

```objc
-(CGFloat)tableView:(UITableView *)tableView heightForRowAtIndexPath:(NSIndexPath *)indexPath {
    return 120;
}
-(UITableViewCell *)tableView:(UITableView *)tableView cellForRowAtIndexPath:(NSIndexPath *)indexPath {
    static int count = 0;
    static NSString *cellIndentifier = @"Cell";
    UITableViewCell *cell=[tableView dequeueReusableCellWithIdentifier:cellIndentifier];
    if (cell == nil) {
        cell=[[UITableViewCell alloc]initWithStyle:UITableViewCellStyleDefault reuseIdentifier:cellIndentifier];
        cell.selectionStyle = UITableViewCellSelectionStyleGray;
        count ++;
    }
    NSLog(@"init textLabel: %@", cell.textLabel.text);
    if (indexPath.row < 10) {
        cell.textLabel.text = [NSString stringWithFormat:@"%ld", indexPath.row];
    }
    NSLog(@"count: %d, textLabel: %@", count, cell.textLabel.text);
    
    return cell;
}
```

[![img](https://univer2012.github.io/2017/05/18/17Implementation-of-UITableviewCell-reuse-mechanism/pic3.png)](https://univer2012.github.io/2017/05/18/17Implementation-of-UITableviewCell-reuse-mechanism/pic3.png)

### 3、reuseIdentifier的作用？

在加载第11个Cell的时候开始，Cell的celleIdentifier换成了@”cell2”，并且将第11个Cell的textLabel内容换乘@”this is 20”。根据截图可知，由于更换了celleIdentifier，在第11个Cell加载时是没有init  textLabel的，并且count开始自增直至12，说明使用新的celleIdentifier使tableview重新建了一个队列并且放入了6个新的Cell，并且在6个一轮之后重用了，新队列里的Cell。

```objc
-(CGFloat)tableView:(UITableView *)tableView heightForRowAtIndexPath:(NSIndexPath *)indexPath {
    return 120;
}
-(UITableViewCell *)tableView:(UITableView *)tableView cellForRowAtIndexPath:(NSIndexPath *)indexPath {
    static int count = 0;
    static NSString *cellIndentifier = @"Cell";
    if (indexPath.row > 10) {
        cellIndentifier = @"Cell2";
    }
    
    UITableViewCell *cell=[tableView dequeueReusableCellWithIdentifier:cellIndentifier];
    if (cell == nil) {
        cell=[[UITableViewCell alloc]initWithStyle:UITableViewCellStyleDefault reuseIdentifier:cellIndentifier];
        cell.selectionStyle = UITableViewCellSelectionStyleGray;
        count ++;
    }
    NSLog(@"init textLabel:%@",cell.textLabel.text);
    if (indexPath.row < 10) {
        cell.textLabel.text = [NSString stringWithFormat:@"%ld", indexPath.row];
    }
    else if (indexPath.row == 11) {
        cell.textLabel.text = @"this is 20";
    }
    NSLog(@"count:%d,textLabel:%@",count,cell.textLabel.text);
    
    return cell;
}
```

[![img](https://univer2012.github.io/2017/05/18/17Implementation-of-UITableviewCell-reuse-mechanism/pic4.png)](https://univer2012.github.io/2017/05/18/17Implementation-of-UITableviewCell-reuse-mechanism/pic4.png)

### 4、为什么使用重用机制后我的数据乱掉了？

在解释第三点时,在第11个Cell之后textLabel一直显示的是null，这是因为我没有在重用后，对这个Cell进行操作，一般在使用重用后，都需要对Cell的数据进行处理。

如果没有处理的话，一种情况就像上面截图一样，由于重用队列里面的Cell的textLabel就是没有值的，取出来之后还是不进行操作，即为null。

还有一种情况是打印出this is  20的这种情况，因为重用队列里面的Celld的textLabel里面有值，取出重用后不进行操作的话就会使用之前队列里Cell的textLabel这个值,所以后面有好几个重用Cell2队列中第一个存入的Cell时，打印出来的数据也为this is 20。

### 总结：

**不同的reuseIdentifier代表了不同的队列，重用时按队列顺序先进先出，一轮之后再重新从第一个开始重用。**



