---
layout: post
title:  "代码中的单词"
date:   2017-04-14 09:30:56 +0800
categories: 开发
tags: [coding, english]
---

#### bail

``` javascript
Tapable.prototype.applyPluginsBailResult = function applyPluginsBailResult(name) {
	if(!this._plugins[name]) return;
	var args = Array.prototype.slice.call(arguments, 1);
	var plugins = this._plugins[name];
	for(var i = 0; i < plugins.length; i++) {
		var result = plugins[i].apply(this, args);
		if(typeof result !== "undefined") {
			return result;
		}
	}
};

```

``` php
$this->validate($request, [
    'title' => 'bail|required|unique:posts|max:255',
    'body' => 'required',
]);
```
