# swagger-autogen

This module performs the automatic construction of the Swagger documentation. The module is able to identify the endpoints and automatically capture methods such as get, post, put, and so on, along with the path. In addition to the methods and the path, the module can also identify the parameters in the path, query, body and response status code. It is possible to add information such as: endpoint description, parameter description, defininitions, security, among others. It is also possible to ignore or disable the automatic capture of an endpoint (having to manually inform each information in this case). The module generates a *.json* file with the documentation in the swagger format.

[![NPM Version](http://img.shields.io/npm/v/swagger-autogen.svg?style=flat)](https://www.npmjs.org/package/commander) [![NPM Downloads](https://img.shields.io/npm/dm/swagger-autogen.svg?style=flat)](https://npmcharts.com/compare/swagger-autogen?minimal=true)

## Contents

- [Installation](#installation)
- [Usage](#usage)
  - [Usage (Basic)](#usage-basic)
  - [Usage (With Optionals)](#usage-with-optionals)
- [Building documentation without starting the project](#building-documentation-without-starting-the-project)
- [Building documentation along with the project](#building-documentation-along-with-the-project)
- [Endpoints](#endpoints)
  - [Automatic capture](#automatic-capture)
  - [Description](#description)
  - [Tags](#tags)
  - [Parameters](#parameters)
  - [Responses](#responses)
  - [Consumes and Produces](#consumes-and-produces)
  - [Schema and Definitions](#schema-and-definitions)
  - [Endpoint with referenced callback](#endpoint-with-referenced-callback)
  - [Endpoint as deprecated](#endpoint-as-deprecated)
  - [Ignoring endpoint](#ignoring-endpoint)
  - [Manual capture](#manual-capture)
  - [Creating endpoint](#creating-endpoint)
  - [Multiple patterns](#multiple-patterns)
- [Security](#security)
- [Response Language](#response-language)
- [Complete example](#complete-example)
- [Compatibility](#compatibility)
- [Bug fixes and features](#bug-fixes-and-features)
- [Help us!](#help-us)
- [License](#license)

## Installation

This is a [Node.js](https://nodejs.org/en/) module available through the [npm](https://www.npmjs.com/).

```bash
$ npm install --save swagger-autogen
```

It is loaded using the require() function:

```js
const swaggerAutogen = require('swagger-autogen')()
```

## Usage
The two sections below will show the most basic and most complete use of this module.

Function signature: `const swaggerAutogen: (outputFile: <string>, endpointsFiles: <Array of string>, data: <object>) => Promise <any>`

**outputFile:** (Required*). Output file. It will be the file generated by the module containing the documentation in the format identified by Swagger.

**endpointsFiles:** (Required*). Files containing the endpoints. These are the files that contain methods such as get, post, put, and so on, for example: `app.get('/path', ...)` or `route.post('/path', ...)`.

**doc:** (Not Required). Object containing the details of the documentation. If not informed, or if any parameter of the object is omitted, the default values ​​will be used. (See: [Usage (With optionals)](#usage-with-optionals) section)

### Usage (Basic)
The code below must be inserted in a separate file, for example: `swagger.js`. For example:

`File: swagger.js`
```js
const swaggerAutogen = require('swagger-autogen')()

const doc = {
    info: {
        title: "My API",
        description: "Description"
    },
    host: "localhost:3000",
    schemes: ['http']
}

const outputFile = './path/swagger-output.json'
const endpointsFiles = ['./path/endpointsUser.js', './path/endpointsBook.js']

swaggerAutogen(outputFile, endpointsFiles, doc)
```

### Usage (With optionals)
The code below must be inserted in a separate file, for example: `swagger.js`. For example:

`File: swagger.js`
```js
const swaggerAutogen = require('swagger-autogen')()

const doc = {
    info: {
        "version": "1.0.0",           // by default: "1.0.0"
        "title": "My API",            // by default: ""
        "description": "Description"  // by default: ""
    },
    host: "",                         // by default: "localhost:3000"
    basePath: "",                     // by default: "/"
    schemes: [],                      // by default: ['http']
    consumes: [],                     // by default: ['application/json']
    produces: [],                     // by default: ['application/json']
    tags: [                           // by default: empty Array
        {
            "name": "",               // Tag name
            "description": ""         // Tag description
        },
        // { ... }
    ],
    securityDefinitions: { },         // by default: empty object
    definitions: { }                  // by default: empty object
}

const outputFile = './path/swagger-output.json'
const endpointsFiles = ['./path/endpointsUser.js', './path/endpointsBook.js']

swaggerAutogen(outputFile, endpointsFiles, doc)
```

## Building documentation without starting the project
To build the documentation without starting your project, add the following script to your project's `package.json` file:

`File: package.json`
```js
    // { ... },
    "scripts": {
        // ... ,
        "swagger-autogen": "node ./swagger.js"
    }
```

Where `./swagger.js` is the file containing the `swaggerAutogen(...)` function call (see section [Usage](#usage). After that, at the root of your project, run the following command:

```bash
$ npm run swagger-autogen
``` 

## Building documentation along with the project
To build the documentation before the project starts and immediately start it, rewrite the `swaggerAutogen(...)` function as follows:

```js
// ...
swaggerAutogen(outputFile, endpointsFiles, doc).then( () => {
    require('./index.js')           // Your project's root file
})
```

Where `index.js` is your project's root file. Change the `start` script in your project's `package.json` to point to the file containing the `swaggerAutogen(...)` function. If you use Visual Studio Code, change the reference in your `launch.json` in the same way. Now, just run your project as usual. With that the documentation will be generated and soon after the project will start, making the documentation always updated when the project is started.

See: [Complete example](https://github.com/davibaltar/example-swagger-autogen)

## Endpoints
The way to configure the module is done within comments, and can be in the format `// ...` or `/* ... */`. The used pattern will be `#swagger.something` tag. Each comment can contain one or more `#swagger.something` tags. **NOTE:** ALL COMMENTS CONTAINING `#swagger.something` MUST BE WITHIN OF FUNCTIONS.

### Automatic capture
In this case it is not necessary to do anything. Considering, for example, the file containing endpoints:

```js
    // ...

    app.get('/users', (req, res) => {

        users.adduser(req.query.obj)

        if(...)
            return res.status(201).send(data)
        return res.status(500).send(false)
    })

    app.get('/users/:id', (req, res) => {

        if(...)
            return res.status(200).send(data)
        return res.status(404).send(false)
    })
```

The capture of the method, path, parameters and status of the response will be automatic.

See [Complete example here!](#complete-example)

### Description
This is the description of the Endpoint. To add it, use the `#swagger.description` tag, for example:

```js
    // ...

    app.get('/users/:id', (req, res) => {
        // #swagger.description = 'Endpoint used to obtain a user.'

        if(...)
            return res.status(200).send(data)
        return res.status(404).send(false)
    })
```

See [Complete example here!](#complete-example)

### Tags
To inform which tags the endpoinst belongs to, use the `#swagger.tags` tag, for example:

```js
    // ...

    app.get('/users', (req, res) => {
        /* #swagger.tags = ['Users'] */

        if(...)
            return res.status(200).send(data)
        return res.status(500).send(false)
    })
```

See [Complete example here!](#complete-example)

### Parameters
It is possible to create or complement automatically detected parameters. Use the `#swagger.parameters['parameterName']` tag to create a new parameter or to complete an existing parameter (automatically detected).

All optional parameters for the tag parameter:
```js
/* #swagger.parameters['parameterName'] = {
    in: string,
    description: string,
    required: boolean,
    type: string,
    format: string,
    schema: object
} */
```

**in:** Values: 'path', 'query' or 'body'  
**description:** The parameter description  
**required:** Values: true or false  
**type:** Values: 'string', 'integer', 'object', etc.  
**format:** Values: 'int64', etc.  
**schema:** Values: See section [Schema and Definitions](#schema-and-definitions)  

For example:
```js
    app.get('/users/:id', (req, res) => {
        //  #swagger.parameters['id'] = { description: "User ID" } 

        if(...)
            return res.status(200).send(data)
        return res.status(404).send(false)
    })

    app.post('/users', (req, res) => {
        /*  #swagger.parameters['obj'] = { 
                in: 'body',
                type: "object",
                description: "User data"
        } */

        users.adduser(req.body)

        if(...)
            return res.status(201).send(data)
        return res.status(500).send(false)
    })
```

See [Complete example here!](#complete-example)

### Responses
It is possible to create or complement automatically detected responses. Use the `#swagger.reponses[statusCode]` tag to create a new answer or to complete an existing answer (automatically detected), for example:

```js
    // ...

    app.get('/users/:id', (req, res) => {

        if(...) {
            /* #swagger.responses[200] = { 
                    description: 'User successfully obtained.',
                    schema: { "$ref": "#/definitions/User" } 
            } */
            return res.status(200).send(data)
        }
        return res.status(404).send(false)
    })

    app.post('/v2/users', (req, res) => {
        
        users.adduser(req.query.obj)

        if(...){
            // #swagger.responses[201] = { description: 'User registered successfully.' }
            return res.status(201).send(data)
        }

        // #swagger.responses[500] = { description: 'Problem with the server.' }
        return res.status(500)
    })
```

**NOTE:** For more information about `schema` and` definitions`, see the section: [Schema and Definitions](#schema-and-definitions)  

**NOTE:** As the 404 status description was not entered, "Not Found" will automatically be added. It is possible to change the language of the automatic response, see the [Response Language](#response-language) section.  

See [Complete example here!](#complete-example)

### Consumes and Produces
Use the `#swagger.produces = ['contentType']` or `#swagger.consumes = ['contentType']` tag to add a new produce or a new consume, respectively. In the example below, the first two endpoints will have the same result in the documentation:

```js
    // ...
    
    app.get('/users/:id', (req, res) => {
        res.setHeader('Content-Type', 'application/xml')

        if(...) 
            return res.status(200).send(data)
        return res.status(404).send(false)
    })

    app.get('/v2/users/:id', (req, res) => {
        // #swagger.consumes = ['application/xml']

        if(...) 
            return res.status(200).send(data)
        return res.status(404).send(false)
    })
```

See [Complete example here!](#complete-example)

### Schema and Definitions
Unlike how Swagger writes, the answers in this module are added in a simpler way, that is, in the way you want to see the result. These responses can be added to the `definitions` parameter of the `doc` object seen in the [Usage](#usage) section, or directly to the response via the `schema` parameter.

**About Examples and Types in schema:** The example comes right in front of the parameter declaration, and the type is abstracted according to the `typeof` of the example. In the code below, the parameter `name` will have as an example `"Jhon Doe"` and type `string`, while `age` will have as an example `29` and type `number`.

**NOTE:** To configure a parameter as **required**, just add the symbol `$` before the parameter, for example: `$name = "Jhon Doe"`.

```js
const doc = {
    // { ... },
    definitions: {
        Parents: {
            father: "Simon Doe",
            mother: "Marie Doe"
        },
        User: {
            name: "Jhon Doe",
            age: 29,
            parents: {
                $ref: '#/definitions/Parents'
            },
            diplomas: [
                {
                    school: "XYZ University",
                    year: 2020,
                    completed: true,
                    internship: {
                        hours: 290,
                        location: "XYZ Company"
                    }
                }
            ]
        },
        AddUser: {
            $name: "Jhon Doe",
            $age: 29,
            about: ""
        },
       // { ... }
    }
}
```

`Endpoint file:`
```js
    app.get('/users/:id', (req, res) => {
        if(...) {
            /* #swagger.responses[200] = { 
                schema: { "$ref": "#/definitions/User" }, 
                description: 'User successfully obtained.' } */
            return res.status(200).send(data)
        }
        return res.status(404).send(false)
    })

    app.get('/v2/users/:id', (req, res) => {
        if(...) {
            // Inserting directly, without using definitions:

            /* #swagger.responses[200] = { 
                description: "User successfully obtained.", 
                schema: {
                    name: "Jhon Doe",
                    age: 29,
                    parents: {
                        father: "Simon Doe",
                        mother: "Marie Doe"
                    },
                    diplomas: [
                        {
                            school: "XYZ University",
                            year: 2020,
                            completed: true,
                            internship: {
                                hours: 290,
                                location: "XYZ Company"
                            }
                        }
                    ]
                }
            }*/
            return res.status(200).send(data)
        }
        return res.status(404).send(false)
    })
    
    app.post('/users', (req, res) => {
        /*    #swagger.parameters['obj'] = { 
                in: 'body',
                description: "User data.",
                schema: { "$ref": "#/definitions/AddUser" }
        } */
        users.adduser(req.body)

        if(...)
            return res.status(201).send(data)
        return res.status(500).send(false)
    })

    app.post('/v2/users', (req, res) => {
        // Inserting directly, without using definitions:

        /*    #swagger.parameters['obj'] = { 
                in: 'body',
                description: "Adding new user.",
                schema: {
                    $name: "Jhon Doe",
                    $age: 29,
                    about: ""
                }
        } */
        users.adduser(req.body)

        if(...)
            return res.status(201).send(data)
        return res.status(500).send(false)
    })
```

See [Complete example here!](#complete-example)

### Endpoint with referenced callback
In case of endpoint with referenced callback it is necessary to add the information manually, for example:

`Before`
```js
    // ...

    function myFunction(req, res, next) => {
        users.adduser(req.body)
        if(...)
            return res.status(201).send(data)
        return res.status(500).send(false)
    })

    app.put('/users/:id', myFunction)
```

`After`
```js
    // ...

    function myFunction(req, res, next) => {
        users.adduser(req.body)
        if(...)
            return res.status(201).send(data)
        return res.status(500).send(false)
    })

    app.put('/users/:id', myFunction
        /*  #swagger.parameters['id'] = { description: "User ID." }

            #swagger.parameters['obj'] = {
                in: 'body',
                description: "User data.",
                schema: { "$ref": "#/definitions/AddUser" }
            }
            
            #swagger.responses[201] = { description: "User updated successfully."}
            #swagger.responses[500] = { description: "Server failure."}
        */
    )
```

See [Complete example here!](#complete-example)

### Endpoint as deprecated
Use the `#swagger.deprecated = true` tag to inform that a given endpoint is depreciated, for example:

```js
    // ...

    app.get('/users/:id', (req, res) => {
        // #swagger.deprecated = true

        if(...)
            return res.status(200).send(data)
        return res.status(404).send(false)
    })
```

See [Complete example here!](#complete-example)

### Ignoring endpoint
Use the `#swagger.ignore = true` tag to ignore a given endpoint. Thus, it will not appear in the documentation, for example:

```js
    // ...

    app.get('/users/:id', (req, res) => {
        // #swagger.ignore = true

        if(...)
            return res.status(200).send(data)
        return res.status(404).send(false)
    })
```

See [Complete example here!](#complete-example)

### Manual capture
Use the `#swagger.auto = false` tag to disable automatic recognition. With that, all parameters of the endpoint must be informed manually, for example:

```js
    app.put('/users/:id', (req, res) => {
        /*  #swagger.auto = false

            #swagger.path = '/users/{id}'
            #swagger.method = 'put'
            #swagger.produces = ["application/json"]
            #swagger.consumes = ["application/json"]
            
            #swagger.parameters['id'] = {
                in: 'path',
                description: 'User ID.',
                required: true
            }

            #swagger.parameters['obj'] = {
                in: 'body',
                description: 'User data.',
                required: true, 
                type: 'string'
            }
        */
                
        if(...) {   
            // #swagger.responses[201] = { description: "User registered successfully." }
            return res.status(201).send(data)
        }
        // #swagger.responses[500] = { description: "Server failure."}
        return res.status(500).send(false)
    })
```

See [Complete example here!](#complete-example)

### Creating endpoint
If you want to forcibly create an endpoint, use the  `#swagger.start` and` #swagger.end` tags, for example:

```js
function myFunction(param) {
    // #swagger.start

    /*
        #swagger.path = '/forcedEndpoint/{id}'
        #swagger.method = 'put'
        #swagger.description = 'Forced endpoint.'
        #swagger.produces = ["application/json"]
    */

    /*  #swagger.parameters['id'] = {
            in: 'path',
            description: 'User ID.' } */
    const dataId = users.getUser(req.params.id)

    /*  #swagger.parameters['obj'] = { 
            in: 'body',
            description: 'User data.',
            type: 'object',
            schema: { $ref: "#/definitions/AddUser" }
    } */
    const dataObj = users.getUser(req.query.obj)

    if (expression)
        return res.status(200).send(true)    // #swagger.responses[200]
    return res.status(404).send(false)       // #swagger.responses[404]

    // #swagger.end
}
```

See [Complete example here!](#complete-example)

## Multiple patterns
If the file containing the endpoints contains multiple patterns before of method, use the `#swagger.patterns` tag, for example:

```js
    const lib = require(...)
    
    // #swagger.patterns = ['app', 'route']

    // ...

    app.get('/users/:id', (req, res) => {
        if(...)
            return res.status(200).send(data)
        return res.status(404).send(false)
    })

    route.get('/test', (req, res) => {
        if(...)
            return res.status(201).send(data)
        return res.status(500).send(false)
    })
```

See [Complete example here!](#complete-example)

## Security
It is possible to add security to endpoints. The security example below was taken from the original Swagger documentation.

```js
const doc = {
    // { ... },
    securityDefinitions: {
        api_key: {
            type: "apiKey",
            name: "api_key",
            in: "header"
        },
        petstore_auth: {
            type: "oauth2",
            authorizationUrl: "https://petstore.swagger.io/oauth/authorize",
            flow: "implicit",
            scopes: {
                read_pets: "read your pets",
                write_pets: "modify pets in your account"
            }
        }
    },
}
```

At the endpoint, add the `#swagger.security` tag, for example:

```js
    app.get('/users/:id', (req, res) => {
        
        /* #swagger.security = [{
            "petstore_auth": [
                "write_pets",
                "read_pets"
            ]
        }] */

        if (...)
            return res.status(200).send(true)
        return res.status(404).send(false)
    })
```

## Response Language
It is possible to change the default language (English) of the description in the automatic response, for example: status code 404, the description will be: 'Not Found'. To change, just do in the module declaration:

```js
const swaggerAutogen = require('swagger-autogen')('pt-BR')  // Portuguese - Brazil
// In this case, for example, the description of status code 404 will be: 'Não Encontrado'
```

OR

```js
const swaggerAutogen = require('swagger-autogen')('zh-CN')  // Chinese (Simplified)
// In this case, for example, the description of status code 404 will be: '未找到'
```

OR

```js
const swaggerAutogen = require('swagger-autogen')()  // English by default
// In this case, for example, the description of status code 404 will be: 'Not Found'
```

For now the module has only the languages: English, Portuguese (Brazil) and Chinese (Simplified).

## Complete example
Link to a project that covers the simplest use of this module as well as the most complete use. See the link below:

[Complete example](https://github.com/davibaltar/example-swagger-autogen)

Directory structure:
```
*
|-- swagger.js
|-- index.js
|-- package.json
|-- src
|     |-- endpoints.js
|     |-- users.js
```

`File: endpoints.js`
```js
const users = require('./users')
let expression = true


module.exports = function (app) {

    /* NOTE: 100% automatic */
    app.get('/automatic/users/:id', (req, res) => {
        res.setHeader('Content-Type', 'application/json')
        const dataId = users.getUser(req.params.id)
        const dataObj = users.getUser(req.query.obj)

        if (expression)
            return res.status(200).send(true)
        return res.status(404).send(false)
    })

    /* NOTE: 100% automatic */
    app.post('/automatic/users', (req, res) => {
        res.setHeader('Content-Type', 'application/xml')
        const data = users.addUser(req.query.obj)

        if (expression)
            return res.status(201).send(data)
        return res.status(500)
    })

    /* NOTE: Completing informations automaticaly obtaineds */
    app.get('/automatic_and_incremented/users/:id', (req, res) => {
        /* #swagger.tags = ['User']
           #swagger.description = 'Endpoint to get the specific user.' */
        res.setHeader('Content-Type', 'application/json')
        const data = users.getUser(req.params.id)

        if (expression) {
            /* #swagger.responses[200] = { 
                    schema: { "$ref": "#/definitions/User" },
                    description: "User registered successfully." } */
            return res.status(200).send(data)
        }
        return res.status(404).send(false)    // #swagger.responses[404]
    })

    /* NOTE: Completing informations automaticaly obtaineds */
    app.post('/automatic_and_incremented/users', (req, res) => {
        res.setHeader('Content-Type', 'application/xml')
        /*  #swagger.tags = ['User']
            #swagger.description = 'Endpoint to add a user.' */

        /*  #swagger.parameters['obj'] = {
                in: 'body',
                description: 'User information.',
                required: true,
                type: 'object',
                schema: { $ref: "#/definitions/AddUser" }
        } */
        const data = users.addUser(req.body)

        if (expression) {
            // #swagger.responses[201] = { description: 'User registered successfully.' }
            return res.status(201).send(data)
        }
        return res.status(500)    // #swagger.responses[500]
    })

    /* NOTE: Function with callback referencied */
    app.delete('/automatic_and_incremented/users/:id', myFunction1
    /*  #swagger.tags = ['User']
        #swagger.parameters['id'] = {
            description: 'User ID.'
        }
        
        #swagger.responses[200]
        #swagger.responses[404]
    */)

    /* NOTE: Will be ignored in the build */
    app.get('/toIgnore', (req, res) => {
        // #swagger.ignore = true
        res.setHeader('Content-Type', 'application/json')

        if (expression)
            return res.status(200).send(true)
        return res.status(404).send(false)
    })

    app.patch('/manual/users/:id', (req, res) => {
        /*  #swagger.auto = false

            #swagger.path = '/manual/users/{id}'
            #swagger.method = 'patch'
            #swagger.description = 'Endpoint added manually.'
            #swagger.produces = ["application/json"]
            #swagger.consumes = ["application/json"]
        */

        /*  #swagger.parameters['id'] = {
                in: 'path',
                description: 'User ID.',
                required: true
            }
        */

        /*  #swagger.parameters['obj'] = {
                in: 'query',
                description: 'User information.',
                required: true, 
                type: 'string'
            }
        */

        if (expression) {
            /* #swagger.responses[200] = { 
                    schema: { "$ref": "#/definitions/User" }, 
                    description: "User found." 
            }*/
            return res.status(200).send(data)
        }
        // #swagger.responses[500] = { description: "Server Failure." }
        return res.status(500).send(false)
    })

    app.head('/security', (req, res) => {
        res.setHeader('Content-Type', 'application/json')
        /* #swagger.security = [{
            "petstore_auth": [
                "write_pets",
                "read_pets"
            ]
        }] */

        const dataObj = users.getUser(req.query.obj)

        if (expression)
            return res.status(200).send(true)
        return res.status(404).send(false)
    })
}

function myFunction1(p) {
    const dataId = users.getUser(req.params.id)

    if (expression)
        return res.status(200).send(true)
    return res.status(404).send(false)
}

function myFunction2(p) {
    // #swagger.start

    /*
       #swagger.path = '/forcedEndpoint/{id}'
       #swagger.method = 'put'
       #swagger.description = 'Forced endpoint.'
       #swagger.produces = ["application/json"]
    */

    /* #swagger.parameters['id'] = { in: 'path', description: 'User ID' } */
    const dataId = users.getUser(req.params.id)

    /* #swagger.parameters['obj'] = { 
           in: 'body',
           description: 'User information.',
           type: 'object',
           schema: {
               $name: "Jhon Doe",
               $age: 29,
               about: ""
            }
    } */
    const dataObj = users.getUser(req.body)

    if (expression)
        return res.status(200).send(true)   // #swagger.responses[200]
    return res.status(404).send(false)      // #swagger.responses[404]

    // #swagger.end
}
```
See the result after construction in the image below:

![](https://raw.githubusercontent.com/davibaltar/public-store/master/screen-swagger-autogen-small.png)

---

![](https://raw.githubusercontent.com/davibaltar/public-store/master/screen-swagger-autogen-cut.png)


## Compatibility
This module is independent of any framework. For the recognition to be **automatic**, it is necessary that your fremawork follow the pattern `foo.method(path, callback)`, where `foo` is the variable belonging to the server or the route, such as: `app`, `server`, `route`, etc. The `method` are HTTP methods, such as get, post, put, and so on. If the `foo.method(path, callback)` pattern is not found in the files, it will be necessary to **manually** enter the beginning and end of the endpoint using the `#swagger.start` and `#swagger.start` tags (see the section: [Creating endpoint](#creating-endpoint)). If you use the `Express.js` fremework, the status code and produces will be automaticaly obtained according to the `status()` and `setHeader()` functions, respectively. If you use a framework that does not contain these functions, you will need to manually add them with the `#swagger.response[statusCode]` and `#swagger.produces` tags (see the [Responses](#responses) and [Consumes and Produces](#consumes-and-produces) sections).

**Swagger version:** 2.0  

## Bug fixes and features
Version 1.x.x will focus on bug fixes.
- Version 1.0.8: Bug fix in definitions
- Version 1.0.9: Allows a definition to reference another definition with: $ref: '#/definitions/_obj_'

## Help us!
Help us improve this module. If you have any information that the module does not provide or provides incompletely or incorrectly, please use our [Github](https://github.com/davibaltar/swagger-autogen) repository.

**pt-Br:**
Ajude-nos a melhorar este módulo. Se você tiver alguma informação que o módulo não forneça ou forneça de maneira incompleta ou incorreta, use nosso repositório do [Github](https://github.com/davibaltar/swagger-autogen). Pode enviar em português Brasil também! :)

Repository: https://github.com/davibaltar/swagger-autogen

## License
[MIT] (LICENSE) License