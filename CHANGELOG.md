## v2.2.3 → v2.3.0
  * **DEPRECATED**: `atomically`, `ensure-atomically` -> `atomic`. The new macro is faster, more robust, more flexible. See docstring for details.
  * Official Redis command fns now contain a `:redis-api` metadata key that describes the first version of Redis to support the command.


## v2.2.0 → v2.2.3
  * Fix race condition for pool creation (thanks cespare!).
  * Fix unnecessary reflection for pool creation (thanks harob!).


## v2.1.4 → v2.2.0
  * Add `hmset*`, `hmget*` helpers.
  * Add `:clojurize?` option to `info*` helper.
  * Allow `hmget*`, `hgetall*`, `zinterstore*` to work with custom parsers.


## v2.1.0 → v2.1.4
  * Fixed `lua` clashing var name regex bug (thanks to Alex Kehayias for report).


## v2.0.0 → v2.1.0
  * Like `with-replies`, `wcar` macro can now take a first `:as-pipeline` arg:
  ```clojure
  (wcar {} (car/ping)) => "PONG"
  (wcar {} :as-pipeline (car/ping)) => ["PONG"]
  ```
  * **DEPRECATED**: `kname` -> `key`. The new fn does NOT automatically filter input parts for nil. This plays better with Redis' key pattern matching style, but may require manual filtering in some cases.


## v1.12.0 → v2.0.0

  * Refactored a bunch of code for simplicity+performance (~20% improved roundtrip times).
  * Upgraded to [Nippy v2][Nippy GitHub] for pluggable compression+crypto. See the [Nippy CHANGELOG][] for details.
  * Added early (alpha) Tundra API for semi-automatic cold data archiving. See the [README](https://github.com/ptaoussanis/carmine#tundra) for details.

  * **DEPRECATED**: `with-conn`, `make-conn-pool`, `make-conn-spec` -> `wcar`:
  ```clojure
  ;;; Old usage
  (def conn-pool (car/make-conn-pool <opts>))
  (def conn-spec (car/make-conn-spec <opts>))
  (defmacro wcar* [& body] `(car/with-conn conn-pool conn-spec ~@body))

  ;;; New idiomatic usage
  (ns my-app (:require [taoensso.carmine :as car :refer (wcar)]))
  (def server1-conn {:pool {<opts>} :spec {<opts>}})
  (wcar server1-conn (car/ping)) => "PONG"

  ;; or
  (defmacro wcar* [& body] `(car/wcar server1-conn ~@body))
  ```

  * **BREAKING**: Keyword args now get stringified instead of serialized:
    ```clojure
    (wcar* (car/set "foo" :bar) (car/get "foo")) => :bar  ; Old behaviour
    (wcar* (car/set "foo" :bar) (car/get "foo")) => "bar" ; New

    ;; Idiomatic usage is now:
    (wcar* (car/set "foo" :bar) (car/parse-keyword (car/get "foo"))) => :bar

    ;; Or use `freeze` if you want to serialize the input arg like before:
    (wcar* (car/set "foo" (car/freeze :bar)) (car/get "foo")) => :bar
    ```

  * **BREAKING**: Raw binary data return type is now unwrapped:
    ```clojure
    (wcar* (car/get "raw-bytes")) => [<length> [B@4d66ea88>]] ; Old behaviour
    (wcar* (car/get "raw-bytes")) => [B@4d66ea88> ; New
    ```

  * **BREAKING**: Distributed locks API has changed:
    The old API used an atom for connection config. The new API takes an explicit `conn` arg for every fn/macro.

  * Added message queue per-message backoffs:
    ```clojure
    ;; Handler fns can now return a response of form:
    {:status     <#{:success :error :retry}>
     :throwable  <Throwable>
     :backoff-ms <retry-backoff-ms>}
    ```

  * Added `redis-call` command for executing arbitrary Redis commands: `(redis-call [:set "foo" "bar"] [:get "foo"])`. See docstring for details.
  * Improved a number of error messages.
  * Fixed a number of subtle reply-parsing bugs.
  * Remove alpha status: `parse`, `return`, message queue ns.
  * **DEPRECATED**: `with-parser` -> `parse`.
  * **DEPRECATED**: `make-dequeue-worker` -> `worker`. See docstring for details.
  * Added support for unnamed vector (non-map) keys/args to `lua-script`.
  * **DEPRECATED**: `lua-script` -> `lua`.
  * **DEPRECATED**: `ring/make-carmine-store` -> `ring/carmine-store`.
  * Add `ensure-atomically`. See docstring for details.

## v1.10.0 → v1.12.0

  * Allow pipelined (non-throwing) Lua script exceptions.
  * Allow pipelined (non-throwing) parser exceptions.
  * Improved parser error messages.
  * Fix a number of subtle `with-replies` and `eval*` bugs.


## v1.9.0 → v1.10.0

  * Simplify reply parsing, remove `with-mparser` (vestigial).
  * Add additional reply parsing tests.
  * Added `raw` and `parse-raw` fns for completely unprocessed Redis comms.


## v1.8.0 → v1.9.0

  * **BREAKING**: Drop official Clojure 1.3 support.
  * Performance tweaks.
  * Internal message queue improvements, add interpretable handler return status.


## v1.7.0 → v1.8.0

  * **DEPRECATED**: `make-conn-spec` `:timeout` opt -> `:timeout-ms`.
  * Internal message queue improvements.


## v1.6.0 → v1.7.0

  * **DEPRECATED**: `make-keyfn` -> `kname`.
  * Pub/sub listener now catches handler exceptions.
  * Add experimental (alpha) distributed locks API.
  * `as-bool` now throws on incoercible arg.
  * Fix `interpolate-script` memoization bug (big perf. boost).
  * **BREAKING**: Remove `skip-replies`.
  * Allow `with-parser` to clear current parsers with `nil` fn arg.


## v1.5.0 → v1.6.0

  * Clean up Pub/sub listener thread (don't keep head!).
  * Add URI support to `make-conn-spec`.


## For older versions please see the [commit history][]

[commit history]: https://github.com/ptaoussanis/carmine/commits/master
[API docs]: http://ptaoussanis.github.io/carmine
[Taoensso libs]: https://www.taoensso.com/clojure-libraries
[Nippy GitHub]: https://github.com/ptaoussanis/nippy
[Nippy CHANGELOG]: https://github.com/ptaoussanis/nippy/blob/master/CHANGELOG.md
