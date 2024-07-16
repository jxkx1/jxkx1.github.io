---
layout: post
title: "My First Post"
subtitle: "Why am I even making one in the first place?"
author: "jxkxl"
header-img: "assets/wallpaper-sea.png"
header-mask: 0.4
multilingual: true
tags:
  - Intro
  - Post
---

<!-- English Version -->
<div class="en post-container">
    {% capture en %}{% include posts/my-first-post.md %}{% endcapture %}
    {{ en | markdownify }}
</div>
