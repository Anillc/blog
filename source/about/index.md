---
title: 关于
date: 2020-12-16 01:39:33
---

<link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/gitalk@1/dist/gitalk.css">
<script src="https://cdn.jsdelivr.net/npm/gitalk@1/dist/gitalk.min.js"></script>

<div id="comment"></div>

<script>
var gitalk = new Gitalk({
  clientID: 'd5b1758ee907a7f17696',
  clientSecret: 'c316109f7e673712d3e785a7fffb33a64872b6ca',
  repo: 'blog',
  owner: 'Anillc',
  admin: ['Anillc'],
  title: 'About Comments',
  id: 'about-comments'
})

gitalk.render('comment')
</script>

<link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/aplayer/dist/APlayer.min.css">
<script src="https://cdn.jsdelivr.net/npm/aplayer/dist/APlayer.min.js"></script>
<script src="https://cdn.jsdelivr.net/npm/meting@2/dist/Meting.min.js"></script>

<meting-js
	server="netease"
	type="playlist"
	id="607264991"
  fixed="true">
</meting-js>
