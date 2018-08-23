---
title: Hexo优化备忘(1)
date: 2018-05-11 09:07:25
categories: 
- 技术
- Hexo
tags: [Hexo,Next]
copyright: true
reward: false
---

# 概述
昨天看了些优化教程，对博客进行了些优化，现在将它们整合下。
<!-- more -->
```html
<!-- HTML方式: 直接在 Markdown 文件中编写 HTML 来调用 -->
<!-- 其中 class="blockquote-center" 是必须的 -->
<blockquote class="blockquote-center">This is an example</blockquote>

<!-- 标签 方式，要求版本在0.4.5或以上 -->
{% centerquote %}This is an example{% endcenterquote %}

<!-- 标签别名 -->
{% cq %} This is an example {% endcq %}
```

<blockquote class="blockquote-center">This is an example.</blockquote>

# 推荐的文章
[hexo的next主题个性化教程:打造炫酷网站](http://shenzekun.cn/hexo%E7%9A%84next%E4%B8%BB%E9%A2%98%E4%B8%AA%E6%80%A7%E5%8C%96%E9%85%8D%E7%BD%AE%E6%95%99%E7%A8%8B.html)  
[hexo的next主题个性化配置教程](https://segmentfault.com/a/1190000009544924)  

这两篇文章似乎是一样的，相似的实在是有点多，找了放两篇作参考
[Hexo+Next主题优化](http://www.dragonstyle.win/2017/11/07/Hexo-Next%E4%B8%BB%E9%A2%98%E4%BC%98%E5%8C%96/)  

# 另外一些优化技巧
主要来自官方文档及自己摸索。  
首先是自己探索的
## 文章内前后文章按扭交换
Hexo在文章底部有到前一篇文章和后一篇文章的链接，但奇怪的是前一篇在右边，后一篇在左边。因此一个个翻渲染文档，终于被我找到了！
```
.\themes\next\layout\_macro\post.swig
```
找到时其中这一段代码
```html
 {% if not is_index and (post.prev or post.next) %}
        <div class="post-nav">
          <div class="post-nav-next post-nav-item">
            {% if post.prev %}
              <a href="{{ url_for(post.prev.path) }}" rel="prev" title="{{ post.prev.title }}">
                <i class="fa fa-chevron-left"></i> {{ post.prev.title }}
              </a>
            {% endif %}
          </div>

          <span class="post-nav-divider"></span>

          <div class="post-nav-prev post-nav-item">
            {% if post.next %}
              <a href="{{ url_for(post.next.path) }}" rel="next" title="{{ post.next.title }}">
                {{ post.next.title }} <i class="fa fa-chevron-right"></i>
              </a>
            {% endif %}
          </div>
        </div>
      {% endif %}
```
其中有两行类似的代码如下
```html
 <a href="{{ url_for(post.prev.path) }}" rel="prev" title="{{ post.prev.title }}">
                <i class="fa fa-chevron-left"></i> {{ post.prev.title }}
```
然后把这两行中的`next`和`prev`交换下就可以实现（前面的`div`标签名也可以顺便改下）

## 修改文章页宽
打开`themes/next/source/css/_variables/base.styl`，找到以下字段并修改为合适的宽度：

```
$main-desktop                   = 1080px
$main-desktop-large             = 1200px

$content-desktop                = 750px //默认700
$content-desktop-large          = 900px
$content-desktop-padding        = 40px
```

## 引用自己博客的文章
这个可以在文末用于建关联文章
```html
{% post_link 文章文件名（不要后缀） 文章标题（可选） %}
```
文章文件名是指`.\source\_posts`下的文件名
文章标题（可选）当填写时，填写的内容作为超链接的文本，否则自动识别文章标题
因为文章的链接标签的颜色是后改的，这里的链接颜色没变，可能是生成机制的问题，然后看下源代码我们发现，文章链接是在`<p>`标签下的！所以前面加下`<p>`标签试试，发现成功了！
这里的语句要尤其当心，如果**漏输**了东西，**hexo会报错！**
```html
{% post_link 2018-04-25-jlucjcx %}
<p>{% post_link 2018-04-25-jlucjcx %}</p>
```
{% post_link 2018-04-25-jlucjcx %}
<p>{% post_link 2018-04-25-jlucjcx %}</p>

```
{% post_link 2018-04-25-jlucjcx 这是个示例 %}
```
{% post_link 2018-04-25-jlucjcx 这是个示例 %}


然后是来自[官方文档](http://theme-next.iissnan.com/tag-plugins.html)的一些 

## 文章居中引用
使用方式
* HTML方式：使用这种方式时，给 img 添加属性 class="blockquote-center" 即可。
* 标签方式：使用 centerquote 或者 简写 cq。

## 突破容器宽度限制的图片
当使用此标签引用图片时，图片将自动扩大 26%，并突破文章容器的宽度。 此标签使用于需要突出显示的图片, 图片的扩大与容器的偏差从视觉上提升图片的吸引力。 此标签有两种调用方式（详细参看底下示例）：
* HTML方式：使用这种方式时，为 img 添加属性 class="full-image"即可。
* 标签方式：使用 fullimage 或者 简写 fi， 并传递图片地址、 alt 和 title 属性即可。 属性之间以逗号分隔。

```html
<!-- HTML方式: 直接在 Markdown 文件中编写 HTML 来调用 -->
<!-- 其中 class="full-image" 是必须的 -->
<img src="/image-url" class="full-image" />

<!-- 标签 方式，要求版本在0.4.5或以上 -->
{% fullimage /image-url, alt, title %}

<!-- 别名 -->
{% fi /image-url, alt, title %}
```
## Bootstrap Callout 
提示块的设置
```html
{% note class_name %} Content (md partial supported) {% endnote %}
```
`class_name`可以是:
* 空
* `default`
* `primary`
* `success`
*  `info`
* `warning`
*  `danger`

{% note %} Content no class name {% endnote %}

{% note default %} class name = default  Content (md partial supported) {% endnote %}

{% note primary %} class name = primary  Content (md partial supported) {% endnote %}

{% note success %} class name = sucess  Content (md partial supported) {% endnote %}

{% note info %} class name = info  Content (md partial supported) {% endnote %}

{% note warning %} class name = warning  Content (md partial supported) {% endnote %}

{% note danger %} class name = danger  Content (md partial supported) {% endnote %}

## Github Fork
[hexo的next主题个性化教程:打造炫酷网站](http://shenzekun.cn/hexo%E7%9A%84next%E4%B8%BB%E9%A2%98%E4%B8%AA%E6%80%A7%E5%8C%96%E9%85%8D%E7%BD%AE%E6%95%99%E7%A8%8B.html) 这篇文章中介绍了`Fork me on Github`的配置，看了下还是蛮喜欢的，但直接照这个弄在用手机访问时`fork me on Github`并不会随着浏览器大小变化，导致页面访问出现问题。而修改css等又不是我擅长的，在一番查找后找到了解决办法。  
首先按上述文章中添加`Fork me on Github`  
然后打开`\themes\next\source\css\_custom\custom.styl`  
在文件下下添加下述代码 
```css
.forkme{
 display: none;
 }
  @media (min-width: 768px) {
 .forkme{
 display: inline;
 }
  }
```
最后在第一步添加的代码前后分别添加下述代码(放入名为`forkme`的`div`标签下)  
```html
<div class="forkme">

#第一部添加的代码#

<\div>
```