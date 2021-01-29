---
title: LaTeX使用入门
permalink: latex-beginning/
date: 2017-04-03 19:28:49
tags:
- LaTeX
- 论文
- Legacy
categories:
- 技术笔记
author: Kaku
---
**LaTeX**（正式名称：L<sup>A</sup>T<sub>E</sub>X，音：/ˈleɪtɛk/)，相信每一个研究生以上学历的人或者是做过科研的高年级本科生都很熟悉了。它是一个基于**TeX**的排版系统，由美国计算机学家莱斯利·兰伯特（Leslie Lamport）在 20 世纪 80 年代初期开发。利用这种格式，即使使用者没有排版和程序设计的知识也可以充分发挥由 TeX 所提供的强大功能，能在几天，甚至几小时内生成很多具有书籍质量的印刷品。上到论文，下到简历，LaTeX都会成为一个有力的帮手。

<!--more-->

---

## 1.准备
TeX系统遍布于各个平台，包括Linux、Windows、OS X等。常见的Unix/Linux下的TEX系统是teTeX，Windows下则有MiKTeX和fpTeX。

**CTeX 中文套装**是基于 Windows 下的 MiKTeX 系统，集成了编辑器 WinEdt 和 PostScript 处理软件 Ghostscript 和 GSview 等主要工具。CTeX 中文套装在 MiKTeX 的基础上增加了对中文的完整支持。

> CTeX官网：http://www.ctex.org/HomePage

注意：*安装程序在某些情况下可能覆盖 path 环境变量，原因不明。请在安装前注意备份 path 环境变量。* ~~更博的时候发现node、git全不好用的时候才在官网看到这句话~~

---

## 2.开始
打开安装目录下的WinEdt，File->New，我们就得到一个空白文档。

LaTeX文档分为宏定义与正文两部分。在开始一篇文档之前，我们需要添加一些必要的宏定义。

```LaTeX
\documentclass{article}              %定义文档为article类型
\usepackage{CJK}                     %中文支持包
\usepackage{indentfirst}             %首行缩进2em
\setlength{\parindent}{2em}
\usepackage{geometry}                %页边距设置
\geometry{left=1.0cm, right=1.0cm, top=1.0cm, bottom=1.0cm}
```

接下来就是LaTeX的正文部分，正文需要一组`\begin{document}`和`\end{document}`。

```LaTeX
\begin{document}
Hello, LaTeX!
\end{document}
```

如果正文包含中文，则需要一组`\begin{CJK*}`和`\end{CJK*}`。

```LaTeX
\begin{document}
\begin{CJK*}{GBK}{song}   %GBK表示汉字编码字符集，song表示宋体
你好，LaTeX！
\end{CJK*}
\end{document}
```

---

## 3.编译
选中编译，文件保存为.tex格式。之后单击红色框图标就可以查看生成文档了。

---

## 4.插入标题与作者
我们可以在正文指定文档标题和作者。

```LaTeX
\title{LaTeX测试文本\footnote{标题可以添加脚注，以“*”为标记}}
\date{}        %手动定义文档日期，不写参数则不显示日期
\author{7mA}
\maketitle     %maketitle把标题和作者插入到生成的文本中
```

此外，LaTeX也支持正文中的多级标题，最多支持三级，标题序号由系统自动生成。

```LaTeX
\section{一级标题}
	\subsection{二级标题}
		\subsubsection{三级标题}
```

---

## 5.换行与缩进
和Markdown类似，在编辑器里直接回车换行，生成的文档不会换行缩进，需要空一行才可以。

```LaTeX
\begin{document}
\begin{CJK*}{GBK}{song}
你好，LaTeX！
很高兴认识你！      %不会换行

你好，LaTeX！

很高兴认识你！      %换行缩进
\end{CJK*}
\end{document}
```

---

## 6.插入算法
LaTeX最重要的功能之一就是插入算法了。插入算法之前，我们需要在宏定义部分添加两个包。

```LaTeX
\usepackage{algorithmic}
\usepackage{algorithm}
```

如果你的文档是中文的，那么需要在`\begin{CJK*}`加上一个更名操作`\floatname{algorithm}{算法}`。

算法的结构如下，算法的序号自动生成。

```LaTeX
\begin{algorithm}[H]                  %[H]代表排版方式为当前位置
	\caption{直接选择排序}
	\label{alg:sel}               %引用标识
	\begin{algorithmic}[1]        %[n]代表每n行生成一个行号
		\STATE template $<$class Record$>$      %$中间的字符按数学符号的形式生成
		\STATE void SelectSort(Record Array[], int n)\{
			\STATE int i, j, Smallest;
			\FOR{($i=0$; $i<n-1$; $i++$)}
				\STATE Smallest=i;
				\FOR{($j=i+1$; $j<n$; $j++$)}
					\IF{(Array[j]$<$Array[Smallest])}
						\STATE Smallest=j;
					\ENDIF
				\ENDFOR
				\STATE swap(Array, i, Smallest);
			\ENDFOR
		\STATE \}             %注意大括号需要用“\”转义
	\end{algorithmic}
\end{algorithm}
```

此外，你还可以在其他文本中引用这个算法。

```LaTeX
直接选择排序算法如算法\ref{alg:sel}所示。
```

---

## 7.插入图片
LaTeX 文档支持多种图片格式，如.jpg，.eps，.tif 等，不同的图片格式需要用不同的方式编译。下面以.jpg为例。

LaTeX 中插入图片需要使用 graphicx 包。

```LaTeX
\usepackage{graphicx}
```

如果你的文档是中文的，那么需要在`\begin{CJK*}`加上一个更名操作`\renewcommand\figurename{图}`。

插入方式如下，注意图片所在目录与.tex文件相同。如不相同，则需要在宏定义里指定路径。

```LaTeX
\begin{figure}[H]                                        %[H]代表排版方式为当前位置
	\centering
	\includegraphics[width=0.3\textwidth]{test.jpg}  %width指定图片宽度
	\caption{这是一张测试图}
	\label{fig:examp}                                %引用标记
\end{figure}
```

---

## 8.其他
关于页码，LaTeX会自动在每一页页脚生成页码，可以在正文开头使用`\pagestyle{empty}`取消页脚页码。

使用了`\maketitle`的页不会受`\pagestyle{empty}`影响，因此需要在该页另加一句`\thispagestyle{empty}`。

LaTeX还支持表格、公式的插入，也可以插入相应期刊会议的模板，生成标准的格式。

---

**Reference Source:**
1. [CTeX 套装](http://www.ctex.org/CTeX "CTeX 套装")
2. [Latex强制图片位置](http://blog.csdn.net/lqhbupt/article/details/24812993 "Latex强制图片位置")
3. [Re: 在latex中如何删除页码？](http://www.newsmth.net/nForum/#!article/Unix/7033 "Re: 在latex中如何删除页码？")
