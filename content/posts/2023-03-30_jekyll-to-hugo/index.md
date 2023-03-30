---
title: "Migrating from Jekyll to Hugo"
date: 2023-03-30
draft: false
---

A static site generator is a tool that generates static HTML pages from templates and content files. Unlike dynamic websites, which generate content on the server-side in response to requests, static sites pre-generate all their content before deployment. This means that the pages are already generated and ready to be served to users without the need for server-side processing.

Static site generators are often used for building simple websites, blogs, or documentation sites, as they provide a fast and secure way to serve content with minimal server resources. They typically use a templating language, which allows users to create reusable page templates and layouts. Content is usually stored in separate files, such as Markdown or JSON files, which are then processed by the generator to create the final HTML pages.

Popular static site generators include [Jekyll](https://jekyllrb.com/) and [Hugo](https://gohugo.io/). Back in 2019, I created my website with the help of Jekyll. Although Hugo was already available as an alternative, Jekyll seemed to be the standard which is why opted for this framework (I guess another reason was that I directly found some themes that I liked :beaming_face_with_smiling_eyes:).

I had only basic knowledge of HTML and CSS, and zero knowledge of Ruby (which Jekyll is written in) but if you don't plan to make big changes to the provided templates, static site generators require pretty much only  text-based input. 

I wrote several posts serving as a portfolio but then abondoned the project. Some days ago, my interest in writing content arose again, and also my interest in trying out Hugo. So I took on the project of migrating my old Jekyll-based site to Hugo. This seems to be a pretty standard thing to do nowadays (there are lots of posts explaining how to do this, e.g. [here](https://chenhuijing.com/blog/migrating-from-jekyll-to-hugo/#%F0%9F%87%B2%F0%9F%87%BE) and [here](https://blog.arkey.fr/2020/04/20/migrating-from-jekyll-to-hugo-deploying-with-github-pages/)).

As a preparation, I consumed some [YouTube Tutorials](https://www.youtube.com/watch?v=qtIqKaDlqXo&list=PLLAZ4kZ9dFpOnyRlyS-liKL5ReHDcj4G3) and then went ahead, took an existing theme ([Congo](https://jpanther.github.io/congo/)), adapted it to my preferences (like moving the appearance switcher to the upper right corner, reformatting the footer, simplifying the homepage, ...). Adapting the theme was the most time-consuming part and I am still not very satisfied and I will probably change the theme again, but for a start it is alright. I mean, I have a dark mode now! :smiling_face_with_sunglasses: In the end, I migrated the content by copying it manually - it was only a couple of posts, so I did not really have the need to automate it. All in all, it took me one afternoon to do so and was pretty smooth.