# webstore-upload

> Automatically upload new versions of Chrome Extensions or Apps to the Chrome Webstore - integrated in your node project!

> Forked from [c301's grunt-webstore-upload](https://github.com/c301/grunt-webstore-upload) to make it usable as a node module instead of a grunt task.

## Getting Started

### Overview
Read more about great ability to automate this task here: [Chrome Web Store Publish API](http://developer.chrome.com/webstore/using_webstore_api).

**Please note, that you have to upload your extension first time manually, and then provide appID to update ( see below ). Also please make sure, that your draft ready to be published, ie all required fields was populated**

### Install
[With npm](https://www.npmjs.com/package/webstore-upload):
`npm install --save webstore-upload`

### Example
```js

// Promise api
var webstore_upload = require('webstore-upload');

webstore_upload(uploadOptions, loggerFn)
.then(function(result) {
    console.log(result);
    // do somethings nice
    return 'yay';
})
.catch(function(err) {
    console.error(err);
});


// Deprecated callback api - meanwhile it's here for compatability and whoever is already depend on it.
var webstore_upload = require('webstore-upload/deprecated');

webstore_upload(uploadOptions, function(result) {
    console.log('complete!');
    console.log(result);
});
```

```js
var uploadOptions = {
    accounts: {
        default: { //account under this section will be used by default
            publish: true, //publish item right after uploading. default false
            client_id: 'ie204es2mninvnb.apps.googleusercontent.com',
            client_secret: 'LEJDeBHfS'
        },
        other_account: {
            publish: true, //publish item right after uploading. default false
            client_id: 'ie204es2mninvnb.apps.googleusercontent.com',
            client_secret: 'LEJDeBHfS',
            refresh_token: '1/eeeeeeeeeeeeeeeeeeeeeee_aaaaaaaaaaaaaaaaaaa'
        },
        new_account: {
            cli_auth: true, // Use server-less cli prompt go get access token. Default false
            publish: true, //publish item right after uploading. default false
            client_id: 'kie204es2mninvnb.apps.googleusercontent.com',
            client_secret: 'EbDeHfShcj'
        }
    },
    extensions: {
        extension1: {
            //required
            appID: 'jcbeonnlikcefedeaijjln',
            //required, we can use dir name and upload most recent zip file
            zip: 'test/files/test1.zip'
        },
        extension2: {
            account: 'new_account',
            //will rewrite values from 'account' section
            publish: true,
            appID: 'jcbeonnlikcefedeaijjln',
            zip: 'test/files/test2.zip',
            publishTarget: 'trustedTesters'
        }
    },
    uploadExtensions : ['extension2']
};
```

### Tests
Test cases:
* All should work with existing refresh_token
* All should work with creating new token from web
* All should work with creating new token from cmd
* Fail on bad existing refresh_token
* Fail on Non developer account
* Fail on incorrect publishTarget value

### Logger
Can be `'default'`, `'quiet'` or your logger `function` (`winston` and similar):

```js
// info, log, warn, err
logger('log', 'message');
```

### CLI
If you want the cli option, you can use the [original grunt project](https://github.com/c301/grunt-webstore-upload)

### Configuration

|   Name    |   Type    |   Required    |   Default |  Description  |   Notes   |
| :-- | :-- | :-- | :-- | :-- | :-- |
|   **accounts**    |   `Object`    |   `Yes`   |       |   *List of the accounts (see `Accounts` section for details).*    |       |
|   **extensions**  |   `Object`    |   `Yes`   |       |   *List of the extension (see `Extensions` section for details).* |       |
|   **onComplete**  |   `Function`  |   `No`    |       |   *Function that will be executed when all extensions uploaded.*  |   See result example below    |

#### onComplete / Promise result
Array of released extensions passed as argument:
```js
[{
    fileName        : zip,
    extensionName   : options.name,
    extensionId     : options.appID,
    published       : true
}..]
```

#### Accounts
Since Google allows only 20 extensions under one account, you can create multiple records here.
It is object with arbitrary meaningful accounts names as a keys (see example above).
Special account named `default` will be used by defaults.

|   Name    |   Type    |   Required    |   Default |  Description  |   Notes   |
| :-- | :-- | :-- | :-- | :-- | :-- |
|   **publish** |   `Boolean`   |   `No`    |   `false` |   *Make item available at Chrome Webstore or not.*    |       |
|   **client_id**   |   `String`    |   `Yes`   |       |   *Client ID to access to Chrome Console API.*    |   [How to get it](http://developer.chrome.com/webstore/using_webstore_api#beforeyoubegin) |
|   **client_secret**   |   `String`    |   `Yes`   |       |   *Client Secret to access to Chrome Console API.*    |   [How to get it](http://developer.chrome.com/webstore/using_webstore_api#beforeyoubegin) |
|   **refresh_token**   |   `String`    |   `No`    |       |   *Refresh token for the Chrome Console API.* |   [How to get it](http://developer.chrome.com/webstore/using_webstore_api#beforeyoubegin) |

#### Extensions
It is object with arbitrary meaningful extensions names as a keys (see example above).

|   Name    |   Type    |   Required    |   Default |  Description  |   Notes   |
| :-- | :-- | :-- | :-- | :-- | :-- |
|   **appID**   |   `String`    |   `Yes`   |       |   *Extension id or Application id at Chrome Webstore* |       |
|   **zip** |   `String`    |   `Yes`   |       |   *Path to zip file. Upload most recent zip file in case of path is directory*    |       |
|   **zip** |   `String`    |   `Yes`   |       |       |       |
|   **publish** |   `Boolean`   |   `No`    |   `false` |   *Make item available at Chrome Webstore or not. This option under `extensions` will rewrite `publish` under related `account` section.* |       |
|   **publishTarget**   |   `String`    |   `No`    |   `default`   |   *Make item available at Chrome Webstore. Can be `trustedTesters` or `default`*  |   [Publish](https://developer.chrome.com/webstore/webstore_api/items/publish) |
|   **account** |   `String`    |   `No`    |   `default`   |   *Name of the account, that we should use to upload extension.*  |       |



### Workflow
Read more about [Chrome Web Store Publish API](http://developer.chrome.com/webstore/using_webstore_api) and how to get Client ID and Client secret
+ require the module with the configuration (see examples)
+ call the module
+ browser should be opened
+ confirm privileges in browser ( we have to manually do this )
+ wait until uploading will be finished

To automatically pull a new access token using a refresh token just set the `refresh_token` property in your configuration.  If the `refresh_token` is present
it will automatically refresh the token for you without any manual intervention.


## Contributing
* [Bugs, features and etc.](https://github.com/arieljannai/webstore-upload/issues) are welcome!
* In lieu of a formal styleguide, take care to maintain the existing coding style. Add unit tests for any new or changed functionality.

## License
**Mine:** Copyright (c) 2016 Ariel Jannai. Licensed under the MIT license.

**Original project:** Copyright (c) 2014 Anton Sivolapov. Licensed under the MIT license.
