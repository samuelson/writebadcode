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

I started writing this in dropbox, but I'm going to move it to a github repo. I _think_ that will make it simpler to trigger lambda to load it to s3. I'm going to stop writing for a minute in order to turn this into a git repo and publish it to github. Ok that's done now. It's in [this public repo](https://github.com/samuelson/writebadcode). Since hugo lets you set if a file is a draft or not, it should be ok to just always push the live version. Then I can just undraft to release.

### Webhooks

Github has the great feature called `webhooks` that let you trigger a POST request to a URL depending on different events on your repo. For example you could trigger it every time you push something to the repo, or when you initiate a release in github, or lots of other things. I'm going to be using the push event to trigger AWS lambda to upload the content from github to s3.  Since I haven't set up those other parts, I'm going to move on for now. One other note, I know that I could just create a local git hook to render the site and upload it to s3, but part of the point of this is learning new things. Also, if I set it up this way, I could theoretically use editor on the github website itself to write new posts.

## Lambda

This is the part where I know the least, I've poked around at lambda but only to do "hello world" type examples. Lambda is a fairly new service provided by S3 that lets you run arbitrary code without running an entire server. It's perfect for this use case. I just want to sync my code and run hugo when I push to github, I don't need to maintain a server just for that. My general rule of them when I'm starting anything new is that I should google it first since there is a good chance that someone else has struggled throug this already. I searched for "aws lambda github s3" and [came up with this](https://github.com/nytlabs/github-s3-deploy). That looks pretty promising, at least for the uploading to s3 part. It doesn't run hugo, so I'll need to figure that part out. Unfortunately, [it's not open source.](https://github.com/nytlabs/github-s3-deploy/issues/5) WTF, thanks for nothing New York Times. It does however link to [this blog and Amazon's own site](https://aws.amazon.com/blogs/compute/dynamic-github-actions-with-aws-lambda/), which may be better for me anyway.

### SNS topic
The first step looks to be creating something called an SNS topic. According to the doc "Amazon SNS helping out by transmitting events between the two systems". It stands for "Simple Notification Service" and I guess it's a sort of translation layer in this project.

Oh, I'm literally following their instructions and I get to the part to create the IAM policy to give my new user access and it says the "Version string" isn't valid. I copied it from the AWS docs, WTF. This has happened to me before and I always have to thrash around to figure out the solution, so hopefully if you run into it you'll benefit from my pain. 

BAH! Ok, I finally figured it out. The version is supposed to be this exact string "2012-10-17" because that's the date they last changed the IAM syntax or something. That's terrible, why couldn't they just say that in the error message. I'm pretty sure I didn't figure this out last time, I just changed it back to what I'd copied and accepted that somehow it worked.

Moving right along, the rest seemed pretty straightforward, although I am questioning why I need to use SNS at all. I guess that adds some security so that random people can't just trigger the lambda and run up my AWS bill?

### Lamda Function

Despite being a bit out of date the AWS guide is so far pretty good. I'm following along. It looks like it all works. I can have github send an SNS message to aws, which run the lambda. Woot! They've got some nice example code for only acting when issues are created. I only want it to do anything if the github event is a 'push' so I'm changing their code to this:

<pre>
if (githubEvent.hasOwnProperty('Pusher') {
  //code goes here
}
</pre>

Yes, that's javascript, more on that later. Since this is just a tiny bit of code, and I've actually made an effort to learn Node.js in the past, I'm going to stick with it.

Now were getting to the fun part, I've gotta figure out how to run hugo from lambda. But there is a hitch. Hugo is written in go, but lambda only supports C#, Java, NodeJS, and Python. So there are a couple of interesting points of reference. First, there is [this blog post and associated github repo](https://rsb.io/posts/overview-of-hugo-lambda/) which also references [this project to run shell scripts within lambda](https://github.com/alestic/lambdash). That second one looks like the kind of thing I was hoping for in taking on this project. I want to use lambda to do some non-trivial things at work, and being able to run shell scripts (and potentnially ruby) opens up some new options.

The hugo-lambda project looks good, but I see depdendencies for both node.js and python, which make me think it might be an unholy beast. I'm going to first try to use it as a reference for writing something in pure node.js. Also, it doesn't quite do what I want, I need the function to grab the content from git and run hugo when triggered, not go from one bucket to another.

### A brief commentary on ripping off other peoples code

The title here is a joke, this is open source, and I don't intend to keep any of the original code. The bigger point is that there is a lot of code out there and it's kindof silly to invent a wheel that someone else has already invented. I think this is especially true for well established libraries, etc. If you're always writing everything from scratch you're probably wasting a lot of time and energy. But, it's also worth looking at someone else's code as a reference for _how_ to do something even if you don't use a single character of their source. This is especially true when you're translating into other languages. Python is easy enough to read, but I have no interest in learning it and no interest in maintaining a project written in Python.

### Adaptation

It looks like Ryan's project uses a command line tool called "kappa" to make working with lambda a bit easier. He also uses CloudFormation, which is more automation than I'm looking for. I'm ok to have this be a one off project.

I actually just need to clone the repo, run hugo, and push the files to s3. I'm going to try something stupid first. Can I just copy the hugo binary from my PC into the zip file and run it? Go is supposed to compile static binaries, so it _should_ work in theory

### Easy mode

I couldn't remember how to run external commands in Node, but thankfully [someone else already asked the question here](http://stackoverflow.com/questions/20643470/execute-a-command-line-binary-with-node-js)

Here's my code to try it out:
<pre>
var GitHubApi = require('github');
var github = new GitHubApi({
  version: '3.0.0'
});

exports.handler = function(event, context) {
  var githubEvent = event.Records[0].Sns.Message
  var exec = require('child_process').exec;

  console.log('Received Github event:', githubEvent);

  if (githubEvent.hasOwnProperty('pusher')) {
    exec('./hugo', function(error, stdout, stderr){
      console.log("stdout:" + stdout);
      console.log("stderr:" + stdout);
    });
  }
}
</pre>

So there's a snag, well once I cleaned up all my typos I found a real issue, the require('github') doesn't work because npm didn't install the package locally. [This looks like it's promising](https://aws.amazon.com/blogs/compute/nodejs-packages-in-lambda/). Basically I just needed to install the github package with `npm install --prefix=. github`. So that's working now. I'm ready to test triggering a push.
