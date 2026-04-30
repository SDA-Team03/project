# PATTERN USAGE

## 1. FACADE

A Facade is a structural pattern that provides a simple entry point to a more complex subsystem. It hides internal details and lets the client use the system without dealing with all its internal classes or modules.

### Which classes play which role?

Facade: `src/index.js`

Subsystem modules behind the facade:
- `src/main/core.js`
- `src/main/plugins/index.js`
- `src/config/resolve-config.js`
- `src/main/support.js`
- `src/document/public.js`
- `src/utilities/public.js`
- `src/common/get-file-info.js`

More specifically, `src/index.js` exposes a small public API such as:
- `format`
- `formatWithCursor`
- `check`
- `resolveConfig`
- `resolveConfigFile`
- `getSupportInfo`

while internally it delegates work to core formatting logic, plugin loading, configuration handling, parser inference, and support utilities.

### Why is the pattern used?

The pattern is used because Prettier has a complex internal structure:
- formatting logic is in `core.js`
- plugins are loaded in `main/plugins/index.js`
- configuration is handled in `config/resolve-config.js`
- support information comes from `main/support.js`
- document utilities and public utilities are exposed from other modules

Without a facade, a client would need to know many internal modules and call them directly. With `src/index.js`, the client can just import Prettier and use a few simple functions.

### Which problem does solve?

It solves the problem of internal complexity exposure.

More concretely, it solves these issues:
- the public user should not know the whole internal architecture
- the client should not manually load plugins, normalize options, and call core modules directly
- the public API should remain simple even if the internal implementation changes

So the facade makes Prettier easier to use and reduces coupling between the client and the internal modules.

### Is there an alternative?

One alternative would be to not use a facade and let clients call internal modules directly.

For example, a client could import:
- `core.js`
- `plugins/index.js`
- `resolve-config.js`
- `support.js`

and manually compose the behavior.

Pros of the alternative:
- more direct access to internal features
- more flexibility for advanced users
- possibly less indirection

Cons of the alternative:
- harder public API
- stronger coupling to internal structure
- clients would break more easily if internals change
- more knowledge required to use the system correctly

---

## 2. STRATEGY

A Strategy is a behavioral pattern used when the same general task can be performed with different algorithms or behaviors. Instead of putting many conditionals inside one big class, each variation is encapsulated in a separate strategy object, and the context uses the selected one.

### Which classes play which role?

The main context is the formatting pipeline, especially:
- `src/main/core.js`
- `src/main/ast-to-doc.js`

These modules coordinate the work, but they do not hardcode the behavior for every language.

The “interface” is not always a formal class interface, but a shared expected contract.

For parsers, the context expects a parser that can parse text into an AST.

For printers, the context expects a printer object with operations such as:
- `print(...)`
- sometimes `preprocess(...)`
- sometimes other printer-related hooks

Examples of concrete parser/printer strategies are:

Parsers:
- `src/language-js/parse/babel.js`
- other language parsers selected through the plugin system

Printers:
- `src/language-js/print/estree.js`
- `src/language-markdown/printer-markdown.js`
- `src/language-html/printer-html.js`
- `src/language-css/printer-postcss.js`
- `src/language-yaml/printer-yaml.js`

So the context stays the same, but the selected parser/printer changes depending on the language or parser option.

### Why is the pattern used?

The pattern is used because Prettier always does the same general task:
- parse code
- build a Doc
- print formatted output

However, the way this is done depends on the language:
- JavaScript needs one parser/printer behavior
- Markdown needs another
- HTML needs another
- CSS needs another

If all this logic were inside one single module full of `if language == ...`, the code would become hard to maintain and extend.

By using separate parser/printer modules, Prettier can keep the main pipeline stable and move language-specific behavior into interchangeable modules.

### Which problem does solve?

It solves the problem of behavioral variation.

More concretely:
- different languages require different parsing logic
- different languages require different printing rules
- the main formatting pipeline should not depend on a huge set of conditionals
- new languages or new parser/printer implementations should be addable without rewriting the core

So Strategy helps Prettier support many languages while keeping the central pipeline reusable.

### Is there an alternative?

The main alternative would be a big conditional-based design.

For example, the core could contain logic like:
- if language is JavaScript → use JS code
- if language is HTML → use HTML code
- if language is Markdown → use Markdown code

Pros of the alternative:
- simpler to understand at the very beginning
- fewer modules
- more direct flow in a small system

Cons of the alternative:
- very poor scalability
- many large conditionals
- difficult maintenance
- harder to add new languages
- stronger coupling between core logic and language-specific behavior

---

## 3. COMPOSITE

A Composite is a structural pattern used to represent part-whole hierarchies. It lets the client treat single objects and compositions of objects in a uniform way.

### Which classes play which role?

The pattern appears mainly in the Doc system.

The general Doc abstraction plays the role of the common component. It is not always a classical class, but it is a common representation used by the printing system.

Simple Doc nodes act like leaves, for example:
- strings
- line markers
- cursor markers
- small primitive doc elements

These are basic elements that do not contain other Doc structures in a meaningful hierarchical way.

Composite Doc nodes are the ones that contain other Docs, for example:
- `group(...)`
- `indent(...)`
- `align(...)`
- arrays of docs
- `fill(...)`
- labeled docs

These combine smaller Docs into larger Docs.

The main client is:
- `src/document/printer/printer.js`

It traverses and prints the Doc structure without needing completely separate logic for “single thing” versus “composed thing”. It checks the Doc type and processes the tree recursively or through its command stack.

### Why is the pattern used?

The pattern is used because Prettier needs an intermediate representation of formatting that can be:
- simple in some cases
- nested in many other cases
- composed step by step from smaller formatting pieces

For example, a formatted expression can be built from:
- a string
- a line break
- an indented sub-expression
- a grouped structure containing other parts

So the system needs a way to combine small formatting elements into larger formatting structures, while still treating them as part of the same overall representation.

### Which problem does solve?

It solves the problem of representing complex formatted output as a hierarchy of smaller pieces.

Without a Composite-like structure, Prettier would need:
- many special cases
- separate handling for small pieces and larger structures
- a much more rigid representation

With this design, Prettier can:
- build formatting structures incrementally
- combine nested docs naturally
- treat simple and complex docs in a mostly uniform way

### Is there an alternative?

One alternative would be to represent formatting with a flat sequence of printing commands or with plain strings plus many ad hoc rules.

Pros of the alternative:
- maybe simpler for very small formatting tasks
- easier to understand at first
- less abstract structure

Cons of the alternative:
- much harder to represent nested layout decisions
- weaker support for grouping and indentation
- more fragile code
- less modular composition of formatting pieces

---

## 4. VISITOR

A Visitor is a behavioral pattern used when there is an object structure with many different element types, and many operations may need to be performed on those elements.

### Which classes play which role?

This pattern appears mainly in the way the AST is traversed and printed.

The AST nodes play the role of the elements being visited.

Examples:
- FunctionDeclaration
- BinaryExpression
- IfStatement
- ArrayExpression
- and many other node types handled in `src/language-js/print/estree.js`

The visitor-like behavior is mainly implemented by:
- `src/main/ast-to-doc.js`
- `src/language-js/print/estree.js`

`ast-to-doc.js` recursively walks the AST through AstPath and delegates printing.

`estree.js` then decides what to do depending on `node.type`.

Traversal support:
- `src/common/ast-path.js` supports the navigation of the tree
- `mainPrint(...)` and `callPluginPrintFunction(...)` in `ast-to-doc.js` manage recursive traversal and delegation

One concrete operation is:
- converting the AST into a Doc

So the “visit” is not used for many unrelated operations in the classic GoF way, but it is clearly used to apply node-specific behavior while traversing a heterogeneous tree.

### Why is the pattern used?

The pattern is used because an AST is a tree with many different node types.

Each node type needs its own formatting behavior.

For example:
- a BinaryExpression must be printed differently from
- an IfStatement, which must be printed differently from
- a FunctionDeclaration

Prettier needs a structured way to traverse the tree and apply different logic depending on the node type, without putting all formatting code inside the nodes themselves.

### Which problem does solve?

It solves the problem of applying node-specific operations over a complex tree structure.

More concretely:
- the AST contains many different node types
- the formatting logic depends on the exact node type
- the traversal must be systematic and recursive
- the formatting operation should stay separated from the AST data structure itself

### Is there an alternative?

One alternative would be to place formatting behavior directly inside the AST node classes or objects.

Another alternative would be to have one very large centralized function with many conditionals and no structured traversal support.

Pros of the alternative:
- may look simpler at first
- fewer modules in a very small system
- direct control flow

Cons of the alternative:
- mixes data structure and formatting behavior
- harder to maintain
- harder to extend
- very large conditional logic
- less clear separation between tree structure and operationsmaintenance  
