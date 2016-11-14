title: hexo_install_config
categories:
  - 工具
  - hexo
date: 2016-05-22 20:13:53
tags:
---
title: hexo_install_config
date: 2016-05-22 20:13:53
categories:
- 工具
- hexo
tags:
---

经过两天折腾，hexo总算像模像样了。中间走了不少弯路，这里记录下来方便后来人。

# 安装hexo
首先建议安装_git_，因为在后面使用主题时会用到。在windows下可以使用*git for windows*代替，一路默认即可。

接下来下载安装_node.js_。从官网下载安装即可。

下面进入正题。安装hexo是极为简单的。

1.  用_node.js_的包管理工具`npm`安装hexo:
> $ npm install -g hexo-cli

2.  在目标文件夹`<folder>`中初始化hexo:
> \$ hexo init &lt;folder>
\$ cd &lt;folder>
$ npm install

# 配置hexo
安装容易配置难。hexo可配置的内容非常多，建议仔细阅读hexo官方docs，当然也可以按照我的顺序来~~挂了我可不负责哟~~。这里暂不涉及hexo主题，hexo主题放在下一节。以下内容分别为

1.  hexo的目录结构
2.  作者信息相关配置
3.  git部署
4.  mathjax支持
5.  流程图支持

<!-- more -->

## hexo的目录结构
安装hexo完毕后，`<folder>`目录结构是下面这样（B.T.W. 使用[GnuWin32](http://gnuwin32.sourceforge.net/packages/tree.htm)的`tree`程序打印，需修改`Path`，linux可直接使用`tree`)
{% code blog %}
|-- _config.yml
|-- db.json
|-- node_modules
|-- package.json
|-- scaffolds
|-- source
`-- themes
{% endcode %}

### config.yml
`_config.yml`是主要的配置文件，和`NexT`主题保持一致我们称之为站点配置文件。顾名思义，修改它将导致全局变更。

### package.json
`package.json`记录了hexo已安装的模块，下面是我的输出
{% codeblock package.json %}
{
  "name": "hexo-site",
  "version": "0.0.0",
  "private": true,
  "hexo": {
    "version": "3.2.0"
  },
  "dependencies": {
    "hexo": "^3.1.1",
    "hexo-deployer-git": "^0.1.0",
    "hexo-generator-archive": "^0.1.4",
    "hexo-generator-category": "^0.1.3",
    "hexo-generator-feed": "^1.1.0",
    "hexo-generator-index": "^0.2.0",
    "hexo-generator-tag": "^0.2.0",
    "hexo-inject": "^1.0.0",
    "hexo-math": "^3.0.1",
    "hexo-renderer-ejs": "^0.1.1",
    "hexo-renderer-marked": "^0.2.9",
    "hexo-renderer-stylus": "^0.3.0",
    "hexo-server": "^0.1.3"
  }
}
{% endcodeblock %}
其中`hexo-generator-feed`、`hexo-deployer-git`和`hexo-math`是后来安装的插件，`hexo-inject`是`hexo-math`的依赖项，其余是hexo标准安装自带。

### node_modules
`node_modules`目录保存hexo模块不需要管它。

### source
`source`目录保存我的原始文稿，markdown文件一般保存在`_posts`目录下。除此之外还可能有`_drafts`目录，它和hexo的草稿功能相关，暂时没有使用到。hexo对于`source`目录的处理规则是：*hexo忽略除`_posts`外所有以`_`为前缀的文件和文件夹以及隐藏文件，__可渲染文件__(markdown, html)被处理并保存至`public`目录下*。如果想要添加额外忽略规则，可以在`_config.yml`文件中搜索到`skip_renderer`标签加上想要忽略的文件/文件夹，如我的`source`结构为
{% code source %}
|-- 404.html
|-- _posts
|-- categories
|-- images
`-- tags
{% endcode %}

想要忽略`404.html`，直接在`_config.yml`中写入
``` 
skip_render: 404.html
```

### scaffold
`scaffold`文件夹保存了模板文件如`post.md`，它给出post的默认_front matter_，hexo会根据它创建新文档。

### themes
hexo非常重要的部分，一个theme定义了页面的外观和行为。想使用一个新的主题时，将主题`git clone`到`themes`文件夹中。我的themes结构如下
{% code themes %}
|-- chan
|-- landscape
|-- next
`-- yilia
{% endcode %}

接着在`_config.yml`中找到`theme`标签更改为想要使用的theme名称。一个良好的theme有很多可配置的内容，在下一节会以`NexT`为例介绍。

### 作者信息相关配置
无论选用什么主题，更改哪些设置，作者信息应该是不变的。

|设置 | 描述 |
| --- | --- |
|title | 站点的主标题|
|subtitle | 站点副标题|
|description | 站点描述，类似格言的样子|
|author | 你的名字|
|language | 站点使用的语言，hexo默认使用[2-lettter ISO-639-1 code](https://en.wikipedia.org/wiki/List_of_ISO_639-1_codes)，默认值是`en`。theme可能有不同的定义，参考theme的说明文档|
|timezone | 站点使用的时区。默认使用系统默认时区，可不作改动|

### git部署
想要自动部署到github上的page site（即*your_name.github.io*）需要一个hexo插件`hexo-deployer-git`。hexo插件的命名法是*hexo + &lt;category> + &lt;specific name>*，`hexo-deployer-git`表明这一插件和部署相关，具体使用git部署。同样使用*node.js*的npm管理器安装
```
npm i hexo-deployer-git --save
```
下面是配置`_config.yml`中和部署相关的内容。首先找到`deploy`标签，在下面添加三个二级标签（两空格缩进）

| label | description |
| --- | --- |
| `type` | 部署类型，这里填git |
| `repo` | page site所在仓库，填写https://github.com/*your_name/your_name.github.io* |
| `branch` | 所在分支，如果是user pages填master，project pages填gh-pages（一般默认使用user pages）

接着配置Url相关内容。下面是四个关联标签

| label | description |
| --- | --- |
| `url` | 你的博客url，user pages填https://&lt;your_name>.github.io，project pages填http(s)://&lt;your_name>.github.io/&lt;projectname>
| `root` | 由于博客内容一般直接放在github repo的根目录下，所以直接写`/`。如果放在`folder`下，则改为`/folder/` |
| `permalink` | hexo产生html的路径名。默认是`:year/:month/:day/:title/`，表示html按照编辑日期（由markdown文档的*front matter*指定）存放和引用
| `permalink_default` | 如果html没有对应的permalink，则按照这一标签的值处理。默认留空，不需要更改

permalink中的占位符有以下可选

| placeholder | description |
| --- | --- |
| `:year` | 文稿年份，四位数字 |
| `:month` | 文稿月份，两位数字 |
| `:day` | 文稿日期，两位数字 |
| `:title` | 文稿标题 |
| `:id` | 文稿id |
| `:category` | 文稿所属类别，如果没有类别则为`default_category`的值 |
|`:i_month` | 去掉0前缀 |
| `:i_day` | 去掉0前缀 |

配置好之后，首先保险起见`hexo clean`一下，然后`hexo d --g`（`hexo generate`然后再`hexo deploy`的简写）。如果没有push权限，请先配置ssh key再试。有可能报`directory is not a git repo`类似错误，在`.deploy_git`目录下运行`git init`即可。`.deploy_git`是`hexo-deployer-git`根据`public`的内容复制出来的用于push的目录。

### mathjax支持
我需要hexo排版数学公式和符号，最好能和$\LaTeX$的用法一致。mathjax符合这一要求。使用mathjax本身很简单，只需要在html中插入几段javascript即可。实际上很多主题也是采取类似方式，参考[常规方法一节](http://lukang.me/2014/mathjax-for-hexo.html)。如[进阶版一节](http://lukang.me/2014/mathjax-for-hexo.html)所说，另外很多主题通过一个`mathjax`标签的布尔值动态加载mathjax，速度会更快。然而这两种方法都有一些缺陷

*  下划线会被看成斜体
*   带有很多特殊符号的命令[会被markdown parser吃掉](http://catx.me/2014/03/09/hexo-mathjax-plugin/)

如果采用`inline mathjax`，下划线只能通过反斜线转义。而一个很大的问题是，`mathjax environment`即类似$\LaTeX$的环境如果全部转义工作量会很大。有一个很好的使用mathjax的工具插件`hexo-math`，它定义了mathjax tag能够包裹一大段mathjax代码使之不受markdown parser干扰。安装方法同样是使用`npm`安装，安装完毕后在`_config.yml`中配置。如果使用的主题提供`mathjax`标签可以设为`false`禁用。注意查看github上的主页，`hexo-math`的用法经过若干次改动，最新的用法可能和这里的记载不同。mathjax tag的用法实例

```
{% math %}
\begin{align*}
-\sum_i{P_i\log_2\frac{P_i}{Q_i}} =& \sum_i{P_i\log_2\frac{Q_i}{P_i}} \\
\le& \log_2\sum_i{P_i\frac{Q_i}{P_i}} \tag{Jensen's Inequality} \\
=& \log_2\sum_i{Q_i} \\
=& 0
\end{align*}
{% endmath %}
```

{% math %}
\begin{align*}
-\sum_i{P_i\log_2\frac{P_i}{Q_i}} =& \sum_i{P_i\log_2\frac{Q_i}{P_i}} \\
\le& \log_2\sum_i{P_i\frac{Q_i}{P_i}} \tag{Jensen's Inequality} \\
=& \log_2\sum_i{Q_i} \\
=& 0
\end{align*}
{% endmath %}

### 流程图支持
作为一个码农，经常需要画图表达自己的构想，框图是必要的，能够产生uml图就更好了。hexo社区一款插件解决了这一需求，`hexo-tag-plantuml`基于[plantuml](http://plantuml.com/)开发，能够简单地在markdown中插入uml图。安装配置不多说了，还是`npm`加上`_config.yml`。最后一遍强调一下，配置前查阅github的readme文档。

uml图示例

{% plantuml %}
    Alice->Bob: Hello Bob, how are you?
Note right of Bob: Bob thinks
Note left of Alice: Alice thinks
Bob-->Alice: I am good thanks!
{% endplantuml %}

# 选用主题

# 丰富你的页面
[各种颜色标签、小图标等等](https://github.com/wzpan/hexo-tag-bootstrap) \(限定 freemind 主题\)   
[github提交卡片](https://github.com/Gisonrg/hexo-github-card)  
[各类视频](https://github.com/m80126colin/hexo-tag-owl)  
[各种 tag](https://github.com/search?utf8=%E2%9C%93&q=hexo+tag)