# About
These are unofficial docs for the Luvit library [coro-http](https://github.com/luvit/lit/blob/master/deps/coro-http.lua) that was originally made by Tim Caswell, and the help of the contributors.

It can be used as a great replacement for the built-in http library in Luvit for those who don't like the callback style, this library uses Lua's coroutines to keep the sync style with optional coroutines wrapping for asynchronous execution. And for its chunks handling, and other behind the scenes.

These docs will guide you through all the available methods and their usage. If you find any wrong documentation, confusing wording, or even typos, please open an issue or a PR.

# Documentations

## Functions

### createServer (host, port, onConnect)
Creates a new server instance and asynchronously binds it to host:port.

#### Parameters
- **host** *(string)*: The host which the server corresponds to.
- **port** *(number)*: The opened port for the server to listen to.
- **onConnect** *(function)*: A callback that will be asynchronously called every time a new connection is established to the server.
The callback has the following parameters:

| Param | Type   | Description |
|:------|:------:|:------------|
| head  | table  | The headers provided by the client alongside with general informations about the request. |
| body  | string | The provided request's body as string, empty string in case nothing is provided. |
| socket | [uv_tcp_t](https://github.com/luvit/luv/blob/master/docs.md#uv_tcp_t--tcp-handle) / [uv_pipe_t](https://github.com/luvit/luv/blob/master/docs.md#uv_pipe_t--pipe-handle)| The socket that the connection was bound to. |

---

### parseUrl (url)
Parses the given string representing an HTTP(s) URL to a Lua table.

#### Parameters:
- **url** *(string)*: The URL that should be parsed.
  - Must be a valid HTTP(s) URL or an error will be raised.

#### Returns:
1. *(table)*: The parsed URL as a Lua table.

The returned table contains the following fields:

| Fields | Type   | Description |
|:------ |:------:|:------------|
| tls  | boolean  | Whether the given URL does use TLS (HTTPS) or not. |
| host | string   | The full host URI including its port. |
| hostname | string | The name of the host, does not include the host port. |
| port | number | The port that the host should use for the provided URL. |
| path | string | The requested path from the host (the part after the host name `/`). |

---

### getConnection (host, port [, tls [, timeout]])
Establishes a new TCP connection with the given host on the given port.

  – If the connection was saved previously using `saveConnection`, calling this will return that saved connection and un-save it.

  – If the saved connection was closed, a new connection will be established instead.

#### Parameters:
- **host** *(string)*: The host which the established connection refers to.
- **port** *(number)*: The port that this connection should use to connect to the host.
- **tls** *(boolean / table)* ***optional***: The use of TLS encrypted protocol.
  - Boolean value whether to use TLS cert or not.
  - Table value to use TLS, with optional configurations.

  *default*: `nil`.

  In case of using a table value for `tls`, acceptable fields are;
    
| Field | Type   | Description |
|:------|:------:|:------------|
| protocol | string | The secure transport layer protocol to use, supported values depends on the openssl version, openssl TLS 1.3 compatible supports: `TLS` (default) or `DTLS`. LibreSSL and others uses `SSLv23` by default and supports other versions.
| ciphers | string | The encryption algorithm to encrypt data with, value **MUST** be a valid cipher suite string. Defaults are `TLS_AES_128_GCM_SHA256:TLS_AES_128_CCM_SHA256` for TLS 1.3, `ECDHE-RSA-AES128-SHA256:AES128-GCM-SHA256` for LTS 1.2, `RC4:HIGH:!MD5:!aNULL:!EDH` for LTS 1.0. |
| key | string | The PEM key of the supplied certification (if `cert` field is passed). |
| cert | string | The TLS x509 certification used for the handshake as string. Used alongside with field `key` or it gets ignored. See field `ca` for defaults. |
| ca | string / table | TODO |
| insecure | boolean | TODO |
*All of the fields are optional and should only be touched when you know what you are doing.*

- **timeout** *(number)* ***optional***: How much time to wait for the response before canceling the request.
  - Time is given in milliseconds.
  - if nothing is supplied libuv will timeout after an undefined amount of seconds.

*default*: `nil`.

#### Returns
1. *(table)*: The established connection.

The returned table contains the following fields:

| Field | Type   | Description |
|:------|:------:|:------------|
| socket| [uv_tcp_t](https://github.com/luvit/luv/blob/master/docs.md#uv_tcp_t--tcp-handle) | The TCP handle used to bind the established connection. |
| host | string | Same as the parameter `host`. |
| port | number | Same as the parameter `port`. |
| tls  | table / boolean | Same as the parameter `tls`. |
| read | function | TODO |
| write| function | TODO |
| updateEncoder | function | TODO |
| updateDecoder | function | TODO |
| reset | function | TODO | 
---

### saveConnection (connection)
Saves a pre-established TCP connection to be used later instead of establishing a new one.

  – If the passed connection is already closed nothing will be saved.

#### Parameters:
- **connection** *(table)*: A TCP connection returned by `getConnection`.

---

### request  (method, url[, headers[, body[, timeout]]])
Synchronously performs an HTTP(s) request after establishing a connection with the said host.

#### Parameters
-  **method** *(string)*: An all uppercase HTTP method.

- **url** *(string)*: An HTTP(s) URL that the request should be sent to.

- **headers** *(table)* ***optional***: An array of headers, where a header is a table its first entry the header name, and its second is the value.
  - Eg. `{{"name", "value"}, {"Expires", "-1"}}`

- **body** *(string)* ***optional***: The request's body as string (if the request method requires one).

*default*: ` ` (empty string).

- **timeout** *(number)* ***optional***: How much time to wait for the response before canceling the request.

  - Time is given in milliseconds.
  - if nothing is supplied libuv will timeout after an undefined amount of seconds.

*default*: `nil`.

---
