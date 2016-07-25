HTML Template Language Specification [1.2](https://github.com/Adobe-Marketing-Cloud/htl-spec/tree/1.2) - March 30th, 2016
====

New Features:
* allow Java enums to be used in [comparisons](https://github.com/Adobe-Marketing-Cloud/htl-spec/blob/1.2/SPECIFICATION.md#1142-comparison-operators)

Enhancements:
* documented the `data-sly-include` `file` option, which was present in the [TCK](https://github.com/Adobe-Marketing-Cloud/htl-tck/blob/io.sightly.tck-1.0.0/src/main/resources/testfiles/scripts/blockstatements/include/include.html#L27) + reference implementation since 1.0

HTML Template Language Specification [1.1](https://github.com/Adobe-Marketing-Cloud/htl-spec/tree/1.1) - February 16th, 2015
====

New Features:
* [`styleComment`](https://github.com/Adobe-Marketing-Cloud/htl-spec/blob/1.1/SPECIFICATION.md#121-display-context) display context
* [URI manipulation](https://github.com/Adobe-Marketing-Cloud/htl-spec/blob/1.1/SPECIFICATION.md#125-uri-manipulation) options
* [`data-sly-repeat`](https://github.com/Adobe-Marketing-Cloud/htl-spec/blob/1.1/SPECIFICATION.md#227-repeat) block element
* special HTML tags - [`<sly>`](https://github.com/Adobe-Marketing-Cloud/htl-spec/blob/1.1/SPECIFICATION.md#31-sly)

Enhancements:
* the [`join`](https://github.com/Adobe-Marketing-Cloud/htl-spec/blob/1.1/SPECIFICATION.md#124-array-join) option can also be used with a simple string, outputting just the string in this case
* for [`data-sly-list`](https://github.com/Adobe-Marketing-Cloud/htl-spec/blob/1.1/SPECIFICATION.md#226-list) (and also for [`data-sly-repeat`](https://github.com/Adobe-Marketing-Cloud/htl-spec/blob/1.1/SPECIFICATION.md#227-repeat)) the element will be shown only if the attribute's value provides a non-empty collection, a string or a number
* no need to strictly define [reserved options](https://github.com/Adobe-Marketing-Cloud/htl-spec/blob/1.0/SPECIFICATION.md#13-reserved-options) like in 1.0
