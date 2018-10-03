# Epoxy.Model

## Quick links

 * [extend](#extend)
 * [mixin](#mixin)
 * [constructor](#constructor)
 * [addComputed](#addcomputed)
 * [clearComputeds](#clearcomputeds)
 * [computeds](#computeds)
 * [destroy](#destroy)
 * [get](#get)
 * [getCopy](#getcopy)
 * [hasComputed](#hascomputed)
 * [initComputeds](#initcomputeds)
 * [modifyArray](#modifyarray)
 * [modifyObject](#modifyobject)
 * [removeComputed](#removecomputed)
 * [set](#set)
 * [toJSON](#tojson)

## Overview

The Epoxy Model object extends `Backbone.Model`, providing a new model abstract to be extended into your application.

The Epoxy model introduces computed attributes on top of Backbone's native model attributes. Computed attributes operate as accessors and mutators, where a computed attribute will **get** an assembled value derived from other model attributes, and will **set** one more more mutated values derived from an input. Computed attributes are virtualized properties of the model: they may be **get** and **set** just like normal model attributes, and will trigger `"change"` events on the model when modified, however they do not exist within the model's **attributes** table, nor will they be saved with model data. Computed model attributes exist only in memory for the lifespan of a model instance. Computed attributes bind themselves to their dependencies and will update themselves in response to any of their dependencies changing, and then in turn trigger their own `"change:attribute"` event on the model.

## extend

`Backbone.Epoxy.Model.extend(properties, [classProperties])`

Extends the Epoxy.Model constructor for use in your own implementation.

## mixin

`Backbone.Epoxy.Model.mixin([target])`

Generates an abstract of Epoxy.Model methods for mixin with your Model classes.

```js
_.extend( MyModel.prototype, Backbone.Epoxy.Model.mixin() );
// or:
Backbone.Epoxy.Model.mixin( MyModel.prototype );
```

When initializing your model instances, you'll need to manually resolve the Epoxy constructor options and then call **initComputeds** to reproduce Epoxy model setup.

## constructor

`new Backbone.Epoxy.Model([attributes], [options])`

Creates a new Epoxy.Model instance. When constructed, computeds may be passed as an options property for direct assignment to the model. A constructed model will automatically initialize all computed property definitions.

## addComputed

  
`model.addComputed(name, params) or...`  
`model.addComputed(name, getter, [setter], [*deps])`

Adds a computed attribute to the model. Computed attributes operate as accessors and mutators, where a computed attribute will **get** an assembled value derived from other model attributes, and will **set** one more more mutated values derived from an input. Computeds bind themselves to their dependency attributes and will update themselves in response to any of their dependencies changing, then in turn trigger their own "change:attribute" event on the model.

<a name="addcomputed--params"></a>The **addComputed** method is called with an attribute name and a params object defining the following:

*   **get:** _Required function; invoked in context of the model_. This getter function assembles its value based on other model attributes (native or computed), and then returns the generated value. If a deps param is also specified, then the dependency array will be mapped and injected as arguments into the getter function. Note that a getter function should always reference other model attributes using the model's **get** method (for [automatic dependency mapping](#automatic-dependency-mapping)).
*   **\[deps\]:** _Optional array_. An array of attribute names that the getter function depends on. When defined, the dependency attributes will be mapped and injected as arguments into the getter function. Technically this manual property mapping is only required if your getter contains unreachable values that would be missed by automatic dependency mapping (discussed below).
*   **\[set\]:** _Optional function; invoked in context of the model_. A mutator function which receives a raw input value, and returns an object defining new key/value pairs to be merged into the model. Computed attributes declared without a **set** function are read-only.

Note that the **get** and **set** functions are invoked in the context of their parent model, so referencing this within their scopes refers to the model.

```js
var model = new Backbone.Epoxy.Model({price:100});
model.addComputed("formattedPrice", {
    deps: ["price"],
    get: function( price ) {
        return "$"+ price;
    },
    set: function( value ) {
        return {price: parseInt(value.replace("$", ""))}
    }
});

alert( model.get("formattedPrice") ); // '$100'
model.set("formattedPrice", "$150");
alert( model.get("price") ); // 150
```

### Automatic Dependency Mapping

In the above example, \["price"\] is manually declared as a dependency of the computed getter (meaning the getter will need to update itself when "price" changes). While this manual declaration is a nice safety net, we could also exclude the deps param and just **get** the price attribute instead, allowing automatic dependency mapping to discover attribute relationships.

Automatic dependency mapping works using the **get** method wrapper of the Epoxy.Model. While generating a computed attribute's initial value, the Epoxy.Model registers all attribute names requested from _all_ Epoxy.Model instances, and binds the computed attribute accordingly. Automatic mapping works great given the following:

*   All model references (the parent model, or others) are instances of Epoxy.Model.
*   All model attributes are accessed using their model's **get** method.
*   All **get** calls are free of conditional logic (keep reading...)

Regarding conditional **get** calls: this situation may create unreachable getters that cannot be automatically mapped. Consider the following BROKEN example:


```js
var model = new Backbone.Epoxy.Model({
    shortName: "Luke",
    fullName: "Luke Skywalker",
    active: false
});

// BROKEN:
model.addComputed("displayName", function() {
    return this.get("active") ? this.get("fullName") : this.get("shortName");
});
```

See the error in the above example? The fullName and shortName attributes are accessed conditionally, therefore one will be unreachable while mapping dependencies. To fix this, we could either manually declare dependencies, or else **get** all dependencies as local variables prior to conditional logic, like so:


```js
// FIXED:
model.addComputed("displayName", function() {
    var fullName = this.get("fullName");
    var shortName = this.get("shortName");
    return this.get("active") ? fullName : shortName;
});
```

### Storing Raw Values

A lesser use-case for computed attributes is the storage of additional raw model values. When a value is assigned to a computed, that value may be **get** and **set** through the model and will trigger change events, however it will not be stored in the model's attributes table, nor saved with model data. Raw values are defined as a computed attribute with a single {value:'myValue'} param.

```js
var model = new Backbone.Epoxy.Model();
model.addComputed("rawValue", {value: 100});

alert( model.get("rawValue") ); // 100
model.set("rawValue", 200);
alert( model.get("rawValue") ); // 200
```

Use the **[computeds](#computeds)** hash to automatically initialize computed attributes on your model instances.

## clearComputeds

`model.clearComputeds()`

Removes all computed properties on the model and cleans up their bound events. Cleared computeds will no longer exist on the model, so will no longer be accessible or trigger events. Be sure to call **clearComputeds** when deprecating an Epoxy model without specifically calling its **destroy** method.

## computeds

`model.computeds`

A hash table declaring computed attributes to be automatically added to a model instance by **initComputeds**. Uses **[addComputed](#addcomputed)** to create computed attributes. Computeds may be declared with a [computed params](#addcomputed--params) object, or as a getter function (uses [automatic dependency mapping](#automatic-dependency-mapping-with-computed-views)).

```js
var ComputedModel = Backbone.Epoxy.Model.extend({
    defaults: {
        firstName: "Luke",
        lastName: "Skywalker"
    },
    computeds: {
        fullNameParams: {
            deps: ["firstName", "lastName"],
            get: function( firstName, lastName ) {
                return firstName +" "+ lastName;
            },
            set: function( value ) {
                var first = value.split(" ")[0];
                var last = value.split(" ")[1];
                return {firstName: first, lastName: last};
            }
        },
        fullNameGetter: function() {
            return this.get("firstName") +" "+ this.get("lastName");
        }
    }
});
```

## destroy

`model.destroy([options])`

Override wrapper for the Backbone Model's native **destroy** method. This wrapper cleans up Epoxy model configuration, then defers to the Backbone Model's native **destroy** method. If you'd like to clean up a model without completely destroying it, use the **clearComputeds** method.

## get

`model.get(attribute)`

Override wrapper for the Backbone Model's native **get** method. This override allows native model attributes and Epoxy computed attributes to be accessed through a common API.

The Epoxy **get** method will first check for a computed attribute, then defers to the Backbone Model's native **get** method if no computed value was found. Remember that a computed attribute overrides a native attribute with the same name, so be mindful of naming collisions.

## getCopy

`model.getCopy(attribute)`

Gets a model attribute value (native or computed), and performs a shallow copy on any non-primitive values returned. This is useful for getting copies of Object and Array values that you plan to modify outside of the model, then resubmit with changes (remember that "change" events only trigger for non-primitive values when their identity changes). The **[modifyArray](#modifyarray)** and **[modifyObject](#modifyobject)** methods may also be used to perform object changes internally within the model.

## hasComputed

`model.hasComputed(attribute)`

Returns true if the model has a computed attribute with the specified name. Note that the Backbone Model's native **has** method is NOT modified by Epoxy, so will still only report on the presence of Backbone-native model attributes.

## initComputeds

`model.initComputeds()`

Clears all computed model attributes, then builds all computed attributes defined in the model's **computeds** hash. This method is automatically called by the Epoxy model constructor after initializing the model, therefore you should only need to call this manually when using Epoxy as a mixin library.

## modifyArray

`model.modifyArray(attribute, arrayMethod, [*args])`

Performs an internal update on a stored Array model attribute (native or computed) using a standard JavaScript array modifier (pop, push, reverse, shift, slice, sort, splice, and unshift), then triggers a change event for the modified model attribute. Invoke with the model attribute name to modify, a string of the array method name to call, and any additional arguments to be passed into the array method. Returns any results from the array method. Calling **modifyArray** on a non-array attribute will take no action.

```js
var model = new Backbone.Epoxy.Model({wordList:[]});
model.modifyArray("wordList", "push", "deathstar");
alert( model.get("wordList").length );
```

## modifyObject

`model.modifyObject(attribute, objectProperty, value)`

Performs an internal update on a stored Object model attribute (native or computed), then triggers a change event for the modified model attribute. Invoke with the model attribute name to modify, a property name to set on the object, and a value to set as the object property. Setting undefined to a property will delete it from the object. Returns the modified object. Calling **modifyObject** will only take action on model attributes with an assessed type of "object".

```js
var model = new Backbone.Epoxy.Model({ wordsDict:{} });
model.modifyObject("wordsDict", "deathstar", true);
alert( model.get("wordsDict").hasOwnProperty("deathstar") );
```

## removeComputed

`model.removeComputed(attribute)`

Removes a computed attribute by name, and cleans up all bound listeners. A removed computed property will no longer exist within the model, so may no longer be accessed or trigger events.

## set

`model.set(attributes, [options])`

Override wrapper for the Backbone Model's native **set** method. This override allows basic model attributes and Epoxy computed/computed attributes to be accessed through a common API.

The Epoxy **set** method will first check for a computed attribute, then defers to the Backbone Model's native **set** method if no computed was found. Remember that a computed attribute overrides a native attribute with the same name, so be mindful of naming collisions.

## toJSON

`model.toJSON([options])`

Override wrapper for the Backbone Model's native **toJSON** method. Allows an optional {computed:true} param to be passed, specifying that computed attribute values should be included in the returned data.