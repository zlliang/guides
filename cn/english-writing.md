# 英文写作

一份关于如何清晰、简洁、地道地用英语写作的综合指南。本指南涵盖技术文章、个人反思和博客文章，从 Paul Graham、Bob Nystrom、Simon Willison、Mitchell Hashimoto 和 Alex Kladov 等有影响力的作家那里汲取灵感。

## 快速参考

### 核心原则

- **清晰第一**：说你想说的。避免歧义。
- **无情删减**：每个词都必须有存在的价值。
- **展示，而非告知**：用例子，而非抽象概念。
- **为人而写**：对话式胜过正式。
- **反复修改**：写作就是重写。

### 常见修正

```
❌ utilize → ✓ use
❌ in order to → ✓ to
❌ it is important to note that → ✓ note that / 删除
❌ at this point in time → ✓ now
❌ due to the fact that → ✓ because
```

### 结构模板

```
1. 开头（吸引注意）
2. 背景（设定场景）
3. 主要观点（陈述论点）
4. 证据/例子（支持观点）
5. 反驳（承认替代方案）
6. 结论（总结）
```

## 简介

### 本指南涵盖的内容

本指南教你写作：

1. **技术文章**：清晰地解释复杂想法（编程、系统、工具）
2. **个人反思**：分享经验和教训
3. **博客文章**：用想法、故事和见解吸引读者
4. **专业沟通**：邮件、文档、提案

### 什么是好的写作？

好的写作是：
- **清晰的**：读者第一遍就能理解
- **简洁的**：没有废话
- **正确的**：语法和事实准确
- **引人入胜的**：读者想继续阅读
- **真实的**：听起来像真人写的

### 本指南适合谁

- 写技术博客的**开发者**
- 分享发现的**研究人员**
- 传达愿景的**创业者**
- 任何想**写得更好**的人

## 基础

### 清晰

清晰意味着你的读者完全理解你的意思。

**写简单的句子**：
```
❌ The implementation of the algorithm, which was designed to optimize 
   performance, resulted in significant improvements.
   
✓ We implemented an algorithm that made the system 3x faster.
```

**使用具体的词**：
```
❌ We need to enhance our velocity.
✓ We need to ship faster.
```

**定义技术术语**：
```
❌ Use memoization to avoid recomputing.

✓ Use memoization—caching results of function calls—to avoid recomputing.
```

**一句一个想法**：
```
❌ The function validates input, which should be a string, and returns 
   true if valid, otherwise false, and logs errors.

✓ The function validates input strings. It returns true if valid, 
   false otherwise. It also logs any errors.
```

### 简洁

每个词都必须有目的。

**删除填充词**：
```
❌ It should be noted that...
❌ It is important to mention that...
❌ Basically...
❌ Actually...

✓ [删除这些短语]
```

**偏好主动语态**：
```
❌ The bug was fixed by the team.
✓ The team fixed the bug.

❌ The decision was made to refactor.
✓ We decided to refactor.
```

**消除冗余**：
```
❌ completely finished → finished
❌ final result → result
❌ past history → history
❌ advance planning → planning
❌ free gift → gift
```

**缩短短语**：
```
❌ at the present time → now
❌ in the event that → if
❌ for the purpose of → to
❌ with regard to → about
❌ a number of → several / many
```

### 正确性

**语法基础**：

*主谓一致*：
```
✓ The system works well.
❌ The system work well.

✓ The developers are ready.
❌ The developers is ready.
```

*代词清晰*：
```
❌ When you update the cache, it might fail.
   （什么可能失败？缓存还是更新？）

✓ When you update the cache, the update might fail.
```

*平行结构*：
```
❌ I like reading, to write, and code.
✓ I like reading, writing, and coding.
✓ I like to read, write, and code.
```

**常见错误**：

*Its vs It's*：
```
✓ The system has its own cache. (所有格)
✓ It's a simple fix. (it is)
```

*Their / They're / There*：
```
✓ Their code is clean. (所有格)
✓ They're shipping today. (they are)
✓ Put it there. (位置)
```

*Affect vs Effect*：
```
✓ The bug affects performance. (动词)
✓ The effect is noticeable. (名词)
```

## 风格和语气

### 对话式写作

像说话一样写（但经过编辑）。

**使用缩写**：
```
正式: I do not think it is a good idea.
更好: I don't think it's a good idea.
```

**对读者说话**：
```
疏远: One might consider using a hash table.
更好: You might consider using a hash table.
最好: Consider using a hash table.
```

**提问**：
```
Why does this matter? Because performance is critical.
How do we solve this? Let's look at three approaches.
```

### 找到你的声音

你的声音让你的写作独一无二。

**做你自己**：
- 不要试图听起来比你更聪明
- 使用你真正会说的词
- 分享你的真实观点
- 承认你不知道的

**展示个性**：
```
平淡: The approach has some advantages.
更好: This approach is clever, if a bit hacky.

平淡: The code needs improvement.
更好: This code makes me sad.
```

**使用幽默（适当时）**：
```
"I spent three hours debugging this. The bug was a missing comma. 
I'm now a professional comma hunter."
```

### 语气

让你的语气与你的受众和目的相匹配。

**技术博客**（Paul Graham 风格）：
- 对话式但深思熟虑
- 挑战传统智慧
- 使用日常生活中的类比

**教程**（Bob Nystrom 风格）：
- 鼓励和支持
- 带例子的循序渐进
- 预见困惑

**个人反思**（Mitchell Hashimoto 风格）：
- 诚实和脆弱
- 分享成功和失败
- 将个人与普遍联系起来

## 技术写作

### 解释复杂想法

**从简单开始**：
```
不要: "Implement a self-balancing binary search tree."

要: "Let's build a phonebook. Each person has a name and number. 
     We want to find people quickly..."
```

**使用类比**：
```
"A hash table is like a filing cabinet. Each drawer (bucket) has a 
label (hash), and you can go straight to the right drawer instead 
of searching every drawer."
```

**逐步构建**（Bob Nystrom 的方法）：
1. 展示简单情况
2. 解释为什么失败
3. 逐步增加复杂性
4. 在每一步展示代码

**例子：解释递归**：
```
差: "A function that calls itself with modified parameters."

好:
"Imagine you're in a maze. At each intersection, you have choices.
 
 You could try to remember every turn. Or you could:
 1. Try the left path
 2. If you hit a dead end, come back and try right
 3. Repeat at each intersection
 
 This is recursion: solving a problem by solving smaller versions 
 of the same problem."
```

### 代码示例

**保持简单**：
```python
# ❌ 一次太多
class ComplexCacheSystem:
    def __init__(self, size, eviction_policy, serializer):
        self.cache = LRUCache(size)
        self.policy = eviction_policy
        ...

# ✓ 从最小开始
cache = {}
cache['key'] = 'value'
result = cache.get('key')
```

**展示前后对比**：
```python
# Before: Slow
for item in items:
    if is_valid(item):
        process(item)

# After: Fast (with explanation why)
valid_items = [item for item in items if is_valid(item)]
for item in valid_items:
    process(item)
```

**解释非显而易见的部分**：
```python
# Check if power of 2 using bit trick
# (powers of 2 have exactly one bit set)
if n & (n - 1) == 0:
    return True
```

### 技术文章的结构

**1. 开头**（Simon Willison 风格）：
```
"I just spent a week debugging a performance issue. 
 The fix was a one-line change. Here's what I learned."
```

**2. 背景**：
- 你在解决什么问题？
- 为什么重要？
- 你尝试了什么？

**3. 解决方案**：
- 走过你的方法
- 展示代码/例子
- 解释权衡

**4. 教训**：
- 你会做什么不同的事？
- 这在什么时候适用？
- 什么仍未解决？

## 个人写作

### 文章和反思

**从具体时刻开始**：
```
不要: "Building a startup is hard."

要: "At 2 AM, staring at our rapidly depleting bank account, 
     I realized we had three weeks of runway left."
```

**诚实**：
```
不要: "We successfully pivoted our strategy."

要: "Our product failed. Nobody wanted it. We were wrong about 
     everything except one tiny feature users loved."
```

**将个人与普遍联系**：
```
"My experience isn't unique. Every founder I know has faced this 
exact moment: when reality doesn't match the pitch deck."
```

### 讲故事

**展示，而非告知**：
```
告知: "I was nervous about the presentation."

展示: "My hands shook as I opened the laptop. I'd rehearsed this 
       a hundred times, but my mind went blank as fifty faces 
       turned toward me."
```

**使用对话**：
```
"You're overthinking this," my cofounder said.
"Maybe. But what if—"
"No. Ship it."
```

**创造张力**：
```
好故事有：
- 问题（冲突）
- 不断升级的赌注（复杂化）
- 转折点（决定/发现）
- 解决（结果和反思）
```

### 寻找想法

**写你正在学的**：
- 记录你的挣扎
- 向过去的自己解释
- 分享你的突破

**写你不同意的**：
- 挑战传统智慧
- 解释为什么每个人都错了
- 提出替代方案

**写你希望存在的**：
- 你需要的教程
- 讨论中缺少的视角
- 没人建立的联系

## 学习伟大的作家

### Paul Graham

**特点**：
- 对话式但智力上的
- 挑战假设
- 用简单的词表达复杂的想法
- 短段落，每段一个想法

**技巧：逆向思维**：
```
"Most people think X. They're wrong because Y. 
 Here's what actually matters: Z."
```

**示例结构**：
1. 挑衅性主张
2. 反直觉的洞察
3. 历史/实践证据
4. 更广泛的影响

**学习资源**：
- [How to Do Great Work](http://www.paulgraham.com/greatwork.html)
- [Write Like You Talk](http://www.paulgraham.com/talk.html)

### Bob Nystrom

**特点**：
- 极其清晰的解释
- 逐步构建复杂性
- 友好、鼓励的语气
- 出色的图表使用

**技巧：分层教学**：
```
1. 展示简单版本
2. 指出哪里坏了
3. 添加一个修复
4. 重复直到完成
```

**示例方法**（来自 Crafting Interpreters）：
- "这是一个计算器"
- "现在让我们添加变量"
- "现在让我们添加函数"
- 每一步都建立在上一步的基础上

**学习资源**：
- [Crafting Interpreters](https://craftinginterpreters.com/)
- [Game Programming Patterns](https://gameprogrammingpatterns.com/)

### Simon Willison

**特点**：
- 实用和及时
- 大量使用例子
- 链接到来源
- "这是我构建的"方法

**技巧：展示你的工作**：
```
1. 这是我需要做的
2. 这是我尝试的
3. 这是我学到的
4. 这是代码/工具
```

**示例模式**：
- 识别问题
- 构建解决方案（通常是小工具）
- 写关于过程的文章
- 分享代码和演示

**学习资源**：
- 关于 AI、工具、datasette 的博客文章
- TIL（今天我学到）文章
- 开源文档

### Alex Kladov (matklad)

**特点**：
- 深刻的技术见解
- 强烈的观点，清晰陈述
- 专注于基础
- 擅长解释"为什么"

**技巧：原则性论证**：
```
1. 陈述原则
2. 展示具体例子
3. 解释权衡
4. 推荐方法
```

**示例主题**：
- 为什么某些模式重要
- 系统应该如何设计
- 什么使代码可维护

**学习资源**：
- 关于 Rust、IDE、架构的博客文章
- 专注于"无聊的解决方案"
- 强调简单性

### Mitchell Hashimoto

**特点**：
- 脆弱和诚实
- 平衡技术和个人
- 长篇深度探讨
- 对旅程的反思

**技巧：综合学习**：
```
1. 分享挑战（个人或技术）
2. 记录旅程
3. 提取教训
4. 连接到更广泛的主题
```

**示例方法**：
- 构建困难的东西
- 面对挫折
- 学习和适应
- 分享代码和情感

**学习资源**：
- 关于构建 HashiCorp 的文章
- 个人成长反思
- 技术深度探讨

## 实用技巧

### 开头钩子

**从行动开始**：
```
"I deleted the production database. By accident."
```

**从问题开始**：
```
"Why do we keep building the same thing over and over?"
```

**从惊人的事实开始**：
```
"90% of performance bugs come from one type of mistake."
```

**从冲突开始**：
```
"Everyone says to use microservices. Everyone is wrong."
```

### 过渡

**段落之间**：
```
But here's the catch...
This leads to a problem...
Let's see how this works...
Here's what I mean...
```

**章节之间**：
```
So far we've covered X. Now let's look at Y.
That's the theory. Here's the practice.
That solves A, but what about B?
```

### 结论

**总结要点**：
```
To recap:
- Point 1
- Point 2  
- Point 3
```

**行动号召**：
```
Try this yourself. Build something. Break something. Learn.
```

**打开一扇门**：
```
This solves X, but leaves Y open. That's what we'll explore next.
```

**个人反思**：
```
Writing this taught me that...
I'm still learning, but here's what I know now...
```

## 常见错误

### 过度复杂化

**不要用大词听起来聪明**：
```
❌ Utilize → ✓ Use
❌ Commence → ✓ Start
❌ Terminate → ✓ End
❌ Facilitate → ✓ Help
```

**不要写长句子**：
```
❌ The system, which was designed to handle large-scale data 
   processing tasks, utilizes a distributed architecture that 
   allows it to scale horizontally across multiple nodes.

✓ The system processes large amounts of data. It uses a 
   distributed architecture. This lets it scale across nodes.
```

### 含糊不清

**提供具体内容**：
```
❌ The performance improved significantly.
✓ Response time dropped from 2 seconds to 200ms.

❌ It's much faster now.
✓ It's 10x faster.
```

### 过度解释

**相信你的读者**：
```
❌ First, we need to import the library, which is a collection 
   of pre-written code that provides functionality...

✓ First, import the library:
   import requests
```

### 躲在被动语态后面

**承担责任**：
```
❌ Mistakes were made.
✓ We made mistakes.

❌ The decision was made to proceed.
✓ We decided to proceed.
```

## 写作过程

### 起草

**初稿规则**：
1. 写作时不要编辑
2. 如果需要可以写得糟糕
3. 跳过你卡住的部分
4. 只管把词写下来

**自由写作**：
```
设置一个 20 分钟的计时器。
不停地写。
不要修正错别字。
不要删除任何东西。
```

### 修改

**多次修改**：

**第一遍：结构**
- 流程有意义吗？
- 章节顺序对吗？
- 有缺失吗？

**第二遍：清晰**
- 我能简化句子吗？
- 例子清楚吗？
- 读者会理解吗？

**第三遍：简洁**
- 我能删掉这个词/句子/段落吗？
- 每个词都必要吗？
- 我在重复吗？

**第四遍：打磨**
- 修正语法
- 检查事实
- 验证代码示例

### 获取反馈

**问审稿人的问题**：
- 什么让你困惑？
- 我能删掉什么？
- 你想要更多什么？
- 主要观点是什么？

**对反馈采取行动**：
```
如果一个人困惑，重写。
如果多人错过重点，重新构建。
如果每个人都说太长，删掉 30%。
```

## 建立习惯

### 定期写作

**每日练习**：
- 晨间页（3 页，意识流）
- 每天写 30 分钟
- 每周发布（博客、新闻通讯）

**小胜利**：
- TIL（今天我学到）文章
- 推特线程
- 解释代码的评论

### 积极阅读

**分析好的写作**：
1. 他们如何吸引你？
2. 他们如何构建？
3. 什么使它清晰？
4. 什么可以更好？

**复制学习**：
- 重新键入你欣赏的段落
- 注意句子结构
- 感受节奏

### 实验

**尝试不同格式**：
- 操作指南
- 个人文章
- 技术深度探讨
- 快速提示
- 故事驱动的文章

**尝试不同风格**：
- 正式 vs 随意
- 长 vs 短
- 技术 vs 易懂
- 严肃 vs 幽默

## 练习

### 清晰练习

把这段话变得更清晰：
```
The implementation of the new feature, which was requested by 
several users, has been completed and deployed to production, 
resulting in improved user experience metrics.
```

**示例修改**：
```
We shipped the new feature users requested. User satisfaction 
improved by 20%.
```

### 简洁练习

把这段 100 字的段落削减到 50 字：
```
In today's fast-paced technological landscape, it is increasingly 
important to note that software development practices are constantly 
evolving. It should be mentioned that developers need to stay up to 
date with the latest trends and best practices. At this point in 
time, we can observe that continuous learning is essential for 
success in the field.
```

**示例修改（43 字）**：
```
Software development evolves constantly. Developers must stay 
current with trends and best practices. Continuous learning 
is essential for success.
```

### 语气练习

用三种不同的语气重写：
```
原始: "Error handling is important for robust applications."

随意: "If your app crashes every time something goes wrong, 
      users will hate it."

学术: "Robust error handling is essential for production-grade 
      software systems."

讲故事: "I learned about error handling the hard way: at 3 AM, 
        when the CEO called about the site being down."
```

## 资源

### 书籍

**On Writing Well** by William Zinsser
- 非虚构写作的经典指南
- 强调清晰和简单
- 永恒的原则

**The Elements of Style** by Strunk and White
- 简洁的用法规则
- 语法基础
- 必读

**On Writing** by Stephen King
- 写作技巧和过程
- 对工作的诚实
- 鼓舞人心和实用

**Bird by Bird** by Anne Lamott
- 写作作为一种实践
- 克服完美主义
- "糟糕的初稿"

**Several Short Sentences About Writing** by Verlyn Klinkenborg
- 独特的句子方法
- 打破坏习惯
- 新鲜视角

### 在线资源

**关于写作的文章**：
- [Write Like You Talk](http://www.paulgraham.com/talk.html) - Paul Graham
- [Writing, Briefly](http://www.paulgraham.com/writing44.html) - Paul Graham  
- [How to Write Technical Posts](https://reasonablypolymorphic.com/blog/writing-technical-posts/) - Sandy Maguire

**风格指南**：
- [Google Developer Documentation Style Guide](https://developers.google.com/style)
- [Microsoft Writing Style Guide](https://learn.microsoft.com/en-us/style-guide/welcome/)
- [Hemingway Editor](http://www.hemingwayapp.com/) - 可读性工具

**社区**：
- Write.as - 极简主义博客
- Dev.to - 开发者社区
- Hacker News - 技术讨论
- IndieHackers - 创始人故事

### 工具

**写作**：
- [Grammarly](https://www.grammarly.com/) - 语法和清晰度
- [Hemingway App](http://www.hemingwayapp.com/) - 可读性
- [LanguageTool](https://languagetool.org/) - 语法检查器

**发布**：
- [Ghost](https://ghost.org/) - 博客平台
- [Substack](https://substack.com/) - 新闻通讯平台
- [Medium](https://medium.com/) - 出版平台

**阅读**：
- RSS 阅读器跟踪作家
- Pocket 保存文章
- Readwise 高亮显示

## 最后的想法

### 写作是一门手艺

通过实践变得更好：
- 定期写作
- 积极阅读
- 无情修改
- 勇敢发布

### 你的声音很重要

不要试图听起来像别人：
- 写你知道的
- 写你关心的
- 写只有你能写的

### 从简单开始

你不需要：
- 完美的语法
- 大词
- 完全理解

你需要：
- 要说的东西
- 愿意修改
- 分享的勇气

### 继续前进

> "The scariest moment is always just before you start." 
> — Stephen King

写那第一稿。
发布那篇文章。
开始那个博客。

你的写作会进步。
你的声音会出现。
你的想法会重要。

现在去写吧。
