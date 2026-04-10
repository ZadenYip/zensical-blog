# CPP 闭包踩坑

最近在做项目，做的过程中发现有部分场景可能会在 CMU 14-445 的 lab 遇到，于是打算顺便“并发”做 CMU14-445 的 lab。在做 proj0 的时候，踩了个闭包带来的坑。

```
// CountMinSketch 类下的一个内联函数，用来创建类变量 vector: hash_functions_
inline auto HashFunction(size_t seed) -> std::function<size_t(const KeyType &)> {
    return [seed, this](const KeyType &item) -> size_t {
      ...
      return bustub::HashUtil::CombineHashes(h1, h2) % **width_**;
    };
  }
```

如上是创建闭包的一个函数，width_ 原本是 countMinSketch 的类变量，而这里创建闭包的 **width_ 变量是 this->width_（划重点）**，闭包捕获的是 this 指针，不是捕获 width_！！！


## 捕获指针带来的坑

由于 lab 需要用到移动（构造/赋值）函数加上 lab 原本的**闭包捕获了类指针**，便有了以下一幕。

**捕获类指针**带来的问题直接在用闭包函数进行取余数操作下，  
`return bustub::HashUtil::CombineHashes(h1, h2) % width_;`炸出来了。

`width_` 为 0 触发了 Float-Point-Exception。

那为什么会是 0 ？

因为这里的 `width_` 是被捕获的 `this` 指针指向的类实例的 `width_`，而 width_ 在相关移动（赋值/构造）函数中时就被我置 0 了（通常这两函数实现是把被转移所有权的类实例中的变量置为 0）。

## 反思
注意，闭包捕获变量（特别是指针），可能在所有权状态下会有更多变种不限于目前遇到的问题，谨以此记。