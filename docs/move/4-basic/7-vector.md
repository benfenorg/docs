# 向量

向量（vector）是Move中内置的存储元素的集合，向量与其他编程语言中的数组类似。在本节中，我们将介绍vector类型及其操作。

## 定义向量

vector类型使用vector关键字，后跟尖括号中的元素类型来定义。元素的类型可以是任何有效的Move类型。

创建空向量的方法：

```
let empty: vector<bool> = vector[]; // 空的bool向量
```

创建指定内容的向量：

```
let v: vector<u8> = vector[10, 20, 30]; // 包含3个u8元素的向量
```

创建包含向量的向量：

```
let vv: vector<vector<u8>> = vector[
    vector[10, 20],
    vector[30, 40]
]; // 包含两个元素的向量，其中每个元素又是一个vector
```

`vector`类型已在Move中内置，不需要从模块导入。但是，操作向量的函数是在`std::vector`模块中定义的，需要导入该模块才能使用它们。

## 向量运算

标准库提供了操作向量的方法。以下代码演示了一些最常用的操作：

```
#[test]
fun test() {
    let v: vector<u8> = vector[10, 20, 30]; // v包含3个元素

    // 将元素添加到向量的末尾:
    v.push_back(40);
    print(&v); // [10, 20, 30, 40]

    // 删除并返回向量的最后一个元素:
    assert!(40 == v.pop_back());
    print(&v); // [10, 20, 30]

    // 返回向量长度:
    assert!(v.length() == 3);

    // 检查向量是否为空:
    assert!(!v.is_empty());

    // 删除指定索引的元素:
    v.remove(1);
    print(&v); // [10, 30]
}
```

在结构体一节中，我们介绍了没有`drop`能力的结构体默认不可丢弃。如果声明一个可容纳该结构体类型的`vector`，即使`vector`为空，也需要显式调用`destroy_empty()`函数才能编译通过：

```
public struct Coffee {}

#[test]
fun test() {
    let v = vector<Coffee>[];
    v.destroy_empty();
}
```
