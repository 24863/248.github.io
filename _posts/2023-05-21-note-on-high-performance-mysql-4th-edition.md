---
title: "课堂练习"
layout: post
category: note
tags: [Liquid Filters]

excerpt: "All of the standard Liquid filters are supported (see below).

To make common tasks easier, Jekyll even adds a few handy filters of its own, all of which you can find on this page. You can also create your own filters using plugins."
---



# Liquid Filters

Relative URL

Prepend baseurl config value to the input to convert a URL path into a relative URL. This is recommended for a site that is hosted on a subpath of a domain.
、、、
{{ "/assets/style.css" | relative_url }}

/my-baseurl/assets/style.css
、、、
