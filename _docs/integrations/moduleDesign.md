---
title: Nexd Module Design
category: Nexd Modules
order: 3
---

A nexd module contains a collection of definitions such as integration types, board types, and content types (more details on that later) along with sync and connect functions that the host server uses to interact with the module.

## Collections

A nexd module can supply some or all of the collections listed below.  A collection can be defined as a javascript function that returns a JSON structure or a JSON object.

### Integration Types

The purpose of the integration types collection is to supply connection and runtime meta data for the module.  Integration types are defined as a JSON structure with the name of the integration type as the key of an object inside the `integrationTypes` object.

An integration type defines a `teamSchema` and and a `userSchema`.  These are object defining what needs to be saved to the nexd database, see Pliable documentation for proper schema format.

`userSchema` is used for individual user integration syncs while `teamSchema` is settings that are the same for all users.  Both `userSchema` and `teamSchema` settings will be available to the integration runtime methods.

Example:
```
{
    "salesforce": {
        "teamSchema": {
            "url": {
                "type": "String",
                "index": "not_analyzed",
                "default": "https://na1.salesforce.com"
            },
            "loginUrl": {
                "type": "String",
                "index": "not_analyzed",
                "default": "https://login.salesforce.com"
            },
        },
        "userSchema": {
            "username": {
                "type": "String",
                "index": "not_analyzed",
                "display": {
                    "hidden": false
                }
            },
            "password": {
                "type": "String",
                "index": "not_analyzed",
                "encrypt": true,
                "display": {
                    "hidden": false
                }
            },
            "securityCode": {
                "type": "String",
                "index": "not_analyzed",
                "display": {
                    "hidden": false
                }
            },
            "lastSyncTimeStamp": {
                "type": "String",
                "index": "not_analyzed",
                "display": {
                    "hidden": true
                }
            }
        }
    }
}
```

### Content Types

If an integration wants to create new or custom data types in Nexd they must define the content type schema here.  Similar in form to the integration types discussed above, this collection is a JSON object with content type name as the key of an object holding its schema.  See content type definition docs for more information.

Example:
```
{
    "contact": {
        "searchable": true,
        "board": {
            "boardType": "contact"
        },
        "managementBoards": [{
            "boardType": "contact-manager"
        }],
        "schema": {
            "address": {
                "type": "string"
            },
            "phone": {
                "type": "string"
            },
            "name": {
                "type": "string"
            }
        }
    }   
}
```

### Board Types

If an integration wants to define a new board to be displayed in the UI of Nexd, they must define one or more boards. Similar in form to the other collections discussed above, this collection is a JSON object with board type name as the key of an object holding its schema.  See board type definition docs for more information.

Example: 
```
{
    "account-board": {},
    "contact-board": {}
}
```

### Signals

#### Generator Signals

In modules, signals definitions are included on board definitions however generator signals must be exposed in the module as a function directing the server by signal name to its generator. For example this could be a simple `case switch` or `if else` code block.

Example:
```
var generate = function generate(log, signalDefinition, context, callback) {
    
    if (signalDefinition.name === 'opportunity-stages') {
        return opportunitySignalGenerator.genOppStageSignals(log, signalDefinition, context, callback);
    }
};
```

See signal generator docs for more information.

#### Query Signals

Query signals use the pliable query language to track metrics over time. These signals are also included on board definitions. Reference values are also defined on the same board definition the signals are included on. See the query signal docs for more information but a simple example can be found below:

```
var emailLastWeek = {
    name: 'email-last-week',
    title: 'Number of email last week',
    options: {
        visibility: 'hidden'
    },
    value: {
        reference: 'emailLastWeek.total'
    },
    valueType: "continuous"
};
```

## Module API

Modules must supply a few public functions to allow nexd to connect to and request data from the integration.

### Connect

The connect API will test any credentials supplied by the user or team for their validity.  If no credentials are needed connect can simply return via callback with no errors and the flow will continue.  If oauth completion is needed the connect api should return an error titled `authorizationUrl` and the authorizationUrl in the callback results.  This API will be called before ever call to the module so be sure to make your connect functions handle repeated calls.

Example:
```
var connect = (user, nexdToken, integration, log, callback) => {

    if(integration.needsOauth){
        return callback('authorizationUrl', authUrl)
    }
    
    return callback();
};
```

### Sync

The sync API is designed to run as a background task.  This API is used to sync data with the third party integration the module is communicating with. Sending content to nexd is done through the nexd SDK send events API.  `syncOptions` are passed in this API to indicate special syncs like `syncReset` that expects the module to run a sync without any sync high-water marks or timestamps. This function should return any errors or settings to be updated.

Example:
```
var sync = (user, nexdToken, integration, syncOptions, log, callback) => {
    
    async.injectAuto({
        contentToSave: (cb) => {
            getIntegrationConent(cb);
        },
        sendConent: (contentToSave, cb) => {
            async.each(results, (resultItem, sendCb) => {
                //resultItem manipulation into a defined contentType
                nexd.events.send(resultItem, sendCb);
            }, cb);
        }
    }, (err, results) => {
        var resultsToReturn = {
            settings: newSettings
        };
        
        return callback(err, resultsToReturn);
    });
    
};

```
