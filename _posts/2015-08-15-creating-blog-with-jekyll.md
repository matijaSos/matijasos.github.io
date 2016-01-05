---
layout: post
title: GitHub Pages + Jekyll + Poole = blog
---

I finally decided to create my personal blog. Being a happy Git(Hub) user, I really liked the idea behind [Jekyll](http://jekyllrb.com) -
have blog as a set of static files that are managed through a git repo!

Another great thing is that such blog can be very easily hosted with [Github Pages](pages.github.com) service that supports Jekyll out-of-the-box.

And at the end there is [Poole](http://getpoole.com/) - minimal foundational setup for any Jekyll site. It's actually Jekyll + some extra styles,
layouts and examples. You'd be totally fine going with vanilla Jekyll install too, but it can give just a bit more to get you up to speed.  
Also, Poole's style is based on [rems](https://github.com/poole/poole#rems-font-size-and-scaling) instead of ems or pixels, so that's one new thing I learned.

My work is actually very closely based on a great [post](http://joshualande.com/jekyll-github-pages-poole/) by [Joshua Lande](http://joshualande.com), so I'll
just refer you there and add here a few extra things that you might find helpful.

* TOC
{:toc}

## How do I initiate my blog with Poole?

I wasn't sure how to include it into my blog, do I have to clone Poole's GitHub repo and then do something?
The answer is very simple - just take these files as they are, copy and push them into your blog's repo. That's it.

If you cloned Poole repo and then copied that, be sure to remove .git files you also got while cloning.
I suppose the most elegant way would be to download ZIP content from GitHub, and unpack it in your target repo.

## Why your font looks bigger than mine?

I was pretty baffled by this one at first, it but turned out that newer Poole versions use different default font size than one
[Joshua](http://joshualande) and I are using. It used to be 20 pixels, and then it was changed to 18 pixels.
I like the bigger font better so I changed it back.

They also forgot to update that change in Poole's README file where it is still stated 20 pixels is used by default, so don't be confused by that.
I issued a [PR](https://github.com/poole/poole/pull/106) for that so they hopefully update it soon.

Another thing I changed back is post's container padding from 1.5**rem** to 1**rem**. It goes better with a bigger font.

## How do I change favicon (icon in a browser's tab)?

Turns out this is not as straightforward as you'd expect it to be. The favicon's location is `public/favicon.ico`.
I replaced it with another, different icon, but just couldn't get Chrome to refresh it, it kept using the old one.

There are several solutions, but this one worked for me - On Mac, delete Chrome's favicon cache with the following command:

{% highlight bash %}
rm ~/Library/Application Support/Google/Chrome/Default/Favicon
{% endhighlight %}

Afterwards restart Chrome and your new icon will get loaded.
Take a look [here](http://www.craiglotter.co.za/2013/09/10/how-to-refresh-site-favicons-in-google-chrome/) 
for the correct path on other operating systems.

If you wonder how to quickly create a favicon from an existing image, I can recommend [favic-o-matic](http://www.favicomatic.com/).

After digging a bit more about the whole favicon thing, it turns out it's a legacy thing not so simple to get it 
[totally](https://github.com/audreyr/favicon-cheat-sheet) right, especially with the arrival of retina and mobile screens.
At the end I decided I'm only concerned with it looking good on retina screen, since initial 16x16 icon I tried out looked blurry.

The solution here is to use 32x32 icon. So I just took the stuff [favic-o-matic](http://www.favicomatic.com/) generated for me, and
replaced this line in `_includes/head.html`:

{% highlight html %}
{% raw %}
<link rel="shortcut icon" href="{{ site.baseurl }}/public/favicon.ico">
{% endraw %}
{% endhighlight %}

with the following:

{% highlight html %}
{% raw %}
<link rel="icon" type="image/png" href="{{ site.baseurl }}/public/favicon-32x32.png" sizes="32x32" />
<link rel="icon" type="image/png" href="{{ site.baseurl }}/public/favicon-16x16.png" sizes="16x16" />
{% endraw %}
{% endhighlight %}

Turns out you could have also just replaced the icon without changing anything in the code, as stated [here](http://daringfireball.net/2013/01/retina_favicons):

> The lazy way to support retina is to replace your old 16 px favicon.ico file with a 32 px file, 
> and allow non-retina browsers to scale the image.

But since I already got both sizes generated, I decided not to be lazy :).

## Adding Disqus comment count

The blog layout proposed by Poole is similar to the one on [codinghorror](http://blog.codinghorror.com/) - a few latest posts can be
seen on the home page in whole, and by clicking on a title you are sent to the specific URL with the selected post only.

I'll refer to those two possible representations of a post as **expanded** and **unexpanded** (contracted somehow doesn't sound 
right, since I'm showing the post's content in whole) states. Expanded is for when a post is shown on its specific URL, while unexpanded
for when it is shown at blog homepage.

## Anchor links for headers

I used [AnchorJS](http://bryanbraun.github.io/anchorjs/) to achieve this.

## VIM not highlighting .md files 

If you're using VIM for file editing and your .md files are not highlighted, the problem is probably you are not using
the newest VIM version (like I wasn't on my Mac) and only .markdown files are highlighted appropriately.

Luckily, that can easily be fixed by adding a single line in your .vimrc file:
{% highlight vim %}
au BufNewFile,BufFilePre,BufRead \*.md set filetype=markdown
{% endhighlight %}

More details can be found on the StackOverflow answer [here](http://stackoverflow.com/a/14779012/1470871).

## Workflow: using `_drafts` vs. branching

Since I just started writing with Jekyll, I don't have an established workflow routine yet.
I see Jekyll is now for some time offering `_drafts` folder to store our work in progress there, and then preview it
with `jekyll serve --drafts`.

That's nice, but I was wondering why couldn't I just make a separate branch for each post, and then merge into master when I'm done?
That seems somewhat cleaner to me (or just more familiar :), so I'll stick with that for now.

Also, imagine if you got a guest to write on your blog. You'd just tell him to make a new branch, write a post there and issue a PR
when he's done. Cool, right?
