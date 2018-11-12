+++ 
draft = false
date = 2018-11-12T09:54:22+01:00
title = "React-Hooks: The end of forbidden local state"
slug = "react-hooks" 
tags = ["react", "react 16.7", "javascript",]
categories = ["frontend"]
+++

##### The magic of useState()

The developers using React were split into two different ways of thinking:  

* Developer that use React mainly without redux, using setState(), passing props throught components.
* Developer that learned React with redux, and for whom setState() is forbidden. They dispatch everything, to be sure that the global state is the perfect image of the app at an instant t.

And there is people in the middle, where I stand. I learned React as a library not as a way to build entire website or application. For me it was just an awesome way to build my own HTML components with their own logics to make jQuery look like something from the past. Quickly I realized that you can actually build your all website with React, only to see that lot of people where already doing it!  
And this is when you really start to think about how to manage your state. And you often turn your head toward Redux at this point. The subject of managing state is a hot subject in the React community, and everybody has an opinion about it. For me it's all about common sense. Which state I'm using? Is it a state only used for the UI? If making the state local help to the ability of reusing the component and had nothing to do with the logic of the app, so making it local surely make sense. Making it global would make the global state unnecessary heavier. But after you have to transform your component as a class. And lof of people prefer having stateless functional component, and they don't like the idea of having a class just for managing few states. Even if the performance between displaying a class and a sfc is not that different, etc...  
Anyway it's where all the debates take form, and I don't want to debate here. Again for me it's common sense, and also about respecting the choice of the team you are working with.  


And this is where React-Hooks will make people happy.  Basically with hooks, you can use state with stateless components. So you can add some logic to components, without using setState(). Problem solved for the two categories of people mentionned above. So how it's work? Quick example:

```javascript
import React, { useState } from "react";

function CountSheep() {

    const [count, setSheep] = useState(0);

    return(
        <div>
            <div>
                <p>{`I already saw ${count} sheep${count > 1 ? 's' : ''}!`}</p>
            </div>
            <button
                onClick={() => setSheep(count + 1)}
            >
                Add one!
            </button>
        </div>
    );
}
```

You take from useState (using `array destructuring`) two variables. First one is the state you want to use. Second one is the name of the function you will use for manipulating the state. Your setState() function for this particuliar state if you prefer. You maybe not familiar with array destructuring, why not using `object destructuring`? Because with `array destructuring`, you can name easily the variable you extract:

```javascript
const tab = [1, 2, 3];
const [un, deux, trois] = tab;

// un = 1, deux = 2, trois = 3
```
So from that, we understand that useState is an array that has in his first index a variable, the state that we pass the default value as an argument of useState at first, and a function for his second index. Easy, efficient. I like it.

##### Ok but what about the lifecycle of the component? A class has function to manage them!

It's why useEffect() got created. Why the name useEffect? I don't know, it's the only negative point of React-Hooks, I don't like this name. Anyway, here how it's work: with one function you can have the behaviour of `componentDidMount`, `componentDidUpdate` and `componentWillUnmount`:

* To make useEffect run after every render, just have one function as argument.
* To make useEffect run just once, like `componentDidMount`, add an empty array as second argument.
* To make useEffect run when one of the state changes, fill the empty array with the state.
* To make useEffect run a function when the component unmount, return a function. It's called **Clean up**.

```javascript
import React, { useState, useEffect } from "react";

function CountSheep() {

    const [count, setSheep] = useState(0);

    useEffect(() => {
        // will take my last count
        fetch('myapi/sheeps/count')
            .then(data = data.json())
            .then(count => setSheep(count));

        //Will save my count when clean up.
        return () => {
            if(count > 100) {
                fetch('myapi/sheeps/saveCount/' + count);
            }
        };
    }, [] /* Will only run once. If [count], will run eachtime count update*/ );

    return(
        <div>
            <div>
                <p>{`I already saw ${count} sheep${count > 1 ? 's' : ''}!`}</p>
            </div>
            <button
                onClick={() => setSheep(count + 1)}
            >
                Add one!
            </button>
        </div>
    );
}
```

Like as said, beside the name "useEffect" that I'm not fond of, it's great. Of course, you can use mutliple times useState and useEffect to separate your logic.  

I think React Hooks are not just an addition to the library. I think it shows the direction React take. With other tools of React 16.6 like Memo, Lazy, etc... and React 16.7, we can start to refactor our React code and make it look complety different. It's not insignificant. Those new functions will work very well with redux frameworks and I will be surprise if people use classes again in a near futur even if [classes are not discouraged for the moment](https://reactjs.org/docs/hooks-intro.html#gradual-adoption-strategy). 


For the negative points, I think "useEffect" will confuse people who start with React and they will think that with classes and classics lifecycle functions, plus now the hooks for the sfc, the learning curve is too steep. It's already the complains I heard from people that prefer Vue.js, that React is too complicated to learn. Well, React evolve, I think in a good direction, so I will encourage people to climb this steep curve, it's worth it!

This article is just a little summary of React Hooks, go see the [docs to a full understanding](https://reactjs.org/docs/hooks-intro.html)!
