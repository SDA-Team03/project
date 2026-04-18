# First step analysis - Prettier v3.8.2

## Code Dependecies based on imports in source code

To extract datas, it has been used `madge` with output DOT, in order to count all directs relations between files.

*command used:* `npx madge --dot src/ | grep " \-> " | awk -F" \-> " '{print $1}' | sort | uniq -c | sort -nr | head -n 10`

- **Top 3 High Coupling**: 
![10_high_couple](image1.png)

*command used:* `npx madge --dot src/ | grep " \-> " | awk -F" \-> " '{print $1}' | sort | uniq -c | sort -n | head -n 10`
- **Top 10 Low Coupling**: 
![10_low_couple](image2.png)

### Which files have most or least dependencies? Why?
- **Most Dependencies (Hubs):** They must aggregate numerous micro-modules (parsers, printers, plugins) to expose a unified interface (e.g. the format() function) and manage the complexity of the various languages.
- **Least Dependencies (Leaves):** They are Atomic Utilities located at the base of the architectural pyramid. They follow the principle of Single Responsibility, providing isolated services without importing core logic to prevent circular dependencies.

## Knowledge dependencies based on co-change (how often 2 files are changed together in the same commit)

*command used:* `git log --pretty=format: --name-only -n 500 | grep "src/" | sort | uniq -c | sort -rg | head -n 15`

![know_dep](image3.png)

### Which are not consistent with code dependencies?
There is a Knowledge Dependency (logical coupling): a structural change to the Abstract Syntax Tree (AST) forces coordinated changes across multiple print forms to maintain consistency, even if files are decoupled in the code by modularity.