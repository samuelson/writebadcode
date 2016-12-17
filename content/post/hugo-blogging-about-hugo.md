+++
date = "2016-12-17T10:53:02-08:00"
draft = true
title = "Hugo, github, lambda, and s3"

+++

When I started this blog, I knew I wanted to use a static blog platform instead of something like wordpress. Last time I did a blog I used octopress which I found to be a little awkwardly heavyweight for a static blog. I think that the future of the internet will be moving away from dynamic content generation the way wordpress does it. Why in the world should I have to run a database server just to have a blog? Honestly, why should I have to run a server at all? S3 will let you serve a static site out of a bucket it's really easy and it's super cheap, like a few cents a month cheap.

The other major thing that I see changing is that a lot of the burden of computation for fancy sites is moving toward the frontend. That is, you can build an entire dynamic application where the backend is just a REST api and all of the stuff that would be handled in the backend on something like Rails is moved into a Javascript frontend. Add to that you can cache the JS framework source files in the end users browser from a CDN and it means you can have a fast and powerful site where all of the "work" is actually done on the end user's computer. That's a project I want to work on someday developing a blog platform where the backend is just undrendered markdown in a public S3 bucket, or something similar, but this is about the current plan for this blog.

I want to set up a publishing chain where I can write and update the site easily. That involves github, AWS lambda, and S3. As of writing this, I haven't figured out how it works or set it up. Since the theme of this blog is about writing bad code and figuring things out, I'm going to try to document the process of figuring out how to do this, meta I know.

## Hugo

One part that I've already figured out is the blog platform I wanted to use. Since I want to learn `go` and I like the simplicity of it, I'm using [hugo](https://gohugo.io/). I won't go into all the setup steps since their docs were pretty easy to follow. I'm starting with the [beautifulhugo theme](http://themes.gohugo.io/beautifulhugo/), but I'll probably change it at some point.

## Github

I started writing this in dropbox, but I'm going to move it to a github repo. I _think_ that will make it simpler to trigger lambda to load it to s3. I'm going to stop writing for a minute in order to turn this into a git repo and publish it to github.

