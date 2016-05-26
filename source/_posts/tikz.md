title: TikZ
tags:
  - latex
  - plot
categories:
  - writing
date: 2016-05-25 19:57:00
---
# TikZ是什么
PGF/TikZ是用于绘制几何和代数的向量图形的语言，PGF是底层语言，TikZ则是基于PGF的宏高级语言。PGF/TikZ可以在$\LaTeX$、plain $\TeX$、和\ConTEXt中使用. 这里主要关注TikZ在$\LaTeX$中的用法。

# TikZ基础
## 设置使用环境
```tex
\documentclass{article} % say
\usepackage{tikz}
\begin{document}
We are working on
\begin{tikzpicture}
  \draw (-1.5,0) -- (1.5,0);
  \draw (0,-1.5) -- (0,1.5);
\end{tikzpicture}.
\end{document}
```

## 构造直线
```tex
\draw (-1.5,0) -- (1.5,0) -- (0,-1.5) -- (0,1.5);
```
不要忘记表达式尾的分号。

## 构造长方形
```tex
\draw (-0.5,-0.5) rectangle (-1,-1);
```

## 增加一点风格
```tex
help lines/.style={color=blue!50,very thin}
```

如果`help lines`定义在全局，则在文档开头使用`\tikzset`

```tex
\tikzset{help lines/.style=very thin}
```

一个风格的定义可以引用另一个风格。如`Karl's grid`引用了上面的`help lines`

```tex
\tikzset{Karl’s grid/.style={help lines,color=blue!50}}
...
  \draw[Karl’s grid] (0,0) grid (5,5);
```

这也同时提示我们风格的用法类似**\draw**[*style*]。如果定义局部（在环境内部）的风格，则像下面这样

```tex
\begin{tikzpicture}
[Karl’s grid/.style ={help lines,color=#1!50},
Karl’s grid/.default=blue]
  \draw[Karl’s grid] (0,0) grid (1.5,2);
  \draw[Karl’s grid=red] (2,0) grid (3.5,2);
\end{tikzpicture}
```

定义在`tikzpicture`的可选参数中。上面也提示我们可以为风格设置默认值，想要替换默认值只需要给风格赋一个新值。

## 绘制选项
线条的粗细有以下一些值

> ultra thin, very thin, thin, semithick, thick, very thick, ultra thick

`thin`是正常粗细，它和$\TeX$的`\hrule`的粗细相同。

线条的样式有以下一些值

> loosely dashed, dashed, densely dashed
> loosely dotted, dotted, densely dotted

含义一看便知。

# TikZ中的node
TikZ中的`node`可以插入文字标签，更重要的是可以方便地构造几何图形。转换图、流程图等等都可以用`node`实现。

## \node和at
```tex
\begin{tikzpicture}
  \node at ( 0,2) [circle,draw] {};
  \node at ( 0,1) [circle,draw] {};
  \node at ( 0,0) [circle,draw] {};
  \node at ( 1,1) [rectangle,draw] {};
  \node at (-1,1) [rectangle,draw] {};
\end{tikzpicture}
```
这种写法使用了`\node`命令和`at`标记，简化了原始写法。行尾的`{}`用于插入文字，为空表示不插入任何文字。可选参数中，`circle`给出了环绕文字（尽管为空）的形状，`draw`选项引起绘制环绕形状的动作。

事实上，`\node`命令是`\path node`的简写。更原始的版本是这样
```tex
\begin{tikzpicture}
\path node at ( 0,2) [shape=circle,draw] {}
	  node at ( 0,1) [shape=circle,draw] {}
	  node at ( 0,0) [shape=circle,draw] {}
	  node at ( 1,1) [shape=rectangle,draw] {}
	  node at (-1,1) [shape=rectangle,draw] {};
\end{tikzpicture}
```
最原始的版本没有使用`at`标记，含义并不直观，所以不再列出。

## node的尺寸
上面绘制出的node尺寸非常小，因为我们没有设定尺寸参数。node默认使用`inner sep`的值自动为node中的文字增加环绕空白，而默认的`inner sep`是很小的。可以手动修改`inner sep`的值
```tex
\begin{tikzpicture}
[inner sep=2mm,
place/.style={circle,draw=blue!50,fill=blue!20,thick},
transition/.style={rectangle,draw=black!50,fill=black!20,thick}]
  \node at ( 0,2) [place] {};
  \node at ( 0,1) [place] {};
  \node at ( 0,0) [place] {};
  \node at ( 1,1) [transition] {};
  \node at (-1,1) [transition] {};
\end{tikzpicture}
```
但是更好的做法是使用`minimum size`选项。它指定了node的*最小*尺寸：无论内部文字多么小（甚至没有），node都能达到这一最小尺寸。同时将`inner sep`设为`0pt`，这样能在`minimum size`较小的时候不至于错误地包含`inner sep`增加的空间。

## node命名
为了后续引用定义好的node（比如在node间绘制连接线），可以为node命名，以后用名字引用node。方便的做法是将名字放置在圆括号内部
```tex
% ... set up styles
\begin{tikzpicture}
  \node (waiting 1) at ( 0,2) [place] {};
  \node (critical 1) at ( 0,1) [place] {};
  \node (semaphore) at ( 0,0) [place] {};
  \node (leave critical) at ( 1,1) [transition] {};
  \node (enter critical) at (-1,1) [transition] {};
\end{tikzpicture}
```
node的语法很宽松，名字、at标识符、可选参数的顺序不做要求。

## 使用相对位置
*fixed positioning*弊端很大，不直观而且不灵活，*relative positioning*使用起来更方便。在TikZ中使用相对位置需要载入`positioning`库。于是使用相对位置改写上面的例子
```tex
\begin{tikzpicture}
  \node[place] (waiting) {};
  \node[place] (critical) [below=of waiting] {};
  \node[place] (semaphore) [below=of critical] {};
  \node[transition] (leave critical) [right=of critical] {};
  \node[transition] (enter critical) [left=of critical] {};
\end{tikzpicture}
```
当类似`below`的选项后面紧跟`of`，TikZ就会启用相对位置。**相对距离**则使用`node distance`选项。

## 为node增加文字标签
想要在node周围增加文字标签，有两种方法
1.  直接使用node，利用node的`anchor`属性
```tex
\begin{tikzpicture}
  \node[place] (waiting) {};
  \node[place] (critical) [below=of waiting] {};
  \node[place] (semaphore) [below=of critical] {};
  \node[transition] (leave critical) [right=of critical] {};
  \node[transition] (enter critical) [left=of critical] {};
  \node [red,above] at (semaphore.north) {$s\le 3$};
\end{tikzpicture}
```
2.  使用`label`选项
```tex
\tikz
  \node [circle,draw,label=60:$60^\circ$,label=below:$-90^\circ$] {my circle};
```
像上面这样，可以增加多个标签。

## node连接线