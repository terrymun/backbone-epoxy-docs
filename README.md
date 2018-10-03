# Backbone.Epoxy documentation
Archived version of Backbone.Epoxy's documentation, original source at https://codepen.io/anon/pen/aYMxJM. HTML to Markdown conversion done using [Turndown](http://domchristie.github.io/turndown/) with manual clean up of internal links and anchors.

## Table of contents

| [Epoxy.Model](#epoxymodel)        | [Epoxy.View](#epoxyview)              | [View Binding Handlers](#view-binding-handlers)   | [View Binding Filters](#view-binding-filters) | [Epoxy.binding](#epoxybinding)    |
|---------------------------------- |-------------------------------------- |-------------------------------------------------- |---------------------------------------------- |---------------------------------- |
| [extend](#extend)                 | [extend](#extend-1)                   | [attr](#attr)                                     | [all](#all)                                   | [addFilter](#addFilter)           |
| [mixin](#mixin)                   | [mixin](#mixin-1)                     | [checked](#checked)                               | [any](#any)                                   | [addHandler](#addHandler)         |
| [constructor](#constructor)       | [constructor](#constructor-1)         | [classes](#classes)                               | [csv](#csv)                                   | [allowedParams](#allowedParams)   |
| [addComputed](#addComputed)       | [binding context](#binding-context)   | [collection](#collection)                         | [decimal](#decimal)                           | [config](#config)                 |
| [clearComputeds](#clearComputeds) | [applyBindings](#applyBindings)       | [css](#css)                                       | [format](#format)                             | [emptyCache](#emptyCache)         |
| [computeds](#computeds)           | [bindingFilters](#bindingFilters)     | [disabled](#disabled)                             | [integer](#integer)                           |                                   |
| [destroy](#destroy)               | [bindingHandlers](#bindingHandlers)   | [enabled](#enabled)                               | [length](#length)                             |                                   |
| [get](#get)                       | [bindings](#bindings)                 | [events](#events)                                 | [none](#none)                                 |                                   |
| [getCopy](#getCopy)               | [bindingSources](#bindingSources)     | [html](#html)                                     | [not](#not)                                   |                                   |
| [hasComputed](#hasComputed)       | [computeds](#computeds-1)             | [itemView](#itemView)                             | [select](#select)                             |                                   |
| [initComputeds](#initComputeds)   | [getBinding](#getBinding)             | [options](#options)                               |                                               |                                   |
| [modifyArray](#modifyArray)       | [remove](#remove)                     | [optionsDefault](#optionsDefault)                 |                                               |                                   |
| [modifyObject](#modifyObject)     | [removeBindings](#removeBindings)     | [optionsEmpty](#optionsEmpty)                     |                                               |                                   |
| [removeComputed](#removeComputed) | [setBinding](#setBinding)             | [template](#template)                             |                                               |                                   |
| [set](#set)                       | [setterOptions](#setterOptions)       | [text](#text)                                     |                                               |                                   |
| [toJSON](#toJSON)                 | [viewModel](#viewModel)               | [toggle](#toggle)                                 |                                               |                                   |
|                                   |                                       | [value](#value)                                   |                                               |                                   |

## API

- Epoxy.Model
- Epoxy.View
- Epoxy.Binding

## View bindings

- View Binding Handlers
- View Binding Filters



***

## Epoxy.View

The Epoxy View object extends Backbone.View, providing a new view abstract to be extended or mixed into your application.

The Epoxy View provides elegant and extensible hooks for binding data sources (Backbone.Model and/or Backbone.Collection instances) directly to DOM elements. When data attributes change, they will automatically trigger refresh of all bound view elements; seamlessly updating the view without requiring a complete render pass.

Also note that an Epoxy view may bind to a native Backbone.Model instance, or to an Epoxy.Model with specialized computed attributes. Because both model types operate with a common API, the Epoxy View does not depend on its Epoxy.Model counterpart to support model bindings. Therefore, you're free to exclude Epoxy.Model resources from your project if you just want view binding features.

### extend

`Backbone.Epoxy.View.extend(properties, [classProperties])`

Extends the Epoxy.View constructor for use in your own implementation.

### mixin

`Backbone.Epoxy.View.mixin([target])`

Generates an abstract of Epoxy.View methods for mixin with your View classes.

```js
_.extend( MyView.prototype, Backbone.Epoxy.View.mixin() );
// or:
Backbone.Epoxy.View.mixin( MyView.prototype );
```

When initializing your view instances, you'll need to manually resolve the Epoxy constructor options and then call **applyBindings** to reproduce Epoxy view setup.

### constructor

`new Backbone.Epoxy.View([options])`

Creates a new Epoxy.View instance. The following attributes may be passed as options for direct assignment to the view instance: viewModel, bindings, bindingFilters, bindingHandlers, bindingSources, and/or computeds.

### binding context

`{$model:model, $collection:collection, [*attributes], [*computeds]}`

To establish bindings, an Epoxy view generates a _binding context_ with routers mapping all available data sources, model attributes, and computed properties of the view. This compiled binding context serves as a router table for exchanging values between data sources and the view. Live values may be read and written through the binding context using the view's **[getBinding](#getbinding)** and **[setBinding](#setbinding)** methods.

As noted above, a binding context includes three sets of resources:

*   **Data Sources**: these are instances of Backbone.Model and/or Backbone.Collection that provide themselves and their attributes to the view for binding. By default, an Epoxy view looks for three potential data sources: the view's **model** property, **viewModel** property, and its **collection** property. References to these data sources are automatically configured in the binding context as $model, $viewModel, and $collection. Additional models and collections may be added through the **[bindingSources](#bindingSources)** hash, and will be added to the context as "$sourceName".
*   **Model Attributes**: all Backbone.Model data sources will have their model attributes added to the binding context as well. For the view's primary **model** and **viewModel** properties, all attributes will be added using their normal names (ie: "attribute"). For additional models defined through **bindingSources**, each model's attributes will be aliased as "source\_attribute". Note that instances of Epoxy.Model will provide aliases to ALL attributes, including their computeds.
*   **Computed View Properties**: all computed property functions defined in the view's computeds hash are the final values added into the binding context.

Be mindful of naming conflicts within the binding context. Attributes of the **model**, **viewModel**, and view **computeds** are all aliased with their normal names. Therefore, specificity in naming is strongly encouraged.

### applyBindings

`view.applyBindings()`

Removes any existing view bindings, then applies all bindings defined within the view's **bindings** object. All bindings are established using the view's **$el** as the display target, and the view's **model**, **collection**, and/or **bindingSources** properties as the data sources (see **[binding context](#binding-context)** for more information). A view with invalid display or data targets will apply no bindings. If the **bindings** object is a string, that string will be used as an attribute selector query for extracting bindings from the view's **$el** container.

The **applyBindings** method is automatically called by the view constructor _after_ the view's **initialize** method runs. This allows the **initialize** step to make adjustments within the view prior to bindings being applied. You generally shouldn't need to call **applyBindings** manually.

Also note that view bindings are baked at the time they are applied. All relationships between data sources and the view are established during this binding step and can only be reconfigured by unbinding and rebinding the view.

### bindingFilters

`view.bindingFilters`

A hash table defining a view's custom binding filters. Binding filters are used to format model data for display within the view. While Epoxy includes a core set of basic [binding filters](#view-binding-filters) for common formatting operations; developers are also encouraged to extend their views with custom binding filters as needed.

Filters are added with a name string and a filter function or methods object. All filter arguments are pre-processed, so you may safely use conditional logic in filters without breaking automated dependency maps. All filtering functions are called anonymously and operate in global scope.

A **one-way** filter is defined with a name and a getter function. This getter function requests one or more values from the model, formats them, and then returns the result to the bound view:

```js
var BindingView = Backbone.Epoxy.View.extend({
    bindingFilters: {
        yourName: function( firstName, lastName ) {
            return "Your name is: "+ firstName +" "+ lastName;
        }
    }
});
```

The above filter definition would be bound as:

```html
<div data-bind="text:yourName(firstName,lastName)"></div>
```

A **two-way** filter is defined with a methods object that provides filtering functions for getting/setting a value. A two-way filter may only apply to a single bound model value. The two-way getter function passes a formatted value from the model to the view, while the setter function passes the counter-formatted value from the view to the model:

```js
var BindingView = Backbone.Epoxy.View.extend({
    bindingFilters: {
    	myFilter: {
            get: function( value ) {
                // Format the value as it passes from the model to the view...
                return value;
            },
            set: function( value ) {
                // Format the value as it passes from the view to the model...
                return value;
            }
    	}
    }
});
```

### bindingHandlers

`view.bindingHandlers`

A hash table defining a view's custom binding handlers. Binding handlers are used to apply binding data to elements in the view. While Epoxy includes a core set of basic [binding handlers](#view-binding-handlers) for managing an element's content and formatting, developers are encouraged to extend their views with customized binding handlers as needed.

To declare binding handlers, define a new property on the **bindingHandlers** hash with a params object defining **set** and **get** function properties:


```js
var BindingView = Backbone.Epoxy.View.extend({
    bindingHandlers: {
        printList: {
            set: function( $element, value ) {
                $element.text( value.join(",") );
            },
            get: function( $element, value, event ) {
                return $element.text().split(",");
            }
        }
    }
});
```

*   **set:** _Required function, invoked in context of the binding_. This function is used to WRITE data from the model into the view. The **set** function receives two arguments: a jQuery representation of the bound element, and the model attribute value to be written into the view. The **set** function returns nothing.
*   **\[get\]:** _Optional function, invoked in context of the binding_. This function is used to READ data from the view to be sent to the model. The **get** function receives three arguments: a jQuery representation of the bound element, the current model value that will be changed (useful for comparison purposes), and the event (if any) that has triggered the getter to run. The **get** function should return a value to replace the current bound model attribute. When modifying Object and Array values, be sure to make changes to a _copy_ of the current value, then submit that back to the model.

While declaring a display-only binding handler (one-way binding), you may declare its setter function directly on the **bindingHandlers** hash:


```js
var BindingView = Backbone.Epoxy.View.extend({
    bindingHandlers: {
        printList: function( $element, value ) {
            $element.text( value.join(",") );
        }
    }
});
```

### bindings

`view.bindings`

A hash declaring the view's element-to-attribute binding scheme, or a string defining an element attribute to query from the view's DOM. All bindings are established using the view's **$el** reference as the display target, and the view's [binding context](#binding-context) as the data provider. A view missing its **$el** property and/or valid binding data will apply no bindings.

The bindings hash syntax keeps all binding declarations within the view. The **bindings** object declares a set of key/value pairs, where the key defines an element query, and the value defines the element's model bindings. A custom ":el" pseudo-selector may be used for the view's main **$el** member:


```js
var BindingView = Backbone.Epoxy.View.extend({
    model: new Backbone.Model({firstName:"Luke", lastName:"Skywalker"}),
    bindings: {
        ":el": "toggle:true",
        "input.first-name": "value:firstName,events:['keyup']",
        "input.last-name": "value:lastName,events:['keyup']"
    }
});
```

Alternatively, you may choose to follow the popular convention of declaring bindings directly within the DOM using element attributes, at which time the **bindings** property should define the attribute name to query for. Epoxy specifies "data-bind" by default. All philosophical opinions aside, this technique works just as well as view-based declarations. Getting into philosophy, this method creates a bizarre (albeit handy) relationship between JavaScript mechanics and raw display. Ultimately, the choice is yours as to which method works best for your purposes.

View for attribute binding declarations:

```js
var BindingView = Backbone.Epoxy.View.extend({
    el: "#binding-view"
    model: new Backbone.Model({firstName:"Luke", lastName:"Skywalker"}),
    bindings: "data-bind"
});
```

DOM for attribute binding declarations:

```html
<div id="binding-view">
    <p>First: <input type="text" data-bind="value:firstName,events:['keyup']"></p>
    <p>Last: <input type="text" data-bind="value:lastName,events:['keyup']"></p>
</div>
```

### bindingSources

`view.bindingSources`

A hash table defining additional binding sources to be added to the view's binding context. Binding sources are instances of Backbone.Model and/or Backbone.Collection. By default, an Epoxy view configures two potential binding sources: the view's **model** and **collection** properties are automatically added to the binding context under the aliases $model and $collection. Additional model and collection data sources may be specified within the **bindingSources** hash; these additional data sources will be added into the binding context under the alias $sourceName.

Additionally, model data sources provide aliases to all of their attributes within the binding context. In the case of the default $model data source, all model attributes are aliased within the binding context using their normal attribute name. For additional model sources added through the **bindingSources** hash, model attributes are added under the alias source\_attribute. This avoids attribute naming conflicts while binding multiple models of the same type.


```js
// Epoxy view specifying five data sources:
var BindingView = Backbone.Epoxy.View.extend({
    model: new Backbone.Model({name: "Luke Skywalker"}),
    collection: new Backbone.Collection(),
    bindingSources: {
        han: function() { return new Backbone.Model({name: "Han Solo"}) },
        obiwan: function() { return new Backbone.Model({name: "Obi-Wan Kenobi"}) },
        users: function() { new Backbone.Collection() }
    }
});

// Resulting binding context (psudocode):
{
    $model: <Backbone.Model>,
    $collection: <Backbone.Collection>,
    $han: <Backbone.Model>,
    $obiwan: <Backbone.Model>,
    $users: <Backbone.Collection>,
    name: <$model.name>,
    han_name: <$han.name>,
    obiwan_name: <$obiwan.name>
}
```

A note about Collection sources: Epoxy bindings do not register a "change" event on collection sources to avoid generally superfluous updates. Instead, you may manually trigger an "update" event on a collection source to refresh its bindings.

### computeds

`view.computeds`

A hash table defining a view's computed properties. Computed view properties act as custom data routers: they assemble custom values from data available in the [binding context](#binding-context), and route formatted values. While this is conceptually similar to computed model attributes, computed view properties are unique in that they do not store their own data, nor do they trigger any events; they simply act as pass-throughs for modifying context data using the view's getBinding and setBinding API methods. Computed view properties are an excellent place to assemble view-specific display values and/or markup.

Computed view properties may be declared with a [computed params](#computed-params) object, or as a getter function (uses [automatic dependency mapping](#automatic-dependency-mapping)). Computed view functions are called in the context of the view, so referencing this refers to the parent view.

```js
var MyView = Backbone.Epoxy.View.extend({
    el: "#my-view",
    computeds: {
        priceParams: {
            deps: ["price"],
            get: function( price ) {
                return "$"+ price;
            },
            set: function( value ) {
                this.setBinding("price", value.replace("$", ""));
            }
        },
        welcomeGetter: function() {
            var name = this.getBinding("firstName");
            return name ? "Welcome back, "+name : "Hello!";
        }
    }	
});
```

When declaring a computed view property using a params object, the following options may be provided:

*   **get:** _Required function; invoked in context of the view_. This getter function assembles its value based on other data in the view context, then returns the generated value. If a deps option is also specified, then the dependency values are mapped and injected as arguments into the getter function. If dependencies are not manually specified, then the **getBinding** method must be used for data access (for [automatic dependency mapping](#automatic-dependency-mapping-with-computed-views)).
*   **\[deps\]:** _Optional array_. An array of context attribute names that the getter function depends on. When provided, these dependency values will be mapped and injected as arguments into the getter function.
*   **\[set\]:** _Optional function; invoked in context of the view_. A function that receives an input value for storage. The **set** method provides no protocol for data storage – storage is entirely dependent on your setter implementation. Use **setBinding** to write data into view sources. Computed view properties declared without a **set** function are read-only.

#### Automatic dependency mapping with computed views

Like computed model attributes, computed view properties must map dependencies between view bindings and their underlying data values to keep the display in sync. You may manually declare dependencies using a params object (see above), or use the following guidelines for automatic dependency mapping:

*   When getting and/or setting data within a computed function, always use the getBinding and setBinding view methods to make sure context data is mapped while being accessed. Getting and setting data through other protocols (such as direct model access) will fail to map dependencies.
*   Keep getBinding and setBinding calls outside of conditional logic blocks. Conditional logic may prevent mapping methods from being reached, therefore it's safest to collect mapped values into local variables before performing conditional logic with them.

### getBinding

`view.getBinding(attribute)`

Gets data through the view's [binding context](#view-binding-context). The **getBinding** method may request any data attribute name available to binding declarations. This applies to data sources (referenced as "$source"), model attributes (referenced as "attribute" or "source\_attribute"), and [computed view properties](#computeds-1).

### remove

`view.remove()`

Override wrapper for the Backbone Views's native **remove** method. This override calls **removeBindings**, then defers to the Backbone View's native **remove** method.

### removeBindings

`view.removeBindings()`

Removes and cleans up all bindings applied to the view. All binding hooks will be nullified, leaving the view's DOM elements formatted in their final bound state. The **removeBindings** method is automatically called with the **remove** method. Be mindful that you should manually call **removeBindings** when deprecating an Epoxy view without specifically calling its **remove** method.

### setBinding

`view.setBinding(attribute, value)`

Sets data through the view's [binding context](#view-binding-context). The **setBinding** method may submit data to any model attribute name available to binding declarations. The submitted data value will pass through binding context routers for storage in the connected data model.

### setterOptions

`view.setterOptions`

Specifies an options object to be passed into all bound setters within the view. This equates to calling model.set({attrs}, {options}) in two-way bindings. Additionally, you may define an Epoxy-specific {save:true} option to have bound setters write to the model via the **save** method rather than with **set** (**setterOptions** will also be passed into the **save** method).

### viewModel

`view.viewModel`

References a model for storing view-specific data. This may be an instance of Backbone.Model or Epoxy.Model, and may be passed into the view through constructor options. A view model instance and all of its attributes will be added to the view's [binding context](#view-binding-context) for use in bindings. While there is nothing exceptionally unique about this model instance, it offers a compositional benefit for organizing stateful and operational data within a view structure.

***

## View Binding Handlers

Binding handlers are used to interchange model data with elements in the view. The Epoxy View includes default binding handlers for common tasks such as managing a bound element's content, class, and style. However, don't feel limited to just Epoxy's out-of-the-box binding handlers. You may easily [define your own binding handlers](#view-binding-handlers) within a view.

One or more binding handlers may be applied to an element as a comma-separated properties list (here shown using DOM attribute bindings):


```html
<a data-bind="text:linkText,attr:{href:linkUrl,title:linkText}"></a>
```

In the above example, "text:" and "attr:" are binding handlers, while "linkText" and "linkUrl" are the bound model attributes assigned to them. In the case of the "attr" handler, its value is a hash of attribute names, each with an associated model value.

Be aware that binding declarations are parsed as JavaScript Object bodies, therefore a binding declaration _must_ conform to proper JavaScript syntax. To give due credit, Epoxy's binding system is modeled after the clever technique used in [Knockout.js](http://knockoutjs.com/). However, Epoxy deliberately limits some of Knockout's free-form binding allowances, such as inline concatenation (which becomes overly technical within the presentation layer, IMHO). If you're coming from Knockout, don't expect to write as much inline binding JavaScript.

### attr

read-only `data-bind="attr:{name:modelAttribute,'data-name':modelAttribute}"`

Binds model attribute values to a list of element attributes. Attribute bindings are defined as a hash of key/value pairs, where each key defines an element attribute and the model reference defines the attribute value. Attribute keys may be defined as strings.

### checked

read-write `data-bind="checked:modelAttribute"`

Binds checkbox and radio elements to the model. The binding behavior differs slightly for various element/value combinations:

*   **Radios:** a bound radio element will become checked when the element's value matches the bound model attribute. Checking a radio will set its value to the bound model attribute.
*   **Checkbox arrays:** when a checkbox element is bound to an Array model attribute, the element will be checked while its value exists within the value array, and unchecked while its value is absent from the array. Likewise, toggling the element's checked property will add and remove its value from the bound array. Keep in mind that arrays modified within a native Backbone.Model will NOT trigger a "change" event unless the array _instance_ is changed. Using an Epoxy.Model computed will help here by managing object copies, and by performing array comparisons by content.
*   **Checkbox toggles:** when a checkbox element is bound to a primitive model attribute (types String, Boolean, or Number), the element will be checked based on a loosely-typed assessment of the model attribute's truthiness. When the element's "checked" status is toggled, it will submit a Boolean value back to the model attribute.

### classes

read-only `data-bind="classes:{active:modelAttr,'is-active':modelAttr}"`

Toggles a list of element class names based on the truthiness of their bound model attributes. Class bindings are defined as a hash of key/value pairs, where each key defines a class name and is value references a model attribute that will be loosely-type checked for truthiness; truthy attributes will enable their class, while falsey attributes will disable their class. Class name keys may be defined as strings.

### collection

read-only `data-bind="collection:$collectionSource"`

Manages the display of a Backbone.Collection. The bound collection must specify a Backbone.View constructor for displaying its models, at which time the **collection** handler will manage adding, removing, resorting, and resetting a list of those views to synchronize the display of the collection's contents. The **collection** binding performs discrete view management, where collection models will be assigned a single view instance for their lifespans, rather than re-rendering all collection views in response to changes. You may choose to use an Epoxy.View constructor for the display of individual collection items, allowing each collection view item to bind to its respective model.

The **collection** binding requires the following setup:

*   The binding must target a Backbone.Collection data source available within the **[binding context](#view-binding-context)**.
*   The bound view must provide a property called **itemView**, which defines a Backbone.View constructor for rendering collection items (use an [itemView](#handler-item-view) binding to access a different view property). This item will receive a collectionView option when constructed that references the parent view.

```js
// View for individual collection items:
var ListItemView = Backbone.View.extend({
    tagName: "li",
    initialize: function() {
        this.$el.text( this.model.get("label") );
    }
});

// Collection defining a Model and View:
var ListCollection = Backbone.Collection.extend({
    model: Backbone.Model
});

// Binding view for list of collection items:
var ListView = Backbone.Epoxy.View.extend({
    el: "<ul data-bind='collection:$collection'></ul>",
    itemView: ListItemView,
    initialize: function() {
        this.collection = new ListCollection();
        this.collection.reset([{label: "Luke Skywalker"}, {label: "Han Solo"}]);
    }
});

var view = new ListView();
```

Note that the **collection** binding does not register a "change" event on its collection to avoid generally superfluous updates. Instead, you may manually trigger an "update" event on the collection to refresh its bindings. For a working demonstration of the **collection** binding, see the [Epoxy ToDos](tutorials.html#epoxy-todos) demo.

### css

read-only `data-bind="css:{color:modelAttribute,'font-size':modelAttribute}"`

Binds a list of CSS styles to model attributes. CSS style bindings are defined as a hash of key/value pairs, where each key defines a CSS style name and the model attribute defines the style's literal value. CSS keys may be defined as strings.

### disabled

read-only `data-bind="disabled:modelAttribute"`

Toggles a form element's "disabled" property based on a loosely-typed assessment of the bound model attribute's truthiness. A truthy value disables the element (inversion of the **enabled** handler).

### enabled

read-only `data-bind="enabled:modelAttribute"`

Toggles a form element's "disabled" status based on a loosely-typed assessment of the bound model attribute's truthiness. A truthy value enables the element (inversion of the **disabled** handler).

### events

read-only `data-bind="events:['keydown','focus']"`

The **events** handler is a special binding used to parameterize what DOM events will trigger changes for read-write element bindings. By default, all read-write bindings will subscribe to their bound element's "change" event to trigger model updates. You may bind additional DOM event triggers for the element by listing them in the **events** array.

### html

read-only `data-bind="html:modelAttribute"`

Sets the element's HTML content to the bound model attribute value. Uses the jQuery **html** method.

### itemView

read-only `data-bind="collection:$collection,itemView:'myView'"`

The **itemView** handler is a special binding used to parameterize what view is used to render **collection** binding items. By default, a **collection** binding will access its parent view for a property called "itemView". Use the **itemView** binding to specify a different attribute of the parent view. This may be useful when binding multiple data sources.

### options

read-only `data-bind="options:arraySource"`

Binds a <select> element's options to an array data source. The bound data source may be an Array object, or a Backbone.Collection instance from which the **models** array will be used. Array contents may be formatted using one of the following methods:

*   \["Luke", "Han", "Leia"\] : Primitives list. Given an array of primitives, options will be created using the array values as both the text and value for each option.
*   \[{label:"Luke", value:"1"}, {label:"Leia", value:"2"}\] : Label/value pairs. Given an array of objects, options will be created using each object's **label** and **value** properties as the text and value of each option. These property names may be adjusted globally through [binding config](#binding-config).
*   Backbone.Collection : Models list. The collection's **models** array will be used as the array source. Options will be created using each model's **label** and **value** attributes as the text and value of each option. These attribute names may be adjusted globally through [binding config](#binding-config). An Epoxy.Model may be used to virtualize these values as computed/computed attributes if needed. Note that the **options** binding does not register a "change" event on its collection to avoid generally superfluous updates. Instead, you may manually trigger an "update" event on the collection to refresh its bindings.

```js
defaults: {
    list1: ["Luke"],
    list2: [{label:"Luke", value:"0"}],
    list3: new Backbone.Collection( {models:[{label:"Luke", value:"0"}]} )
}

// list1: <option value='Luke'>Luke</option>
// list2: <option value='0'>Luke</option>
// list3: <option value='0'>Luke</option>

```

To manage a <select> element's default first option and/or empty state, you may choose to also include the **optionsDefault** and/or **optionsEmpty** helper bindings.

### optionsDefault

read-only `data-bind="options:opts,optionsDefault:modelAttribute"`

Defines a default option made available at the top of a <select> element in addition to the **options** list. This binding must be used in tandem with the **options** binding. The **optionsDefault** value will appear as the first and/or only item in the select, regardless if the **options** list has content. The **optionsDefault** value may be formatted using one of the following methods:

*   "Default Value" : Given a primitive value (string, number, or boolean), the default option will be created using the primitive as both text and value of the option.
*   {label:"Default", value:""} : Given an object, the default option will be created using the object's **label** and **value** properties as the text and value of the option.
*   Backbone.Model : Given a Backbone Model, the default option will be created using the model's **label** and **value** attributes as the text and value of the option. An Epoxy.Model may be used to virtualize these as computed/computed attributes if needed.

```js
defaults: {
    default1: "Choose...",
    default2: {label:"Choose...", value:""},
    default3: new Backbone.Model({label:"Choose...", value:""})
}

// default1: <option value='Choose...'>Choose...</option>
// default2: <option value=''>Choose...</option>
// default3: <option value=''>Choose...</option>

```

You may also choose to declare **optionsDefault** as a static object within a binding declaration, as in:  
data-bind="value:val,options:opts,optionsDefault:{label:'Choose...', value:''}"

### optionsEmpty

read-only `data-bind="options:opts,optionsEmpty:modelAttribute"`

Defines a placeholder option to apply to an empty <select> element, and automatically disables the select element while displaying this placeholder. This must be used in tandem with the **options** binding, and is generally only needed when there is no default option. The empty placeholder option may be formatted using one of the following methods:

*   "Placeholder Value" : Given a primitive value (string, number, or boolean), the placeholder option will be created using the primitive as both text and value of the option.
*   {label:"None", value:""} : Given an object, the placeholder option will be created using the object's **label** and **value** properties as the text and value of the option.
*   Backbone.Model : Given a Backbone Model, the placeholder option will be created using the model's **label** and **value** attributes as the text and value of the option. An Epoxy.Model may be used to virtualize these as computed/computed attributes if needed.

```js
defaults: {
    empty1: "None",
    empty2: {label:"None", value:""},
    empty3: new Backbone.Model({label:"None", value:""})
}

// empty1: <option value='None'>None</option>
// empty2: <option value=''>None</option>
// empty3: <option value=''>None</option>

```

You may also choose to declare **optionsEmpty** as a static object within a binding declaration, as in:  
data-bind="value:val,options:opts,optionsEmpty:{label:'none', value:''}"

### template

read-only `data-bind="template:$source"`

The **template** binding extracts its element's HTML content while initializing, and then parses that content into an Underscore template using the bound data. You have some options on how both the template string and the bound data sources are defined.

**Defining the template string**

When the **template** binding initializes, it will extract content from its element to use as the template string. However, to safeguard against malformed HTML within the DOM, the **template** binding will first search its element for a <script> or a <template> tag to extract content from, such as:

```html
<div data-bind="template:$model">
    <script type="text/tmpl"><%= firstName %> <%= lastName %></script>
</div>

<div data-bind="template:$model">
    <template><%= firstName %> <%= lastName %></template>
</div>
```

If a <script> or <template> container is _not_ found within the element, then the element's entire contents are parsed as the template string. If you choose this approach, then it's highly recommended that you modify Underscore's [templateSettings](http://underscorejs.org/#template) param to choose more DOM-friendly field delimiters. Also, to avoid the infamous [FOUC](http://www.bluerobot.com/web/css/fouc.asp/), you're best off hiding the element with CSS and then toggle it's visibility with a binding:

```html
<div style="display:none;" data-bind="template:$model,toggle:true">
    {{ firstName }} {{ lastName }}
</div>
```

**Defining data sources**

The **template** binding may specify data sources in a few different ways. First, let's assume the following example view configuration:

```js
var MyView = Backbone.Epoxy.View.extend({
    el: "#my-view",
    model: new Backbone.Model({
        firstName: "Luke",
        lastName: "Skywalker",
        isActive: false
    });
});
```

*   **Model Source** : given the above MyView example, the **template** binding may reference a Backbone.Model data source directly through its context reference ($sourceName), such as:
    
    ```
    <div id="my-view" data-bind="template:$model">
        <template><%= firstName %> <%= lastName %></template>
    </div>
    ```
    
    When binding to a model source, all model attributes will be available to the template (this is equivalent to providing model.toJSON() to the template renderer). While this is a quick and easy binding method, it may not be the most efficient. The bound template will re-render in response to _all_ model changes, rather than just changes to specific attributes used in the template. In this scenario, our template would re-render in response to isActive changing, even though it's not included within the template.
    
*   **Array Source** : Given the above MyView example, the **template** binding may specify an array of attribute name strings to bind on, such as:
    
    ```html
    <div id="my-view" data-bind="template:['firstName','lastName']">
       <template><%= firstName %> <%= lastName %></template>
    </div>
    ```
    
    When binding on attribute names, only the specified names are made available to the template. Also, our template will only re-render in response to changes in these attributes (in this scenario, changing the isActive attribute will no longer trigger a template refresh).
    
*   **Hash Source** : Given the above MyView example, the **template** binding may define an object hash that maps keys to data attributes, such as:
    
    ```html
    <div id="my-view" data-bind="template:{first:firstName, last:lastName}">
        <template><%= first %> <%= last %></template>
    </div>
    ```
    
    When defining a mapping hash, the declared object keys are the variables available within the template, and their values are bound attributes from the model. This is similar to the array declaration method (which effectively builds this hash mapping for you with matching key/value pairs). Like the array method, the template will only re-render in response to changes in the specifically referenced model attributes.
    

### text

read-write `data-bind="text:modelAttribute"`

Sets the element's text content to the bound model attribute value. Uses the jQuery **text** method.

To enable a writeable binding, add the HTML5 contenteditable property to the element:

```html
<span contenteditable="true" data-bind="text:yourName,events:['blur paste']">
    Select to edit your name.
</span>
```

### toggle

read-only `data-bind="toggle:modelAttribute"`

Toggles the element's display based on a loosely-typed assessment of the bound model attribute's truthiness. Uses the jQuery **toggle( \[boolean\] )** method.

### value

read-write `data-bind="value:modelAttribute"`

Binds an input element's (input, select, textarea) value to an underlying model attribute. The element will set its value to the bound model attribute upon change event triggers (see the **[events](#handler-events)** handler), and update its value when the underlying model attribute changes.

***

## View Binding Filters

Binding filters provide a layer of flexibility for formatting data attributes directly within a binding declaration; they may perform read-only formatting on multiple inputs, or perform read-write formatting on a single input. Filters are intended to format raw values for binding-specific purposes:

```html
<span data-bind="toggle:not(firstName)">Please enter a first name.</span>
```

Binding filters may be parameterized with any data attribute(s) available in the [binding context](#view-binding-context), or with primitive values (String, Number, or Boolean). However, binding filters may **NOT** be nested within one another — this is very deliberate: application logic does not belong within binding declarations. If you need to perform multi-pass value formatting, that work should be done in a computed property, or else applied as a custom binding handler.

See view [bindingFilters](#view-binding-filters) for process on defining your own custom filters.

### all

read-only `data-bind="toggle:all(dataAttribute,[...])"`

Assesses one or more data attributes as true if _all_ attributes are truthy.

### any

read-only `data-bind="toggle:any(dataAttribute,[...])"`

Assesses one or more data attributes as true if _any_ of the attributes are truthy.

### csv

read-write `data-bind="checked:csv(dataList)"`

Exchanges comma-separated values data between models and bindings. A string is parsed into a comma-separated array when read through the **csv** filter, and then is formatted as a comma-separated string when written back through the filter. This is handy for storing a primitive list value within a model, while using it as an array in view bindings. Note that this is only a simple split/join implementation of comma-separated values, so advanced CSV formatting (string closures, etc) will not apply.

### decimal

read-write `data-bind="checked:decimal(floatAttribute)"`

Exchanges a floating-point decimal value between models and bindings. A string is parsed into a float when being read or written through the **decimal** filter. This is handy for preserving float data formatting when binding to input elements.

### format

read-only `data-bind="text:format('$1 of $2',dataAttribute,dataAttribute)"`

Formats a string with a series of data attribute replacements. Operates the same as string replacement for RegExp capture groups, where field insertions are denoted as "$1, $2, ... $n". Escape intentional "$n" patterns within the template string as "\\$n". Note that the first value insertion index is 1, not 0.

### integer

read-write `data-bind="checked:integer(numberAttribute)"`

Exchanges an integer value between models and bindings. A string is parsed into an integer when being read or written through the **integer** filter. This is handy for preserving numeric data formatting when binding to input elements.

### length

read-only `data-bind="toggle:length(dataAttribute)"`

Returns the length of an Array or Collection data attribute, or 0 by default. Useful for loosely-typed assessments of a collection's content status.

### none

read-only `data-bind="toggle:none(dataAttribute,[...])"`

Assesses one or more data attributes as true if _none_ of the attributes are truthy.

### not

read-only `data-bind="toggle:not(dataAttribute)"`

Inverts the loosely-typed boolean value of a data attribute.

### select

read-only `data-bind="text:select(conditionalValue,truthyValue,falseyValue)"`

Performs a ternary operation using JavaScript's ?: operator. The **select** operation receives three arguments: the first argument is assessed as a loosely-typed conditional; if truthy, the second argument is returned; if falsey, the third argument is returned.

***

## Epoxy.binding

`Backbone.Epoxy.binding`

The Epoxy.binding API allows customization of Epoxy's default binding capabilities and configuration of binding settings.

### addFilter

`Backbone.Epoxy.binding.addFilter( name, filter )`

Adds a [binding filter](#binding-filters) to Epoxy's table of internal defaults. Filters added with this method become available to all Epoxy views. Filters are added with a name string and a filter function/methods object. All filter arguments are pre-processed, so you may safely use conditional logic in filters without breaking automated dependency maps.

A **one-way** filter is defined with a name and a getter function. This getter function requests one or more values from the model, formats them, and then returns the result to the bound view:

```js
Backbone.Epoxy.binding.addFilter("select", function(condition, truthy, falsey) {
    return condition ? truthy : falsey;
});
```

A **two-way** filter is defined with a methods object that provides filtering functions for getting/setting a value. A two-way filter may only apply to a single bound model value. The two-way getter function passes a formatted value from the model to the view, while the setter function passes the counter-formatted value from the view to the model:

```js
Backbone.Epoxy.binding.addFilter("filter_name", {
    get: function( value ) {
        // Format the value as it passes from the model to the view...
        return value;
    },
    set: function( value ) {
        // Format the value as it passes from the view to the model...
        return value;
    }
});
```

### addHandler

`Backbone.Epoxy.binding.addHandler( name, handler )`

Adds a [binding handler](#binding-handlers) to Epoxy's table of internal defaults. Handlers added with this method become available to all Epoxy views. Binding handlers are added with a name string and a handler function or methods object. A simple one-way binding handler (writes to the DOM) may be defined as:

```js
Backbone.Epoxy.binding.addHandler("readonly", function( $element, value ) {
    $element.prop( "readonly", !!value );
});
```

Binding handlers may also be defined as a methods object that provides functions for getting/setting the element value, and initialization/cleanup of the handler:

```js
Backbone.Epoxy.binding.addHandler("handler_name", {
    init: function( $element, value, bindings, context ) {
        // Initialize the binding handler...
    },
    get: function( $element, value, event ) {
        // Get data from the bound element...
        return $element.val();
    },
    set: function( $element, value ) {
        // Set data into the bound element...
        $element.val( value );
    },
    clean: function() {
        // Cleanup the binding handler...
    }
});
```

*   handler.init( $element, value, bindings, context ) : _optional_. A handler's **init** method is called immediately to configure the handler. This is generally only needed for complex handlers that store operational data on themselves. The method receives the bound element (as jQuery) and the bound data value as arguments, as well as the hash of parsed bindings being applied to the element, and a reference to the full binding context of all available view data. The bindings and context arguments are handy for pulling additional data into the handler if needed. The **init** method may also return a reformatted version of its managed value, although this is only recommended for developers with insight into Epoxy internals. If you're not sure, don't return anything.
*   handler.get( $element, value, event ) : _required for two-way bindings_. Reads and returns the bound element's value. This should poll data from the DOM element, and then return the value formatted for the model. The existing model value is provided for comparative reference, as well as the event (if any) that has triggered the getter to run.
*   handler.set( $element, value ) : _required_. Writes a data value into the DOM element. Receives the element (as jQuery) and the value to set, and returns nothing.
*   handler.clean() : _optional_. Called during handler disposal; offers a cleanup hook to remove any custom binding configuration as the handler is deprecated.

### allowedParams

`Backbone.Epoxy.binding.allowedParams`

A hash defining all non-handler attributes that are allowed within binding declarations. When validating bindings, Epoxy will throw an error for binding declarations that do not have a handler method or an allowedParams key. By default, allowedParams defines the following allowed keys:

*   events
*   optionsDefault
*   optionsEmpty

If you define a custom binding handler that utilizes additional params within the binding declaration, then you must specifically add these additional parameter names into the allowedParams hash.

```
Epoxy.binding.allowedParams.myCustomParam = true;
```

In the above example, Epoxy will no longer throw an error when it encounters a myCustomParam definition within a binding declaration.

### config

`Backbone.Epoxy.binding.config( settings )`

Merges a settings object into the bindings API configuration. The following settings may be passed:

*   optionText : configures the **options** binding. Specifies an attribute name to pick from objects and/or models as the text written into option elements. Default is "label".
*   optionValue : configures the **options** binding. Specifies an attribute name to pick from objects and/or models as the value written into option elements. Default is "value".

```js
Backbone.Epoxy.binding.config({
    optionText: "optText",
    optionValue: "optValue"
});
```

### emptyCache

`Backbone.Epoxy.binding.emptyCache()`

Resets the internal hash table of cached binding parser functions. Cached parser functions are highly efficient for parsing redundant binding schemes (as is: one binding scheme is instanced repeatedly for multiple views). You should rarely (or never) need to empty Epoxy's internal cache. The only circumstance where you may want to consider emptying the cache is on long-running single page applications where numerous different binding schemes are encountered sporadically over time, and may never be reused.
