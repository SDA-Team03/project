graph LR
    %% Nodo Centrale
    Flow["src/language-js/print/flow.js"]
    
    %% --- BLOCCO CORE DOCUMENT ---
    subgraph Document["<b>CORE PRINTING ENGINE</b>"]
        DocIdx["../../document/index.js"]
        REOL["replaceEndOfLine"]
    end

    %% --- BLOCCO UTILITIES ---
    subgraph Utils["<b>UTILITIES & HELPERS</b>"]
        PNum["print-number.js"]
        PStr["print-string.js"]
        GRaw["get-raw.js"]
        UIdx["../utilities/index.js (isMethod)"]
        IFlow["is-flow-keyword-type.js"]
        Assert["#universal/assert"]
    end

    %% --- BLOCCO SUB-PRINTERS (EFFERENT COUPLING) ---
    subgraph SubPrinters["<b>SPECIFIC LANGUAGE PRINTERS</b>"]
        PArray["array.js"]
        PArrayT["array-type.js"]
        PCast["cast-expression.js"]
        PFuncT["function-type.js"]
        PIndexA["indexed-access-type.js"]
        PInfer["infer-type.js"]
        PInter["intersection-type.js"]
        PLit["literal.js (printBigInt)"]
        PMapped["mapped-type.js"]
        PMatch["match.js"]
        PMisc["misc.js"]
        PMod["module.js"]
        POpaque["opaque-type.js"]
        PProp["property.js"]
        PRestE["rest-element.js"]
        PRestT["rest-type.js"]
        PTern["ternary.js"]
        PTuple["tuple.js"]
        PTypeA["type-alias.js"]
        PParam["type-parameters.js"]
        PTypePred["type-predicate.js"]
        PTypeQ["type-query.js"]
        PUnion["union-type.js"]
    end

    %% --- COLLEGAMENTI ---
    Flow --> Document
    Flow --> Utils
    Flow --> SubPrinters

    %% --- STILIZZAZIONE ---
    
    %% Nodo principale: Arancione scuro
    style Flow fill:#fff3e0,stroke:#e65100,stroke-width:4px,color:#e65100

    %% Blocco Document: Blu
    style Document fill:#e1f5fe,stroke:#01579b,stroke-width:2px,color:#01579b
    
    %% Blocco Utilities: Verde
    style Utils fill:#f1f8e9,stroke:#33691e,stroke-width:2px,color:#33691e
    
    %% Blocco SubPrinters: Viola (Il cuore del Ce=35)
    style SubPrinters fill:#f3e5f5,stroke:#4a148c,stroke-width:2px,color:#4a148c

    %% Nodi interni (opzionale per pulizia)
    style DocIdx fill:#fff,stroke:#01579b
    style PArray fill:#fff,stroke:#4a148c