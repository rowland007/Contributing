# Introduction

[![XKCD](../static/img/code-quality.png)](https://www.xkcd.com/1513/)

This first page is a guide to producing [readable, reusable, and refactorable](https://github.com/ryanmcdermott/3rs-of-software-architecture) software that should be language independent.

!!! info
    This is based on [Ryan McDermott's clean-code-javascript](https://github.com/ryanmcdermott/clean-code-javascript#functions) that was written in December 2016. Some of the principles may be very JavaScript specific. Most of the principles will apply to all languages and this is a good working base to start from.

## Readability

Readability is the simplest way of assessing code quality and it's the most straightforward to fix. It is the most obvious thing you see right when you open up a piece of code, and it generally consists of:

* Formatting
* Variable names
* Function names
* Amount of function arguments
* Function length (number of lines)
* Nesting levels

These aren't the only things to consider, but they are immediate red flags.

## Reusability

Reusability is the sole reason you are able to read this code, communicate with strangers online, and even program at all. Reusability allows us to express new ideas with little pieces of the past.

That is why reusability is such an essential concept that should guide your software architecture. We commonly think of reusability in terms of DRY (Don't Repeat Yourself). That is one aspect of it -- don't have duplicate code if you can abstract it properly. Reusability goes beyond that though. It's about making clean, simple APIs that make your fellow programmer say, "Yep, I know exactly what that does!" Reusability makes your code a delight to work with, and it means you can ship features faster.

## Refactorability

Code that is refactorable is code that you can change without fear. It's code that you can deploy on a Friday night, and come back to on Monday morning without any concern that your users encountered runtime errors.

Refactorability is about the system as a whole. It's about how your reusable modules connect together like LEGO pieces. If you change your Employee module and somehow it breaks your Reporting module, then you know you have some refactorability issues. Refactorability is the highest piece of the 3 R hierarchy, and it's the hardest to achieve and maintain. There will always be issues with any human system, and code is no different. However, there are things that we can do to make our code refactorable:

* Isolated side effects
* Tests
* Static types

## **SOLID**

### Single Responsibility Principle (SRP)

As stated in Clean Code, "There should never be more than one reason for a class
to change". It's tempting to jam-pack a class with a lot of functionality, like
when you can only take one suitcase on your flight. The issue with this is
that your class won't be conceptually cohesive and it will give it many reasons
to change. Minimizing the amount of times you need to change a class is important.
It's important because if too much functionality is in one class and you modify
a piece of it, it can be difficult to understand how that will affect other
dependent modules in your codebase.

!!! failure "Bad"
    ```javascript
    class UserSettings {
        constructor(user) {
            this.user = user;
        }

        changeSettings(settings) {
            if (this.verifyCredentials()) {
                // ...
            }
        }

        verifyCredentials() {
            // ...
        }
    }
    ```

!!! success "Good"
    ```javascript
    class UserAuth {
        constructor(user) {
            this.user = user;
        }

        verifyCredentials() {
            // ...
        }
    }

    class UserSettings {
        constructor(user) {
            this.user = user;
            this.auth = new UserAuth(user);
        }

        changeSettings(settings) {
            if (this.auth.verifyCredentials()) {
                // ...
            }
        }
    }
    ```

### Open/Closed Principle (OCP)

As stated by Bertrand Meyer, "software entities (classes, modules, functions,
etc.) should be open for extension, but closed for modification." What does that
mean though? This principle basically states that you should allow users to
add new functionalities without changing existing code.

!!! failure "Bad"
    ```javascript
    class AjaxAdapter extends Adapter {
        constructor() {
            super();
            this.name = "ajaxAdapter";
        }
    }

    class NodeAdapter extends Adapter {
        constructor() {
            super();
            this.name = "nodeAdapter";
        }
    }

    class HttpRequester {
        constructor(adapter) {
            this.adapter = adapter;
        }

        fetch(url) {
            if (this.adapter.name === "ajaxAdapter") {
                return makeAjaxCall(url).then(response => {
                    // transform response and return
                });
            } else if (this.adapter.name === "nodeAdapter") {
                return makeHttpCall(url).then(response => {
                    // transform response and return
                });
            }
        }
    }

    function makeAjaxCall(url) {
        // request and return promise
    }

    function makeHttpCall(url) {
        // request and return promise
    }
    ```

!!! success "Good"
    ```javascript
    class AjaxAdapter extends Adapter {
        constructor() {
            super();
            this.name = "ajaxAdapter";
        }

        request(url) {
            // request and return promise
        }
    }

    class NodeAdapter extends Adapter {
        constructor() {
            super();
            this.name = "nodeAdapter";
        }

        request(url) {
            // request and return promise
        }
    }

    class HttpRequester {
        constructor(adapter) {
            this.adapter = adapter;
        }

        fetch(url) {
            return this.adapter.request(url).then(response => {
                // transform response and return
            });
        }
    }
    ```

### Liskov Substitution Principle (LSP)

This is a scary term for a very simple concept. It's formally defined as "If S
is a subtype of T, then objects of type T may be replaced with objects of type S
(i.e., objects of type S may substitute objects of type T) without altering any
of the desirable properties of that program (correctness, task performed,
etc.)." That's an even scarier definition.

The best explanation for this is if you have a parent class and a child class,
then the base class and child class can be used interchangeably without getting
incorrect results. This might still be confusing, so let's take a look at the
classic Square-Rectangle example. Mathematically, a square is a rectangle, but
if you model it using the "is-a" relationship via inheritance, you quickly
get into trouble.

!!! failure "Bad"
    ```javascript
    class Rectangle {
        constructor() {
            this.width = 0;
            this.height = 0;
        }

        setColor(color) {
            // ...
        }

        render(area) {
            // ...
        }

        setWidth(width) {
            this.width = width;
        }

        setHeight(height) {
            this.height = height;
        }

        getArea() {
            return this.width * this.height;
        }
    }

    class Square extends Rectangle {
        setWidth(width) {
            this.width = width;
            this.height = width;
        }

        setHeight(height) {
            this.width = height;
            this.height = height;
        }
    }

    function renderLargeRectangles(rectangles) {
        rectangles.forEach(rectangle => {
            rectangle.setWidth(4);
            rectangle.setHeight(5);
            const area = rectangle.getArea(); // BAD: Returns 25 for Square. Should be 20.
            rectangle.render(area);
        });
    }

    const rectangles = [new Rectangle(), new Rectangle(), new Square()];
    renderLargeRectangles(rectangles);
    ```

!!! success "Good"
    ```javascript
    class Shape {
        setColor(color) {
            // ...
        }

        render(area) {
            // ...
        }
    }

    class Rectangle extends Shape {
        constructor(width, height) {
            super();
            this.width = width;
            this.height = height;
        }

        getArea() {
            return this.width * this.height;
        }
    }

    class Square extends Shape {
        constructor(length) {
            super();
            this.length = length;
        }

        getArea() {
            return this.length * this.length;
        }
    }

    function renderLargeShapes(shapes) {
        shapes.forEach(shape => {
            const area = shape.getArea();
            shape.render(area);
        });
    }

    const shapes = [new Rectangle(4, 5), new Rectangle(4, 5), new Square(5)];
    renderLargeShapes(shapes);
    ```

### Interface Segregation Principle (ISP)

JavaScript doesn't have interfaces so this principle doesn't apply as strictly
as others. However, it's important and relevant even with JavaScript's lack of
type system.

ISP states that "Clients should not be forced to depend upon interfaces that
they do not use." Interfaces are implicit contracts in JavaScript because of
duck typing.

A good example to look at that demonstrates this principle in JavaScript is for
classes that require large settings objects. Not requiring clients to setup
huge amounts of options is beneficial, because most of the time they won't need
all of the settings. Making them optional helps prevent having a
"fat interface".

!!! failure "Bad"
    ```javascript
    class DOMTraverser {
        constructor(settings) {
            this.settings = settings;
            this.setup();
        }

        setup() {
            this.rootNode = this.settings.rootNode;
            this.animationModule.setup();
        }

        traverse() {
            // ...
        }
    }

    const $ = new DOMTraverser({
        rootNode: document.getElementsByTagName("body"),
        animationModule() {} // Most of the time, we won't need to animate when traversing.
        // ...
    });
    ```

!!! success "Good"
    ```javascript
    class DOMTraverser {
        constructor(settings) {
            this.settings = settings;
            this.options = settings.options;
            this.setup();
        }

        setup() {
            this.rootNode = this.settings.rootNode;
            this.setupOptions();
        }

        setupOptions() {
            if (this.options.animationModule) {
                // ...
            }
        }

        traverse() {
            // ...
        }
    }

    const $ = new DOMTraverser({
        rootNode: document.getElementsByTagName("body"),
        options: {
            animationModule() {}
        }
    });
    ```

### Dependency Inversion Principle (DIP)

This principle states two essential things:

1. High-level modules should not depend on low-level modules. Both should
   depend on abstractions.
2. Abstractions should not depend upon details. Details should depend on
   abstractions.

This can be hard to understand at first, but if you've worked with AngularJS,
you've seen an implementation of this principle in the form of Dependency
Injection (DI). While they are not identical concepts, DIP keeps high-level
modules from knowing the details of its low-level modules and setting them up.
It can accomplish this through DI. A huge benefit of this is that it reduces
the coupling between modules. Coupling is a very bad development pattern because
it makes your code hard to refactor.

As stated previously, JavaScript doesn't have interfaces so the abstractions
that are depended upon are implicit contracts. That is to say, the methods
and properties that an object/class exposes to another object/class. In the
example below, the implicit contract is that any Request module for an
`InventoryTracker` will have a `requestItems` method.

!!! failure "Bad"
    ```javascript
    class InventoryRequester {
        constructor() {
            this.REQ_METHODS = ["HTTP"];
        }

        requestItem(item) {
            // ...
        }
    }

    class InventoryTracker {
        constructor(items) {
            this.items = items;

            // BAD: We have created a dependency on a specific request implementation.
            // We should just have requestItems depend on a request method: `request`
            this.requester = new InventoryRequester();
        }

        requestItems() {
            this.items.forEach(item => {
                this.requester.requestItem(item);
            });
        }
    }

    const inventoryTracker = new InventoryTracker(["apples", "bananas"]);
    inventoryTracker.requestItems();
    ```

!!! success "Good"
    ```javascript
    class InventoryTracker {
        constructor(items, requester) {
            this.items = items;
            this.requester = requester;
        }

        requestItems() {
            this.items.forEach(item => {
                this.requester.requestItem(item);
            });
        }
    }

    class InventoryRequesterV1 {
        constructor() {
            this.REQ_METHODS = ["HTTP"];
        }

        requestItem(item) {
            // ...
        }
    }

    class InventoryRequesterV2 {
        constructor() {
            this.REQ_METHODS = ["WS"];
        }

        requestItem(item) {
            // ...
        }
    }

    // By constructing our dependencies externally and injecting them, we can easily
    // substitute our request module for a fancy new one that uses WebSockets.
    const inventoryTracker = new InventoryTracker(
        ["apples", "bananas"],
        new InventoryRequesterV2()
    );
    inventoryTracker.requestItems();
    ```

## **Testing**

Testing is more important than shipping. If you have no tests or an
inadequate amount, then every time you ship code you won't be sure that you
didn't break anything. Deciding on what constitutes an adequate amount is up
to your team, but having 100% coverage (all statements and branches) is how
you achieve very high confidence and developer peace of mind. This means that
in addition to having a great testing framework, you also need to use a
[good coverage tool](https://gotwarlost.github.io/istanbul/).

There's no excuse to not write tests. There are [plenty of good JS test frameworks](https://jstherightway.org/#testing-tools), so find one that your team prefers.
When you find one that works for your team, then aim to always write tests
for every new feature/module you introduce. If your preferred method is
Test Driven Development (TDD), that is great, but the main point is to just
make sure you are reaching your coverage goals before launching any feature,
or refactoring an existing one.

### Single concept per test

!!! failure "Bad"
    ```javascript
    import assert from "assert";
    describe("MomentJS", () => {
        it("handles date boundaries", () => {
            let date;

            date = new MomentJS("1/1/2015");
            date.addDays(30);
            assert.equal("1/31/2015", date);

            date = new MomentJS("2/1/2016");
            date.addDays(28);
            assert.equal("02/29/2016", date);

            date = new MomentJS("2/1/2015");
            date.addDays(28);
            assert.equal("03/01/2015", date);
        });
    });
    ```

!!! success "Good"
    ```javascript
    import assert from "assert";

    describe("MomentJS", () => {
        it("handles 30-day months", () => {
            const date = new MomentJS("1/1/2015");
            date.addDays(30);
            assert.equal("1/31/2015", date);
        });

        it("handles leap year", () => {
            const date = new MomentJS("2/1/2016");
            date.addDays(28);
            assert.equal("02/29/2016", date);
        });

        it("handles non-leap year", () => {
            const date = new MomentJS("2/1/2015");
            date.addDays(28);
            assert.equal("03/01/2015", date);
        });
    });
    ```

## **Concurrency**

### Use Promises, not callbacks

Callbacks aren't clean, and they cause excessive amounts of nesting. With ES2015/ES6,
Promises are a built-in global type. Use them!

!!! failure "Bad"
    ```javascript
    import { get } from "request";
    import { writeFile } from "fs";

    get(
        "https://en.wikipedia.org/wiki/Robert_Cecil_Martin",
        (requestErr, response, body) => {
            if (requestErr) {
                console.error(requestErr);
            } else {
                writeFile("article.html", body, writeErr => {
                    if (writeErr) {
                        console.error(writeErr);
                    } else {
                        console.log("File written");
                    }
                });
            }
        }
    );
    ```

!!! success "Good"
    ```javascript
    import { get } from "request-promise";
    import { writeFile } from "fs-extra";

    get("https://en.wikipedia.org/wiki/Robert_Cecil_Martin")
    .then(body => {
        return writeFile("article.html", body);
    })
    .then(() => {
        console.log("File written");
    })
    .catch(err => {
        console.error(err);
    });
    ```

### Async/Await are even cleaner than Promises

Promises are a very clean alternative to callbacks, but ES2017/ES8 brings async and await
which offer an even cleaner solution. All you need is a function that is prefixed
in an `async` keyword, and then you can write your logic imperatively without
a `then` chain of functions. Use this if you can take advantage of ES2017/ES8 features
today!

!!! failure "Bad"
    ```javascript
    import { get } from "request-promise";
    import { writeFile } from "fs-extra";

    get("https://en.wikipedia.org/wiki/Robert_Cecil_Martin")
    .then(body => {
        return writeFile("article.html", body);
    })
    .then(() => {
        console.log("File written");
    })
    .catch(err => {
        console.error(err);
    });
    ```

!!! success "Good"
    ```javascript
    import { get } from "request-promise";
    import { writeFile } from "fs-extra";

    async function getCleanCodeArticle() {
        try {
            const body = await get(
                "https://en.wikipedia.org/wiki/Robert_Cecil_Martin"
            );
            await writeFile("article.html", body);
            console.log("File written");
        } catch (err) {
            console.error(err);
        }
    }

    getCleanCodeArticle()
    ```

## **Error Handling**

Thrown errors are a good thing! They mean the runtime has successfully
identified when something in your program has gone wrong and it's letting
you know by stopping function execution on the current stack, killing the
process (in Node), and notifying you in the console with a stack trace.

### Don't ignore caught errors

Doing nothing with a caught error doesn't give you the ability to ever fix
or react to said error. Logging the error to the console (`console.log`)
isn't much better as often times it can get lost in a sea of things printed
to the console. If you wrap any bit of code in a `try/catch` it means you
think an error may occur there and therefore you should have a plan,
or create a code path, for when it occurs.

!!! failure "Bad"
    ```javascript
    try {
        functionThatMightThrow();
    } catch (error) {
        console.log(error);
    }
    ```

!!! success "Good"
    ```javascript
    try {
        functionThatMightThrow();
    } catch (error) {
        // One option (more noisy than console.log):
        console.error(error);
        // Another option:
        notifyUserOfError(error);
        // Another option:
        reportErrorToService(error);
        // OR do all three!
    }
    ```

### Don't ignore rejected promises

For the same reason you shouldn't ignore caught errors
from `try/catch`.

!!! failure "Bad"
    ```javascript
    getdata()
    .then(data => {
        functionThatMightThrow(data);
    })
    .catch(error => {
        console.log(error);
    });
    ```

!!! success "Good"
    ```javascript
    getdata()
    .then(data => {
        functionThatMightThrow(data);
    })
    .catch(error => {
        // One option (more noisy than console.log):
        console.error(error);
        // Another option:
        notifyUserOfError(error);
        // Another option:
        reportErrorToService(error);
        // OR do all three!
    });
    ```

## **Formatting**

Formatting is subjective. Like many rules herein, there is no hard and fast
rule that you must follow. The main point is DO NOT ARGUE over formatting.
There are [tons of tools](https://standardjs.com/rules.html) to automate this.
Use one! It's a waste of time and money for engineers to argue over formatting.

For things that don't fall under the purview of automatic formatting
(indentation, tabs vs. spaces, double vs. single quotes, etc.) look here
for some guidance.

### Use consistent capitalization

JavaScript is untyped, so capitalization tells you a lot about your variables,
functions, etc. These rules are subjective, so your team can choose whatever
they want. The point is, no matter what you all choose, just be consistent.

!!! failure "Bad"
    ```javascript
    const DAYS_IN_WEEK = 7;
    const daysInMonth = 30;

    const songs = ["Back In Black", "Stairway to Heaven", "Hey Jude"];
    const Artists = ["ACDC", "Led Zeppelin", "The Beatles"];

    function eraseDatabase() {}
    function restore_database() {}

    class animal {}
    class Alpaca {}
    ```

!!! success "Good"
    ```javascript
    const DAYS_IN_WEEK = 7;
    const DAYS_IN_MONTH = 30;

    const SONGS = ["Back In Black", "Stairway to Heaven", "Hey Jude"];
    const ARTISTS = ["ACDC", "Led Zeppelin", "The Beatles"];

    function eraseDatabase() {}
    function restoreDatabase() {}

    class Animal {}
    class Alpaca {}
    ```

### Function callers and callees should be close

If a function calls another, keep those functions vertically close in the source
file. Ideally, keep the caller right above the callee. We tend to read code from
top-to-bottom, like a newspaper. Because of this, make your code read that way.

!!! failure "Bad"
    ```javascript
    class PerformanceReview {
        constructor(employee) {
            this.employee = employee;
        }

        lookupPeers() {
            return db.lookup(this.employee, "peers");
        }

        lookupManager() {
            return db.lookup(this.employee, "manager");
        }

        getPeerReviews() {
            const peers = this.lookupPeers();
            // ...
        }

        perfReview() {
            this.getPeerReviews();
            this.getManagerReview();
            this.getSelfReview();
        }

        getManagerReview() {
            const manager = this.lookupManager();
        }

        getSelfReview() {
            // ...
        }
    }

    const review = new PerformanceReview(employee);
    review.perfReview();
    ```

!!! success "Good"
    ```javascript
    class PerformanceReview {
        constructor(employee) {
            this.employee = employee;
        }

        perfReview() {
            this.getPeerReviews();
            this.getManagerReview();
            this.getSelfReview();
        }

        getPeerReviews() {
            const peers = this.lookupPeers();
            // ...
        }

        lookupPeers() {
            return db.lookup(this.employee, "peers");
        }

        getManagerReview() {
            const manager = this.lookupManager();
        }

        lookupManager() {
            return db.lookup(this.employee, "manager");
        }

        getSelfReview() {
            // ...
        }
    }

    const review = new PerformanceReview(employee);
    review.perfReview();
    ```

## **Comments**

### Only comment things that have business logic complexity

Comments are an apology, not a requirement. Good code _mostly_ documents itself.

!!! failure "Bad"
    ```javascript
    function hashIt(data) {
        // The hash
        let hash = 0;

        // Length of string
        const length = data.length;

        // Loop through every character in data
        for (let i = 0; i < length; i++) {
            // Get character code.
            const char = data.charCodeAt(i);
            // Make the hash
            hash = (hash << 5) - hash + char;
            // Convert to 32-bit integer
            hash &= hash;
        }
    }
    ```

!!! success "Good"
    ```javascript
    function hashIt(data) {
        let hash = 0;
        const length = data.length;

        for (let i = 0; i < length; i++) {
            const char = data.charCodeAt(i);
            hash = (hash << 5) - hash + char;

            // Convert to 32-bit integer
            hash &= hash;
        }
    }
    ```

### Don't leave commented out code in your codebase

Version control exists for a reason. Leave old code in your history.

!!! failure "Bad"
    ```javascript
    doStuff();
    // doOtherStuff();
    // doSomeMoreStuff();
    // doSoMuchStuff();
    ```

!!! success "Good"
    ```javascript
    doStuff();
    ```

### Don't have journal comments

Remember, use version control! There's no need for dead code, commented code,
and especially journal comments. Use `git log` to get history!

!!! failure "Bad"
    ```javascript
    /**
    * 2016-12-20: Removed monads, didn't understand them (RM)
    * 2016-10-01: Improved using special monads (JP)
    * 2016-02-03: Removed type-checking (LI)
    * 2015-03-14: Added combine with type-checking (JR)
    */
    function combine(a, b) {
        return a + b;
    }
    ```

!!! success "Good"
    ```javascript
    function combine(a, b) {
        return a + b;
    }
    ```

### Avoid positional markers

They usually just add noise. Let the functions and variable names along with the
proper indentation and formatting give the visual structure to your code.

!!! failure "Bad"
    ```javascript
    ////////////////////////////////////////////////////////////////////////////////  
    // Scope Model Instantiation  
    ////////////////////////////////////////////////////////////////////////////////  
    $scope.model = {  
        menu: "foo",  
        nav: "bar"  
    };  

    ////////////////////////////////////////////////////////////////////////////////  
    // Action setup  
    ////////////////////////////////////////////////////////////////////////////////  
    const actions = function() {  
        // ...  
    };  
    ```

!!! success "Good"
    ```javascript
    $scope.model = {  
        menu: "foo",  
        nav: "bar"  
    };  

    const actions = function() {  
        // ...  
    };  
    ```
