graph LR
    %% Nodo Centrale
    EST["src/language-js/print/estree.js"]
    
    %% --- BLOCCO CORE ENGINE & COMMONS ---
    subgraph EST_Core["<b>CORE ENGINE & AST</b>"]
        CPath["../../common/ast-path.js"]
        CDoc["../../document/index.js"]
        CComm["../../main/comments/print.js (printDanglingComments)"]
        CLoc["../loc.js (locStart, locEnd)"]
    end

    %% --- BLOCCO UTILITIES ---
    subgraph EST_Utils["<b>UTILITIES</b>"]
        UNode["unexpected-node-error.js"]
        UNewl["has-newline.js"]
        UBlock["is-block-comment.js"]
    end

    %% --- BLOCCO SUB-PRINTERS (EFFERENT COUPLING) ---
    subgraph EST_SubPrinters["<b>ESTREE SPECIFIC PRINTERS</b>"]
        EArray["array.js"]
        EArrow["arrow-function.js"]
        EBin["binaryish.js"]
        EBind["bind-expression.js"]
        EBlock["block.js"]
        ECall["call-expression.js"]
        EExpr["expression-statement.js"]
        EHtml["html-binding.js"]
        ELit["literal.js"]
        EMemb["member.js"]
        EObj["object.js"]
        EProp["property.js"]
        ERest["rest-element.js"]
        EStat["statement.js"]
        ETern["ternary.js"]
        EType["type-annotation.js"]
    end

    %% --- COLLEGAMENTI ---
    EST --> EST_Core
    EST --> EST_Utils
    EST --> EST_SubPrinters

    %% --- STILIZZAZIONE ---
    
    %% Nodo principale: Giallo JavaScript (ESTree)
    style EST fill:#fff9c4,stroke:#fbc02d,stroke-width:4px,color:#fbc02d

    %% Blocco Core: Rosso/Rosa (Indica dipendenza critica dal motore principale)
    style EST_Core fill:#ffebee,stroke:#c62828,stroke-width:2px,color:#c62828
    
    %% Blocco Utilities: Verde
    style EST_Utils fill:#f1f8e9,stroke:#33691e,stroke-width:2px,color:#33691e
    
    %% Blocco SubPrinters: Viola (Coerenza con Flow e TS)
    style EST_SubPrinters fill:#f3e5f5,stroke:#4a148c,stroke-width:2px,color:#4a148c