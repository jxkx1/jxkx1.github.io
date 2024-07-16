---
layout: post
title: "My First Post"
subtitle: "Why am I even making one in the first place?"
author: "jxkxl"
header-style: text
tags:
  - Intro
  - Post
---

<!-- English Version -->
<div class="en post-container">
    {% capture en %}{% include posts/my-first-post.md %}{% endcapture %}
    {{ en | markdownify }}
</div>
