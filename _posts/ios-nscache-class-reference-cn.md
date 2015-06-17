title: NSCache指南
categories:
- 趴来趴去
tags:
- iOS
- NSCache
- Reference
---

An NSCache object is a collection-like container, or cache, that stores key-value pairs, similar to the NSDictionary class. Developers often incorporate caches to temporarily store objects with transient data that are expensive to create. Reusing these objects can provide performance benefits, because their values do not have to be recalculated. However, the objects are not critical to the application and can be discarded if memory is tight. If discarded, their values will have to be recomputed again when needed.

While a key-value pair is in the cache, the cache maintains a strong reference to it. A common data type stored in NSCache objects is an object that implements the NSDiscardableContent protocol. Storing this type of object in a cache has benefits, because its content can be discarded when it is not needed anymore, thus saving memory. By default, NSDiscardableContent objects in the cache are automatically removed from the cache if their content is discarded, although this automatic removal policy can be changed. If an NSDiscardableContent object is put into the cache, the cache calls discardContentIfPossible on it upon its removal.

NSCache objects differ from other mutable collections in a few ways:

The NSCache class incorporates various auto-removal policies, which ensure that it does not use too much of the system’s memory. The system automatically carries out these policies if memory is needed by other applications. When invoked, these policies remove some items from the cache, minimizing its memory footprint.

You can add, remove, and query items in the cache from different threads without having to lock the cache yourself.

Unlike an NSMutableDictionary object, a cache does not copy the key objects that are put into it.

These features are necessary for the NSCache class, as the cache may decide to automatically mutate itself asynchronously behind the scenes if it is called to free up memory.