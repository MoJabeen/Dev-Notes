
## Introduction

Javascript based framework, the app is built *declaratively* by using components. The components combine markup, styles and behaviours.

Using an application framework like SvelteKit allows you to build the entire app. **Components** can be also be shipped as standalone packages.

## Components

A component is a reusable self contained block encapsulating HTML, CSS and JS into a .svelte file. 

Use script tag to add variables (then can be referred to by their name in markup):

```js
<script>
	let name = 'meh';
</script>

<h1> {name}! </h1>
```

Inside the curly brackets in the mark up can use ant JS function.

```js
<h1> {name.toUpperCase()} </h1>
```

### Elements

A variable is referred to as an element.

**Image Example**
```js
<script>
	let name = 'img.jpeg';
</script>

<img src={name} />
```

#### Accessibility (A11y)

Important to make elements accessible to any user. Warning shown as A11y.

Add the **alt** attribute for a description of the element.

```js
<img src={name} alt="A meh" />
```

*Attributes can also access script code.*

If the value of the attribute and element are the same dont need to specify attribute can only use curly brackets.

```js
<img src={src} />
//OR
<img {src} />
```