# CPP 闭包踩坑

最近因为做项目遇到一些场景可能有些场景会受到 CMU14-445 启发，于是开始做 CMU14-445 的 lab，然后踩了个坑。

```
// CountMinSketch 类下的一个内联函数，用来创建类变量 vector: hash_functions_
inline auto HashFunction(size_t seed) -> std::function<size_t(const KeyType &)> {
    return [seed, this](const KeyType &item) -> size_t {
      ...
      return bustub::HashUtil::CombineHashes(h1, h2) % **width_**;
    };
  }
```

如上是创建闭包的一个函数，width_ 是 countMinSketch 的类变量。
在这里实际上 `width_` 变量是 `this->width_`，而闭包捕获了 this，所以闭包捕获**保持了指针 this**。**（划重点）**


## 捕获指针带来的坑

由于 lab 需要写 move 构造函数和赋值函数加上配合闭包捕获了类指针，便有了以下一幕。

**捕获类指针**带来的问题直接在利用闭包函数进行操作时，遇到取余数操作下，  
`return bustub::HashUtil::CombineHashes(h1, h2) % width_;`炸出来了。

width_ 为 0 触发了 Float-Point-Exception。

那为什么会是 0 ？

因为通常移动（构造/赋值）函数通常会把被转移所有权的类的变量置为 0，而 width_ 在相关移动（赋值/构造）函数中时就被我置 0 了，而闭包捕获的 this 类指针指向的就是原来被转移所有权的类实例，即被置 0 的类实例。

## 反思
注意，闭包捕获变量（特别是指针），可能在所有权状态下会有更多变种不限于目前遇到的问题，谨以此记。