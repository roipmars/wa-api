## Websocket

### 1. Websocket connection initialization

The process begins with the creation of a new instance of the `Websocket` object.The connection URL includes the path to the Websocket server, an event parameter that specifies the type of event to which the customer is enrolling and an authentication token.This token will be used by the server to see if the customer is allowed to connect and receive these events.

```javascript
const ws = new WebSocket(`${url}?event=${encodeURIComponent(eventName)}&token=${encodeURIComponent(instanceToken)}`);
```

The URL is built to include:

- **`event`**: A parameter that specifies the type of event of interest, making it easier for the server to route or filter messages according to the desired data or command type.
- **`token`**: A security parameter that is probably used by the server to authenticate and authorize the connection.

### 2. Event manipulators

**`onopen`**:
- This event is fired when the Websocket connection is successfully established.
- In the `onopen` manipulator, a test message is sent immediately to the server.This can be used to confirm that the communication route is working or to inform the server about the desired initial state of the customer.

```javascript
ws.onopen = () => {
  console.log('Connected to the server');
  ws.send(JSON.stringify({ message: "test data" }));
};
```

**`onmessage`**:
- This event is triggered whenever a message is received from the server.
- The messages received are treated by converting the contents of `data` from a string JSON to a JavaScript object, which is then passed to the `callback` function provided by the script user.

```javascript
ws.onmessage = (event) => {
  if (callback) {
    const data = JSON.parse(event.data)
    callback(data, event)
  }
};
```

**`onerror`**:
- Triggered when an error occurs in the Websocket connection.
- It can be used to login or treat network errors, transmission failures, etc.

```javascript
ws.onerror = (error) => {
  console.log('Error:', error);
};
```

**`onclose`**:
- This event is triggered when the Websocket connection is closed, either at customer, server, or network failures.
- The `Onclose` event manipulator automatically tries to reconnect to the server after a defined range.Important to correct here, the `Settimeout` should call` SOCKET (EventName, Callback) `instead of` SOCKET (Event) `, to ensure the reconnection is done correctly.

```javascript
ws.onclose = (event) => {
  console.log(`Connection closed with code ${event.code} and reason ${event.reason}, attempting to reconnect...`);
  setTimeout(() => socket(eventName, callback), reconnectInterval);
};
```

### 3. Automatic reconnection

- After the connection is closed, the customer tries to reconnect using a defined time interval (`reconnectinterval`).This behavior ensures that the customer tries to maintain a persistent connection even in the face of server network or reinforcement problems.

### 4. Full example

```javascript
const url = "ws://localhost:8084/ws/events";
const token = ""
const reconnectInterval = 5000;

function socket(eventName, callback) {
  const ws = new WebSocket(`${url}?event=${encodeURIComponent(eventName)}&token=${encodeURIComponent(token)}`);

  ws.onopen = () => {
    console.log("Connected to the server");
  };

  ws.onmessage = (event) => {
    if (callback) {
      const data = JSON.parse(event.data)
      callback(data, event)
    }
  };

  ws.onerror = (error) => {
    console.log('Error:', error);
  };

  ws.onclose = (event) => {
    console.log(`Connection closed with code ${event.code} and reason ${event.reason}, attempting to reconnect...`);
    setTimeout(() => socket(event), reconnectInterval);
  };
}

// An instance of the function will be created for each event
// The events are the same fired by the webhok
// Except the event "Messaging-History.Set"

socket("connection.update", (msg, event) => {
    console.log(msg)
})

socket("messages.upsert", (msg, event) => {
    console.log(msg)
})

socket("qrcode.updated", (msg, event) => {
    console.log(msg)
})
```