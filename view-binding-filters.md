# View Binding Filters

## Quick links

* [all](#all)
* [any](#any)
* [csv](#csv)
* [decimal](#decimal)
* [format](#format)
* [integer](#integer)
* [length](#length)
* [none](#none)
* [not](#not)
* [select](#select)

## Overview

Binding filters provide a layer of flexibility for formatting data attributes directly within a binding declaration; they may perform read-only formatting on multiple inputs, or perform read-write formatting on a single input. Filters are intended to format raw values for binding-specific purposes:

```html
<span data-bind="toggle:not(firstName)">Please enter a first name.</span>
```

Binding filters may be parameterized with any data attribute(s) available in the [binding context](#view-binding-context), or with primitive values (String, Number, or Boolean). However, binding filters may **NOT** be nested within one another — this is very deliberate: application logic does not belong within binding declarations. If you need to perform multi-pass value formatting, that work should be done in a computed property, or else applied as a custom binding handler.

See view [bindingFilters](#view-binding-filters) for process on defining your own custom filters.

## all

read-only `data-bind="toggle:all(dataAttribute,[...])"`

Assesses one or more data attributes as true if _all_ attributes are truthy.

## any

read-only `data-bind="toggle:any(dataAttribute,[...])"`

Assesses one or more data attributes as true if _any_ of the attributes are truthy.

## csv

read-write `data-bind="checked:csv(dataList)"`

Exchanges comma-separated values data between models and bindings. A string is parsed into a comma-separated array when read through the **csv** filter, and then is formatted as a comma-separated string when written back through the filter. This is handy for storing a primitive list value within a model, while using it as an array in view bindings. Note that this is only a simple split/join implementation of comma-separated values, so advanced CSV formatting (string closures, etc) will not apply.

## decimal

read-write `data-bind="checked:decimal(floatAttribute)"`

Exchanges a floating-point decimal value between models and bindings. A string is parsed into a float when being read or written through the **decimal** filter. This is handy for preserving float data formatting when binding to input elements.

## format

read-only `data-bind="text:format('$1 of $2',dataAttribute,dataAttribute)"`

Formats a string with a series of data attribute replacements. Operates the same as string replacement for RegExp capture groups, where field insertions are denoted as "$1, $2, ... $n". Escape intentional "$n" patterns within the template string as "\\$n". Note that the first value insertion index is 1, not 0.

## integer

read-write `data-bind="checked:integer(numberAttribute)"`

Exchanges an integer value between models and bindings. A string is parsed into an integer when being read or written through the **integer** filter. This is handy for preserving numeric data formatting when binding to input elements.

## length

read-only `data-bind="toggle:length(dataAttribute)"`

Returns the length of an Array or Collection data attribute, or 0 by default. Useful for loosely-typed assessments of a collection's content status.

## none

read-only `data-bind="toggle:none(dataAttribute,[...])"`

Assesses one or more data attributes as true if _none_ of the attributes are truthy.

## not

read-only `data-bind="toggle:not(dataAttribute)"`

Inverts the loosely-typed boolean value of a data attribute.

## select

read-only `data-bind="text:select(conditionalValue,truthyValue,falseyValue)"`

Performs a ternary operation using JavaScript's ?: operator. The **select** operation receives three arguments: the first argument is assessed as a loosely-typed conditional; if truthy, the second argument is returned; if falsey, the third argument is returned.
