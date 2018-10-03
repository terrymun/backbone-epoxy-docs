# View Binding Handlers

## Quick links

* [attr](#attr)
* [checked](#checked)
* [classes](#classes)
* [collection](#collection)
* [css](#css)
* [disabled](#disabled)
* [enabled](#enabled)
* [events](#events)
* [html](#html)
* [itemView](#itemview)
* [options](#options)
* [optionsDefault](#optionsdefault)
* [optionsEmpty](#optionsempty)
* [template](#template)
* [text](#text)
* [toggle](#toggle)
* [value](#value)

## Overview

Binding handlers are used to interchange model data with elements in the view. The Epoxy View includes default binding handlers for common tasks such as managing a bound element's content, class, and style. However, don't feel limited to just Epoxy's out-of-the-box binding handlers. You may easily [define your own binding handlers](#view-binding-handlers) within a view.

One or more binding handlers may be applied to an element as a comma-separated properties list (here shown using DOM attribute bindings):


```html
<a data-bind="text:linkText,attr:{href:linkUrl,title:linkText}"></a>
```

In the above example, "text:" and "attr:" are binding handlers, while "linkText" and "linkUrl" are the bound model attributes assigned to them. In the case of the "attr" handler, its value is a hash of attribute names, each with an associated model value.

Be aware that binding declarations are parsed as JavaScript Object bodies, therefore a binding declaration _must_ conform to proper JavaScript syntax. To give due credit, Epoxy's binding system is modeled after the clever technique used in [Knockout.js](http://knockoutjs.com/). However, Epoxy deliberately limits some of Knockout's free-form binding allowances, such as inline concatenation (which becomes overly technical within the presentation layer, IMHO). If you're coming from Knockout, don't expect to write as much inline binding JavaScript.

## attr

read-only `data-bind="attr:{name:modelAttribute,'data-name':modelAttribute}"`

Binds model attribute values to a list of element attributes. Attribute bindings are defined as a hash of key/value pairs, where each key defines an element attribute and the model reference defines the attribute value. Attribute keys may be defined as strings.

## checked

read-write `data-bind="checked:modelAttribute"`

Binds checkbox and radio elements to the model. The binding behavior differs slightly for various element/value combinations:

*   **Radios:** a bound radio element will become checked when the element's value matches the bound model attribute. Checking a radio will set its value to the bound model attribute.
*   **Checkbox arrays:** when a checkbox element is bound to an Array model attribute, the element will be checked while its value exists within the value array, and unchecked while its value is absent from the array. Likewise, toggling the element's checked property will add and remove its value from the bound array. Keep in mind that arrays modified within a native Backbone.Model will NOT trigger a "change" event unless the array _instance_ is changed. Using an Epoxy.Model computed will help here by managing object copies, and by performing array comparisons by content.
*   **Checkbox toggles:** when a checkbox element is bound to a primitive model attribute (types String, Boolean, or Number), the element will be checked based on a loosely-typed assessment of the model attribute's truthiness. When the element's "checked" status is toggled, it will submit a Boolean value back to the model attribute.

## classes

read-only `data-bind="classes:{active:modelAttr,'is-active':modelAttr}"`

Toggles a list of element class names based on the truthiness of their bound model attributes. Class bindings are defined as a hash of key/value pairs, where each key defines a class name and is value references a model attribute that will be loosely-type checked for truthiness; truthy attributes will enable their class, while falsey attributes will disable their class. Class name keys may be defined as strings.

## collection

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

## css

read-only `data-bind="css:{color:modelAttribute,'font-size':modelAttribute}"`

Binds a list of CSS styles to model attributes. CSS style bindings are defined as a hash of key/value pairs, where each key defines a CSS style name and the model attribute defines the style's literal value. CSS keys may be defined as strings.

## disabled

read-only `data-bind="disabled:modelAttribute"`

Toggles a form element's "disabled" property based on a loosely-typed assessment of the bound model attribute's truthiness. A truthy value disables the element (inversion of the **enabled** handler).

## enabled

read-only `data-bind="enabled:modelAttribute"`

Toggles a form element's "disabled" status based on a loosely-typed assessment of the bound model attribute's truthiness. A truthy value enables the element (inversion of the **disabled** handler).

## events

read-only `data-bind="events:['keydown','focus']"`

The **events** handler is a special binding used to parameterize what DOM events will trigger changes for read-write element bindings. By default, all read-write bindings will subscribe to their bound element's "change" event to trigger model updates. You may bind additional DOM event triggers for the element by listing them in the **events** array.

## html

read-only `data-bind="html:modelAttribute"`

Sets the element's HTML content to the bound model attribute value. Uses the jQuery **html** method.

## itemView

read-only `data-bind="collection:$collection,itemView:'myView'"`

The **itemView** handler is a special binding used to parameterize what view is used to render **collection** binding items. By default, a **collection** binding will access its parent view for a property called "itemView". Use the **itemView** binding to specify a different attribute of the parent view. This may be useful when binding multiple data sources.

## options

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

## optionsDefault

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

## optionsEmpty

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

## template

read-only `data-bind="template:$source"`

The **template** binding extracts its element's HTML content while initializing, and then parses that content into an Underscore template using the bound data. You have some options on how both the template string and the bound data sources are defined.

### Defining the template string

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

   **Model Source** : given the above MyView example, the **template** binding may reference a Backbone.Model data source directly through its context reference ($sourceName), such as:
    
    ```html
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
    

## text

read-write `data-bind="text:modelAttribute"`

Sets the element's text content to the bound model attribute value. Uses the jQuery **text** method.

To enable a writeable binding, add the HTML5 contenteditable property to the element:

```html
<span contenteditable="true" data-bind="text:yourName,events:['blur paste']">
    Select to edit your name.
</span>
```

## toggle

read-only `data-bind="toggle:modelAttribute"`

Toggles the element's display based on a loosely-typed assessment of the bound model attribute's truthiness. Uses the jQuery **toggle( \[boolean\] )** method.

## value

read-write `data-bind="value:modelAttribute"`

Binds an input element's (input, select, textarea) value to an underlying model attribute. The element will set its value to the bound model attribute upon change event triggers (see the **[events](#handler-events)** handler), and update its value when the underlying model attribute changes.
