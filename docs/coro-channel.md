---
layout: doc
---

# Documentation

Unofficial docs for the [coro-channel](https://github.com/luvit/lit/blob/master/deps/coro-channel.lua) module, version 3.0.2.

[coro-channel](https://github.com/luvit/lit/blob/master/deps/coro-channel.lua) is a wrapper module that provides manipulating read/write streams in a sync style of code using Lua coroutines.

----

## Functions

----

### wrapRead(socket) {#wrapRead}

Wraps a [stream](https://github.com/luvit/luv/blob/master/docs.md#uv_stream_t--stream-handle) handle for a coroutine based reading.

*This method does not require running in a coroutine*

#### Parameters {#wrapRead-parameters}

| Param  | Type   | Description |
|:------:|:------:|:------------|
| socket | [uv_stream_t](https://github.com/luvit/luv/blob/master/docs.md#uv_stream_t--stream-handle) | The stream [handle](https://github.com/luvit/luv/blob/master/docs.md#uv_handle_t--base-handle) to be wrapped for reading. |

#### Returns {#wrapRead-returns}

| Return | Type     | Description |
|:-------|:--------:|:------------|
| reader | function | See [reader](#reader). |
| closer | function | see [closer](#closer). |

----

### wrapWrite(socket) {#wrapWrite}

Wraps a [stream](https://github.com/luvit/luv/blob/master/docs.md#uv_stream_t--stream-handle) handle for a coroutine based writing.

#### Parameters {#wrapWrite-parameters}

| Param  | Type   | Description |
|:------:|:------:|:------------|
| socket | [uv_stream_t](https://github.com/luvit/luv/blob/master/docs.md#uv_stream_t--stream-handle) | The stream [handle](https://github.com/luvit/luv/blob/master/docs.md#uv_handle_t--base-handle) to be wrapped for writing. |

#### Returns {#wrapWrite-returns}

| Return | Type     | Description |
|:-------|:--------:|:------------|
| writer | function | See [writer](#writer). |
| closer | function | see [closer](#closer). |

----

### wrapSteam(socket) {#wrapStream}

Wraps a [stream](https://github.com/luvit/luv/blob/master/docs.md#uv_stream_t--stream-handle) handle for both writing and reading.

#### Parameters {#wrapStream-parameters}

| Param  | Type   | Description |
|:------:|:------:|:------------|
| socket | [uv_stream_t](https://github.com/luvit/luv/blob/master/docs.md#uv_stream_t--stream-handle) | The stream [handle](https://github.com/luvit/luv/blob/master/docs.md#uv_handle_t--base-handle) to be wrapped for writing. |

#### Returns {#wrapStream-returns}

| Return | Type     | Description |
|:-------|:--------:|:------------|
| reader | function | See [reader](#reader). |
| writer | function | See [writer](#writer). |
| closer | function | see [closer](#closer). |

----

## Returned Functions

----

### reader() {#reader}

Yields the running coroutine and resumes it after receiving a chunk of data.

#### Returns {#reader-returns}

| Return | Type     | Description | Provided On |
|:-------|:--------:|:------------|:-----------:|
| data   | string / boolean | Returns the chunk of data as string on success, otherwise false. | Always |
| error  | string   | A string containing the error code caused the failure. | Failure Only |

#### Examples {#reader-examples}

An example of using `reader` in an iterator would be:

```lua
local handle = uv.new_tty(1, true) -- or any kind of streams
local reader = coro_channel.wrapRead(handle)

for chunk, err in reader do
  if err then
    print("An error has occurred: " .. err)
    break
  end
  print("Data chunk successfully read: " .. chunk)
end

print("Reading data done")
```

----

### writer(chunk) {#writer}

Yields the running coroutine and resumes it when done writing the provided chunk of data.

#### Parameters {#writer-parameters}

| Param  | Type   | Description |
|:------:|:------:|:------------|
| chunk  | string / table / nil | `nil` indicates EOF and will completely close the stream if there's nothing to read, otherwise it will shutdown the writing duplex only.<br> If the provided value is a string it will be written to the stream as a single chunk.<br> If a table of strings is provided the strings will be written in a one system call as if they were concatenated. |

#### Returns {#writer-returns}

| Return | Type     | Description | Provided On |
|:-------|:--------:|:------------|:-----------:|
| success| boolean  | Whether or not the operation was successful. | Always |
| error  | string   | A string containing the error code caused this failure | Failure Only |

#### Examples {#writer-examples}

Assuming the following example of using `writer` with a TTY handle running in a coroutine:

```lua
local handle = uv.new_tty(0, false) -- or any kind of streams
local writer = coro_channel.wrapWrite(handle)

local tbl = {"Hello ", "There", "!!", "\n"}
local success, err = writer(tbl)
if not success then
  print("An error has occurred: " .. err)
  return
end
```

Note: usually though you don't use it like this with a TTY handle, it was used like this in the above example just to make it simple.

----


### closer() {#closer}

Closes the wrapped stream completely if it wasn't already closed. You cannot read or write from a closed stream. Note that this returns immediately, even if the stream isn't closed yet.

----
