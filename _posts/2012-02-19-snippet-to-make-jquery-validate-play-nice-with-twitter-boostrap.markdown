---
layout: post
title: Snippet to make jquery validate play nice with twitter boostrap
---

[Twitter bootstrap](http://twitter.github.com/bootstrap/index.html) is growing on me, I dont have to like the fact that [less](http://lesscss.org/) 1.2.1 (current as of today) refuses to compile bootstrap in real time for some import error (charAt errors from less are really unhelpful here). However since bootstrap compiles very fine via node, I can live with it, since I am just touching up some base colors.

Making jQuery validate play nice is not immediatly obvious, and I've not found a great snippet for that, so here is mine.

<script src="https://gist.github.com/1864941.js"> </script>

Sometimes you want to set <code>onfocusout: false</code>, especially when external actions are required and revalidation should happen server side rather than just by entering garbage. This has served me well in any case.