+++ 
draft = true
date = 2018-10-29T18:05:35+01:00
title = "Managing the State of states"
slug = "managing-the-state-of-states" 
tags = ["front-end", "react", "redux", "development", "web", "application"]
categories = ["frontend"]
+++

I wanted to write an article about *state* in **React** since a while now. But there is so many things to say that I didn't know how to organized it. I know that I should make in a notebook a little plan, with the main points of my article to structure my ideas, etc...   
But I do that for my work as a developper, and writting a blog post is more about relaxation, sitting on my couch with the laptop on your lap and the cat snoring beside you. And now with React 16.7, we will have this new (awesome) feature : **React Hooks** ([nice introduction here](https://scotch.io/tutorials/getting-started-with-react-hooks)). So I'm like "*OK, now it's time to write this article, or it will become too complicated after that*". But it's gonna be a little messy.

#### redux?

Managing the states of your application has become a big concern and we can see lot of people very opinionated about how you supposed to do it. Especially with React where the debate over Redux was strong few months ago. I read during this period that lof of developpers where using Redux automatically with React without asking any question. That surprise me because, well React in a sense, is already a formidable Javascript library to manage states in your application. And the fact that React is a library and not a framework is awesome. You don't have any rule to follow, you can do what you want and create app very fast, which is not really the case with Angular. And it's also not really the case if you use Redux. You have to make the good choice, and if you have to do a Todo list for a client (why not?), don't go to the Redux website copying their tutorial! It's absurdly complicated, there is like fifteen files!
Just create one React Component with local states, one file, perfect. If you have the time, extract the logic to pure functions easy to test, make some stateless functional components to display the UI, and it's good. And even if your app is more complicated than that, you should be able to not use Redux until a certain degree. Until you feel the need (the pain?) for it. Why?  
Because you have to learn how to use React alone. It's where all the fun is, it's working very well, and it's always better to understand how things works. And you may find a new way to organize projects that gonna lead to a new workflow that will revolutionize the world of software development, and that gonna make you gain hundreds of thousands of new followers on twitter!

#### react alone









