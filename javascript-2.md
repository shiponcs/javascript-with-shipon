# JavaScript-2

- ***this*** keyword

  - in global context

  - object method

  - simple function

  - arrow function

  - even listener

  - object construction

    ------

    

1. ***this* In global context**:

   ```js
   function f1() {
       console.log('this: ', this); // this ---> global object or window object
   }
   f1();
   ```

2. **in Object method**:

   ```js
   const ob = {
       name: 'alice',
       f1: function() {
           
           console.log(this);
           debugger;
       }
   }
   ob.f1();  // this --> ob
   ```

3. **in simple function:**

   ```js
   function simpleFunct() {
       console.log('from simpleFunct, this', this);
   }
   
   const ob = {
       name: 'Alice',
       f1: function() {
           console.log('inside of ob: this', this);
           simpleFunct(); 
       }
   }
   
   simpleFunct(); // window
   ob.f1();       // ob
   ```

   Another Example, 

   ```js
   const ob = {
       name: 'Alice',
       doSomething: function() {
           setTimeout(function() {
               this.f1();
           }, 1000);
       },
       f1: function() {
           console.log('this', this);
       }
   }
   
   ob.doSomething();
   ```

   What should be the output?

   Shows an error? yes. Let's see why.

   At line `5`, *this* points to *window*, not at `ob`, because here the call back is a simple function not a object method, you could debug this way:

   

   ```js
   const ob = {
       name: 'Alice',
       doSomething: function() {
           setTimeout(function() {
               debugger;
               this.f1();
           }, 1000);
       },
       f1: function() {
           console.log('this', this);
       }
   }
   
   ob.doSomething();
   ```

   

    Notice, at *local scope*, `this` points to window, and there's no `f1` function in *window* or *global*.

   Try  the same code to rewrite in these two ways:

   <u>way-1:</u>

   ```js
   const ob = {
       name: 'Alice',
       doSomething: function() {
           debugger;
           this.f1();
           setTimeout(function() {
               console.log('from simple function', this);
           }, 1000);
       },
       f1: function() {
           console.log('this', this);
       }
   }
   
   ob.doSomething();
   ```

   <u>way-2:</u>

   ```js
   const ob = {
       name: 'Alice',
       doSomething: function() {
   
           setTimeout(function() {
               debugger;
               this.f1();
           }, 1000);
       }
   }
   function f1() {
       console.log('this', this);
   }
   
   ob.doSomething();
   ```

   Examine the values at the breakpoint, check what  `this` points to.

   Now one remedy to the error we got at first code:

   ```js
   const ob = {
       name: 'Alice',
       doSomething: function() {
           setTimeout(function() {
               ob.f1(); // this time we're calling object-method, f1,  specifically
           }, 1000);
       },
       f1: function() {
           console.log('this', this);
       }
   }
   
   ob.doSomething();
   ```

   But, if we use **strict mode**, `this` remains undefined unless your define it by yourself:

   ```js
   function simpleFunc() {
       'use strict';
       console.log(this);  // this --> undefined
   }
   simpleFunc();
   ```





4. **Arrow Function:** It was introduced in ECMAScript6,

   Let's learn about it first:

   Consider this function:

   ```js
   const add = function(x, y) {
       return x + y;
   };
   ```

   We can rewrite it using *arrow function:*

   ```js
   const add = (x, y) => {
       return x + y;
   };
   ```

   or, {} and `return` keyword can be omitted if we have only the `return` statement as function description.

   ```js
   const add = (x, y) => x + y; 
   ```

   Furthermore, if we have only one parameter, () can be omitted too:

   ```js
   const doubleOf = x => 2 * x;
   ```

    Now recall this example:

   ```js
   const ob = {
       name: 'Alice',
       doSomething: function() {
           setTimeout(function() {
               debugger;
               this.f1();
           }, 1000);
       },
       f1: function() {
           console.log('this', this);
       }
   }
   
   ob.doSomething();
   ```

   We got an error here because, as we said earlier, **this** inside a simple function points to window by default.

   But if we rewrite the simple function with *arrow function*, `this` will point to its lexical scope. This time it'll show no error.

   ```js
   const ob = {
       name: 'Alice',
       doSomething: function() {
           setTimeout(() => {
               debugger;
               this.f1(); // this points to ob
           }, 1000);
       },
       f1: function() {
           console.log('this', this);
       }
   }
   
   ob.doSomething();
   ```

   





#### Apply(), call(), bind() - hard binding: 

```js
function f1() {
    console.log(this.name);
}

const ob = {
    name:'Alice',
    f2: function() {
        console.log()
    }
}

f1();
```

name is empty as name was not declared in the window so far. But, our intention is to use `ob.name` , to achieve this we can use ***call()***. See the example below:

```js
function f1() {
    debugger;
    console.log(this.name);
}

const ob = {
    name:'Alice',
    f2: function() {
        console.log()
    }
}

f1.call(ob); // look carefully, now the 'this' pointer of f1() will points to ob, first                      parameter of f1.call()
```

Let's extend this example to understand the power of ***call()***:

```js
function f1() {
    debugger;
    console.log(this.name);
}

const ob = {
    name:'Alice',
    f2: function() {
        console.log()
    }
}

const ob1 = {
    name: 'Bob'
}

f1.call(ob); // prints Alice
f1.call(ob1); // prints Bob

```

- You see how scope are changing in runtime? This is called ***Dynamic Scoping*** .Remember, in case of **closure** you(programmers) have no power to change once it's set. 

Till now, all above code examples can be rewritten using *.apply()* method, nothing will change in output. 

Let's see one:

```js
function f1() {
    debugger;
    console.log(this.name);
}

const ob = {
    name:'Alice',
    f2: function() {
        console.log()
    }
}

const ob1 = {
    name: 'Bob'
}

f1.apply(ob); // prints Alice
f1.apply(ob1); // prints Bob

```

But don't think that .*apply()* is 100% clone of *.call()*, there are some differences in further applications:

Let's recall the above example with slight modification:

```js
function f1(x, y) {
    debugger;
    console.log('sum: ', x + y);
    console.log(this.name);
}

const ob = {
    name:'Alice',
    f2: function() {
        console.log()
    }
}

const ob1 = {
    name: 'Bob'
}

f1.apply(ob, [1, 99]); // prints 100, Alice
f1.apply(ob1, [99, 100]); // prints 199, Bob

```

If we use *.call()* here:

```js
function f1(x, y) {
    debugger;
    console.log('sum: ', x + y);
    console.log(this.name);
}

const ob = {
    name:'Alice',
    f2: function() {
        console.log()
    }
}

const ob1 = {
    name: 'Bob'
}

f1.call(ob, 1, 99); // prints 100, Alice
f1.call(ob1, 99, 100); // prints 199, Bob

```

You got the differences? If you need to pass the arguments for the function you're calling the *.call()/.apply()* upon, then in case of *apply()* you've to pass the arguments as enclosed with square bracket. `[1, 99]`, an array which is the second parameter of the *.apply()* function, and in case of  *.call()*. you just start the from second parameter of the *call()* function and put them separated by commas.

Let's see how to use .*bind()*:

> The bind() method creates a new function that, when called, has its this keyword set to the provided value, with a given sequence of arguments preceding any provided when the new function is called. 
>
> -MDN

```js
function f1() {
  debugger;
  console.log(this.name);
}

const ob = {
  name: "Alice",
  f2: function() {
    console.log();
  }
};
const ob1 = {
  name: "Bob"
};
const a = f1.bind(ob);
a(); // prints Alice
a.apply(ob1); // but this can't change the 'this' pointer, as it was binded for ob earlier.                 so this will still print Alice

```



5. Event Listener:

   

6. Object Construction (new keyword):

   - what's a construction call?
   
     ​    function  call with *new* keyboard. What happens when we call a function with *`new`* keyword?
   
     ​    Four things happen-
   
        1. Create a brand new empty object (aka constructed) out of thin layer air.
   
        2. Newly created/constructed object is linked to (`[[prototype]]`) the function's prototype. (All variables of the function (constructor in that case) we called are set as the attributes of that brand new object.)
   
        3. Newly created/constructed object is set as the `this` binding for that function call.
   
        4. Unless the function returns its own alternate object, the new-invoked call will automatically return the **newly created object**. (If the constructor doesn't return anything then the created object is returned.)
   
           Look at this example:
   
           ```js
           function F1(name) {
             console.log(name);
             this.name1 = name;
           }
           
           ob = new F1('hey');
           console.log(ob);
           ob1 = new F1('hello');
           console.log(ob1);
           ```
   
           In this example *this* in the function points to the newly created object.
   
           ( *constructor call* will be covered in details in ***prototype-chain***  )
   
           

