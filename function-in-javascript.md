# Function in Javascript


*this post assumes that your already know javascript function*



It's crucial to understand how javascript works, I mean how *this* is assigned in javascript function.
Remember when we call a method/function of some objects in this way 
```js
obj.method()
```
In this case, an explicit argument is passed to the method and that is `this` assigned to the object before `.` (dot notation).

Now, consider the following example carefully:

```js
const myObj = {
  score: 12,
  increment: function () {
    this.score++;
  }
}
myObj.increment();
```
According to the rule just mentioned above, for the given call `obj.incement()`, `this` keyword in increment method refers to `myObj`. Clear?

Let's modify the above example:

```js
const myObj = {
  score: 12,
  increment: function () {
    const add = function () { this.score++ }
    add();
  }
}
myObj.increment();
```
Annoying modification? But you have to continue with me to understand how this binds.

Anyways, now `myObj.increment()` couldn't increment the value of score. Why?
Because, for the `myObj.increment()` call inside `increment()` this is assigned to `myObj`, but, `do()` function don't know where to assigned to it `this`, because in Javascript function this not default binded. Hence, `this.score` is `undefined` To fix this you have to bind `do()` function to somthing increment function's this point to. Like this way:

```js
const myObj = {
  score: 12,
  increment: function () {
    const add = function () { this.score++ }.bind(this);
    add();
  }
}
myObj.increment();
```
Now it's okay.

Next, I will explore the samething for event handler.

```js
const loginfo = {
  isLoggedIn: false,
  toggle: function() {
    this.isLoggedIn = true;
  }
}
```
Let's say we have a button and we want to add a event listener to that button, that invokes `loginfo.toggle`.

We can achieve this like as follows:
```js
document.querySelector(".btn").addEventListener( "onClick", 
  function() {
    loginfo.toggle();
  }
);
```
It works because you are referring a function for `onClick` that invokes `loginfo.toggle` function and just by the rule I mentioned at very earler `toggle()` method gets the explicit argument `this` that points to `loginfo` object. But it this approach we are creating a extra callback you may not like this. Again, you can't do in this way(what we do whene the function we refer to is just a funciton declared globally not inside another object):
```js
document.querySelector(".btn").addEventListener( "onClick", loginfo.toggle);
```
Because, here your are just referring to `loginfo.toggle`, you are not calling this, it's javascript itself that calls that referred function later. So when javascript will be invoking that function `this` keyword is undefined inside `toggle` method. 
There are more than one solution to this problem.

1. Using Bind()
```js
document.querySelector(".btn").addEventListener( "onClick", loginfo.toggle.bind(loginfo));
```
What `bind()` does is that it just creates a new function like `loginfo.toggle` but this time `this` is defined and `this` is assigned to what we pass as argument to the `bind` method. Why above solution works is simply because now we referring to a function where `this` is defined.

Morever, you can also bind it just where you define it.

2. Use arrow function in the method definition;
```js
const loginfo = {
  isLoggedIn: false,
  toggle: () => {
    this.isLoggedIn = true;
  }
}
document.querySelector(".btn").addEventListener( "onClick", loginfo.toggle);
```
Nooooooooo! Don't do that. 
Arrow function does have it own `this` binding, in arrow function `this` points to it lexically enclosing context. And in above example `this` inside arrow function is enclosed by `Window/global` object or `Object` (Uppercase starting) and you don't have isLogged defined in global scope. But you could solve this by using arrow function if loginfo was a function like this:
```js
const Loginfo = function() {
  this.isLoggedIn = false;
  this.toggle = () => {this.isLoggedIn = true;}
}
const obj1 = new Loginfor
document.querySelector(".btn").addEventListener( "onClick", obj1.toggle);
```

































****** All above examples were contrived 