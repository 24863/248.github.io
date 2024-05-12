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

{{ "/assets/style.css" | relative_url }}

/my-baseurl/assets/style.css


Absolute URL

Prepend url and baseurl values to the input to convert a URL path to an absolute URL.

{{ "/assets/style.css" | absolute_url }}



Inspect

Convert an object into its String representation for debugging.

{{ some_var | inspect }}


Date to XML Schema

Convert a Date into XML Schema (ISO 8601) format.

{{ site.time | date_to_xmlschema }}

2008-11-07T13:07:54-08:00


Date to RFC-822 Format

Convert a Date into the RFC-822 format used for RSS feeds.

{{ site.time | date_to_rfc822 }}

Mon, 07 Nov 2008 13:07:54 -0800



Date to String

Convert a date to short format.

{{ site.time | date_to_string }}

07 Nov 2008


ate to String in ordinal US style

Format a date to ordinal, US, short format.

{{ site.time | date_to_string: "ordinal", "US" }}

Nov 7th, 2008


Date to Long String

Format a date to long format.

{{ site.time | date_to_long_string }}

07 November 2008


Where

Select all the objects in an array where the key has the given value.

{{ site.members | where:"graduation_year","2014" }}

Where Expression

Select all the objects in an array where the expression is true. 

{{ site.members | where_exp:"item",
"item.graduation_year == 2014" }}

{{ site.members | where_exp:"item",
"item.graduation_year < 2014" }}

{{ site.members | where_exp:"item",
"item.projects contains 'foo'" }}



XML Escape

Escape some text for use in XML.

{{ page.content | xml_escape }}


Markdownify

Convert a Markdown-formatted string into HTML.

{{ page.excerpt | markdownify }}

Converting Sass/SCSS

Convert a Sass- or SCSS-formatted string into CSS.

{{ some_sass | sassify }}

{{ some_scss | scssify }}


Data To JSON

Convert Hash or Array to JSON.

{{ site.data.projects | jsonify }}


Normalize Whitespace

Replace any occurrence of whitespace with a single space.

{{ "a \n b" | normalize_whitespace }}









