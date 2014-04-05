---
layout: post
title: "Twitter Bootstrap and JSF"
date: 2012-12-02 00:37
comments: true
categories: 
---
<p> I was amazed by the magic that Twitter Bootstrap can bring into one application. If you are not a front end guy and want to develop one nice looking application, then Twitter Bootstrap is the way to go. The way it manges to arrange the layout and its basic components has always surprised me.</p>
<p>I decided to give a try, to use Twitter Bootstrap on a JSF implementation. Of course there was no doubt about which implementation to use. With a rich set of components <a href="http://primefaces.org/">Primefaces</a> is one of my all time favorite.</p>

<p>The code has been hosted on Github. One can checkout <a href="https://github.com/shasts/jsf-twitter-bootstrap">here</a>.</p>

<p>The screen-shot of the application can be seen <a href="https://dl.dropbox.com/u/77538501/Screenshot%20from%202012-12-02%2000%3A52%3A32.png">here</a></p>

<h5>Lessons Learned</h5>
1. The layout classes can be very well integrated to JSF.
2. Some of the component styling would behave weird when integrated with Primefaces.
3. Primefaces has its own twitter-bootstrap theme. If one would like to use the default bootstrap colors and not the Layout and other basic components then this is the best option.
4. Don't use your own jquery version. Primefaces has one.
