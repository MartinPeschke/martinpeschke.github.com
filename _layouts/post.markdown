---
layout: default
article-class: post
---

<article>
<h2>{{ page.title | xml_escape }}</a></h2>
    <a class="label permalink" href="{{ page.url }}">permalink</a>
    <span class="datetime">
      <time datetime="{{ page.date | date: "%Y-%m-%d" }}">
        {{ page.date | date: "%B %d, %Y" }}
      </time>
    </span>
    <div class="content">{{ page.content }}</div>
</article>

<div id="disqus_thread"></div>
<script type="text/javascript">
    var disqus_shortname = 'berlincodeistan';
    (function() {
        var dsq = document.createElement('script'); dsq.type = 'text/javascript'; dsq.async = true;
        dsq.src = 'http://' + disqus_shortname + '.disqus.com/embed.js';
        (document.getElementsByTagName('head')[0] || document.getElementsByTagName('body')[0]).appendChild(dsq);
    })();
</script>
<noscript>Please enable JavaScript to view the <a href="http://disqus.com/?ref_noscript">comments.</a></noscript>
