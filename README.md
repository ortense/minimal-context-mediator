![Mediator banner - the mediator mascot generated by dall-e 2](https://raw.githubusercontent.com/ortense/mediator/main/mediator.jpg)

# @ortense/mediator

A minimalistic and dependency-free event mediator with internal context for front-end.
Written typescript for a good development experience and really light, just 300 bytes in your bundle!

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
type MyEvents = 'value:change' | 'active:toggle' | 'item:added' | 'item:removed'

export const myMediator = createMediator<MyContext, MyEvents>(initialContext)
```

This is a good practice to help developers who will interact with the mediator, providing predictability of the events that can be listened or send.

### Listening to events

To listen to events use the `.on` method

```typescript
import { myMediator, MyContext } from './my-mediator'

function myEventListener(ctx: Readonly<MyContext>) {
  // do what you want
}

myMediator.on('event-name', myEventListener)
```

If you prefer you could use the type `MediatorEventListener`

```typescript
import { MediatorEventListener } from '@ortense/mediator'
import { myMediator, MyContext } from './my-mediator'

const myEventListener: MediatorEventListener = (ctx) => {
  // do what you want
}

myMediator.on('event-name', myEventListener)
```

To stop use the `.off` method

```typescript
myMediator.off('event-name', myEventListener)
```

### Send events

To send events use the `.send` method.

```typescript
import { myMediator} from './my-mediator'

myMediator.send('hello-world')
```

All listener functions for the `hello-world` event will be called in the order they were added to the mediator.

The `.send` method could receive a function to modifiy the context:

```typescript
import { myMediator, MyContext } from './my-mediator'

function toggleActive(ctx: Readonly<MyContext>) {
  return {
    ...ctx,
    active: !ctx.active
  }
}

myMediator.send('toggle', toggleActive)
```

If you prefer you could use the `MediatorContextModifier` type.

```typescript
import { MediatorContextModifier } from '@ortense/mediator'
import { myMediator, MyContext } from './my-mediator'

const toggleActive: MediatorContextModifier<MyContext> = (ctx) => ({
  ...ctx,
  active: !ctx.active
})

myMediator.send('toggle', toggleActive)
```

Or an inline declaration:

```typescript
import { myMediator } from './my-mediator'

myMediator.send('toggle', (ctx) => ({ ...ctx, active: !ctx.active }))
```

### Get current context

Use the method `.getContext` to get a readonly version of the current context.

```typescript
import { myMediator } from './my-mediator'

const ctx = myMediator.getContext() //? Readonly<MyContext>
```
