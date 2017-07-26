---
title: FreeCodeCamp 高级算法题 - 验证美国电话号
date: 2017-07-18 21:21:47
tags: [FreeCodeCamp,高级,算法]
categories: FCC
---

# 验证美国电话号 (Validate US Telephone Numbers)

## 题目链接
- [中文链接](https://freecodecamp.cn/challenges/validate-us-telephone-numbers)
- [英文链接](https://freecodecamp.com/challenges/validate-us-telephone-numbers)

## 问题解释
- 这个 `function` 接收一个字符串参数 `str`，表示一个电话号码。返回值为 `Boolean`，表示这个 `str` 是否符合美国电话号码格式
- 如果 `str` 是 `555-555-5555`，那么应该返回 `true`，如果是 `555-55-555`，那么应该返回 `false`

<!--more-->

# 解题思路
高级算法第一题，但这道题和算法无关，只是正则的事儿。那我们就先来分析一下题目要求，看看正则该如何写。请在想思路之前再好好读一遍题目说明，以及底下的测试用例

另外，如果你不熟悉正则表达式，建议打开 MDN [RegExp](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/RegExp) 页面，对照着看

先推荐一个网站，[regex101](https://regex101.com/)。我一直在用这个调试正则

- 通过观察例子，我们得出一个结论，美国电话号码是十位的，事实上也确实如此。当然，十位都得是数字=。=
    - 这一步的正则可以写成 `^\d{10}$`。可以思考一下，为什么要加上 `^` 和 `$`
- 额外地，我们可以在前面加上国家码 `1`，但在这道题目中，不加也是可以的，但不能加上其他数字。也就是说，`1` 要么出现在开头，总长度为 `11`，否则总长度为 `10`
    - 这一步的正则更新为 `^1?\d{10}$`。其中，`1?` 代表的是 `1` 的出现次数为 `0` 或 `1` 次
- 仔细观察一下例子，只有一种情况，`5555555555` 十个 `5` 是连着写的，其他情况，都是按照 `三个数 三个数 四个数` 分为三组。那么我们把这三组在正则中也分一下，只要不影响十个 `5` 的那种情况判断
    - 这一步正则更新为 `^1?\d{3}\d{3}\d{4}$`。显然，十个 `5` 那种情况，也符合这个正则表达式
- 考虑到如果开头存在 `1`，那么这个 `1` 之后可以有一个空格，也可以没有
    - 因此，正则更新为 `^1?\s?\d{3}\d{3}\d{4}$`
- 分组之后，就要考虑组与组之间连接符的可能了。例子中总共有三种情况，分别是空格，`-` 和**什么都没有**。但要注意，不存在既有 `-` 又有空格的情况。所以，这个只需要写成 `(\s|-)?`
- 连接符位置可以存在于第一组与第二组之间，也可以存在于第二组和第三组之间
    - 这一步正则可以更新为 `^1?\s?\d{3}(\s|-)?\d{3}(\s|-)?\d{4}$`
- 最后一点，注意到第一组数字可以加上括号，那只有两种可能，`(555)` 和 `555`。需要注意，正则表达式中，小括号是有含义的 (matched group)，因此要转义。就像这样：`\(\d{3}\)`
    - 最终，正则为 `^1?\s?(\(\d{3}\)|\d{3})(\s|-)?\d{3}(\s|-)?\d{4}$`

# 基本解法 - 正则
## 代码
```js
function telephoneCheck(str) {
    return /^1?\s?(\(\d{3}\)|\d{3})(\s|-)?\d{3}(\s|-)?\d{4}$/.test(str);
}
```

多说一句，这个正则是有 bug 的。因为它不能 cover 所有情况。比如，`1 123456-7890` 也是可以通过的。事实上，真正在写手机号的时候，没有人会这么写。题目中没有要求这种 case，我们可以进一步去处理，只是那样写出来的正则会更复杂。大致思路会是，当第一组和第二组之间存在 `)` 或 `-` 或 `) ` 的时候，第二组和第三组之间才允许出现 `-`。有兴趣的朋友可以尝试一下

再多说一点，关于 `\s`。事实上，`\s` 与空格不是严格等价的，与
```
[\t\n\v\f\r\u00a0\u2000\u2001\u2002\u2003\u2004\u2005\u2006\u2007\u2008\u2009\u200a\u200b\u2028\u2029\u3000]
```
才是严格等价的。它不仅包含空格，还包含了换行符之类的其他东西。严格上来说，这里我们是不应该用 `\s` 的。只是个人觉得，写成 `^1? ?` 看起来会很奇怪

常见的一个坑还有 `\w`。因为 `\w` 等价于 `[a-zA-Z0-9_]`，包含下划线。类似地，`\W` 也排除了下划线，使用的时候需要注意