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

http://example.com/my-baseurl/assets/style.css
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

Date to String in ordinal US style

Format a date to ordinal, US, short format. 3.8.0

{{ site.time | date_to_string: "ordinal", "US" }}

Nov 7th, 2008

Date to Long String

Format a date to long format.

{{ site.time | date_to_long_string }}

07 November 2008

Date to Long String in ordinal UK style

Format a date to ordinal, UK, long format. 3.8.0

{{ site.time | date_to_long_string: "ordinal" }}

7th November 2008

Where

Select all the objects in an array where the key has the given value.

{{ site.members | where:"graduation_year","2014" }}

Where Expression

Select all the objects in an array where the expression is true. 3.2.0

{{ site.members | where_exp:"item",
"item.graduation_year == 2014" }}

{{ site.members | where_exp:"item",
"item.graduation_year < 2014" }}

{{ site.members | where_exp:"item",
"item.projects contains 'foo'" }}

Find

Return the first object in an array for which the queried attribute has the given value or return nil if no item in the array satisfies the given criteria. 4.1.0

{{ site.members | find: "graduation_year", "2014" }}

Find Expression

Return the first object in an array for which the given expression evaluates to true or return nil if no item in the array satisfies the evaluated expression. 4.1.0

{{ site.members | find_exp:"item",
"item.graduation_year == 2014" }}

{{ site.members | find_exp:"item",
"item.graduation_year < 2014" }}

{{ site.members | find_exp:"item",
"item.projects contains 'foo'" }}

Group By

Group an array's items by a given property.

{{ site.members | group_by:"graduation_year" }}

[{"name"=>"2013", "items"=>[...]},
{"name"=>"2014", "items"=>[...]}]

Group By Expression

Group an array's items using a Liquid expression. 3.4.0

{{ site.members | group_by_exp: "item",
"item.graduation_year | truncate: 3, ''" }}

[{"name"=>"201", "items"=>[...]},
{"name"=>"200", "items"=>[...]}]

XML Escape

Escape some text for use in XML.

{{ page.content | xml_escape }}

CGI Escape

CGI escape a string for use in a URL. Replaces any special characters with appropriate %XX replacements. CGI escape normally replaces a space with a plus + sign.

{{ "foo, bar; baz?" | cgi_escape }}

foo%2C+bar%3B+baz%3F

URI Escape

Percent encodes any special characters in a URI. URI escape normally replaces a space with %20. Reserved characters will not be escaped.

{{ "http://foo.com/?q=foo, \bar?" | uri_escape }}

http://foo.com/?q=foo,%20%5Cbar?

Number of Words

Count the number of words in some text.
From v4.1.0, this filter takes an optional argument to control the handling of Chinese-Japanese-Korean (CJK) characters in the input string.
Passing 'cjk' as the argument will count every CJK character detected as one word irrespective of being separated by whitespace.
Passing 'auto' (auto-detect) works similar to 'cjk' but is more performant if the filter is used on a variable string that may or may not contain CJK chars.

{{ "Hello world!" | number_of_words }}

2

{{ "你好hello世界world" | number_of_words }}

1

{{ "你好hello世界world" | number_of_words: "cjk" }}

6

{{ "你好hello世界world" | number_of_words: "auto" }}

6

Array to Sentence

Convert an array into a sentence. Useful for listing tags. Optional argument for connector.

{{ page.tags | array_to_sentence_string }}

foo, bar, and baz

{{ page.tags | array_to_sentence_string: "or" }}

foo, bar, or baz

Markdownify

Convert a Markdown-formatted string into HTML.

{{ page.excerpt | markdownify }}

Smartify

Convert "quotes" into “smart quotes.”

{{ page.title | smartify }}

Converting Sass/SCSS

Convert a Sass- or SCSS-formatted string into CSS.

{{ some_sass | sassify }}

{{ some_scss | scssify }}

Slugify

Convert a string into a lowercase URL "slug". See below for options.

{{ "The _config.yml file" | slugify }}

the-config-yml-file

{{ "The _config.yml file" | slugify: "pretty" }}

the-_config.yml-file

{{ "The _cönfig.yml file" | slugify: "ascii" }}

the-c-nfig-yml-file

{{ "The cönfig.yml file" | slugify: "latin" }}

the-config-yml-file

Data To JSON

Convert Hash or Array to JSON.

{{ site.data.projects | jsonify }}

Normalize Whitespace

Replace any occurrence of whitespace with a single space.

{{ "a \n b" | normalize_whitespace }}

Sort

Sort an array. Optional arguments for hashes 1. property name 2. nils order (first or last).

{{ page.tags | sort }}

{{ site.posts | sort: "author" }}

{{ site.pages | sort: "title", "last" }}

Sample

Pick a random value from an array. Optionally, pick multiple values.

{{ site.pages | sample }}

{{ site.pages | sample: 2 }}

To Integer

Convert a string or boolean to integer.

{{ some_var | to_integer }}

Array Filters

Push, pop, shift, and unshift elements from an Array. These are NON-DESTRUCTIVE, i.e. they do not mutate the array, but rather make a copy and mutate that.

{{ page.tags | push: "Spokane" }}

["Seattle", "Tacoma", "Spokane"]

{{ page.tags | pop }}

["Seattle"]

{{ page.tags | shift }}

["Tacoma"]

{{ page.tags | unshift: "Olympia" }}

["Olympia", "Seattle", "Tacoma"]

Inspect

Convert an object into its String representation for debugging.

{{ some_var | inspect }}
