title: iOS NSSortOptions
---
NSSortOptions是iOS中对数组（NSMutableArray）进行排序时的一个可选项。

{% codeblock lang:objc %}
typedef NS_OPTIONS(NSUInteger, NSSortOptions) {
    NSSortConcurrent = (1UL << 0),
    NSSortStable = (1UL << 4),
};
{% endcodeblock %}

官方的解释：

- NSSortConcurrent

	Specifies that the Block sort operation should be concurrent.
	（排序操作应该同步进行）

	This option is a hint and may be ignored by the implementation under some circumstances; the code of the Block must be safe against concurrent invocation.
	（这是一种暗示——可能在某些特定的情况下会忽视这个设置；代码块中的代码必须是线程安全的。）

- NSSortStable

	Specifies that the sorted results should return compared items have equal value in the order they occurred originally.
	（假如两个对象的对比值是相同的，那么排序后他们的前后位置应该跟排序前一样。）

	If this option is unspecified equal objects may, or may not, be returned in their original order.
	（假如没有指定这个设置，两个对比值相同的对象在排序后的前后位置是无法确定的。）


在[stackoverflow](http://stackoverflow.com/questions/9794957/documentation-for-nssortstable-is-ungrammatical-what-is-it-trying-to-say)中对NSSortStable进行了说明。

在[排序算法](https://zh.wikipedia.org/wiki/%E6%8E%92%E5%BA%8F%E7%AE%97%E6%B3%95)中说了排序算法的稳定性，并列出了稳定排序和非稳定排序的算法列表。


通常情况下，若以同步的方式执行一个方法，这个方法会立刻返回，执行下一个方法。但是，假如你对NSSortConcurrent做过测试，你会发现排序方法并不会立刻返回；并且有时执行排序的线程是main thread，有时是另一个concurrent线程。

对于以NSSortConcurrent的方式进行排序，排序方法不立刻返回，[这个网站](http://q2a.science/ios-determine-when-nsmutablearray-sortusingcomparator-is-complete-i607576.htm)有人给出了说明：
{% blockquote %}
Interestingly, all the array sorting methods operate synchronously, even sortWithOptions:usingComparator: with an options value of NSSortConcurrent.
The docs are silent on this point, but I just tested and confirmed. I wrote a test comparator block that logged the time and thread id, and logged the time before and after the sort. With a large enough array I as able to see the comparator block firing concurrently from different threads, but the sort method did not return until the sort process was complete.
I just posted the following documentation comment:
The documentation for the method sortWithOptions:usingComparator: is incomplete.
This method always operates synchronously, even when you specify an opts value of NSSortConcurrent.
Put in plain english: 'This method does not return until the array is sorted, even if you specify sort options of NSSortConcurrent". In that case the comparator blocks may be invoked on background threads, but the method waits until the sort is complete before returning control to the caller.
Let's all send in feedback on this issue, since Apple is more likely to fix something if they receive multiple reports about it.
{% endblockquote %}
所以，NSSortConcurrent虽然是以同步的方式执行的，但是在iOS中排序方法并不会立刻返回，而是等到排序完成后才返回。这种方式虽然将排序的工作放到了其他线程，但是还是后阻塞main thread。苹果要是能够提供一个真正的同步排序方法就太好了。不过，你可以手动以多线程的方式执行排序方法（GCD等）。


通过下面的代码查看一下进行排序操作的线程：

{% codeblock lang:objc %}
    NSMutableArray *array = [NSMutableArray array];
    for (int i = 0; i < 160; i++) {
        switch (i%4) {
            case 0:
                [array addObject:@"a"];
                break;
            case 1:
                [array addObject:@"aa"];
                break;
            case 2:
                [array addObject:@"aaa"];
                break;
            case 3:
                [array addObject:@"aaaa"];
                break;
            default:
                break;
        }
    }
    [array sortWithOptions:NSSortConcurrent usingComparator:^NSComparisonResult(id obj1, id obj2) {
        if ( [obj1 length] < [obj2 length] )
            return NSOrderedAscending;
        if ( [obj1 length] > [obj2 length] )
            return NSOrderedDescending;
        return NSOrderedSame;
    }];
    [array sortWithOptions:NSSortStable usingComparator:^NSComparisonResult(id obj1, id obj2) {
        if ( [obj1 length] > [obj2 length] )
            return NSOrderedAscending;
        if ( [obj1 length] < [obj2 length] )
            return NSOrderedDescending;
        return NSOrderedSame;
    }];
    [array sortWithOptions:0 usingComparator:^NSComparisonResult(id obj1, id obj2) {
        if ( [obj1 length] < [obj2 length] )
            return NSOrderedAscending;
        if ( [obj1 length] > [obj2 length] )
            return NSOrderedDescending;
        return NSOrderedSame;
    }];
{% endcodeblock %}

- 将Options设置为0：
{% img [sort options 0] http://justben.me/images/ios_sort_options_0.png [将Options设置为0]%}

- 将Options设置为NSSortConcurrent，且将数组的长度设置为159：
{% img [sort options concurrent 159] http://justben.me/images/ios_sort_options_concurrent_159.png [将Options设置为NSSortConcurrent，且将数组的长度设置为159]%}

- 将Options设置为NSSortConcurrent，且将数组的长度设置为160：
{% img [sort options concurrent 160] http://justben.me/images/ios_sort_options_concurrent_160.png [将Options设置为NSSortConcurrent，且将数组的长度设置为160]%}

在NSSortConcurrent的官方说明中提到：在某些特殊的情况下NSSortConcurrent的设置可能会被忽略。从上面的截图中，当数组长度为159时，NSSortConcurrent的设置就被忽略了。应该是在排序中有一种检测机制，只有当排序操作对系统资源的消耗超过某个上限才会使用同步的方式。

- 将Options设置为NSSortStable：
{% img [sort options stable] http://justben.me/images/ios_sort_options_stable.png [将Options设置为NSSortStable]%}

