---
layout: page
title: A Portfolio/Blog SPA Part 1 - The Idea
---

Recently I have been working on a static site generator (like jekyll) which will turn your static content into a single page app powered by React and React-Router!  In this post, I will explain the idea behind it, and in future posts, I will dive into the code that makes it happen.

I really like the way static site generators work.  I like having the ability to drop in content, not worry about what is going on under the hood, and having everything just _work_.  I went into the project with a list of things that I wanted to try out.  Here is said list:

1. Static site generator which works like jekyll, allowing the user to ignore what is going on under the hood
2. Single page application, using [React Router](https://github.com/reactjs/react-router)
3. Smooth transitions between views
4. Be consistent with Material Design
5. Server side rendering
6. Hosting on github pages because I am a broke student

Wow, that is a lot of buzzwords! Let's jump into how it is done.

First, lets go over the basic directory structure that the user will interact with.

```
.
├── about.md
├── blog
│   ├── post1.md
│   └── post2.md
├── photography
│   ├── folder1
│   │   ├── image1.jpg
│   │   ├── image2.jpg
│   │   ├── image3.jpg
│   │   └── link.jpg
│   └── folder2
│       ├── image1.jpg
│       ├── image2.jpg
│       ├── image3.jpg
│       └── link.jpg
└── portfolio
    ├── item1.md
    └── item2.md
```

The idea is that the user will drop their content into these folders, run a script, and watch as the website builds and deploys itself.  

Here is what the files mean:

`about.md` represents the about page. Simple enough.

Each post in the `blog` directory represents a blog post.  There will be a page that has an index of all of the blog posts, with links to each individual blog post, just like on my site.

Within `photography`, you notice there are subdirectories.  These represent different groups of photos that should be displayed together.  The `link.jpg` is the link that will be displayed on the index page for the photography section.  It will have an overlay that shows the name of the folder.  Every image other than `link.jpg` will be in a gallery with a lightbox.

Each item in `portfolio` will have the data that will be displayed in the portfolio section.

This is obviously a rough sketch of what this project is going to look like.  In the next post, I will go into the meat of the static site generator: the scripts that parse the files.
