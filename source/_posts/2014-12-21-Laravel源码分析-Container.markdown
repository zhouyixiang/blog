---
layout: post
title:  "Laravel源码分析 - Container"
date:   2014-12-21 13:26:55 +0000
categories: 开发
tags: [php, laravel]
---

作为SOLID基本原则之一，DI也被Laravel采纳，用于解耦框架内大量的组件。关于依赖注入，可以阅读[维基百科上的介绍和示例](https://en.wikipedia.org/wiki/Dependency_injection)，这是理解Laravel框容器的基础，按照wikipedia上的介绍，DI涉及到四个基本组件：

* Service Object
* Client Object depending on the service;
* Interface the client uses to communicate with the service;
* Injector (assembler, provider, container, factory, or spring)

Laravel框架组件管理的基础是DI，本文基于Laravel 4.2的Container（也就是上面的Injector）实现进行分析。 Laravel的Container实现位于src\Illuminate\Container\Container.php文件中，可以看到文件中有很多高频使用的变量命名，包括：

* abstract,
* concrete,
* alias

这些变量名反映了Laravel容器里面的一些基本概念，下面对它们做一些说明。了解这些概念之后，容器的源代码理解起来会简单一些。

# Service

组件对应DI中的Service Object，是向外部提供某种服务的对象。容器负责Service Object的产生，Service Object由Service定义，Laravel框架中有很多提供不同功能的Service例子，如提供认证服务的Service，提供Cache功能的Service，提供Mail功能的Service等等。

这些Service的功能由一些类或者接口定义（例如Auth Service的定义在Illuminate\Auth\AuthManager中），他们被注册后，容器负责其对象的实例化。注册接口由容器的bind函数定义，可以参看后面“Service Object产生”一节。

# Service名 (abstract)

容器里的Service都可以通过名字访问。例如，要产生一个提供认证服务的Service Object，可以调用Container的make方法，传入字符串’auth’，让容器给我们返回一个认证组件的实例。

``` php
$authServiceObject = app()->make(‘auth’)
```

Laravel框架本身提供了很多Service，Laravel对这些Service进行了命名，例如: ‘auth’, ‘cache’, ‘config’, ‘cookie’, 等等。

Laravel本身按照模块化的方式对代码进行组织，每个模块都提供了一些Service，可以通过查看各个模块中的ServiceProvider代码看注册了哪些Service，及其对应的Service名。另外，通过src\Illuminate\Foundation\Application.php文件中的registerCoreContainerAliases方法，也可以看到一个service名字列表。 

对于Service命名的原则，通过代码可以看到，Laravel选择的是简洁明了，像’auth’这样的名字。由于Laravel容器还可以根据类型产生Service Object，例如：

``` php
$authServiceObject = app()->make('Illuminate\Auth\AuthManager')
```
因此容器提供了通过多个名字引用Service Object的机制，这就是别名。

# Service别名 (alias)

Laravel中，每个Service可以有多个名字，其中一个被称为abstract名，其余的则称为alias。容器通过abstract名知道如何产生Service Object。对于alias名称，容器首先查找alias对应的abstract，然后找到产生Service Object的方式。 

Container中有一个$aliases字段，包含了alias到abstract名字的映射关系。容器使用者可以通过下面的函数注册别名。

``` php
public function alias($abstract, $alias)
{
    $this->aliases[$alias] = $abstract;
}
```

# Service Object的产生 (concrete)

调用者向容器注册Service的同时提供了产生Service Object的办法。最基本的注册接口如下：

``` php
public function bind($abstract, $concrete = null, $shared = false)
```

其中abstract是Service名字，$concrete指定了Service Object的产生方式，可以是一个Closure对象，string或者null对象，对于后两种，在Container内部最终也会转换成产生Service Object的Closure。 注册之后，容器使用者就可以通过make接口产生Service Object，接口签名如下：

``` php
public function make($abstract, $parameters = array())
```

make函数的参数名字为abstract，其值可以是Service名或者别名，在make函数内部都会转换成为Service名(abstract)，然后找到其对应的Service Object产生方式生成Object。 make函数调用了一个函数getAlias：

``` php
public function make($abstract, $parameters = array())
{
    $abstract = $this->getAlias($abstract);
```

这个getAlias函数命名很让人很容易产生混淆，因为其功能实际上是通过aliases数组查询alias对应的abstract名。

``` php
public function getAlias($abstract)
{
    return isset($this->aliases[$abstract]) ? $this->aliases[$abstract] : $abstract;
}
```