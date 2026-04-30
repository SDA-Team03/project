# Patterns in Prettier source code

## 1.Visitor pattern
Main files:
```
src/language-js/print/angular.js:16:  switch (node.type) 
src/language-js/print/assignment.js:343:  switch (node.type) 
src/language-js/print/assignment.js:453:  switch (node.type) 
src/language-js/print/estree.js:97:  switch (node.type)
src/language-js/print/flow.js:66:  switch (node.type) 
src/language-js/print/jsx.js:804:  switch (node.type) 
src/language-js/print/literal.js:12:  switch (node.type) 
src/language-js/print/match.js:93:  switch (node.type) 
src/language-js/print/typescript.js:64:  switch (node.type) 
```

Command:
`grep -rn "switch (node.type)" src/language-js/print/`

The output shows a centralized "switch" logic that reacts to different node types. This proves the use of a Visitor, where a single function manages the printing logic for diverse elements without requiring the elements themselves to know how to print.

Which classes/roles?
- Visitor: The genericPrint function (containing the switch).
- Element: The AST nodes (e.g., BinaryExpression, IfStatement) provided by external parsers.

Why use it?
It separates the printing logic from the data structure. Since Prettier uses external parsers (Babel, Flow), it cannot add a .print() method to the node classes directly.

Alternatives: Adding printing methods directly into node classes.

- Pros: Better encapsulation.

- Cons: Impossible with third-party ASTs; violates the Single Responsibility Principle.


## 2.Builder pattern
Main file:
```
src\document\builders\group.js:22:function group(contents, options = {}) {
```

Command:
`Get-ChildItem -Recurse src/document/*.js | Select-String -Pattern "function group\("`

The output shows specialized functions like group() dedicated solely to constructing the IR (Intermediate Representation). This confirms the Builder pattern because it abstracts the creation of complex objects, ensuring the printer (Director) stays clean and focused on layout logic.

Which classes/roles?

- Builder: The utility functions in src/document/builders/ (group, indent, ifBreak).
- Product: The Doc object (an Intermediate Representation of the formatted code).

Why use it? To simplify the creation of complex, nested "Doc" structures. It provides a clean API to define how code should look before the final string is generated.

Alternatives: Direct manual creation of JSON objects for the IR.

- Pros: No function call overhead.

- Cons: Error-prone; hard to maintain if the Doc structure changes.


## 3.Strategy pattern
Main files:
```
src/language-css/index.js
src/language-graphql/index.js
src/language-handlebars/index.js
src/language-html/index.js
src/language-markdown/index.js
src/language-yaml/index.js
```


Command:
`find src -name "index.js" -path "*/language-*" -exec grep -l "languages" {} +`

The output shows a consistent "index.js" file across multiple independent language directories. This proves the Strategy pattern: Prettier treats each language as a swappable strategy, loading the correct one at runtime based on the file type.

Which classes/roles?

- Context: Prettier Core (the main formatting engine).

- Strategy Interface: The plugin contract (requiring parsers and printers).

- Concrete Strategy: Each src/language-*/ directory.

Why use it? To support multiple languages (CSS, HTML, Markdown) through a common interface, making the engine extensible via plugins.

Alternatives: A massive monolithic switch statement in the core.

- Pros: Easier to follow the execution flow in one place.

- Cons: Not extensible; high risk of breaking one language while fixing another.




## 4.Command pattern
Main files:
```
src\document\debug.js:65:function printDocToDebug(doc) {
src\document\debug.js:72:  function printDoc(doc, index, parentParts) {
src\document\printer\printer.js:178:function printDocToString(doc, options) {
src\main\core.js:416:async function printDocToString(doc, options) {
```

Command:
`Get-ChildItem -Recurse src -Filter "*.js" | Select-String -Pattern "function printDoc"`

The output shows a standalone `printDocToString` function that takes a "doc" (a list of commands) as an argument. This confirms the Command pattern, as the formatting "intent" is encapsulated in an object and passed to an invoker that decides when and how to execute it.

Which classes/roles?

- Command: The Doc objects (instructions like indent, group, break).

- Invoker/Interpreter: The printDocToString function in printer.js.

- Client: The language printers that queue these commands.

Why use it? To decouple formatting intent from physical execution. It allows "backtracking": the engine can simulate commands to see if they fit in the line width and change the layout if they don't.

Alternatives: Immediate recursive printing to string.

- Pros: Faster execution and lower memory usage.

- Cons: Impossible to implement intelligent line-wrapping (the engine can't "undo" a printed string).
