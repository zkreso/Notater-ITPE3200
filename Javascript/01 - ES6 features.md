# ES6 features

Some useful features introduced with ES6 (2015)

### let og const

let og const replace var. let has block scope, while var had (has) function scope. Block scope is what java and most other programming languages use. let and const also follow the rules for global scope.

### Template literals

Template literals allow us to add javascript in the middle of a string. One uses backticks instead of single or double quotes. Inside the string, `${}` is the syntax to insert the javascript code.

```js
let name = "Bucky";

const myString = `My favorite person is ${name} because he is kind.`;
```

Note that you don't need to have strings inside the curly braces, you can have any javascript. For example 2 + 2 or anything else.

### Spread operator

The spread operator `...` is a way to select elements from an array.

```js
function addNumbers(a, b, c) {
    return a + b + c;
}

var nums = [3, 4, 7];
addNumbers(...nums); // 14

var meats = ["bacon", "ham"];
var food = ["apples", ...meats, "kiwi", "rice"];
// ["apples", "bacon", "ham", "kiwi", "rice"]
```

The spread operator also works with objects, but it's used with curly braces. It will combine all the properties of two objects in to one array.

```js
const newObject = { ...object1, ...object2 };
```

This will take all the properties of object1 and object2 and combine them in newObject. If the objects have any properties with the same name, they will retain the value of the last object (here object2).

Finally, one can use ...rest to get the rest of the objects in an array if one is using destructuring.

### Array/object destructuring

```js
const alphabet = ["A", "B", "C", "D", "E", "F"];

const a = alphabet[0];
const b = alphabet[1];

// The above done but with destructuring:
const [a, b] = alphabet;
```

The position of the elements in the array will determine which variable gets what value. If you want to skip an element, you can leave out the name, like this:

```js
const[a,, c] = alphabet;
```

You can also get the rest of the array with a spread operator. It will return the remaining elements as a new array.

```js
const[a,, c, ...rest] = alphabet;
```

One common use for this is to get only the value you want from a funtion that returns multiple values in an array.

One can also set default values if no value is present with an equal sign.

```js
const [a, b, c = "C"];
```

In case a third value does not exist in the array, the const c will get the value C.

Destructuring becomes even more useful because you can do it with objects. The difference is that you use curly braces instead, and the variables are not based on position, but on variable name. That means the variable names have to match.

```js
const { name, age } = personTwo;
```

This will create a new object containing the properties and values of name and age in the personTwo object.

One can also change the property name in the same line when selecting it:

```js
const { name: firstName, age } = personTwo;
```

This will create a new object with a property firstName with the value of name from the personTwo object, and with a property age with the value of age from the personTwo object. Admittedly, this can get a bit confusing if you are using TypeScript.

It's also possible to use default values as in array destructuring. And ...rest will also work. If you want to get values from nested objects:

```js
const { name, address: { city } } = personTwo;
```

This will create two variables (constants) with the name name and city.

Finally, you can do this when calling functions, which is probably the most common use for this:

```js
function printUser({name, age}) {
    console.log(`Name is: ${name}. Age is ${age}`);
}
```

As long as we pass an object with name and age properties, this will work. It's also possible to use default values in line like shown before.

### Short circuiting

This isn't really a ES6 feature, but it's possible to use logical operators to write shorter if statements.

```javascript
true && function() // Function will run
false && function() // Function will not run

false || function() // Function will run
true || function() // Function will not run

// Compare to null coalescing operator

const foo =false ?? "default" // foo: false
const foo = null ?? "default" // foo: "default"

const foo = false || "default" // foo: "default"
const foo = null || "default" // foo: "default"
```

#### Optional chaining

```javascript
const a = foo?.bar?.baz 
// Will assign baz to a only if foo and bar are defined
// Otherwise will assign undefined

// Also works with arrays
const a = foo.?[0] // Only assigns to a if the array exists
```

#### Console stuff

```javascript
// Print a table
console.table([foo, bar, baz]) // Where foo, bar and baz are objects of the same type

// Measure how long something took
console.time('looper')

// some function

console.timeEnd('looper')
```
