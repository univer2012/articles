来自：[iOS算法提升之四(买卖股票的最佳时机)](https://blog.csdn.net/ZhaiAlan/article/details/105977432)



---



### 题目

买卖股票的最佳时机

题目解析： 给定一个数组，它的第 i 个元素是一直给定股票 第 i 天的加个。

如果只允许完成一逼交易(即 买入和卖出一直股票一次)，设计一个算法来计算你所能获取的最大利润

注意： 你不能在买入股票之前，卖出股票

输入：[7,1,5,3,6,4]

输出：5

解释：在第2天的时候股票价格=1 的时候买入，在第5天股票价格 = 6 的时候卖出，最大利润 = 6-1 =5； 



### 暴力破解法

两层for循环，获取差值中最大值

```objectivec
/**
 暴力破解法，二次for循环获取差值最大值
 **/
void function1(NSArray *dataArr) {
    int max = 0;
    int startIndex = 0;
    int saleIndex = 0;
 
    for (int i = 0 ; i < dataArr.count; i++) {
        for (int j = i+1; j<dataArr.count; j++) {
            //获取第j天股票价格
            int indexj = [dataArr[j] intValue];
            //获取第i天股票价格
            int indexi = [dataArr[i] intValue];
            int priceSpread  = indexj - indexi;
            if (priceSpread > max) {
                startIndex = i;
                saleIndex = j;
            }
            max = max> priceSpread ? max : priceSpread;
            
        }
        
    }
    NSLog(@"第%d天买入，第%d天卖出，最大赚取max ----%d ",startIndex+1,saleIndex +1,max);
}
```

### 贪心算法

先获取数组中最小值，然后做差值的出最大差价

```objectivec
/**
动态规划
 1.获取最少购入价格
 2.判断最大售出价格
 **/
void function2(NSArray *dataArr) {
    int min = [dataArr[0] intValue];    //最小值
    int maxPrice = 0;   //最大赚取
    int startIndex = 0;
    int saleIndex = 0;
 
    for (int i = 1 ; i < dataArr.count; i++) {
        int indexi = [dataArr[i] intValue];
        //首先获取最少购入价格
        if (min > indexi) {
            min = indexi;
            //获取最大售出价格
        } else if (indexi - min > maxPrice) {
            /**
             切记这个开始位置，不是min的位置，
             如果数组NSArray *arr = @[@7,@2,@5,@13,@1,@5,@3,@6,@4,@8];
             是上面的话，min最终结果是1， startIndex应该是2的位置
             **/
            startIndex = [dataArr containsObject:@(min)];
            saleIndex = i;
            maxPrice = indexi - min;
        }
    }
    
    NSLog(@"第%d天买入，第%d天卖出，最大赚取max ----%d ",startIndex+1,saleIndex +1,maxPrice);
}
```

调用方法查看运行时间：

```objectivec
int main(int argc, const char * argv[]) {
    @autoreleasepool {
        // insert code here...
//        NSArray *arr = @[@7,@2,@5,@13,@1,@5,@3,@6,@4,@8];
        NSArray *arr = @[@7,@1,@5,@3,@6,@4];
        CFAbsoluteTime   StartTime1 = CFAbsoluteTimeGetCurrent();
        function1(arr);
        CFAbsoluteTime EndTime1 = CFAbsoluteTimeGetCurrent();
        NSLog(@"function1执行时间为：----%f",EndTime1 - StartTime1);
 
        CFAbsoluteTime   StartTime2 = CFAbsoluteTimeGetCurrent();
        function2(arr);
        CFAbsoluteTime EndTime2 = CFAbsoluteTimeGetCurrent();
 
        NSLog(@"function2执行时间为：----%f",EndTime2 - StartTime2);
    }
    return 0;
}
```

可以从打印结果上看出，算法优化后结果还是很明显的 

![](https://img-blog.csdnimg.cn/20200507180916499.png)

这里我们只需要最大差值，可以将开始时间，和结束时间去掉再看下输出结果 

![](https://img-blog.csdnimg.cn/20200507181128234.png)

因为在获取初始值是，第二个有使用到数组中获取元素位置，可以看出第二个算法中优化时间有有所降低，第一个没有什么变化；



### 总结：

由此可以看出，选对正确的算法，在开发中还是很重要的，作为iOS开发，其中使用的是iOS中语法，请大家见谅，这个算法不是最优解，只是小编的解法，能够完成这个,大家可以一起讨论学习;有兴趣可以在git中下载[demo](https://github.com/zhaiAlan/ArithmeticLearning.git)，也可以试试自己写出更好的算法。



---

【完】