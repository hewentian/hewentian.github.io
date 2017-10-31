---
title: markdown 学习笔记
date: 2017-09-27 16:56:35
categories: other
---

今天开始学习下 **markdown** 这种标记语言，我习惯了学习一门语言，会去看它的官方文档里面的语法说明，**markdown** 作者的官方网站：

[https://daringfireball.net/projects/markdown/syntax](https://daringfireball.net/projects/markdown/syntax "点击将跳到目标页")

极力推荐大家去看下。方便自己以后查看，摘录如下：

在 markdown 中插入 `<br />`，只需在行尾添加 2 个空格，然后回车，即可

要使 markdown 中的#，`>` 等，显示原义，只要在它们前置 4 个空格即可

### 1. 标题
标题有1-6级，使用下面这种方式产生：
    # This is an H1
    ## This is an H2
    ###### This is an H6

**Example as below:**	
# This is an H1
## This is an H2
###### This is an H6

------
### 2. blockquote
blockquote 以 `>` 开头，一段当中每一行的开头都加上，你也可以偷懒，只在段落的第一行加上，并且它可以 nested 其他  
markdown 标记，
    > This is a blockquote with two paragraphs. Lorem ipsum dolor sit amet,
    > consectetuer adipiscing elit. Aliquam hendrerit mi posuere lectus.
    > Vestibulum enim wisi, viverra nec, fringilla in, laoreet vitae, risus.
    > 
    > Donec sit amet nisl. Aliquam semper ipsum sit amet velit. Suspendisse
    > id sem consectetuer libero luctus adipiscing.

**Example as below:**
> This is a blockquote with two paragraphs. Lorem ipsum dolor sit amet,
> consectetuer adipiscing elit. Aliquam hendrerit mi posuere lectus.
> Vestibulum enim wisi, viverra nec, fringilla in, laoreet vitae, risus.
> 
> Donec sit amet nisl. Aliquam semper ipsum sit amet velit. Suspendisse
> id sem consectetuer libero luctus adipiscing.	

blockquote 还可以 nested 其他 blockquote
    > This is the first level of quoting.
    >
    > > This is nested blockquote.
    >
    > Back to the first level.

**Example as below:**
> This is the first level of quoting.
>
> > This is nested blockquote.
>
> Back to the first level.

------
### 3. List
列表分为有序和无序
#### 3.1 无序列表写法如下：
    * Red
    * Green
    * Blue

**Example as below:**
* Red
* Green
* Blue

#### 3.2 有序列表写法如下：
    1. Red
    2. Green
    3. Blue

**Example as below:**
1. Red
2. Green
3. Blue

其中的序号，不影响它的效果，你也可以像下面这样写，它的效果是一样的。
    1. Red
    1. Green
    1. Blue
	或
	1. Red
    6. Green
    8. Blue

List items may consist of multiple paragraphs. Each subsequent paragraph in a list item must be indented by either 4 spaces or one tab:
    1.  This is a list item with two paragraphs. Lorem ipsum dolor
        sit amet, consectetuer adipiscing elit. Aliquam hendrerit
        mi posuere lectus.

        Vestibulum enim wisi, viverra nec, fringilla in, laoreet
        vitae, risus. Donec sit amet nisl. Aliquam semper ipsum
        sit amet velit.

    2.  Suspendisse id sem consectetuer libero luctus adipiscing.
**Example as below:**
1.  This is a list item with two paragraphs. Lorem ipsum dolor
    sit amet, consectetuer adipiscing elit. Aliquam hendrerit
    mi posuere lectus.

    Vestibulum enim wisi, viverra nec, fringilla in, laoreet
    vitae, risus. Donec sit amet nisl. Aliquam semper ipsum
    sit amet velit.

2.  Suspendisse id sem consectetuer libero luctus adipiscing.

To put a code block within a list item, the code block needs
to be indented *twice* -- 8 spaces or two tabs:
    * A list item with a code block:
            <code goes here>
**Example as below:**
* A list item with a code block:
        printf("hello world")

------
### 4. CODE BLOCKS
To produce a code block in Markdown, simply indent every line of the block by at least 4 spaces or 1 tab. For example, given this input:

	This is a normal paragraph:
		This is a code block.

**Example as below:**
This is a normal paragraph:

	This is a code block.

------
### 5. HORIZONTAL RULES
产生水平分割线的方法有以下几种，它们的效果都是一样的
    * * * 
    ***
    ******
    - - -
    ------

------
### 6. LINKS
    This is [an example](http://example.com/ "Title") inline link.

    [This link](http://example.net/) has no title attribute.
**Example as below:**
This is [an example](http://example.com/ "Title") inline link.

[This link](http://example.net/) has no title attribute.

If you’re referring to a local resource on the same server, you can use relative paths:

    See my [About](/about/) page for details.   
**Example as below:**
See my [About](/about/) page for details.

你还可以这样写 links
Reference-style links use a second set of square brackets, inside which you place a label of your choosing to identify the link:

    This is [an example][id] reference-style link.

Then, anywhere in the document, you define your link label like this, on a line by itself:
    [id]: http://example.com/  "Optional Title Here"
**Example as below:**
This is [an example][ae_id] reference-style link.

[ae_id]: http://example.com/ "Optional Title Here"

Link definition names may consist of letters, numbers, spaces, and punctuation — but they are not case sensitive. E.g. these two links:

    [link text][a]
    [link text][A]
are equivalent.

The implicit link name shortcut allows you to omit the name of the link, in which case the link text itself is used as the name. Just use an empty set of square brackets — e.g., to link the word “Google” to the google.com web site, you could simply write:

    [Google][]
And then define the link:

    [Google]: http://google.com/
**Example as below:**
In china, you can't use [Google][], hahaha...
[Google]: http://google.com/

------
### 7. EMPHASIS
Markdown treats asterisks (*) and underscores (_) as indicators of emphasis. Text wrapped with one * or _ will be wrapped with an HTML <em> tag; double *’s or _’s will be wrapped with an HTML <strong> tag. E.g., this input:

    *single asterisks*
    _single underscores_
    **double asterisks**
    __double underscores__
will produce:

    <em>single asterisks</em>
    <em>single underscores</em>
    <strong>double asterisks</strong>
    <strong>double underscores</strong>

**Example as below:**
*single asterisks*
_single underscores_
**double asterisks**
__double underscores__

To produce a literal asterisk or underscore at a position where it would otherwise be used as an emphasis delimiter, you can backslash escape it:

    \*this text is surrounded by literal asterisks\*

------
### 8. CODE
To indicate a span of code, wrap it with backtick quotes (`). Unlike a pre-formatted code block, a code span indicates code within a normal paragraph. For example:

    Use the `printf()` function.
will produce:

    <p>Use the <code>printf()</code> function.</p>

**Example as below:**
Use the `printf()` function.

To include a literal backtick character within a code span, you can use multiple backticks as the opening and closing delimiters:

    ``There is a literal backtick (`) here.``
which will produce this:

    <p><code>There is a literal backtick (`) here.</code></p>

**Example as below:**
``There is a literal backtick (`) here.``

Other code format:

	` `` java
		public static void main(String[] args) {
			System.out.println("hello world.");
		}
		注意：请将这里的`之间的空格去掉
	` ``

**Example as below:**
``` java
	public static void main(String[] args) {
		System.out.println("hello world.");
	}
```
	
------
### 9. IMAGES
Inline image syntax looks like this:

    ![Alt text](/path/to/img.jpg)
    ![Alt text](/path/to/img.jpg "Optional title")
**Example as below:**
![Alt text](http://www.apache.org/img/asf_logo.png "Optional title")

Reference-style image syntax looks like this:

    ![Alt text][id]
Where “id” is the name of a defined image reference. Image references are defined using syntax identical to link references:

    [id]: url/to/image  "Optional title attribute"

------
### 10. MISCELLANEOUS
AUTOMATIC LINKS

Markdown supports a shortcut style for creating “automatic” links for URLs and email addresses: simply surround the URL or email address with angle brackets. What this means is that if you want to show the actual text of a URL or email address, and also have it be a clickable link, you can do this:

    <http://example.com/>
Markdown will turn this into:

    <a href="http://example.com/">http://example.com/</a>

BACKSLASH ESCAPES

Markdown allows you to use backslash escapes to generate literal characters which would otherwise have special meaning in Markdown’s formatting syntax. For example, if you wanted to surround a word with literal asterisks (instead of an HTML <em> tag), you can use backslashes before the asterisks, like this:

    \*literal asterisks\*
Markdown provides backslash escapes for the following characters:

    \   backslash
    `   backtick
    *   asterisk
    _   underscore
    {}  curly braces
    []  square brackets
    ()  parentheses
    #   hash mark
    +   plus sign
    -   minus sign (hyphen)
    .   dot
    !   exclamation mark


创建表格语法如下：
| Tables        | Are           | Cool  |
| ------------- |:-------------:| -----:|
| col 3 is      | right-aligned | $1600 |
| col 2 is      | centered      |   $12 |
| zebra stripes | are neat      |    $1 |

Markdown 生成表格时，可以给一整列的单元格设置同样的对齐样式
| 第一列 	| 第二列	| 第三列	|
| -------:	| :------:	| :-------	|
| 右对齐	| 居中		| 左对齐	|

end.
