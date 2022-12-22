```javascript
npm config set registry http://registry.npmjs.org/
```

## npm install 
**Problems with npm install**
I had the same problem. The reason - wrong proxy was configured and because of that npm was unable to download packages.

So your best bet is to the see the output of
```javascript
$ npm install --verbose
```
and identify the problem. If you have never configured proxy, then possible causes can be:
-   Very outdated npm version;
-   Some problem with your internet connection;
-   Permissions are not sufficient for npm to modify files.

## npm login
To be more helpful, when you type `npm login` and give username and password interactively, npm will generate an auth_token automatically for you and insert it into the .npmrc.

The auth_token is constant given the username/password is the same.

___
Take a look at the `.npmrc` [file](https://docs.npmjs.com/files/npmrc) you can use this file to set npm configuration variables, such as credentials, registry location, etc... This file is located in your **HOME** directory. Here is an example `.npmrc` file to use for reference:

**~/.npmrc**

```bash
registry=https://registry.npmjs.com/
_auth="<token>"
email=<email>
always-auth=true
```

_substitute your email and _auth [token](https://docs.npmjs.com/creating-and-viewing-authentication-tokens) appropriately for your credentials_. Your script will use these global configurations set within your `.npmrc` file.

## npmrc
https://docs.npmjs.com/cli/v9/configuring-npm/npmrc

### Description

npm gets its config settings from the command line, environment variables, and `npmrc` files.

The `npm config` command can be used to update and edit the contents of the user and global npmrc files.

For a list of available configuration options, see [config](https://docs.npmjs.com/cli/v9/using-npm/config).

### [](https://docs.npmjs.com/cli/v9/configuring-npm/npmrc#files)Files

The four relevant files are:

-   per-project config file (/path/to/my/project/.npmrc)
-   per-user config file (~/.npmrc)
-   global config file ($PREFIX/etc/npmrc)
-   npm builtin config file (/path/to/npm/npmrc)

All npm config files are an ini-formatted list of `key = value` parameters. Environment variables can be replaced using `${VARIABLE_NAME}`. For example:

prefix = ${HOME}/.npm-packages

Each of these files is loaded, and config options are resolved in priority order. For example, a setting in the userconfig file would override the setting in the globalconfig file.

Array values are specified by adding "[]" after the key name. For example:

key[] = "first value"

key[] = "second value"

#### [](https://docs.npmjs.com/cli/v9/configuring-npm/npmrc#comments)Comments

Lines in `.npmrc` files are interpreted as comments when they begin with a `;` or `#` character. `.npmrc` files are parsed by [npm/ini](https://github.com/npm/ini), which specifies this comment syntax.

For example:

# last modified: 01 Jan 2016

; Set a new registry for a scoped package

@myscope:registry=https://mycustomregistry.example.org

#### [](https://docs.npmjs.com/cli/v9/configuring-npm/npmrc#per-project-config-file)Per-project config file

When working locally in a project, a `.npmrc` file in the root of the project (ie, a sibling of `node_modules` and `package.json`) will set config values specific to this project.

Note that this only applies to the root of the project that you're running npm in. It has no effect when your module is published. For example, you can't publish a module that forces itself to install globally, or in a different location.

Additionally, this file is not read in global mode, such as when running `npm install -g`.

#### [](https://docs.npmjs.com/cli/v9/configuring-npm/npmrc#per-user-config-file)Per-user config file

`$HOME/.npmrc` (or the `userconfig` param, if set in the environment or on the command line)

#### [](https://docs.npmjs.com/cli/v9/configuring-npm/npmrc#global-config-file)Global config file

`$PREFIX/etc/npmrc` (or the `globalconfig` param, if set above): This file is an ini-file formatted list of `key = value` parameters. Environment variables can be replaced as above.

#### [](https://docs.npmjs.com/cli/v9/configuring-npm/npmrc#built-in-config-file)Built-in config file

`path/to/npm/itself/npmrc`

This is an unchangeable "builtin" configuration file that npm keeps consistent across updates. Set fields in here using the `./configure` script that comes with npm. This is primarily for distribution maintainers to override default configs in a standard and consistent manner.

### [](https://docs.npmjs.com/cli/v9/configuring-npm/npmrc#auth-related-configuration)Auth related configuration

The settings `_auth`, `_authToken`, `username` and `_password` must all be scoped to a specific registry. This ensures that `npm` will never send credentials to the wrong host.

The full list is:

-   `_auth` (base64 authentication string)
-   `_authToken` (authentication token)
-   `username`
-   `_password`
-   `email`
-   `certfile` (path to certificate file)
-   `keyfile` (path to key file)

In order to scope these values, they must be prefixed by a URI fragment. If the credential is meant for any request to a registry on a single host, the scope may look like `//registry.npmjs.org/:`. If it must be scoped to a specific path on the host that path may also be provided, such as `//my-custom-registry.org/unique/path:`.

; bad config

_authToken=MYTOKEN

; good config

@myorg:registry=https://somewhere-else.com/myorg

@another:registry=https://somewhere-else.com/another

//registry.npmjs.org/:_authToken=MYTOKEN

; would apply to both @myorg and @another

; //somewhere-else.com/:_authToken=MYTOKEN

; would apply only to @myorg

//somewhere-else.com/myorg/:_authToken=MYTOKEN1

; would apply only to @another

//somewhere-else.com/another/:_authToken=MYTOKEN2
