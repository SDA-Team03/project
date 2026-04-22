graph LR
    %% Nodo Centrale
    TS["src/language-js/print/typescript.js"]
    
    %% --- BLOCCO CORE & UTILITIES ---
    subgraph TS_Utils["<b>UTILITIES & HELPERS</b>"]
        UNode["unexpected-node-error.js"]
        ITsKey["is-ts-keyword-type.js"]
    end

    %% --- BLOCCO SUB-PRINTERS (EFFERENT COUPLING) ---
    subgraph TS_SubPrinters["<b>SPECIFIC LANGUAGE PRINTERS</b>"]
        TArray["array.js"]
        TArrayT["array-type.js"]
        TBlock["block.js"]
        TCall["call-expression.js"]
        TCast["cast-expression.js"]
        TFunc["function.js (printMethodValue)"]
        TFuncT["function-type.js"]
        TIdxSig["index-signature.js"]
        TIdxAcc["indexed-access-type.js"]
        TInfer["infer-type.js"]
        TInter["intersection-type.js"]
        TJSDoc["js-doc-type.js"]
        TMapped["mapped-type.js"]
        TMethod["method-signature.js"]
        TMod["module.js (printImportKind)"]
        TModDecl["module-declaration.js"]
        TProp["property.js"]
        TRest["rest-type.js"]
        TTempl["template-literal.js"]
        TTern["ternary.js"]
        TTuple["tuple.js"]
        TTypeA["type-alias.js"]
        TTypeAssert["type-assertion.js"]
        TParam["type-parameters.js"]
        TTypePred["type-predicate.js"]
        TTypeQ["type-query.js"]
        TUnion["union-type.js"]
    end

    %% --- COLLEGAMENTI ---
    TS --> TS_Utils
    TS --> TS_SubPrinters

    %% --- STILIZZAZIONE ---
    
    %% Nodo principale: Blu TypeScript
    style TS fill:#e1f5fe,stroke:#0277bd,stroke-width:4px,color:#0277bd

    %% Blocco Utilities: Grigio/Verde
    style TS_Utils fill:#f1f8e9,stroke:#33691e,stroke-width:2px,color:#33691e
    
    %% Blocco SubPrinters: Viola (Simile a Flow per coerenza)
    style TS_SubPrinters fill:#f3e5f5,stroke:#4a148c,stroke-width:2px,color:#4a148c

    %% Style nodi interni
    style TArray fill:#fff,stroke:#4a148c
    style TBlock fill:#fff,stroke:#4a148c