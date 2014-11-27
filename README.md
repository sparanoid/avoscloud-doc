# LeanCloud Documentation

LeanCloud 开发者文档

## 说明

这个项目是 LeanCloud [文档](http://leancloud.cn/docs/)上的所有文档的 Markdown 格式的源码，最终将渲染成你在网站上看到的 [HTML 文档](http://leancloud.cn/docs/)。

因此 Markdown 文件里部分链接写的是最终渲染后的链接，如果直接点击会发现 404 错误。敬请谅解。

## 贡献

我们欢迎所有用户为我们贡献或者修正错误。您只要 [fork](https://github.com/leancloud/avoscloud-doc/fork) 这个项目，并提交 [Pull Request](https://github.com/leancloud/avoscloud-doc/pulls) 即可。

## 协议

[GNU LGPL](https://www.gnu.org/licenses/lgpl.html)

## 书写注意事项

由于文档会采用 Angular 渲染，当文档中需要显示 `{{content}}` 这种格式时，外面需要加上 `<span ng-non-bindable></span>`，以不被 Angular 渲染。
