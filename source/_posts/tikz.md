title: TikZ
tags:
  - latex
  - plot
categories:
  - writing
date: 2016-05-25 19:57:00
---
# TikZ是什么
PGF/TikZ是用于绘制几何和代数的向量图形的语言，PGF是底层语言，TikZ则是基于PGF的宏高级语言。PGF/TikZ可以在$\LaTeX$、plain $\TeX$、和ConTEXt中使用. 这里主要关注TikZ在$\LaTex$中的用法。

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



