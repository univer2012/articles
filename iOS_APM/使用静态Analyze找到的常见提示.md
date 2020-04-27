#### 1、`Call to 'dispatch_once' uses the instance variable '_onceToken' for the predicate value. Using such transient memory for the predicate is potentially dangerous`

YYAnimatedImageView.m中的错误
```
Call to 'dispatch_once' uses the instance variable '_onceToken' for the predicate value. Using such transient memory for the predicate is potentially dangerous
```
截图如下：
![](https://raw.githubusercontent.com/univer2012/univer2012.github.io/master/2018/%E4%BD%BF%E7%94%A8%E9%9D%99%E6%80%81Analyze%E6%89%BE%E5%88%B0%E7%9A%84%E5%B8%B8%E8%A7%81%E6%8F%90%E7%A4%BA_1.png)

调用dispatch_once时使用实例变量_onceToken作为断言值。对这个断言使用这样的临时内存有潜在风险。
修正为：
```
static dispatch_once_t onceToken;
dispatch_once(&onceToken, ^{
    //...
});
```

#### 2、`Value stored to 'resultsDict' during its initialization is never read`
```
Value stored to 'resultsDict' during its initialization is never read
```
代码如下：
```
NSMutableArray *tempMutArr = [NSMutableArray arrayWithCapacity:0];
if ([self.clickedButtonTpye isEqualToString:KClickedButtonTypeLast]) {
    tempMutArr = self.lastDataSourceArr;
}else{
    tempMutArr = self.hotDataSourceArr;   
}
```
该问题的原因是：变量申请了内存并初始化了，但没有用使此变量，接着将此变量又重新赋值。


仔细看了代码后才发现代码存在一个细节上的问题，也正是这个细节导致的内存泄漏。在项目里我的self.lastDataSourceArr和self.hotDataSourceArr都是已经初始化过的可变数组，但是在if里我又重新初始化了一个新的可变数组，并且把之前已经初始化过的可变数组赋值给了它，这就是内训泄漏的问题所在，看似代码没有大的问题，但是其实已经造成了内存泄漏。**原因是：因为self.lastDataSourceArr和self.hotDataSourceArr是已经创建并且分配过内存的可变数组了，但是我把这些数组又赋值给了重新创建并分配了内存的可变数组tempMutArr，所以这样就出现了一个数据源却申请了两块内存的情况，那么就存在一块内存空闲了，所以就存在了内存泄漏。**

 其实本意很简单，就是想把两个可变数组分情况赋值给可变数组tempMutArr，那么我们完全可以这样做：
 ```
 NSMutableArray *tempMutArr;
if ([self.clickedButtonTpye isEqualToString:KClickedButtonTypeLast]) {
    tempMutArr = self.lastDataSourceArr;
}else{
    tempMutArr = self.hotDataSourceArr;   
}
 ```

####  3、`nil returned from a method that is expected to return a non-null value`
```
nil returned from a method that is expected to return a non-null value
```
代码如下：
```
-(UITableViewCell *)tableView:(UITableView *)tableView cellForRowAtIndexPath:(NSIndexPath *)indexPath
{
    if(tableView == self.selectedContentTableView) {
        // make cell...
        return cell;
    }
    else if(tableView == self.recipientTableView) {
        // make cell...
        return cell;
    }
    else {
        NSLog(@"Some exception message for unexpected tableView");
        return nil; // ??? what now instead
    }
}
```
如果该情况是不应该发生的编程错误，则适当的操作是终止带有错误消息的程序，以便检测到程序错误并可以修复：
```
-(UITableViewCell *)tableView:(UITableView *)tableView cellForRowAtIndexPath:(NSIndexPath *)indexPath
{
    if(tableView == self.selectedContentTableView) {
        // make cell...
        return cell;
    }
    else if(tableView == self.recipientTableView) {
        // make cell...
        return cell;
    }
    else {
        NSLog(@"Some exception message for unexpected tableView");
        abort();
    }
}
```
`abort()` 函数带有 `__attribute__((noreturn))` 标记，因此编译器不会报缺少返回值。

#### 4、`Dictionary value cannot be nil`
```
Dictionary value cannot be nil
```
代码截图如下：
![](https://raw.githubusercontent.com/univer2012/univer2012.github.io/master/2018/%E4%BD%BF%E7%94%A8%E9%9D%99%E6%80%81Analyze%E6%89%BE%E5%88%B0%E7%9A%84%E5%B8%B8%E8%A7%81%E6%8F%90%E7%A4%BA_2.png)

部分代码如下：
```
/**
 *	@author hsj, 16-07-12 11:07:55
 *
 *	自定义H5全量更新和增量更新 的页面
 */
-(void)invokeAppH5Update {
    
    _updateButtonIsFirstClicked = YES;
    __block typeof(_updateButtonIsFirstClicked) blockIsFirstClicked = _updateButtonIsFirstClicked;
    //socket更新方式的参数处理
    NSMutableDictionary *reqParam = [NSMutableDictionary dictionary];
    reqParam[@"funcNo"] = @"1108001";
    reqParam[@"channel"] = @"2";
    reqParam[@"version_code"] = [TKSystemHelper getVersionSn];
    reqParam[@"soft_no"] = [TKSystemHelper getAppIdentifier];
    reqParam[@"ip"] = [TKNetHelper getIP];
    NSArray *results = (NSArray *)[[TKAppEngine shareInstance]callPlugin:@"50022" param:nil module:nil].results;
    NSDictionary *deviceInfo = results[0];
    NSString *device_id = [deviceInfo getStringWithKey:@"deviceToken"];
    reqParam[@"device_id"] = device_id;
    
    //http://220.178.81.218:8002/servlet/json?funcNo=1108001&channel=2&version_code=110&soft_no=com.thinkive.pushTest&ip=&device_id=e4d72a51e6aaae87d3e3850ea8fb143874c6443e
    [_announcementService p_checkVersionUpdateSocket:reqParam callBackFunc:^(ResultVo *resultVo) {
        if (resultVo.errorNo == 0) {
            NSArray *resultsArray = (NSArray *)resultVo.results;
            //缓存返回的结果，给设置的 是否是最新版本 使用 hsj-16-07-21
            if (resultsArray.count != 0) {
                NSMutableDictionary *cacheResultDict = [resultsArray[0] mutableCopy];
                [TKCacheHelper setCustomItem:cacheResultDict withKey:@"tkha_check_url_update_h5_cache"];
            }else{
                return ;
            }
            
            NSMutableDictionary *data = resultsArray[0];
            if (data == nil) {
                NSLog(@"服务器增量更新检测返回的resultVo.results[0]无数据");
                return;
            }
            //内部版本升级序号
            NSString *versionSn = [data getStringWithKey:@"version_code"];
            //版本名称
            NSString *version = [data getStringWithKey:@"version_name"];
            //升级描述
            NSString *description = [data getStringWithKey:@"description"];
            if (!description) {
                description = @"";
            }
            //下载地址
            NSString *download_url = [data getStringWithKey:@"download_url"];
            //强制更新标志
            NSString *update_flag = [data getStringWithKey:@"update_flag"];
            self.update_flag = update_flag;
            //是否H5
            NSString *isH5 = [data getStringWithKey:@"isH5"];
            self.isH5 = isH5;
            //获取H5下载包体积大小
            NSString *download_Size = [data getStringWithKey:@"download_size"];
            //下载包的大小
            //当前版本序号
            NSString *currentVersionSn = [TKSystemHelper getVersionSn];
            
            //无H5或原生更新, 进行原生AppStore升级后的通知权限检测
            //每次原生AppStore升级后,检测用户是否打开通知开关,过滤第一次安装. ---add by wc
            //处理原生和H5的更新弹框.  isH5字段.   0->原生、 1->H5
            if (isH5 && [isH5 isEqualToString:@"0"]) {
                [self skipAppStoreWithVersion:version downloadURL:download_url description:description updateFlag:update_flag download_size:download_Size];
                return;
            }
            
            //判断是否进行更新
            if (versionSn.intValue > currentVersionSn.intValue ) {
                
                UIWindow *rootWindow = [UIApplication sharedApplication].keyWindow.rootWindow;
                
                _updateH5AnnounceView = [[TKUpdateH5AnnounceView alloc]initWithFrame:rootWindow.frame withOptions:
                                         @{@"description" : description,
                                           @"download_url" : download_url,
                                           @"version" : version,
                                           @"versionSn" : versionSn,
                                           @"isUpdateH5" : isH5,
                                           @"download_size" : download_Size } ];
                [rootWindow addSubview:_updateH5AnnounceView];
                //设置下载时，进度条等数据的获取代理 hsj -16-07-15
                [TKUpdateManager shareInstance].delegate = self;
                //如果为1，就强制更新 hsj-16-07-15
                if ([update_flag isEqualToString:@"1"]) {
                    
                    [_updateH5AnnounceView.updateButton mas_updateConstraints:^(MASConstraintMaker *make) {
                        make.top.equalTo(_updateH5AnnounceView.progressView.mas_bottom).offset( 6 );
                        make.leading.equalTo(_updateH5AnnounceView.progressView).offset( 4 );
                        
                        make.trailing.equalTo(_updateH5AnnounceView.progressView).offset( -6 );
                        make.height.equalTo( @40 );
                    }];
                    _updateH5AnnounceView.cancelButton.hidden = YES;
                }
                
                dispatch_async(dispatch_get_main_queue(), ^{
                    [[NSNotificationCenter defaultCenter]postNotificationName:NOTE_APP_ISUPDATE object:@{NOTE_APP_ISUPDATE: @"1"}];
                });
                
                //进行了H5更新后, 就不进行原生AppStore通知权限检测,防止弹2个框的情况
                return;
            }
            else {
                dispatch_async(dispatch_get_main_queue(), ^{
                    [[NSNotificationCenter defaultCenter]postNotificationName:NOTE_APP_ISUPDATE object:@{NOTE_APP_ISUPDATE: @"0"}];
                });
            }
        }
    }];
}
```

修改为：**要确保Dictionary和Array的value是非nil的。**

#### 5、`Access to field 'next' results in a dereference of a null pointer(loaded from variable 'prev')`
```
Access to field 'next' results in a dereference of a null pointer(loaded from variable 'prev')
```
访问字段在引用一个废弃的空指针下的结果（从变量prev加载）
代码截图：
![](https://raw.githubusercontent.com/univer2012/univer2012.github.io/master/2018/%E4%BD%BF%E7%94%A8%E9%9D%99%E6%80%81Analyze%E6%89%BE%E5%88%B0%E7%9A%84%E5%B8%B8%E8%A7%81%E6%8F%90%E7%A4%BA_3.png)

代码如下：
```
QRinput_Struct *QRinput_splitQRinputToStruct(QRinput *input)
{
	QRinput *p;
	QRinput_Struct *s;
	int bits, maxbits, nextbits, bytes, ret;
	QRinput_List *list, *next, *prev;

	s = QRinput_Struct_new();
	if(s == NULL) return NULL;

	input = QRinput_dup(input);
	if(input == NULL) {
		QRinput_Struct_free(s);
		return NULL;
	}

	QRinput_Struct_setParity(s, QRinput_calcParity(input));
	maxbits = QRspec_getDataLength(input->version, input->level) * 8 - STRUCTURE_HEADER_BITS;

	if(maxbits <= 0) {
		QRinput_Struct_free(s);
		QRinput_free(input);
		return NULL;
	}

	bits = 0;
	list = input->head;
	prev = NULL;
	while(list != NULL) {
		nextbits = QRinput_estimateBitStreamSizeOfEntry(list, input->version);
		if(bits + nextbits <= maxbits) {
			ret = QRinput_encodeBitStream(list, input->version);
			if(ret < 0) goto ABORT;
			bits += ret;
			prev = list;
			list = list->next;
		} else {
			bytes = QRinput_lengthOfCode(list->mode, input->version, maxbits - bits);
			if(bytes > 0) {
				/* Splits this entry into 2 entries. */
				ret = QRinput_splitEntry(list, bytes);
				if(ret < 0) goto ABORT;
				/* First half is the tail of the current input. */
				next = list->next;
				list->next = NULL;
				/* Second half is the head of the next input, p.*/
				p = QRinput_new2(input->version, input->level);
				if(p == NULL) goto ABORT;
				p->head = next;
				/* Renew QRinput.tail. */
				p->tail = input->tail;
				input->tail = list;
				/* Point to the next entry. */
				prev = list;
				list = next;
			} else {
				/* Current entry will go to the next input. */
				prev->next = NULL;
				p = QRinput_new2(input->version, input->level);
				if(p == NULL) goto ABORT;
				p->head = list;
				p->tail = input->tail;
				input->tail = prev;
			}
			ret = QRinput_Struct_appendInput(s, input);
			if(ret < 0) goto ABORT;
			input = p;
			bits = 0;
		}
	}
	QRinput_Struct_appendInput(s, input);
	if(s->size > MAX_STRUCTURED_SYMBOLS) {
		QRinput_Struct_free(s);
		errno = ERANGE;
		return NULL;
	}
	ret = QRinput_Struct_insertStructuredAppendHeaders(s);
	if(ret < 0) {
		QRinput_Struct_free(s);
		return NULL;
	}

	return s;

ABORT:
	QRinput_free(input);
	QRinput_Struct_free(s);
	return NULL;
}
```
在箭头所示的情况下，`prev`本身是NULL，而`prev->next`是在访问废弃的空指针。所以报警告了。解决办法就是把这一句去掉。


#### 6、`Potential leak of memory pointed to by 'input'`
```
Potential leak of memory pointed to by 'input'
```
截图如下：
![](https://raw.githubusercontent.com/univer2012/univer2012.github.io/master/2018/%E4%BD%BF%E7%94%A8%E9%9D%99%E6%80%81Analyze%E6%89%BE%E5%88%B0%E7%9A%84%E5%B8%B8%E8%A7%81%E6%8F%90%E7%A4%BA_4.png)

代码还是上面错误那段。
错误原因是：引用对象在最后没有释放掉，需要把这2个对象释放掉。
```
//...
/*为解决：Potential lead of memory pointed to by 'input'*/
    QRinput_free(input);
	ret = QRinput_Struct_insertStructuredAppendHeaders(s);
	if(ret < 0) {
		QRinput_Struct_free(s);
		return NULL;
	}

	return s;

ABORT:
    /*为解决：Potential lead of memory pointed to by 'p'*/
    QRinput_free(p);
	QRinput_free(input);
	QRinput_Struct_free(s);
	return NULL;
	//...
```

#### 7、`Property of mutable type 'NSMutableURLRequest' has 'copy' attribute; an immutable object will be stored instead`

```
Property of mutable type 'NSMutableURLRequest' has 'copy' attribute; an immutable object will be stored instead
```
翻译过来是：可变类型的属性 NSMutableURLRequest 有copy属性；一个不可改变的对象将代替它被存储。
代码截图如下：
![](https://raw.githubusercontent.com/univer2012/univer2012.github.io/master/2018/%E4%BD%BF%E7%94%A8%E9%9D%99%E6%80%81Analyze%E6%89%BE%E5%88%B0%E7%9A%84%E5%B8%B8%E8%A7%81%E6%8F%90%E7%A4%BA_5.png)
也就是说，可变类型的属性，是不可以用用copy修饰符去修饰的，否则变成了不可变类型。 
解决办法是：把这个copy修饰符去掉：
```
@property (readwrite, nonatomic) NSMutableURLRequest *request;
```

#### 8、Null pointer argument in call to CFRelease
```
Null pointer argument in call to CFRelease
```
Null指针调用了CFRelease。
代码截图：
![](https://raw.githubusercontent.com/univer2012/univer2012.github.io/master/2018/%E4%BD%BF%E7%94%A8%E9%9D%99%E6%80%81Analyze%E6%89%BE%E5%88%B0%E7%9A%84%E5%B8%B8%E8%A7%81%E6%8F%90%E7%A4%BA_6.png)
部分代码如下：
```
for (NSInteger j = 0; j < multiPropertiesTotal; j++) {
                ABPropertyID property = multiProperties[j];
                ABMultiValueRef valuesRef = ABRecordCopyValue(person, property);
                NSInteger valuesCount = 0;
                if (valuesRef != nil) valuesCount = ABMultiValueGetCount(valuesRef);
                if (valuesCount == 0) {
                    CFRelease(valuesRef);
                    continue;
                }
                //获取电话号码和email
                for (NSInteger k = 0; k < valuesCount; k++) {
                    CFTypeRef value = ABMultiValueCopyValueAtIndex(valuesRef, k);
                    switch (j) {
                        case 0: {// Phone number
                            contact.tel = (__bridge NSString*)value;
                            break;
                        }
                        case 1: {// Email
                            contact.email = (__bridge NSString*)value;
                            break;
                        }
                        case 2: {// 住址
                            contact.address = (__bridge NSString*)value;
                            break;
                        }
                    }
                    CFRelease(value);
                }
                CFRelease(valuesRef);
            }
```
解决办法是：在valuesRef为非Null的情况下，使用完成就释放掉：
```
for (NSInteger j = 0; j < multiPropertiesTotal; j++) {
                ABPropertyID property = multiProperties[j];
                ABMultiValueRef valuesRef = ABRecordCopyValue(person, property);
                NSInteger valuesCount = 0;
                /*为解决：Null pointer argument in call to CFRelease*/
                if (valuesRef != nil) {
                    valuesCount = ABMultiValueGetCount(valuesRef);
                }
                if (valuesCount == 0) {
                    if (valuesRef) {
                        CFRelease(valuesRef);
                    }
                    continue;
                }
                /*if (valuesRef != nil) valuesCount = ABMultiValueGetCount(valuesRef);
                if (valuesCount == 0) {
                    CFRelease(valuesRef);
                    continue;
                }*/
                //获取电话号码和email
                for (NSInteger k = 0; k < valuesCount; k++) {
                    CFTypeRef value = ABMultiValueCopyValueAtIndex(valuesRef, k);
                    switch (j) {
                        case 0: {// Phone number
                            contact.tel = (__bridge NSString*)value;
                            break;
                        }
                        case 1: {// Email
                            contact.email = (__bridge NSString*)value;
                            break;
                        }
                        case 2: {// 住址
                            contact.address = (__bridge NSString*)value;
                            break;
                        }
                    }
                    CFRelease(value);
                }
                CFRelease(valuesRef);
            }
```

#### 9、`Value stored to 'size' is never read`

```
Value stored to 'size' is never read
```
代码示例如下：
```
- (NSString*) newXMLString
{
    NSMutableString *xmlString = [[NSMutableString alloc] initWithString:@"<?xml version=\"1.0\" encoding=\"utf-8\"?>"];
    NSStack *stack = [[NSStack alloc] init];
    NSArray  *keys = nil;
    NSString *key  = nil;
    NSObject *value    = nil;
    NSObject *subvalue = nil;
    NSInteger size = 0;
    [stack push:self];
    while (![stack empty])
    {
        value = [stack top];
        [stack pop];
        if (value)
        {
            if ([value isKindOfClass:[NSString class]])
            {
                [xmlString appendFormat:@"</%@>", value];
            }
            else if([value isKindOfClass:[NSDictionary class]])
            {
                keys = [(NSDictionary*)value allKeys];
                size = [(NSDictionary*)value count];
                for (key in keys)
                {
                    subvalue = [(NSDictionary*)value objectForKey:key];
                    if ([subvalue isKindOfClass:[NSDictionary class]])
                    {
                        [xmlString appendFormat:@"<%@>", key];
                        [stack push:key];
                        [stack push:subvalue];
                    }
                    else if([subvalue isKindOfClass:[NSString class]])
                    {
                        [xmlString appendFormat:@"<%@>%@</%@>", key, subvalue, key];
                    }
                }
            }
        }
    }
    return xmlString;
}
```
其中size就是从来就没有使用过的字段。所以把它删掉即可。