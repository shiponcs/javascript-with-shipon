# Asynchronous JavaScript

```js
function fn() {
    console.log('A');
    setTimeout( () => {       // it's bad practise to use anonymous function 
        console.log('B');
    }, 0 );
    console.log('C');
}
fn();
```

*read [this article](https://medium.com/front-end-weekly/javascript-event-loop-explained-4cd26af121d4) the above example has been explained there.*

After reading the explanation of the first example you should understand why 'B' is getting printed after 'C'. Let's summarize, **v8 engine** handover statements inside `setTImeout` to browser and as browser counts the delay time and then it comes to **task queue/message queue/callback queue** and waits till **event loop** sees that **engine's stack** is free. Hence, in the min-time other statements after `setTimeout` function are executed (even though the *delaytime* set is 0 ms).

Now look at this example(This example has also been explained in the article I mentioned above):

```js
function fn() {
    console.log('A');
    setTimeout(
        function exec() => {
            console.log('B');
    }, 0);
    holdNSeconds(3); // blocker
    console.log(C);
}
fn();
holdNSeconds(n) {
    let start = Date.now(); // in milisecond
    let cur = start;
    while( cur - start <= n * 1000 ) {
        cur = Date.now();
    }
}
```



You read the article clearly you should know at the very end of the article that that article was heavily influenced by a talk in JSConf by  [*Philip Roberts*](http://latentflip.com/), [watch this](https://www.youtube.com/watch?v=8aGhZQkoFbQ) and you should have also come to know [a visulaisation tool by him](http://latentflip.com/loupe/?code=JC5vbignYnV0dG9uJywgJ2NsaWNrJywgZnVuY3Rpb24gb25DbGljaygpIHsKICAgIHNldFRpbWVvdXQoZnVuY3Rpb24gdGltZXIoKSB7CiAgICAgICAgY29uc29sZS5sb2coJ1lvdSBjbGlja2VkIHRoZSBidXR0b24hJyk7ICAgIAogICAgfSwgMjAwMCk7Cn0pOwoKY29uc29sZS5sb2coIkhpISIpOwoKc2V0VGltZW91dChmdW5jdGlvbiB0aW1lb3V0KCkgewogICAgY29uc29sZS5sb2coIkNsaWNrIHRoZSBidXR0b24hIik7Cn0sIDUwMDApOwoKY29uc29sZS5sb2coIldlbGNvbWUgdG8gbG91cGUuIik7!!!PGJ1dHRvbj5DbGljayBtZSE8L2J1dHRvbj4%3D).

#### Callback hell:

Example-1:

```js
doA();
function doA() {
  console.log('doA');
 (function doB(){
   console.log('doB');
 })();
 (function doC(n){
   n();
   console.log('doC');
 })(doG);
 (function doE(){
   console.log('doE');
 })();
}

(function doF(){
  console.log('doF');
})();
function doG() {
  console.log('doG');
}
```



Example-2:

```js
(function doA(fn){
  console.log('inside doA');
  fn();
})(() => {
  doB();
  doC(function fn(){
    console.log('callback of doC');
  });
  doD();
});

doE();

function doB(){
  console.log('doB');
}

function doC(cc){
  cc();
}

function doD(){
  console.log('doD');
}

function doE() {
  console.log('doE');
}
```

Example-3:

```js
(function doA(fn){
  console.log('inside doA');
  setTimeout(fn, 5000);
})(() => {
  setTimeout(doB, 5000);
  doC(function fn(){
    console.log('callback of doC');
  });
  doD();
});

doE();

function doB(){
  console.log('doB');
}

function doC(cc){
  cc();
}

function doD(){
  console.log('doD');
}

function doE() {
  console.log('doE');
}
```



#### Promise

```js
function foo() {
  return new Promise((resolve, reject) => {
      setTimeout(() => {
          console.log('inside executor of promise');
          resolve('foo');
      }, 5000);
  });
}

foo()
   .then((resp) =>{
     console.log('Response: ', resp);
     return "I'm from first response";
   })
   .then((resp) => {
     console.log('from 1st response', resp);
   } )
   .catch((err) =>{
      console.log('error: ', err);
   });

   console.log('after foo');
```



Now carefully observe this practical use of promise:

```js
function getUser(userId) {
  return new Promise( ( resolve, reject ) => {
    console.log('resolving getUser');
    setTimeout( () => {
      resolve('john');
    }, 1000 );
  } );
}

function getBalance(userName) {
  return new Promise( ( resolve, reject ) => {
    console.log('resovling getBalance');
    setTimeout( () => {
      if( userName == 'john' ) {
        resolve('$2000');
      } else {
        reject('Username invalid');
      }
    }, 2000 );
  } );
}

async function getDoneTheBalanceFindingJob() {
    getUser('1')
        .then( getBalance ) 
        .then( (balance) => {
            console.log('your current balance is: ', balance);
         } )
        .catch( (err) => {
        console.log('error: ', err );
        } );
}
getDoneTheBalanceFindingJob();
```

This example is written in *ES6*, let's rewrite it in *ES8*:

```js
function getUser(userId) {
  return new Promise( ( resolve, reject ) => {
    console.log('resolving getUser');
    setTimeout( () => {
      resolve('john');
    }, 1000 );
  } );
}

function getBalance(userName) {
  return new Promise( ( resolve, reject ) => {
    console.log('resovling getBalance');
    setTimeout( () => {
      if( userName == 'john' ) {
        resolve('$2000');
      } else {
        reject('Username invalid');
      }
    }, 2000 );
  } );
}

async function getDoneTheBalanceFindingJob() {
  try{
    const userName = await getUser('1');
    const balance = await getBalance(userName);
    console.log('your current balance is ', balance);
  } catch(err) {
    console.log('err: ', err);
  }
}
getDoneTheBalanceFindingJob();
```



In the above two examples you must have noticed a `line 26` won't execute unless `line 25` is done. If we have many of such executions that will wait unless execution immediately precedes them is not done this will be a time consuming only if they have no dependency to each other.

```js
async function doubleAndAdd(a, b) {
  [resp1, resp2] = await Promise.all( [doubleAfter1Sec(a), doubleAfter2sec(b)] );
  return resp1 + resp2;
}

function doubleAfter1Sec(val) {
  return new Promise( (resoleve, reject) => {
    setTimeout( () => {
      resoleve(val * 2);
    }, 1000 );
  } );
}

function doubleAfter2sec(val) {
  return new Promise( (resoleve, reject) => {
    setTimeout( () => {
      resoleve(val * 2);
    }, 2000 );
  });
}

doubleAndAdd(1, 2)
    .then( (resp) => {
      console.log(resp);
    } )
    .catch( (err) => {
      console.log(err)
    } ); 

```

  

### Object.assign()

To add some new properties in an existing object:

```js
const a1 = {
  x: 1
};

Object.assign(a1, {y:2, z:3});  // fist parameter is the target object
 
console.log(a1);  // you will see x, y, z inside a1
```

`Object.assign()` returns a reference to that object updated. So we can write in this way:

```js
const a1 = {
  x: 1
};

b1 = Object.assign(a1, {y:2, z:3});  // fist parameter is the target object
 
console.log(a1);  // you will see x, y, z inside a1

console.log('a1 === b1', a1 === b1); // true
```

Another application of Object.assign is *cloning an object*:

Let's create a brand new object from one existing object:

```js
const a1 = {
  x: 1
};

b1 = Object.assign({}, a1);  // {} creates a new object, and clone a1 to the new object and returns a refernce to this new object to b1
 
console.log(b1);

console.log('a1 === b1', a1 === b1); // false
```

