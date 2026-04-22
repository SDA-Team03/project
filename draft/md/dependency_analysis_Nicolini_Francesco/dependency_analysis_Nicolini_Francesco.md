# Dependencies in Prettier source code based on imports

The analysis hase been carried out through the "cruiser" tool, with the command:<br>
`npx dependency-cruiser src --no-config --output-type metrics > adependency_metrics.txt`

The output given shows some important metrics for each folder and module in the "src" folder. The 
analysis is carried out in that specific folder because it contains the business logic of the application
and also to avoid noise due to the interference of all the other folders, making the analysis more understandable
and clear. The parameters shown are:
- "Ca": "Afferent coupling", that is how many modules depend from the analyzed file;
- "Ce": "Efferent coupling", that is how many extern modules are used from the analyzed file;
- "I": "Instability", Ca/(Ca+Ce);
- "N": "Number of modules",  that is the number of files in that package or path.

# Top 3 modules with most dependencies
In order to find the modules with most dependencies the survey must be based on the efferent coupling "Ce".
The command, usable on the file *dependency_metrics.txt*, is the following:<br>
`grep "\.js" dependency_metrics.txt | sort -k5 -nr | head -n 3`

The result is:

|type    |name                                                                  | N     |Ca     |Ce  |I(%) |       
|------- |----------------------------------------------------------------------| ------|-------|----|-----| 
|module  |`src/language-js/print/flow.js`                                       |      1|      1|  35|  97%|
|module  |`src/language-js/print/typescript.js`                                 |      1|      1|  33|  97%|
|module  |`src/language-js/print/estree.js`                                     |      1|      1|  29|  97%|

The reason why those are the 3 files with the most dependencies is that they are builders, so their role is to take the AST tree and write it, so they have to implement the process in only one module, using a lot of functions created in other modules. As we can see all the files are instable, that is due to the fact that, if one of the files used in them is modified, probably also the file itself will have to be modified

# Top 3 modules with least dependencies

As before, using the same logic, the command to find the 3 modules with the least amount of dependencies (lowest Ce) is slightly different:

`grep "\.js" dependency_metrics.txt | sort -k5 -n | head -n 3`


|type   |name                                                                   |     N|   Ca |   Ce |I(%) |       
|-------|-----------------------------------------------------------------------|------|------|------|-----| 
|module |`package.json`                                                         |     1|     2|     0|   0%|
|module |`src/cli/options/create-minimist-options.js`                           |     1|     2|     0|   0%|
|module |`src/common/ast-path.js`                                               |     1|     1|     0|   0%|


The 3 modules given as an output from the command are all "leaf nodes", that means that they are at the base of the
dependency graph: for example *package.json* contains general information regardin the project, *create-minimist-options.js* contains the information about the interpretation of the commands in the terminal, to conclude *ast-path.js* manages the analysis and movement inside the AST trees.
The common characteristic of all leaf nodes is that their Ce is equal to 0, in fact the functions inside them are crucial, because they contain the utility functions of the software, and their low efferent coupling makes the base of the project stable. In a hypothetical future, if the software will be hardly modified, it will be improbable that they will be changed.


# Knowledge dependencies: based on co-change

In order to analyze the knowledge dependencies of the program it is crucial to use git logs in order to track the last changes and to couple the files that change together. The command to analyze that behaviour in the software is:

```
git log -n 300 --skip=1 --pretty=format:"COMMIT:%h" --name-only | grep -v "^$" | awk '
/^COMMIT:/ { if (files != "") print files; files=""; next }
{ files = (files == "" ? $0 : files "," $0) }
END { print files }' | grep "," | sort | uniq -c | sort -nr | head -n 15 
```


The following data, given as a result of the previous command, are based on the last 300 commits for the version 3.8.0 of Prettier and show which files have been modified together the most:

| Frequency | File group                                                                | 
|-----------|---------------------------------------------------------------------------| 
| **49**    | `package.json`, `yarn.lock`                                               | 
| **7**     | `website/package.json`, `website/yarn.lock`                               | 
| **6**     | `packages/plugin-hermes/package.json`, `packages/plugin-oxc/package.json` | 
| **4**     | `CHANGELOG.md`, `package.json`, `.github/ISSUE_TEMPLATE/formatting.md`    | 
| **2**     | `docs/configuration.md`, `website/versioned_docs/.../configuration.md`    | 

As we can see a lot of files change with others, now let's analyze if those changes are consistent or not with the code: a change is considered consistent when, in case two or more files are modified, there is a code dependence between them, otherwise the changes are not consistent.

|File group                                                                 |Consistency| 
|---------------------------------------------------------------------------|-----------|
| `package.json`, `yarn.lock`                                               |**Yes**: each time a library is imported the system automatically updates both files: one registers the name and the other the version|
| `website/package.json`, `website/yarn.lock`                               |**Yes**: same as above but with a part of the project|
| `packages/plugin-hermes/package.json`, `packages/plugin-oxc/package.json` |**No**: there are no inport between the 2 files|
| `CHANGELOG.md`, `package.json`, `.github/ISSUE_TEMPLATE/formatting.md`    |**Yes**: documentation and meta data must be modified together|
| `docs/configuration.md`, `website/versioned_docs/.../configuration.md`    |**No**: there is no link between these 2 files|




