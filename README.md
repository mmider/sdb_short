# Reveal.js-d3 (reveald3)

[Reveal.js](https://github.com/hakimel/reveal.js/) plugin to integrate any javascript-based visualization (from pure [D3](https://d3js.org) visualizations to visualizations made with react-based libraries, like [semiotic](https://emeeks.github.io/semiotic/#/) for example) into HTML slides and trigger transitions/animations fully compatible with the Reveal.js `data-fragment-index` fragements attribute. More generally, you can embed anything that can be rendered in an `.html` file (including embedding of a full website, served locally or remotely). [Check out the live demo](https://gcalmettes.github.io/reveal.js-d3/demo/) (navigate from slide to slide with right/left arrows) and [code of the demo examples](https://github.com/gcalmettes/reveal.js-d3/tree/master/demo).

## Browser compatibility:

The plugin should work with the latest versions of Chrome, Firefox and Safari.

## Principal features:

The development of this plugin has been inspired by [Reveal.js-d3js-plugin](https://github.com/jlegewie/reveal.js-d3js-plugin), but has some major differences:
- the plugin itself is not dependent on `D3` and does not require to add `D3` as a dependencies of Reveal.js
- the visualizations are loaded only when the slide hosting them becomes active (to prevent performance issues).
- the visualizations are removed when the slide is not active anymore (e.g.: navigation to previous/next slide) so the browser is not overloaded by running multiples iframes in the background (this behavior [can be configured](#configuration)).
- this plugin support the insertion of multiple visualizations on the same slide (and even multiple visualizations on the same slide + visualization on the background, if you're into those kind of things).
- the triggering of the transitions for the visualizations is fully compatible with Reveal.js [`data-fragment-index`](https://github.com/hakimel/reveal.js/#fragments) feature.
- Any javascript-based visualizations are supported (slide examples: [D3 visualizations](https://gcalmettes.github.io/reveal.js-d3/demo/#/3), [Semiotic (React-based) visualisations](https://gcalmettes.github.io/reveal.js-d3/demo/#/5), [Vega-lite (declarative javascript) visualizations](https://gcalmettes.github.io/reveal.js-d3/demo/#/6), etc ...).

## Installation

### npm

1. Download and install the package in your project:

```
npm install --save reveald3
```

2. Add the plugin to the dependencies in your presentation, as below:

```javascript
Reveal.initialize({
    // ...

    dependencies: [
        // ...

        { src: 'node_modules/reveald3/reveald3.js' }
    ]
});
```

### Manual

1. Copy the file `reveald3.js` into a local folder, i.e. the `plugin/` folder of your Reveal.js presentation.

2. Add the `reveald3` plugin to the presentation dependencies:

```javascript
Reveal.initialize({
    // ...
    dependencies: [
        // ...
        { src: 'plugin/reveald3.js' },
        // ...
    ]
});
```



## Usage:

### Adding visualization(s) to a slide

To add a visualization to your presentation, simply add a container DOM element (`<div>`, `<span>`, etc ...) with the class `fig-container`, and give it a `data-file` attribute with the path of the html file hosting the javascript-based visualization that you want to embed.

```html
<section>
    <div class="fig-container"
         data-file="d3-fig/eclipses.html"></div> <!-- path to the html file with visualization code -->
</section>
```

To embed more than one visualization, simply create multiple containers:

```html
<section>
    <h3>Two visualizations side by side</h3>
    <div class="row">
        <div class="fig-container col-50" data-file="d3-fig/gooey.html"></div>
        <div class="fig-container col-50" data-file="d3-fig/modular-multiplication.html"></div>
        </div>
</section>
```

You can also embed the visualization in the background of the slide by adding the `fig-container` class directly to the `section` element of your Reveal.js code.

```html
<section class="fig-container"
        data-file="d3-fig/collision-detection.html">
    <h2>Some title</h2>

    <p>some text</p>
</section>
```

### Embedding a website into a slide

You can also directly provide a `url` to embed whatever is served at this `url`. This will work for local (e.g.: `http://localhost/4300`) or remote (any website address) `url`s

```html
<!-- Include a beautiful website directly in your presentation -->
<section class="fig-container" 
         data-file="https://students.brown.edu/seeing-theory/" 
</section>
```


### Advanced styling of the embedded iframe

Direct styling of the iframe your visualization will be embedded in can be controlled via the `data-style` attribute. The format of the styles string provided follows the same pattern you would use to inline-style any DOM element. Any valid `CSS` attribute can be used. Directly styling the iframe could be useful for example if you would like to show only part of a visualization you made, or part of a website you would like to display (in a similar way of how you would use the `viewbox` attribute in an `svg` to define the boundaries of an area to display). In this case, you would play with the `margin-top`/`margin-left` as well as the `height`/`width` of the iframe:

```html
<!-- Include a website directly in your presentation  but don't show the header by removing 100px at the top -->
<section>
  <h3>Website without the header</h3>
  <div  class="fig-container" style="overflow:hidden;" 
        data-file="https://students.brown.edu/seeing-theory/" 
        data-style="height: 500px; margin-top: -100px; width: 650px;">
  </div>
</section>
```

The trick above would work because by default the container hosting your iframe is styled with a `style="overflow: hidden;"`. If you do not want the overflow to be hidden, then you can set the `data-overflow-shown` attribute to `true`. If you use the same code than the example above but add `data-overflow-shown=true`, this will result in adding the top 100px of your visualization (in addition of whatever was shown before) in this particular case, since now the overflow of the container would be shown. See the demo for more information.

```html
<!-- Include a website directly in your presentation and offset the website by 100px up compared to the div -->
<section>
  <h3>Website without the header</h3>
  <div  class="fig-container" style="overflow:hidden;" 
        data-overflow-shown=true
        data-file="https://students.brown.edu/seeing-theory/" 
        data-style="height: 500px; margin-top: -100px; width: 650px;">
  </div>
</section>
```


### Adding and controlling animations/transitions for the visualization(s)

To add transitions to a visualization, simply create a global (`var`) variable in the javascript code of your `html` file with the name `_transitions`, and assign to it an array of javascript objects defining the transitions functions for each fragment steps. Each javascript object in the `_transitions` array has to be defined as at least one obligatory `name:value` pair, and two optional `name:value` pairs can also be declared:
- `transitionForward` (obligatory): defines a function for the transition from the current state to the NEXT state of the visualization
- `transitionBackward` (optional): defines a specific function to be ran for the transition to the PREVIOUS state of the visualization when user navigates back. If no `transitionBackward` is specified, this will take the `transitionForward` value of the previous step by default (note that you'll still have to define a `transitionBackward` function for the first animation of the visualization since there is no previous animation/transition in this case. See [example](https://github.com/gcalmettes/reveal.js-d3/blob/master/demo/d3-fig/rainbow.html) in the [demo](https://github.com/gcalmettes/reveal.js-d3/tree/master/demo/d3-fig) folder). `transitionBackward` can also take a `"none"` value to specify that no animation/function has to be played for the reverse transition.
- `index` (optional): defines the `data-fragment-index` step at which the transition has to be triggered. If no `index` is defined, the transitions will be played in the order they appear in the `_transitions` array, with the first one getting the `0` index (first fragment played).

```
var _transitions = [
    {   transitionForward: () => (function to be run to the next state),
        transitionBackward: () => (function to be run from the next state), // optional
        index: 0 // optional
      }
    ]
```

Example:

```
var _transitions = [
      {
        transitionForward: () => recolorize("yellow"),
        index: 0 // will be run at the data-fragment-index=0 state
      },
      {
        transitionForward: () => recolorize("red"),
        transitionBackward: () => recolorize("blue"), // different behavior on the way back
        index: 3 // will be run at the data-fragment-index=3 state
      },
      {
        transitionForward: () => resize(0.1, 6),
      }, // no index, so will be played at the next data-fragment-index state (4)
    ]
```

For each slide containing at least one visualization, the plugin looks at the number and index of each transition in all the arrays `_transitions` for the different visualization embedded in the slide and automatically creates corresponding fragments (`<span>`) in the container slide if needed. So there is no need to create empty fragments in the Reveal slide.

Note that each index specified is considered intelligently in context of the Reveal.js fragments already present in the slide. For example, if a slide has already 3 fragements (so the `data-fragment-index` steps for the slides are 0, 1, 2) and a transition is set to `index: 14` in the `_transitions` array, the plugin will understand that this transition has to be played at the `data-fragment-index=3` step (and not really at `data-fragment-index=14`, since 12 empty steps doesn't make real sense) and will automatically create a `<span>` with this `data-fragment-index`.

## Configuration

You can configure some options for the behavior of the plugin in your presentation by providing a ```reveald3``` option in the Reveal.js initialization options. Note that all configuration values are optional and will default as specified below.

```javascript
Reveal.initialize({
    // ...

    reveald3: {
        // Specify if the last state of the visualization has to be
        // triggered when navigating back to a slide from a slide further
        // in the deck (i.e. we come back to the slide from the next slide).
        // By default the last fragment transition will be triggered to
        // show the last state of the visualization. This behavior can be
        // discarded (for example if the transition duration is long
        // or is a very resource-intensive task or there is no need to
        // retrieve the last state of the visualization).
        runLastState: true, // true/false, default: true

        // Specifies if iframes (that host the visualization) have to be kept
        // on the slide once the slide is not active anymore (e.g.: navigating 
        // to next slide). If true, the current visualization will be kept 
        // active so that its state will be the one displayed if we navigate 
        // back to the slide. This is false by default, as it can be the source
        // of performance issues if complex visualizations (e.g. force layout)
        // are displayed and kept in the background.
        // Also, see the runLastState option as a simpler less 
        // resource-demanding alternative.
        keepIframe: false // true/false, default: false

        // This will prefix the path attributes of the source html paths with the given path.
        // (by default "src" if set to true or with the specified path if string)
        mapPath: false, // true / false / "spefific/path/as/string", default: false
          
        // If true, will try to locate the file at a fallback url without the mapPath prefix in case no file is found
        // at the stipulated url with mapPath
        tryFallbackURL: false, // true/false, default false
     },

});
```
# sdb_pres
# sdb_pres
# sdb_short
