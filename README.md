# Rollup Plugin for Salesforce Platform Lightning Web Components
## Overview
Rollup plugin that resolves Salesforce Platform lwc import paths to file-system import paths, to facilitate the integration of platform components with off-platform tools (build systems, automated testing, etc) that use `rollup.js`.

* `lwc.config.js` file will be used to determine component directories:
  * https://rfcs.lwc.dev/rfcs/lwc/0020-module-resolution
  * Module type `Directory` is of particular interest here, as all other module types will be resolved by another tool.
* `sfdx-project.json` file will be used to determine current component namespace, so that self-references to that namespace will also be resolved as a local component path. Specifically, the `namespace` property is used.

### Logic
* For each file with an import path that "looks" like a Salesforce-platform LWC import path (`namespace/component`):
  * If `namespace == 'c'` OR `namespace == <sfdx-project.json:namespace>`:
    * Search all local component directories, and replace namespace with first discovered path where component exists
  * Else:
    * Search all external component directories (as specified in `lwc.config.json` with type of `module`), checking their `sfdx-project.json:namespace`, and replace namespace with first discovered path where component exists with the target namespace

### Features Checklist
- [ ] **FEATURE**: Replacement of `c` namespace with `force-app/main/default`
- [ ] **ENHANCMENT**: Replacement of `c` namespace with compatible directory from `lwc.config.js`
- [ ] **ENHANCEMENT**: Allow local directory replacement to also work with local namespace (defined in `sfdx-project.json:namespace`)
- [ ] **ENHANCEMENT**: Scan through `node_modules` for directory replacements for external namespaces

## Example
### Given:
#### File Structure
```
📦project
 ┣ 📂force-app
 ┃ ┗ 📂main
 ┃   ┗ 📂default
 ┃     ┗ 📂lwc
 ┃       ┣ 📂alert
 ┃       ┃ ┣ 📜alert.html
 ┃       ┃ ┣ 📜alert.js
 ┃       ┃ ┗ 📜alert.js-meta.xml
 ┃       ┗ 📂app
 ┃         ┣ 📜app.html
 ┃         ┣ 📜app.js
 ┃         ┗ 📜app.js-meta.xml
 ┣ 📜lwc.config.json
 ┣ 📜sfdx-project.json
```
##### alert.js
```javascript
import { LightningElement, api } from 'lwc';

export default class Alert extends LightningElement {
    @api message;
    @api variant;
}
```
##### app.js
```javascript
import { LightningElement } from 'lwc';

import Alert from 'c/alert';

export default class App extends LightningElement {

}
```
##### lwc.config.json
```json
{
    "modules": [
        {
            "dir": "force-app/main/default"
        }
    ]
}
```
##### sfdx-project.json
```json
{
    ...,
    "namespace": "ex",
    ...
}
```
### Expect:
* Import path `c/alert` will be rewritten as `force-app/main/default/lwc/alert`.
* Import path `ex/alert` will be rewritten as `force-app/main/default/lwc/alert`.