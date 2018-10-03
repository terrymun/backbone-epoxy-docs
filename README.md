# Backbone.Epoxy documentation
Cleaned up version of Backbone.Epoxy's documentation, original source at https://codepen.io/anon/pen/aYMxJM. HTML to Markdown conversion done using [Turndown](http://domchristie.github.io/turndown/) with manual clean up of internal links and anchors.

## Quick links

| [Epoxy.Model](./epoxy-model.md)                   | [Epoxy.View](./epoxy-view.md)                         | [View Binding Handlers](./view-binding-handlers)          | [View Binding Filters](./view-binding-filters.md) | [Epoxy.binding](./epoxy-binding.md)               |
|-------------------------------------------------- |------------------------------------------------------ |---------------------------------------------------------- |-------------------------------------------------- |-------------------------------------------------- |
| [extend](./epoxy-model.md#extend)                 | [extend](./epoxy-view.md#extend)                      | [attr](./view-binding-handlers#attr)                      | [all](./view-binding-filters.md#all)              | [addFilter](./epoxy-binding.md#addFilter)         |
| [mixin](./epoxy-model.md#mixin)                   | [mixin](./epoxy-view.md#mixin)                        | [checked](./view-binding-handlers#checked)                | [any](./view-binding-filters.md#any)              | [addHandler](./epoxy-binding.md#addHandler)       |
| [constructor](./epoxy-model.md#constructor)       | [constructor](./epoxy-view.md#constructor)            | [classes](./view-binding-handlers#classes)                | [csv](./view-binding-filters.md#csv)              | [allowedParams](./epoxy-binding.md#allowedParams) |
| [addComputed](./epoxy-model.md#addComputed)       | [binding context](./epoxy-view.md#binding-context)    | [collection](./view-binding-handlers#collection)          | [decimal](./view-binding-filters.md#decimal)      | [config](./epoxy-binding.md#config)               |
| [clearComputeds](./epoxy-model.md#clearComputeds) | [applyBindings](./epoxy-view.md#applyBindings)        | [css](./view-binding-handlers#css)                        | [format](./view-binding-filters.md#format)        | [emptyCache](./epoxy-binding.md#emptyCache)       |
| [computeds](./epoxy-model.md#computeds)           | [bindingFilters](./epoxy-view.md#bindingFilters)      | [disabled](./view-binding-handlers#disabled)              | [integer](./view-binding-filters.md#integer)      |                                                   |
| [destroy](./epoxy-model.md#destroy)               | [bindingHandlers](./epoxy-view.md#bindingHandlers)    | [enabled](./view-binding-handlers#enabled)                | [length](./view-binding-filters.md#length)        |                                                   |
| [get](./epoxy-model.md#get)                       | [bindings](./epoxy-view.md#bindings)                  | [events](./view-binding-handlers#events)                  | [none](./view-binding-filters.md#none)            |                                                   |
| [getCopy](./epoxy-model.md#getCopy)               | [bindingSources](./epoxy-view.md#bindingSources)      | [html](./view-binding-handlers#html)                      | [not](./view-binding-filters.md#not)              |                                                   |
| [hasComputed](./epoxy-model.md#hasComputed)       | [computeds](./epoxy-view.md#computeds)                | [itemView](./view-binding-handlers#itemView)              | [select](./view-binding-filters.md#select)        |                                                   |
| [initComputeds](./epoxy-model.md#initComputeds)   | [getBinding](./epoxy-view.md#getBinding)              | [options](./view-binding-handlers#options)                |                                                   |                                                   |
| [modifyArray](./epoxy-model.md#modifyArray)       | [remove](./epoxy-view.md#remove)                      | [optionsDefault](./view-binding-handlers#optionsDefault)  |                                                   |                                                   |
| [modifyObject](./epoxy-model.md#modifyObject)     | [removeBindings](./epoxy-view.md#removeBindings)      | [optionsEmpty](./view-binding-handlers#optionsEmpty)      |                                                   |                                                   |
| [removeComputed](./epoxy-model.md#removeComputed) | [setBinding](./epoxy-view.md#setBinding)              | [template](./view-binding-handlers#template)              |                                                   |                                                   |
| [set](./epoxy-model.md#set)                       | [setterOptions](./epoxy-view.md#setterOptions)        | [text](./view-binding-handlers#text)                      |                                                   |                                                   |
| [toJSON](./epoxy-model.md#toJSON)                 | [viewModel](./epoxy-view.md#viewModel)                | [toggle](./view-binding-handlers#toggle)                  |                                                   |                                                   |
|                                                   |                                                       | [value](./view-binding-handlers#value)                    |                                                   |                                                   |

## APIs

- [Epoxy.Model](./epoxy-model.md)
- [Epoxy.View](./epoxy-view.md)
- [Epoxy.binding](./epoxy-binding.md)

## View bindings

- [View Binding Handlers](./view-binding-handlers)
- [View Binding Filter](./view-binding-filters.md)
