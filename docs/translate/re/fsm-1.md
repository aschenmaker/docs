# 从 if-else 地狱到状态机流转 

## 1. intro

在一次面试中，面试官提到了一个简单的题目，在不利用Go自带包或者第三包的情况下，对一个字符串进行切分。

比如对于字符串` i  come  from somewhere you do not   know"`进行切分的话，那么就是`["i","come","from","somewhere","you","do","not","know"]`。

这是一个简单的需求，我们只用记录从读取到第一个字符开始，到空格就结束了。代码写起来也很简单。

## 2. 逐渐复杂

### 2.1 单词和空格

也就是intro中的需求，思路如下

1. 遍历字符串，如果是字母则记录下标为`index1`
1. 直到遇到了空格，则将此时下标`index2`之间的单词加入到列表中，重置`index1`。
2. 到了最后一个单词，如果仍然`index1`不为重置值则将最后一个单词，也加入

实际代码如下：

```go
// stringSplit
func stringSplit(str string) []string {
	index1 := -1
	wordList := []string{}
	for i, v := range str {
		if v != ' ' {
			if index1 == -1 {
				index1 = i
			}
			continue
		} else {
			if index1 != -1 {
				wordList = append(wordList, str[index1:i])
				index1 = -1
			}
		}
	}

	if index1 != -1 {
		wordList = append(wordList, str[index1:])
	}

	return wordList
}

// input: "i  come  from somewhere you do not   know"
// output: [i come from somewhere you do not know]
```

这个代码很好的胜任了，面试官的需求。

### 2.2 单词和句号符号

这时候面试官说，如果我输入的 `i  come  from somewhere you do not   know.And I will tell you`。显然，`know.And`应该是两个单词，你的输出是`konw.And`，不符合我的需求了。

于是，进行修改，只是增加条件罢了，只要把'`‘.‘`也作为分割符即可。新一版的代码很快改好出来。

```go
func stringSplit(str string) []string {
	index1 := -1
	wordList := []string{}
	for i, v := range str {
    // 增加了一个判断条件
		if v != ' ' && v != '.' {
			if index1 == -1 {
				index1 = i
			}
			continue
		}
    // 增加了一个判断条件
		if v == ' ' || v == '.' {
			if index1 != -1 {
				wordList = append(wordList, str[index1:i])
				index1 = -1
			}
		}
	}

	if index1 != -1 {
		wordList = append(wordList, str[index1:])
	}

	return wordList
}
```

但是别高兴还没完呢，也有人会使用`,`来对文本进行分割，当然这时候在添加条件的区域新增一个条件即可。

```go

func stringSplit(str string) []string {
	index1 := -1
	wordList := []string{}
	for i, v := range str {
    // 判断分割符
		if !isSeparator(v) {
			if index1 == -1 {
				index1 = i
			}
			continue
		} else {
			if index1 != -1 {
				wordList = append(wordList, str[index1:i])
				index1 = -1
			}
		}
	}

	if index1 != -1 {
		wordList = append(wordList, str[index1:])
	}

	return wordList
}
// 判断是否是分割符
func isSeparator(character rune) bool {
	seps := []rune{' ', '.', ','}
	for _, s := range seps {
		if s == character {
			return true
		}
	}
	return false
}

```

这时候代码看起来又可以了。

### 2.3单词和小数

面试官说到，那么`I come from somewhere you do not know.And It's 10.1 km away.`显然这时候，`10.1`作为一个小数，当中的 `.`不可以再被作为，分割符号了。对于两个数字之间`.`，我们需要把他作为小数符号来看待。

```go

func stringSplit(str string) []string {
	....
	for i, v := range str {
    // 如果这个 . 前后是数字的时候，我们进行跳过
		if v == '.' && index1 != -1 && isDigit(str[index1]) && i+1 < len(str) && isDigit(str[i+1]) {
			continue
		} else if !isSeparator(byte(v)) {
			if index1 == -1 {
				index1 = i
			}
			continue
		} else {
			if index1 != -1 {
				wordList = append(wordList, str[index1:i])
				index1 = -1
			}
		}
	}
	......
	return wordList
}
......
// 判断是否是数字，通过ASCII进行判断
func isDigit(character byte) bool {
	if character >= 48 && character <= 57 {
		return true
	}
	return false
}
```

这时候，代码逐渐趋于复杂了。

### 2.4单词、小数、时间

面试官接着说到，如果我现在的字符串是，`I come from somewhere you do not know.And It's 10.1 km away. I will tell you at 10:00 am` 那么 `10:00 am`我们可以作为一个时间的。

emmmm，其实只要继续在分割函数中添加，逻辑判断代码就可以了，但是一层又一层的 `if-else`我们仿	佛陷入了一个循环♻️，只要有新的需求，那么我就一定得增加，`if-else`的代码。甚至也需要吧以前已经完成了的代码，进行修改。 **这显然是不合理** 。

如何去优化了？

## 3.有限状态机

使用状态机，让分词和单词所出的状态，进行流转。

**有限状态机有两个必要的特点，一是离散的，二是有限的。基于这两点，现实世界上绝大多数事物因为复杂的状态而无法用有限状态机表示。**

而描述事物的有限状态机模型的元素由以下组成：

- 状态(State)：事物的状态，包括初始状态和所有事件触发后的状态
- 事件(Event)：触发状态变化或者保持原状态的事件
- 行为或转换(Action/Transition)：执行状态转换的过程
- 检测器(Guard)：检测某种状态要转换成另一种状态的条件是否满足



有一个字符串 "I'm a 10 years old boy, 'And i sleep at 10 pm,and' i eat 10.10 kg food everyday" 你需要做的是对这个字符串进行切分，判断当前数组中的单词个数。

得到目标字符串数组 `[I'm, a , 10, years, old, boy, And, i, sleep, at, 10 pm, and, i ,eat, 10.10, kg, food, everyday]`，

即 切分出，`小数`,`时间`,以及`单词`的总共个数

<script src="https://gist.github.com/aschenmaker/54009840e07f2b76f6022ed3f71b681b.js"></script>