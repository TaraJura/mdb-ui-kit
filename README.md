# Material design Bootstrap

## Installation

Add to `package.json` dependencies:
`"mdb-ui-kit": "git+ssh://git@gitlab.vmin.cz:matous/mdb-ui-kit.git"`

Enable the project's deploy key in this repo to allow the pipeline to install this package:
`Settings -> Repository -> Deploy keys`

## Adding to an application

The automatic initialization runs when the Javascript is loaded. Some components will not be initialized automatically if MDB is not imported at th eend of the `body` tag.

If MDB is imported in the `head`, like in a typical Rails app, these components will need to be initialized manually. When using Turbolinks or Turbo, manual initialization is also necessary on `turbolinks:load` as well as manual dispose on `turbolink:before-cache`.

Known components that need to be initialized manually:
- Sidenav
- Datatable

This code takes care of initializing and disposing MDB components with Turbolinks:

```javascript
import * as mdb from 'mdb-ui-kit/src/js/mdb.pro'

document.addEventListener('turbolinks:load', () => {
  initializeMdb(document.body);
})

document.addEventListener('turbolinks:before-cache', () => {
  disposeMdb()
})

// To initialize mdb components that are not properly initialized automatically,
// add the selector and component name pair to the selectorMap below.
// Configuration is taken from the relevant data attributes on the elements.
const selectorMap = {
  '.sidenav': 'Sidenav',
  '.datatable': 'Datatable'
}

function initializeMdb(container) {
  for (const selector in selectorMap) {
    const elements = container.querySelectorAll(selector)
    const component = mdb[selectorMap[selector]]
    elements.forEach(element => new component(element))
  }
}

function disposeMdb() {
  for (const selector in selectorMap) {
    const elements = document.querySelectorAll(selector)
    const component = mdb[selectorMap[selector]]
    elements.forEach(element => {
      const existingInstance = component.getInstance(element)
      if (existingInstance) { existingInstance.dispose() }
    })
  }
}
```

## Modified components

Some components need to be modified to work properly with Turbolinks. Import the `src/js/` files, since the modifications are not present in the compiled `.min` files.

### Datatable

The original implementation does not return the HTML to its initial state when the Datatable is disposed. This is fixed here.

### Sidenav

The active link keeps its highlight when clicking the back button. This results in multiple links being highlighted at the same time. This is fixed here.

### Lightbox

Icon font changed to `Material Icons` due to incompatible Font Awesome versions. MDB used 6, Lightbox used 5. Version 6 is loaded as 'svg', version 5 use 'pseudo class :before'.
Material Icons needs to be added to your project: eg. `material-design-icons-iconfont": "^6.1.0"`

### Manipulator.js

`mdb-ui-kit/src/js/mdb/dom/manipulator.js`

Add Array parse for `mdb-data-attribute=[Array]`

### Chips

By default, a chip is added when the input is blurred. This breaks autocomplete, so the feature has been removed and it is only possible to add a chip by pressing Enter.

### Range

Range did not originally remove the thumb element when disposed.
