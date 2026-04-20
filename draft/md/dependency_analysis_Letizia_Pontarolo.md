# Code and Knowledge Dependencies Analysis

## Code Dependencies

To analyze code dependencies, the number of `import` statements in each file was counted.  
A higher number of imports means that a file depends on many other modules.

---

### Command (files with most dependencies)

The following command scans all source files and counts how many `import` statements each file contains.  
The expected output is a list of files sorted by number of imports, showing the most dependent files.

```
Get-ChildItem src -Recurse -File -Include *.js,*.mjs |
ForEach-Object {
$relative = Resolve-Path -Relative $_.FullName
$count = (Select-String -Path $_.FullName -Pattern '^\s*import\s' -AllMatches | Measure-Object).Count
[PSCustomObject]@{ Imports = $count; File = $relative }
} |
Sort-Object Imports -Descending |
Select-Object -First 10
```

### Files with most dependencies (first 10)
- `.\src\language-js\print\flow.js` (35 imports) 
- `.\src\language-js\print\typescript.js` (33 imports) 
- `.\src\language-js\print\estree.js` (29 imports) 
- `.\src\language-markdown\printer-markdown.js` (21 imports) 
- `.\src\index.js` (18 imports) 
- `.\src\language-js\utilities\index.js` (16 imports)
- `.\src\language-html\printer-html.js` (16 imports)
- `.\src\language-yaml\printer-yaml.js` (14 imports)
- `.\src\language-css\parser-postcss.js` (14 imports)
- `.\src\plugins\builtin-plugins-proxy.js` (14 imports)

### Explanation

Files with many imports are usually central or complex parts of the system.

- The JavaScript printer files (`flow.js`, `typescript.js`, `estree.js`) handle many different syntax cases. They need many helper functions and modules to correctly format all language features.
- The Markdown printer also has many dependencies because Markdown is flexible and can contain embedded code.
- `src/index.js` is the main entry point of Prettier. It connects plugins, configuration, and the core formatting pipeline. Because of this, it naturally depends on many modules.

In general, a high number of imports indicates:

- complex logic
- central role in the architecture
- high coupling with other modules

---

### Command (files with least dependencies)

This command lists the files with the lowest number of imports.
The expected output is a list of simple or independent modules.

```
Get-ChildItem src -Recurse -File -Include *.js,*.mjs |
ForEach-Object {
$relative = Resolve-Path -Relative $_.FullName
$count = (Select-String -Path $_.FullName -Pattern '^\s*import\s' -AllMatches | Measure-Object).Count
[PSCustomObject]@{ Imports = $count; File = $relative }
} |
Sort-Object Imports, File |
Select-Object -First 30
```

### Files with least dependencies (first 30)

- `.\src\cli\options\create-minimist-options.js`
- `.\src\common\ast-path.js`
- `.\src\common\common-options.evaluate.js`
- `.\src\common\errors.js`
- `.\src\common\parser-create-error.js`
- `.\src\config\editorconfig\editorconfig-to-prettier.js`
- `.\src\document\builders\index.js`
- `.\src\document\builders\types.js`
- `.\src\document\index.js`
- `.\src\document\printer\indent.js`
- `.\src\document\printer\trim-indentation.js`
- `.\src\experimental-cli\constants.evaluate.js`
- `.\src\experimental-cli\index.js`
- `.\src\experimental-cli\worker.js`
- `.\src\language-css\embed.js`
- `.\src\language-css\utilities\get-value-root.js`
- `.\src\language-css\utilities\has-string-or-function.js`
- `.\src\language-css\utilities\index.js`
- `.\src\language-css\utilities\is-module-rule-name.js`
- `.\src\language-css\utilities\is-scss-nested-property-node.js`
- `.\src\language-css\utilities\is-scss-variable.js`
- `.\src\language-graphql\loc.js`
- `.\src\language-handlebars\embed.js`
- `.\src\language-handlebars\loc.js`
- `.\src\language-handlebars\visitor-keys.js`
- `.\src\language-html\clean.js`
- `.\src\language-html\loc.js`
- `.\src\language-html\print\angular-control-flow-block-settings.evaluate.js`
- `.\src\language-html\utilities\is-unknown-namespace.js`
- `.\src\language-js\comments\is-gap.js`

### Explanation

Files with zero or very few imports are usually simple and isolated components.
- Many of them are utility files (e.g., errors.js, ast-path.js)
- Others define small helpers or basic structures
- Some files just export functions without needing external modules

These files typically:
- do one specific task
- are easy to understand
- are reused by other modules

Low number of imports usually means:
- low complexity
- low coupling
- good modularity 

---

## Knowledge Dependencies

Knowledge dependencies were identified by analyzing co-change in Git history.

### Command

This command shows which files are modified together in the same commit.
The expected output is a sequence of commits with the list of changed files.

```
git --no-pager log -n 100 --name-only --pretty=format:"--- COMMIT ---"
```

---

## Identified Relationships

### package.json + yarn.lock 
- Not consistent with code dependencies.
- These two files are often modified together when project dependencies are added or updated.
For example, when a new library is installed, both files change automatically.
- However, this relationship is not visible in the source code because these files are not imported anywhere.
They are related to dependency management rather than to the program logic.

---

### Source files + Test files 
- Not consistent with code dependencies.
- These files are often modified together because tests must be updated when the formatting behavior changes. However, the test files do not directly import the source files. They use a shared testing framework (`runFormatTest`) and refer to the language (e.g. "markdown"), so the dependency is indirect. For this reason, the relation is not visible through import-based code dependencies.
- Example:  
`changelog_unreleased/markdown/18519.md`  
`src/language-markdown/embed.js`  
`tests/format/markdown/code/angular/__snapshots__/format.test.js.snap`  
`tests/format/markdown/code/angular/angular-html-invalid.md`  
`tests/format/markdown/code/angular/angular-html.md`  
`tests/format/markdown/code/angular/angular-ts-invalid.md`  
`tests/format/markdown/code/angular/angular-ts.md`  
`tests/format/markdown/code/angular/format.test.js`

---

### Source code + Changelog files 
- Not consistent with code dependencies.
- When a feature is added or modified, the change is usually documented in the changelog in the same commit.
- Even if these files are changed together, there is no real code dependency between them. The changelog is only used to describe what has been done, not to run or support the code.
- Example:  
`changelog_unreleased/typescript/18395.md`  
`src/language-js/print/union-type.js`  
`...`

---

### Documentation and release files 
- Not consistent with code dependencies.
- These files are often updated together during releases or documentation updates.
For example, when a new version is published, documentation pages, changelog, and configuration files may all be updated.
- This relationship is not related to the internal structure of the code. It exists because of the development and release process.
- Example:  
`.github/ISSUE_TEMPLATE/formatting.md`  
`.github/ISSUE_TEMPLATE/integration.md`  
`CHANGELOG.md`  
`package.json`  
`website/versioned_docs/version-stable/browser.md`

---

### Multiple language modules changed together 
- Partially consistent with code dependencies.
- Sometimes multiple language modules are modified in the same commit.
This usually happens when a change affects shared behavior or common logic used across different languages.
- In some cases, these modules share utilities or parts of the formatting pipeline, so there is a real dependency.
In other cases, they are simply updated together because the same feature must be implemented in multiple languages.
- For this reason, the relationship is only partially consistent with code dependencies.
- Example:  
`src/language-css/parse/parse-value.js`  
`src/language-css/parse/utilities.js`  
`src/language-css/parser-postcss.js`  
`src/language-js/parse/postprocess/index.js`  
`src/language-js/parse/postprocess/visit-node.js`  
`src/language-js/utilities/index.js`  
`src/main/ast-to-doc.js`

---

### Files inside the same language-specific printing area 
- Consistent with code dependencies.
- Files that belong to the same language-specific printing area are often changed together. This is expected because they are part of the same subsystem and usually work together to format the same language. They often share helper functions, printing logic, and internal utilities.
- Example:  
`src/language-markdown/print/children.js`  
`src/language-markdown/print/list.js`  
`src/language-markdown/printer-markdown.js`  
`src/language-markdown/utils.js`

---

### Core modules changed together 
- Consistent with code dependencies.
- These files belong to the main formatting pipeline of Prettier.
They work closely together to transform code into its formatted version.
- Because of this, a change in one of these files often requires changes in the others. This is a strong and direct dependency, and it is fully consistent with the code structure.
- Example:  
`src/common/end-of-line.js`  
`src/document/printer/printer.js`  
`src/main/core.js`

---

## Commit History (first 50 commits)

```
--- COMMIT ---
scripts/release/steps/show-instructions-after-npm-publish.js
scripts/release/tests/publish-to-npm.test.js

--- COMMIT ---
package.json
yarn.lock

--- COMMIT ---
changelog_unreleased/scss/18471.md

--- COMMIT ---
.github/ISSUE_TEMPLATE/formatting.md
.github/ISSUE_TEMPLATE/integration.md
CHANGELOG.md
package.json
website/versioned_docs/version-stable/browser.md

--- COMMIT ---
changelog_unreleased/scss/18471.md
src/language-css/utilities/index.js
tests/format/scss/function/18465.scss
tests/format/scss/function/__snapshots__/format.test.js.snap

--- COMMIT ---
packages/plugin-oxc/package.json

--- COMMIT ---
package.json
yarn.lock

--- COMMIT ---
tests/format/angular/expression/__snapshots__/format.test.js.snap
tests/format/angular/expression/arrow-function-options/__snapshots__/format.test.js.snap

--- COMMIT ---
package.json
yarn.lock

--- COMMIT ---
changelog_unreleased/angular/18722.md

--- COMMIT ---
website/src/pages/index.jsx

--- COMMIT ---
.github/ISSUE_TEMPLATE/formatting.md
.github/ISSUE_TEMPLATE/integration.md
CHANGELOG.md
package.json
website/versioned_docs/version-stable/browser.md

--- COMMIT ---
changelog_unreleased/angular/18722.md
package.json
src/language-js/print/function-parameters.js
tests/format/angular/expression/__snapshots__/format.test.js.snap
tests/format/angular/expression/arrow-function-options/__snapshots__/format.test.js.snap
tests/format/angular/expression/arrow-function-options/format.test.js
tests/format/angular/expression/arrow-function.html
tests/format/angular/expression/instanceof-operator.html
yarn.lock

--- COMMIT ---
package.json
src/language-html/print/angular-control-flow-block.js
tests/format/angular/control-flow/__snapshots__/format.test.js.snap
tests/format/angular/control-flow/default-never.html
yarn.lock

--- COMMIT ---
package.json
yarn.lock

--- COMMIT ---
changelog_unreleased/api/18706.md

--- COMMIT ---
.github/ISSUE_TEMPLATE/formatting.md
.github/ISSUE_TEMPLATE/integration.md
CHANGELOG.md
package.json
website/versioned_docs/version-stable/browser.md

--- COMMIT ---
changelog_unreleased/api/18706.md
scripts/build/build-types.js
tests/dts/unit/cases/parsers.ts

--- COMMIT ---
scripts/release/run.js

--- COMMIT ---
package.json
yarn.lock

--- COMMIT ---
changelog_unreleased/angular/18378.md
changelog_unreleased/angular/18593.md
changelog_unreleased/angular/18596.md
changelog_unreleased/blog-post-intro.md
changelog_unreleased/markdown/18519.md

--- COMMIT ---
scripts/release/run.js

--- COMMIT ---
scripts/release/steps/merge-blog-post.js

--- COMMIT ---
.github/ISSUE_TEMPLATE/formatting.md
.github/ISSUE_TEMPLATE/integration.md
CHANGELOG.md
package.json
website/versioned_docs/version-stable/browser.md

--- COMMIT ---
changelog_unreleased/angular/18593.md
changelog_unreleased/angular/18596.md
changelog_unreleased/blog-post-intro.md
website/blog/2026-01-14-3.8.0.md

--- COMMIT ---
scripts/release/run.js
scripts/release/steps/index.js
scripts/release/steps/lint-files.js

--- COMMIT ---
tests/format/misc/errors/front-matter/__snapshots__/format.test.js.snap

--- COMMIT ---
changelog_unreleased/angular/18596.md
package.json
tests/format/angular/expression/__snapshots__/format.test.js.snap
tests/format/angular/expression/spread-element.html
yarn.lock

--- COMMIT ---
changelog_unreleased/angular/18593.md
package.json
src/language-html/print/angular-control-flow-block.js
tests/format/angular/control-flow/__snapshots__/format.test.js.snap
tests/format/angular/control-flow/multiple-switch-cases.html
yarn.lock

--- COMMIT ---
changelog_unreleased/angular/18378.md
changelog_unreleased/angular/18596.md
package.json
tests/format/angular/expression/__snapshots__/format.test.js.snap
tests/format/angular/expression/exponentiation-operators.html
tests/format/angular/expression/format.test.js
tests/format/angular/expression/in-operators.html
tests/format/angular/expression/spread-element.html
tests/format/angular/expression/tagged-template-literal.html
tests/format/angular/expression/template-literal.html
tests/format/angular/expression/void-operators.html
yarn.lock

--- COMMIT ---
changelog_unreleased/markdown/18519.md
src/language-markdown/embed.js
tests/format/markdown/code/angular/__snapshots__/format.test.js.snap
tests/format/markdown/code/angular/angular-html-invalid.md
tests/format/markdown/code/angular/angular-html.md
tests/format/markdown/code/angular/angular-ts-invalid.md
tests/format/markdown/code/angular/angular-ts.md
tests/format/markdown/code/angular/format.test.js

--- COMMIT ---
src/language-markdown/embed.js

--- COMMIT ---
changelog_unreleased/angular/18378.md
src/language-html/embed/utilities.js
tests/format/angular/angular/__snapshots__/format.test.js.snap
tests/format/angular/attribute-value-hugging/__snapshots__/format.test.js.snap
tests/format/angular/attribute-value-hugging/attribute-value-hugging.html
tests/format/angular/attribute-value-hugging/format.test.js
tests/format/vue/vue/__snapshots__/format.test.js.snap
tests/format/vue/vue/expression-binding.vue

--- COMMIT ---
.github/workflows/codeql.yml
.github/workflows/dev-test.yml
.github/workflows/eslint-rules.yml
.github/workflows/lint.yml
.github/workflows/prod-test.yml

--- COMMIT ---
package.json
yarn.lock

--- COMMIT ---
changelog_unreleased/lwc/18383.md
changelog_unreleased/typescript/18393.md
changelog_unreleased/typescript/18395.md

--- COMMIT ---
.github/ISSUE_TEMPLATE/formatting.md
.github/ISSUE_TEMPLATE/integration.md
CHANGELOG.md
package.json
website/versioned_docs/version-stable/browser.md

--- COMMIT ---
packages/plugin-hermes/package.json
packages/plugin-oxc/package.json

--- COMMIT ---
src/language-js/parentheses/needs-parentheses.js

--- COMMIT ---
changelog_unreleased/typescript/18395.md
src/language-js/comments/will-print-own-comments.js
src/language-js/print/union-type.js
src/language-js/utilities/index.js
src/main/comments/print.js
tests/format/typescript/union/comments/18379.ts
tests/format/typescript/union/comments/__snapshots__/format.test.js.snap

--- COMMIT ---
changelog_unreleased/typescript/18393.md
src/language-js/print/union-type.js
tests/format/typescript/union/comments/18389.ts
tests/format/typescript/union/comments/__snapshots__/format.test.js.snap

--- COMMIT ---
package.json
yarn.lock

--- COMMIT ---
website/package.json
website/yarn.lock

--- COMMIT ---
src/language-css/parse/parse-value.js
src/language-css/parse/utilities.js
src/language-css/parser-postcss.js
src/language-js/parse/postprocess/index.js
src/language-js/parse/postprocess/visit-node.js
src/language-js/utilities/index.js
src/main/ast-to-doc.js

--- COMMIT ---
scripts/build/config.js
scripts/build/esbuild-plugins/evaluate.js
src/language-css/get-visitor-keys.js
src/language-css/visitor-keys.evaluate.js
src/language-handlebars/get-visitor-keys.js
src/language-handlebars/html-void-elements.evaluate.js
src/language-handlebars/utilities.js
src/language-handlebars/visitor-keys.js
src/language-html/get-visitor-keys.js
src/language-html/visitor-keys.evaluate.js
src/language-js/traverse/visitor-keys.evaluate.js
src/language-json/get-visitor-keys.js
src/language-json/visitor-keys.evaluate.js
src/language-markdown/get-visitor-keys.js
src/language-markdown/visitor-keys.evaluate.js
src/language-yaml/get-visitor-keys.js
src/language-yaml/visitor-keys.evaluate.js
src/utilities/visitor-keys.js
tests/integration/__tests__/bundle.js
tests/unit/__snapshots__/visitor-keys.js.snap
tests/unit/visitor-keys.js

--- COMMIT ---
changelog_unreleased/lwc/18383.md
src/language-html/embed/attribute.js
src/language-html/printer-html.js
src/language-html/utilities/index.js
tests/format/lwc/attribute/quotes/__snapshots__/format.test.js.snap
tests/format/lwc/attribute/quotes/quotes.html

--- COMMIT ---
benchmarks/get-preferred-quote.js
src/utilities/get-preferred-quote.js

--- COMMIT ---
src/common/end-of-line.js
src/document/printer/printer.js
src/main/core.js

--- COMMIT ---
website/playground/Playground.jsx

--- COMMIT ---
src/language-js/print/literal.js

```
