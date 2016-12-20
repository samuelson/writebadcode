+++
title = "Premature Optimization"
draft = false
date = "2016-12-19T21:34:55-08:00"

+++

Premature optimization is one habit that often stifles creativity in programming. It's a great example of how writing bad code might aftually be better in the long run. This is especially true of those who have taken programming courses because they've had the idea of efficiency drilled into their heads. It's also easy to miss that it's happening to you, in fact, it might even feel good to be writing such efficient code. I've found that it's often actually easier to improve code efficiency after it's written rather than as you go.

When I'm working on a project I go through a process like this:
1. Identify the problem to be solved and gather requirements.
1. Search the internet to see if someone else has a complete solution to this.
  * If there is something, stop here.
1. Look at programming libraries like rubygems to see if all the parts of it have been solved.
1. Hack together a proof of concept in the language that feels most comfortable. Just get it out, don't focus on efficiency and don't obsess about things like variable names.
1. Use the POC for a while, are there any problems? If not, just revise the code to make sure you'll understand it in a few months by adding clear names, comments, etc.
1. Optimize and refactor if/when there are problems or if you want to share the code and don't want to be embarrassed by it.
1. When you do refactor start with the question, is there something inefficient about the premise of the POC rather than the code?

I think the first two steps are the most important and were the hardest for me to learn. A lot of my education in programming involved reinventing the wheel. We were assigned programming tasks like sorting lists in the most efficient way or even just implementing some sorting algorithm from the textbook like bucket sort. The idea sank in that _real_ programmers wrote everything from scratch and didn't use libraries. The thing is, that's totally false. Real programmers, good ones anyway, only write something themseleves when there is no other option. That's why I always start any task with the question "Has someone already solved this?". Most of the time they have, and even if they haven't solved my exact problem, I can look at their code as a place to start.

I'll leave you with an example of some un-optimized code that I wrote recently. I was editing some audio using [audacity](http://audacity.com) for a video narration at work. The problem was that the two clips were recorded at diferent times and on different audio equipment. The narrator's voice was the same, but it was obvious that they didn't match. What I wanted was a way to auto-eq the new recording to sound more like the old one. The EQ feature in audacity allows you to import EQs in XML format, but there was no way to compare the difference between the two recordings. The analyze feature did let you export a histogram as a txt file of frequency/db pairs. What I needed was a way to find the difference between the histograms and export it to the proper XML format.

Here's the code:
<code>
#! /usr/bin/env ruby

first = {}
second = {}
freq_regex = /^\d+\.\d{6}\s-?\d+\.\d{6}$/

File.open(ARGV[0]).read.gsub(/\r\n?/,"\n").each_line do |line|
  if line =~ freq_regex
    first[line.split(/\s/)[0]] = line.chomp.split(/\s/)[1].to_f
  end
end
File.open(ARGV[1]).read.gsub(/\r\n?/,"\n").each_line do |line|
  if line =~ freq_regex
    second[line.split(/\s/)[0]] = line.chomp.split(/\s/)[1].to_f
  end
end

puts '<equalizationeffect>'
puts '<curve name="' + ARGV[0] + ' ' + ARGV[1] + ' difference">'
first.each do |freq, offset|
  if second[freq]
    puts '<point f="' + freq + '" d="' + (offset - second[freq]).to_s + '"/>'
  end
end
puts '</curve>'
puts '</equalizationeffect>'
</code>

There's a little fiddlyness at the beginning because audacity used windows line endings, but basically it just loads both files into hashes keyed on frequency.  Then it loops through the first and prints out the difference. What it doesn't do is understand XML which is why I'm using it as an example. I'm not embarrassed to say, this is "bad" code. Also, I have no reason to refactor it. The only reason I think I would is if I wanted to create an audacity plugin to do this.

The point is, bad code that works is often better than good code that isn't finished.
