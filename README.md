# Backbone.Epoxy documentation
Archived version of Backbone.Epoxy's documentation, original source at https://codepen.io/anon/pen/aYMxJM. HTML to Markdown conversion done using [Turndown](http://domchristie.github.io/turndown/) with manual clean up of internal links and anchors.

## Quick links

| [Epoxy.Model](./epoxy-model)                      | [Epoxy.View](./epoxy-view)                        | [View Binding Handlers](./view-binding-handlers)          | [View Binding Filters](./view-binding-filters)    | [Epoxy.binding](./epoxy-binding)                  |
|-------------------------------------------------- |-------------------------------------------------- |---------------------------------------------------------- |-------------------------------------------------- |-------------------------------------------------- |
| [extend](./epoxy-model#extend)                    | [extend](./epoxy-view#extend)                     | [attr](./view-binding-handlers#attr)                      | [all](./view-binding-filters#all)                 | [addFilter](./epoxy-binding#addFilter)            |
| [mixin](./epoxy-model#mixin)                      | [mixin](./epoxy-view#mixin)                       | [checked](./view-binding-handlers#checked)                | [any](./view-binding-filters#any)                 | [addHandler](./epoxy-binding#addHandler)          |
| [constructor](./epoxy-model#constructor)          | [constructor](./epoxy-view#constructor)           | [classes](./view-binding-handlers#classes)                | [csv](./view-binding-filters#csv)                 | [allowedParams](./epoxy-binding#allowedParams)    |
| [addComputed](./epoxy-model#addComputed)          | [binding context](./epoxy-view#binding-context)   | [collection](./view-binding-handlers#collection)          | [decimal](./view-binding-filters#decimal)         | [config](./epoxy-binding#config)                  |
| [clearComputeds](./epoxy-model#clearComputeds)    | [applyBindings](./epoxy-view#applyBindings)       | [css](./view-binding-handlers#css)                        | [format](./view-binding-filters#format)           | [emptyCache](./epoxy-binding#emptyCache)          |
| [computeds](./epoxy-model#computeds)              | [bindingFilters](./epoxy-view#bindingFilters)     | [disabled](./view-binding-handlers#disabled)              | [integer](./view-binding-filters#integer)         |                                                   |
| [destroy](./epoxy-model#destroy)                  | [bindingHandlers](./epoxy-view#bindingHandlers)   | [enabled](./view-binding-handlers#enabled)                | [length](./view-binding-filters#length)           |                                                   |
| [get](./epoxy-model#get)                          | [bindings](./epoxy-view#bindings)                 | [events](./view-binding-handlers#events)                  | [none](./view-binding-filters#none)               |                                                   |
| [getCopy](./epoxy-model#getCopy)                  | [bindingSources](./epoxy-view#bindingSources)     | [html](./view-binding-handlers#html)                      | [not](./view-binding-filters#not)                 |                                                   |
| [hasComputed](./epoxy-model#hasComputed)          | [computeds](./epoxy-view#computeds)               | [itemView](./view-binding-handlers#itemView)              | [select](./view-binding-filters#select)           |                                                   |
| [initComputeds](./epoxy-model#initComputeds)      | [getBinding](./epoxy-view#getBinding)             | [options](./view-binding-handlers#options)                |                                                   |                                                   |
| [modifyArray](./epoxy-model#modifyArray)          | [remove](./epoxy-view#remove)                     | [optionsDefault](./view-binding-handlers#optionsDefault)  |                                                   |                                                   |
| [modifyObject](./epoxy-model#modifyObject)        | [removeBindings](./epoxy-view#removeBindings)     | [optionsEmpty](./view-binding-handlers#optionsEmpty)      |                                                   |                                                   |
| [removeComputed](./epoxy-model#removeComputed)    | [setBinding](./epoxy-view#setBinding)             | [template](./view-binding-handlers#template)              |                                                   |                                                   |
| [set](./epoxy-model#set)                          | [setterOptions](./epoxy-view#setterOptions)       | [text](./view-binding-handlers#text)                      |                                                   |                                                   |
| [toJSON](./epoxy-model#toJSON)                    | [viewModel](./epoxy-view#viewModel)               | [toggle](./view-binding-handlers#toggle)                  |                                                   |                                                   |
|                                                   |                                                   | [value](./view-binding-handlers#value)                    |                                                   |                                                   |

## APIs

- [Epoxy.Model](./epoxy-model)
- [Epoxy.View](./epoxy-view)
- [Epoxy.binding](./epoxy-binding)

## View bindings

- [View Binding Handlers](./view-binding-handlers)
- [View Binding Filter](./view-binding-filters)