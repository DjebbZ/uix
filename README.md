<img src="logo.png" width="125" />

_Experimental ClojureScript wrapper for modern React.js_

Bug reports, feature requests and PRs are welcome 👌

There are no versioned releases yet, use `deps.edn` to depend on the code via git deps.

[![CircleCI](https://circleci.com/gh/roman01la/uix.svg?style=svg)](https://circleci.com/gh/roman01la/uix)

[API Documentation](https://roman01la.github.io/uix/)

```clj
(require '[uix.core.alpha :as uix])

(defn button [{:keys [on-click]} text]
  [:button.btn {:on-click on-click}
    text])

(defn app []
  (let [state (uix/state 0)]
    [:<>
      [button {:on-click #(swap! state dec)} "-"]
      [:span @state]
      [button {:on-click #(swap! state inc)} "+"]]))

(uix/render [button {:on-click js/console.log} "button"])
```

## Recipes

- [State hook](https://github.com/roman01la/uix/blob/master/dev/uix/recipes/state_hook.cljs)
- [Global state and effects](https://github.com/roman01la/uix/blob/master/dev/uix/recipes/global_state.cljs)
- [Dynamic styles](https://github.com/roman01la/uix/blob/master/dev/uix/recipes/dynamic_styles.cljs)
- [Lazy loading](https://github.com/roman01la/uix/blob/master/dev/uix/recipes/lazy_loading.cljs)
- [Server-side rendering](https://github.com/roman01la/uix/blob/master/dev/uix/recipes/server_rendering.clj)
- [Interop between UIx and JS components](https://github.com/roman01la/uix/blob/master/dev/uix/recipes/interop.cljs)
- [Popups](https://github.com/roman01la/uix/blob/master/dev/uix/recipes/popup.cljs)

## Features

### Hiccup syntax extension

- `[:div#id.class]` or `[:#id.class]`
- `[:> js/Component attrs & children]` - interop with JS components
- `[:<> attrs & children]` - `React.Fragment`
- `[:-> element :#selector]` or `[:-> element dom-node]` - `React.createPortal`
- `[:# {:fallback element} & children]` - `React.Suspense`

### Hooks

React Hooks in idiomatic Clojure style

```clj
;; state hook
;; (mutable ref type, re-renders component when mutated)
(let [state (uix/state 0)]
  (swap! state inc)
  @state) ; 1

;; ref hook
;; (mutable ref type, doesn't cause re-renders)
(let [ref (uix/ref 0)]
  (swap! ref inc)
  @ref) ; 1

;; effect hook
(uix/effect!
  (fn []
    (prn "after update")
    #(prn "before unmount"))
  [deps])

;; convenience macro for uix.core/effect!
(uix/with-effect [deps]
  (prn "after update")
  #(prn "before unmount"))

;; more in uix.core.alpha ns
```

### Attributes syntax extension

Injects provided function into attributes transformation stage. Could be used for various side effects, such as processing styles with CSS-in-JS libraries (see `uix.recipes.dynamic-styles`).

```clj
(uix.core.alpha/add-transform-fn
  (fn [attrs]
    (my-transform-attrs attrs)))
```

### Hiccup pre-compilation (advanced)

_NOTE: UIx interpreter is already super fast (2x faster than Reagent and only 1.5x slower than vanilla React).
Use pre-compilation ONLY if you are hitting performance problems._

Optionally compiles Hiccup into inlined React elements at compile-time

```clj
(uix/html
  [:h1 "Title"])

;; emits this
{
  $$typeof: Symbol.for("react.element"),
  key: null,
  ref: null,
  props: { children: "Title" },
  _owner: null
}
```

Compiler will try to inline as much as possible based on type information provided by ClojureScript's compiler (inspired by [“On fast ClojureScript React templates”](https://kevinlynagh.com/notes/fast-cljs-react-templates/)). When it is unable to determine type of the value it will emit interpretation call for the value and print a warning asking to annotate a value with either `^:inline` or `^:interpret`.

`uix.core.alpha/defui` does the same as `html` macro and additionally skips type checking arguments if a spec is provided.

```clj
(s/fdef button
  :args (s/cat :attrs map? :child string?)

(uix/defui button [{:keys [on-click]} child]
  [:button {:on-click on-click} child])
```

### Lazy loading components

Loading React components on-demand as Closure modules. See [code splitting](https://clojurescript.org/guides/code-splitting) guide and how lazy loading is used in React with Suspense: [guide](https://reactjs.org/docs/code-splitting.html).

```clj
(uix/require-lazy '[uix.components :refer [ui-list]])

[:# {:fallback "Loading..."}
  (when show?
    [ui-list])]
```

### Server-side rendering (JVM)

See an example in `uix.recipes.server-rendering`

```clj
(uix/render-to-string element) ;; see https://reactjs.org/docs/react-dom-server.html#rendertostring
(uix/render-to-static-markup element) ;; see https://reactjs.org/docs/react-dom-server.html#rendertostaticmarkup

;; Streaming HTML
(uix/render-to-stream element {:on-chunk f}) ;; see https://reactjs.org/docs/react-dom-server.html#rendertonodestream
(uix/render-to-static-stream element {:on-chunk f}) ;; see https://reactjs.org/docs/react-dom-server.html#rendertostaticnodestream
```

## Benchmarks

### Hiccup interpretation

```
react x 23202 ops/s, elapsed 431ms
uix-compile x 21834 ops/s, elapsed 458ms
uix-interpret x 14368 ops/s, elapsed 696ms
reagent-interpret x 7174 ops/s, elapsed 1394ms
```

### SSR on JVM

| lib           | test 1   | test 2 | test 3  |
| ------------- | -------- | ------ | ------- |
| rum           | 107.8 µs | 3.6 ms | 7.7 ms  |
| uix           | 120.8 µs | 3.8 ms | 8.1 ms  |
| uix streaming | 115.7 µs | 3.4 ms | 7.6 ms  |
| hiccup        | 205.7 µs | 6.5 ms | 16.6 ms |

### TodoMVC bundle size

| lib     | size  | gzip |
| ------- | ----- | ---- |
| rum     | 254KB | 70KB |
| reagent | 269KB | 74KB |
| uix     | 234KB | 65KB |

## Building

- Recipes `clojure -A:dev -m figwheel.main -O advanced -bo dev:prod`
- Benchmark `clojure -A:dev:benchmark -m figwheel.main -O advanced -bo benchmark`
- SSR Benchmark `clojure -A:dev:benchmark -m uix.benchmark`
- Tests `clojure -A:dev:test -m cljs.main -re node -m uix.compiler-test`
