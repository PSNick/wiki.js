---
title: Flags
description: 
published: true
date: 2022-05-19T13:27:04.256Z
tags: development, api, documentation, docs
editor: markdown
dateCreated: 2021-11-17T14:27:34.646Z
---

# Flag

![Up to date as of v9](https://img.shields.io/static/v1?label=FoundryVTT&message=v9&color=informational)

## Overview

Flags are the safest way that packages can store arbitrary data on existing documents. If a package allows the user to set some data which isn't normally on the [Document](/en/development/api/document)'s data schema, it should leverage flags.


> Note that this does not necessarily apply to Systems when talking about the Actor and Item document types. Systems can provide a `template.json` which allows the system to define the data schema for Actors and Items.
>
> _For more information about this data template, see the [Intro to System Development](https://foundryvtt.com/article/system-development/#data) article in the Foundry KB._
{.is-warning}

A flag does not have to be a specific type, anything which can be `JSON.stringify`ed is valid.

---

## Key Concepts

### Data Format
Flags live on the root of a Document's [DocumentData](/en/development/api/document#document-data) schema, on the same level as `name`.

```text
Document
├─ id
├─ parent
└─ data
   ├─ name
   ├─ flags <---
   └─ someOtherSchemaKey (e.g. `data`)
```

The `flags` object is keyed by scope as set in `setFlag`. Its values are objects keyed by flag name as set in `setFlag`.

```js
flags: {
  <scope>: {
    <flag name>: value
  }
}
```

---

## API Interactions

### Setting a flag's value
Flags are automatically namespaced within the first parameter given to [`Document#setFlag`](https://foundryvtt.com/api/abstract.Document.html#setFlag).

The following are expected scopes:
- `core`
- `world`
- The `id` of the world's system.
- The `name` of a module present in `game.modules` - Note that this does not need to be an active module.

If an unexpected scope is provided, Foundry core will `throw` an error.

```js
const newFlagValue = 'foo';

someDocument.setFlag('myModuleName', 'myFlagName', newFlagValue);
```


> There are some caveats and pitfalls to be aware of when interacting with objects stored in flags.
>
> _See [Some Details about `setFlag` and objects]() below for more information._
{.is-info}

#### Without `setFlag`

##### As part of a Document Update

> Manually updating a Document's `flags` value does not have any of the client-side validations that `setFlag` has.
>
> For instance, nothing prevents an update from setting `flags` to be a `string` or `array` instead of the expected format of scope-namespaced objects. 
> 
> Be careful when interacting with flags manually to keep this structure lest you accidentally break someone else's package (or they yours).
{.is-warning}

Changing a flag value during a normal [document update](https://foundryvtt.wiki/en/development/api/document#update-the-database) is a way to set multiple flags at once, or to both update the flag and standard schema data at the same time.

To do so, the data fed to the `Document#update` method should target the specific namespaced object and flag name being edited.

```js
const updateData = {
  ['flags.myModuleName.myFlagName']: newFlagValue,
};

// is the same as

const updateData = {
  flags: {
    myModuleName: {
      myFlagName: newFlagValue
    }
  }
}

someDocument.update(updateData);
```


##### Mutation
Simply mutating a flag's value on a document's data will not persist that change in the database, nor broadcast that change to other connected clients.


### Getting a flag's value
There are two places to get a flag value: On the data model itself, or with [`Document#getFlag`](https://foundryvtt.com/api/abstract.Document.html#getFlag)

The arguments passed to `getFlag` have the same constraints as those passed to `setFlag`. Providing an unexpected scope will `throw` rather than return `undefined`. If you need a flag from a module which might not exist in the world, it is safer to access it on the data model itself.

```js
const flagValue = someDocument.getFlag('myModuleName', 'myFlagName');
// flagValue === 'foo'
```

#### On the data model itself
Flags are readable on the Document's data as detailed in the [Data Format](#Data-Format) section:

```
someDocument.data.flags
```

### Unset a flag
A safe way to delete your flag's value is with [`Document#unsetFlag`](https://foundryvtt.com/api/abstract.Document.html#unsetFlag). This will fully delete that key from your module's flags on the provided document.

```js
someDocument.unsetFlag('myModuleName', 'myFlagName');
```

## Specific Use Cases

### Check for a flag during a Hook
Since flags are simply data on a Document, any hook that fires during that [document's event cycle](/en/development/api/document#event-cycles) and most other hooks involving the document will have access to that data.

For example, a `flag` on `ChatMessage` might be injected into the DOM for that message.

```js
chatMessage.setFlag('myModule', 'emoji', '❤️');

Hooks.on('renderChatMessage', (message, html) => {
  if (hasProperty(message, 'data.flags.myModule.emoji')) {
    html.append(`<p>${message.data.flags.myModule.emoji}</p>`);
  }
});
```

### Some Details about `setFlag` and objects

When the value being set is an object, the API doesn't replace the object with the value provided, instead it merges the update in. `Document#setFlag` is a very thin wrapper around `Document#update`. 

The database operation that `update` eventually calls is configured by default to update objects with [mergeObject](https://foundryvtt.com/api/module-helpers.html#.mergeObject)'s default arguments.

Example to demonstrate:
```js
game.user.setFlag('world', 'todos', { foo: 'bar', zip: 'zap' });
// flag value: { foo: 'bar', zip: 'zap' }

game.user.setFlag('world', 'todos', {});
// flag value: { foo: 'bar', zip: 'zap' }
// no change because update was empty

game.user.setFlag('world', 'todos', { zip: 'zop' });
// flag value: { foo: 'bar', zip: 'zop' }
```

`Document#setFlag` should perhaps be thought about as "updateFlag" instead, but that's only partly true because it can set that which doesn't exist yet.

Where this has the most effect is when one wants to store data as an object, and wants to be able to delete individual keys on that object.

The initial instinct of "I'll `setFlag` with an object that has everything but that key which was deleted," does not work. There are some options available, some more intuitive than others.

#### Foundry Specific key deletion syntax 

> This is the recommended way to use `setFlag` to delete a key within a flag object.
{.is-info}

The foundry-specific syntax for deleting keys in `Document#update` (`-=key: null`) works with `setFlag` as well:
```js
game.user.setFlag('world', 'todos', { ['-=foo']: null });
// flag value: { zip: 'zop' }
```
#### Hijacking `Document#unsetFlag`
The `unsetFlag` method is a thin wrapper around `Document#update` which first modifies the key being updated to use the Foundry Specific key deletion syntax (`-=key: null`).

This means `unsetFlag` could be used in a roundabout way to remove a specific key within a flag's object:
```js
game.user.unsetFlag('world', 'todos.foo');
// flag value: { zip: 'zop' }
```

#### Setting to `null`
If you're happy with the key being `null`, setting a key's value to `null` explicitly works as expected:
```js
game.user.setFlag('world', 'todos', { foo: null });
// flag value: { foo: null, zip: 'zop' }
```

---

## Troubleshooting

### Invalid Scope for flag

This error is thrown when the first argument to `Document#setFlag` (or `unsetFlag`/`getFlag`) mentions a package which is not installed. 

Typically this is due to a typo.

This is commonly encountered in use cases where one module checks another module's data and operates based on that data and can be sneaky as a deactivated module does not trigger it.

