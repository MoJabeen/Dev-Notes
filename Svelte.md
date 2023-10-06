
## Introduction

Javascript based framework, the app is built *declaratively* by using components. The components combine markup, styles and behaviours.

Using an application framework like SvelteKit allows you to build the entire app. **Components** can be also be shipped as standalone packages.

## Components

A component is a reusable self contained block encapsulating HTML, CSS and JS into a .svelte file. 

Use script tag to add variables (then can be referred to by their name in markup):

```js
<script>
	let name = 'meh'
</script>

<h1> {name}! </h1>
```

