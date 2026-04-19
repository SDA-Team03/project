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
```
type    name                                                                          N     Ca     Ce  I(%)        
------- ------------------------------------------------------------------------ ------ ------ ------ ------ 
module  src/language-js/print/flow.js                                                 1      1     35    97%
module  src/language-js/print/typescript.js                                           1      1     33    97%
module  src/language-js/print/estree.js                                               1      1     29    97%
```
The reason why those are the 3 files with the most dependencies is that they are builders, so their role is to take the AST tree and write it, so they have to implement the process in only one module, using a lot of functions created in other modules. As we can see all the files are instable, that is due to the fact that, if one of the files used in them is modified, probably also the file itself will have to be modified

/*
ADD DIAGRAMS
*/