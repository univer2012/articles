## web浏览器到原生app的Handoff

来自：[Web Browser–to–Native App Handoff](https://developer.apple.com/library/archive/documentation/UserExperience/Conceptual/Handoff/AdoptingHandoff/AdoptingHandoff.html#//apple_ref/doc/uid/TP40014338-CH2-SW10)

---

在相反的情况下,如果用户在原设备使用web浏览器，且接收设备是一个带有原生app的iOS设备，该app声明了`webpageURL`属性的域部分，那么iOS启动原生app，并向其发送一个`NSUserActivity`对象，该对象的`activityType`值为`NSUserActivityTypeBrowsingWeb`。当`userInfo`字典为空时，`webpageURL`属性包含用户正在访问的URL。

接收设备上的原生app必须通过在`com.apple.developer.associated-domains`权限中声明一个域来选择这种行为。该权限的值的格式为`<service>:<fully qualified domain name>`，例如`activitycontinuation:example.com`。在这种情况下，服务必须使`activitycontinuation`。要匹配关联域名的所有子域名，可以在特定域的开头前面加上前缀`*.`（需要使用句点）来指定通配符。在Xcode的`TARGETS`设置的`Capabilities`选项卡下的`Associated Domains`部分，添加`com.apple.developer.associated-domains`权限。你指定的域名应该不超过20到30个。

如果域名匹配`webpageURL`属性，Handoff从域名中下载被认可的app IDs 列表。域名被认可的apps被授权继续该activity。在你的网站上，你可以在一个叫`apple-app-site-association`的JSON文件中列出被认可的apps，例如，`https://example.com/apple-app-site-association`。(你必须用真实设备，而不是模拟器，来测试下载JSON文件。)Handoff 首先在`.well-known`子目录查找该文件(例如，`https://example.com/apple-app-site-association`)，如果不使用`.well-known`子目录，则返回到顶层域名。

该JSON文件包含一个字典，它在`TARGETS`设置的`General`选项卡中以格式`<team identifier>.<bundle identifier>`指定了一个app identifiers列表，例如，`YWBN8XTPBJ.com.example.myApp`。清单2-8展示了一个用于读取的格式化JSON文件示例。

清单2-8 服务器端web凭证
```
{
    "activitycontinuation": {
    "apps": [    "YWBN8XTPBJ.com.example.myApp",
                 "YWBN8XTPBJ.com.example.myOtherApp" ]
    }
}
```

> NOTE
> 在iOS9.3.1及更高版本中运行的app中，文件`apple-app-site-association`(未压缩)的大小必须不大于128KB，不论该文件是否签名。因为关联文件中的路径列表应该保持简短，所以你可以使用通配符匹配来匹配更大的路径集。

如果你的app运行在iOS 9及更高版本，`apple-app-site-association`文件可能是一个MIME类型为`application/json`的JSON文件，并且不需要对其签名。如果你的app运行在iOS 8中，该文件必须由一个有效的TLS证书签署CMS，且它的MIME类型为`application/pkcs7-mime`。要对该JSON文件签名，请将内容放入文本文件并对其签名。你可以使用如清单2-9所示的终端命令执行此任务。为便于操作，删除文本中的空白，并将`openssl`命令与证书和密钥一起用于由iOS信任的证书颁发机构颁发的标识（即，在[http://support.apple.com/kb/ht5012](http://support.apple.com/kb/ht5012) 中列出）。它不需要是承载web凭据的相同identity(示例清单中的https://example.com)，但必须是所涉及域名的有效TLS证书。

清单2-9 签名凭据文件
```
echo '{"activitycontinuation":{"apps":["YWBN8XTPBJ.com.example.myApp",
"YWBN8XTPBJ.com.example.myOtherApp"]}}' > json.txt

cat json.txt | openssl smime -sign -inkey example.com.key
                             -signer example.com.pem
                             -certfile intermediate.pem
                             -noattr -nodetach
                             -outform DER > apple-app-site-association
```

`openssl`命令的输出是JSON文件，您将其放在网站上的`apple-app-site-association` URL中，在这个例子中，是`https://example.com/apple-app-site-association`。

app可以将`webpageURL`属性设置为任何网页URL，但是它只接收`webpageURL`域位于其`com.apple.developer.associated-domains`权限中的的activity对象。此外，`webpageURL`的方案必须是`http`或`https`。任何其他方案都会抛出异常。






