# Software Dependency and Co-change Analysis
- **Scope:**: 
Architectural analysis of Code and Knowledge Dependencies

## Methodology and Tooling

To evaluate dependencies based on imports within the source code and identify which files possess the highest or lowest number of dependencies, Madge is invoked directly from the terminal. For convenience, the output is saved into a text file:

*command used:* `npx madge src/ > dipendenze.txt`

To evaluate the formal correctness of the architecture, a circular dependency check was performed using Madge.

*command used:* `npx madge --circular src/`

To identify hidden dependencies that are not visible in the source code, the last 1000 commits were analyzed to find patterns of simultaneous file modifications.

*command used:* `git log tags/3.8.0 -n 1000 --name-only --format="format:--- COMMIT %h ---" -- src/ > cochange.txt`

### High Efferent Coupling (Fan-out)
**Most Dependencies (Hubs):** A few files stand out for importing dozens of other modules:

- `language-js/print/estree.js`: Approximately 39 dependencies.

- `language-js/print/flow.js`: Approximately 36 dependencies.

- `language-js/print/typescript.js`: Approximately 34 dependencies.

- `main/plugins/builtin-plugins/production-plugins.js`: 27 dependencies.

- `index.js`: 17 dependencies.

These files act as 'Hubs' (or orchestrators) within Prettier's architecture. Take estree.js, flow.js, and typescript.js as examples: Prettier operates by parsing code and transforming it into an Abstract Syntax Tree (AST). To print (format) this tree, the main file must know how to handle every single type of element (arrays, classes, functions, for loops, etc.). Consequently, it imports specific modules for every individual construct (e.g., it imports array.js, class.js, function.js, etc.).

Files such as production-plugins.js or index.js serve as 'entry points' or global configurations: they aggregate all the plugins and languages supported by Prettier to expose them to the user.

**Least Dependencies (Leaves):** , there are numerous files that import nothing at all. Some examples include:

- `constants.js`

- `common/errors.js`

- `document/builders/types.js`

Various utilities such as `utilities/is-non-empty-array.js` or `utilities/noop.js`.

These files are 'Leaf Nodes.' They contain pure logic, constant definitions, data types, or extremely generic utility functions (such as checking if an array is empty). Their purpose is to be independent and reusable. They are designed to be imported by many other files, yet they do not need to know anything else about the system to function.


### High Afferent Coupling (Fan-in)
These are the most 'popular' files in the repository. If these files were to break, the entire system would cease to function.
1. `document/index.js` (Over 70 incoming dependencies)
This module is imported by nearly every 'printer' file across various languages (e.g., language-css/printer-postcss.js, language-html/print/tag.js, language-js/print/arrow-function.js). Prettier operates in three stages: it ingests the source code, transforms it into an Abstract Syntax Tree (AST), converts that tree into an intermediate format called 'Doc,' and finally prints the Doc. The document/index.js file exports the primitives required to build this 'Doc' format. As the core of the formatting engine, every single language supported by Prettier must import it to function correctly.

2. `language-js/utilities/node-types.js` (Over 35 incoming dependencies)
Used extensively by files within the language-js/print/ and language-js/utilities/ directories. This file acts as a 'dictionary' for the various types of nodes within the AST. All modules responsible for printing JavaScript code query this file constantly to determine how to handle specific nodes.

3. `Cross-cutting 'Utilities'` (e.g., utilities/is-non-empty-array.js, main/comments/print.js)
These are found in files scattered throughout the architecture—from parsers to printers—across different languages such as HTML, CSS, JS, and Markdown. These represent purely logical support functions. Tasks such as verifying if an array is empty or managing comment printing are universal operations. These are centralized into small modules with very high Fan-in to prevent code duplication.

## Structural Integrity: Circular Dependencies Analysis
The analysis identified **2 circular dependencies** located within the core formatting engine:
1. `document/builders/index.js` > `document/builders/align.js` > `document/builders/indent.js` > `document/utilities/assert-doc.js` > `document/utilities/index.js`
2. `document/utilities/assert-doc.js` > `document/utilities/index.js`

While circular dependencies are generally considered an "architectural smell" because they increase coupling and complicate testing, their presence in the `document/` directory of Prettier is likely due to the **recursive nature of the "Doc" data structure**. 
The builders (which create nodes) and the utilities (which validate them) are mutually dependent because they operate on the same recursive definitions. However, from a clean code perspective, this represents a violation of the **Acyclic Dependencies Principle (ADP)**.


## Knowledge Dependencies based on Co-change
Through the analysis of the cochange.txt file, several instances of logical coupling were identified:

**Inconsistency 1: The JavaScript Parser Family** 
The Git log reveals a group of files that are consistently modified within the same commit. Evident proof can be found in commits `ca5c11e95`, `2bcc3c2a5`, and `8e75fde18`. In these commits, the following modules are 'touched' simultaneously:

`src/language-js/parse/acorn.js` 
`src/language-js/parse/babel.js` 
`src/language-js/parse/espree.js` 
`src/language-js/parse/meriyah.js` `src/language-js/parse/oxc.js` 
`src/language-js/parse/typescript.js`

At the code level **(Code Dependencies)**, these files are entirely isolated; for instance, the babel.js parser does not import acorn.js or meriyah.js to function. These modules represent alternative implementations designed to solve the same problem (parsing JavaScript code). Whenever a new JavaScript syntax must be supported or a general parsing behavior corrected, all supported parsers must be updated simultaneously to maintain system consistency.

**Inconsistency 2: Plugin Interfaces for Various Languages** 
A second significant inconsistency involves the joint modification of files belonging to entirely different programming languages.

- Visitor Keys Example: In commit `d3eb2b2d0`, dozens of files named `get-visitor-keys.js` scattered across the `language-css`, `language-html`, `language-handlebars`, `language-json`, `language-markdown`, and `language-yaml` directories were modified together.

- Pragma Example: In commit `b35ef6a54`, `src/language-css/pragma.js`, `src/language-html/pragma.js`, `src/language-graphql/pragma.js`, and `src/language-markdown/pragma.js` were updated at the same time.

This is considered an inconsistency because a CSS formatter (`language-css`) has no requirement to import or depend on the code of an HTML formatter (`language-html`). At a static level, there is no coupling. This occurs because Prettier relies on a complex **plugin architecture**. The central 'engine' requires each language to expose specific functions or configurations (such as Visitor Keys for tree navigation or Pragma directives for special comments). When the contract (the API) between the core engine and the plugins changes, or when a new global policy is introduced, developers are forced to update the interface files of all supported languages simultaneously.