---
layout: default
title: Data validation
parent: Data models
nav_order: 4
permalink: /data-models/validation
---

# Custom Validator Function for AJV
{: .no_toc }

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

It is possible to add custom asynchronous validation functions using the `asyncValidatorFunction` keyword.

## A running example
If there is a model called `example`, the attribute `id` in the model would be validated. Only if its value is `"1"` does it pass validation.

Implementing this takes two steps:
1. Find the file `example.js` in the `validations` folder.
2. Add the `asyncValidatorFunction` keyword and the corresponding asynchronous validation function for attribute `id` in `validatorSchema`.

Example code:
```js
example.prototype.validatorSchema = {
  "$async": true,
  "properties": {
    "id": {
      "asyncValidatorFunction": async function(data) {
        if (data === "1") {
          return true
        } else {
          return new Promise(function(resolve, reject) {
            return reject(new Ajv.ValidationError([{
              keyword: 'asyncValidatorFunction',
              message: `${data} is not 1`
            }]))
          })
        }
      }
    }
  }
}
```

See also [Customize Zendro]({% link getting-started/customizing-zendro.md %}#custom-validations) for how this fits into the broader customization workflow.
