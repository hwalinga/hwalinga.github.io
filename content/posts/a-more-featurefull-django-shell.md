---
title: "A More Featurefull Django Shell"
summary: "A More Featurefull Django Shell"
tags: ["django"]
date: 2026-09-19
draft: true
showToc: true
TocOpen: true
lightCode: true
comments: true
---

Instead of the normal `manage.py shell` of Django, the package `django-extensions`,
provides a `shell_plus` that imports all the model classes already for you.
It also selects the most advanced python shell alternative you have installed. 
That is ptpython, bpython, ipython in that order.

What is more, if you for example use factories, you can specify you want those be imported by default as well by putting the following in the settings: 

```py
SHELL_PLUS_SUBCLASSES_IMPORT = ["factory.django.DjangoModelFactory"]
```
