---
layout: default
title: Data validation
parent: Data models
nav_order: 2
grand_parent: Getting started
permalink: /setup_root/data_models/validation
---


# Custom Validator Function for AJV
{: .no_toc }


## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

# Custom Validator Function for AJV
It is possible to add custom asynchronous validation functions with keyword `asyncValidatorFunction`.

## A Running Example
If there is a model called `example`, the attribute `id` in the model would be validated. Only if its value is `"1"`, it passes the validation. 

In specifically, it should be implemented by two steps:
1. find a file called `example.js` in folder `validations`.
2. add keyword `asyncValidatorFunction` and corresponding asynchronous validation function for attribute `id` in `validatorSchema`.

And the example code is as follows:
```
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