![Mediator banner - the mediator mascot generated by dall-e 2](https://raw.githubusercontent.com/ortense/mediator/main/media/mediator.jpg)

# @ortense/mediator

[![install size](https://packagephobia.com/badge?p=@ortense/mediator)](https://packagephobia.com/result?p=@ortense/mediator) [![Coverage Status](https://coveralls.io/repos/github/ortense/mediator/badge.svg?branch=github-actions)](https://coveralls.io/github/ortense/mediator?branch=github-actions)

A minimalistic and dependency-free event mediator with internal context for front-end.
Written typescript for a good development experience and really light, just 300 bytes in your bundle!

Access the complete documentation at [ortense.github.io/mediator/](https://ortense.github.io/mediator/)

## Use case

You want to simplify communication between independent components in your web app. The mediator can be used to facilitate the exchange of data and events between different parts of the application without crate a strong coupling, keeping the separation of concerns between the components of your app or external integrations like third party scripts or extensions.

![Mediator flow chart - made in excalidraw.com](https://raw.githubusercontent.com/ortense/mediator/main/media/flow.png)

## Install

Pick your favorite package manager.

```sh
npm install @ortense/mediator  # npm
yarn add  @ortense/mediator    # yarn
pnpm add @ortense/mediator     # pnpm
bun add @ortense/mediator      # bun
```

## Usage

First, define an interface that extends `MediatorContext` to represent your context, this interface must be an object with properties serializable to JSON.

```typescript
export interface MyContext extends MediatorContext {
  value: string
  active: boolean
  nested: {
    items: number[]
  }
}
```
Now create an object to be your initial context.

```typescript
const initialContext: MyContext = {
  value: 'hello world',
  active: true,
  nested: {
    items: [],
  },
}
```

Then create the mediator object:

```typescript
export const myMediator = createMediator(initialContext)
```

The complete setup file should look like this:

```typescript
import { MediatorContext, createMediator } from '@ortense/mediator'

export interface MyContext extends MediatorContext {
  value: string
  active: boolean
  nested: {
    items: number[]
  }
}

const initialContext: MyContext = {
  value: 'hello world',
  active: true,
  nested: {
    items: [],
  },
}

export const myMediator = createMediator(initialContext)
```

### Events

The mediator use simple strings to identify events, think of it as a unique identifier to be used to send or listen to events.

Optionally, you can define a type that extends from `string` to represent the events that your mediator has.

```typescript
type MyEvents = 'loaded' | 'value:change' | 'item:added' | 'item:removed'

export const myMediator = createMediator<MyContext, MyEvents>(initialContext)
```

This is a good practice to help developers who will interact with the mediator, providing predictability of the events that can be listened or send.

### Listening to events

To listen to events use the `.on` method

```typescript
import { myMediator, MyContext } from './my-mediator'

function myEventListener(ctx: Readonly<MyContext>, event: MyEvents) {
  // do what you want
}

myMediator.on('loaded', myEventListener)
```

If you prefer you could use the type `MediatorEventListener`

```typescript
import { MediatorEventListener } from '@ortense/mediator'
import { myMediator, MyContext, MyEvents } from './my-mediator'

const myEventListener: MediatorEventListener<MyContext, MyEvents> = (ctx, event) => {
  // do what you want
}

myMediator.on('loaded', myEventListener)
```

You also use the wildcard `*` to listen all events.

```typescript
myMediator.on('*', (ctx, event) => console.log(ctx, event))
```

Wildcard listeners could be useful for debugging, for example logging whenever an event is triggered.

```typescript
myMediator.on('*', (ctx, event) => {
  console.log(`Event ${event} change the context to`, ctx)
})
```

To stop use the `.off` method

```typescript
myMediator.off('loaded', myEventListener)
```

### Send events

To send events use the `.send` method.

```typescript
import { myMediator} from './my-mediator'

myMediator.send('loaded')
```

All listener functions for the `loaded` event will be called in the order they were added to the mediator.

The `.send` method could receive a function to modifiy the context:

```typescript
import { myMediator, MyContext } from './my-mediator'

function changeValue(ctx: Readonly<MyContext>) {
  return {
    ...ctx,
    value: 'new value'
  }
}

myMediator.send('value:change', changeValue)
```

If you prefer you could use the `MediatorContextModifier` type.

```typescript
import { MediatorContextModifier } from '@ortense/mediator'
import { myMediator, MyContext } from './my-mediator'

const changeValue: MediatorContextModifier<MyContext> = (ctx) => ({
  ...ctx,
  value: 'new value'
})

myMediator.send('value:change', changeValue)
```

Or an inline declaration:

```typescript
import { myMediator } from './my-mediator'

myMediator.send('value:change', (ctx) => ({ ...ctx, active: 'new value }))
```

### Get current context

Use the method `.getContext` to get a readonly version of the current context.

```typescript
import { myMediator } from './my-mediator'

const ctx = myMediator.getContext() //? Readonly<MyContext>
```
