# Refactoring for Clean Code

Ensure that you are writing and refactoring code for maintainability.

**Lean into Object Oriented Programming (OOP).**

Write code as a collection of interacting objects, methods, and functions with clear responsibilities rather than a mere list of instructions.

**Owners of responsibility**

When you encounter procedural code, shift your focus from how a task is done to what object, method, or function is responsible for doing it.

- A class is useful when there is durable state, a lifecycle, caching, or an API that benefits from grouping related operations.
- A module is enough when the responsibility needs private helpers but no per-instance state.
- A plain helper function is enough when the behavior is stateless, has no hidden invariants, and can be named precisely.
- Data that carries important invariants should usually be manipulated through the module or object that owns those invariants.
- Think of relationships between data and concepts as objects with their own responsibilities and capabilities.

**Use encapsulation**

In perfect code, each object is completely independent of the others: You could work in any of the objects without knowing anything about any of the other objects.

Encapsulation is also known as "information hiding". The information hidden within a module usually consists of details about how to implement some mechanism which other parts of the system don't need to know about.

Encapsulate your code in two ways:

1. Your interfaces - classes, method signatures, function signatures - should reflect a simpler, more abstract view of the object's functionality and hide the details.
2. There are no dependencies on an object's, method's, or function's internal information from outside it.

**Separate specialized code from general-purpose code.**

User interface code should be separated from general purpose domain code:

- Handlers, views, and templates in a web application
- A command line interface or terminal user interface

Low level adapters and drivers should be separated from general purpose domain code:

- Database adapters or mappings
- File system abstractions
- HTTP API clients

**Smaller modules, classes, and methods are NOT always better.**

Creating small classes, methods, and functions is NOT your goal when writing or refactoring code; making the code simpler is your goal. There are times when you should bring code together to make it simpler instead of decomposing it.

Bringing pieces of code together when:

- They share information; for example, both pieces of code might depend on information about a common protocol.
- They are used together: anyone using one of the pieces of code is likely to use the other as well.
- They overlap conceptually, in that there is a simple higher-level category that includes both of the pieces of code.
- It is hard to understand one of the pieces of code without looking at the other.

**No thin wrapper classes.**

Avoid writing classes which are just thin wrappers over some data with getters and setters.

This is bad:

```javascript
class Dog {

    #color;
    #weight;
    #breeds;

    constructor(args) {
        this.#color = args.color;
        this.#weight = args.weight;
        this.#breeds = args.breeds;
    }

    get color() { return this.#color; }

    get weight() {
        return Number.parseFloat(this.#weight);
    }

    get breeds() {
        return this.#breeds.join(', ');
    }
}
```

Don't write useless getters and setters like that. When you see it, either remove them, or remove the whole class if it serves no other purpose. Just operate on the plain JavaScript object, and maybe include a JSDoc @typedef type definition for it.

This is better:

```javascript
class Dog {

    constructor(args) {
        this.color = args.color;
        this.weight = Number.parseFloat(args.weight);
        this.breeds = args.breeds.join(', ');
    }
}
```

Or use a type definition:

```javascript
/**
 * @typedef
 * @property {string} color - The common color of the dog
 * @property {number} weight - The weight of the dog expressed as a float
 * @property {string} breeds - The common breeds of the dog expressed as a comma separated list
 */

/**
 * @return {Dog}
 */
function createDog(args) {
    return {
        color: args.color,
        weight: Number.parseFloat(args.weight),
        breeds: args.breeds.join(', '),
    };
}
```

**Methods and functions should do one thing.**

The methods and functions you write should do one thing and only that thing.

A method should change the state of an object, or it should return some information about that object.

In order to make sure our functions are doing one thing, you need to make sure that the statements within your method are all at the same level of abstraction.

**Minimize the number of arguments for methods and functions.**

Use the fewest number of arguments to a method or function as possible. More than three arguments should not be used. When a method seems to need more than two or three arguments, it is likely that some of those arguments ought to be wrapped into a class or object of their own.

**Name precisely.**

Constants, classes, methods, and functions should read so that a future reader has no surprises about what they are or do. Prefer nouns for classes that model a thing (`ApplicationConfig`, `SourceBundle`) and verb phrases for classes that perform an action (`WorkerDeployment`, not `WorkerModuleHandler`). If a name needs a comment to explain it, rename instead of commenting.
