---
layout: post
title: HTML5 flowplayer, ios UIWebView, allowsInlineMediaPlayback and webkit-playsinline
---
[By now it is well understood](http://blog.millermedeiros.com/unsolved-html5-video-issues-on-ios/) 
that apple has disabled video inline playing on the [iPhone but not on the iPad](http://roblaplaca.com/blog/2010/04/14/ipad-and-iphone-html5-video-autoplay/). 
Now in a WebView you can make it work by both, adding "webkit-playsinline" to the video tag and setting 
[allowsInlineMediaPlayback](http://developer.apple.com/library/ios/#documentation/uikit/reference/UIWebView_Class/Reference/Reference.html) to YES.

However Flowplayer has been worked for best browser experience, so you need to hack a bit around in it to show full screen videos with html5 controls. Essentially I needed to disable native controls for the mobile browsers:

<pre class="prettyprint linenums language-javascript">
    var instances = [],
       extensions = [],
       UA = navigator.userAgent,
       use_native = false,
       // IPHONE - this bypasses fullscreen native ui for iphone app webview
        bypass_native_fs = /Android/.test(UA) || /iP(hone|od)/i.test(UA);
</pre>


Add the webkit-playsinline property to the flowplayer videoTag, since it gets removed from the html video tag () as do all unrecognized attributes):

<pre class="prettyprint linenums language-javascript">
    if (bypass_native_fs) videoTag.attr("webkit-playsinline","on");
</pre>

Now you think you got it? Of course NOT. Above we have told flowplayer to use the actual HTML5 full screen API, which mobile safari supports. Now we need to Tell flowplayer to just fake fullscreen:

<pre class="prettyprint linenums language-javascript">
  if (FS_SUPPORT && !bypass_native_fs) {
</pre>

While we are at it, lets also remove this goodie:

<pre class="prettyprint linenums language-javascript">
      // native fullscreen
      if (player.conf.native_fullscreen && $.browser.webkit) {
         player.fullscreen = function() {
            $('video', root)[0].webkitEnterFullScreen();
         }
      }
</pre>


And now we are nearly there, since of course, you have to set preload="none" on the video tag. The which and why eludes me yet, just do it, or it'll cost you years of your life.

<pre class="prettyprint linenums language-javascript">
  &lt;video preload="none" webkit-playsinline&gt;
      &lt;source src="${url}"/&gt;
  &lt;/video&gt;
</pre>

PS: the webkit-playsonline is gratuitous. I'll leave it there just because. It doesn't do anything anyways.