# Minimal Context Mediator

A minimal implementation for an event broker with internal context without any production dependencies.

## how to use

### Setup

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
Now create an object to be your initial context

```typescript
const initialContext: MyContext = {
  value: 'hello world',
  active: true,
  nested: {
    items: [],
  },
}
```

Then create the mediator object

```typescript
export const myMediator = createMediatorContext(initialContext)
```

The complete setup file should look like this

```typescript
import { MediatorContext, createMediator } from 'context-mediator'

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

The context mediator use simple strings to identify events, think of it as a unique identifier and is used to send or listen events.

### Listening to events

To listen to events use the `.on` method

```typescript
import { myMediator, MyContext } from './my-mediator'

function myEventListener(ctx: Readonly<MyContext>) {
  // do what you want
}

myMediator.on('event-name', myEventListener)
```

If you prefer could use the type `MediatorEventListener`

```typescript
import { MediatorEventListener } from 'context-mediator'
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

To send events use the `.send`

```typescript
import { myMediator} from './my-mediator'

myMediator.send('event-name')
```

The `.send` method could receive a function to modifiy the context

```typescript
import { myMediator, MyContext } from './my-mediator'

function toggleActive(ctx: Readonly<MyContext>) {
  return {
    ...ctx,
    active: !ctx.active
  }
}

myMediator.send('event-name', toggleActive)
```

If you prefer could use the `MediatorContextReducer` type

```typescript
import { MediatorContextReducer } from 'context-mediator'
import { myMediator, MyContext } from './my-mediator'

const toggleActive: MediatorContextReducer<MyContext> = (ctx) => ({
  ...ctx,
  active: !ctx.active
})

myMediator.send('event-name', toggleActive)
```

### Get current context

Use the method `.getContext` to get a readonly version of the current context.

```typescript
import { myMediator } from './my-mediator'

const ctx = myMediator.getContext() //? Readonly<MyContext>
```