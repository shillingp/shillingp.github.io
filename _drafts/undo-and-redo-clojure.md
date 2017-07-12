---
layout: post
title: "Undo and Redo in Clojure"
categories: clojure clojurescript history
---

In ClojureScript we store all of our app state in a single source of data. As we have a single source of truth it lends itself well to 'time travelling' or having history associated with it. This is because we know that no other vital data will be 'left behind' if we jump back and forth between history states because there is no other vital data if you have only one data source.

To visualise the problem at hand open up a new tab in your browser, then search for something, anything. You will know that the back button can now be pressed to take you back to the previous page and when you press it the forward button will be available. However now if you click a link you will see that the forward button is not available anymore, the "forward history" has been removed, this is because things get unclear if you retain it.

So what have we discovered from doing this activity? Well first of all you probably realised that you knew exactly what would happen at every stage. It has become second nature to use to browse in this way and when we can't jump back and forth we get a bit uneasy. The other thing we learned is that the "backward history" is more important than the "forward history" therefore we clear the "forward history" whenever we add new data.

To implement an undo/redo history. We need 3 things.
  1. A single source of application state
  2. A single history atom or undo stack
  3. A single future atom or redo stack


|variable   |initial|concat "a" |concat "b"   |concat "c"         |
|-----------|-------|-----------|-------------|-------------------|
|app-state  |""     |"a"        |"ab"         |"abc"              |
|app-history|[""]   |["" "a"]   |["" "a" "ab"]|["" "a" "ab" "abc"]|
|app-future |[]     |[]         |[]           |[]                 |


|variable   |initial            |undo         |undo         |
|-----------|-------------------|-------------|-------------|
|app-state  |"abc"              |"ab"         |"a"          |
|app-history|["" "a" "ab" "abc"]|["" "a" "ab"]|["" "a"]     |
|app-future |[]                 |["abc"]      |["abc" "ab"] |

|variable   |initial      |redo         |append "d"         |
|-----------|-------------|-------------|-------------------|
|app-state  |"a"          |"ab"         |"abd"              |
|app-history|["" "a"]     |["" "a" "ab"]|["" "a" "ab" "abd"]|
|app-future |["abc" "ab"] |["abc"]      |[]                 |

These three variables are clojure Atoms, `app-state` is going to be changing constantly and each change will be pushed onto `app-history` as it does if you visit a web page. The `app-future` stack will only be appended to when we undo something and if we add new state the `app-future` stack will be cleared.

So enough logic, lets see some code.

```clj
(def app-state (atom 0))

(def app-history (atom [@app-state]))
(def app-future (atom []))
```

Simple assigning our variables to atoms.

Next we want to make sure that we are able to undo and redo.

```clj
(defn can-undo []
  (> (count @app-history) 1))

(defn can-redo []
  (> (count @app-future) 0))
```

So far so good, we have a few variables and a few basic functions that check the size of the `app-history` and `app-future` variables.

Next we need to look into the logic behind making an undo action.
Here you will see that it takes 3 steps to commit an undo request.


|variable   |initial      |step 1.      | step 2. | step 3. |
|-----------|-------------|-------------|-------------------|
|app-state  |"ab"         |"ab"         |"ab"     |"a"      |
|app-history|["" "a" "ab"]|["" "a" "ab"]|["" "a"] |["" "a"] |
|app-future |[]           |["ab"]       |["ab"]   |["ab"]   |

  1. push the last element in app-history to app-future
  2. pop the last element off app-history
  3. set app-state to be the last element in app-history

```clj
(defn do-undo []
  (when (can-undo)
    (swap! app-future conj (last @app-history))
    (swap! app-history pop)
    (reset! app-state (last @app-history))))
```

Now we need to do the opposite, we need to make a redo step. Which is almost the exact opposite of undo, with the exception of step 3.
To commit a redo request we again need to take 3 steps.

|variable   |initial      |step 1.      | step 2.     |step 3.      |
|-----------|-------------|-------------|-------------|-------------|
|app-state  |"a"          |"a"          |"a"          |"ab"         |
|app-history|["" "a"]     |["" "a" "ab"]|["" "a" "ab"]|["" "a" "ab"]|
|app-future |["abc" "ab"] |["abc" "ab"] |["abc"]      |["abc"]      |

  1. push the last element of app-future to app-history
  2. pop the last element off app-future
  3. set app-state to be the last element in app-history

```clj
(defn do-redo []
  (when (can-redo)
    (swap! app-history conj (last @app-future))
    (swap! app-future pop)
    (reset! app-state (last @app-history))))
```

So we have undo and redo working perfectly they will skip back and forth with ease, however they have no history to traverse.

We need to make a function that will form our tree as we add new data, or visit new pages in the example of websites. When we are pushing new state we clear the `app-future` so we do not break the history when we come to undo. We also reset the `app-state` to whatever state we supplied, we will not need this later but until then know that whatever is passed to our push-change function will become our state. Then we push the new-state onto the end of the `app-history` stack.

```clj
(defn push-state-change [new-state]
  (reset! app-state new-state)
  (reset! app-future [])
  (swap! app-history conj new-state))
```

We can use this immediately in our app, however as it is we must carefully pass all of our state into our `push-change` function. But what if we could just mutate app-state and let the app fix our history. Well we can!! Just use a watch and when we change the app-state by using `reset!` or `swap!` it will run the `push-state-change` with the new value of `app-state` so if we do `(reset app-state 3)` it will pass 3 to the `push-state-change` function.

```clj
(add-watch app-state :history
  (fn [_ _ _ new-state]
    (when-not (= new-state (last @app-history))
      (push-state-change new-state))))
```

Now when we mutate `app-state` not only will the current state be changed but the history will be updated accordingly. For example

```clj
(reset app-state {})                ;; -> {}
(swap! app-state assoc :foo 0)      ;; -> {:foo 0}
(swap! app-state update :foo inc)   ;; -> {:foo 1}
(swap! app-state update :foo inc)   ;; -> {:foo 2}
(swap! app-state update :foo inc)   ;; -> {:foo 3}
(do-undo)                           ;; -> {:foo 2}
(do-undo)                           ;; -> {:foo 1}
(do-redo)                           ;; -> {:foo 2}
```

Great it looks like we are all done, everything is working. But just to be sure lets check whats going on behind the scenes when we change `app-state`.

```clj
;;                              app-state   app-history     app-future
(swap! app-state str "a")   ;; -> "a"    ["" "a"]             []
(swap! app-state str "b")   ;; -> "ab"   ["" "a" "ab"]        []
(swap! app-state str "c")   ;; -> "abc"  ["" "a" "ab" "abc"]  []
(do-undo)                   ;; -> "ab"   ["" "a" "ab"]        ["abc"]
(do-undo)                   ;; -> "a"    ["" "a"]             ["abc" "ab"]
(do-redo)                   ;; -> "ab"   ["" "a" "ab"]        ["abc"]
(swap! app-state str "d")   ;; -> "abd"  ["" "a" "ab" "abd"]  []
```
Perfect. This has got limitations, the entire state of the app and therefore the app itself must rely on a single global atomic store. This has its benefits and its fair share of problems. I will direct you to Roman Liutikov's Medium post [Single atom state tree buzzword explained](https://medium.com/@roman01la/single-atom-state-tree-buzzword-explained-4935d265343) which as the name suggest explains in more detail about global app state.

Here is the full code in case you want to use it. If you are interested I have made a naive [multi atom history](https://gist.github.com/shillingp/3feb15212968d0ff6ac1a060c4544dc2).

```clj
(def app-state (atom {}))

(def app-history (atom [@app-state]))
(def app-future  (atom []))


(defn clear-all-state []
  "hard reset both history and future"
  (reset! app-history [])
  (reset! app-future  []))


(defn can-undo []
  "is the app-history large enough to step back"
  (> (count @app-history) 1))

(defn can-redo []
  "is the app-future large enough to step forward"
  (> (count @app-future) 0))


(defn do-undo []
  "step back in time to the previous state"
  (when (can-undo)
    (swap! app-future conj (last @app-history))
    (swap! app-history pop)
    (reset! app-state (last @app-history))))

(defn do-redo []
  "step forward in time to the next state"
  (when (can-redo)
    (swap! app-history conj (last @app-future))
    (swap! app-future pop)
    (reset! app-state (last @app-history))))


(defn push-state-change [new-state]
  "grow the history stack"
  (reset! app-future [])
  (swap! app-history conj new-state))


(add-watch app-state :history
  (fn [_ _ _ new-state]
    "watch the new state changes and push to stack"
    (when-not (= new-state (last @app-history))
      (push-state-change new-state))))
```
