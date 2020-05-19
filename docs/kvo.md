# KVO的底层实现学习
---

## 什么是KVO?

?> **NSKeyValueObserving** \
An infromal protocol that objects adopt to be notified of changes to the specified properties of other objects. 

以上为官方提供的KVO描述，翻译过来大致为：

> 一种非正式协议，用于其他对象指定的属性改变时通知该对象。

这里提到了知识点：**非正式协议**，当然对应的也存在 **正式协议**。
- Informal Protocol: NSObject的一个Category
- Formal Protocol: 平时经常用到的delegate

## KVO实现

- 添加监听

```Objective C
- (void)addObserver:(NSObject *)observer forKeyPath:(NSString *)keyPath options:(NSKeyValueObservingOptions)options context:(nullable void *)context;
```

- 实现监听方法


```Objective C
- (void)observeValueForKeyPath:(nullable NSString *)keyPath ofObject:(nullable id)object change:(nullable NSDictionary<NSKeyValueChangeKey, id> *)change context:(nullable void *)context;
```

- 移除监听


```Objective C
- (void)removeObserver:(NSObject *)observer forKeyPath:(NSString *)keyPath;
```

- 测试代码

```Objective C
- (void)viewDidLoad {
    [super viewDidLoad];
    self.person = [[Person alloc] init];
    [self.person setName:@"Tom"];
    [self.person addObserver:self forKeyPath:@"name" options:NSKeyValueObservingOptionNew | NSKeyValueObservingOptionOld context:nil];
    
}

- (void)touchesBegan:(NSSet<UITouch *> *)touches withEvent:(UIEvent *)event{
    [self.person setName:@"Jerry"];
}

- (void)observeValueForKeyPath:(NSString *)keyPath ofObject:(id)object change:(NSDictionary<NSKeyValueChangeKey,id> *)change context:(void *)context{
    NSLog(@"%@", change);
}

- (void)dealloc{
    [self.person removeObserver:self forKeyPath:@"name"];
}
```

- 测试结果
  
```Text
2018-05-19 14:42:57.564605+0800 KVO[49040:1658166] {
    kind = 1;
    new = Jerry;
    old = Tom;
}
```

## KVO底层实现

先探究下，当监听对象属性发生改变时，也就是调用**setter**方法时，内部时做了什么操作。
测试方法：

- 在发生改变的地方打个断点运行程序，在`lldb`窗口中添加观察点:

```
(lldb) watchpoint set variable _person->_name
```
- 接着运行，可以看到函数调用栈:

```
Foundation`_NSSetObjectValueAndNotify:
    0x7fff258e45bd <+114>: je     0x7fff258e45ff            ; <+180>
    0x7fff258e45bf <+116>: movq   0x62228612(%rip), %rsi    ; "willChangeValueForKey:"
    0x7fff258e45c6 <+123>: movq   0x5b094f93(%rip), %r14    ; (void *)0x00007fff50b37400: objc_msgSend
    0x7fff258e45cd <+130>: movq   %r13, %rdi
    0x7fff258e45d0 <+133>: movq   %r12, %rdx
    0x7fff258e45d3 <+136>: callq  *%r14
    0x7fff258e45d6 <+139>: movq   (%rbx), %rdi
    0x7fff258e45d9 <+142>: movq   %r15, %rsi
    0x7fff258e45dc <+145>: callq  0x7fff25ae68e4            ; symbol stub for: class_getMethodImplementation
    0x7fff258e45e1 <+150>: movq   %r13, %rdi
    0x7fff258e45e4 <+153>: movq   %r15, %rsi
    0x7fff258e45e7 <+156>: movq   -0x78(%rbp), %rdx
    0x7fff258e45eb <+160>: callq  *%rax
    0x7fff258e45ed <+162>: movq   0x622285fc(%rip), %rsi    ; "didChangeValueForKey:"
    0x7fff258e45f4 <+169>: movq   %r13, %rdi
    0x7fff258e45f7 <+172>: movq   %r12, %rdx
    0x7fff258e45fa <+175>: callq  *%r14
    0x7fff258e45fd <+178>: jmp    0x7fff258e4658            ; <+269>
    0x7fff258e45ff <+180>: movq   0x5b094a22(%rip), %rax    ; (void *)0x00007fff89c7bf10: _NSConcreteStackBlock
    0x7fff258e4606 <+187>: leaq   -0x70(%rbp), %r9
    0x7fff258e460a <+191>: movq   %rax, (%r9)
    0x7fff258e460d <+194>: movl   $0xc2000000, %eax         ; imm = 0xC2000000 
    0x7fff258e4612 <+199>: movq   %rax, 0x8(%r9)
    0x7fff258e4616 <+203>: leaq   0x15d6(%rip), %rax        ; ___NSSetObjectValueAndNotify_block_invoke
    0x7fff258e461d <+210>: movq   %rax, 0x10(%r9)
    0x7fff258e4621 <+214>: leaq   0x5b0994e8(%rip), %rax    ; __block_descriptor_64_e8_32o40o_e5_v8?0l
    0x7fff258e4628 <+221>: movq   %rax, 0x18(%r9)
    0x7fff258e462c <+225>: movq   %rbx, 0x30(%r9)
    0x7fff258e4630 <+229>: movq   %r15, 0x38(%r9)
    0x7fff258e4634 <+233>: movq   %r13, 0x20(%r9)
    0x7fff258e4638 <+237>: movq   -0x78(%rbp), %rax
    0x7fff258e463c <+241>: movq   %rax, 0x28(%r9)
    0x7fff258e4640 <+245>: movq   0x62229259(%rip), %rsi    ; "_changeValueForKey:key:key:usingBlock:"
    0x7fff258e4647 <+252>: movq   %r13, %rdi
```
可以看到，当调用监听对象属性的`setter`方法是，系统内部调用了`Foundation`框架的`_NSSetObjectValueAndNotify`的方法，整体的实现过程简化为：

```Objective C
- (void)setName:(NSString *)name {
    _NSSetObjectValueAndNotify();
}

- (void)_NSSetObjectValueAndNotify {
    [self willChangeValueForKey:@"name"];
    [super setName: name];
    [self didChangeValueForKey:@"name"];
}

- (void)didChangeValueForKey:(NSString *)key {
    [observe observerValueForKeyPath:key ofObject:self change: nil context:nil];
}
```

接下来，通过调用`runtime`的`object_getClass`方法来查看监听对象的发生的变化

```Objective C
NSLog(@"Before: %s", object_getClassName(self.person));
[self.person addObserver:self forKeyPath:@"name" options:NSKeyValueObservingOptionNew | NSKeyValueObservingOptionOld context:nil];
NSLog(@"After: %s", object_getClassName(self.person));
```
打印结果：

```Text
2018-05-19 14:50:16.237749+0800 KVO[49246:1665274] Before: Person
2018-05-19 14:50:16.238177+0800 KVO[49246:1665274] After: NSKVONotifying_Person
```
很明显可以看到添加监听前后，监听对象发生了改变，系统自动创建了**Person**类的一个子类**NSKVONotifying_+Person**。

接着查看下在NSKVONotifying_+Person中都重写了哪些方法:

```Objective C
[self.person addObserver:self forKeyPath:@"name" options:NSKeyValueObservingOptionNew | NSKeyValueObservingOptionOld context:nil];
unsigned int count;
Class class = object_getClass(self.person);
Method *methods = class_copyMethodList(object_getClass(self.person), &count);
for (NSInteger index = 0; index < count; index++) {
    Method method = methods[index];
    NSString *methodStr = NSStringFromSelector(method_getName(method));
    NSLog(@"%@:%@\n", class, methodStr);
}
```

打印结果：

```Text
2018-05-19 16:20:10.981351+0800 KVO[50118:1706181] NSKVONotifying_Person:setName:
2018-05-19 16:20:10.981539+0800 KVO[50118:1706181] NSKVONotifying_Person:class
2018-05-19 16:20:10.981680+0800 KVO[50118:1706181] NSKVONotifying_Person:dealloc
2018-05-19 16:20:10.981851+0800 KVO[50118:1706181] NSKVONotifying_Person:_isKVOA
```

其中的方法：

- setName 修改属性
- class   将NSKVONotifying_Person自身伪装成Person类
- dealloc 释放资源
- _isKVOA 使用了KVO的标志

下面记录下如何通过手动实现KVO。




