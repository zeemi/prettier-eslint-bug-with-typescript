#Formatting returns corrupted output when run on multiple typescript files 
https://github.com/prettier/prettier-eslint/issues/224

**Versions**:

- `prettier-eslint` version: v9.0.0
- `node` version: v11.10.1
- `npm` (or `yarn`) version: v6.9.0

**Have you followed [the debugging tips](https://github.com/prettier/prettier-eslint/blob/master/README.md#debugging-issues)?**

Yes
Meaningful log:
```
executeOnText returned the following report: Object {
  "errorCount": 1,
  "fixableErrorCount": 0,
  "fixableWarningCount": 0,
  "results": Array [
    Object {
      "errorCount": 1,
      "filePath": "...",
      "fixableErrorCount": 0,
      "fixableWarningCount": 0,
      "messages": Array [
        Object {
          "column": 11,
          "fatal": true,
          "line": 2,
          "message": "Parsing error: ';' expected.",
          "ruleId": null,
          "severity": 2,
        },
      ],
      "output": "const parse = () => {
     eturn {};
};
export default parse;
",
```

**Relevant code or config**

eslint config:
```javascript
module.exports = {
    parser: '@typescript-eslint/parser',
    extends: [
        'plugin:@typescript-eslint/recommended',
    ],
    parserOptions: {
        project: './tsconfig.json',
        tsconfigRootDir: '.',
        ecmaVersion: 2018,  // Allows for the parsing of modern ECMAScript features
        sourceType: 'module',  // Allows for the use of imports
    },
};
```
tsconfig:
```json
{
  "compilerOptions": {
    "outDir": "./dist/",       
    "sourceMap": true,         
    "strictNullChecks": true,
    "module": "es6",           
    "target": "es5",            
    "allowJs": true, 
    "moduleResolution": "node",
    "esModuleInterop":true,
    "strict": true
  },
  "include": [
    "./src/"
  ]
}
```
**What I did:**
Run prettier-eslint on few typescript files:
`prettier-eslint --write '**/*.[tj]s'`

**What happened:**
Reformatting generated corrupted file
input:
```javascript
const parse = ()  => {
  return {};
};
export default parse;
```

output (`return` => `eturn`)
```javasctipt
const parse = () => {
     eturn {};
};
export default parse;
```

**Reproduction repository:**
https://github.com/zeemi/prettier-eslint-bug-with-typescript
<!--
If possible, please create a repository that reproduces the issue with the
minimal amount of code possible.
-->

**Problem description:**
[run by prettier-eslint-cli]
When prettier-eslint is run on each file separately, everything is working great (`npm run format-succ` in reproduction repo)

When prettier-eslint is run on many files at once (via list or glob), some of the files are returned corrupted (`npm run format-fail` in reproduction repo).
During file corruption, parsing error is visible in the logs.

This issue occurs only when `parserOptions.project` property is set in eslint config.
Nevertheless, both: eslint and prettier run from command line works without any issues. 

**Suggested solution:**
Check if prettier-eslint is not holding any state between files formatting that might corrupt next file parsing. 

