# An Opinionated Guide To Learning And Using Typescript

Most people learn typescript by looking through [the typescript handbook](https://www.typescriptlang.org/docs/handbook/intro.html). The handbook is a fantastic resource, which does its job very well. One of its best features is that it is comprehensive and unopinionated, making it a great reference text.

This document aims to be short(ish) and opinionated. It tells you what you need to know to starting working in a codebase (especially the things that can be confusing about typescript) and some principles guiding how typescript (and types more generally) should be used. The goal is to be terse, so you can get to work in the codebase more quickly. That said, there's a lot of interesting thing to say about types, so we provide links to interesting material if you want more detail.

Let's get started.

*Additional Links:*
-   [The typescript handbook](https://www.typescriptlang.org/docs/handbook/intro.html)
-   See especially [the getting started section](https://www.typescriptlang.org/docs/handbook/intro.html#get-started)
-   You may want to create your own typescript project to follow along with this document. Here's [how to install and use the typescript compiler](https://www.typescriptlang.org/download)
-   [Why typescript](https://www.typescriptlang.org/why-create-typescript) is also an interesting, and perhaps under-visited, page

## Basics
### Basic Types
Most people would say javascript doesn't have types. That's not strictly true. The [ES5.1 spec](https://262.ecma-international.org/5.1/#sec-8) defines 6 types present in javascript: `undefined`, `null`, `boolean`, `string`, `number`, and `object`. (The [more recent specs](https://262.ecma-international.org/) define even more types!)

The reason most people would say javascript doesn't have types is because those types are not explicitly annotated, there is no static type checking (i.e. javascript won't throw an error before runtime), and we coerce values implicitly (`"42" + 2 === "422"`). However, under the hood, the interpreter which is executing the javascript code (e.g. [V8](https://v8.dev/)) actually [does use types](https://www.youtube.com/watch?v=p-iiEDtpy6I).

Typescript makes types explicit, allows for static type checking, and doesn't allow us to coerce values implicitly.

For any of the basic types in javascript, we can tell the typescript compiler the type of something using a `:`. 
For example: 
```typescript
let x: string
x = "hello" // valid
const y = x + 2 // compile time error
x = 2 // compile time error
```

In most cases, typescript can infer the type of a variable. So if we write
```typescript
let x = "hello"
x = "world" // valid
x = 2 // compile time error
```
The type of `x` is inferred to be a `string`. Things would work analogously if we made `x` a `number`, `boolean`, `null`, etc.

Similarly, if we write something like
```typescript
const x = "hello"
const y = x + " world"
```
then `y` is inferred to be a `string`.

When the typescript compiler cannot determine the type of something, it will throw a compile time error (assuming the correct settings are set for the compiler, namely [--strict](https://www.typescriptlang.org/tsconfig#strict)). This makes it easy to write typescript code, as you only need to annotate the types which are not obvious to the compiler.

### Types Added By Typescript
However, typescript is much more powerful than just making the types in javascript explicit. It also adds a host of other types which are useful.

Here's a list of the types you'll probably use. (There are also other types, but these are most frequent):
-   [Literals](https://www.typescriptlang.org/docs/handbook/2/everyday-types.html#literal-types)
    -   See also [Enum types](https://www.typescriptlang.org/docs/handbook/2/everyday-types.html#enums), which are similar to literals. Roughly speaking, the difference is that in enums, each literal is associated with a number, and you can iterate through all elements of an enum, but not of a literal.
-   [Array](https://www.typescriptlang.org/docs/handbook/2/everyday-types.html#arrays)
    -   See also the [tuple type](https://www.typescriptlang.org/docs/handbook/basic-types.html#tuple), which is used for arrays with a fixed number of elements that have fixed types
-   [Function](https://www.typescriptlang.org/docs/handbook/2/everyday-types.html#functions)
    -   Each parameter of the function is typed with a and the return type is annotated with a after the parens
    -   In anonymous functions, the return type is annotated with a `=>` instead of a `:`
        -   For example:
            
            ```typescript
            // Pay attention to the type of `f`
            function apply (f: (a: number, b: number) => String, x: number, y: number): String {
              return f(x, y)
            }
            ```
            
        -   Another example:
            
            ```typescript
            // valid
            const f: (x: number, y: number) => String = (x, y) => String(x) + String(y)
            
            // compile time error
            const f: (x: number, y: number): String = (x, y) => String(x) + String(y)
            ```
            
    -   See also [more on functions](https://www.typescriptlang.org/docs/handbook/2/functions.html) for how to deal with variadic functions, generics, etc.
-   [Any](https://www.typescriptlang.org/docs/handbook/2/everyday-types.html#any)
    -   The broadest possible type (any syntactically valid javascript will work with the any type)
    -   In general, you don't want to use these, but they're helpful as an intermediate step in converting untyped javascript to typescript
    -   Occasionally used when you don't know what value's inside a variable
    -   See also [Unknown](https://www.typescriptlang.org/docs/handbook/basic-types.html#unknown)

## Combining Types
Most of the usefulness of types comes from the ability to combine them in interesting ways. Defining your own types (called type aliases) is easy in typescript, just use the `type` keyword.
```typescript
type version = "old" | "current" | "new"
type NumsToString = (a: number, b: number) => String
```
A basic extension of DRY to types: If you use a type in many places, create a type alias.

### Object Types
What's the type of the following?: 
```typescript
person1 = {
  age: 42,
  name: "ARTHUR DENT"
}
```

Typescript automatically infers the type `{age: number, name: string}`
We can also define similar types explicitly, where the first element of each pair is the name of the field and the second element is the type, e.g. 
```typescript
type Person = {
  age: number,
  name: string
}
```

If we have an object where not all the fields are known, we can use the [Record type](https://www.typescriptlang.org/docs/handbook/utility-types.html#recordkeystype) 
```typescript
type PhoneBook = Record<string, PhoneNumber>

// Equivalent to
type Phonebook = {
    [name: string]: phoneNumber
}
```

### 2 Operators
If you want a variable which is one type OR another, you can use the `|` operator. For example:
```typescript
// A String or a number
let x: string | number
x = "hello" // valid
x = 2 // valid
```

This is called a union type. Literals are most useful when combined with unions.

If you want a variable which is one type AND another, you can use the `&` operator -- namely, the new type will contain all the fields of the types used to create it. For example:
```typescript
type Person = {
  age: number,
  name: String,
  data: any
}

type Graded = {
  id: number
  data: Grade[]
}

type Student = Person & Graded

// Equivalent to:
type Student = {
  age: number,
  name: String,
  id: number,
  data: Grade[]
}
```

This is called an intersection type. If two object types share a field, their intersection will only contain that field once.

If a shared field has a narrower type in one object than another, the field in the intersection is the narrower of the two types:
```typescript
type Pair = {x: any, y: any}
type Point = {x: number, y: number}

Pair & Point === {x: number, y: number}
```

If a shared field has types in the two objects which have no shared elements, the field in the intersection is the `never` type
```typescript
type containsString = {x: string}
type containsNumber = {x: number}

containsString & containsNumber === {x: never}
```

The behavior for intersecting two types which are not objects is used much less frequently, and works somewhat differently. The intersection of two non-object types is the broader of the two types; for example `
string & any === any`.

*Additional Links:*
-   These two operators are powerful enough to express *any* type you want in typescript. This is true for the same reason that traditional and are powerful enough to express any logical statement.
    -   Nonetheless, typescript provides additional ways of combining types just to make our lives easier (see the [utility types page of the typescript handbook](https://www.typescriptlang.org/docs/handbook/utility-types.html)). In general, you won't need these, and if you use them (with the exception of the type) you're probably over complicating something. But on occasion they may be helpful.
-   The fact that these two operations are enough is useful to know. For some less immediately useful (but very interesting) mathematics about these operators, see the following:
    -   Types constructed in this way are called Algebraic Data Types (ADTs). You can learn more about them with the article [The algebra (and calculus!) of algebraic data types](https://codewords.recurse.com/issues/three/algebra-and-calculus-of-algebraic-data-types)
        -   Note: in most languages, i.e. the intersection type is a pair of values. It works differently in typescript because javascript makes such heavy use of objects that it made sense to specialize the behavior of the type to better suit objects. In this article, `(a, b)` is roughly the equivalent of `&` in typescript
    -   For the more mathematically inclined, once you've read the above article, you can see why this is true by reading [some abstract algebra](https://pavpanchekha.com/blog/zippers/derivative.html#sec-2)
    -   And for the connection to logic (and category theory), the extremely mathematically inclined can read about the Curry-Howard-Lambek isomorphism.
-   There are some types which can't be encoded in typescript (for example: higher kinded types -- which are analogous to higher order logics -- or refinement types). For most practical purposes, however, this won't be an issue.

## Some Advanced Concepts

### [Generics](https://www.typescriptlang.org/docs/handbook/2/generics.html#generic-types)
Generics let you use "type variables" when defining types.
For example, let's say you have a system which produces objects, and occasionally, the objects might have errors. You might write
```typescript
type PersonWithError = Person & {error: string}
type StudentWithError = Student & {error: string}
type CarWithError = Car & {error: string}
```

We can use generics to make this DRY-er:
```typescript
type WithError<T> = T & {error: string}

// PersonWithError === WithError<Person>
// StudentWithError === WithError<Student>
// CarWithError === WithError<Car>
// ... and analogously for any future types
```
The `<T>` "declares" the type variable `T`, so we can then use just as we would use any other type. (We could have called it anything `Type`, `BaseObject`, `MonkeyFlyingOverARainbow`, etc.)

We can also use generics with functions:
```typescript
// Example: The type for mapping a function over an array
function map<T, U>(arr: T[], f: (b: T) => U): U[] {
  // We can also use the types T and U in the body of the function
  let intermediateVariable: T  
  // ...
}
```
Use generics when you have a relationship you want to encode in the type system (i.e. two things have the same type or we need a narrow type). If there isn't a specific relationship, just use `any`.

### Recursive Types

How would you represent a tree in typescript?

One attempt might be something like this: 
```typescript
type AttemptedTree1<LEAF> = {children: LEAF[]}
```

That doesn't quite work. The tree is only one layer deep. It can't represent a tree with a depth greater than 1, like this:
```typescript
// Compile time error
let tree: AttemptedTree1<String> = {
  children: [
    "leaf in layer 1",
    {
      children: [
        "leaf in layer 2"
      ]
  	}
  ]
}
```

A naive solution is to try adding in another layer manually 
```typescript
type AttemptedTree2<LEAF> = {
  children: (LEAF | {children: LEAF})[]
}
```
If we do that, we simply run into the same problem one layer deeper. Does that mean it's impossible to write the type of an arbitrary tree? No!

Instead, we define a tree recursively in terms of itself -- every child in a tree is either a leaf, or a subtree: 
```typescript
type Tree<LEAF> = {
  children: (LEAF | Tree<LEAF>)[]
}
```

This is useful, for example, when typing JSON -- every field in a JSON object is a value (`String`, `number`, etc.), array, or a JSON object.

There are other ways of writing recursive types too:
```typescript
// A tree where naked leaves are also considered trees
type Tree<LEAF> = LEAF | Tree<LEAF>[]

// A tree built using Records
type Tree<LEAF> = Record<String, (LEAF | Tree<LEAF>)>

// A tree built using two types
type Tree<LEAF> = LEAF | TreeHelper<LEAF>
type TreeHelper<LEAF> = Tree<LEAF>[]

// etc.
```

A recursive type will error out if we reference the type directly in its definition (rather than an array or Record of the thing). This is an infinite loop in the type system!
```typescript
// compile time error
type invalidTree<LEAF> = LEAF | invalidTree<LEAF>
```

### [Narrowing](https://www.typescriptlang.org/docs/handbook/2/narrowing.html)
-   See also [typeguards](https://www.typescriptlang.org/docs/handbook/advanced-types.html#type-guards-and-differentiating-types)
-   Narrowing and typeguards are useful, and can be somewhat confusing, so make sure to re-read these and make sure you understand them

### [Coercing types](https://www.typescriptlang.org/docs/handbook/2/everyday-types.html#type-assertions)

# General Principles
When writing typescript, the most important principle to keep in mind is this: **Types are for people, not just the compiler**.
 
 ## How To Name Types

### Capitalization
Often times, people name their variables based on the type of that variable. This is a good practice, as it makes the name of the variable explanatory and generally more readable.
For example, if we have a function which takes a number, we might write 
```javascript
function triple(number) {
  return 3 * number
}
```
(As a sidenote, it is probably more readable to write `triple(aNumber)` than it is to write `triple(number)`, and variables named after their types should generally be preceded by an `a`).

Unfortunately, naming variables based on their type can be confusing when using typescript: it can become hard to distinguish between the variable and the type. E.g. 
```typescript
let number: number
let x: number
let y: number
```
In order to avoid this, all user-defined types should be written in `PascalCase` (camelCase with the first letter capitalized). That way, we can tell at a glance whether something is a type or a variable. E.g.
```typescript
type Person  = {
  // ...
}

for (const person of people) {
  let newPerson: Person
  // ...
}
```
Many languages (e.g. Haskell) enforce `PascalCase` for types. It is notable that the basic types in typescript do not follow this pattern. `number` and `Number` are semantically different. Same for `string` and `String`. Ditto for many other types with a lowercase and uppercase variant. In most cases, the lowercase version is the right one. Some builtin types only have a lowercase variant, such as `any` or `null`.

So, it is up to you as a developer to write your types `TheRightWay`.

### Renaming Basic Types
Sometimes it makes sense to rename a type which already exists.

For example, let's say you have a library which logs messages, where all those messages are `string`s. It makes no difference to the compiler if you make a new type alias `type Message = string`. However, to a human reader, that alias can help distinguish between string manipulation and logging, or make more clear what the library is doing.

Another example can be found when converting an existing codebase from javascript to typescript.

To distinguish `any`-as-in-unconverted from `any`-because-TS-can't-type-this, you can export a distinguished type alias `type Unconverted = any` from one module. Then you can immediately search for either places that module is imported or uses of the distinguished name, and find places new types need to be written without any false positives where someone's already thought it through and decided `any` is necessary for real.

`Record<string, any>` (or an `UnconvertedRecord` or `AnyRecord` alias) is a slightly safer alternative to `any` when you mean "an object, dunno what's in it"; it's compatible with all objects but not with primitives.

## How to organize types
In theory, types can go anywhere in any of your files -- compiler doesn't care. In practice, you should adopt the same standards that are used in Haskell:

-   Generally, put types at the top of a file.
-   If every major function that operates on a type is going in one file anyway, put the type in there too -- it doesn't make sense to have to import the type separately. (In TS-land I think of this as the "well it's not a class but it _could_ be" situation -- the type is bundled up with its supporting code the way classes are.)
-   Types representing some domain concept that's used in many places go in a file along with the canonical functions for working with that type -- printing/validation/combiner/constructor/utility functions, but not business logic.
-   Multiple types that collectively define some API may be together in one file, especially if they're closely interconnected (cyclic types have to be defined together, usually). In this case the file probably has nothing but types, or types and brainlessly-simple utility functions (isX type guard functions, Redux-style x() constructors). Usually this turns the one big types file into the main entrypoint for new people reading the code, for better or for worse, so you hopefully organize and comment it in a way that makes it a good overview/introduction to what all the types _mean_ as well as what they are.

## How to refactor with types

Types add new sets of what Martin Fowler, in his very good textbook *[Refactoring](https://martinfowler.com/books/refactoring.html)*, calls "code smells". These are things in your code that indicate something funky is going on and that you might want to refactor.

### Based on "How to name types"
Often times, variables are named after their types. If you are adding types to a file, you can borrow the names of the types from the names of the variables (assuming the variables are named well).

Conversely, if you have a well-named type, and you have a variable of the type, you're often better off renaming the variable based on the type. E.g. `let x: Person` turns into `let aPerson: Person`.

### Based on "How to organize types"
Files which have functions operating on two different sets of types should smell funky. This type smell is a violation of the Single Responsibility Principle. Things of different types should be split into different files (with the type declarations at the top of the file).

A particular egregious example of this is when you find yourself defining new types halfway through a file in order to type a set of functions.

Conversely, if you keep seeing the same type at many different points in your codebase, maybe the things relating to that type want to live together in one place.

## How NOT to make types
Generally speaking, [simplicity is better than ease](https://www.infoq.com/presentations/Simple-Made-Easy/). This is because simple things (while potentially hard to come up with) are faster to understand and to change. Types are no different. Whenever possible, aim for simpler types and simpler mechanisms for creating types.

### Interface vs Type Alias

Typescript provides two ways of defining object types. The first is using the `type` keyword, and the second is using the `interface` keyword. Generally speaking, these things are the same (they express the same concept and are equally powerful).

The main difference between `type`s and `interface`s is that you can only define a type in one place. With an interface, you can keep adding to the definition, either at multiple points in a file or across multiple files.

This makes `interface`s far more complex. Rather than having to look at one place to understand the type, you have now interwoven multiple definitions, usually unnecessarily. Multiple files can thus become entangled in non-obvious or unneeded ways. So, you should generally avoid `interface` and use `type`. (That's why this guide doesn't mention them elsewhere.)

If you want to learn more about `interface`s, see the [relevant page in the typescript handbook](https://www.typescriptlang.org/docs/handbook/2/everyday-types.html#interfaces)

### Utility Types

Typescript provides many [utility types](https://www.typescriptlang.org/docs/handbook/utility-types.html) to help you construct new types. You should think twice before using these -- while they make things easier, they often make things more complex. That said, these are the right solution a significant percent of the time, so you should make an effort to understand them and whether your particular use will make things simpler or just easier.

## A Final Note
Hopefully this guide has given you a sense of not just how to use typescript, but *what types are about*. They are useful tools for a programmer. They find errors for you. They help make your code easier to read and understand. They aid in refactoring. Often times, I find that one of the easiest ways to come to grips with a large codebase is to start adding types, and then to start refactoring based on the type smells.
But getting the most out of typescript -- and out of types in generall -- requires using those types intelligently and using them well. They are a forcing function to think about what your code is actually doing. If you write the types and type signatures of all the pieces in your code (variables, functions, etc.) before you starting implementing them, you'll often find that the code ends up far better than it otherwise would have. And if you use types haphazardly, you'll sometimes find that you've created a mess which is worse than what you started with. If for no other reason, it's useful to use typescript to get a better understanding of types so that you can use them well; that way, you come closer to mastery of a powerful tool. (Although, once you've done a bit of that, you should really check out Haskell and Coq.) This guide is a good starting place for understanding the principles around using types well... but there's no better way to understand a tool than to use it yourself!
