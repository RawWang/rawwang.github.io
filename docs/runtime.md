# iOS Runtime学习实例
---

## 1.[self class] 与 [super class]的区别

> 以下代码段输出结果

``` Objective-C
@implementation Son : Father
- (id)init{
  self = [super init];
  if(self){
    NSlog(@"%@",NSStringFromClass([self class]));
    NSlog(@"%@",NSStringFromClass([super class]));
  }
  return self;
}
```

**分析**

super是一个"编译器标识符"，它告诉编译器，去调用父类的方法。

[super class]调用的是runtime的objc_msgSendSuper方法。

[self class]调用的是runtime中的objc_msgSend。

``` Objective-C

OBJC_EXPORT void objc_msgSendSuper(void /* struct objc_super *super, SEL op, ... */ )

/// Specifies the superclass of an instance.
struct objc_super {
    /// Specifies an instance of a class.
    __unsafe_unretained id receiver;

    /// Specifies the particular superclass of the instance to message.
#if !defined(__cplusplus)  &&  !__OBJC2__
    /* For compatibility with old objc-runtime.h header */
    __unsafe_unretained Class class;
#else
    __unsafe_unretained Class super_class;
#endif
    /* super_class is the first class to search */
};

```

objc_msgSendSuper方法可以有两个参数:
> `objc_super`的结构体  
> 方法选择器`op`  

其中`objc_super`这个结构体包含两个变量
> 接收消息的`receiver`
> 当前类的父类`super_class`  

`objc_msgSendSuper`的工作原理:  
从`objc_super`结构体指向的`superClass`父类的方法列表开始查找`selector`，找到后以`objc->receiver`去调用父类的这个`selector`，最后调用者是`objc_recevier`，而不是`super_class`，那么`objc_msgSendSuper`最后则变成:  
```Objective-C
objc_msgSend(objc_super->receiver,@selector(class))
+(class)class{
  return self;
}
```
因此结果都是Son。
