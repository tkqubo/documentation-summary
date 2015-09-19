# Pusher Documentation

### Enable loggin in the Pusher JavaScript Library

```javascript
Pusher.log = message => {
  if (window.console && window.console.log) {
    window.console.log(message);
  }
}
```

## Channels

### Presence Channels

```typescript
channel
  .bind('pusher:subscription_succeeded', (members: Members) => {
    // ...
  })
  .bind('pusher:member_added', (member: UserInfo<any>) => {
    // ...
  })
  .bind('pusher:member_removed', (member: UserInfo<any>) => {
    // ...
  })
;
```

## Pusher Events

```typescript
pusher.bind(...);
channel.bind(...);
```

### Pusher channel events

```typescript
channel
  .bind('pusher:subscription_succeeded', () => {
    // ...
  })
  .bind('pusher:subscription_error', (status: number) => {
    // ...
  })
;
```

### Triggering Client Events

*You cannot trigger client events from the debug console*

```typescript
let isTriggered = channel.trigger('client-xxxxxx', data);
```

# Server REST API



