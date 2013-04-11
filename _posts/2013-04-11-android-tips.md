---
layout: post
title: "android tips"
description: ""
category: android开发
tags: [Android]
---
{% include JB/setup %}
Android Tips
=============
Android Layout
-------------

两个元素同时居中
<RelativeLayout
	android:gravity="center"
	.../>
	<View android:id="@+id/view1"
	    android:layout_centerInParent="true"
	.../>
	<View android:id="@+id/view2"
	    android:layout_centerInParent="true"
	.../>
<RelativeLayout/>
