---
layout: page
title: "Testing Exception Scenarios using Node.js assert module"
comments: true
sharing: true
footer: true
---

 In this article, we will see how to test an exception in javascript, using Node.js assert module. 

## Exceptions
 
  It's annoying to users when an application displays uncaught generic exceptions. Most common are accessing a null object. If it's happening in the client side in a javascript code, first it annoys users, and second, we, developers don't have any idea of that error.  So it's worth to take time to handle these scenarios, and display user friendly messages.
 
  Work is not complete until the code is error safe. If you are a defensive programmer, you would want to handle all possible scenarios, so that your code does not throw generic exceptions.  

  There are various assertion libraries available in Javascript such as Chai, Should, Node.js assert and so on. 

## node.js's assert module

  In this article, we will see how to test exception scenario using Node.js assert module.
##### Divide by zero 
 We will take very simple example, a divide by zero scenario. 

``` javascript
  function divide(dividend,divisor)
  {
    if(divisor === 0)
      throw new Error("can't divide by zero");
    return dividend/divisor;
  }
```
 As we will be focusing on assert.throws, we will just use this javascript file to test the assert.throws. We will not use a test framework. This concept can be used in any node based javascript testing framework. 
 
 Let's go ahead, and test the function with a valid input. 
After adding a function call ```divide(2,2)```, the file should look like below

``` javascript
  function divide(dividend,divisor)
  {
    if(divisor === 0)
      throw new Error("can't divide by zero");
    return dividend/divisor;
  }
  divide(2,2);
```
 Now run in the node ```node divideTest.js```, and it should run without any errors.

Now let's divide by zero by calling ```divide(2,0)``` this should throw error.
The file should now look like

``` javascript
  function divide(dividend,divisor)
  {
    if(divisor === 0)
      throw new Error("can't divide by zero");
    return dividend/divisor;
  }
  divide(2,2);
  divide(2,0);
```
    Let's run this file now in the node, and you should get an error as shown below.

 {%img /images/articles/javascript/nodejs/using-assert/divide_by_zero.png %}

  Now, we have a code, that throws for sure, an error.

 When we are implementing exception scnearios, we will have to verify code under test throws expected exception. 
we can use ```assert.throws``` to test an exception. In this scenario, we are expecting the code under test to throw an exception. 
In this example, we are expecting ```divide(2,0)``` to throw *can't divide by zero* exception. 

## assert.throws
 
   ```assert.throws``` takes a function that needs to be monitored for an exception, and an expectation. This expectation can be simply ```Error``` to match the error type, or a regex */can't be divide by zero/* or a function to do more expectation checking.
Let's see these in examples:

##### Expecting any Error
```
assert.throws( function(){divide(2,0);}, Error);
```
##### Expecting a particular error message using RegEx
```
assert.throws( function(){divide(2,0);}, /can't divide by zero/);
```

##### Expection in a function 
```
assert.throws( function(){divide(2,0);},function(error) {
  //any logic to verify the expectation
  return /can't divide by zero/.test(error); 
});
```
    
You can use any one of these three approaches to test now.
I am using second approach, which is using a RegEx.
Then, the file should look

``` javascript
  function divide(dividend,divisor)
  {
    if(divisor === 0)
      throw new Error("can't divide by zero");
    return dividend/divisor;
  }
  divide(2,2);
  assert.throws(function() { divide(2,0);}, /can't divide by zero/);
```
 
Let's run this file in the node. Now it should run successfully. 

 If you are wondering why call to divide function needs to be wrapped in another function, you are in the right place. Let's try running the file after removing this wrapper function. You should see an error similar to what you have seen earlier.
 
 {%img /images/articles/javascript/nodejs/using-assert/divide_by_zero.png %}

  When you pass divide(2,0) directly to assert.throws, what assert.throws receives is a return value of the function. By then, the call has already been executed with an exception.
So that's why, the function under test should be wrapped to use it in ```assert.throws```.

  We have seen how we can use ```assert.throws``` to expect an exception in a test scenario.
  
---
 Last updated: 08/07/2013
