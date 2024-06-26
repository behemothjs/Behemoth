# Behemoth

![stability](https://img.shields.io/badge/stability-Alpha-F00)
![node 18.x](https://img.shields.io/badge/node-18.x-0B0)
[![XO code style](https://shields.io/badge/code_style-5ed9c7?logo=xo&labelColor=gray)](https://github.com/xojs/xo)

An engine for building systems necessary for web production.

## Document Translation

[🇯🇵 日本語](./README_ja.md)

## 🚫 Project stability is "Alpha"

This project is currently under development. Please refrain from using it until release as it may undergo specification changes.

## Install

```bash
npm install @behemothjs/behemoth
```

## Features

### 🍄 App

The main object.

#### Global Configuration for All Modules

```javascript
import {behemoth as app} from '@behemothjs/behemoth';

app.configure({
 schema: SchemaConfig,
 logger: LoggerConfig,
});
```

#### (alias) Logger

```javascript
app.log(message);
app.warn(message);
app.error(message);
```

#### (alias) Observer

```javascript
// Notify
app.notify(channel, topic, payload);

// Subscribe
const subscription = app.listen(channel, topic, (event) => {
 app.log(event);
});

// Unsubscribe
subscription.remove();
```

### 🍄 Schema

A schema processing tool useful for creating model classes, etc.

#### Global Configuration for Schema Class

```javascript
import {Schema} from '@behemothjs/behemoth';

Schema.configure({
  // [defaults]
  allowAdditionalKeys: false,
  allowUndefinedKeys: false,
  idStrategy: () => crypto.randomUUID(),
  timestampStrategy: () => new Date().toISOString(),
})
```

#### Example

```javascript
import {schema} from '@behemothjs/behemoth';

// Temporary Configuration
schema.configure({
  // [defaults]
  allowAdditionalKeys: false,
  allowUndefinedKeys: false,
  idStrategy: () => crypto.randomUUID(),
  timestampStrategy: () => new Date().toISOString(),
})

class SampleSchema {
  id;
  name;
  description;

  /**
   * @param {SampleSchema} data
   */
  constructor(data) {
    // Before: Input undefined key -> Throw Error
    // After:  not set keys       -> Set undefined to null
    schema.assign(this, data);

    // If `id` is empty, set the UUID.
    schema.autoId(this, 'id');

    // If `createdAt/updatedAt` is empty, set the DateTime.
    schema.autoTimestamp(this, 'createdAt');
    schema.autoTimestamp(this, 'updatedAt');
  }
}

const sampleSchema = new SampleSchema({
  name: 'Behemoth',
  description: 'This is web tool kit.'
})

console.log(sampleSchema)
```

```javascript
SampleSchema {
  id: '8481d7e4-478c-4233-b4a1-7ea6d7d63749',
  name: 'Behemoth',
  description: 'This is web tool kit.',
  createdAt: '2024-03-21T07:15:45.092Z',
  updatedAt: '2024-03-21T07:15:45.092Z',
}
```

### 🍄 Observer

PubSub Module

```javascript
import {observer} from '@behemothjs/behemoth';

// Notify
observer.notify('channelName', 'topicName', payload);

// Subscribe
const subscription = observer.listen('channelName', 'topicName', (event) => {
  console.log(event);
});

// Unsubscribe
subscription.remove();
```

### 🍄 Log

Logging Tool

```javascript
import {log} from '@behemothjs/behemoth';

// Configuration or Set environment variable: LOG_LEVEL
log.configure({logLevel: 'WARN'}); // LOG / INFO / WARN / ERROR / SILENT

log.log('LOG');     // console.log('LOG')
log.info('INFO');   // console.info('INFO')
log.warn('WARN');   // console.warn('WARN')
log.error('ERROR'); // console.error('ERROR')
```

Since this log is output via Observer, it is effective for streaming log collection processes.

```javascript
const channel = 'Log';
const topic = 'ERROR'; // <-- "*" (wildcard) can be used.

observer.listen(channel, topic, async event => {
  const {payload} = event;
  await otherStreamingApi.push(payload);
});
