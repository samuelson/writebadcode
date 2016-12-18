+++
date = "2016-12-17T10:53:02-08:00"
draft = true
title = "Hugo, github, lambda, and s3"

+++

When I started this blog, I knew I wanted to use a static blog platform instead of something like wordpress. Last time I did a blog I used octopress which I found to be a little awkwardly heavyweight for a static blog. I think that the future of the internet will be moving away from dynamic content generation the way wordpress does it. Why in the world should I have to run a database server just to have a blog? Honestly, why should I have to run a server at all? S3 will let you serve a static site out of a bucket it's really easy and it's super cheap, like a few cents a month cheap.

The other major thing that I see changing is that a lot of the burden of computation for fancy sites is moving toward the frontend. That is, you can build an entire dynamic application where the backend is just a REST api and all of the stuff that would be handled in the backend on something like Rails is moved into a Javascript frontend. Add to that you can cache the JS framework source files in the end users browser from a CDN and it means you can have a fast and powerful site where all of the "work" is actually done on the end user's computer. That's a project I want to work on someday developing a blog platform where the backend is just undrendered markdown in a public S3 bucket, or something similar, but this is about the current plan for this blog.

I want to set up a publishing chain where I can write and update the site easily. That involves github, AWS lambda, and S3. As of writing this, I haven't figured out how it works or set it up. Since the theme of this blog is about writing bad code and figuring things out, I'm going to try to document the process of figuring out how to do this, meta I know.

This is a running document of the process including the false starts and distractions. It's "Write bad code", not "Only show the finished product". If you want to see the finished product just skip to the end.

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
      console.log("stdout:" + stdout);git static linked binary
      console.log("stderr:" + stdout);
    });
  }
}
</pre>

So there's a snag, well once I cleaned up all my typos I found a real issue, the require('github') doesn't work because npm didn't install the package locally. [This looks like it's promising](https://aws.amazon.com/blogs/compute/nodejs-packages-in-lambda/). Basically I just needed to install the github package with `npm install --prefix=. github`. So that's working now. I'm ready to test triggering a push.

The message got through, and it looks like it should've triggered the exec, but I don't see that in the logs. I'm going to comment out the if statement and try running it again. Well that actually worked! Hugo just worked because it's a staic binary. So now I just need to figure out how to download the code from github. I think I'll take a look at the github module for Node.

### Github

I just realized that I'm not actually using that github module, that was from the code example on the AWS blog, and they were using it because they were handling github issues. I just want to receive the event from SNS and trigger a git clone, which the github module doesn't do. But this [nodegit module](http://www.nodegit.org/) looks like it might work. Actually the very first example is how to clone a repo, so... yeah let's try that.

First, I need to delete the old node_modules directory and install that `nodegit` module locally. That module happens to include some compiled code, so it takes a little while to compile on my old desktop. Ok, not just a little while, it's been at least 10 min.

Yeah... so that didn't work. Not sure what the deal is, but node.js kindof sucks and I don't want to track it down. So in the writebadcode spirit I'm going to drop this idea and try something else. Compiling `hugo` as a single binary worked, maybe I can do the same with `git`. Somebody named "TheHerk" (which is an awesomely obscure Star Trek reference) solved [got this working](http://stackoverflow.com/questions/11570188/how-to-build-git-with-static-linking).

Ok... that also didn't work. I guess I'll take another crack at the native Node.js git implementation. I'm getting an error about module versions not matching, which I think might be because the version of node I have installed is much newer than the one that's installed on AWS. I think I'll move on to the part about uploading files to s3, since I'm going to need to figure that out anyway. Then I'll just write another lambda that will compile all of these modules and upload them to s3. I can then include those in my zip and use that for this part.

### S3 upload

Something I've learned is that if you get too stuck on one part of a project, you should move on to something else rather than thrash at that one thing. So in that sprit, I'm moving on to the s3 upload part. Basically I need to upload the output of my lambda to s3, i.e. to publish my static site once it's compiled. This should be the simplest and most intuitive thing right? You'd think so, but sadly... no.

I think I'm just going to have to dig down and read [the documentation Amazon provides about using lambda with s3](http://docs.aws.amazon.com/lambda/latest/dg/with-s3.html) because it seems like I'm missing something.

One thing I wasn't sure about was how permissions for the bucket worked. The docs say that the permissions are inherited from the IAM used to run the lambda function, so that's actually kindof logical. For now I added a new role with full access to s3 and lambda execute permissions. I'll narrow things down once I've got it working.

First off, I'm going to try a very simple version, I'll just upload a file to my bucket with the text "test".

Here's the code:
<pre>
var AWS = require('aws-sdk');
var s3 = new AWS.S3();

exports.handler = function(event, context) {
  var targetBucket = 'writebadcode.com';

  s3.putObject({
    Bucket: targetBucket,
    Key: "test",
    Body: "test",
  },function(err, data) {
      if (err) console.log(err, err.stack); // an error occurred
      else     console.log(data);           // successful response
  });
}
</pre>

After a little fiddling that worked, for some reason when I ran it without the function argument it said it worked but didn't upload the file. Adding the logging made it actually upload the file.

Now the next step is to loop through the local files and upload them. This is turning out to be a bit more Node.js then I was planning to do, but I guess that's ok.

### Using files in node
The API docs for nodejs weren't immediately clear what method to use for looping through files. Thankfully [I found this on stackexchange](http://stackoverflow.com/questions/32511789/looping-through-files-in-a-folder-node-js), man how did I ever do any programming before the internet?

Now it's starting to get interesting. I tried just looping through the local directory and uploading each file to s3. This worked, but there is a hitch, it doesn't handle directories. What I need it a way of checking if the file is a directory and if it is, create descend into it and upload the files. But what if there are directories in there? I'll need to do the same thing. This sounds like a gnarly problem, but actually with recursion it's trivially simple.

Step one is to break this out into a function that I can call. I need to define the base case, which is the point at which it does something rather than recursing. The base case for this function is if it's just a regular file to upload that file. Thankfully s3 doesn't actually use directories, it just simulates them by naming the file with the full path. So we don't need to create the directories on s3, but upload the objects with the full path.

In pseudocode the function looks like this:
<pre>
function upload_directory ( baseDir )
  for each file in baseDir
    if file is a directory call upload_directory ( baseDir + file )
    else upload file to s3
</pre>

It has to pass in the directory as it recurses otherwise it will just upload all of the files into the same directory.

### Regular Expression
This works, but that little thing about s3 not using directories adds a tricky little catch. If you call the function with '.' for the current directory, it will upload all the files as './filename'. To get around this, I decided to use the javascript string object's replace() method. It looks for a regular expression that matches the './' part of the filename and replaces it with nothing. Actually, I used something a bit more versatile because I want to be able to point the function at the 'public' dir that hugo outputs.

The regex needs to match any non '/' character from the beginning of the line and then the first '/' it encounters. Here's what that looks like: `^[^\/]*\/`. It's a little confusing because of the '\' characters and the '^' character represents two different things. The '\' are just to escape the '/' characters, so that javascript doesn't interpret them as the closing '/' of the regex. The first '^' denotes the start of the line and the '^' inside the brackets reprents "Not", so '[^\/]' means any character that isn't a '/'.

By the way, I didn't just type this out correctly the first time. I used [this awesome website](https://regexper.com/#%5E%5B%5E%5C%2F%5D*%5C%2F) that actually interprets your regexes into a handy little diagram.

### The final upload function
Here's what this all ended up with for that recursive function:
<pre>
function upload_directory( dirName ) {
  fs.readdir( dirName, function( err, files ) {
    if ( err ) {
      console.error( err );
      process.exit( 1 );
    }
    files.forEach( function ( baseFile, index ) {
      var file = dirName + "/" + baseFile;
      console.log( file );
      fs.stat( file , function ( err, fileStats ) {
        if ( fileStats.isDirectory() ) {
          upload_directory ( file );
        } else {
          fs.readFile( file, function ( err, data ) {
            //Remove first part of dirname for s3 key
            keyName = file.replace( /^[^\/]*\//, '' ) 
            s3.putObject({
              Bucket: targetBucket,
              Key: keyName,
              Body: data,
            },function(err, resp) {
                if (err) console.log(err, err.stack);
                else     console.log(resp);
            });
          });
        }
      });
    });
  });
}
</pre>

### Back to git
Now that the upload part is sorted out, lets get back to the git download part. I generally find that stepping away from a difficult problem and working on something else for a while usually makes the thing I was stuck on much easier. It's especially true in a project like this where I'm using a language that I'm not entirely comfortable in.

I was trying to figure out a way to get the right version of `nodegit` installed, since the version of node on lambda is way older than the one that archlinux installed. I realized it was crazy to try to do the build in lambda and I should just figure out how to have multiple versions of node installed. So I searched for that and discovered a tool called `nvm` which seems to pretty much be the same as `rvm` for ruby. It manages the version of node and npm and lets you easily install other version. I installed it, ran `nvm install 4.3.2` (which I'd figured out was the version of node that lambda uses) and it worked! Then I was able to use `npm install nodegit` and it installed a version that works with lambda.

Now, let's figure out how to get this working. One thing I realized I haven't tried is to put my other code within the callback for the clone. It wouldn't seem like it would matter since the clone should be writing to the local filesystem, but let's see if it works.
