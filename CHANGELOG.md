HTML Template Language Specification [1.4](https://github.com/adobe/htl-spec/tree/1.4) - June 18th, 2018
====
Version 1.4 brings the following enhancements:
* `data-sly-list` and `data-sly-repeat` iteration control (#55) [0][1]
*  the introduction of the `in` relational operator (#56) [2]
* support for negative Number literals (#44) [3]
* attribute identifier for the `data-sly-unwrap` block statement (#52) [4]
* an extended list of attributes for which the `uri` display context is applied automatically (#62) [5]
* a new block statement - `data-sly-set` [6]

For a full list of issues addressed by this release please check [7].

[0] - https://github.com/adobe/htl-spec/blob/1.4/SPECIFICATION.md#226-list
[1] - https://github.com/adobe/htl-spec/blob/1.4/SPECIFICATION.md#227-repeat
[2] - https://github.com/adobe/htl-spec/blob/1.4/SPECIFICATION.md#1143-relational-operators
[3] - https://github.com/adobe/htl-spec/blob/1.4/SPECIFICATION.md#111-grammar
[4] - https://github.com/adobe/htl-spec/blob/1.4/SPECIFICATION.md#2211-unwrap
[5] - https://github.com/adobe/htl-spec/blob/1.4/SPECIFICATION.md#113-context-sensitive
[6] - https://github.com/adobe/htl-spec/blob/1.4/SPECIFICATION.md#2212-set
[7] - https://github.com/adobe/htl-spec/milestone/3?closed=1

HTML Template Language Specification [1.3.1](https://github.com/adobe/htl-spec/tree/1.3.1) - August 1st, 2017
====
Enhancements:
* corrected examples from the [URI manipulation](https://github.com/adobe/htl-spec/blob/1.3.1/SPECIFICATION.md#125-uri-manipulation) section
* added paragraph about the style and the event attributes for the `data-sly-attribute` block element

HTML Template Language Specification [1.3](https://github.com/adobe/htl-spec/tree/1.3) - March 16th, 2017
====
New features:
- the [`format`](https://github.com/adobe/htl-spec/blob/1.3/SPECIFICATION.md#122-format) option has been extended to allow formatting `strings`, `dates` and `numbers`


HTML Template Language Specification [1.2](https://github.com/adobe/htl-spec/tree/1.2) - March 30th, 2016
====

New Features:
* allow Java enums to be used in [comparisons](https://github.com/adobe/htl-spec/blob/1.2/SPECIFICATION.md#1142-comparison-operators)

Enhancements:
* documented the `data-sly-include` `file` option, which was present in the [TCK](https://github.com/adobe/htl-tck/blob/io.sightly.tck-1.0.0/src/main/resources/testfiles/scripts/blockstatements/include/include.html#L27) + reference implementation since 1.0

HTML Template Language Specification [1.1](https://github.com/adobe/htl-spec/tree/1.1) - February 16th, 2015
====

New Features:
* [`styleComment`](https://github.com/adobe/htl-spec/blob/1.1/SPECIFICATION.md#121-display-context) display context
* [URI manipulation](https://github.com/adobe/htl-spec/blob/1.1/SPECIFICATION.md#125-uri-manipulation) options
* [`data-sly-repeat`](https://github.com/adobe/htl-spec/blob/1.1/SPECIFICATION.md#227-repeat) block element
* special HTML tags - [`<sly>`](https://github.com/adobe/htl-spec/blob/1.1/SPECIFICATION.md#31-sly)

Enhancements:
* the [`join`](https://github.com/adobe/htl-spec/blob/1.1/SPECIFICATION.md#124-array-join) option can also be used with a simple string, outputting just the string in this case
* for [`data-sly-list`](https://github.com/adobe/htl-spec/blob/1.1/SPECIFICATION.md#226-list) (and also for [`data-sly-repeat`](https://github.com/adobe/htl-spec/blob/1.1/SPECIFICATION.md#227-repeat)) the element will be shown only if the attribute's value provides a non-empty collection, a string or a number
* no need to strictly define [reserved options](https://github.com/adobe/htl-spec/blob/1.0/SPECIFICATION.md#13-reserved-options) like in 1.0
