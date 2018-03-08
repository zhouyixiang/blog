---
title: Laravel Mix 分开打包
date: 2017-10-10 13:41:06
tags:
---

通常一个Web项目会包含多个端，比如至少有用户端和管理端。有些项目，我们会按照不同的端将代分拆到不同的项目中，有些项目同一端也会将前后端项目进一步拆开。不过在一些规模不大的项目或者项目初期，为了简单，我们也常常把这些各个端的代码放到一个项目中。

在使用Laravel进行开发时，Laravel Mix提供的前端资源打包非常方便，但默认的配置更适合一个单端的开发。例如一个简单的包含用户端和管理端的项目，我们把用户端称为site，把管理端称为admin。 我们希望用户端和管理端的前端资源打包后分别放到/public/sites和/public/admin目录下，我们可以这样配置Laravel Mix。

``` javascript
// webpack.mix.js

...
mix.js('site.js', '/public/site/js/')
    .scss('site.scss', '/public/site/css/');

mix.js('admin.js', '/public/admin/js/')
    .scss('admin.scss', '/public/admin/css/');
...

```

没什么问题，利用这个配置，webpack可以帮我们正确的打包。不过很快我们可能就觉得有些不那么完美了。用户端要围绕业务设计独特的UE和UI，而管理端强调效率，因此两端使用的前端技术栈可能有所不同，使用的库类和框架也可能不同。