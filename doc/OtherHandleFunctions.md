# Other Handle Functions

The `handle` provides additional functions which perform common but complicated tasks.

#### handle.openAMQPCommunication(username, password, heartbeat, vhost, callback(error))

- `username` The username used to log into AMQPLAIN. Defaults to `'guest'`.
- `password` The password used to log into AMQPLAIN. Defaults to `'guest'`.
- `heartbeat` A boolean which controls if heartbeats are enabled.  If set to true, heartbeats are sent at the time suggested by the server. Defaults to `true`.
- `vhost` The vhost used to open the connection. Defaults to `'/'`.
- `callback(error)` Called once the content has been written to the socket.

`openAMQPCommunication` performs the following tasks:

- open the amqp connection using AMQPLAIN
- tune the connection and optionally enable heartbeats
- open the vhost provided
- open channel 1
- channel 1 will re-open if closed by the server
- the socket will be paused and resumed as requested by channel 1

#### handle.closeAMQPCommunication(callback(error))
  
- `callback(error)` Called once the content has been written to the socket.

`closeAMQPCommunication` performs the following tasks:

- close channel 1
- close the amqp connection
- stop the heartbeats

#### handle.setFrameMax(frameMax)
  
- `frameMax` The new largest frame that should be used;

`setFrameMax` updates the size of the buffers used for AMQP communication.  Should be called after receiving `connection.tune` method.


#### handle.validateProps(serverProps)

- `serverProps` The AMQP `server-properties` sent from the server during the `connection.start` method.

This method is not meant to be called by the client but instead should be overridden by the client if needed.  There are numerous AMQP servers in operation with varying uses of the `server-properties` and `client-properties` fields within the `connection.start` and `connection.start-ok` methods, respectively.  For example, Apache QPid's [extensions](https://cwiki.apache.org/confluence/display/qpid/Qpid+extensions+to+AMQP) differ from those of RabbitMQ's [extensions](http://www.rabbitmq.com/consumer-cancel.html#capabilities).

`validateProps` is called by `openAMQPCommunication` and by default it inspects the server's properties and verifies that the server meets RabbitMQ's default [capabilities](http://www.rabbitmq.com/consumer-cancel.html#capabilities) and provides the corresponding client capabilities back to the server.

`validateProps` should be overridden by the client if it needs to negotiate the client-server properties by inspecting the supplied `serverProps` and providing the custom client properties as the return value.  If the client wishes to abort the connection/channel setup due to insufficient server capabilities this function should return an object with an `error` property containing the appropriate error description.  This will cause `openAMQPCommunication` to stop processing the connection and call back with the error value set.

