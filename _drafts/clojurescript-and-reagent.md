---
layout: post
title: "ClojureScript + Reagent = 😍"
categories: clojurescript reagent react
---

You may at this point be wondering, WTF is reagent.

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




a

a

a

a

a

a

```clj
(defn TextInput [text-atom]
 [:input {:type "text"
          :class "text-input"
          :value @text-input
          :on-change #(reset! text-atom (.-target.value %))}])

[:div
  [TextInput (r/atom "")]]
```



```clj````
