# Epoxy.View

## Quick links

* [extend](#extend)
* [mixin](#mixin)
* [constructor](#constructor)
* [binding context](#binding-context)
* [applyBindings](#applybindings)
* [bindingFilters](#bindingfilters)
* [bindingHandlers](#bindinghandlers)
* [bindings](#bindings)
* [bindingSources](#bindingsources)
* [computeds](#computeds)
* [getBinding](#getbinding)
* [remove](#remove)
* [removeBindings](#removebindings)
* [setBinding](#setbinding)
* [setterOptions](#setteroptions)
* [viewModel](#viewmodel)

## Overview

The Epoxy View object extends Backbone.View, providing a new view abstract to be extended or mixed into your application.

The Epoxy View provides elegant and extensible hooks for binding data sources (Backbone.Model and/or Backbone.Collection instances) directly to DOM elements. When data attributes change, they will automatically trigger refresh of all bound view elements; seamlessly updating the view without requiring a complete render pass.

Also note that an Epoxy view may bind to a native Backbone.Model instance, or to an Epoxy.Model with specialized computed attributes. Because both model types operate with a common API, the Epoxy View does not depend on its Epoxy.Model counterpart to support model bindings. Therefore, you're free to exclude Epoxy.Model resources from your project if you just want view binding features.

## extend

`Backbone.Epoxy.View.extend(properties, [classProperties])`

Extends the Epoxy.View constructor for use in your own implementation.

## mixin

`Backbone.Epoxy.View.mixin([target])`

Generates an abstract of Epoxy.View methods for mixin with your View classes.

```js
_.extend( MyView.prototype, Backbone.Epoxy.View.mixin() );
// or:
Backbone.Epoxy.View.mixin( MyView.prototype );
```

When initializing your view instances, you'll need to manually resolve the Epoxy constructor options and then call **applyBindings** to reproduce Epoxy view setup.

## constructor

`new Backbone.Epoxy.View([options])`

Creates a new Epoxy.View instance. The following attributes may be passed as options for direct assignment to the view instance: viewModel, bindings, bindingFilters, bindingHandlers, bindingSources, and/or computeds.

## binding context

`{$model:model, $collection:collection, [*attributes], [*computeds]}`

To establish bindings, an Epoxy view generates a _binding context_ with routers mapping all available data sources, model attributes, and computed properties of the view. This compiled binding context serves as a router table for exchanging values between data sources and the view. Live values may be read and written through the binding context using the view's **[getBinding](#getbinding)** and **[setBinding](#setbinding)** methods.

As noted above, a binding context includes three sets of resources:

*   **Data Sources**: these are instances of Backbone.Model and/or Backbone.Collection that provide themselves and their attributes to the view for binding. By default, an Epoxy view looks for three potential data sources: the view's **model** property, **viewModel** property, and its **collection** property. References to these data sources are automatically configured in the binding context as $model, $viewModel, and $collection. Additional models and collections may be added through the **[bindingSources](#bindingSources)** hash, and will be added to the context as "$sourceName".
*   **Model Attributes**: all Backbone.Model data sources will have their model attributes added to the binding context as well. For the view's primary **model** and **viewModel** properties, all attributes will be added using their normal names (ie: "attribute"). For additional models defined through **bindingSources**, each model's attributes will be aliased as "source\_attribute". Note that instances of Epoxy.Model will provide aliases to ALL attributes, including their computeds.
*   **Computed View Properties**: all computed property functions defined in the view's computeds hash are the final values added into the binding context.

Be mindful of naming conflicts within the binding context. Attributes of the **model**, **viewModel**, and view **computeds** are all aliased with their normal names. Therefore, specificity in naming is strongly encouraged.

## applyBindings

`view.applyBindings()`

Removes any existing view bindings, then applies all bindings defined within the view's **bindings** object. All bindings are established using the view's **$el** as the display target, and the view's **model**, **collection**, and/or **bindingSources** properties as the data sources (see **[binding context](#binding-context)** for more information). A view with invalid display or data targets will apply no bindings. If the **bindings** object is a string, that string will be used as an attribute selector query for extracting bindings from the view's **$el** container.

The **applyBindings** method is automatically called by the view constructor _after_ the view's **initialize** method runs. This allows the **initialize** step to make adjustments within the view prior to bindings being applied. You generally shouldn't need to call **applyBindings** manually.

Also note that view bindings are baked at the time they are applied. All relationships between data sources and the view are established during this binding step and can only be reconfigured by unbinding and rebinding the view.

## bindingFilters

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

## bindingHandlers

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

## bindings

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

## bindingSources

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

## computeds

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

### Automatic dependency mapping with computed views

Like computed model attributes, computed view properties must map dependencies between view bindings and their underlying data values to keep the display in sync. You may manually declare dependencies using a params object (see above), or use the following guidelines for automatic dependency mapping:

*   When getting and/or setting data within a computed function, always use the getBinding and setBinding view methods to make sure context data is mapped while being accessed. Getting and setting data through other protocols (such as direct model access) will fail to map dependencies.
*   Keep getBinding and setBinding calls outside of conditional logic blocks. Conditional logic may prevent mapping methods from being reached, therefore it's safest to collect mapped values into local variables before performing conditional logic with them.

## getBinding

`view.getBinding(attribute)`

Gets data through the view's [binding context](#view-binding-context). The **getBinding** method may request any data attribute name available to binding declarations. This applies to data sources (referenced as "$source"), model attributes (referenced as "attribute" or "source\_attribute"), and [computed view properties](#computeds).

## remove

`view.remove()`

Override wrapper for the Backbone Views's native **remove** method. This override calls **removeBindings**, then defers to the Backbone View's native **remove** method.

## removeBindings

`view.removeBindings()`

Removes and cleans up all bindings applied to the view. All binding hooks will be nullified, leaving the view's DOM elements formatted in their final bound state. The **removeBindings** method is automatically called with the **remove** method. Be mindful that you should manually call **removeBindings** when deprecating an Epoxy view without specifically calling its **remove** method.

## setBinding

`view.setBinding(attribute, value)`

Sets data through the view's [binding context](#view-binding-context). The **setBinding** method may submit data to any model attribute name available to binding declarations. The submitted data value will pass through binding context routers for storage in the connected data model.

## setterOptions

`view.setterOptions`

Specifies an options object to be passed into all bound setters within the view. This equates to calling model.set({attrs}, {options}) in two-way bindings. Additionally, you may define an Epoxy-specific {save:true} option to have bound setters write to the model via the **save** method rather than with **set** (**setterOptions** will also be passed into the **save** method).

## viewModel

`view.viewModel`

References a model for storing view-specific data. This may be an instance of Backbone.Model or Epoxy.Model, and may be passed into the view through constructor options. A view model instance and all of its attributes will be added to the view's [binding context](#view-binding-context) for use in bindings. While there is nothing exceptionally unique about this model instance, it offers a compositional benefit for organizing stateful and operational data within a view structure.
