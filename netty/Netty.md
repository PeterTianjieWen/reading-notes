# Netty 

## Concepts

* Java OIO (Old input and output): `java.net` blocking functions from nactive system socket libraries
* Java NIO (New input and output): `java.nio` non blocking native libraries

### Selectors

* Use event notification API to indicate which nonblocking `sockets` are ready for I/O

### Core Components

* `Channel` represents an open connection to an entity

* `Callbacks` method reference provided to another method (to call back).  Callback triggered -> `<ChannelHandler>`

* `Future`: notify an application when an operation has completed. Placeholder for the result of async op. `ChannelFuture`

  ChannelFutureListener ---register--> ChannelFuture

  Notes: Callbacks and Futures are complementary mechanisms. Future is more elaborate than callbacks.

* `ChannelHandler`: pipeline of events and handlers(action for Inbound/Outbound)

![ChannelHandlerPipeline](ChannelHandlerPipeline.png)

### ELECTORS, EVENTS, AND EVENT LOOPS

`Selector` is abstracted away (from Channel, Future, Callbacks, ChannelHandler) to fire different events. (event management)

![Selector](/Users/peter/Desktop/reading notes/netty/Selector.png)

An `EventLoop` is assigned to each Channel to handle all of the events

* Registration of interesting events

* Dispatching events to ChannelHandlers

* Scheduling further actions

## Netty components and design

### `Channel`-`Sockets`
* Basic I/O operations (`bind()`, `connect()`, `read()`, and `write()`)
* Provides APIs that reduce the complexity of working directly with `Sockets`

Types of channel:    
* `EmbeddedChannel`
* `LocalServerChannel`
* `NioDatagramChannel`
* `NioSctpChannel`
* `NioSocketChannel`

### `Eventloop`
pg.34 diagram
Relationship between `Channels` `EventLoops` `Threads` `EventLoopGroups`
* `EventLoopGroups` contains `EventLoops`
* An `EventLoop` is bound to a single `Thread` for its lifetime
* All I/O events processed by an `EventLoop` are handled on its dedicated `Thread`.
* **A Channel is registered for its lifetime with a single EventLoop.**
* A single `EventLoop` may be assigned to one or more `Channels`.

### `ChannelFuture`
* `ChannelFuture.addListener()` registers `ChannelFutrueListener`

### `ChannelHandler` && `ChannelPipeline`
* `ChannelHandler` serves as the container for all application logic that applies to handling inbound and outbound data
* `ChannelPipeline` a container for a chain of ChannelHandlers and defines
an API for propagating the flow of inbound and outbound events along the chain
* Types of channel handler adapters
    * `ChannelHandlerAdapter`
    * `ChannelInboundHandlerAdapter`
    * `ChannelOutboundHandlerAdapter`
    * `ChannelDuplexHandlerAdapter`

* Encoders decoders

### Bootstrapping
* ServerBootstrap
    * The EventLoopGroup associated with the ServerChannel assigns an EventLoop
that is responsible for creating Channels for incoming connection requests. Once a connection has been accepted, the second EventLoopGroup assigns an EventLoop to its Channel.
* (Client)Bootstrap
