# intermediate-reactjs

## useState
useState allows us to make our components stateful. Whereas this previously required using a class component, hooks give us the ability to write it using just functions. It allows us to have more flexible components. In our example component, everytime you click on the h1 (bad a11y, by the way) it'll change colors. It's doing this by keeping that bit of state in a hook which is being fed in anew every render so it always has the latest state.

## useEffect
Effects are how you recreate componentDidMount, componentDidUpdate, and componentDidUnmount from React. Inside useEffect, you can do any sort of sidd-effect type action that you would have previously done in one of React's lifecycle method. You can do things like fire AJAX requests, integrate with third party libraries (like a jQuery plugin), fire off some telemetry, or anything else that need to happen on the side for your component.

In our case, we want our component to continually update to show the time so we use setTimeout inside our effect. After the timeout calls the callback, it updates the state. After that render happens, it schedules another effect to happen, hence why it continues to update. You could provide a second parameter of [] to useEffect (after the function) which would make it only update once. This second array is a list of dependencies: only re-run this effect if one of these parameters changed. In our case, we want to run after every render so we don't give it this second parameter.

## useContext
An early problem with the React problem is called "data tunneling" or "prop drilling". This is when you have a top level component (in our case the parent component) and a child component way down in the hierarchy that need the same data (like the user object.) We could pass that data down, parent-to-child, for each of the intermediary components but that sucks because now each of LevelTwo, LevelThree, and LevelFour all have to know about the user object even when they themselves don't need it, just their children. This is prop drilling: passing down this data in unnecessary intermediaries.

Enter context. Context allows you to create a wormhole where stuff goes in and a wormhole in a child component where that same data comes out and the stuff in the middle doesn't know it's there. Now that data is available anywhere inside of the UserContext.Provider. useContext just pulls that data out when given a Context object as a parameter. You don't have to use useState and useContext together (the data can be any shape, not just useState-shaped) but I find it convenient when child components need to be able to update the context as well.

In general, context adds a decent amount of complexity to an app. A bit of prop drilling is fine. Only put things in context that are truly application-wide state like user information or auth keys and then use local state for the rest.

Often you'll use context instead of Redux or another state store. You could get fancy and use useReducer and useContext together to get a pretty great approximation of Redux-like features.

## useRef
Refs are useful for several things, we'll explore two of the main reasons in these examples. I want to show you the first use case: how to emulate instance variables from React.

In order to understand why refs are useful, you need to understand [how closures work][closures]. In our component, when a user clicks, it sets a timeout to log both the state and the ref's number after a second. One thing to keep in mind that the state and the ref's number are always the same. They are never out of lockstep since they're updated at the same time. However, since we delay the logging for a second, when it alerts the new values, it will capture what the state was when we first called the timeout (since it's held on to by the closure) but it will always log the current value since that ref is on an object that React consistently gives the same object back to you. Because it's the same object and the number is a property on the object, it will always be up to date and not subject to the closure's scope.

Why is this useful? It can be useful for things like holding on to setInterval and setTimeout IDs so they can be cleared later. Or any bit of statefulness that could change but you don't want it to cause a re-render when it does.

It's also useful for referencing DOM nodes directly and we'll see that a bit later in this section.

## useReducer
useReducer allows us to do Redux-style reducers but inside a hook. Here, instead of having a bunch of functions to update our various properties, we have one reducer that handles all the updates based on an action type. This is a preferable approach if you have complex state updates or if you have a situation like this: all of the state updates are very similar so it makes sense to contain all of them in one function.

## useMemo
useMemo and useCallback are performance optimizations. Use them only when you already have a performance problem instead of pre-emptively. It adds unnecessary complexity otherwise. useMemo memoizes expensive function calls so they only are re-evaluated when needed.

## useCallback
useCallback is quite similar and indeed it's implemented with the same mechanisms as useMemo. Our goal is that ExpensiveComputationComponent only re-renders whenever it absolutely must. Typically whenever React detects a change higher-up in an app, it re-renders everything underneath it. This normally isn't a big deal because React is quite fast at normal things. However you can run into performance issues sometimes where some components are bad to re-render without reason.

In this case, we're using a new feature of React called React.memo. This is similar to PureComponent where a component will do a simple check on its props to see if they've changed and if not it will not re-render this component (or its children, which can bite you.) React.memo provides this functionality for function components. Given that, we need to make sure that the function itself given to ExpensiveComputationComponent is the same function every time.

## useLayoutEffect
useLayoutEffect is almost the same as useEffect except that it's synchronous to render as opposed to scheduled like useEffect is. If you're migrating from a class component to a hooks-using function component, this can be helpful too because useLayout runs at the same time as componentDidMount and componentDidUpdate whereas useEffect is scheduled after. This should be a temporary fix.

The only time you should be using useLayoutEffect is to measure DOM nodes for things like animations. In the example, I measure the textarea after every time you click on it (the onClick is to force a re-render.) This means you're running render twice but it's also necessary to be able to capture the correct measurments.

## useImperativeHandle
Here's one you will likely never directly use but you may use libraries that use it for you. We're going to use it in conjunction with another feature called forwardRef that again, you probably won't use but libraries will use on your behalf. Let's explain first what it does using the example and then we'll explain the moving parts.
The first thing we use is useImperativeHandle. This allows us to customize methods on an object that is made available to the parents via the useRef API.

## useDebugValue
Here's another hook that's baked into React that I don't foresee many of you using but I still want you to know it's there. It, like useImperativeHandle, is more built for library authors.

useDebugValue allows you to surface information from your custom hook into the dev tools. This allows the developer who is consuming your hook (possibly you, possibly your coworker) to have whatever debugging information you choose to surfaced to them. If you're doing a little custom hook for your app (like the breed one we did in the Intro course) this probably isn't necessary. However if you're consuming a library that has hooks (like how react-router-dom has hooks) these can be useful hints to developers.

## CSS & React
There are many ways to do CSS with React. Even just in this course I've done several ways with [style-components][sc] and [emotion][emotion]. Both of those are fantastic pieces of software and could definitely still be used today. As is the case when I teach this course I try to cover the latest material and what I think is the current state-of-the-art of the industry and today I think that is [TailwindCSS][tailwind].

Both style-components and emotion are libraries that execute in the JavaScript layer. They bring your CSS into your JavaScript. This allows you all the power of JavaScript to manipulate styles using JavaScript.

Tailwind however is a different approach to this. And it bears mentioning that Tailwind isn't tied to React at all (whereas styled-components is and emotion mostly is.) Everything I'm showing you here is just incidentally using React (though Tailwind is particularly popular amongst React devs.)

Let's get it set up. Run this:

```sh
npm install -D tailwindcss@npm:@tailwindcss/postcss7-compat@2.0.3 postcss@7.0.35 autoprefixer@9.8.6 @tailwindcss/postcss7-compat@2.0.3
```

