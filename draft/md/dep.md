# Legend of `svg/dep-graph.svg` (prettier 3.8.2)


[DEPENDENCY GRAPH LINK](../svg/dep-graph.svg)


- <span style="color: #d1b3ff;">**PURPLE**</span> **NODES**: Represent files that have dependencies. These are the files that **import other modules**.

- <span style="color: #2ecc71;">**GREEN**</span> **NODES**: Represent **"leaf"** files. These are modules that **do NOT import anything** else; they are **INDEPENDENT**

- <span style="color: #e74c3c;">**RED**</span> **NODES**: These indicate a **WARNING**. They represent **circular dependencies** (e.g., File A imports File B, which in turn imports File A). These are the ones we should investigate to **improve our code architecture**.

- <span style="color: #f1c40f;">**YELLOW/ORANGE**</span> **NODES**: Depending on settings, these can represent **orphaned files** or modules that are **NOT imported by any other file** in the analyzed set.




```mermaid
graph TD
  A1["language-js/print/<br/>flow.js (35)"] --> A2["language-js/print/<br/>typescript.js (33)"]
  A2 --> A3["language-js/print/<br/>estree.js (29)"]
  A3 --> A4["plugins/<br/>builtin-plugins-proxy.js (27)"]
  A4 --> A5["language-markdown/<br/>printer-markdown.js (19)"]
  
  A5 --> B1["utilities/<br/>public.js (17)"]
  B1 --> B2["language-js/utilities/<br/>index.js (17)"]
  B2 --> B3["index.js (17)"]
  
  B3 --> C1["language-html/<br/>printer-html.js (16)"]
  C1 --> C2["document/builders/<br/>index.js (15)"]
  C2 --> C3["language-yaml/<br/>printer-yaml.js (14)"]
  C3 --> C4["language-css/<br/>printer-postcss.js (14)"]
  C4 --> C5["language-js/print/<br/>index.js (13)"]
  
  C5 --> D1["language-js/<br/>printer.js (12)"]
  D1 --> D2["language-js/print/<br/>object.js (12)"]
  D2 --> D3["language-js/print/<br/>class.js (12)"]
  
  D3 -.-> NEXT[...]
  NEXT --> END["... towards files with<br/>0 imports <br/>(e.g. package.json)"]

  style A1 fill:#ff0000,stroke:#333,stroke-width:4px,color:#fff
  style A2 fill:#ff4d4d,stroke:#333,color:#fff
  style A3 fill:#ff8080,stroke:#333
```