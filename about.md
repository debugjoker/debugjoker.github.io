---
layout: page
title: "关于"
permalink: about.html
image: /public/images/redflag.jpg
color: '#f44336'
sequence: 9
---


{% comment %}
  This inserts the "about" photo and text from `_config.yml`.
  You can edit it there (jekyll needs restart!) or remove it and provide your own photo/text.
  Don't forget to add the `me` class to the photo, like this: `![alt](src){:.me}`.
{% endcomment %}

{% if site.author.photo %}
  ![{{site.author.name}}]({{site.author.photo}}){:.me}
{% endif %}


我是<u>夏木逢秋</u>，目前在中国上海工作，从事 IT 行业。我喜爱阅读、电影，也喜欢运动和摄影。


## 为什么要写这个博客？

花时间进行写作是一件很有意义也很值得去做的事。我希望能在这里来分享技术、记录生活，同时也希望能结交到更多朋友。

本站使用 GitHub Page 和 Jekyll 进行搭建，主题参考[myanbin的源代码](https://github.com/myanbin/myanbin.github.io)进行修改。

## 联系我

请发邮件到[{{site.author.email}}](mailto:{{site.author.email}})与我联系。你也可以通过页面左下角的链接给我留言。


## 请我喝一杯咖啡 {#buymeacoffee}

如若阁下觉得本站内容对你有所帮助，可考虑略表心意，通过以下赞赏码来支持本人。

![Buy me a coffee](/public/images/buymeacoffee.png){:.center}
