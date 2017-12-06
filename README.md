# solid-utils

A library with utilities to write [Solid](https://solid.mit.edu/) apps.

## Requirements

 * [rdflib.js](https://github.com/linkeddata/rdflib.js)
 * [SolidAuthClient]https://github.com/solid/solid-auth-client), for convenience 
this is available in this repository at [lib/solid-auth-client-bundle.js](lib/solid-auth-client-bundle.js).

## Usage

### Login

Initiate login with

    SolidUtils.login()

This requires a `popup.html` file located at `./popup.html` relative to the invoking app. 
See https://github.com/solid/solid-auth-client#building-the-popup on how to generate such a file.

The method returns a promise with the result of the login process.

Note: The `SolidUtils.fetch()` and `SolidUtils.rdfFetch()` will initiate the login
process when needed. So you may not need to explicitely invoke `SolidUtils.login()`.

You can set a function executed after successful login as follows:

    SolidUtils.postLoginAction = function(loginResult) {
        //do something useful
        return loginResult;
    }; 

This function is executed after each successful login, independently if `login()`
is invoked explicitely or if the login process was initiated by a fetch-operation.
Such a function is typically used to keep an area displaying information about the
current user up to date.

### Fetching data

The `SolidUtils.fetch()` method is a replacement for the standard fetch method
that takes care of sending the authorization tokens and initiating the login process
when needed.

The `SolidUtils.rdfFetch()` method will parse an RDF response and provide and add
a property `graph` to the response object.

Example usage:

```JavaScript
SolidAuthClient.currentSession().then(function (session) {
    if (session) {
        var user = $rdf.sym(session.webId);
        SolidUtils.rdfFetch(session.webId).then(function (response) {
            var name = response.graph.any(user, SolidUtils.vocab.foaf('name'));
            console.log("Hello "+name);
        }
    }
}       
```

### Working with LDPCs

LDPCs is the Linked Data Platform equivalent of folders.

Create an LDPC with `SolidUtils.createLdpc(base, name)`, if your not sure if the
LDPC you need already exist, or when you want to create parent LDPCs as well use
`SolidUtils.createPath(base, path)`, `base`must point to an existing LDPC, but 
`path` can contain slashes expressing a hierarchy (e.g. `path/to/my/new/ldpc`.

Example usage:

```JavaScript
//returns a promise for the LDPC where we publish our data
function appDataDir() {
    return SolidUtils.getStorageRootContainer().then(function (root) {
                return SolidUtils.createPath(root.value + "public","where/my/app/publishes/data");
    }
}
```