---
layout: article
mathjax: true
title:  "My experience (so far) with Mobx in React Native"
date:   2018-07-23 21:30:40 +1000
tags: [Javascript, React Native]
category: blog
key: my-experience-so-far-with-mobx-in-react-native
---

# Previous experience with Redux & Redux Thunk

When I started learning React, Redux Thunk was my first choice. It appears that most people would recommend it when you just start getting into state management and using Redux. It allows you to write action creators to return functions instead of actions, which is great for async operations and creating actions conditionally. My overall experience with Redux Thunk was positive however sometimes it felt a bit cumbersome.

## The good

The good thing for me using Redux & Redux Thunk was that it helped me understand the benefits of using statement tool. Having the applications's state stored separately from the components keeps the code clean and easier to debug. With actions and reducers clearly defined you can easily see what data in the state can be changed and who can change it. This prevent accidentaly messing up the state from a place that shouldn't be dealing with state directly. Using the Redux Dev Tool you can watch state changes on the fly which is really awesome.

## The bad

The cumbersome part with Redux & Redux Thunk for me was that it was a lot of work to get everything set up properly before I can actually implement real logic. You'll need to combine the reducers together, create the store, apply the thunk middleware and inject the store to the top level component. If you are using the Container and Presentational Components pattern you will need to some extra work to map action creators and state to props in the container component and pass them along to the presentational component. This whole process takes time, and down the road when you need to add in new stuff you will need to go through the process again to create an action, add an action creator and a reducer function.

The other thing I found specifically with Redux Thunk is that things get tricky when you need to chain async actions. Suppose in your component you are calling two async actions, but you need one to be executed only when the other one completes. I had to return a `Promise` from the first action and use `await`. It took me some time to find out that the dispatch function actually returns the Promise back to the caller so you can use `await` or call `.then()` on it.

# Experience (so far) with Mobx

## The good

With a recent project I tried Mobx and I have been enjoying it so far. I think having used Redux & Redux Thunk helps me to appreciate how simple and effcient Mobx is. Mobx takes a different approach and uses reactive programming. From [https://mobx.js.org/](https://mobx.js.org/):

> Anything that can be derived from the application state, should be derived. Automatically.

With Mobx you still define your stores, however you don't need to define reducers and actions, Mobx takes care of that part for you. You simply mark the data you will be using in the component as observable, and write actions to update the state data just like a normal javascript function (except you only need to wrap that bit in `runInAction` when you modify the state in a async function). When it changes your component will "react" to it and re-render automatically, that's it.

I soon started enjoying this approach and it is so much easier to add new states and new actions, along with handy utilies like computed values, autorun and when etc it's much less code to write. 

When you need to chain asyn actions, after you've defined them in the store you simply call `await` in your component, no hidden things here.

## The bad

One thing I haven't figured out is how to get the Mobx Dev Tools working with React Native. The tool has not been updated in a while and isn't working with the latest version of Mobx. At the moment I mainly use `console.log()`, which isn't bad just not as efficient.

Another thing is that I found when using destructing assignment such as `const { user } = this.props.store`, the component won't react to state changes. This is a common pitfall described in Mobx's documentation at [here](https://mobx.js.org/best/pitfalls.html#don-t-copy-observables-properties-and-store-them-locally). So I ended up using the state directly in the component as `{this.props.store.user}`. But this is not a deal breaker at all.

# Final thoughts
I think from now on Mobx will be my go-to option for state management with React/React Native. I do hope the dev tool for React Native will be updated sooner.  I suggest you start with Redux and understand why/how it works before using Mobx. But if you are familiar with reactive programming already by all means use Mobx.
