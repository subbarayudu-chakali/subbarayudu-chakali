# Node.js Interview Questions & Answers

This reference collects 100 interview-ready questions and answers about Node.js, spanning the topics that come up most often in real interviews. It moves from fundamentals and the event loop through modules, core APIs, HTTP servers, error handling, performance, security, testing, and a handful of advanced internals. Questions are grouped into sections by theme but numbered continuously from 1 to 100, so you can read straight through or jump to a weak spot. Each answer aims to be accurate and concise, with code snippets where they clarify the point. A short quick-fire round at the end covers rapid one-liners.

---

## Fundamentals

**1. What is Node.js?**

Node.js is a JavaScript runtime built on Chrome's V8 engine that lets you run JavaScript outside the browser, typically on the server. It provides a non-blocking, event-driven I/O model that makes it well suited for building scalable network applications.

It bundles V8 for executing JavaScript, libuv for asynchronous I/O and the event loop, and a set of core modules (`fs`, `http`, `net`, `stream`, and others) exposed as JavaScript APIs.

**2. What is V8 and what role does it play in Node.js?**

V8 is Google's open-source, high-performance JavaScript and WebAssembly engine, written in C++. It compiles JavaScript directly to native machine code using just-in-time (JIT) compilation rather than interpreting it.

In Node.js, V8 executes your JavaScript. Node adds bindings that let JavaScript call into C/C++ libraries (like libuv and OpenSSL) so you can do file I/O, networking, and cryptography that the language itself does not provide.

**3. Is Node.js single-threaded or multi-threaded?**

Your JavaScript executes on a single main thread with a single event loop. This is what people mean when they call Node "single-threaded."

Under the hood it is not purely single-threaded: libuv maintains a thread pool (default size 4) that handles blocking operations like file system access and DNS lookups, and native crypto/compression work. You can also spin up additional JavaScript threads with `worker_threads`.

**4. What are typical use cases for Node.js, and when should you avoid it?**

Node excels at I/O-bound, high-concurrency workloads: REST and GraphQL APIs, real-time apps (chat, collaboration, WebSockets), streaming services, microservices, and BFF (backend-for-frontend) layers.

It is a poor fit for heavy CPU-bound work (video encoding, large-scale numeric computation, cryptographic mining) because long synchronous tasks block the event loop. For those, offload to `worker_threads`, native addons, or a separate service.

**5. What does "non-blocking I/O" mean in Node.js?**

Non-blocking I/O means that when you start an operation like reading a file or querying a database, Node does not wait for it to finish. It registers a callback and continues executing other code; when the operation completes, the callback is queued and eventually run.

This lets a single thread handle thousands of concurrent connections, because time spent waiting on I/O is never idle CPU time.

**6. What is the difference between CommonJS and ES modules?**

CommonJS (CJS) is Node's original module system: you use `require()` to import and `module.exports` / `exports` to export. It loads modules synchronously and resolves them at runtime.

ES Modules (ESM) is the standard JavaScript module system using `import` / `export`. It is statically analyzable, supports top-level `await`, and loads asynchronously.

```js
// CommonJS
const fs = require('fs');
module.exports = { doThing };

// ES Modules
import fs from 'fs';
export { doThing };
```

You opt into ESM with `"type": "module"` in `package.json` or the `.mjs` extension; `.cjs` forces CommonJS.

**7. What is the global object in Node.js, and how does it differ from the browser?**

In Node the global object is `global` (and the standardized `globalThis`). In the browser it is `window`. Node has no `window`, `document`, or DOM.

Node provides globals like `process`, `Buffer`, `__dirname`, `__filename` (in CommonJS), `setImmediate`, and `require`. Note that variables declared at the top of a module are scoped to that module, not truly global, because each file is wrapped in a module function.

**8. What is the `process` object?**

`process` is a global that provides information about and control over the current Node process. Common uses include `process.env` (environment variables), `process.argv` (command-line arguments), `process.cwd()`, `process.exit()`, `process.pid`, and `process.platform`.

It is also an `EventEmitter`, emitting events like `exit`, `beforeExit`, `uncaughtException`, and `unhandledRejection`.

**9. How do you read environment variables and command-line arguments?**

Environment variables are available on `process.env`, and CLI arguments on `process.argv` (an array where index 0 is the node binary, index 1 is the script path, and the rest are user arguments).

```js
const port = process.env.PORT || 3000;
const args = process.argv.slice(2); // just the user args
```

**10. What is the difference between `process.env.NODE_ENV` values and why does it matter?**

`NODE_ENV` is a conventional environment variable (commonly `development`, `production`, or `test`) used to toggle behavior. Many frameworks and libraries enable optimizations and disable verbose logging when it is `production`.

It has no built-in meaning to Node itself; it is a convention your code and dependencies choose to respect.

## Event loop & async

**11. What is the event loop?**

The event loop is the mechanism, provided by libuv, that lets Node perform non-blocking I/O despite running JavaScript on a single thread. It continuously checks queues of completed operations and runs their callbacks.

It processes work in ordered phases, and between phases it drains the microtask queues (promises and `process.nextTick`).

**12. What are the phases of the event loop?**

libuv's loop runs through several phases in order on each iteration (a "tick"):

- **timers** — callbacks scheduled by `setTimeout` and `setInterval`.
- **pending callbacks** — certain deferred system callbacks.
- **idle, prepare** — internal use.
- **poll** — retrieve new I/O events and run their callbacks (this is where most work happens).
- **check** — `setImmediate` callbacks run here.
- **close callbacks** — e.g. `socket.on('close', ...)`.

Between each phase, the microtask queues (`process.nextTick` first, then promise jobs) are fully drained.

**13. What is libuv?**

libuv is the C library that provides Node's event loop, thread pool, and cross-platform asynchronous I/O. It abstracts platform differences (epoll on Linux, kqueue on macOS, IOCP on Windows).

It handles file system operations, TCP/UDP networking, DNS, child processes, and the worker thread pool.

**14. What is the difference between macrotasks and microtasks?**

Macrotasks are callbacks scheduled through the event loop phases: timers (`setTimeout`), `setImmediate`, and I/O callbacks. One macrotask runs per relevant phase iteration.

Microtasks are `process.nextTick` callbacks and resolved promise handlers. They run immediately after the currently executing operation completes and before the loop moves to the next phase, and the queue is drained completely each time.

**15. What is the difference between `process.nextTick`, `setImmediate`, and `setTimeout`?**

- `process.nextTick(cb)` runs `cb` before the event loop continues, after the current operation — it has the highest priority and can starve I/O if abused.
- `setImmediate(cb)` runs in the check phase, after the poll phase of the current iteration.
- `setTimeout(cb, 0)` runs in the timers phase of a future iteration, subject to a minimum delay.

```js
setTimeout(() => console.log('timeout'), 0);
setImmediate(() => console.log('immediate'));
process.nextTick(() => console.log('nextTick'));
// nextTick, then (immediate/timeout order varies at top level)
```

**16. Why can the order of `setTimeout(fn, 0)` and `setImmediate` be nondeterministic?**

At the top level of a script, the order depends on how quickly the loop reaches the timers phase versus the check phase, which is influenced by process startup timing. If both are scheduled inside an I/O callback, `setImmediate` always fires first because the loop is already past timers and enters the check phase next.

**17. What are callbacks and what is "callback hell"?**

A callback is a function passed to another function to be invoked later, commonly when an async operation completes. Node's traditional style is error-first callbacks.

"Callback hell" (the pyramid of doom) is deeply nested callbacks that become hard to read and maintain:

```js
readFile(a, (err, x) => {
  readFile(b, (err, y) => {
    readFile(c, (err, z) => { /* ... */ });
  });
});
```

Promises and `async/await` were introduced to flatten this.

**18. What is a Promise?**

A Promise represents the eventual result of an asynchronous operation. It is in one of three states: pending, fulfilled, or rejected. You consume it with `.then()`, `.catch()`, and `.finally()`.

Promises are chainable and compose well, avoiding deep nesting and enabling combinators like `Promise.all`, `Promise.race`, `Promise.allSettled`, and `Promise.any`.

**19. What is async/await and how does it relate to promises?**

`async/await` is syntactic sugar over promises. An `async` function always returns a promise, and `await` pauses execution until a promise settles, letting you write asynchronous code that reads like synchronous code.

```js
async function load() {
  try {
    const user = await getUser();
    const orders = await getOrders(user.id);
    return orders;
  } catch (err) {
    // handles rejection from either await
  }
}
```

**20. How do you run async operations in parallel versus in series?**

Use `await` sequentially for series (each waits for the previous). For parallel, start all operations first and await them together with `Promise.all`.

```js
// Series
const a = await taskA();
const b = await taskB();

// Parallel
const [a, b] = await Promise.all([taskA(), taskB()]);
```

`Promise.all` rejects as soon as any promise rejects; use `Promise.allSettled` when you need every result regardless of failures.

**21. What is the difference between `Promise.all`, `Promise.allSettled`, `Promise.race`, and `Promise.any`?**

- `Promise.all` — resolves with all values, or rejects on the first rejection.
- `Promise.allSettled` — always resolves with an array of `{status, value/reason}` for every promise.
- `Promise.race` — settles as soon as the first promise settles (fulfilled or rejected).
- `Promise.any` — resolves with the first fulfilled value, rejects only if all reject (with an `AggregateError`).

**22. How do you convert a callback-based function to return a promise?**

Use `util.promisify` for standard error-first callback functions, or wrap manually with `new Promise`.

```js
const { promisify } = require('util');
const fs = require('fs');
const readFile = promisify(fs.readFile);

// Or use the promise-based API directly:
const fsp = require('fs/promises');
await fsp.readFile('file.txt', 'utf8');
```

**23. What happens if you forget to `await` a promise?**

The function continues without waiting, so you may use results before they exist. If the promise rejects and nothing handles it, you get an unhandled rejection, which in modern Node terminates the process by default.

Linters (`no-floating-promises`) help catch missing awaits.

## Modules & npm

**24. What is the difference between `require` and `import`?**

`require` is CommonJS, synchronous, and can be called conditionally anywhere in a file. `import` is ESM, statically hoisted, and normally placed at the top of the module; dynamic `import()` returns a promise and can be used conditionally.

You cannot use `require` in an ESM file directly (though `createRequire` exists), and top-level `import` statements are not valid in CommonJS.

**25. How does Node resolve a module path?**

For `require('x')` Node checks, in order: core modules (like `fs`), then if the path starts with `./`, `../`, or `/` it resolves a file or directory relative to the current file. Otherwise it treats `x` as a package and walks up `node_modules` directories from the current location to the root.

For directories it reads the `main` (or `exports`) field of `package.json`, falling back to `index.js`.

**26. What is `package.json`?**

`package.json` is the manifest for a Node project. It records metadata (name, version, description), the entry point (`main`/`exports`), scripts, and dependency lists.

```json
{
  "name": "my-app",
  "version": "1.0.0",
  "type": "module",
  "scripts": { "start": "node index.js", "test": "jest" },
  "dependencies": { "express": "^4.19.0" },
  "devDependencies": { "jest": "^29.0.0" }
}
```

**27. What is `package-lock.json` and why does it matter?**

`package-lock.json` records the exact resolved version, integrity hash, and dependency tree of everything installed. It makes installs deterministic and reproducible across machines and CI.

You should commit it. `npm ci` installs strictly from the lockfile and fails if it is out of sync with `package.json`, which is ideal for CI.

**28. What is semantic versioning (semver)?**

Semver uses `MAJOR.MINOR.PATCH`. MAJOR is incremented for breaking changes, MINOR for backward-compatible features, and PATCH for backward-compatible bug fixes.

In `package.json`, `^1.2.3` allows updates that do not change the leftmost non-zero segment (up to but not including `2.0.0`), while `~1.2.3` allows only patch updates (up to `1.3.0`).

**29. What is the difference between dependencies and devDependencies?**

`dependencies` are packages your application needs at runtime (Express, a database driver). `devDependencies` are needed only during development or build (test runners, linters, bundlers, TypeScript).

`npm install --production` (or `NODE_ENV=production`) skips devDependencies. There are also `peerDependencies` (expected to be provided by the host project) and `optionalDependencies`.

**30. What is `npx`?**

`npx` runs a package's binary without installing it globally. If the package is not present it fetches it temporarily, executes it, and cleans up.

```bash
npx create-react-app my-app
npx eslint .
```

It is handy for one-off tools and scaffolding commands.

**31. What are npm workspaces?**

Workspaces let you manage multiple packages within a single repository (a monorepo) from one root `package.json`. npm hoists shared dependencies and symlinks local packages so they can reference each other.

```json
{ "workspaces": ["packages/*"] }
```

This simplifies dependency management and cross-package development. Yarn and pnpm offer similar features.

**32. What is the difference between `npm install` and `npm ci`?**

`npm install` resolves dependencies, may update `package-lock.json`, and installs into `node_modules`. `npm ci` does a clean, reproducible install strictly from the lockfile: it deletes `node_modules` first and errors if `package.json` and the lockfile disagree.

Use `npm ci` in CI/CD for speed and determinism.

**33. What is the difference between global and local package installation?**

Local installs (`npm install pkg`) place packages in the project's `node_modules` and are the default. Global installs (`npm install -g pkg`) put binaries on your system PATH, appropriate for CLI tools used across projects.

Prefer local installs plus `npx` for project tools so versions stay pinned per project.

**34. How does module caching work in Node.js?**

The first time a module is required, Node executes it and caches the resulting `module.exports` in `require.cache` keyed by resolved filename. Subsequent `require` calls return the cached object without re-executing.

This means module-level code runs once, so modules effectively act as singletons.

## Core APIs

**35. What is a Stream in Node.js?**

A stream is an abstraction for reading or writing data piece by piece rather than all at once. Streams are memory-efficient for large data and enable processing to start before all data arrives.

There are four types: Readable, Writable, Duplex, and Transform.

**36. What are the four types of streams?**

- **Readable** — you read from it (e.g., `fs.createReadStream`, an HTTP request on the server).
- **Writable** — you write to it (e.g., `fs.createWriteStream`, an HTTP response).
- **Duplex** — both readable and writable, with independent channels (e.g., a TCP socket).
- **Transform** — a duplex stream whose output is a transformation of its input (e.g., `zlib.createGzip`, crypto ciphers).

**37. What is piping and why use `pipeline`?**

`pipe()` connects a readable stream to a writable stream, automatically managing data flow and backpressure. `stream.pipeline()` does the same but also propagates errors and cleans up all streams properly.

```js
const { pipeline } = require('stream/promises');
await pipeline(
  fs.createReadStream('in.txt'),
  zlib.createGzip(),
  fs.createWriteStream('out.txt.gz')
);
```

Prefer `pipeline` over chained `pipe()` because `pipe` does not forward errors, which can leak resources.

**38. What is a Buffer?**

A `Buffer` is a fixed-length chunk of memory for handling raw binary data outside V8's heap. It is used for file contents, network packets, and any byte-level work.

```js
const buf = Buffer.from('hello', 'utf8');
console.log(buf.toString('hex'));
```

Use `Buffer.from`/`Buffer.alloc`; avoid the deprecated `new Buffer()` constructor, and prefer `Buffer.alloc` over `Buffer.allocUnsafe` unless you immediately overwrite the memory.

**39. What is the EventEmitter?**

`EventEmitter` is the core class that implements the observer pattern in Node. Objects emit named events and listeners subscribe to them. Many Node objects (streams, servers, `process`) are emitters.

```js
const { EventEmitter } = require('events');
const bus = new EventEmitter();
bus.on('order', (id) => console.log('got', id));
bus.emit('order', 42);
```

**40. How do you avoid memory leaks with EventEmitter?**

Remove listeners you no longer need with `off()`/`removeListener()`, and use `once()` for one-time events. Node warns when a single emitter exceeds 10 listeners for one event (`MaxListenersExceededWarning`), which often signals a leak from adding listeners in a loop.

You can adjust the limit with `setMaxListeners` when a high count is legitimate.

**41. What is the difference between synchronous and asynchronous `fs` methods?**

The `fs` module offers both. Synchronous methods (e.g., `fs.readFileSync`) block the event loop until the operation completes, which is fine at startup but harmful under load. Asynchronous methods (callback-based `fs.readFile` or promise-based `fs.promises.readFile`) do not block.

Use async in request-handling paths; sync versions are acceptable in CLI scripts or one-time initialization.

**42. What does the `path` module do?**

`path` provides utilities for working with file and directory paths in a cross-platform way, handling differences between POSIX and Windows separators.

```js
const path = require('path');
path.join('/foo', 'bar', 'baz.txt'); // '/foo/bar/baz.txt'
path.resolve('src', 'index.js');     // absolute path
path.extname('index.js');            // '.js'
```

**43. What does the `os` module provide?**

`os` exposes operating-system-level information such as `os.cpus()` (useful for sizing a cluster), `os.totalmem()`/`os.freemem()`, `os.hostname()`, `os.platform()`, and `os.tmpdir()`.

**44. What is `__dirname` and `__filename`?**

In CommonJS, `__dirname` is the absolute path of the directory containing the current module, and `__filename` is the absolute path of the current file. They are useful for building robust file paths independent of the working directory.

In ES modules they are not defined; instead derive them from `import.meta.url`:

```js
import { fileURLToPath } from 'url';
import path from 'path';
const __filename = fileURLToPath(import.meta.url);
const __dirname = path.dirname(__filename);
```

**45. How do you read a large file without loading it all into memory?**

Use a readable stream and process chunks as they arrive instead of `fs.readFile`, which buffers the entire file.

```js
const rl = require('readline').createInterface({
  input: fs.createReadStream('big.log')
});
for await (const line of rl) {
  process(line);
}
```

## HTTP & servers

**46. How do you create an HTTP server with the core `http` module?**

```js
const http = require('http');
const server = http.createServer((req, res) => {
  res.writeHead(200, { 'Content-Type': 'application/json' });
  res.end(JSON.stringify({ ok: true }));
});
server.listen(3000, () => console.log('listening on 3000'));
```

The callback receives a Readable request stream and a Writable response stream for each incoming request.

**47. How do you read the request body from a raw `http` request?**

The request is a stream, so you collect its chunks and concatenate them.

```js
let body = '';
req.on('data', (chunk) => { body += chunk; });
req.on('end', () => {
  const data = JSON.parse(body);
  // ...
});
```

Frameworks like Express provide body-parsing middleware so you rarely do this manually.

**48. What is Express?**

Express is a minimal, unopinionated web framework for Node built on the `http` module. It provides routing, middleware support, and helpers for requests and responses, greatly reducing boilerplate.

```js
const express = require('express');
const app = express();
app.get('/users/:id', (req, res) => res.json({ id: req.params.id }));
app.listen(3000);
```

**49. What is middleware in Express?**

Middleware are functions with the signature `(req, res, next)` that execute in order during the request-response cycle. They can inspect and modify the request/response, end the response, or call `next()` to pass control onward.

```js
app.use((req, res, next) => {
  console.log(`${req.method} ${req.url}`);
  next();
});
```

Middleware powers logging, authentication, body parsing, CORS, and more.

**50. How does routing work in Express?**

Routes map an HTTP method and path pattern to a handler. You can define them on the `app` or group them with `express.Router()`.

```js
const router = express.Router();
router.get('/', list);
router.post('/', create);
app.use('/products', router);
```

Path parameters (`:id`), query strings (`req.query`), and wildcards are supported.

**51. How do you handle errors in Express?**

Express recognizes error-handling middleware by its four arguments `(err, req, res, next)`. Pass errors to it by calling `next(err)`.

```js
app.use((err, req, res, next) => {
  console.error(err);
  res.status(err.status || 500).json({ error: err.message });
});
```

In Express 4, errors thrown in async handlers are not caught automatically — you must call `next(err)` or use a wrapper. Express 5 forwards rejected promises to the error handler automatically.

**52. How do you parse request bodies in Express?**

Use the built-in body parsers. `express.json()` parses JSON payloads and `express.urlencoded()` parses form submissions.

```js
app.use(express.json());
app.use(express.urlencoded({ extended: true }));
```

Set size limits to guard against oversized payloads.

**53. What is the difference between `res.send`, `res.json`, and `res.end`?**

`res.json` serializes an object to JSON and sets the `Content-Type` to `application/json`. `res.send` is more general — it infers the content type from the argument (string, Buffer, or object). `res.end` (from the core `http` module) ends the response, optionally writing a final chunk, without any content-type conveniences.

**54. How do you serve static files in Express?**

Use the built-in `express.static` middleware pointed at a directory.

```js
app.use(express.static('public'));
```

Requests are matched against files in that folder. For production, a reverse proxy or CDN typically serves static assets more efficiently.

**55. What is CORS and how do you enable it?**

CORS (Cross-Origin Resource Sharing) is a browser mechanism that controls which origins may call your API. The server signals permission via `Access-Control-*` response headers.

In Express, the `cors` middleware handles it: `app.use(cors({ origin: 'https://example.com' }))`. Restrict origins rather than using a wildcard for authenticated endpoints.

**56. How do you handle graceful HTTP shutdown?**

Stop accepting new connections with `server.close()` (which waits for in-flight requests), and set a timeout as a backstop.

```js
process.on('SIGTERM', () => {
  server.close(() => process.exit(0));
  setTimeout(() => process.exit(1), 10000).unref();
});
```

Also close database pools and other resources before exiting.

## Error handling

**57. How do you handle errors with async/await?**

Wrap awaited calls in `try/catch`. Because `await` unwraps promise rejections into thrown errors, a single `try/catch` can cover multiple awaits.

```js
try {
  const data = await fetchData();
} catch (err) {
  logger.error(err);
}
```

For code that fires many independent operations, `Promise.allSettled` lets you inspect each result's status without an early throw.

**58. What is the error-first callback pattern?**

Node's traditional convention passes an error as the first argument to a callback; it is `null` on success, and the result follows.

```js
fs.readFile('f.txt', (err, data) => {
  if (err) return handle(err);
  use(data);
});
```

Always check the error first and return early to avoid using undefined results.

**59. What is the difference between operational errors and programmer errors?**

Operational errors are expected runtime problems in correct programs (network timeouts, file not found, invalid user input). You handle them gracefully. Programmer errors are bugs (calling a function with the wrong type, reading a property of undefined) — you fix the code rather than catch them.

Crashing and restarting is often the safest response to programmer errors, while operational errors should be recovered from.

**60. What is `unhandledRejection`?**

`unhandledRejection` is a `process` event emitted when a promise rejects and no `.catch` handler is attached in the same tick. In modern Node the default behavior is to print the error and terminate the process.

```js
process.on('unhandledRejection', (reason) => {
  console.error('Unhandled rejection:', reason);
  process.exit(1);
});
```

**61. What is `uncaughtException` and should you keep running after it?**

`uncaughtException` fires when a synchronous error propagates to the top of the stack with no handler. You can listen for it, but the process is in an undefined state afterward, so the safe practice is to log, run minimal cleanup, and exit — then let a process manager restart you.

```js
process.on('uncaughtException', (err) => {
  console.error(err);
  process.exit(1);
});
```

Do not use it to swallow errors and continue serving requests.

**62. What happened to the `domain` module?**

The `domain` module was an early attempt to group and handle errors across asynchronous operations. It is deprecated because it has fundamental design problems and does not compose well with promises.

Modern code uses promises, `async/await`, `AsyncLocalStorage` for context propagation, and proper `try/catch` plus process-level handlers instead.

**63. How do you create custom error types?**

Extend the built-in `Error` class, which gives you a proper stack trace and lets callers use `instanceof` checks.

```js
class ValidationError extends Error {
  constructor(message, field) {
    super(message);
    this.name = 'ValidationError';
    this.field = field;
    this.status = 400;
  }
}
```

Attaching a status code or error code makes centralized handling easier.

**64. How do you propagate errors from a stream?**

Streams emit an `error` event rather than throwing. Unhandled `error` events crash the process, so always attach a handler, or use `pipeline`, which forwards and centralizes errors.

```js
readStream.on('error', handle);
```

## Performance & scaling

**65. What is the `cluster` module?**

`cluster` lets you fork multiple worker processes that share the same server port, so you can utilize all CPU cores with a single-threaded runtime. The master process distributes incoming connections among workers.

```js
const cluster = require('cluster');
const os = require('os');
if (cluster.isPrimary) {
  os.cpus().forEach(() => cluster.fork());
} else {
  require('./server');
}
```

Each worker is a separate process with its own memory and event loop.

**66. What are worker threads and when should you use them?**

`worker_threads` provides real multithreading within a single process, sharing memory via `SharedArrayBuffer` and passing messages. Use them for CPU-intensive JavaScript work (parsing, compression, image processing) that would otherwise block the event loop.

Unlike `cluster`, workers share the process and can share memory, making them lighter for computation offloading rather than for scaling network servers.

**67. What is the `child_process` module?**

`child_process` spawns separate OS processes. It offers `spawn` (streams output, good for long-running or large-output processes), `exec` (buffers output, convenient for short commands), `execFile` (runs a binary without a shell), and `fork` (spawns a new Node process with an IPC channel).

```js
const { spawn } = require('child_process');
const ls = spawn('ls', ['-la']);
ls.stdout.on('data', (d) => console.log(d.toString()));
```

**68. When would you choose cluster vs worker_threads vs child_process?**

- **cluster** — scale a network server across CPU cores using multiple processes sharing a port.
- **worker_threads** — offload CPU-bound JavaScript within one process with shared memory and low overhead.
- **child_process** — run external programs or fully isolated Node processes, communicating via streams or IPC.

Choose based on isolation needs, whether you are scaling I/O concurrency or crunching CPU, and whether you must invoke external binaries.

**69. How does load balancing work across cluster workers?**

By default on most platforms Node uses a round-robin approach in the primary process: it accepts connections and hands each to a worker in turn. On Windows the OS distributes connections directly to worker sockets, which can be less even.

You can also front multiple independent Node instances with an external load balancer like Nginx or a cloud load balancer.

**70. What is PM2?**

PM2 is a production process manager for Node. It keeps apps alive by restarting them on crash, runs them in cluster mode across cores without code changes, handles log aggregation, zero-downtime reloads, and startup on boot.

```bash
pm2 start app.js -i max   # cluster mode, one worker per core
pm2 reload app            # zero-downtime reload
```

**71. How do you avoid blocking the event loop?**

Keep synchronous work short. Avoid long-running loops, large synchronous JSON parsing, synchronous `fs` in request paths, and heavy regular expressions. Break large tasks into chunks with `setImmediate`, or offload CPU work to worker threads or a separate service.

Measuring event loop lag (via `perf_hooks.monitorEventLoopDelay`) helps detect blocking in production.

**72. What is caching and how does it help Node performance?**

Caching stores expensive-to-produce results so repeated requests are served faster and with less load. Common layers include in-memory caches (a `Map` or LRU cache), a shared cache like Redis, and HTTP caching via headers/CDN.

Be mindful of invalidation and memory growth for in-process caches; for multi-instance deployments a shared cache keeps data consistent.

**73. How do you scale a Node application horizontally?**

Run multiple stateless instances behind a load balancer, keep session/state in a shared store (Redis, a database) rather than in process memory, and use a message queue for background jobs. Containerize and orchestrate with Kubernetes or similar for elasticity.

Statelessness is the key enabler: any instance can handle any request.

## Security

**74. What are common security vulnerabilities in Node.js apps?**

Typical risks include injection (SQL, NoSQL, command), cross-site scripting (XSS) in rendered output, cross-site request forgery (CSRF), insecure dependencies, exposed secrets, insecure deserialization, and denial of service from unbounded input or ReDoS.

The OWASP Top 10 is a good checklist to review against.

**75. What is Helmet and what does it do?**

Helmet is Express middleware that sets a collection of HTTP security headers with sensible defaults — Content-Security-Policy, X-Content-Type-Options, Strict-Transport-Security, X-Frame-Options, and others.

```js
const helmet = require('helmet');
app.use(helmet());
```

It reduces exposure to common browser-based attacks with a single line.

**76. Why should you validate and sanitize input?**

Untrusted input is the root of most injection and XSS vulnerabilities. Validation enforces expected types, formats, and ranges; sanitization removes or escapes dangerous content. Together they keep malformed or malicious data out of your logic, queries, and output.

Use schema validators like Zod, Joi, or `express-validator`, and rely on parameterized queries for databases.

**77. Why should you avoid `eval` and similar constructs?**

`eval`, `new Function`, and passing strings to `setTimeout` execute arbitrary code. If any part of that string comes from user input, an attacker can run code in your process. They also defeat V8 optimizations.

Avoid them; use `JSON.parse` for data and proper parsers or lookups for dynamic behavior.

**78. How do you audit and manage dependency vulnerabilities?**

Run `npm audit` to report known vulnerabilities and `npm audit fix` to apply compatible updates. Keep dependencies current, minimize their number, and use tools like Snyk or Dependabot for continuous monitoring.

```bash
npm audit --production
```

Commit and use the lockfile so audited versions are exactly what ships.

**79. How should you manage secrets and environment variables?**

Never hardcode secrets or commit them to source control. Load them from environment variables (via the platform, or a `.env` file kept out of git with `dotenv` in development), and use a dedicated secrets manager (Vault, AWS Secrets Manager) in production.

Add `.env` to `.gitignore` and rotate credentials that may have leaked.

**80. How do you prevent brute-force and denial-of-service attacks?**

Apply rate limiting (e.g., `express-rate-limit`), set request body size limits, add timeouts, and put a reverse proxy or WAF in front of the app. Guard against ReDoS by avoiding catastrophic-backtracking regexes and validating input length.

**81. How do you securely store passwords?**

Never store plaintext. Hash passwords with a slow, salted algorithm designed for the purpose — bcrypt, scrypt, or Argon2 — and store only the hash.

```js
const bcrypt = require('bcrypt');
const hash = await bcrypt.hash(password, 12);
const ok = await bcrypt.compare(input, hash);
```

Use per-password salts (these algorithms handle salting) and a sufficient cost factor.

**82. Why prefer HTTPS and how do you handle it in Node?**

HTTPS encrypts traffic, preventing eavesdropping and tampering. In production you usually terminate TLS at a reverse proxy or load balancer (Nginx, a cloud LB) rather than in Node, which simplifies certificate management.

If terminating in Node, use the `https` module with a valid certificate and enforce HSTS.

## Testing & tooling

**83. What is the difference between Jest and Mocha?**

Jest is an all-in-one testing framework with a built-in assertion library, mocking, and code coverage, popular in the JavaScript/React ecosystem. Mocha is a flexible test runner that you pair with your own assertion library (like Chai) and mocking tools (like Sinon).

Jest favors convention and batteries-included; Mocha favors composition and configurability. Node also now ships a built-in `node:test` runner.

**84. What is the difference between unit, integration, and end-to-end tests?**

Unit tests verify a single function or module in isolation, often with dependencies mocked. Integration tests check that multiple components work together (e.g., a route hitting a real test database). End-to-end tests exercise the whole system through its external interface, as a user or client would.

A common guideline is many fast unit tests, fewer integration tests, and a small number of end-to-end tests.

**85. How do you mock dependencies in tests?**

Replace real dependencies with controllable stubs so tests are fast and deterministic. Jest provides `jest.fn()`, `jest.mock()`, and module auto-mocking; with Mocha you typically use Sinon spies, stubs, and mocks.

Mock network calls, timers, and the file system to isolate the unit under test and avoid flakiness.

**86. How do you debug a Node.js application?**

Start Node with `--inspect` (or `--inspect-brk` to break on the first line) and connect Chrome DevTools or your editor's debugger.

```bash
node --inspect-brk app.js
```

You can also use `console.log`, the built-in debugger statement, and VS Code's launch configurations for breakpoints and step-through debugging.

**87. What does the `--inspect` flag do?**

`--inspect` opens the V8 Inspector protocol on a port (default 9229), letting external debuggers attach for breakpoints, stepping, and profiling. `--inspect-brk` additionally pauses execution before the first line so you can set breakpoints before anything runs.

**88. How do you profile a Node application?**

Use the built-in profiler (`node --prof`, then process the log with `--prof-process`), Chrome DevTools CPU and memory profiles via `--inspect`, or the `perf_hooks` module for custom measurements. Clinic.js and 0x provide flame graphs and higher-level diagnostics.

Profiling reveals hot functions, event loop delays, and memory growth so you optimize the right thing.

**89. What are linters and formatters, and why use them?**

Linters like ESLint statically analyze code for likely bugs and style violations; formatters like Prettier automatically enforce consistent formatting. Together they catch mistakes early and keep a codebase uniform across a team.

Wiring them into pre-commit hooks and CI keeps standards enforced automatically.

**90. How do you measure code coverage?**

Coverage tools report which lines, branches, and functions your tests exercise. Jest has coverage built in (`jest --coverage`); with other runners you use `c8` or `nyc` (Istanbul).

Treat coverage as a signal, not a goal — high coverage of meaningless assertions is not useful, and 100% is rarely worth chasing.

## Advanced

**91. How does garbage collection work in Node.js?**

V8 uses a generational, mark-and-sweep garbage collector. Objects start in the "young generation" (new space), which is collected frequently and cheaply (scavenge); survivors are promoted to the "old generation," collected less often with more expensive mark-sweep-compact cycles.

GC pauses the JavaScript thread briefly. You rarely manage it manually, though flags like `--max-old-space-size` tune the heap limit.

**92. What causes memory leaks in Node, and how do you find them?**

Common causes are unbounded caches or arrays, forgotten timers, listeners added but never removed, and closures holding large objects. Symptoms include steadily rising heap usage and eventual out-of-memory crashes.

Diagnose by taking heap snapshots in Chrome DevTools (via `--inspect`), comparing them over time to find growing retained objects, and watching `process.memoryUsage()`.

**93. What is backpressure in streams and how is it handled?**

Backpressure occurs when a writable stream (or consumer) cannot keep up with the rate of incoming data. If ignored, data buffers in memory unboundedly.

`writable.write()` returns `false` when its internal buffer is full; a well-behaved producer pauses and resumes on the `drain` event. `pipe()` and `pipeline()` handle this automatically, which is a strong reason to prefer them over manual `data` handling.

**94. What are C++ addons in Node.js?**

C++ addons are native modules that let JavaScript call into C/C++ code, used for performance-critical work or to bind existing native libraries. They are built with node-gyp and modern ones use the stable Node-API (N-API), which keeps the addon compatible across Node versions.

Most applications never need them, but they underlie packages like native database drivers and cryptography.

**95. What is graceful shutdown and why does it matter?**

Graceful shutdown means, on receiving a termination signal (`SIGTERM`/`SIGINT`), the process stops accepting new work, finishes in-flight requests, closes connections and pools, and then exits. This prevents dropped requests and corrupted state during deploys and scaling events.

```js
process.on('SIGTERM', async () => {
  await server.close();
  await db.end();
  process.exit(0);
});
```

**96. What is `AsyncLocalStorage`?**

`AsyncLocalStorage` (from `async_hooks`) provides per-request context that persists across asynchronous calls without threading a parameter through every function. It is commonly used for request IDs, tracing, and per-request logging.

```js
const { AsyncLocalStorage } = require('async_hooks');
const als = new AsyncLocalStorage();
als.run({ reqId }, () => handle(req));
```

**97. What is the difference between the stream `flowing` and `paused` modes?**

A readable stream in flowing mode emits `data` events automatically as fast as data arrives. In paused mode you must explicitly call `read()` to pull data. Attaching a `data` listener or calling `resume()` switches to flowing; `pause()` switches back.

Async iteration (`for await...of`) and `pipe` manage the mode for you and respect backpressure.

**98. What is the module wrapper and why do modules have their own scope?**

Before executing a CommonJS module, Node wraps its code in a function:

```js
(function (exports, require, module, __filename, __dirname) {
  // module code
});
```

This gives each module its own scope (so top-level variables are not global) and injects the module-specific variables. It is why `__dirname` and `require` are available inside a module.

**99. What are `perf_hooks` used for?**

The `perf_hooks` module exposes the Performance Timing API in Node: `performance.now()` for high-resolution timing, `PerformanceObserver` for measuring marked spans, and `monitorEventLoopDelay()` for detecting event loop lag.

It is the standard way to add precise, low-overhead instrumentation and to catch blocking in production.

**100. How do you keep a long-running Node process healthy in production?**

Combine several practices: run under a process manager (PM2, systemd, or a container orchestrator) that restarts on crash; implement graceful shutdown; expose health/readiness endpoints; monitor memory, event loop lag, and error rates; set appropriate heap limits; and log structured data to a central system.

Crash-only design — let the process exit on unrecoverable errors and rely on the supervisor to restart — is generally safer than trying to keep a corrupted process alive.

## Quick-fire round

- **What returns the number of CPU cores?** `os.cpus().length`.
- **Default libuv thread pool size?** 4 (tunable via `UV_THREADPOOL_SIZE`).
- **Which is faster to write large output, `exec` or `spawn`?** `spawn` (streams; `exec` buffers).
- **How to exit with a failure code?** `process.exit(1)`.
- **JSON body parser in Express?** `express.json()`.
- **Promisify a callback function?** `util.promisify(fn)`.
- **Read a file as a promise?** `require('fs/promises').readFile`.
- **Highest-priority async callback?** `process.nextTick`.
- **Command for reproducible CI installs?** `npm ci`.
- **Flag to break on first line when debugging?** `--inspect-brk`.
- **Hash passwords with?** bcrypt, scrypt, or Argon2.
- **Header-setting security middleware?** `helmet`.
- **Signal for graceful shutdown?** `SIGTERM`.
- **Stream event signaling drained buffer?** `drain`.
- **Which module gives cross-platform paths?** `path`.

Prepare for a Node.js interview by pairing this theory with hands-on practice: build a small API, wire in streaming, add tests, and profile it under load so you can speak from experience. Interviewers value candidates who understand *why* the event loop, backpressure, and async patterns behave as they do — not just definitions. When you do not know an answer, reason out loud from fundamentals; showing how you think about single-threaded concurrency, error propagation, and scaling often matters more than reciting a perfect definition.
