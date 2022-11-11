# Typescript essentials

TypeScript is a superset of JavaScript. It contains all the features of JavaScript and more. It is "compiled" to JS. Oftentimes, projects using TypeScript will contain a config file with compile options, named tsconfig.json

### Implicit types

While TS is strongly typed, you don't necessarily have to define the type always. The interpreter will try to detect the type from the context. The variable will still be strongly typed.

### Explicit types

Of course, as with any strontly typed language, you can expliclitly define types. It is done with a : sign followed by the type. Examples:

```ts
let name: string = 'John';
let age: number = 30;
```

Commonly used built-in types are:
- Boolean
- Number
- String
- Array
- Tuple
- Enum
- Unknown
- Any
- Void
- Null
- Undefined

### Tuple

```ts
type: stringAndNumber = [string, number];

let x: stringAndNumber = ["Hello", 10];
```

### Enums

Enums are like in Java or C#

```ts
enum Continents {
    North_America,
    South_America,
    Africa,
    Asia,
    Europe,
    Antartica,
    Australia
}

var region = Continents.Africa // 2
```

Enums are auto incremented with 0 as the first value. Typescript will compile enum to a class with a function to access it.

### Interface

```ts
interface user {
    name: string;
    id: number;
}

const user: User = {
    name: 'John',
    id: 0,
}
```

### Union types

```ts
type WindowStates = "open" | "closed" | "minimized";

const windowState: WindowStates = "I don't know, this is not a window."; // This will not work

type LockStates = "locked" | "unlocked";
type OddNumberUnderTen = 1 | 3 | 5 | 7 | 9;

const odd: OddNumberUnderTen = 6; // This will not work

const getLength = (param: string | string[]) { // any[] also works for any type of array
    return param.length;
}

getLength('test'); // 4
getLength(['test', 'test1']); // 2
getLength(5); // will not work, and more importantly, interpreter will generate a warning
```