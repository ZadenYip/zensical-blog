# Pandas 关于 insert 导致 null 的踩坑

## 踩坑经过和解析

今天在做单元测试从爬虫清洗后得到的字典抽取几个数据准备加 fixture 进行单元测试的时候发现 uuid 值居然是 null。

排查发现是清洗数据生成 uuid 的时候 DataFrame 使用 insert 方法添加新列的时候会匹配索引，如果没匹配索引则会直接不理会默认为 null 值。

```
word_poses_tb.insert(
    loc=0,
    column="pose_id",
    value=uuid7_series(len(word_poses_tb))
)
```

如上代码，words_poses_tb.insert，如果 value 恰好是 Series 这种带索引的类型会进行索引匹配，就会遗漏 word_poses_tb 部分索引没匹配到series连续的索引。


## 踩坑解析

遗漏产生 null 值的原因是 word_poses_tb 清洗的时候删除了一些行，导致了索引不再连续就像这样：[0, 1, 9, ...]，而这里新的 uuid 的 series，索引是连续并且最大值是实际的行数。那么就会产生：

```
// words_poses_tb 索引
[0, 1, 9, ..., 699, *700, words_poses_tb 最大索引(10000)*] 
// uuid series 索引
[0, 1, 2, 2333, words_poses_tb 的目前实际行数 - 1(699)] 
 
```

如上体现，星号是遗漏匹配的，所以主要体现就是 insert 后，到达一定后面的行后 uuid 属性就为 null 了

注意：只要 words_poses_tb 索引没在 series 索引里就会 null 不是后面就一定会产生）


## 解决方案

如果不需要保留现在的行索引的话，推荐 insert 前直接 reset_index(drop=True) 重置索引让其从 0 开始计数。  

到这里，问题就已经解决了。

不过这里还有第二种方案，那就是在 insert 的时候 value 参数使用 list 类型而不是 series，从而放弃使用索引对齐转向使用物理位置对齐。意味着数据是按照 words_poses_tb 物理行顺序与 list 元素顺序一一对齐插入。

因此，出于防守性编程，我更加推荐把这两种方案结合起来

```
word_poses_tb = word_poses_tb.reset_index(drop=True) // 重置索引
word_poses_tb.insert(
    loc=0,
    column="pose_id",
    value=list(uuid7_series(len(word_poses_tb))) // 这里多了个构造出 list 去掉索引匹配
)
```
