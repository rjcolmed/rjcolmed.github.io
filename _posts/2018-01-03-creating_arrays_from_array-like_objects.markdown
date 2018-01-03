---
layout: post
title:      "Creating Arrays from Array-like Objects"
date:       2018-01-03 17:56:15 +0000
permalink:  creating_arrays_from_array-like_objects
---


During my deeper travels into JavaScript, I've encountered a few useful ways to convert Array-like objects into arrays.

## Objects Pretending to Be Arrays

First, though, what is an Array-like object? It's something that looks like an array, but isn't. 

An Array-like object:

* Properties with numeric indices
* A `.length` property
* Lacks the methods you'd expect an Array object to have, e.g. `.push`, `.pop`, `.forEach`.

Have you ever tried to use `.forEach` on a bunch of `<div>` elements you gathered with `document.getElementsByName`? Yeah? If so, you've run into an array-like object, namely an HTMLCollection. Similar thereto is a NodeList, which you'd encounter if you tried to gather your divs with `document.querySelectorAll('div')`

Another commonly-encountered Array-like object is `[arguments](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Functions/arguments)`. If you've never run in to `arguments` or had cause to, it's a local variable—more specifically, an object—available within all non-arrow functions. 

Here's a simple piece of code you can use to interact with `arguments`:

```
function getArguments() {
    return arguments;
}

getArguments('flippity', 'floppity');
```

Copy and paste it into Chrome's dev console and you'll see that it returns an Arguments object.

## Helping Non-Arrays Be All They Can Be

Now that we have a good idea of what an Array-like object is, and where we might encounter it in the wild, how do we convert them into Array objects, so we can take advantage of  useful methods, e.g. `.forEach`?

Let's say you want to iterate over, and do something with, a bunch of divs you gathered from a page. Normally, you wouldn't be able to, or would have to make use of a `for` loop, like so:

```
const divs = document.getElementsByTagName('div');

for ( let i = 0; i < divs.length; i++ ) {
    //do something with each div
}
```

If you were so inclined, you could convert the HTMLCollection of divs you've got on your hands into an array using one of the following:

### Array.prototype.slice and .call

You can call on the Array object to help you out by chaining its [.slice](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/slice) method and [.call](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Function/call).

Using our HTMLCollection as an example:

```
const newArray = Array.prototype.slice.call(document.getElementsByTagName('div'));

newArray.forEach(div => //do something with div);

```

### [].slice() and .call()

`[]` is a proxy for Array.prototype, which means less typing, which is good for our fingers. 

Using our HTMLCollection as an example:

```
const newArray = [].slice.call(document.getElementsByTagName('div'));

newArray.forEach(div => //do something with div);
```

### Array.from()

Continuing our petition to the Array object, we can also use [Array.from()](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/from). **NB**: This method won't work in Internet Explorer without a bit of extra help (see the linked MDN page for more info).

Using our HTMLCollection as an example:

```
const newArray = Array.from(document.getElementsByTagName('div'));

newArray.forEach(div => //do something with div);
```

### [...]

Finally, the spread operator is a fantastic way to convert Array-like objects into arrays. It's cleaner and pithier than the previously-listed methods. One downside, however, is that it's a bit less semantic than `Array.from`, but I still prefer it.

Using our HTMLCollection as an example:

```
const newArray = [...document.getElementsByTagName('div')]

newArray.forEach(div => //do something with div);

//In fact, we can more readily dispense with the creation of a variable as a container for the new array and do the following:

[...document.getElementsByTagName('div')].forEach(div => //do something with div);
```





