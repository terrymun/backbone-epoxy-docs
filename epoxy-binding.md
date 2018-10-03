# Epoxy.binding

## Quick links

* [addFilter](#addFilter)
* [addHandler](#addHandler)
* [allowedParams](#allowedParams)
* [config](#config)
* [emptyCache](#emptyCache)

## Overview

`Backbone.Epoxy.binding`

The Epoxy.binding API allows customization of Epoxy's default binding capabilities and configuration of binding settings.

## addFilter

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

## addHandler

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

## allowedParams

`Backbone.Epoxy.binding.allowedParams`

A hash defining all non-handler attributes that are allowed within binding declarations. When validating bindings, Epoxy will throw an error for binding declarations that do not have a handler method or an allowedParams key. By default, allowedParams defines the following allowed keys:

*   events
*   optionsDefault
*   optionsEmpty

If you define a custom binding handler that utilizes additional params within the binding declaration, then you must specifically add these additional parameter names into the allowedParams hash.

```js
Epoxy.binding.allowedParams.myCustomParam = true;
```

In the above example, Epoxy will no longer throw an error when it encounters a myCustomParam definition within a binding declaration.

## config

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

## emptyCache

`Backbone.Epoxy.binding.emptyCache()`

Resets the internal hash table of cached binding parser functions. Cached parser functions are highly efficient for parsing redundant binding schemes (as is: one binding scheme is instanced repeatedly for multiple views). You should rarely (or never) need to empty Epoxy's internal cache. The only circumstance where you may want to consider emptying the cache is on long-running single page applications where numerous different binding schemes are encountered sporadically over time, and may never be reused.
