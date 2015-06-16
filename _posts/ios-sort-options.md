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

{% img [sort options 0] http://justben.me/images/ios_sort_options_0.png [将Options设置为0]%}
{% img [sort options concurrent 159] http://justben.me/images/ios_sort_options_concurrent_159.png [将Options设置为NSSortConcurrent，且将数组的长度设置为159]%}
{% img [sort options concurrent 160] http://justben.me/images/ios_sort_options_concurrent_160.png [将Options设置为NSSortConcurrent，且将数组的长度设置为160]%}
{% img [sort options stable] http://justben.me/images/ios_sort_options_stable.png [将Options设置为NSSortStable]%}
