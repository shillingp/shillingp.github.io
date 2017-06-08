---
layout: post
title: "ClojureScript + Reagent = 😍"
categories: clojurescript reagent react
---

You may at this point be wondering, WTF is Reagent.

You may or may not know about JSX which is the pragma that React uses to convert code into virtual DOM. In essence the line `<p>Hello<p>` will be compiled to `h("div", null, "Hello World")` if you would like to learn more about it Jason Miller's talk [Preact: Into the void 0](https://www.youtube.com/watch?v=LY6y3HbDVmg).

```html
function TextInput() {
  const { value: onChange } = this.props;

  return (
    <input
      type="text"
      class="text-input"
      value={value}
      onChange={onChange}
    />
  );
}
```

So this is how we do React in JavaScript and I'm sure you this is what you have done day in day out. But the title of this post is `ClojureScript + Reagent`, so lets get to it. So how do we use react in ClojureScript? There are 2 main ways, [Om](https://github.com/omcljs/om) and [Reagent](https://github.com/reagent-project/reagent), I prefer Reagent for its more clojure-y form.

So what is Reagent? It is a minimal interface between ClojureScript and React. Reagent allows us to create React components using pure data. If this is sounding fairly complex, let me help you out.

```html
<div class="im-a-div">
  <p>Hello <span style="color:red;">World</span</p>
</div>
```
```clj
[:div.im-a-div
 [:p "Hello "
  [:span {:style {:color "red"}}
    "World"]]]
```
These two code snippets will create an identical DOM. But these are far from similar, most importantly html is not a language it is merely a syntax it passes only necessary information about what we are trying to form. The ClojureScript version however is data, if you look at it again you will realise it is made of arrays, maps and strings. These are things we can reason with.


What does this really mean? It means that we can use core functions to actually manipulate our DOM. Lets say we have an TextInput component we will want to customize its properties for each use case.


```clj
(defn TextInput [text-atom & [attrs]]
  (let [def-attrs {:type "text"
                   :class "text-input"
                   :value @text-atom
                   :on-change #(reset! text-atom (.-target.value %}
    [:input (merge def-attrs attrs)]))}])}])

(TextInput (r/atom "hello world")
           {:class "new-class" :placeholder "enter text here"})
;;  [:input {:type "text"
;;           :class "new-class"
;;           :value "hello world"
;;           :placeholder "enter text here"
;;           :on-change fn}]
```

Of course in react we can pass props down the tree easily enough but in this way we know the exact state of our component because it is always pure state. So lets sum up what we know so far:

- Data === DOM
- Don't need to write HTML
- Get to use ClojureScript



### State Control and Stores

#### 1. Passing Props
You may have noticed in the previous section that we pass arguments for components as function arguments, this is because we don't have an added syntax, we are just calling functions. So what if we want named parameters?

```clj
(defn Example [{:keys [value]}]
  [:input {:value value}])

[:div.input-group
  [Example {:value "Hello, World!!"}]]
```

Well I think that just about covered that.


#### 2. Redux and Stores
Nope.

Don't need it. We need to have some sort of data store that can be accessed globally something Redux calls a store. However in Clojure all we need is an atom which is a core part of ClojureScript. Need to update some components when something changes, Reagent provides its own atom which just allows Reagent to update components when the atom is dereferenced.

Another nifty idea to think about is time travelling. David Nolen covered this very well in his talk [The Functional Final Frontier](https://www.youtube.com/watch?v=DMtwq3QtddY). Let me set the scene;

```clj
(def app-state (atom ""))
(def history (atom [[(js/Date.now) @app-state]]))

(add-watch app-state :history
  (fn [_ _ _ new-state]
    (swap! history conj [(js/Date.now) new-state])))

(reset! app-state "H")
(reset! app-state "He")
(reset! app-state "Hel")
(reset! app-state "Hell")
(reset! app-state "Hello")

@history
;; [[1496758420105 {:value "H"}]
;;  [1496758420191 {:value "He"}]
;;  [1496758420206 {:value "Hel"}]
;;  [1496758420220 {:value "Hell"}]
;;  [1496758420863 {:value "Hello"}]]
```

Now each time we make a change to the app state we automatically push the new state onto our history stack. When we want to undo something or jump back all we need to do is use an old history value. Of course we can do this in JavaScript but because of the efficiency of immutable data it is safer and more efficient to do this.

---

Go and check out Redux I highly recommend playing around at [reagent-project.github.io](https://reagent-project.github.io/). If you like what you see and want to give it a go I suggest using figwheel

```bash
lein new figwheel reagent-project -- --reagent
cd reagent-project
lein figwheel
```
