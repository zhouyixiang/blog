---
layout: post
title:  "Laravel + Dingo API 遇到一个小坑"
date:   2016-09-25 09:30:56 +0800
categories: 开发
tags: [php, laravel, dingo]
---

使用Laravel和Dingo API包写一个多租户系统，采用了Multi tenants share a single database的方式，在租户相关的表中有租户ID列。实现了一个租户相关的Global Scope，在需要限定租户的地方进行调用。

在API Controller中，如果大多数接口方法都要使用这个Global Scope，我就把它在constructor中调用了，能够省事，不过这里有个潜在的问题：

Laravel在构造路由的时候会创建一遍Controller，因此会添加上这些Global Scope。本来在API Controller方法中，我只希望该Controller按照构造函数中的方式添加Global Scope，但实际上其它API Controller中的constructor也被调用过了，因此可能在Eloquent中添加了设想之外的全局查询约束。

这个问题在执行Console命令的时候同样存在。虽然直观感觉上，Controller中的代码不应该影响到Console命令，但根据下面的Stack trace可以看出，执行Console命令在Framework Bootstrap时Dingo会构造Controller实例搜集路由配置，这个...

```
#0 example/app/Modules/Course/Api/Controllers/CourseController.php(40): App\\Models\\HospitalScope->__construct(NULL)
#1 [internal function]: App\\Modules\\Course\\Api\\Controllers\\CourseController->__construct()
#2 example/vendor/laravel/framework/src/Illuminate/Container/Container.php(779): ReflectionClass->newInstanceArgs(Array)
#3 example/vendor/laravel/framework/src/Illuminate/Container/Container.php(629): Illuminate\\Container\\Container->build('App\\\\Modules\\\\Cou...', Array)
#4 example/vendor/laravel/framework/src/Illuminate/Foundation/Application.php(709): Illuminate\\Container\\Container->make('App\\\\Modules\\\\Cou...', Array)
#5 example/vendor/dingo/api/src/Routing/Route.php(318): Illuminate\\Foundation\\Application->make('App\\\\Modules\\\\Cou...')
#6 example/vendor/dingo/api/src/Routing/Route.php(180): Dingo\\Api\\Routing\\Route->makeControllerInstance()
#7 example/vendor/dingo/api/src/Routing/Route.php(163): Dingo\\Api\\Routing\\Route->mergeControllerProperties()
#8 example/vendor/dingo/api/src/Routing/Route.php(142): Dingo\\Api\\Routing\\Route->setupRouteProperties(Object(Illuminate\\Http\\Request), Object(Illuminate\\Routing\\Route))
#9 example/vendor/dingo/api/src/Routing/Router.php(652): Dingo\\Api\\Routing\\Route->__construct(Object(Dingo\\Api\\Routing\\Adapter\\Laravel), Object(Illuminate\\Foundation\\Application), Object(Illuminate\\Http\\Request), Object(Illuminate\\Routing\\Route))
#10 example/vendor/dingo/api/src/Routing/Router.php(714): Dingo\\Api\\Routing\\Router->createRoute(Object(Illuminate\\Routing\\Route))
#11 example/vendor/dingo/api/src/Routing/Router.php(744): Dingo\\Api\\Routing\\Router->getRoutes()
#12 example/bootstrap/cache/routes.php(17): Dingo\\Api\\Routing\\Router->setAdapterRoutes(Array)
#13 example/vendor/laravel/framework/src/Illuminate/Foundation/Support/Providers/RouteServiceProvider.php(59): require('/var/www/vhosts...')
#14 [internal function]: Illuminate\\Foundation\\Support\\Providers\\RouteServiceProvider->Illuminate\\Foundation\\Support\\Providers\\{closure}(Object(Illuminate\\Foundation\\Application))
#15 example/vendor/laravel/framework/src/Illuminate/Foundation/Application.php(808): call_user_func(Object(Closure), Object(Illuminate\\Foundation\\Application))
#16 example/vendor/laravel/framework/src/Illuminate/Foundation/Application.php(757): Illuminate\\Foundation\\Application->fireAppCallbacks(Array)
#17 example/vendor/laravel/framework/src/Illuminate/Foundation/Bootstrap/BootProviders.php(17): Illuminate\\Foundation\\Application->boot()
#18 example/vendor/laravel/framework/src/Illuminate/Foundation/Application.php(203): Illuminate\\Foundation\\Bootstrap\\BootProviders->bootstrap(Object(Illuminate\\Foundation\\Application))
#19 example/vendor/laravel/framework/src/Illuminate/Foundation/Console/Kernel.php(262): Illuminate\\Foundation\\Application->bootstrapWith(Array)
#20 example/vendor/laravel/framework/src/Illuminate/Foundation/Console/Kernel.php(114): Illuminate\\Foundation\\Console\\Kernel->bootstrap()
#21 example/artisan(35): Illuminate\\Foundation\\Console\\Kernel->handle(Object(Symfony\\Component\\Console\\Input\\ArgvInput), Object(Symfony\\Component\\Console\\Output\\ConsoleOutput))
#22 {main}"
```
