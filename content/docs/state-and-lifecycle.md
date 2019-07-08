---
id: state-and-lifecycle
title: State and Lifecycle
permalink: docs/state-and-lifecycle.html
redirect_from:
  - "docs/interactivity-and-dynamic-uis.html"
prev: components-and-props.html
next: handling-events.html
---

This page introduces the concept of state and lifecycle in a React component. You can find a [detailed component API reference here](/docs/react-component.html).

Consider the ticking clock example from [one of the previous sections](/docs/rendering-elements.html#updating-the-rendered-element). In [Rendering Elements](/docs/rendering-elements.html#rendering-an-element-into-the-dom), we have only learned one way to update the UI. We call `ReactDOM.render()` to change the rendered output:

```js{8-11}
function tick() {
  const element = (
    <div>
      <h1>Hello, world!</h1>
      <h2>It is {new Date().toLocaleTimeString()}.</h2>
    </div>
  );
  ReactDOM.render(
    element,
    document.getElementById('root')
  );
}

setInterval(tick, 1000);
```

[**Try it on CodePen**](https://codepen.io/gaearon/pen/gwoJZk?editors=0010)

In this section, we will learn how to make the `Clock` component truly reusable and encapsulated. It will set up its own timer and update itself every second.

We can start by encapsulating how the clock looks:

```js{3-6,12}
function Clock(props) {
  return (
    <div>
      <h1>Hello, world!</h1>
      <h2>It is {props.date.toLocaleTimeString()}.</h2>
    </div>
  );
}

function tick() {
  ReactDOM.render(
    <Clock date={new Date()} />,
    document.getElementById('root')
  );
}

setInterval(tick, 1000);
```

[**Try it on CodePen**](https://codepen.io/gaearon/pen/dpdoYR?editors=0010)

However, it misses a crucial requirement: the fact that the `Clock` sets up a timer and updates the UI every second should be an implementation detail of the `Clock`.

Ideally we want to write this once and have the `Clock` update itself:

```js{2}
ReactDOM.render(
  <Clock />,
  document.getElementById('root')
);
```

To implement this, we need to add "state" to the `Clock` component.

State is similar to props, but it is private and fully controlled by the component.

## Adding Local State Hook to a Class {#adding-local-state-to-a-class}

We will move the `date` from props to state hook in four steps:

1) Replace `this.props.date` with `date` in the `render()` method:

```js
import React from 'react';

function Clock(props) {
  return (
    <div>
      <h1>Hello, world!</h1>
      <h2>It is {date.toLocaleTimeString()}.</h2>
    </div>
  );
}

// class Clock extends React.Component {
//   render() {
//     return (
//       <div>
//         <h1>Hello, world!</h1>
//         <h2>It is {this.state.date.toLocaleTimeString()}.</h2>
//       </div>
//     );
//   }
// }
```

2) Import `{ useState }` from the `react` package
```
import React, { useState } from 'react';
```

3) Add `useState` to create a pair of values: the current state (`date`) and a function that updates it (`setDate`).

```js
import React, { useState } from 'react';

function Clock(props) {
  const [date, setDate] = useState(new Date());
  return (
    <div>
      <h1>Hello, world!</h1>
      <h2>It is {date.toLocaleTimeString()}.</h2>
    </div>
  );
}
```

4) Remove the `date` prop from the `<Clock />` element:

```js{2}
ReactDOM.render(
  <Clock />,
  document.getElementById('root')
);
```

We will later add the timer code back to the component itself.

The result looks like this:

```js
import React, { useState } from 'react';

function Clock(props) {
  const [date, setDate] = useState(new Date());
  return (
    <div>
      <h1>Hello, world!</h1>
      <h2>It is {date.toLocaleTimeString()}.</h2>
    </div>
  );
}

ReactDOM.render(
  <Clock />,
  document.getElementById('root')
);
```

[**Try it on CodePen**](https://codepen.io/maxencebouret/pen/KjGLJL?editors=0010)

Next, we'll make the `Clock` set up its own timer and update itself every second.

## Adding the Effect Hook {#adding-lifecycle-methods-to-a-class}

In applications with many components, it's very important to free up resources taken by the components when they are destroyed.

We want to [set up a timer](https://developer.mozilla.org/en-US/docs/Web/API/WindowTimers/setInterval) whenever the `Clock` is rendered to the DOM for the first time. This is called "mounting" in React.

We also want to [clear that timer](https://developer.mozilla.org/en-US/docs/Web/API/WindowTimers/clearInterval) whenever the DOM produced by the `Clock` is removed. This is called "unmounting" in React.

I simple version where the timer is not cleared can be written like this:

```js
import React, { useState, useEffect } from 'react';

function Clock(props) {
  const [date, setDate] = useState(new Date());
  
  // The useEffect method is triggered the 1st time the component is rendered (because of the parameter[])
  useEffect(() => {
    setInterval(() => {
      setDate(new Date());
    }, 1000);
  }, [])
  
  return (
    <div>
      <h1>Hello, world!</h1>
      <h2>It is {date.toLocaleTimeString()}.</h2>
    </div>
  );
}

ReactDOM.render(
  <Clock />,
  document.getElementById('root')
);
```
[**Try it on CodePen**](https://codepen.io/maxencebouret/pen/xoyoZq?editors=0010)


The `useEffect` function runs after the component output has been rendered to the DOM. In our case, it's only executed after the 1st render (because of the second parameter `[]`) and it sets the timer.


**But our `Clock` never stops the `setInterval` and it could be problematic.** Let's inspect and try the following code


```js
import React, { useState, useEffect } from 'react';

function Clock(props) {
  const [date, setDate] = useState(new Date());
  
  useEffect(() => {
    setInterval(() => {
      console.log("Tic");
      setDate(new Date());
    }, 1000);
  }, [])
  
  return (
    <div>
      <h1>Hello, world!</h1>
      <h2>It is {date.toLocaleTimeString()}.</h2>
    </div>
  );
}

function App(props) {
  const [displayClock, setDisplayClock] = useState(true);
  return (
    <div>
      {displayClock && <Clock />}
      <button onClick={() => setDisplayClock(false)}>Remove Clock</button>
    </div>
  )
}

ReactDOM.render(
  <App />,
  document.getElementById('root')
);
```
[**Try it on CodePen**](https://codepen.io/maxencebouret/pen/mZzZqQ?editors=0010)

We have some new elements in our `App` component that we will explain later. The most important thing to understand is that the `App` component can remove the `Clock`, that will be stopped rendering.

But right now, when the `Clock` is removed, the `setInterval` is still running, displaying *"Tic"* every second and trying to call `setDate` even if it's not necessary anymore.

To solve the problem, we can write the component this way:
```js
import React, { useState, useEffect } from 'react';

function Clock(props) {
  const [date, setDate] = useState(new Date());
  
  useEffect(() => {
    let intervalId = setInterval(() => {
      console.log("Tic");
      setDate(new Date());
    }, 1000);
    // The function is executed just before destroying the component
    return () => {
      clearInterval(intervalId)
    }
  }, [])
  
  return (
    <div>
      <h1>Hello, world!</h1>
      <h2>It is {date.toLocaleTimeString()}.</h2>
    </div>
  );
}

function App(props) {
  const [displayClock, setDisplayClock] = useState(true);
  return (
    <div>
      {displayClock && <Clock />}
      <button onClick={() => setDisplayClock(false)}>Remove Clock</button>
    </div>
  )
}

ReactDOM.render(
  <App />,
  document.getElementById('root')
);
```
[**Try it on CodePen**](https://codepen.io/maxencebouret/pen/YoJmPg?editors=0010)



## Using State Correctly {#using-state-correctly}

There are three things you should know about state hooks/

### Do Not Modify State Directly {#do-not-modify-state-directly}

For example, this will not re-render a component:

```js
const [comment, setComment] = useState('');

// Wrong
this.state.comment = 'Hello';
```

Instead, use `setComment()`:

```js
const [comment, setComment] = useState('');

// Correct
this.setComment('Hello');
```

The only place where you can assign `this.state` is the constructor.

## The Data Flows Down {#the-data-flows-down}

Neither parent nor child components can know if a certain component is stateful or stateless, and they shouldn't care whether it is defined as a function or a class.

This is why state is often called local or encapsulated. It is not accessible to any component other than the one that owns and sets it.

A component may choose to pass its state down as props to its child components:

```js
<h2>It is {date.toLocaleTimeString()}.</h2>
```

This also works for user-defined components:

```js
<FormattedDate date={date} />
```

The `FormattedDate` component would receive the `date` in its props and wouldn't know whether it came from the `Clock`'s state, from the `Clock`'s props, or was typed by hand:

```js
function FormattedDate(props) {
  return <h2>It is {props.date.toLocaleTimeString()}.</h2>;
}
```

[**Try it on CodePen**](https://codepen.io/gaearon/pen/zKRqNB?editors=0010)

This is commonly called a "top-down" or "unidirectional" data flow. Any state is always owned by some specific component, and any data or UI derived from that state can only affect components "below" them in the tree.

If you imagine a component tree as a waterfall of props, each component's state is like an additional water source that joins it at an arbitrary point but also flows down.

To show that all components are truly isolated, we can create an `App` component that renders three `<Clock>`s:

```js{4-6}
function App() {
  return (
    <div>
      <Clock />
      <Clock />
      <Clock />
    </div>
  );
}

ReactDOM.render(
  <App />,
  document.getElementById('root')
);
```

[**Try it on CodePen**](https://codepen.io/gaearon/pen/vXdGmd?editors=0010)

Each `Clock` sets up its own timer and updates independently.

In React apps, whether a component is stateful or stateless is considered an implementation detail of the component that may change over time. You can use stateless components inside stateful components, and vice versa.
