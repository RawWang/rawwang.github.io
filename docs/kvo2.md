# 手动实现KVO
---

前面学习了关于系统实现KVO的几个步骤:
- 给类A的对象添加属性观察
- 运行期自动创建A的子类B
- 子类B重写`class`方法，修改`isa`指针指向子类B
- 子类B重写类A的`setter`方法，并在方法中实现通知机制
- 注册子类B

接下来看看如何手动实现**KVO**

## 代码实现

OC中所有对象的基类为`NSObject`，因此新建一个`NSObject`的category, **NSObject+KVO**

.h文件
```Objective C 
#import <Foundation/Foundation.h>

@interface NSObject (kvo)

- (void)RW_addObserver:(NSObject *)observer forKeyPath:(NSString *)keyPath options:(NSKeyValueObservingOptions)options context:(nullable void *)context;

@end
```

.m文件
```Objective C 
#import "NSObject+KVO.h"
#import <objc/runtime.h>
#import <objc/message.h>

@implementation NSObject (KVO)

- (void)RW_addObserver:(NSObject *)observer forKeyPath:(NSString *)keyPath options:(NSKeyValueObservingOptions)options context:(nullable void *)context {
    //定义类的派生类的名字
    NSString *oldClassName = NSStringFromClass([self class]);
    NSString *newClassName = [@"RWKVO_" stringByAppendingString:oldClassName];
    
    //创建子类
    Class kvoClass = objc_allocateClassPair(self.class, newClassName.UTF8String, 0);
    
    //添加setter方法
    class_addMethod(kvoClass, [self setterSEL:keyPath], (IMP)rw_setter, "v@:@");
    
    //重写class方法，修改isa指针
    object_setClass(self, kvoClass);
    
    //注册子类
    objc_registerClassPair(kvoClass);
    
    //保存观察者属性到子类中
    objc_setAssociatedObject(self, (__bridge const void *)@"observer", observer, OBJC_ASSOCIATION_RETAIN_NONATOMIC);
}

//重写父类Setter方法
void rw_setter(id self, SEL _cmd, NSString *name) {
    // 保存当前KVO的类
    Class kvoClass = [self class];
    
    // 将self的isa指针指向父类Person，调用父类setter方法
    object_setClass(self, [self superclass]);
    
    //调用父类的getter方法获取变量值
    NSString *oldValue = objc_msgSend(self, [self getterSEL:_cmd]);
    
    // 调用父类setter方法，重新给变量赋值
    objc_msgSend(self, _cmd, name);
    
    // 取出RWKVO_Person观察者
    id objc = objc_getAssociatedObject(self, (__bridge const void *)@"observer");

    //组装数据格式
    NSDictionary *dic = [[NSDictionary alloc] initWithObjectsAndKeys:[NSNumber numberWithInt:1],@"kind", name, @"new", oldValue, @"old", nil];
    
    // 通知观察者，执行通知方法
    objc_msgSend(objc, @selector(observeValueForKeyPath:ofObject:change:context:), @"name", self, dic, nil);
    
    // 重新修改为RWKVO_Person类
    object_setClass(self, kvoClass);
}

#pragma mark - 私有方法
// name -> Name -> setName:
- (SEL)setterSEL:(NSString *)keyPath {
    return NSSelectorFromString([NSString stringWithFormat:@"set%@:", [keyPath capitalizedString]]);
}

// setName: -> Name -> name
- (SEL)getterSEL:(SEL)cmd {
    NSString *tmp = NSStringFromSelector(cmd);
    tmp = [tmp substringFromIndex:3];
    tmp = [tmp substringToIndex:tmp.length - 1];
    return NSSelectorFromString([tmp lowercaseString]);
}

@end
```

使用示例：

```Objective C
//引入类别
#import "NSObject+KVO.h"

//改为调用类别里的监听方法
- (void)viewDidLoad {
    [super viewDidLoad];
    [self.person RW_addObserver:self forKeyPath:@"name" options:NSKeyValueObservingOptionNew | NSKeyValueObservingOptionOld context:nil];
}

- (void)observeValueForKeyPath:(NSString *)keyPath ofObject:(id)object change:(NSDictionary *)change context:(void *)context {
    NSLog(@"%@", change);
}

```

输出结果:
```Text
2018-05-19 21:20:52.841028+0800 KVO[52407:1799826] {
    kind = 1;
    new = Jerry;
    old = Tom;
}
```
通过上面几步，我们就可以手动实现了KVO了。
