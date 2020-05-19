---
layout: post
title: "Objc_Runtime"
date: 2020-05-12 12:00:00
comments: true
catagories: language
tags: [iOS]
---

Objc是一门动态语言，所以它总是想办法把一些决定工作从编译连接推迟到运行时。也就是说只有编译器是不够的，还需要一个运行时系统(runtime system) 来执行编译后的代码。这就是 Objective-C Runtime.
<!--more-->
RunTime简称运行时。OC就是运行时机制，其中最主要的是消息机制。对于C语言，函数的调用在编译的时候会决定调用哪个函数。对于OC的函数，属于动态调用过程，在编译的时候并不能决定真正调用哪个函数，只有在真正运行的时候才会根据函数的名称找到对应的函数来调用

# id&Class
```C
!Objc
typedef struct objc_class *Class;
typedef struct objc_object *id;
struct objc_object {
    Class isa;
};
struct objc_class {
    Class isa;
}
/// 不透明结构体, selector
typedef struct objc_selector *SEL;
/// 函数指针, 用于表示对象方法的实现
typedef id (*IMP)(id, SEL, ...);
```
# Define

- 对对象进行操作的方法一般以object_开头
- 对类进行操作的方法一般以class_开头
- 对类或对象的方法进行操作的方法一般以method_开头
- 对成员变量进行操作的方法一般以ivar_开头
- 对属性进行操作的方法一般以property_开头
- 对协议进行操作的方法一般以protocol_开头

根据以上的函数的前缀 可以大致了解到层级关系。对于以objc_开头的方法，则是runtime最终的管家，可以获取内存中类的加载信息,类的列表，关联对象和关联属性等操作。

# important method
```C
objc_copyClassList

class_copyIvarList

class_copyPropertyList
```
ivarList可以获取到@property关键字定义的属性 ，而propertyList不可以获取到成员变量。使用ivarList是可以将所有的成员变量和属性都获取的。

```c
+ (BOOL)resolveClassMethod:(SEL)sel 
+ (BOOL)resolveInstanceMethod:(SEL)sel

class_addIvar

class_addMethod
```

# NSCoding default implement
```C
!Objc
- (void)encodeWithCoder:(NSCoder *)aCoder {
unsigned int count = 0;
Ivar *ivars = class_copyIvarList(self.class, &count);
for (int i = 0; i < count; i++) {
    const char *cname = ivar_getName(ivars[i]);
    NSString *name = [NSString stringWithUTF8String:cname];
    NSString *key = [name substringFromIndex:1];
    
    id value = [self valueForKey:key]; // KVC隐性数据转换
    [aCoder encodeObject:value forKey:key]; // 编码
    }
}
- (id)initWithCoder:(NSCoder *)aDecoder {
if (self = [super init]) {
    unsigned int count = 0;
    Ivar *ivars = class_copyIvarList(self.class, &count);
    for (int i = 0; i < count; i++) {
        const char *cname = ivar_getName(ivars[i]);
        NSString *name = [NSString stringWithUTF8String:cname];
        NSString *key = [name substringFromIndex:1];
        
        id value = [aDecoder decodeObjectForKey:key]; // 解码
        [self setValue:value forKey:key]; // KVC隐性数据转换
    }
}
return self;    
}
```