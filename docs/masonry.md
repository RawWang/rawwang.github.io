# Masonry简单使用
---

## 安装
利用CocoaPods安装。
> pod Masonry

## 添加Masonry步骤

* 自定义`UIView`
* add UIView
* 添加约束

## Masonry属性

```Objective-C
@property (nonatomic, strong, readonly) MASConstraint *left;
@property (nonatomic, strong, readonly) MASConstraint *right;
@property (nonatomic, strong, readonly) MASConstraint *top;
@property (nonatomic, strong, readonly) MASConstraint *bottom;
@property (nonatomic, strong, readonly) MASConstraint *leading;       //等价于left
@property (nonatomic, strong, readonly) MASConstraint *trailing;      //等价于right
@property (nonatomic, strong, readonly) MASConstraint *width;
@property (nonatomic, strong, readonly) MASConstraint *height;
@property (nonatomic, strong, readonly) MASConstraint *centerX;
@property (nonatomic, strong, readonly) MASConstraint *centerY;
@property (nonatomic, strong, readonly) MASConstraint *baseline;      //底部到顶点的距离
```
## Masonry约束

```Objective-C
//添加约束
- (NSArray *)mas_makeConstraints:(void(^)(MASConstraintMaker *make))block;
//更新约束，只更新已存在的约束，不会增加新的约束属性
- (NSArray *)mas_updateConstraints:(void(^)(MASConstraintMaker *make))block;
//清除已有的约束，保留新增的约束
- (NSArray *)mas_remakeConstraints:(void(^)(MASConstraintMaker *make))block;
```
