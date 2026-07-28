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

Zendro generates a `validations/<model>.js` file for every model, using [AJV](https://ajv.js.org/) to check each attribute's basic type (e.g. `"river_id": { "type": ["string"] }`). Beyond that, you can add custom asynchronous validation logic for any attribute using the `asyncValidatorFunction` keyword — a Zendro-registered AJV keyword whose value is your own `async` function.

## A running example
Say we have a model called `example` with an `id` attribute, and we only want to accept the literal value `"1"`.

In `validations/example.js`, generated code already looks like this (the exact attributes will differ per model):

```js
module.exports.validator_patch = function(example) {
  example.prototype.validationControl = {
    validateForCreate: true,
    validateForUpdate: true,
    validateForDelete: false,
    validateAfterRead: false
  }

  example.prototype.validatorSchema = {
    "$async": true,
    "type": "object",
    "properties": {
      "id": {
        "type": ["string"]
      }
      // ...other attributes
    }
  }

  example.prototype.asyncValidate = ajv.compile(
    example.prototype.validatorSchema
  )

  // validateForCreate/validateForUpdate/validateAfterRead all call
  // asyncValidate(record) - see the generated file for the full contents.

  return example
}
```

Add `asyncValidatorFunction` to the `id` property inside the existing `validatorSchema.properties` object — don't replace the whole file, just extend what's already there:

```js
"id": {
  "type": ["string"],
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
```

See also [Customize Zendro]({% link getting-started/customizing-zendro.md %}#custom-validations) for how this fits into the broader customization workflow.
