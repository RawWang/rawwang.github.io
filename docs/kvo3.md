# KVO进阶
---

## Apple的KVO不足之处

由于系统提供的`KVO`耦合性比较高，如果在需要同时监听多个对象的属性时，那么:

```Objective C
- (void)observeValueForKeyPath:(nullable NSString *)keyPath ofObject:(nullable id)object change:(nullable NSDictionary<NSKeyValueChangeKey, id> *)change context:(nullable void *)context;
```
就需要在这个方法里通过多个`if..else`组合来判断处理，这明显不是太友好。

## 实现带Block的KVO

实现后的效果，这样就更加方便使用KVO了。

```Objective C
- (void)RW_addObserver:(NSObject *)observer forKeyPath:(NSString *)keyPath callback:(KVOCallback)callback {
    // do something here
}];

- (void)RW_addObserver:(NSObject *)observer forKeyPath:(NSString *)keyPath callback:(KVOCallback)callback {
    // do something here
}];
```
## 具体实现

- .h文件

```Objective C
#import <Foundation/Foundation.h>

@interface NSObject (kvo)
- (void)RW_addObserver:(NSObject *)observer forKeyPath:(NSString *)keyPath callback:(KVOCallback)callback;
@end
```

- .m文件

```Objective C
- (void)RW_addObserver:(NSObject *)observer forKeyPath:(NSString *)keyPath callback:(KVOCallback)callback {
    Class class = object_getClass(self);
    NSString *className = NSStringFromClass(class);

    Class kvoClass = nil;
    //创建子类
    if (![className hasPrefix:@"RWKVO_"]) {
        //定义类的派生类的名字
        NSString *oldClassName = NSStringFromClass([self class]);
        NSString *newClassName = [@"RWKVO_" stringByAppendingString:oldClassName];

        //创建子类
        kvoClass = objc_allocateClassPair(self.class, newClassName.UTF8String, 0);

        //重写class方法，修改isa指针
        object_setClass(self, kvoClass);

        //注册子类
        objc_registerClassPair(kvoClass);
        
        //添加setter方法
        class_addMethod(kvoClass, [self setterSEL:keyPath], (IMP)rw_setterWithCallback, "v@:@");
    }
    

    //添加多个观察者到观察者列表中
    RWObserverInfo *info = [[RWObserverInfo alloc] initWithObserver:observer keyPath:keyPath callback:callback];
    NSMutableArray *observers = objc_getAssociatedObject(self, @"observer");

    if (!observers) {
        observers = [NSMutableArray array];
        objc_setAssociatedObject(self, (__bridge const void *)@"observer", observers, OBJC_ASSOCIATION_RETAIN_NONATOMIC);
    }

    [observers addObject:info];
}
```
- 具体调用

```Objective C

[self.person RW_addObserver:self forKeyPath:@"age" callback:^(id  _Nonnull observer, NSString * _Nonnull key, id  _Nonnull oldValue, id  _Nonnull newValue) {
    NSLog(@"key: %@, oldValue: %@, newValue: %@", key, oldValue, newValue);
}];

[self.person RW_addObserver:self forKeyPath:@"gender" callback:^(id  _Nonnull observer, NSString * _Nonnull key, id  _Nonnull oldValue, id  _Nonnull newValue) {
    NSLog(@"key: %@, oldValue: %@, newValue: %@", key, oldValue, newValue);
}];
```

源码下载[Github](https://github.com/RawWang/RWKVO-OC)





