TODOS:

- Set up github actions to deploy to github pages
- What is robots.txt
- Fixup base website name
- Lookup copyright icon/what even is it
- Syntax highlighting (went with visual-studio-dark for now)

* Variables for colors
* Write about page
* Write 404 page
* Handle long titles / many tags

===

Page parameters: (slug is filename)

title = "Title can go here."
date = 2022-08-12
[taxonomies]
tags = ["Rust", "PureScript"]

====

Section Parameters: (slug is directory name)

title = "Title can go here, optional."
description = "Description can go here, optional."
sort_by = "date" | "weight"
template = "directory.html"
page_template = "entry.html"
