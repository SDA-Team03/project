# version: 1.0.0

#  Prettier v3.8.2

*table generation*: `npx dependency-cruiser src --no-config --output-type metrics > dependency_metrics.txt`
## Top 3 modules with most dependencies
*command*: `grep "src/.*\.js" dependency_metrics.txt | sort -k5 -nr | head -n 3`

## Top 3 modules with least dependencies
*command*: `grep "src/.*\.js" dependency_metrics.txt | sort -k5 -n | head -n 3`

### Which files have most or least dependencies? Why?
...

## Knowledge dependencies based
*command*: `git --no-pager log -n 100 --name-only --pretty=format:"--- COMMIT ---"`

### Which are not consistent with code dependencies?
...