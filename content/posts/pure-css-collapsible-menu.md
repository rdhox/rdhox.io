+++ 
draft = false
date = 2018-07-12T23:37:02+02:00
title = "CSS Click event cheat: a collapsible mobile menu without any JS"
slug = "collapsible-menu-with-just-css" 
tags = ["front-end", "css", "collapsible", "menu", "webdesign"]
categories = ["front-end"]
+++

Sometimes it's good to return to the basics. With this post, no fancy new web technologies, just pure css and some html. I just discovered a nice css cheat, that maybe everybody knows for years except me, that I wanted to share with you.  
By hiding a html checkbox input and by playing with its css selector ':checked', you can have a click event and affect what's nested in the input's label. This way, you can for example, add a collapsible menu for mobile without any help of CSS frameworks or JavaScript.  


Here a quick example:

<iframe src="https://codesandbox.io/embed/lpmw53kr6m?fontsize=8" style="width:100%; height:500px; border:0; border-radius: 4px; overflow:hidden;" sandbox="allow-modals allow-forms allow-popups allow-scripts allow-same-origin"></iframe>

You can aslo clone and test the code on github [here](https://github.com/rdhox/pure-css-collapsible-menu).

