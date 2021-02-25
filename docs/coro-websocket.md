# Documentations

Unofficial docs for the module [coro-websocket](https://github.com/luvit/lit/blob/master/deps/coro-websocket.lua) version 3.1.0.

[coro-websocket](https://github.com/luvit/lit/blob/master/deps/coro-websocket.lua) is a library that implements the WebSockt ws(s) protocol and provides synchronous manipulations over it.

----

## Functions

----

### parseUrl(url) {#parseUrl}

Parses a WebSocket URL into a Lua table.

*This method does not require running in a coroutine*

#### Parameters {#parseUrl-parameters}

| Param | Type   | Description |
|:-----:|:------:|:------------|
| url   | string | The WebSocket URL to be parsed, must be a valid WebSocket URL. |

#### Returns {#parseUrl-returns}

**On Success:**

| Name  | Type   | Description |
|:------|:------:|:------------|
| result| table  | The parsed URL as a table structure, see [WS URL](#ws-url) for details. |

**On Failure:**

| Name | Type   | Description |
|:-----|:------:|:------------|
| fail | nil    | If the first return was `nil`, that'd mean the operation has failed. |
| error| string | An error message explaining what went wrong. |

----

####

### wrapIo(rawRead, rawWrite, options) {#wrapIo}

Wraps [coro-channel](https://bilal2453.github.io/coro-docs/docs/coro-channel.html) wrappers to be WS compatible. This assumes the raw wrapper can understand WebSocket protocol, meaning, it uses [WebSocket-Codec](https://github.com/luvit/lit/blob/master/deps/websocket-codec.lua) encoders&decoders adapters.

*This method does not require running in a coroutine*

#### Parameters {#wrapIo-parameters}

| Param | Type   | Description |
|:-----:|:------:|:------------|
| rawRead | function | Supposedly [coro-channel reader](https://bilal2453.github.io/coro-docs/docs/coro-channel.html#reader), or an [adapter](https://bilal2453.github.io/coro-docs/docs/coro-wrapper.html) of it that can correctly decode WebSocket frames. |
| rawWrite | function | Supposedly [coro-channel writer](https://bilal2453.github.io/coro-docs/docs/coro-channel.html#writer), or an [adapter](https://bilal2453.github.io/coro-docs/docs/coro-wrapper.html#encoder) of it that can correctly encode WebSocket frames. |
| options | table | See [Options](#options) for details. |

#### Returns {#wrapIo-returns}

| Name  | Type   | Description |
|:------|:------:|:------------|
| read  | function | The wrapped reader, works similarly to [coro-channel reader](https://bilal2453.github.io/coro-docs/docs/coro-channel.html#reader). Except that it smartly responds to opcodes 8 & 9 messages making use out of [WebSocket-Codec](https://github.com/luvit/lit/blob/master/deps/websocket-codec.lua) decoder. |
| write | function | the wrapped writer, works similarly to [coro-channel writer](https://bilal2453.github.io/coro-docs/docs/coro-channel.html#writer). Except that it handles opcodes and masking making use out of [WebSocket-Codec](https://github.com/luvit/lit/blob/master/deps/websocket-codec.lua) encoder. |

----

### connect(options) {#connect}

Establishes a WebSocket connection with the said host.

***This method MUST be run in a coroutine***

#### Parameters {#connect-parameters}

| Param | Type   | Description |
|:-----:|:------:|:------------|
| options | table | See [Options](#options) for details. |

#### Returns {#connect-returns}

| Name  | Type   | Description |
|:------|:------:|:------------|
| socket| [uv_tcp_t](https://github.com/luvit/luv/blob/master/docs.md#uv_tcp_t--tcp-handle)/[uv_pipe_t](https://github.com/luvit/luv/blob/master/docs.md#uv_pipe_t--pipe-handle) | The socket handle the WebSocket connection was bound to. |
| read  | function | See [read](#read) for details. |
| write | function | See [write](#write) for details. |

----

## Returned Functions

----

### write(message) {#write}

Sends (writes) a WebSocket [Message](#message) into the host stream while yielding the current coroutine, and resumes when done writing.

***This method MUST be run in a coroutine***

#### Parameters {#write-parameters}

| Param | Type   | Description |
|:-----:|:------:|:------------|
| msg   | [Message](#message) | The WS(S) message to be written into the host stream. See [Message](#message) for more details. |

#### Returns {#write-returns}

| Name  | Type   | Description |
|:------|:------:|:------------|
| success | boolean | Whether or not the operation has succeeded. |
| error | string | A string containing the error code caused this failure. |

----

### read() {#read}

Yields the current coroutine till the stream to receive a WebSocket message, then resumes with the received message structure.

***This method MUST be run in a coroutine***  

#### Returns {#read-returns}

**On Success:**

| Name  | Type   | Description |
|:------|:------:|:------------|
| msg   | [Message](#message) | A received WebSocket message from the host. |

**On Failure:**

| Name  | Type   | Description |
|:------|:------:|:------------|
| success | boolean | A `false` value indicting the operation has failed. |
| error | string | A string containing the error code caused this failure. |

----

## Structures

### Message

String-indexed table that represents a WebSocket message, whether it's sent or received.

#### Fields {#message-fields}

| Field | Type   | Description |
|:-----:|:------:|:------------|
| fin   | boolean| Whether or not this is the last fragment in the message. See [RFC6455-Section 5.2](https://tools.ietf.org/html/rfc6455#section-5.2) for details. |
| rsv1  | boolean| The first reserved field, its meaning's defined by the host. See [RFC6455-Section 5.2](https://tools.ietf.org/html/rfc6455#section-5.2) for details. |
| rsv2  | boolean| The second reserved field, its meaning's defined by the host. See [RFC6455-Section 5.2](https://tools.ietf.org/html/rfc6455#section-5.2) for details. |
| rsv3  | boolean| The third reserved field, its meaning's defined by the host. See [RFC6455-Section 5.2](https://tools.ietf.org/html/rfc6455#section-5.2) for details. |
| opcode| number | Defines how the received payload should be interpreted. Those are defined by [RFC6455 Page-29](https://tools.ietf.org/html/rfc6455#page-29) and are handled by coro-websocket. |
| mask  | boolean| Whether or not the frame is bit masked. See [RFC6455-Section 5.1](https://tools.ietf.org/html/rfc6455#section-5.1) for more details.
| len   | number | The length(size) of the message payload in bytes. |
| payload | string | The actual contents of the message. |

----

### Options {#options}

A string-indexed table that provides further configurations over the ws connection and over how coro-websocket behave.

If an option field name matches one of coro-net [Options](https://bilal2453.github.io/coro-docs/docs/coro-net.html#options) name then same rules applies. All fields are only available to [connect](#connect), unless stated otherwise.

#### Fields {#options-fields}

| Field | Type   | Description |
|:-----:|:------:|:------------|
| path  | string | The pipe name the stream should refer to, only relevant if `host` and `port` aren't provided. |
| host  | string | The hostname a TCP connection should be referring to. If `port` is provided but `host` isn't, the default value's used. |
| port  | number | The port of the TCP host the connection refers to. |
| tls   | boolean/table | Whether or not to use secure-layer on top of the connection. A table value for using secure-layer with further configurations, see [coro-net TLS Options](https://bilal2453.github.io/coro-docs/docs/coro-net.html#tls-options) for more details. |
| pathname | string | The path at which to connect to the host. That's, everything after the first `/`. |
| subprotocol | string | The WebSocket sub-protocol. See [RFC6455 Section 1.9](https://tools.ietf.org/html/rfc6455#section-1.9) for details. |
| headers | table | The outgoing request's headers. See [Headers](#headers) for details. |
| timeout | number | How many millisecond to wait before canceling the outgoing connection. |
| heartbeat | number | If set, automatically sends a heartbeat message (opcode 10) each `heartbeat` ms. Available to [wrapIo](#wrapIo) as well as [connect](#connect).|
| mask  | boolean | Whether or not to apply a mask on every message. Only available with [wrapIo](#wrapIo). |

### WS URL {#ws-url}

A table structure that represents a WebSocket URL.

#### Fields {#ws-url-fields}

| Field | Type   | Description |
|:-----:|:------:|:------------|
| host  | string | The hostname which this URL refer to. |
| port  | number | The port at which this URL open on the host. |
| tls   | boolean| Whether or not to use secure-layer, WSS instead of WS. |
| pathname | string | The path at which this URL refers to on the host. |

----

### Headers

An array of string list representing multiple headers, where the first value is the header name and the second is its value.

#### The Structuring {#headers-structuring}

**Full**: `{{header-name, header-value}, ...}}`.

  **Where**:

| Entry       | Type   | Description             |
|:------------|:------:|:------------------------|
| header-name | string | The name of the header. |
| header-value| string | The value of the header.|

**Examples**:

   - `{{"Expires", "-1"}, {"Accept", "text/plain"}}`.

----