---
id: forms
title: Forms
permalink: docs/forms.html
prev: lists-and-keys.html
next: lifting-state-up.html
redirect_from:
  - "tips/controlled-input-null-value.html"
  - "docs/forms-zh-CN.html"
---

HTML form elements work a little bit differently from other DOM elements in React, because form elements naturally keep some internal state. For example, this form in plain HTML accepts a single name:

```html
<form>
  <label>
    Name:
    <input type="text" name="name" />
  </label>
  <input type="submit" value="Submit" />
</form>
```

This form has the default HTML form behavior of browsing to a new page when the user submits the form. If you want this behavior in React, it just works. But in most cases, it's convenient to have a JavaScript function that handles the submission of the form and has access to the data that the user entered into the form. The standard way to achieve this is with a technique called "controlled components".

## Controlled Components {#controlled-components}

In HTML, form elements such as `<input>`, `<textarea>`, and `<select>` typically maintain their own state and update it based on user input. In React, mutable state is typically kept in the state property of components, and only updated with [`setState()`](/docs/react-component.html#setstate).

We can combine the two by making the React state be the "single source of truth". Then the React component that renders a form also controls what happens in that form on subsequent user input. An input form element whose value is controlled by React in this way is called a "controlled component".

For example, if we want to make the previous example log the name when it is submitted, we can write the form as a controlled component:

```javascript
function NameForm(props) {
  const [value,setValue] = useState('');

  function handleChange(event) {
    setValue(event.target.value);
  }

  function handleSubmit(event) {
    alert('A name was submitted: ' + value);
    event.preventDefault();
  }

  return (
    <form onSubmit={handleSubmit}>
      <label>
        Name:
        <input type="text" value={value} onChange={handleChange} />
      </label>
      <input type="submit" value="Submit" />
    </form>
  );
}

ReactDOM.render(
  <NameForm />,
  document.getElementById('root')
);
```

[**Try it on CodePen**](https://codepen.io/maxencebouret/pen/wLQMZd?editors=0010)

Since the `value` attribute is set on our form element, the displayed value will always be `value`, making the React state the source of truth. Since `handleChange` runs on every keystroke to update the React state, the displayed value will update as the user types.

With a controlled component, every state mutation will have an associated handler function. This makes it straightforward to modify or validate user input. For example, if we wanted to enforce that names are written with all uppercase letters, we could write `handleChange` as:

```javascript{2}
handleChange(event) {
  setValue(event.target.value.toUpperCase());
}
```

## The textarea Tag {#the-textarea-tag}

In HTML, a `<textarea>` element defines its text by its children:

```html
<textarea>
  Hello there, this is some text in a text area
</textarea>
```

In React, a `<textarea>` uses a `value` attribute instead. This way, a form using a `<textarea>` can be written very similarly to a form that uses a single-line input:

```javascript
function EssayForm(props) {
  const [value,setValue] = useState('Please write an essay about your favorite DOM element.')

  function handleChange(event) {
    setValue(event.target.value);
  }

  function handleSubmit(event) {
    alert('An essay was submitted: ' + value);
    event.preventDefault();
  }

  return (
    <form onSubmit={handleSubmit}>
      <label>
        Essay:
        <textarea value={value} onChange={handleChange} />
      </label>
      <input type="submit" value="Submit" />
    </form>
  );
}

ReactDOM.render(
  <EssayForm />,
  document.getElementById('root')
);
```

[**Try it on CodePen**](https://codepen.io/maxencebouret/pen/YoRqKw?editors=0010)

Notice that the state hook `value` is initialized, so that the text area starts off with some text in it.

## The select Tag {#the-select-tag}

In HTML, `<select>` creates a drop-down list. For example, this HTML creates a drop-down list of flavors:

```html
<select>
  <option value="grapefruit">Grapefruit</option>
  <option value="lime">Lime</option>
  <option selected value="coconut">Coconut</option>
  <option value="mango">Mango</option>
</select>
```

Note that the Coconut option is initially selected, because of the `selected` attribute. React, instead of using this `selected` attribute, uses a `value` attribute on the root `select` tag. This is more convenient in a controlled component because you only need to update it in one place. For example:

```js
function FlavorForm(props) {
  const [value, setValue] = useState('coconut')

  function handleChange(event) {
    setValue(event.target.value);
  }

  function handleSubmit(event) {
    alert('Your favorite flavor is: ' + value);
    event.preventDefault();
  }

  return (
    <form onSubmit={handleSubmit}>
      <label>
        Pick your favorite flavor:
        <select value={value} onChange={handleChange}>
          <option value="grapefruit">Grapefruit</option>
          <option value="lime">Lime</option>
          <option value="coconut">Coconut</option>
          <option value="mango">Mango</option>
        </select>
      </label>
      <input type="submit" value="Submit" />
    </form>
  );
}
```

[**Try it on CodePen**](https://codepen.io/maxencebouret/pen/zVMqWY?editors=0010)

Overall, this makes it so that `<input type="text">`, `<textarea>`, and `<select>` all work very similarly - they all accept a `value` attribute that you can use to implement a controlled component.

> Note
>
> You can pass an array into the `value` attribute, allowing you to select multiple options in a `select` tag:
>
>```js
><select multiple={true} value={['B', 'C']}>
>```

## The file input Tag {#the-file-input-tag}

In HTML, an `<input type="file">` lets the user choose one or more files from their device storage to be uploaded to a server or manipulated by JavaScript via the [File API](https://developer.mozilla.org/en-US/docs/Web/API/File/Using_files_from_web_applications).

```html
<input type="file" />
```

Because its value is read-only, it is an **uncontrolled** component in React. It is discussed together with other uncontrolled components [later in the documentation](/docs/uncontrolled-components.html#the-file-input-tag).

## Handling Multiple Inputs {#handling-multiple-inputs}

When you need to handle multiple controlled `input` elements, you can add a `name` attribute to each element and let the handler function choose what to do based on the value of `event.target.name`.

For example:

```js
function Reservation(props) {
  const [state, setState] = useState({ 
    isGoing: true,
    numberOfGuests: 2
  })

  function handleInputChange(event) {
    const target = event.target;
    const value = target.type === 'checkbox' ? target.checked : target.value;
    const name = target.name;

    setState({
      ...state,
      [event.target.name]: value
    });
  }

  return (
    <form>
      <label>
        Is going:
        <input
          name="isGoing"
          type="checkbox"
          checked={state.isGoing}
          onChange={handleInputChange} />
      </label>
      <br />
      <label>
        Number of guests:
        <input
          name="numberOfGuests"
          type="number"
          value={state.numberOfGuests}
          onChange={handleInputChange} />
      </label>
    </form>
  );
}
```

[**Try it on CodePen**](https://codepen.io/maxencebouret/pen/BgGzLv?editors=0010)

Note how we used the ES6 [computed property name](https://developer.mozilla.org/en/docs/Web/JavaScript/Reference/Operators/Object_initializer#Computed_property_names) syntax to update the state key corresponding to the given input name:

```js
setState({
  ...state,
  [event.target.name]: value
});
```


## Controlled Input Null Value {#controlled-input-null-value}

Specifying the value prop on a [controlled component](/docs/forms.html#controlled-components) prevents the user from changing the input unless you desire so. If you've specified a `value` but the input is still editable, you may have accidentally set `value` to `undefined` or `null`.

The following code demonstrates this. (The input is locked at first but becomes editable after a short delay.)

```javascript
ReactDOM.render(<input value="hi" />, mountNode);

setTimeout(function() {
  ReactDOM.render(<input value={null} />, mountNode);
}, 1000);

```

## Alternatives to Controlled Components {#alternatives-to-controlled-components}

It can sometimes be tedious to use controlled components, because you need to write an event handler for every way your data can change and pipe all of the input state through a React component. This can become particularly annoying when you are converting a preexisting codebase to React, or integrating a React application with a non-React library. In these situations, you might want to check out [uncontrolled components](/docs/uncontrolled-components.html), an alternative technique for implementing input forms.

## Fully-Fledged Solutions {#fully-fledged-solutions}

If you're looking for a complete solution including validation, keeping track of the visited fields, and handling form submission, [Formik](https://jaredpalmer.com/formik) is one of the popular choices. However, it is built on the same principles of controlled components and managing state — so don't neglect to learn them.
