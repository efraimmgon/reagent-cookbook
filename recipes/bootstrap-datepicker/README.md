# Problem

You want to add [bootstrap-datepicker](https://github.com/eternicode/bootstrap-datepicker) to your [reagent](https://github.com/reagent-project/reagent) web application.

# Solution

We are going to follow this [example](http://runnable.com/UmOlOZbXvZRqAABU/bootstrap-datepicker-example-text-input-with-specifying-date-format2).

*Steps*

1. Create a new project
2. Add necessary items to `resources/public/index.html`
3. Add text input to `home-render`
4. Convert javascript to clojurescript and put inside a *did-mount* function called `home-did-mount`
5. Use `home-render` and `home-did-mount` to create a reagent component called `home`
6. Add externs

#### Step 1: Create a new project

```
$ lein new rc bootstrap-datepicker
```

#### Step 2: Add necessary items to `resources/public/index.html`

```html
<!DOCTYPE html>
<html lang="en">
  <body>
    <div id="app"></div>

    <!-- ATTENTION \/ -->
    <!-- jQuery -->
    <script src="https://ajax.googleapis.com/ajax/libs/jquery/1.11.1/jquery.min.js"></script>
    <!-- Bootstrap -->
    <link rel="stylesheet" href="https://maxcdn.bootstrapcdn.com/bootstrap/3.2.0/css/bootstrap.min.css">
    <script src="https://maxcdn.bootstrapcdn.com/bootstrap/3.2.0/js/bootstrap.min.js"></script>
    <!-- datepicker -->
    <link rel="stylesheet" href="http://cdnjs.cloudflare.com/ajax/libs/bootstrap-datepicker/1.7.1/css/datepicker3.min.css">
    <script src="http://cdnjs.cloudflare.com/ajax/libs/bootstrap-datepicker/1.7.1/js/bootstrap-datepicker.min.js"></script>
    <!-- ATTENTION /\ -->

    <script src="js/compiled/app.js"></script>
    <script>bootstrap_datepicker.core.main();</script>
  </body>
</html>
```

#### Step 3: Add text input to `home-render`

Navigate to `src/cljs/bootstrap_datepicker/core.cljs`.

```clojure
(defn home-render []
  [:input {:type "text" :placeholder "click to show datepicker"}])
```

#### Step 4: Convert javascript to clojurescript and put inside a *did-mount* function called `home-did-mount`

This is the javascript we need.

```javascript
$(document).ready(function () {
    $('#example1').datepicker({
        format: "dd/mm/yyyy"
    });
});
```

Let's convert this to clojurescript and place in `home-did-mount`

```clojure
(defn home-did-mount []
  (.ready (js/$ js/document)
          (fn [] (.datepicker (js/$ "#example1") (clj->js {:format "dd/mm/yyyy"})))))
```

The `.ready` method is used to assure that the DOM node exists on the page before executing the `.datepicker` method. However, since we are tapping into the did-mount lifecycle of the component, we are already assured that the component will exist. In addition, rather than using jQuery to select the element by id, we can get the DOM node directly using React/Reagent.  Let's refactor the code as follows:

```clojure
(defn home-did-mount [this]
  (.datepicker (js/$ (reagent/dom-node this)) (clj->js {:format "dd/mm/yyyy"})))
```

#### Step 5: Use `home-render` and `home-did-mount` to create a reagent component called `home`

```clojure
(defn home []
  (reagent/create-class {:reagent-render home-render
                         :component-did-mount home-did-mount}))
```

#### Step 6: Add externs

For advanced compilation, we need to protect `$.datepicker` from getting renamed. Add an `externs.js` file.

```js
var $ = function(){};
$.datepicker = function(){};
```

Open `project.clj` and add a reference to the externs in the cljsbuild portion.

```clojure
:externs ["externs.js"]
```

# Usage

Compile cljs files.

```
$ lein clean
$ lein cljsbuild once prod
```

Open `resources/public/index.html`.
