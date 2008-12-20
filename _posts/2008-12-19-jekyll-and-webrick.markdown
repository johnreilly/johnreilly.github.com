---
title: Jekyll and Webrick
layout: post
---
<span class="post_date">{{ page.date | date_to_string }}</span>
Jekyll is pretty rad. But don't take my word for it. [Check it out](http://github.com/mojombo/jekyll) for yourself. It's also what powers the [GitHub Pages](http://github.com/blog/272-github-pages) feature which was rolled out a day or so ago.  Being a GitHub fanboy, I of course wanted in on this as soon as possible.

So, I created this site, and in doing so, learned how Jekyll does its thing. As I started to build the site, I discovered that site that Jekyll generates was somewhat difficult to preview locally. Here's a stylesheet I was including in my main layout (the layout that applies to all pages in the site):

{% highlight html %}
<link rel="stylesheet" href="/css/style.css" 
      type="text/css" media="screen" charset="utf-8"/>
{% endhighlight %}

In particular notice the <code>href</code> attribute; it's referencing a css file relative to the site root. 

When any page is requested on the server, it will look for the style.css file here: <code>http://johnreilly.github.com/css/style.css</code>. Doesn't matter where on the server the page is, it will always trace back to the root, then look in the css directory, etc.  Perfect.

But this didn't work out so well when I was previewing the generated site on my local computer.  Since I didn't have any sort of webserver configured to look at the site directory, I was just opening the generated files directly in a browser. Something along the lines of <code>file:///Users/john/src/site/_site/index.html</code>.

Problem. Since the stylesheet link was looking from the root of the server, it couldn't find any stylesheets when I was using file://. This becomes particularly troublesome once you start nesting pages in directories, as Jekyll does for blog posts.  I suppose I could use absolute paths to assets already uploaded to the server, but that's no help when you're building or changing the css locally. 

So, long story short, to preview the generated site correctly, it really needed to be served up via a real web server.

As I considered configuring an apache virtual host to serve up the directory, I started thinking, wouldn't it be easier if Jekyll could do this for me? I'd much rather have it fire up a web server pointed to the generated site directory whenever I ask for it.

Enter the greatness that is GitHub. After a [fork](http://github.com/johnreilly/jekyll) and a [commit](http://github.com/johnreilly/jekyll/commit/9ecbfb2253e11f5cb0364e24d4f13595efdd20b6) or [two](http://github.com/johnreilly/jekyll/commit/6bfaa6bac05cf734b9cb9d850998f2f4a0a8b987), I had my own copy of Jekyll that happily launches a [WEBrick](http://www.ruby-doc.org/stdlib/libdoc/webrick/rdoc/) server whenever you pass it the <code>--server</code> option.

The whole thing really shines when combined with the <code>--auto</code> option:

{% highlight bash %}
$ jekyll --auto --server              
[2008-12-19 18:58:43] INFO  WEBrick 1.3.1
[2008-12-19 18:58:43] INFO  ruby 1.8.6 (2008-03-03) [universal-darwin9.0]
[2008-12-19 18:58:43] INFO  WEBrick::HTTPServer#start: pid=9707 port=4000
Auto-regenerating enabled: . -> ./_site
[2008-12-19 18:58:43] regeneration: 12 files changed
{% endhighlight %}

As you can see, Jekyll is monitoring for and regenerating the changes as you make them, and at the same time, WEBrick is purring along serving up your site.  Load up <code>http://localhost:4000/</code> to see your site.  The turnaround time is quick, feels a lot like rails.

Find my fork of Jekyll [here](http://github.com/johnreilly/jekyll).

**Caveat**: Right now, it's hard coded to run on port 4000, simply because I was too lazy to figure out how to pass in a <code>port</code> option. Maybe I'll change that someday, maybe someone else will.  For now, it's working great for me.

**Why WEBrick?** I didn't feel like including mongrel as a gem dependency.  Everyone's got WEBrick installed, and there's no need for this to be a speed demon, so why not pick the easiest option?
