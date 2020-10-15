---
title: TITLE
# Use command in `get_post_timestamp.md` to pull in post date format
# (including offset, which will change)
date: YYYY-MM-DD HH:MM:SS +/-TTTT
# categories contains up to two strings. First is the top category, the other the sub-category.
categories: [TOP_CATEGORIES, SUB_CATEGORIES]
# multiple tags can be applied to each post.
# TAG names should always be lowercase
tags: [TAG]

### Optional ###
## Whether to show a table of contents on right side of a post (default true)
# toc: false

## Whether to include comments (default true)
# comments: false

## Whether to use math notations (default false)
# math: true

## Path to an image to use as the preview image for this post.
# image: assets/img/blah.png

# Whether to pin this post (default false)
# pin: true
---

*Your content here*

```python
def code_snippet():
    """Code as usual for MD."""
```

Refer to `write-a-new-post.md` for more details from the theme authors.

## Footnotes

Footnote syntax is similar to link ref syntax in Markdown. However, there is no need to wrap the text being linked, and the footnote in `[]` uses `^` before the link name.

Example[^1]

Footnote definitions go at the end of the file, just link link refs.

They can also have text names[^something].

[^1]: Footnote 1 source.
[^something]: Footnote "something" source
