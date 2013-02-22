---
title: 解决Google经常性连接重置
date: '2013-01-14'
description:
categories: 技术
---
使用Google搜索的时候，结果中可能会有被墙的页面。由于在打开结果页面时Google会使用自己的域作跳转，因此在打开被墙的页面时，Google搜索也会受牵连，导致一段时间不可访问。明显，解决这个问题最简单有效的办法就是让Google不作跳转，直接打开目标页面。  

先前找了很久，想着官方应该是不支持此类设置的，于是自己做了一个脚本。自己用很久了，应该没啥不良反应~  

脚本代码原来只有一行，后来为了兼容Chrome和Firefox加入了动态加载jQuery的相关代码。

	// ==UserScript==
	// @name        Google Jump Remove
	// @namespace   http://www.zhouxiongzhi.net/post/40501975141/resolve-google-connection-reset-by-fucking-gfw
	// @description Remove Google Search Url Jump
	// @include     http://www.google.com.hk/search*
	// @include     https://www.google.com.hk/search*
	// @grant       none
	// @version     1
	// ==/UserScript==
	
	// a function that loads jQuery and calls a callback function when jQuery has finished loading
	function addJQuery(callback) {
		var script = document.createElement("script");
		script.setAttribute("src", "//ajax.googleapis.com/ajax/libs/jquery/1/jquery.min.js");
		script.addEventListener('load', function() {
			var script = document.createElement("script");
			script.textContent = "window.jQ=jQuery.noConflict(true);(" + callback.toString() + ")();";
			document.body.appendChild(script);
		}, false);
		document.body.appendChild(script);
	}
	
	// the guts of this userscript
	function main() {
		// Note, jQ replaces $ to avoid conflicts.
		jQ('li.g>.vsc>h3>a').removeAttr('onmousedown');
	}
	
	// load jQuery and execute the main function
	addJQuery(main);

[这里][Google-Jump-Remove-UserScript]可以下载，也可以自行保存上述代码为Google_Jump_Remove.user.js。chrome下将js文件拖到chrome://extensions/页面安装；firefox下请使用[GreaseMonkey][GreaseMonkey]安装。  

为了更方便安全的使用Google，建议使用chrome的HSTS设置强制使用https访问Google。用chrome访问chrome://net-internals/#hsts，添加Google相关域（www.google.com.hk）到HSTS set。


[Google-Jump-Remove-UserScript]: http://userscripts.org/scripts/show/156455
[GreaseMonkey]: http://www.greasespot.net/