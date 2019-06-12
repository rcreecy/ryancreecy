---
layout: post
title:  "Modifying your Disqus theme, a comprehensive CSS guide"
date:   2019-06-11 21:49:36 -0500
tags: web-dev tutorial
---
### Disqus is an awesome tool.
They have brought good functionality for people like me, who utilize Static Site Generators such as Jekyll to deliver content that is responsive and secure, with the ability to add feedback and comments by offloading and dealing with all the database BS themselves. With that being said.. there is always room for improvement. We're going to work on that. The biggest drawback I see from a plugin or tool such as this, is a lack of customization for the people that choose to get into the thick of it. Disqus leaves us a little limited in that effect.
<!--more-->
We're going to review the styles they use, and see if we can't override it and make some changes.

First, let's go over some pre-reqs. Disqus has a few [rules](https://help.disqus.com/installation/disqus-appearance-tweaks) when it comes to changing their appearance. Understandably so, it's a free plugin, and they have to make their money somehow. Thusly, the rules mostly apply to their brand visibility. We won't be touching that, and I encourage you not too either.

Per their website..

#### Elements that can't be edited or removed
* Social login buttons (Facebook, Twitter, Google, Disqus)
* Community tab (your site name at the top of the embed)
* Font size
* Voting, sharing, and favorite star buttons
* "Add Disqus to your site" text
* "# Comments" text above the embed (For home page comment counts, see Adding comment count links to your home page)
* "Join the Discussion" text

#### With that out of the way, let's take a look at what we *CAN* change.
This is our Disqus theme, set to "dark" from their site's config.

[Image needs to go here]

The first step is cracking open the inspector tool on your compiled and served website. Simply analyzing the `<head>` tag should point you straight to the CSS they utilize. This is crucial in determining where we need to place our CSS to override in the head tag, as well as having it available to compare and contrast with where the changes need to go.

CSS Specificity is the thing to keep in mind here. [w3schools](https://www.w3schools.com/css/css_specificity.asp) has a good, straightforward write-up of how this works. Essentially, if you can target a more accurate tag or component than an applied style, it will take precedence.

For example, the following CSS selector applied to the body would change the background color of the page to blue.
```css
body {
    background-color: blue;
}
```
However, if we apply a further styling to a `class` within that body, that class will have a background color of red.
```css
body {
    background-color: blue;
}
.more-specific-body {
    background-color: red;
}
```
That's the pretty high level explanation of it, but the heirarchy goes, from highest specificity to lowest, Inline Styles (applied at the TAG element within your HTML) -> ID's (# selectors), Classes (. selectors), and finally elements (core html tags such as body).

The goal here is to utilize specificity when we can, and if worst comes to worst (and I mean, bad practice all your friends will make fun of you for), we can always resort to the `!important` tag to really tell the browser what we want.

By now, you should have found your disqus CSS style sheet using the inspector tool. You can crack this open by alt-clicking the link from inspector. Copy and paste that into your editor (VScode is what I use).
But wait! You probably don't want to read that mess. Min-css is designed to help speed up load times, but at the high cost of readability.

[The internet has a tool for that.](https://unminify.com/)

With that out of the way, let's start to inspect some of the elements related to our in-place Disqus comments.
Firefox's inspector tool is handy for this, because it will outline the grid of the elements you are hovering over within the source code for the site.

The first think I want to try to do is get rid of the rounded corners, so let's find the class controls that.
Using the inspector tool, we can see that the whole input box is surrounded by a `<DIV>` with the class `.textarea-wrapper`, so thats a good place to look.
```css
.textarea-wrapper {
    background: #fff;
    border: 2px solid #dbdfe4;
    position: relative;
    border-radius: 4px;
    margin: 0 0 0 48px;
}
```
And sure enough, we have a `border-radius` attribute at the ready.
In our own style sheet then, all we need to do is define the same class, with the modifications we would like to make. In this case, I want a more simplistic approach.
```css
.textarea-wrapper {
    border-radius: 0px;
    border: 1px solid white;
}
```
Now, we can already see that we've made a direct impact on the appearance of the field.