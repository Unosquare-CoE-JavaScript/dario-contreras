# Chapter 7: Using Closures 

Closure builds on this approach: for variables we need to use over time, instead of placing them in larger outer scops, we can encapsulate them but still preserve access from inside functions, for broader use. Functions remember these references scoped variables via closure. 

If you've ever written a callback that accesses variables outside its own scope, that's closure. 

Closure is one of the most important language characteristics ever invented in programming-it underlies major programming paradigmns. Getting confortable with closure is required for mastering JS and effectively leveraging many important design patterns throughout your code. 

##  See the closure 

Closure is originally a mathematical concept, from lambda calculus, however, we'll focus on a practical perspective. We'll start by defining closure in term of what we can observe in different behavior of our programs, as opposed to if closure was not present in JS.  

Closure is a behavior of functions and ony functions. If you aren't dealing with a function, closure does not apply. An object cannot have closure, nor does a class have closure. 

For closure to be observed, a function must be invoked, and specifically it must be invoked in a different branch of the scope chain from where it was originally defined. A function executing in the same scope it was defined would not exhibit any observably different behavior with or without closure being possible; by the observational perspective and definition, that is not closure. 

Consider: 

    // outer/global scope: RED(1)

    function lookupStudent(studentID) {
        // function scope: BLUE(2)

        var students = [
            { id: 14, name: "Kyle"},
            { id: 73, name: "Suzy"},
            { id: 112, name: "Frank"},
            { id: 6, name: "Sarah"}
        ];

        return function greetStudent(greeting){
            // function scope: GREEN(3)

            var student = students.find(
                student => student.id == studentID
            );

            return `${ greeting }, ${ student.name }!`;
        };
    }

    var chosenStudents = [
        lookupStudent(6),
        lookupStudent(112)
    ];

    // accessing the function's name: 
    chosenStudents[0].name;
    // greetStudent

    chosenStudents[0]("Hello"); 
    // Hello, Sarah!

    chosenStudents[1]("Howdy");
    // Howdy, Frank!

The first thing to notice is that the outer function creates an returns an inner function, the outer function is called twice, producing two instances of its inner function, both of which are saved into the `chosenStudents` array. 

We verify that's the case by checking the `.name` property of the returned function saved in `chosenStudents[0]`, and it's indeed an instance the inner `greetStudent(..)`.

After each call to `lookupStudent(..)` finishes, it would seem like all its inner variables would be discarded and GC’d (garbage collected). The inner function is the only thing that seems to be returned and preserved. But here’s where the behavior differs in ways we can start to observe.

While `greetStudent(..)` does receive a single argument as the parameter named `greeting`,  it also makes reference to both `students` and `studentID`, identifiers which come from the enclosing scope of `lookupStudent(..)`. Each of those references from the inner function to the variable in an outer scope is called a closure. In academic terms, each instance of `greetStudent(..)` closes over the outer variables `students` and `studentID`.

Closure allows `greetStudent(..)` to continue to access those outer variables even after the outer scope is finished. Instead of the instances of `students` and `studentID` being GC'd, they stay around in memory. At a later time when either instance of the `greetStudent(..)` function is invoked, those variables are still there, hoding their current values.

This is a direct observation of closure.

##  Adding Up Closures

Let's examine one of the canonical examples often cited for closure: 

    function adder(num1) {
        return function addTo(num2) {
            return num1 + num2;
        }
    }

    var add10To = adder(10);
    var add42To = adder(42);

    add10To(15);        // 25
    add42To(9);         // 51

Each instance of the inner `addTo(..)` function is closing over its own `num1` varible (with values 10 and 42, respectively), so those `num1`'s don't go away just because `adder(..)` finishes. When we later invoke one of those inner `addTo(..)` instances, such as the `add10To(15)` call, its closed-over `num1` variable still exists and still holds the original `10` value. The operation is thus able to perform `10 + 15` and return the answer `25`.

It's important to note that closure is observed as a runtime characeristic of function instances. 

##  Live Link, Not a Snapshot

Closure is a live link, preserving access to the full variable itself. We're not limited to merely reading a value: they closed-over variable can be updated (re-assigned) as well.

By closing over a variable in a function, we can keep using that variable as long as that function reference exists in the program, and from anywhere we want to invoke that function. 

Let's examine an example where the closed-over variable is updated: 

    function makeCounter() {
        var count = 0; 

        return getCurrent() {
            count = count + 1;
            return count;
        };
    }

    var hits = makeCounter();

    //later

    hits();         // 1

    //later

    hits();         // 2
    hits();         // 3

The `count` variable is closed over by the inner `getCurrent()` function, which keeps it around instead of it being subjected to GC. The `hits()` function call access and update this variable, returning an incrementing count each time. 

Though the enclosing scope of a closure is typically from a function, that's not actually requiered; there only needs to be an inner function present inside an outer scope: 

    var hits; 
    {   // an outer scope (but not a function)
        let count = 0; 
        hits = function getCurrent() {
            count = count + 1; 
            return count;
        };
    }

    hits();         // 1
    hits();         // 2
    hits();         // 3

Now consider: 

    var keeps = [];

    for (var i = 0; i < 3; i++) {
        keeps[i] = function keepI(){
            // closure over `i`
            return i; 
        };
    }

    keeps[0]();         // 3 -- WHY !?
    keeps[1]();         // 3
    keeps[2]();         // 3 

Something about the structure of a for-loop can trick us into thinking that each iteration gets its own new `i` variable; in fact, this program only has one `i` since it was declared with `var`. 

Each saved function returns `3`, because by the end of the loop, the single `i` variable in the program has been assigned `3`. Each of the three functions in the `keeps` array do have individual closures, but they're all closed over that same shared `i` variable. 

A single variable can only ever hold one value at any given moment. To preserve multiple values, you need a different variable for each. 

Consider: 

    var keeps = [];

    for (var i = 0; i < 3; i++) {
        // new `j` created each iteration, which gets
        // a copy of the value of `i` at this moment
        let j = i; 

        // the `i` here isn't being closed over, so 
        // it's fine to immediately use its current 
        // value in each loop iteration
        keeps[i] = function keepEachJ() { 
            // close over `j`, not `i`!
            return j;
        };
    }    

    keeps[0]();         // 0
    keeps[1]();         // 1
    keeps[2]();         // 2

Each function is now closed over a separete-new variable from each iteration, even though all of them are named `j`. 

Recall the "Loops" section, which illustrates how a `let` delaration in a `for` loop actually creates not just one variable for the loop, but actually creates a new variable for each iteration of the loop. That trick/quirk is exactly what we need for our loop closures: 

    var keeps = [];

    for (let i = 0; i < 3; i++) {
        // the `let i` gives us a new `i` for 
        // each iteration, automatically
        keeps[i] = function keepEachI() {
            return i;
        };
    }

    keeps[0]();         // 0
    keeps[1]();         // 1
    keeps[2]();         // 2

Since we're using `let`, three i's are created, one for each loop, so each of the three closures just work as expected. 

##  Common Closures: Ajax and Events

Closure is most commonly encountered with callbacks: 

    function lookupStudentRecord(studentID) {
        ajax(
            `https://some.api/student/${ studentID }`,
            function onRecord(record){
                console.log(
                    `${ record.name } (${ studentID })`
                );
            }
        );
    }

    lookupStudentRecord(114);
    // Frank (114)

The `onRecord(..)` callback is going to be invoked at some point in the future, after the response from the Ajax call comes back. This invocation will happen from the internals of the `ajax(..)` utility, wherever that comes from. Furthermore, when that happens, the `lookupStudentRecord(..)` call will long since have completed. 

Why then is `studentID` still around and accessible to the callback? Closure. 

Event handlers are another common usage of closure: 

    function listenForClicks(btn,label) {
        btn.addEventListener("click", function onClick(){
            console.log(
                `The ${ label } button was clicked!`
            );
        })
    }

    var submitBtn = document.getElementById("submit-btn");


    listenForClicks(submitBtn, "Checkout");

The `label` parameter is closed over by the `onClick(..)` even handler callback. When the button is clicked, `label` still exists to be used. This is the closure.

##  Observable Definition 

We're now ready to define closure: 

"Closure is observed when a function uses variable(s) frmo outer scope(s) even while running in a scope where those variable(s) wouldn't be accessible."

Key parts of the definition: 

- Must be a function involved 
- Must reference at least one variable from an outer scope 
- Must be invoked in a different branch of the scope  chain from the variable(s)

This observation-oriented definition means we shouldn't dismiss closure as some indirect, academic trivia. Instead, we should look and plan for the direct, concrete effects closure has on our program behavior. 

##  The Closure Lifecycle and Garbage Collection (GC)

Since closure is inherently tied to a function intance, its closure over a variable lasts as long as there is still a reference to that function. 

If ten functions all close over the same variable, and over time nine of these function references are discarded, the lone remaining function reference still preserves that variable. Once that final function reference is discarded, the last closure over that variable is gone, and the variable itself is GC'd. 

This has an important impact on building efficient and performant programs. Closure can unexpectedly prevent the GC of a variable that you´re otherwise done with, which leads to run-away memory usage over time. That's why it's important to discard function references (and thus their closures) when they're not needed anmore. 

Consider: 

    function manageBtnClickEvents(btn) {
        var clickHandlers = [];

        return function listener(cb) {
            if (cb) {
                let clickHandler = 
                    function onClick(evt){
                        console.log("clicked!");
                        cb(evt);
                    };
                clickHandlers.push(clickHandler);
                btn.addEventListener(
                    "click",
                    clickHandler
                );
            }
            else {
                // passing no callback unsubscribes 
                // all click handlers
                for (let handler of clickHandlers) {
                    btn.removeEventListener(
                        "click", 
                        handler
                    );
                }
                clickHandlers = [];
            }
        };
    }

    // var mySubmitBtn = ..
    var onSubmit = manageBtnClickEvents(mySubmitBtn);

    onSubmit(function checkout(evt){
        // handle checkout
    });

    onSubmit(function trackAction(evt){
        // log action to analytics
    });

    // later, unsubscribe all handlers: 
    onSubmit();

When we call `onSubmit()` with no input on the last line, all event handlers are unsubscribed, and the `clickHandlers` array is emptied. Once all click handler function references are discarded, the closure of `cb` references to `checkout()` and `trackAction()` are discarded. 

When considering the overall health and efficiency of the program, unsubscribing an event handler when it's no longer needed can be even more important than the initial subscription. 

##  Per Variable or Per Scope 

Conceptually, closure is per variable rather than per scope. Ajax callbacks, event handlers, and all other forms of function closures are typically assumed to close over only what they explicitly reference.

But consider: 

    function manageStudentGrades(studentRecords) {
        var grades = studentRecords.map(getGrade);

        return addGrade; 

        // ********

        function getGrade(record) {
            return record.grade;
        }

        function sortAndTrimGradesList() {
            // sort by grades, descending 
            grades.sort(function desc(g1, g2){
                return g2 - g1;
            });

            // only keep the top 10 grades
            grades = grades.slice(0,10);
        }

        function addGrade(newGrade) {
            grades.push(newGrade);
            sortAndTrimGradesList();
            return grades;
        }
    }

    var addNextGrade = manageStudentGrades([
        {id: 14, name: "Kyle", grade: 86 },
        {id: 73, name: "Suzy", grade: 87 },
        {id: 112, name: "Frank", grade: 75 },
        // ..many more records..
        {id: 6, name: "Sarah", grade: 91 }
    ]);

    // later 

    addNextGrade(81);
    addNextGrade(68);
    // [.., .., ...]

The outer function `manageStudentGrades(..)` takes a list of student records, and returns an `addGrade(..)` function reference, which we externally label `addNextGrade(..)`. Each time we call `addNextGrade(..)` with a new grade, we get back a current list of the top 10 grades, sorted numerically descending. 

In cases where a variable holds a large value (like an object or array) and that variable is present in a closure scope, if you don't need that value anumore and don't want that memory held, it's safer to manually discard the value rather than relying on closure optimization/GC. 

Let's apply a fix to the earlier `manageStudentGrades(..)` example to ensure the potentially large array held in `studentRecords` is not caught up in a closure scope unnecessarily: 

    function manageStudentGrades(studentRecords) {
        var grades = studentRecords.map(getGrade);

        // unset `studentRecords` to prevent unwanted
        // memory retention in the closure
        studentRecords = null;

        return addGrade;
        // ..
    }

We’re not removing `studentRecords` from the closure scope; that we cannot control. We’re ensuring that even if `studentRecords` remains in the closure scope, that variable is no longer referencing the potentially large array of data; the array can be GC’d. 

Again, in many cases JS might automatically optimize the program to the same effect. But it’s still a good habit to becareful and explicitly make sure we don’t keep any significant amount of device memory tied up any longer than necessary.

It’s important to know where closures appearin our programs, and what variables are included. We should manage these closures carefully so we’re only holding on to what’s minimally needed and not wasting memory.

##  An Alternative Perspective 

Reviewing our working definition for closure, the assertion is that functions are "first-class values" that can be passed around the program, just like any other value. Closure is the link-association that connects that function to the scope/variable outside of itself, no matter where that function goes. 

There's another way of thinking avout closure, and more precisely, the nature of functions being passed around, that may help deepen the mental models. 

This alternative model de-emphasizes "functions as first-class values", and instead embraces how functions are held by reference in JS, and assigned/passed by referece-copy.

So what then is closure, if not the magic that lets a function maintain a link to its orginal scope chaing even as that function moves around in other scopes? In this alternative model, functinos stay in place and keep accessing their original scope chain just like they always could. 

Closure instead describes the magic of keeping alive a function, along with its whole scope environment and chain, for as long as there's at least one reference to that function instance floating around in any other part of the program. 

The previous model is not wrong at describing closure in JS. It's just more conceptually inspired, an academic perspective on closure. The alternative model could be described as a bit more implementation focused, how JS actually works. 

##  Why Closure? 

Imagine you have a button on a page that when clicked, should retrieve and send some data via an Ajax request. 

Without using closure: 

    var APIendpoints = {
        studentIDs: 
            "https://some.api/register-students",
            // ..
    };

    var data = {
        studentIDs: [ 14, 73, 112, 6 ],
        // ..   
    };

    function makeRequest(evt) {
        var btn = evt.target;

        var recordKind = btn.dataset.kind;
        ajax(
            APIendpoints[recordKind],
            data[recordKind]
        );
    }

    //  <button data-kind="studentIDs">
    //  Register Students
    // <>/button
    btn.addEventListener("click", makeRequest);

Let's use closure to improve the code: 

    var APIendpoints = {
        studentIDs:
            "https://some.api/register-students",
            // ..
    };

    var data = {
        studentIDs: [ 14, 73, 112, 6 ],
        //..
    };

    function setupButtonHandler(btn) {
        var recordKind = btn.dataset.kind;

        btn.addEventListener(
            "click",
            function makeRequest(evt){
                ajax(
                    APIendpoints[recordKind], 
                    data[recordKind]
                );
            }
        );
    }

    // <button data-kind="studentsIDs">
    //      Register Students
    // </button>

    setupButtonHandler(btn);

With the `setupButtonHandler(..)` approach, the `datakind` attribute is retrieved once and assigned to the `recordKind` variable at initial setup. `recordKind` is then closed over by the inner `makeRequest(..)` click handler, and its value is used on each event firing to look up the URL and data that should be sent. 

By placing `recordKind` inside `setupButtonHandler(..)`, we limit the scope exposure of that variable to a more appro-priate subset of the program; storing it globally would havebeen worse for code organization and readability. Closure lets the `innermakeRequest()` function instance remember this variable and access whenever it’s needed.

Building on this pattern, we could have looked up both the URL and data once, at setup: 

    function setupButtonHandler(btn) {
        var recordKind = btn.dataset.kind;
        var requestURL = APIendpoints[recordKind];
        var requestData = data[recordKind];

        btn.addEventListener(
            "click",
            function makeRequest(evt) {
                ajax(requestURL, requestData);
            }
        ); 
    }

`NowmakeRequest(..)` is closed over `requestURL` and `requestData`, which is a little bit cleaner to understand, and also slightly more performant. 

Adapting partial application, we can further improve the preceding code: 

    functiondefineHandler(requestURL,requestData) {
        return functionmakeRequest(evt) {
            ajax(requestURL,requestData);
        };
    }

    functionsetupButtonHandler(btn) {
        var recordKind = btn.dataset.kind;
        var handler = defineHandler(
            APIendpoints[recordKind],
            data[recordKind]
        );
        btn.addEventListener("click",handler);
    }

The `requestURL` and `requestData` inputs are provided ahead of time, resulting in the 
`makeRequest(..)` partially applied function, which we locally label `handler`. When the 
event eventually fires, the final input (`evt`, even though it’s ignored) is passed to `handler()`,
completing its inputs and triggering the underlying Ajax request.

##  Closer to Closure 

We explored two models for mentally tackling closure: 

- Observational: closure is a function instance remembering its outer variables even as that function is passed to and invoked in other scopes. 
- Implementational: Closure is a function instance and its scope environment preserved in-place while any references to it are passed around and invoked from other scopes. 

Summarizing the benefits: 

- Closure can improve efficiency by allowing a function instance to remember previously determined information instead of having to compute it each time. 
- Closure can improve code readability, bounding scope-exposure by encapsulating variable(s) inside function instances, while still making sure the information in those variables is accessible for future use. The resultant narrower, more specialized function instances are cleaner to interact with, since the preserved information doesn't need to be passed in every invocation. 