---
layout: post
title: New Website!
tags:       [personal]

noindex: true
---

# Welcome to my new website!

Like so many people, I've been under the COVID-19 quarantine and desperately searching for ways to cling to sanity. I've only been partially sucessful at that, since 8+ weeks in lockdown when you basicaly live alone isn't just something that can be stubbornly pushed through. But, thankfully, while procrastinating my thesis writing I have been somewhat productive - I built myself a website!

By now, I have quite a bit of experience with programming, but it's all limited to research, which has its own particular styles, requirements, and norms and I have no experience with coding for the internet. So, I decided to dive in and see how successful I could get in setting up a simple, modern looking custom website. I think it's been pretty successful, but you'll have to decide for yourself by taking a look around!

## How I set up the site
Since I have no experience in typical web design stuff like HTML and CSS, and I don't have a server or want to pay for hosting, I needed to look for a very simple system which also enabled simple site hosting. Of course, you can do this through services like Squarespace and Wordpress, and I'd set up a blog on Wordpress at the start of the PhD (which has all been migrated to here), but I really wanted to have a go at it myself and have more control over the site and not worry about ownership of my content. It turns out, the perfect solution for someone like me, a young academic with simple needs and some programming skills, is to host through [Github Pages](https://pages.github.com/). 

I already use Github for managing several of my programming projects and as a version control and cloud storage system for paper writing (I might get into that setup some day), so I'm fairly comfortable with it at this point. Github Pages is super simple to use from a hosting point of view - just create a private repository, setup your directory structures as needed, then activate Pages. If you have a Github account already, it's just like creating a new project repository, which can be cloned and synced with local copies on your computer, but it's all hosted directly from Github. They even give you the option for setting it up with a custom url, although for now the default of using your Github username works for me. 

The more difficult bit is in setting up the directory structure and formatting pages and posts and links, etc. to actually make a proper website. Github has some pretty good step-by-step instructions on doing it from scratch, but I decided I wanted a sightly slicker looking site than what was available that way, so I searched around for some themes and templates for working with Github Pages. The one I settled on was [Hydejack](https://hydejack.com/) a theme set up through Jekyll and designed to be a clean blog style website. I'm not going to get into the setup instructions as their documentation does a pretty good job, but basically I just had to fork the Hydejack repository on Github, set it up as my own Pages repository, and replace the default template pages with my own. There were some challenges and bugs to work through, but all of them stemmed from my own inexperience, and not from Hydejack itself. Even better, Hydejack is completely open-source and free to use for a basic website template, with an upgraded version which includes a nifty resume page that can be exported and printed, as well as some other nice features, for a very reasonable price. It even makes it straightforward to include my own photo for the frontpage and author tag. So that little logo and the beautiful lake photo on the left right now are all mine (courtesy of my 2018 trip to Alaska).

The nicest thing about this system is that everything can be done through markdown. I'm already familiar with markdown from using Rmarkdown and Jupyter notebooks, but even if you're new to it, it's a super simple text formatting system. No HTML tags, no coding-style instructions, just basic text formatting. The majority of the structure of the site comes from some basic folder structures, and the rest is handled with some simple tags and formatting in the page's markdown document header. 

Once everything is setup - which took me only a couple hours, from finding Hydejack to cleaning up the website and publishing this post - adding a new post is super simple. Create a new markdown text document, change the header info, write the post, make sure the filename has the date, then save the blog folder, push to Github and that's it! The new post get's added to the chronological list of posts, added to whatever sub-page I tagged it for, and the template takes care of all of the formatting and prettifying. Even adding photos to posts is pretty simple, just include a link to it either saved in the Github repository or as a url, and let markdown know it's a photo and it get's inserted wherever you asked for it. 

Like I said, I had a few twists to work out at the start, and I'm sure it could be much smoother but for what I wanted to accomplish it's spot on and I'm very happy with it! Hopefully the simplicity of adding new posts will encourage me to keep it more up to date than the Wordpress version did...

For now, enjoy whatever random assortment of posts is up here, and keep an eye out for my future posts and publications!
