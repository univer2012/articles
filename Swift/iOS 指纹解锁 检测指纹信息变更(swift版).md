来自：[iOS 指纹解锁 检测指纹信息变更](https://www.jianshu.com/p/1d497508f1cb)



---

通过LAContext evaluatedPolicyDomainState属性可以获取到当前data类型的指纹信息数据，当指纹增加或者删除，该data就会发生变化，通过记录这个TouchIdData与最新的data做对比就能判断指纹信息是否变更，从而定制app功能。

##### 存在的疑问：

1. TouchIdData可能为空吗？
    官方文档说明：
    
    > Discussion
    > This property returns a value only when the canEvaluatePolicy(*:error:) method succeeds for a biometric policy or the evaluatePolicy(*:localizedReason:reply:) method is called and a successful biometric authentication is performed. Otherwise, nil is returned.
    > 只有当canEvaluatePolicy方法执行并返回YES或者evaluatePolicy执行并指纹识别通过，这个属性才能有值，否则为空。



2. TouchIdData能否获取具体的指纹信息？

   >
   >
   >The returned data is an opaque structure. It can be used to compare with other values returned by this property to determine whether the authorized database has been updated. However, the nature of the change cannot be determined from this data.
   > 返回的数据是一个不透明的结构。它可以用来与此属性返回的其他值进行比较，以确定是否更新了授权数据库。然而，变化的性质不能从这些数据中确定。
   >
   >

3. 在指纹信息没有修改的时候，不同app获取到的TouchIdData是一样的吗?
    实测不同的app，在指纹没有变化的情况下TouchIdData是不一样的。但这个是不能打包票的，如果苹果修改了这部分的算法，返回一个相同值也是有可能的。

4. 添加一个新指纹，再删除刚添加的那个指纹，TouchIdData相对一轮操作之前变化了吗？
    实测TouchIdData没有变化，也就是说TouchIdData是面向结果的，而不是面向过程的，只要最终结果指纹集合一样，TouchIdData就一样。





代码实现

```swift
static var IDENTIFY:String? = nil
    static let SERVICE = "TOUCHID_SERVICE"
    static let ACCOUNT_PREFIX = "TOUCHID_PERFIX"
    open class func setCurrentTouchIdDataIdentity(identity:String )
    {
        //设定当前身份用于存储data
        TouchIdManager.IDENTIFY = identity
    }
  
//获取当前时刻的data
  class func currentOriTouchIdData() -> Data?{
        let context = LAContext()
        var error:NSError? = nil;
//先使用canEvaluatePolicy方法进行评估
        if context.canEvaluatePolicy(LAPolicy.deviceOwnerAuthenticationWithBiometrics, error: &error) {
            
            return context.evaluatedPolicyDomainState
        }
        print("errorMsg:" + self.errorMessageForFails(errorCode:(error?.code)! ))
        return nil
    }


//使用keychain保存当前身份的data
open class func setCurrentIdentityTouchIdData()-> Bool
    {
        if self.currentTouchIdDataIdentity() == nil
        {
            return false;
        }
        else
        {
            if self.currentOriTouchIdData() != nil
            {
                //storage by keychain
                SAMKeychain.setPasswordData(self.currentOriTouchIdData()!, forService:SERVICE, account: ACCOUNT_PREFIX + self.currentTouchIdDataIdentity()!)
                return true;
            }
            else
            {
                return false;
            }
        }
    }


//获取当前身份的上一次存储的data，用于对比
 class func currentIdentityTouchIdData()->Data?
    {
        guard (self.currentTouchIdDataIdentity() != nil) else {
            return nil;
        }
        
        return  SAMKeychain.passwordData(forService: TouchIdManager.SERVICE, account: TouchIdManager.ACCOUNT_PREFIX + self.currentTouchIdDataIdentity()!)
    }


//检测以这个身份设置开始到当前时刻指纹信息是否变更
open class func touchIdInfoDidChange()->Bool
    {
        let data = self.currentOriTouchIdData()
        if data == nil && self.isErrorTouchIDLockout() {
            //lock after unlock failed many times,and the fingerprint is not changed.
            return false
        }
        else
        {
            let oldData = self.currentIdentityTouchIdData()
            
            if oldData == nil
            {
                //never set
                return false
            }
            else if oldData == data
            {
                //not change
                return false
            }
            else
            {
                return true
            }
        }
    }


//检测当前是否为biometryLockout状态
    class func isErrorTouchIDLockout()->Bool
    {
        let context = LAContext()
        var error:NSError?
        context.canEvaluatePolicy(LAPolicy.deviceOwnerAuthenticationWithBiometrics, error: &error)
        
        guard error != nil else {
            return false
        }
        
        if error!.code == LAError.biometryLockout.rawValue {
            return true
        }
        else
        {
            return false
        }
    }
```



##### 指纹识别的两种LAPolicy：

- deviceOwnerAuthenticationWithBiometrics
   这个类型不能弹出密码解锁界面，但能更精准的反馈用户操作的状态：如指纹识别三次失败等。
- deviceOwnerAuthentication
   对识别行为的结果做了简化，无法判断具体状态。但能弹出密码解锁界面。
   结合两者可以使指纹解锁做的更友善一点。
- 最终效果[正常流程]：指纹识别错误三次回调失败->再点击再识别错误两次->弹出密码解锁界面->密码错误5次->锁定1分钟->再输错->锁定五分钟。

代码实现

```swift
 open class func showTouchId(title:String,fallbackTitle:String?, fallbackBlock:TouchIdFallBackBlokc?,resultBlock:TouchIdResultBlock?)
    {
        let context = LAContext();
        context.localizedFallbackTitle = fallbackTitle
        var useableError:NSError?
        if context.canEvaluatePolicy(LAPolicy.deviceOwnerAuthenticationWithBiometrics, error: &useableError) {
            context.evaluatePolicy(LAPolicy.deviceOwnerAuthenticationWithBiometrics, localizedReason: title) { (success, error) in
                DispatchQueue.main.async {
                    if success
                    {
                        if resultBlock != nil
                        {
                            resultBlock!(true,success,error)
                        }
                    }
                    else
                    {
                        guard let error = error else
                        {
                            return;
                        }
                        
                        print("errorMsg:" + self.errorMessageForFails(errorCode: error._code))
                        
                        if error._code == LAError.userFallback.rawValue
                        {
                            if fallbackBlock != nil
                            {
                                fallbackBlock!()
                            }
                        }
                        else if error._code == LAError.biometryLockout.rawValue
                        {
                            //try to show password interface
                            self.tryShowTouchIdOrPwdInterface(title: title, resultBlock: resultBlock)
                        }
                        else
                        {
                            if resultBlock != nil
                            {
                                resultBlock!(true,success,error)
                            }
                        }
                    }
                }
            }
        }
        else
        {
            print("errorMsg:" + self.errorMessageForFails(errorCode:(useableError?.code)! ))
            
            if useableError?.code == LAError.biometryLockout.rawValue
            {
                //try to show password interface
                self.tryShowTouchIdOrPwdInterface(title: title, resultBlock: resultBlock)
            }
            else
            {
                if resultBlock != nil
                {
                    resultBlock!(false,false,useableError)
                }
            }
        }
    }
    
    class func tryShowTouchIdOrPwdInterface(title:String,resultBlock:TouchIdResultBlock?)
    {
        let context = LAContext();
        context.localizedFallbackTitle = ""
        var useableError:NSError?
        if context.canEvaluatePolicy(LAPolicy.deviceOwnerAuthentication, error: &useableError) {
            context.evaluatePolicy(LAPolicy.deviceOwnerAuthentication, localizedReason: title) { (success, error) in
                
                DispatchQueue.main.async {
                    if resultBlock != nil
                    {
                        resultBlock!(true,success,error)
                    }
                }
                
                guard let error = error else
                {
                    return;
                }
                print("errorMsg:" + self.errorMessageForFails(errorCode: error._code))
            }
        }
        else
        {
            print("errorMsg:" + self.errorMessageForFails(errorCode:(useableError?.code)! ))
            
            if resultBlock != nil
            {
                resultBlock!(false,false,useableError)
            }
        }
    }
```





测试demo:
 swift:https://github.com/zmubai/TouchIDTest-swift
 object-c:https://github.com/zmubai/TouchIDTest-OC





---

【完】



