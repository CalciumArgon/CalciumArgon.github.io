# Jekyll 搭建笔记


### 本地服务器预览网站

#### 调试与构建

代码初次拉到本地先下载相应依赖, 会根据 `Gemfile` 中指定的源地址（Gem 服务器地址）来找相关 Gem 配置
```bash
$ bundle install
```
依赖可以手写在 `Gemfile` 中, 而 Bundler 会自动生成 `Gemfile.lock` 文件来记录所有的依赖细节, 包括版本号, 下面的下载命令就是根据 `lock` 文件中的名字和版本进行相应下载. Bundler 相关的版本记录和管理语法、功能等可以参考 [Bundler的作用和原理](https://ruby-china.org/topics/25530)

检查构建是否有错误
```bash
$ jekyll build --trace
```

直接发布到本地服务器（也包含构建错误报错）
```bash
$ bundle exec jekyll serve
```

注意到发布后最后几行会给出一个服务器网址 `Server address: http://127.0.0.1:4000` , 点击就是发布的网站
其中 `127.0.0.1` 是**环回地址**, `4000` 是接口名称

环回地址是访问本机的一种方式（还可以使用 `localhost` 或直接用本机 IP）

电脑主板上内置了多种网卡, 一般主要有以下几类, 他们都绑定了一个本机 IP：

* 虚拟网卡（Loopback）

  虚拟的而非物理网卡, 也被称为**本地环回地址**（或接口）, 一般将 **127.0.0.1** 作为本地环回地址

* 有线网卡/以太网卡（Ethernet）

  这是以太网（局域网）使用的, 即日常说的网卡, 插入网线

* 无线网卡（WLAN）

  无线局域网所使用, 笔记本上内置, 使用无线电技术, 不需要像以太网卡那样插网线

而 `localhost` 是一种特殊的 IP, 默认的情况下解析到 `127.0.0.1` , 主要通过本机的 `C:\Windows\System32\drivers\etc\hosts` 文件进行管理，也就是说可以把 localhost 域名解析到任意的 IP 地址. 不过通常都解析到 `127.0.0.1(IPv4), [::1](IPv6)`

那环回地址是做什么的呢？它是主机用于向自身发送通信的一个特殊地址, 如果两项服务使用环回地址而非分配的主机地址, 就可以绕开 TCP/IP 协议栈的下层（也就是不再通过链路层、物理层、以太网传出去了, 而是可以直接在自己的网络层，运输层进行处理）
网络号为 127 的地址根本就不是网络地址, 产生的 IP 数据包不会到达外部网络接口中, 是不离开主机的包。
所以说 `127.0.0.1` 是保留地址之一，经常用来检验本机 TCP/IP 协议栈, 如果我们可以 ping 通就说明本机的网卡和 IP 协议安装都没有问题

### 样式修改和相应文件关系

```css
_posts 博客内容
_pages 其他需要生成的网页，如About页
_layouts 网页排版模板
_includes 被模板包含的HTML片段，可在_config.yml中修改位置
assets 辅助资源 css布局 js脚本 图片等
_data 动态数据
_sites 最终生成的静态网页
_config.yml 网站的一些配置信息
index.html 网站的入口
```

1. `index.html`

是网页一打开的第一界面

```yaml
---
layout: home / article / ...


---
```

2. `_data/navigation.yml`

是自定义导航栏的样式, 比如某个页面想采用左边栏设计, 需要在相应页面的 `html` 或 `md` 文件（）头部声明
```yaml
---
sidebar:
  nav: side_bar_1
---
```

其中 `side_bar_1` 就是选区的一个自定义的边栏样式, 具体的自定义方法需要就需要写在 `_data/navigation.yml` 里
```yaml
---
sidebar:
  nav: side_bar_1
---
```

3. `_includes\header.html`

```html
<!-- 注意都是双下划线 -->
<header class="header" ...>
  <div class="main">
    <div class="header__title">
      <div class="header__brand">
      <button class="button ...">
    <nav class="navigation">
```
整个文件用来控制页面整个抬头, 其中 `<header class= "" >` 是页面主题, 由 if 判断 theme 类型；`<div class="main">` 是抬头的样式；`header__title` 是抬头的左侧部分, 包含标题 `<header__brand>` 和图标 `<button>` 两部分；`<navigation>` 是右侧导航栏, 具体修改方法见下面：

想要修改导航栏的标签、标签对应的索引网站, 同样需要在 `_data/navigation.yml` 中更改 `header` 标签下的内容, 其中注意如果有多种语言的版本, 要将前面写为 `titles`, 如果不想针对多种语言, 需要写 `title` 并直接在同一行写标题 

```yaml
header:
  - titles:
      en      : &EN       Archive
      zh-Hans : &ZH_HANS  归档
    url: /archive.html

  - titles:
      en      : &EN       About
      zh-Hans : &ZH_HANS  关于
    url: /about.html

  - title: SinglePage
  - url: /
```

在构建 `_includes\header.html` 的时候会自动遍历 `header` 中的 `title` 并依次打上 `<li><a></a></li>` 的嵌套标签做成按钮, `<li class="">` 的样式有如下两种, 分别是 有下划线高亮 和 没有 的版本, 用来表示当前页面来自哪个导航栏索引

但这也意味着, 可以手动给导航栏改变索引的样式, 比如可以让 `index.html` 一上来什么都没点的时候, `About` 索引就被高亮；也可以让任何索引任何时候都不高亮标记（比如就剩一个导航栏索引, 就可以不高亮它）
```html
<li class="navigation__item">
  <a href="/archive.html">Archive</a>
</li>

<li class="navigation__item navigation__item--active">
  <a href="/about.html">About</a>
</li>
```

4. 图片和 `card` 样式

> 图片样式与 [如何添加彩色文本框和彩色标签](https://kitian616.github.io/jekyll-TeXt-theme/docs/en/additional-styles) 在一起

在 md 文件中可以用如下的图片格式来创建不同的图片板块

```markdown
![image-name](image-path){:__class__}
```

其中 `__class__` 可选有 `.border` `.shadow` `.rounded` `.circle`, 注意每个类型前面都有点, 这些类型有的是描述图片的边缘样式, 有的是描述图片的形状, 将他们组合在一起的时候每种前面都要加点, 比如 
```markdown
![image](path){:.border.rounded}
![image](path){:.circle.shadow}
![image](path){:.circle.border.shadow}
```

而在 html 文件里, 只需要改变 `<img class="" src="">` 标签中的 class 即可
```html
<img class="border" src="ai_meeting_square.png"/>
```

5. 卡片 `card` 样式

但单独一张图片不美观, 而且格式很难控制, 所以可以将图片嵌入到各种框架中, 比如 `card` 样式. 该样式本质上只有两类元素：图片和文本
```html
<div class="card">
  <div class="card__image">
  <div class="card__content">
</div>
```

其中图片下属的样式 class 就和上面 md 中的样式声明很像了, 区别就在于样式名称没有点, 并且用空格分隔. 文本下属的样式直接输入文字就行, 会自动把字放在同一张卡片里
```html
<div class="card">
  <div class="card__image">
    <img class="circle shadow" src="ai_meeting_square.png"/>
  </div>

  <div class="card__content">
    <p>图片说明</p>
  </div>
</div>
```

其他更具体的可以看 [card 样式](https://kitian616.github.io/jekyll-TeXt-theme/docs/en/card), 卡片还可以调整成 可点击、扁平化、文字在图片上.

另外我发现 `card__content` 有一个子样式 `<div class="card__header">` , 虽然加不加对于文字大小没有影响, 但是加上之后, 会使得鼠标放在卡片文字上时, 文字高亮且具有下划线（但是不知道怎么添加点击跳转链接）

6. 按钮样式

[这里写的很清楚](https://kitian616.github.io/jekyll-TeXt-theme/docs/en/button)