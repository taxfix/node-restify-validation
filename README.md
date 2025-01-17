# node-restify-validation

> *Note*
> Forked from <https://github.com/taxfix/node-restify-validation>

```diff
-Maintener wanted
```
Validation for REST Services built with [node-restify](https://github.com/mcavage/node-restify) in node.js

[![Build Status](https://travis-ci.org/gchauvet/node-restify-validation.png)](https://travis-ci.org/gchauvet/node-restify-validation)
[![Coverage Status](https://coveralls.io/repos/gchauvet/node-restify-validation/badge.png?branch=master)](https://coveralls.io/r/gchauvet/node-restify-validation?branch=master)
[![Dependency Status](https://gemnasium.com/gchauvet/node-restify-validation.png)](https://gemnasium.com/gchauvet/node-restify-validation)

## Requirements
* node-restify-validation requires at least restify 2.6.0 since the validation model is defined in the route-object. (https://github.com/mcavage/node-restify/pull/408)

## Simple request validation with node-restify
Goal of this little project is to have the validation rules / schema as close to the route itself as possible on one hand without messing up the logic with further LOCs on the other hand.

Example:
```javascript
    var server = restify.createServer();
    server.use(restify.queryParser());
    server.use(restifyValidation.validationPlugin( {
        // Shows errors as an array
        errorsAsArray: false,
        // Not exclude incoming variables not specified in validator rules
        forbidUndefinedVariables: false,
        errorHandler: restify.errors.InvalidArgumentError
    }));

    server.get({url: '/test/:name', validation: {
      resources: {
        name: { isRequired: true, isIn: ['foo','bar'] }
      },
      queries : {
        status: { isRequired: true, isIn: ['foo','bar'] },
        email: { isRequired: false, isEmail: true },
        age: { isRequired: true, isNatural: true }
      }
    }}, function (req, res, next) {
        res.send(req.params);
    });

    // Ensure there is a file uploaded:
    server.post({url: '/test/:name', validation: {
      resources: {
        name: { isRequired: true, isIn: ['foo','bar'] }
      },
      files : {
        myfile: { isRequired: true }
      }
    }}, function (req, res, next) {
        res.send(req.params);
    });

    // Checks the body of the request contains a json payload with a `person` object with the following attributes:
    //   firstName - required
    //   middle - optional
    //   lastName - required
    //   emails - optional array of email addresses with at most 5 elements
    server.put({url: '/products/:id/labels/:label', validation: {
      resources: {
        id: { isRequired: true, isNatural: true },
        label: { isRequired: true }
      },
      queries : {
        status: { isRequired: true, isIn: ['foo','bar'] }
      },
      content: {
        person: {
            isObject: {
                properties: {
                    firstName: { isRequired: true },
                    middle: { isRequired: false },
                    lastName: { isRequired: true },
                    emails: {
                        isArray: {
                            maxLength: 5,
                            element: { isEmail: true }
                        }
                    }
                }
            }
        },
        label: { isRequired: true }
      }
    }}, function (req, res, next) {
        res.send(req.params);
    });

    // Validate header fields
    server.get({url: '/test/something/:name', validation: {
      resources: {
        name: { isRequired: true, isIn: ['foo','bar'] }
      },
      headers: {
        requestid: { isRequired: true }
      }
    }}, function (req, res, next) {
        res.send(req.params);
    });

    // Validate header fields
    server.get({url: '/test/something/:name', validation: {
      resources: {
        name: { isRequired: true, isIn: ['foo','bar'] }
      },
      headers: {
        requestid: { isRequired: true }
      }
    }}, function (req, res, next) {
        res.send(req.params);
    });

    server.listen(8001, function () {
        console.log('%s listening at %s', server.name, server.url);
    });
```


## Use
Simply install it through npm

    npm install node-restify-validation


## Documentation powered by swagger
On top of the validation schema the [node-restify-swagger](https://github.com/z0mt3c/node-restify-swagger) library should later-on generate the swagger resources to provide a hands-on documentation.


## Supported validations

```javascript
    isRequired: true | function()
    equalTo: {'fieldName'}
```

Powered by [node-validator](https://github.com/chriso/validator.js).

    contains
    equals
    is
    isAfter
    isAlpha
    isAlphanumeric
    isBefore
    isBoolean
    isCreditCard
    isDate
    isDecimal
    isDivisibleBy
    isEmail
    isFloat
    isHexColor
    isHexadecimal
    isIP
    isIPNet
    isIPv4
    isIPv6
    isIn
    isInt
    isNatural
    isLowercase
    isNumeric
    isUUID
    isUUIDv3
    isUUIDv4
    isUppercase
    isUrl
    max
    min
    not
    notContains
    notIn
    notRegex
    regex

## Nested validation

Validation nesting allows validating content that contains objects and arrays.

### isArray

Validates that specified value is an array.  It can take a boolean value or
an object with the following attributes:

- minLength - minimum number of elements (use with `{ isRequired: true }`)
- maxLength - maximum number of elements
- element - validation specification for elements inside the array


```
{
  content: {
    comments: {
        isArray: true
    },
    emailAddresses: {
        isRequired: true,
        isArray: {
            minLength: 1,
            maxLength: 10,
            element: { isEmail: true }
        }
    }
  }
}
```

### isObject

Validates that the specified value is an object.  It can take a boolean value or
and object with the following attributes:

- properties: validation specification for the contents of the object

```
{
    content: {
        data: {
            isObject: true
        },
        person: {
            isRequired: true,
            isObject: {
                properties: {
                    first: { isRequired: true },
                    last: { isRequired: true },
                    middle: { isRequired: false },
                    address: {
                        isObject: {
                            properties: {
                                street: { isRequired: true },
                                city: { isRequired: true },
                                state: { isRequired: true }
                            }
                        }
                    },
                    emails: {
                        isArray: {
                            maxLength: 5,
                            element: { isEmail: true }
                        }
                    }
                }
            }
        }
    }
}
```

### isDictionary

Validates that specified value is a dictionary.  It can take a boolean value or
an object with the following attributes:

- minLength - minimum number of elements (use with `{ isRequired: true }`)
- maxLength - maximum number of elements
- key - validation specification for the dictionary keys
- value - validation specification for the dictionary values


```
{
  content: {
    users: {
        isDictionary: {
            minLength: 1,
            maxLength: 10,
            key: { isEmail: true },
            value: {
                isObject: {
                    name: { isRequired: true },
                    phone: { isRequired: true },
                    year: { isInt: true }
                }
            }
        }
    }
  }
}
```

## Conditional validations
All validation parameters are able to deal with functions as parameters.

For instance the parameterMatches-Condition:
```javascript
module.exports.paramMatches = function (params) {
    return conditionalChecker(params, function (matches, value) {
        var result;
        if (_.isArray(matches)) {
            result = _.contains(matches, value);
        } else {
            result = _.isEqual(matches, value);
        }
        return result;
    });
};
```
Which will be used for instance as follows:

```javascript
    var validation = isRequired: require('node-restify-validation');
    //...
    parameter: { isRequired: validation.when.paramMatches({(scope: '...',) variable: 'param1', matches: ['a', 'b']}) }
```

As result the parameter will only be required when param1 matches a or b. The called method will have a context (this) containing the following information:

* req: the request object
* scope: (real) current scope validation
* validationModel: the complete validation model
* validationRules: the validationRules for the current atribute
* options: the options which have initially been passed
* recentErrors: errors which have been computed until now

## Models

The request `resource`, `query` and `headers` contain string values, even though they may
be representing numbers, booleans, Dates or other types.

An optional `model` property can be specified to transform the request value to a specific type.
When validation of a property succeeds and a `model` is specified, the original request value
is replaced with the transformed value.

```javascript
// Creates an instance of date from a numeric or string value
function DateModel(value) {
    return new Date(Number(value) || value);
}

server.get({url: '/search', validation: {
    queries: {
        text: { isRequired: true },
        from: { model: DateModel },
        to: { model: DateModel },
        summary: { isBoolean, model: Boolean },
        page: { isNatural: true, model: Number }
    }
}, function (req, res, next) {
    console.log("Query:", JSON.stringify(req.query));
    res.send(req.query);
}))
```

When handling the request `GET /search?text=Hello&from=2017-12-1&to=1514678400000&summary=true&page=3`,
the query property types will be `string`, `number`, `Date`, `Date` and `number`, respectively
and the following will be logged:

```
Query: {"text":"Hello","from":"2017-12-01T06:00:00.000Z","to":"2017-12-31T00:00:00.000Z","summary":true,"page":3}
```

Without specifying the `model` settings, the query property types would have been strings and the following would
have been logged:

```
Query: {"text":"Hello","from":"2017-12-1","to":"1514678400000","summary":"true","page":"3"}
```

### Validator Models

To avoid having to specify models in your validation configurations, you can
configure models to be automatically applied based on the validator.

```javascript
server.use(restifyValidation.validationPlugin({
    validatorModels: {
        isInt: Number
    }
}));
```

When you specify the standard `restifyValidation.validatorModels`, it will supply models
for the following validators:

- isBoolean - converts value to `boolean`
- isDate - converts value to `Date`
- isDecimal - converts value to `number`
- isDivisibleBy - converts value to `number`
- isFloat - converts value to `number`
- isInt - converts value to `number`
- isNatural - converts value to `number`

The `validatorModels` configuration can also take an array of settings.
The following example uses `restifyValidation.validatorModels` but then overrides
the model for the `isDate` validator to map values to `moment` rather than to `Date`.

```javascript
var moment = require('moment');
server.use(restifyValidation.validationPlugin({
    validatorModels: [
        restifyValidation.validatorModels,
        { isDate: moment }
    ]
}));
```

## Inspiration
node-restify-validation was & is inspired by [backbone.validation](https://github.com/thedersen/backbone.validation).
In terms of validation node-restify-validation makes use of [node-validator](https://github.com/chriso/node-validator).

## License
The MIT License (MIT)

Copyright (c) 2014 Timo Behrmann, Guillaume Chauvet

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in
all copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN
THE SOFTWARE.
