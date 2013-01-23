---
layout: post
title: Also, .is-splash kills webkit-playsinline, i.e. flowplayer 5.3.1 does
---
Still not entirely sure what actually breaks the webkit-playsinline, but having flowplayer with a class "is-splash" is a big NONO.

Now the newest flowplayer (5.3.1) introduced a support.firstframe concept, which is solely defined by the browser not being [Iphone, Ipad et al, i.e.](https://github.com/flowplayer/flowplayer/blob/master/lib/ext/support.js#L28)

<pre class="code">
    firstframe: !IS_IPHONE && !IS_IPAD && !IS_ANDROID && !IS_SILK
</pre>

That together with [the goodie below](https://github.com/flowplayer/flowplayer/blob/master/lib/flowplayer.js#L322)

<pre class="code">
   // splash
   if (conf.splash || root.hasClass("is-splash") || !flowplayer.support.firstframe) {
      api.splash = conf.splash = conf.autoplay = true;
      root.addClass("is-splash");
      videoTag.attr("preload", "none");
   }
</pre>

makes iPhones behave as splash always, which in a WebView will drive you insane. It did me in anyway. However for now, it plays along nicely and displays all HTML controls.