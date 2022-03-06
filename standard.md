## 代码基本结构

该标准是“纯文本友好”的，即如果你不想加入任何样式，那么直接使用纯文本也可以，不会产生错误（前提是不要触发保留词及正常的代码符号），即不需要像 latex 一样先设置文档属性。该标准是 html 友好的，不会对内容进行 html 转义，所以可以使用原生 html 标签，因此若想开放编辑（例如在论坛使用），需要过滤 `script` 和 `link` 标签。

该标准中，有两种基本对象，分别为“宏”和“元素”。都使用反斜线标志（与 tex 语法类似），参数在小括号中，不需要添加引号，如果不填则使用默认值，支持留空，但除换行和换段外，必须有括号。比如：
```
\setfont(size,weight,color,align,decoration,font-family)
```
是“宏”。`\setfont()` 则可以恢复默认，`\setfont(size,,,align)`则可以只修改 `size` 和 `align` 属性
```
\image(url,sizeR)
```
为“元素”，除此之外，元素还有其他形式，比如
```
\beginbox(sizeR,align,bg-color)
Something...
\endbox()
```
可继续嵌套其他内容，这对应html中的 "div"

## 可用宏

这是 1.0 版本中的可用宏。（等号后为默认值）

```
\setfont(size=1em,weight=normal,color=#000000,align=left,decoration=none,font-family="黑体"#"sans-serif")
```
注意，`font-family` 中逗号请用 # 号替代（仅这个宏如此，因为逗号会被分割成新的参数）
```
\\
```
换行，从下一行的最左端开始。
```
\par
```
换段。注意：由于采用正则表达式搜索，换行和换段和接下来的内容之间必须有空格。
## 可用元素

```
\beginbox(sizeR=100%,align=default,background-color=transparent)
Something...
\endbox()
```
容器，相当于 "div"。其中 `sizeR` 为相对尺寸，按照 css 的定义处理（仅横向，纵向会自动撑开）。`align` 的取值为 `center` 或 `default`。`background-color` 为背景颜色，按照 css 的定义处理。
```
\title(text=default)
```
如果传入了参数则以此为标题，如不传入则向编译器请求标题。编译器设计中，若没有提前在外部定义标题，则应抛出错误 `Argument missing Exception`。
```
\smalltitle(text=sample)
```
小标题
```
\image(url,sizeR=100%)
```
必选参数 `url`。`sizeR` 按 css 的 `width` 处理。

```
\\CODE
some code here
\\CODE
```
代码块。

```
\backslash
```
反斜线，即 `\`，因为 `\` 是保留字符，所以需要这样输出
```
\dollar
```
美元符号，即 `$`，因为 `$` 是保留字符，所以需要这样输出
```
\backquote
```
反引号，即 `` ` ``，因为 `` ` `` 是保留字符，所以需要这样输出
### 特殊对象
行内公式，例如：
```
$f(x)$ is a function
```
按照 LaTeX 处理。  
整行公式：
```
$$f(x)=e^x$$
```
行内代码：
```
`some code here`
```
需要提前在 TSStudio 代码上传器上传。此处填写编号。行内代码不进行高亮。

## 编译器的算法逻辑及转义
如果你想在文章中直接显示上面所提到的关键字，你就需要用到转义。由于本标准不涉及到“字符串”类型和引号，因此没有多层转义，你只需要注意上面提到的转义字符即可，如 `\backslash`。  
编译器的逻辑应当为：  
1> 公式、代码内的内容，永远不会被认为是“宏”或“元素”（即该部分内容会优先被“保护”起来，使用占位符替换掉）。    
2> 从前到后遍历文章内的 `\`，使用正则表达式匹配 `\\[a-zA-Z-\\]+` 并校验参数是否合法。只要发现不合法，立刻抛出错误并停止解析，若无错误加入 html 标签。  
3> 将公式和代码加回去。
