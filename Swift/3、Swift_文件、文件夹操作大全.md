来自：[Swift - 文件，文件夹操作大全](https://www.hangge.com/blog/cache/detail_527.html)



iOS开发经常会遇到读文件，写文件等，对文件和文件夹的操作，这时就可以使用FileManager，FileHandle等类来实现。 

下面总结了各种常用的操作：
 

### 1，遍历一个目录下的所有文件

 假设用户文档下有如下文件和文件夹：test1.txt、fold1/test2.txt 

![原文:Swift - 文件，文件夹操作大全](https://raw.githubusercontent.com/univer2012/personal-document/master/Pictures/2019/swift_file_folder1.png)



（1）首先我们获取用户文档目录路径

```swift
let manager = FileManager.default
let urlForDocument = manager.urls(for: .documentDirectory, in:.userDomainMask)
let url = urlForDocument[0] as URL
print(url)
```

打印：

```
file:///Users/mac/Library/Developer/XCPGDevices/661E1E96-54AF-4302-8F96-12A2AAB137EE/data/Containers/Data/Application/977545AD-7554-4F84-8C86-7482CDF27B69/Documents/
```



 （2）对指定路径执行浅搜索，返回指定目录路径下的文件、子目录及符号链接的列表  

```swift
let contentsOfPath = try? manager.contentsOfDirectory(atPath: url.path)
print("contentsOfPath: \(contentsOfPath)")
```

打印：

```
contentsOfPath: Optional(["test1.txt", "fold1"])
```





 （3）类似上面的，对指定路径执行浅搜索，返回指定目录路径下的文件、子目录及符号链接的列表 

```swift
let contentsOfURL = try? manager.contentsOfDirectory(at: url,
                        includingPropertiesForKeys: nil, options: .skipsHiddenFiles)
print("contentsOfURL: \(contentsOfURL)")
```

打印：

```
contentsOfURL: Optional([file:///Users/mac/Library/Developer/XCPGDevices/661E1E96-54AF-4302-8F96-12A2AAB137EE/data/Containers/Data/Application/81832EDB-34BD-4783-8E68-33285B8B05A7/Documents/test1.txt, file:///Users/mac/Library/Developer/XCPGDevices/661E1E96-54AF-4302-8F96-12A2AAB137EE/data/Containers/Data/Application/81832EDB-34BD-4783-8E68-33285B8B05A7/Documents/fold1/])
```




 （4）深度遍历，会递归遍历子文件夹（但不会递归符号链接） 

```swift
let enumeratorAtPath = manager.enumerator(atPath: url.path)
print("enumeratorAtPath: \(enumeratorAtPath?.allObjects)")
```

打印：

```
enumeratorAtPath: Optional([test1.txt, fold1, fold1/test2.txt])
```



 （5）类似上面的，深度遍历，会递归遍历子文件夹（但不会递归符号链接） 

```swift
let enumeratorAtURL = manager.enumerator(at: url, includingPropertiesForKeys: nil, options: FileManager.DirectoryEnumerationOptions.skipsHiddenFiles, errorHandler: nil)
print("enumeratorAtURL: \(enumeratorAtURL?.allObjects)")
```

打印：

```
enumeratorAtURL: Optional([file:///Users/mac/Library/Developer/XCPGDevices/661E1E96-54AF-4302-8F96-12A2AAB137EE/data/Containers/Data/Application/81832EDB-34BD-4783-8E68-33285B8B05A7/Documents/test1.txt, file:///Users/mac/Library/Developer/XCPGDevices/661E1E96-54AF-4302-8F96-12A2AAB137EE/data/Containers/Data/Application/81832EDB-34BD-4783-8E68-33285B8B05A7/Documents/fold1/, file:///Users/mac/Library/Developer/XCPGDevices/661E1E96-54AF-4302-8F96-12A2AAB137EE/data/Containers/Data/Application/81832EDB-34BD-4783-8E68-33285B8B05A7/Documents/fold1/test2.txt])
```



（6）深度遍历，会递归遍历子文件夹（包括符号链接，所以要求性能的话用`enumeratorAtPath`） 

```swift
let subPaths = manager.subpaths(atPath: url.path)
print("subPaths: \(subPaths)")
```

打印：

```
subPaths: Optional([".DS_Store", "test1.txt", "fold1", "fold1/test2.txt"])
```





### 2，判断文件或文件夹是否存在 

```swift
let fileManager = FileManager.default
let filePath: String = NSHomeDirectory() + "/Documents/univer2012.txt"
let exist = fileManager.fileExists(atPath: filePath)
print("univer2012.txt is exist:\(exist)")
```

打印：

```
univer2012.txt is exist:false
```



### 3，创建文件夹 

方式1：

```swift
let myDirectory: String = NSHomeDirectory() + "/Documents/myFolder/Files"
let fileManager = FileManager.default

//withIntermediateDirectories为true，表示路径之间如果有不存在的文件夹都会创建
try! fileManager.createDirectory(atPath: myDirectory, withIntermediateDirectories: true, attributes: nil)
```

![](https://raw.githubusercontent.com/univer2012/personal-document/master/Pictures/2019/swift_file_folder2.png)



方式2：

```swift
func createFolder(name: String, baseUrl: NSURL) {
    let manager = FileManager.default
    let folder = baseUrl.appendingPathComponent(name, isDirectory: true)
    print("文件夹：\(folder)")
    let exist = manager.fileExists(atPath: folder!.path)
    if !exist {
        try! manager.createDirectory(atPath: folder!.path, withIntermediateDirectories: true, attributes: nil)
    }
}

//在文档目录下新建folder目录
let manager = FileManager.default
let urlForDocument = manager.urls(for: .documentDirectory, in: .userDomainMask)
let url = urlForDocument[0] as NSURL
createFolder(name: "folder", baseUrl: url)
```

![](https://raw.githubusercontent.com/univer2012/personal-document/master/Pictures/2019/swift_file_folder3.png)



### 4，将对象写入文件

可以通过write(to:)方法，可以创建文件并将对象写入，对象包括String，NSString，UIImage，NSArray，NSDictionary等。
 **（1）把String保存到文件** 

```swift
let filePath: String = NSHomeDirectory() + "/Documents/univer2012.txt"
let info = "欢迎来到univer2012.com"
try! info.write(toFile: filePath, atomically: true, encoding: String.Encoding.utf8)
```

![](https://raw.githubusercontent.com/univer2012/personal-document/master/Pictures/2019/swift_file_folder4.png)



**（2）把图片保存到文件路径下**

先把图片拖入Playground的Resources中，然后在运行如下代码：

```swift
let filePath: String = NSHomeDirectory() + "/Documents/univer2012.txt"
let image = UIImage(named: "apple")
if let data: Data = image?.pngData() {
    try? data.write(to: URL(fileURLWithPath: filePath))
}
```

![](https://raw.githubusercontent.com/univer2012/personal-document/master/Pictures/2019/swift_file_folder5.png)



**（3）把NSArray保存到文件路径下** 

```swift
let array = NSArray(objects: "aaa", "bbb", "ccc")
let filePath: String = NSHomeDirectory() + "/Documents/array.plist"
array.write(toFile: filePath, atomically: true)
```



**（4）把NSDictionary保存到文件路径下** 

```swift
let dictionary: NSDictionary = ["Gold": "1st Place", "Silver": "2nd Place"]
let filePath: String = NSHomeDirectory() + "/Documents/dictionary.plist"
try! dictionary.write(toFile: filePath, atomically: true)
```



### 5，创建文件

```swift
func createFolder(name: String, fileBaseUrl: URL) {
    let manager = FileManager.default
    
    let file = fileBaseUrl.appendingPathComponent(name)
    print("文件夹：\(file)")
    let exist = manager.fileExists(atPath: file.path)
    if !exist {
        let data = Data(base64Encoded: "aGVsbG8gd29ybGQ=", options: .ignoreUnknownCharacters)
        let createSuccess = manager.createFile(atPath: file.path, contents: data, attributes: nil)
        print("文件创建结果： \(createSuccess)")
    }
}

//在文档目录下新建test.txt文件
let manager = FileManager.default
let urlForDocument = manager.urls(for: .documentDirectory, in: .userDomainMask)
let url = urlForDocument[0]
createFolder(name: "folder/new.txt", fileBaseUrl: url)
```

打印：

```
文件夹：file:///Users/mac/Library/Developer/XCPGDevices/661E1E96-54AF-4302-8F96-12A2AAB137EE/data/Containers/Data/Application/81832EDB-34BD-4783-8E68-33285B8B05A7/Documents/folder/new.txt
文件创建结果： true
```



### 6，复制文件

（1）方法1 

```swift
let fileManager = FileManager.default
let homeDirectory = NSHomeDirectory()
let scrUrl = homeDirectory + "/Documents/univer2012.txt"
let toUrl = homeDirectory + "/Documents/copyed.txt"
try? fileManager.copyItem(atPath: scrUrl, toPath: toUrl)
```

（2）方法2 

```swift
//定位到用户文档目录
let manager = FileManager.default
let urlForDocument = manager.urls(for: .documentDirectory, in: .userDomainMask)
let url = urlForDocument[0]

//将test.txt文件拷贝到文档目录根目录下的copyed.txt文件
let srcUrl = url.appendingPathComponent("test1.txt")
let toUrl = url.appendingPathComponent("copyed1.txt")

try? manager.copyItem(at: srcUrl, to: toUrl)
```



### 7，移动文件 

 （1）方法1 

注意：

1、要先确保该目录下有`\moved\` 这个文件目录，不然会崩溃

2、要确保`\moved\` 目录下没有`univer2012.txt`文件，否则会崩溃

```swift
let fileManager = FileManager.default
let homeDirectory = NSHomeDirectory()
let srcUrl = homeDirectory + "/Documents/univer2012.txt"
let toUrl = homeDirectory + "/Documents/moved/univer2012.txt"
try? fileManager.moveItem(atPath: srcUrl, toPath: toUrl)
```

（2）方法2 

注意：

1、要确保目标目录存在，且没有`copyed2.txt`文件，否则移动失败，

```swift
//定位到用户文档目录
let manager = FileManager.default
let urlForDocument = manager.urls(for: .documentDirectory, in: .userDomainMask)
let url = urlForDocument[0]

let srcUrl = url.appendingPathComponent("test1.txt")
let toUrl = url.appendingPathComponent("copyed2.txt")
// 移动srcUrl中的文件（test1.txt）到toUrl中（copyed.2txt）
try? manager.moveItem(at: srcUrl, to: toUrl)
```



### 8，删除文件 

 （1）方法1 

注意：要确保要删除的文件目录是正确的，且该目录下有指定的文件，否则删除失败

```swift
let fileManager = FileManager.default
let homeDirectory = NSHomeDirectory()
let srcUrl = homeDirectory + "/Documents/moved/univer2012.txt"
try? fileManager.removeItem(atPath: srcUrl)
```



（2）方法2 

```swift
//定位到用户文档目录
let manager = FileManager.default
let urlForDocument = manager.urls(for: .documentDirectory, in: .userDomainMask)
let url = urlForDocument[0]

let toUrl = url.appendingPathComponent("copyed.txt")
// 删除文档根目录下的toUrl路径的文件（copyed.txt文件）
try? manager.removeItem(at: toUrl)
```



###  9，删除目录下所有的文件

 （1）方法1：获取所有文件，然后遍历删除 

```swift
let fileManager = FileManager.default
let myDirectory = NSHomeDirectory() + "/Documents/folder"
if let fileArray = fileManager.subpaths(atPath: myDirectory) {
    for fn in fileArray {
        try? fileManager.removeItem(atPath: myDirectory + "/\(fn)")
    }
}
```



（2）方法2：删除目录后重新创建该目录

```swift
let fileManager = FileManager.default
let myDirectory = NSHomeDirectory() + "/Documents/folder"
try? fileManager.removeItem(atPath: myDirectory)

try? fileManager.createDirectory(atPath: myDirectory, withIntermediateDirectories: true, attributes: nil)

```



### 10，读取文件 

```swift
let manager = FileManager.default
let urlForDocDirectory = manager.urls(for: .documentDirectory, in: .userDomainMask)
let docPath = urlForDocDirectory[0]
let file = docPath.appendingPathComponent("test.txt")

//方法1
let readHandler = try? FileHandle(forReadingFrom: file)
if let data = readHandler?.readDataToEndOfFile() {
    let readString = String(data: data, encoding: String.Encoding.utf8)
    print("文件内容： \(readString)")
}

//方法2
if let data2 = manager.contents(atPath: file.path) {
    let readString2 = String(data: data2, encoding: String.Encoding.utf8)
    print("文件内容2： \(readString2)")
}
```



### 11，在任意位置写入数据 

```swift
let manager = FileManager.default
let urlForDocDirectory = manager.urls(for: .documentDirectory, in: .userDomainMask)
let docPath = urlForDocDirectory[0]
let file = docPath.appendingPathComponent("test.txt")

let string = "添加一些文字到文件末尾"
if let appendedData = string.data(using: String.Encoding.utf8, allowLossyConversion: true),
    let writeHandler = try? FileHandle(forWritingTo: file) {
    
    writeHandler.seekToEndOfFile()
    writeHandler.write(appendedData)
}
```



### 12，文件权限判断 

```swift
let manager = FileManager.default
let urlForDocDirectory = manager.urls(for: .documentDirectory, in: .userDomainMask)
let docPath = urlForDocDirectory[0]
let file = docPath.appendingPathComponent("test.txt")

let readable = manager.isReadableFile(atPath: file.path)
print("可读： \(readable)")
let writeable = manager.isWritableFile(atPath: file.path)
print("可写： \(writeable)")
let executable = manager.isExecutableFile(atPath: file.path)
print("可执行： \(executable)")
let deleteable = manager.isDeletableFile(atPath: file.path)
print("可删除： \(deleteable)")
```

打印如下：

```
可读： true
可写： true
可执行： false
可删除： true
```



### 13，获取文件属性（创建时间，修改时间，文件大小，文件类型等信息）

```swift
let manager = FileManager.default
let urlForDocDirectory = manager.urls(for: .documentDirectory, in: .userDomainMask)
let docPath = urlForDocDirectory[0]
let file = docPath.appendingPathComponent("test.txt")

let attributes = try? manager.attributesOfItem(atPath: file.path) //结果为[FileAttributeKey : Any]类型
print("attributes: \(attributes)")
```

打印：

```
attributes: Optional([__C.NSFileAttributeKey(_rawValue: NSFileSystemFileNumber): 12912504674, __C.NSFileAttributeKey(_rawValue: NSFileOwnerAccountID): 501, __C.NSFileAttributeKey(_rawValue: NSFilePosixPermissions): 420, __C.NSFileAttributeKey(_rawValue: NSFileReferenceCount): 1, __C.NSFileAttributeKey(_rawValue: NSFileGroupOwnerAccountID): 20, __C.NSFileAttributeKey(_rawValue: NSFileType): NSFileTypeRegular, __C.NSFileAttributeKey(_rawValue: NSFileSize): 40, __C.NSFileAttributeKey(_rawValue: NSFileExtensionHidden): 0, __C.NSFileAttributeKey(_rawValue: NSFileCreationDate): 2019-11-26 03:32:25 +0000, __C.NSFileAttributeKey(_rawValue: NSFileExtendedAttributes): {
    "com.apple.TextEncoding" = {length = 15, bytes = 0x7574662d383b313334323137393834};
    "com.apple.lastuseddate#PS" = {length = 16, bytes = 0x91dfdc5d00000000a4e1fa0100000000};
    "com.apple.metadata:kMDLabel_hmjdj4dtfnyx5hvkxed3x5oehy" = {length = 89, bytes = 0xf2657db6 afb639d3 2524f279 9af97cba ... 6aab79f2 60b9f2c6 };
    "com.apple.quarantine" = {length = 29, bytes = 0x30303836 3b356464 63646638 633b7465 ... 7967726f 756e643b };
}, __C.NSFileAttributeKey(_rawValue: NSFileModificationDate): 2019-11-26 08:17:16 +0000, __C.NSFileAttributeKey(_rawValue: NSFileSystemNumber): 16777220])
```



从 **attributes** 中获取具体的属性：

```swift
 //结果为[FileAttributeKey : Any]类型
if let attributes = try? manager.attributesOfItem(atPath: file.path) {
    //print("attributes: \(attributes)")
    print("创建时间：\(attributes[FileAttributeKey.creationDate])")
    print("修改时间：\(attributes[FileAttributeKey.modificationDate])")
    print("文件大小：\(attributes[FileAttributeKey.size])")
}
```

打印：

```
创建时间：Optional(2019-11-26 03:32:25 +0000)
修改时间：Optional(2019-11-26 08:17:16 +0000)
文件大小：Optional(40)
```



### 14，文件/文件夹比较 

```swift
let manager = FileManager.default
let urlForDocDirectory = manager.urls(for: .documentDirectory, in: .userDomainMask)
let docPath = urlForDocDirectory[0]
let contents = try! manager.contentsOfDirectory(atPath: docPath.path)

//下面比较用户文档中前面两个文件是否内容相同（该方法也可以用来比较目录）
let count = contents.count
if count > 1 {
    let path1 = docPath.path + "/" + (contents[0] as String)
    let path2 = docPath.path + "/" + (contents[1] as String)
    let equal = manager.contentsEqual(atPath: path1, andPath: path2)
    print("path1: \(path1)")
    print("path2: \(path2)")
    print("比较结果: \(equal)")
}
```

---

【完】