# PubSubES: A tiny ES6-based pub-sub library

[![npm version](https://img.shields.io/npm/v/pub-sub-es.svg)](https://www.npmjs.com/package/pub-sub-es)
[![stability experimental](https://img.shields.io/badge/stability-stable-green.svg)](https://nodejs.org/api/documentation.html#documentation_stability_index)
[![build status](https://img.shields.io/github/actions/workflow/status/flekschas/pub-sub/build.yml?branch=master&color=a17fff)](https://github.com/flekschas/pub-sub/actions?query=workflow%3Abuild)
[![gzipped size](https://img.shields.io/bundlephobia/minzip/pub-sub-es?color=6ae3c7&label=gzipped%20size)](https://unpkg.com/pub-sub-es)
[![code style prettier](https://img.shields.io/badge/code_style-prettier-ff69b4.svg)](https://github.com/prettier/prettier)
[![demo](https://img.shields.io/badge/demo-online-c15de2.svg)](http://pub-sub.lekschas.de)

> A [tiny <1KB](https://bundlephobia.com/result?p=pub-sub-es) [pub-sub](https://en.wikipedia.org/wiki/Publish%E2%80%93subscribe_pattern)-based event library that lets you send custom message within and cross the window! It's written in ES6 and makes use of the awesome [Broadcast Channel API](https://developer.mozilla.org/en-US/docs/Web/API/Broadcast_Channel_API).

**Demo:** [pub-sub.lekschas.de](http://pub-sub.lekschas.de)

## Install

```
npm -i -D pub-sub-es
```

## Get Started

```javascript
import createPubSub from 'pub-sub-es';

// Creates a new pub-sub event listener stack.
const pubSub = createPubSub();

// Subscribe to an event
const myEmojiEventHandler = (msg) => console.log(`🎉 ${msg} 😍`);
pubSub.subscribe('my-emoji-event', myEmojiEventHandler);

// Publish an event
pubSub.publish('my-emoji-event', 'Yuuhuu'); // >> 🎉 Yuuhuu 😍

// Unsubscribe
pubSub.unsubscribe('my-emoji-event', myEmojiEventHandler);
```

### Type Events

By default, `pub-sub` assumes you are publishing any kind of events with an
unknown payload. With TypeScript you can specify which events (name and payload)
you are publishing as follows.

```typescript
import createPubSub, { Event } from 'pub-sub-es';

type Events = Event<'init' | 'destroy', undefined> &
  Event<'solution', number> &
  Event<'selection', number[]>;

const pubSub = createPubSub<Events>();
```

With this, we established that `pubSub` will handle four events:

- `init` and `destroy` feature no payload (i.e., the payload is `undefined`)
- `solution` features a single `number` as payload
- `selection` features an array of numbers as payload

When you listen to or publish an event, the payload will now be typed based of the event. I.e.:

```typescript
pubSub.subscribe('destroy', (payload) => payload === undefined); // => ✅
pubSub.subscribe('destroy', (payload) => payload * 2); // => ❌
pubSub.subscribe('solution', (payload) => console.log(payload + 2)); // => ✅
pubSub.subscribe('solution', (payload) => console.log(payload.length)); // => ❌
pubSub.subscribe('selection', (payload) => console.log(payload.length)); // => ✅
pubSub.subscribe('selection', (payload) => console.log(payload * 2)); // => ❌

pubSub.publish('init'); // => ✅
pubSub.publish('init', 'w00t'); // => ❌
pubSub.publish('solution', 42); // => ✅
pubSub.publish('solution', '42'); // => ❌
pubSub.publish('selection', [1,3,3,7]); // => ✅
pubSub.publish('selection', 1337); // => ❌
```

## API

`pub-sub-es` exports the factory function (`createPubSub`) for creating a new pub-sub service by default and a global pub-sub service (`globalPubSub`). The API is the same for both.

#### `createPubSub(options = { async: false, caseInsensitive: false, stack: {})`

> Creates a new pub-sub instances

**Arguments:**

- `options` is an object for customizing pubSub. The following properties are understood:
  - `async`: when `true`, events are published asynchronously to decouple the process of the originator and consumer.
  - `caseInsensitive`: when `true`, event names are case insensitive
  - `stack`: is an object holding the event listeners and defaults to a new stack when being omitted.

**Returns:** a new pub-sub service

#### `pubSub.publish(event, news, options = { async: false, isNoGlobalBroadcast: false})`

> Publishes or broadcasts an event with some news or payload

**Arguments:**

- `event` is the name of the event to be published.
- `news` is an arbitrary value that is being broadcasted.
- `options` is an object for customizing the behavior. The following properties are understood:
  - `async`: overrides the global `async` flag for a single call.
  - `isNoGlobalBroadcast`: overrides the global `isGlobal` flag for a single call.

#### `pubSub.subscribe(event, handler, times)`

> Subscribes a handler to a specific event

- `event` is the name of the event to be published.
- `handler` is the handler function that is being called together with the broadcasted value.
- `times` is the number of times the handler is invoked before it's automatically unsubscribed.

**Returns:** an object of form `{ event, handler }`. This object can be used to automatically unsubscribe, e.g., `pubSub.unsubscribe(pubSub.subscribe('my-event', myHandler))`.

#### `pubSub.unsubscribe(eventOrSubscription, handler?)`

> Unsubscribes a handler from listening to an event

- `eventOrSubscription` is the name of the event to be published. Optionally, `unsubscribe` accepts an object of form `{ event, handler}` coming from `subscribe`.
- `handler` is the event handler function to be unsubscribed from the event

#### `pubSub.clear()`

> Removes all currently active event listeners and unsets the event times.

_Note: this endpoint is not available on the global pub-sub because it could have undesired side effects. You need to unsubscribe global event listeners manually instead._

### Between-context messaging

The global pub-sub instance is created when `pub-sub-es.js` is loaded and provides a way for
global messaging within and between contexts. It utilizes the [Broadcast Channel API](https://developer.mozilla.org/en-US/docs/Web/API/BroadcastChannel). Note, when sending sending objects from one context to another you have to make sure that they are cloneable. For example, trying to broadcast a reference to a DOM element will fail.

## Difference to other pub-sub libraries

You might have come across the awesome [PubSubJS](https://github.com/mroderick/PubSubJS) library and wonder which one to choose. First of all, both are tiny, have no dependencies, and offer async event broadcasting. The two main features that PubSubES offers are:

1. Auto-unsubscribable event listening: via `pubSub.subscribe(eventName, eventHandler, times)` you can make the `eventHandler` unsubscribed from `eventName` automatically after `eventName` was broadcasted `times` times.

2. Between-context messaging: PubSubES supports the new Broadcast Channel API so you can send events between different browser contexts, e.g., tabs.
