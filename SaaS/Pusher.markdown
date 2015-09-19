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

### Publish an event

```typescript
import Pusher from 'pusher';

let pusher = new Pusher({ appId, key, secret });
pusher.trigger(['channel'], 'event', data[, socketId, callback]);
```

### Querying application state

```typescript
pusher.get({ path, params }, callback);
```

#### Application channels

```typescript
pusher.get(
  { path: '/channels' params: {} },
  (error: any, request: any, response: any) => {
    // ...
  });
```

#### Channel information

```typescript
pusher.get(
  { path: `/channels/#{channelName}` params: {} },
  callback
  );
```

#### Presence users

```typescript
pusher.get(
  { path: `/channels/#{channelName}/users` params: {} },
  callback
  );
```

### WebHooks

#### Format

```json
{
  "time_ms": 1327078148132,
  "events": [
    { "name": "eventName", ... }
  ]
}
```

#### Events

```json
{ "name": "channel_occupied", "channel": "test_channel" }
```

```json
{ "name": "channel_vacated", "channel": "test_channel" }
```

```json
{
  "name": "member_added",
  "channel": "presence-your_channel_name",
  "user_id": "a_user_id"
}
```

```json
{
  "name": "member_removed",
  "channel": "presence-your_channel_name",
  "user_id": "a_user_id"
}
```

```json
{
  "name": "client_event",
  "channel": "name of the channel the event was published on",
  "event": "name of the event",
  "data": "data associated with the event",
  "socket_id": "socket_id of the sending socket",
  "user_id": "user_id associated with the sending socket" # Only for presence channels
}
```

## Authenticating users

### Implementing the auth endpoint for a private channel

```typescript
app.post('/pusher/auth', (req: Request, res: Response) => {
  let { socket_id, channel_name } = req.body;
  let auth = pusher.authenticate(socket_id, channel_name);
  res.send(auth);
});
```

### Implementing the auth endpoint for a presence channel

```typescript
app.post('/pusher/auth', (req: Request, res: Response) => {
  let { socket_id, channel_name } = req.body;
  var presenceData = {
    user_id: 'unique_user_id',
    user_info: {
      name: 'Mr Pusher',
      twitter_id: '@pusher'
    }
  };
  let auth = pusher.authenticate(socket_id, channel_name, presenceData);
  res.send(auth);
});
```

## Excluding recipients

```typescript
// client side
let socketId = null;
pusher.connection.bind('connected', () => {
  socketId = pusher.connection.socket_id;
  $.post('/trigger_event', { data, socketId });
});

// server side
app.post('/trigger_event', (req: Request, res: Response) => {
  let { socketId } = req.body;
  pusher.trigger('my-channel', 'my-event', data, socketId);
});
```



