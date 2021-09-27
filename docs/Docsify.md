[官方网站](https://docsify.js.org/#/zh-cn/)

# 1. 基本安装使用

## 初始化项目

如果想在项目的 `./docs` 目录里写文档，直接通过 `init` 初始化项目。

```bash
docsify init ./docs
```

## 开始写文档

初始化成功后，可以看到 `./docs` 目录下创建的几个文件

- `index.html` 入口文件
- `README.md` 会做为主页内容渲染
- `.nojekyll` 用于阻止 GitHub Pages 忽略掉下划线开头的文件

直接编辑 `docs/README.md` 就能更新文档内容，当然也可以[添加更多页面](zh-cn/more-pages.md)。

## 本地预览

通过运行 `docsify serve` 启动一个本地服务器，可以方便地实时预览效果。默认访问地址 http://localhost:3000 。

```bash
docsify serve docs
```



# 2. 定制侧边栏

首先在`index.html`中设置

```html
<!-- index.html -->

<script>
  window.$docsify = {
    loadSidebar: true
  }
</script>
<script src="//cdn.jsdelivr.net/npm/docsify/lib/docsify.min.js"></script>
```

接着在docs文件夹下创建 `_sidebar.md` 文件，内容如下

```markdown
<!-- docs/_sidebar.md -->

- [JavaWeb](JavaWeb.md)
- [java基础](java基础.md)
- [Java基础_2](Java基础_2.md)
- [Java面向对象](Java面向对象.md)
- [Mybatis](Mybatis.md)
```

开启目录功能，在`index.html`中添加`    subMaxLevel: 3`

```html
<script>
  window.$docsify = {
    loadSidebar: true,
    subMaxLevel: 3
  }
</script>
```

# 3. 开启封面

1. 在`index.html`中设置`coverpage: true`

```html
<script>
  window.$docsify = {
    coverpage: true
  }
</script>
```

2. 添加`_coverpage.md`文档设置封面信息

```markdown
<img width="180px" style="border-radius: 50%" bor src="img/gitlogo.png">

# 一位研一自学Java的独白

- 在科研中苦寻无果，自学Java求温饱！
- 分享我的学习历程和经验，以及收藏整理的各种Java学习书籍！

[![stars](https://badgen.net/github/stars/ahang1598/Blog?icon=github&color=4ab8a1)](https://ahang1598.github.io/Blog/) [![forks](https://badgen.net/github/forks/ahang1598/Blog?icon=github&color=4ab8a1)](https://ahang1598.github.io/Blog/)

[GitHub](<https://github.com/ahang1598/Blog/>)
[开始阅读](README.md)

```

# 4. 插件小功能



## 图片缩放 - Zoom image

Medium's 风格的图片缩放插件. 基于 [medium-zoom](https://github.com/francoischalifour/medium-zoom)。

```html
<script src="//cdn.jsdelivr.net/npm/docsify/lib/plugins/zoom-image.min.js"></script>
```

## 复制到剪贴板

在所有的代码块上添加一个简单的`Click to copy`按钮来允许用户从你的文档中轻易地复制代码。由[@jperasmus](https://github.com/jperasmus)提供。

```html
<script src="//cdn.jsdelivr.net/npm/docsify-copy-code/dist/docsify-copy-code.min.js"></script>
```

## 字数统计

这是一款为docsify提供文字统计的插件. [@827652549](https://github.com/827652549)提供

它提供了统计中文汉字和英文单词的功能，并且排除了一些markdown语法的特殊字符例如*、-等

**Add JS**
```html
<script src="//unpkg.com/docsify-count/dist/countable.js"></script>
```

**Add settings**
```js
window.$docsify = {
  count:{
    countable:true,
    fontsize:'0.9em',
    color:'rgb(90,90,90)',
    language:'chinese'
  }
}
```

check [document](https://github.com/827652549/docsify-count)



## 添加LaTeX公式插件

```javascript
<link rel="stylesheet" href="//cdn.jsdelivr.net/npm/katex@latest/dist/katex.min.css"/>

<script src="//cdn.jsdelivr.net/npm/docsify-katex@latest/dist/docsify-katex.js"></script>
```



## 侧边目录折叠

```javascript
<script>
  window.$docsify = {
    loadSidebar: true,
    subMaxLevel: 3,
    ...
    sidebarDisplayLevel: 1, // set sidebar display level
  }
</script>

<!-- plugins -->
<script src="//cdn.jsdelivr.net/npm/docsify-sidebar-collapse/dist/docsify-sidebar-collapse.min.js"></script>

```



## 添加暗黑模式

```javascript
 <link 
    rel="stylesheet"
    href="//cdn.jsdelivr.net/npm/docsify-darklight-theme@latest/dist/style.min.css"
    title="docsify-darklight-theme"
    type="text/css"
  />
        
   <script 
    src="//cdn.jsdelivr.net/npm/docsify-darklight-theme@latest/dist/index.min.js"
    type="text/javascript">
  </script>       
```



## 添加页面页脚

```javascript
 window.$docsify = {
     
	plugins: [
        function(hook) {
          var footer = [
            '<footer style="text-align: center;margin-top: 50px;">',
            '<span> Made with <svg xmlns="http://www.w3.org/2000/svg" width="24" height="15" viewBox="0 0 24 24" fill="red" stroke="red" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="feather feather-heart"><path d="M20.84 4.61a5.5 5.5 0 0 0-7.78 0L12 5.67l-1.06-1.06a5.5 5.5 0 0 0-7.78 7.78l1.06 1.06L12 21.23l7.78-7.78 1.06-1.06a5.5 5.5 0 0 0 0-7.78z"></path></svg> by <a href="https://www.linkedin.com/in/boopathikumar/" target="_blank">@boopathikumar</a>  &copy; 2020 </span>',
            '</footer>'
          ].join('');

          hook.afterEach(function(html) {
            return html + footer;
          });
        }
      ],
.....
 }     
```





# 5. 代码高亮

**docsify**内置的代码高亮工具是 [Prism](https://github.com/PrismJS/prism)。Prism 默认支持的语言如下：

* Markup - `markup`, `html`, `xml`, `svg`, `mathml`, `ssml`, `atom`, `rss`
* CSS - `css`
* C-like - `clike`
* JavaScript - `javascript`, `js`

[添加额外的语法支持](https://prismjs.com/#supported-languages)需要通过CDN添加相应的[语法文件](https://cdn.jsdelivr.net/npm/prismjs@1/components/) :

```html
<script src="//cdn.jsdelivr.net/npm/prismjs@1.23.0/components/prism-bash.min.js"></script>
<script src="//cdn.jsdelivr.net/npm/prismjs@1.23.0/components/prism-java.min.js"></script>
```

这里注意可能会出现设置后代码高亮没有显示

需要注意版本问题，最好根据默认`index.html`里面的版本来决定，然后去[这个网站](https://cdn.jsdelivr.net/npm/prismjs@1.23.0/components/)选择需要的语法支持，同时选择相应的版本

