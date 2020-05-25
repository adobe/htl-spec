HTML Template Language Specification
====

**Version:** 1.4  
**Authors:** Radu Cotescu, Marius Dănilă, Peeter Piegaze, Senol Tas, Gabriel Walt, Honwai Wong  
**License:** [Apache License 2.0](http://www.apache.org/licenses/LICENSE-2.0.txt)  
**Status:** Final release  
**Release:** 18 June 2018

#### Contents
1. [Expression language, syntax and semantics](#1-expression-language-syntax-and-semantics)  
    1. [Syntax](#11-syntax)
        1. [Grammar](#111-grammar)
        2. [Expressions](#112-expressions)
        3. [Context-Sensitive](#113-context-sensitive)
        4. [Operators](#114-operators)
            1. [Logical Operators](#1141-logical-operators)
            2. [Comparison Operators](#1142-comparison-operators)
            3. [Relational Operators](#1143-relational-operators)
        5. [Casting](#115-casting)
            1. [Boolean](#1151-boolean)
            2. [String](#1152-string)
        6. [Options](#116-options)
        7. [Parametric Expressions](#117-parametric-expressions)
        8. [Whitespace](#118-whitespace)
        9. [Comments](#119-comments)
    2. [Available Expression Options](#12-available-expression-options)
        1. [Display Context](#121-display-context)
        2. [Format](#122-format)
            1. [Strings](#1221-strings)
            2. [Dates](#1222-dates)
            3. [Numbers](#1223-numbers)
        3. [i18n](#123-i18n)
        4. [Array Join](#124-array-join)
        5. [URI Manipulation](#125-uri-manipulation)
2. [Block Statements](#2-block-statements)
    1. [Syntax](#21-syntax)
        1. [Identifiers](#211-identifiers)
    2. [Available Block Statements](#22-available-block-statements)
        1. [Use](#221-use)
        2. [Text](#222-text)
        3. [Attribute](#223-attribute)
            1. [Detailed Examples](#2231-detailed-examples)
        4. [Element](#224-element)
        5. [Test](#225-test)
        6. [List](#226-list)
        7. [Repeat](#227-repeat)
        8. [Include](#228-include)
        9. [Resource](#229-resource)
        10. [Template & Call](#2210-template--call)
            1. [Template](#22101-template)
            2. [Call](#22102-call)
            3. [Examples](#22103-examples)
        11. [Unwrap](#2211-unwrap)
        12. [Set](#2212-set)
    3. [Block Statements Priority](#23-block-statements-priority)
3. [Special HTML tags](#3-special-html-tags)
    1. [&lt;sly&gt;](#31-sly)
4. [Use-API](#4-use-api)
    1. [Java Use-API](#41-java-use-api)
    2. [JavaScript Use-API](#42-javascript-use-api)

## 1. Expression language, syntax and semantics

### 1.1. Syntax

#### 1.1.1. Grammar
The grammar of the HTL Expression Language is pretty simple and can be summarised to the following definitions:

    expression = '${' , [exprNode] , [ , '@' , optionList] , '}' ;
    
    optionList = option {',' , option} ;
    
    option = id [ , '=' , optionValues] ; 
    
    optionValues = exprNode
                 | '[' , valueList , ']' ;
    
    valueList = exprNode {',' exprNode} ;
    
    exprNode = orBinaryOp , '?' , orBinaryOp , ws , ':' , ws , orBinaryOp
             | orBinaryOp ;
    
    orBinaryOp = andBinaryOp {, '||', andBinaryOp};
    
    andBinaryOp = inBinaryOp {, '&&', inBinaryOp};

    inBinaryOp = comparisonOp [, 'in', comparisonOp];

    comparisonOp = factor [, comparisonOperator, factor];

    comparisonOperator = '<'
         | '<='
         | '=='
         | '>='
         | '>'
         | '!=' ;
    
    factor = term
           | '!' , term ;
    
    term = propertyAccess
         | '(' , exprNode  , ')'
         | '[', valueList, ']' ;
    
    /* Note the 'comma rule' means zero or more whitespace characters. Used to indicate optional whitespace around terminals above */
    , = {ws} ;
    
    ws = ' '
       | '\t'
       | '\r'
       | '\n'
       | '\u000B'
       | '\u00A0';
    
    /* Note that unlike terminals above, the field access character '.' cannot have optional whitespace around it */
    propertyAccess = atom {'.' id}
                   | atom {'[' , exprNode , ']'};

    atom = string
         | id
         | int
         | float
         | bool ;
    
    bool = 'true'
         | 'false' ;
    
    id = ('a'..'z'|'A'..'Z'|'_') {'a'..'z'|'A'..'Z'|'0'..'9'|'_'|':'} ;
    
    int = ['-']('1'..'9'){'0'..'9'}
        | '0' ;
    
    float = ['-']('1'..'9'){'0'..'9'} '.' {'0'..'9'} [exponent]
          | ['-']'0.' ('0'..'9') {'0'..'9'} [exponent]
          | ['-']('1'..'9'){'0'..'9'} exponent ;
    
    /* An HTL comment can contain any character sequence other than '*/-->' */
    comment = '<!--/*' {-('*/-->')} '*/-->' ;
    
    /* A string can be delimited by either double or single quotes. Within these delimiters it may contain either escape sequences or any characters other than backslash and whichever quote was used for delimiting. */
    string = '"' {escSeq | -('\\' | '"')} '"'
           | '\'' {escSeq | -('\\'|'\'')} '\'' ;
    
    exponent = ('e'|'E') ['+'|'-'] ('0'..'9'){'0'..'9'} ;
    
    escSeq = '\\' ('b'|'t'|'n'|'f'|'r'|'\"'|'\''|'\\')
           | unicodeEsc;
    
    unicodeEsc = '\\' 'u' hexDigit hexDigit hexDigit hexDigit ;
    
    hexDigit = ('0'..'9'|'a'..'f'|'A'..'F') ;

The above grammar is adapted from the source ANTLR files. It uses the following conventions:

    foo   Alphabetic words and comma (',') are rule names.
    /**/  Slash-star and star-slash delimit comments.
          Whitespace within a rule production indicates direct concatenation.
    =     Equals sign indicates the rule production.
    ;     Semicolon indicates end of production.
    ' '   Single quotes delimit terminals.
    |     Pipe indicates OR.
    []    Square brackets indicate an option (zero or one times).
    {}    Curly brackets indicate repetition (zero or more times).
    -     Minus sign indicates 'any character or character sequence other than the following'.
    ..    Ellipses indicates a range between two single-character terminals, by collation order.

[Like in JavaScript](http://www.ecma-international.org/ecma-262/5.1/#sec-7.8.4), strings quotes can be escaped by prefixing a backslash to the quote (`\'`) or double-quote (`\"`).

Single character escape sequences: `\t \b \n \r \f \' \" \\`

Unicode escape sequences: `\u` followed by 4 hexadecimal digits (e.g.: `\u0022` for `"`, `\u0027` for `'`, `\u003c` for `<`, or `\u003e` for `>`) 

Like in JSP (see section "1.2.2 Literal-expression" from the [JSP 2.1 Expression Language Specification](http://download.oracle.com/otn-pub/jcp/jsp-2.1-fr-spec-oth-JSpec/jsp-2_1-fr-spec-el.pdf)), to escape an expression (the `${`), it can be prefixed it with a backslash (`\${`).

#### 1.1.2. Expressions
Here are some examples of HTL expressions:

```html
<!--/* Identifiers: */-->
${myVar}

<!--/* Accesses a member of an object: */-->
${myObject.key}
${myObject['key']}
${myObject[keyVar]}

<!--/* Accesses an index of an array: */-->
${myArray[1]}
${myArray[indexVar]}

<!--/* Literals: */-->
${true}
${42}
${'string'}
${"string"}
${[1, 2, 3, true, 'string']}
```

#### 1.1.3. Context-Sensitive
Expressions can be used in following contexts for outputting identifiers into the markup with automatic context-aware XSS protection.

```html
<tag>Some text ${myText}</tag>
<tag attr="Some text ${myAttrValue}"></tag>
<tag href="${myHrefValue}"></tag>
```

HTL expressions used to output values for the following HTML attributes that provide URIs or URLs will automatically be processed with the
`uri` [display context](#121-display-context), unless an explicit `context` is provided:
* `action` (`<form>`)
* `cite` (`<blockquote>`, `<del>`, `<ins>`, `<q>` tags)
* `data` (`<object>`)
* `formaction` (`<button>`, `<input>`)
* `href` (`<a>`, `<area>`, `<link>`, `<base>`)
* `manifest` (`<html>`)
* `poster` (`<video>`)
* `src` (`<audio>`, `<embed>`, `<iframe>`, `<img>`, `<input>`, `<script>`, `<source>`, `<track>`, `<video>`)

For style and script contexts, it is mandatory to set a context. If the context isn't set, the expression shouldn't output anything. Some examples:

```html
<!--/* Scripts */-->
<a href="#whatever" onclick="${myFunctionName @ context='scriptToken'}()">Link</a>
<script>var ${myVarName @ context="scriptToken"}="Bar";</script>
<script>var bar='${someText @ context="scriptString"}';</script>
 
<!--/* Styles */-->
<a href="#whatever" style="color: ${colorName @ context='styleToken'};">Link</a>
<style>
    a.${className @ context="styleToken"} {
        font-family: '${fontFamily @ context="styleString"}', sans-serif;
        color: #${colorHashValue @ context="styleToken"};
        margin-${side @ context="styleToken"}: 1em; /* E.g. for bi-directional text */
    }
</style>
```

#### 1.1.4. Operators
##### 1.1.4.1. Logical Operators
Only the following logical operators are currently supported, all other operations have to be prepared through the Use-API:

```html
${varOne && !(varTwo || varThree)} <!--/* 1. Grouping parenthesis */-->
${!myVar}                          <!--/* 2. Logical NOT */-->
${varOne && varTwo}                <!--/* 3. Logical AND */-->
${varOne || varTwo}                <!--/* 4. Logical OR */-->
${varChoice ? varOne : varTwo}     <!--/* 5. Conditional (ternary) (note that the ':' separator must be surrounded by a space) */-->
```

The numbers written in the comments above correspond to the precedence of the operators.

The logical `&&` and `||` operators work like the [JavaScript `||` and `&&` operators](http://www.ecma-international.org/ecma-262/5.1/#sec-11.11): they return the value of one of the specified operands, so if these operators are used with non-Boolean values, they may return a non-Boolean value. This offers a handy way to use the `||` operator to specify default string values:

```html
<!--/*
    For example in following case it will show pageTitle if it exists,
    else it shows jcr:title, and if that doesn't exist either, then
    resource.name is shown.
*/-->
${properties.pageTitle || properties.jcr:title || resource.name}
```

##### 1.1.4.2. Comparison Operators
HTL also provides a set of strict comparison operators which can be used for comparing values of operands of the same type; no type conversion will be applied to any of the operands. The equality operators (`==`, `!=`) work similarly to the [JavaScript `===`](http://www.ecma-international.org/ecma-262/5.1/#sec-11.9.4) and the [JavaScript `!==`](http://www.ecma-international.org/ecma-262/5.1/#sec-11.9.4) identity operators.

```html
${nullValueOne == nullValueTwo}        <!-- null comparison -->
${nullValueOne != nullValueTwo}        <!-- null comparison -->
${stringValueOne == stringValueTwo}    <!-- string comparison -->
${stringValueOne != stringValueTwo}    <!-- string comparison -->
${numberValueOne < numberValueTwo}     <!-- number comparison -->
${numberValueOne <= numberValueTwo}    <!-- number comparison -->
${numberValueOne == numberValueTwo}    <!-- number comparison -->
${numberValueOne >= numberValueTwo}    <!-- number comparison -->
${numberValueOne > numberValueTwo}     <!-- number comparison -->
${numberValueOne != numberValueTwo}    <!-- number comparison -->
${booleanValueOne == booleanValueTwo}  <!-- boolean comparison -->
${booleanValueOne != booleanValueTwo}  <!-- boolean comparison -->
${enumConstant == 'CONSTANT_NAME'}     <!-- Java Enum comparison -->
```

##### 1.1.4.3. Relational Operators
The `in` relational operator can be used to:

1. Check if a `String` is contained by another `String` (case-sensitive):
    ```html
    ${'a' in 'abc'} <!--/* returns true */-->
    ${'ab' in 'abc'} <!--/* returns true */-->
    ${'bc' in 'abc'} <!--/* returns true */-->
    ${'abc' in 'abc'} <!--/* returns true */-->
    ${'d' in 'abc'} <!--/* returns false */-->
    ```

2. Check if an `Array` or a `List` contains an object:
    ```html
    <!--/*
      Assuming myArray would be in scope and would have the following content:
      [100, 200, 300, 400, 500]
    */-->
    ${100 in myArray} <!--/* returns true */-->
    ${300 in myArray} <!--/* returns true */-->
    ${1 in myArray} <!--/* returns false */-->
    ```

3. Check if an object has a property or a `Map` has a key:

    Assuming the following use object (more details in the [Use-API](#4-use-api) section) provided by a `logic.js` file:
    ```javascript
    use(function () {
        return {
            a: true,
            b: 'two',
            c: 3
        };
    });
    ```

    ```html
    <sly data-sly-use.logic="logic.js" />
    ${'a' in logic} <!--/* returns true */-->
    ${'b' in logic} <!--/* returns true */-->
    ${'c' in logic} <!--/* returns true */-->
    ${'two' in logic} <!--/* returns false */-->
    ```


#### 1.1.5. Casting

##### 1.1.5.1. Boolean
These expressions evaluate to false:

* `false`
* `0` (zero)
* `''` or `""` (empty string)
* `[]` (empty iterable)

These evaluate to true:

* `"false"` (non-empty string)
* `[0]` (non-empty iterable)

##### 1.1.5.2. String
This is how non-string types are converted when being output:

```html
${0}              <!--/* outputs: 0 */-->
${true}           <!--/* outputs: true */-->
${false}          <!--/* outputs: false */-->
${[1, 2, 3]}      <!--/* outputs: 1,2,3 */-->
${[true, false]}  <!--/* outputs: true,false */-->
${['foo', 'bar']} <!--/* outputs: foo,bar */-->
${['foo', '']}    <!--/* outputs: foo, */-->
```

#### 1.1.6. Options
Expression options can act on the expression and modify it.

```html
<!--/* An option without a value: */-->
${myVar @ optName}
 
<!--/* Values can be in the form of identifiers or of literals: */-->
${myVar @ optName=myVar}
${myVar @ optName=true}
${myVar @ optName=42}
${myVar @ optName='string'}
${myVar @ optName="string"}
${myVar @ optName=[myVar, 'string']}
 
<!--/* Values can also use operators: */-->
${myVar @ optName=(varOne && varTwo) || !varThree}
 
<!--/* Multiple options: */-->
${myVar @ optOne, optTwo=myVar, optThree='string', optFour=[myVar, 'string']}
```

#### 1.1.7 Parametric Expressions
Expressions with only options can be used for passing parameters to block elements.

```html
<tag data-sly-BLOCK="${@ paramOne, paramTwo=myVar, paramThree='string', paramFour=[myVar, 'stringing']}">element content</tag>
```

#### 1.1.8. Whitespace
Whitespace characters (spaces and tabs) are allowed between any part of an expression:

```html
<!--/* No spaces: */-->
${myVar@argOne,argTwo=myVar,argThree='string',argFour=[myVar,'string']}
 
<!--/* Spaces everywhere: */-->
${ myVar @ argOne , argTwo = myVar , argThree = 'string' , argFour = [ myVar , 'string' ] }
 
<!--/* This is the recommended spacing style though: */-->
${myVar @ argOne, argTwo=myVar, argThree='string', argFour=[myVar, 'string']}
```

#### 1.1.9. Comments
HTL comments combine HTML and JavaScript multi-line comments: `<!--/* */-->`

HTL comments are not evaluated and are removed from the result.

```html
<!--/* The content of this comment will be removed from the output. */-->
```

HTL expressions inside HTML comments are evaluated, but not block statements:

```html
<!-- Page title: ${currentPage.jcr:title} -->
```

### 1.2. Available Expression Options

#### 1.2.1. Display Context
To protect against cross-site scripting (XSS) vulnerabilities, HTL automatically recognises the context within which an output string is to be displayed within the final HTML output, and escapes that string appropriately. 

It is also possible to override the automatic display context handling with the `context` option.

```html
${properties.jcr:title @ context='html'}          <!--/* Use this in case you want to output HTML - Removes markup that may contain XSS risks */-->
${properties.jcr:title @ context='text'}          <!--/* Use this for simple HTML content - Encodes all HTML */-->
${properties.jcr:title @ context='elementName'}   <!--/* Allows only element names that are white-listed, outputs 'div' otherwise */-->
${properties.jcr:title @ context='attributeName'} <!--/* Outputs nothing if the value doesn't correspond to the HTML attribute name syntax - doesn't allow 'style' and 'on*' attributes */-->
${properties.jcr:title @ context='attribute'}     <!--/* Applies HTML attribute escaping */-->
${properties.jcr:title @ context='uri'}           <!--/* Outputs nothing if the value contains XSS risks */-->
${properties.jcr:title @ context='scriptToken'}   <!--/* Outputs nothing if the value doesn't correspond to an Identifier, String literal or Numeric literal JavaScript token */-->
${properties.jcr:title @ context='scriptString'}  <!--/* Applies JavaScript string escaping */-->
${properties.jcr:title @ context='scriptComment'} <!--/* Context for Javascript block comments. Outputs nothing if value is trying to break out of the comment context */-->
${properties.jcr:title @ context='scriptRegExp'}  <!--/* Applies JavaScript regular expression escaping */-->
${properties.jcr:title @ context='styleToken'}    <!--/* Outputs nothing if the value doesn't correspond to the CSS token syntax */-->
${properties.jcr:title @ context='styleString'}   <!--/* Applies CSS string escaping */-->
${properties.jcr:title @ context='styleComment'}  <!--/* Context for CSS comments. Outputs nothing if value is trying to break out of the comment context */-->
${properties.jcr:title @ context='comment'}       <!--/* Applies HTML comment escaping */-->
${properties.jcr:title @ context='number'}        <!--/* Outputs zero if the value is not a number */-->
${properties.jcr:title @ context='unsafe'}        <!--/* Use this at your own risk, this disables XSS protection completely */-->
```

Note that `context='elementName'` allows only the following element names:

```html
section, nav, article, aside, h1, h2, h3, h4, h5, h6, header, footer, address, main, p, pre, blockquote, ol, li, dl, dt, dd, figure, figcaption, div, a, em, strong, small, s, cite, q, dfn, abbr, data, time, code, var, samp, kbd, sub, sup, i, b, u, mark, ruby, rt, rp, bdi, bdo, span, br, wbr, ins, del, table, caption, colgroup, col, tbody, thead, tfoot, tr, td, th
```

If you want to use HTL expressions within HTML comments you might need to adjust the context depending on what you want to output, as the automatically implied context will be `comment`:

```html
<!--[if IE]><link rel="shortcut icon" href="${site.root @ context='uri'}/images/favicon/favicon.ico?v2"><![endif]-->
```

#### 1.2.2. Format
This option can be used to format `Strings`, `Dates` and `Numbers`. A formatting pattern string must be supplied in the expression and the `format` option will contain the value(s) to be used. Type of formatting will be decided based on:

1. the `type` option, if present (accepted values are `string`, `date` and `number`)
2. placeholders (eg: `{0}`) in the pattern, triggers string formatting
3. type of `format` option object, when the type is a `Date` or a `Number`
4. default, fallback to string formatting

##### 1.2.2.1. Strings
String formatting can be combined with the [`i18n`](#123-i18n) option so that placeholders are replaced after the string has been run through the dictionary.

##### Examples

```html
<!--/* Numbered parameters for injecting variables: */-->
${'Asset {0}' @ format=properties.assetName}   <!--/* Basically a shortcut of the array notation, useful when it has only one element */-->
${'Asset {0}' @ format=[properties.assetName]}
${'Asset {0} out of {1}' @ format=[properties.current, properties.total]}
${'Asset {0} out of {1}' @ format=[properties.current, properties.total], i18n, locale='de'}
```

will generate the following output

```html
Asset Night Sky
Asset Night Sky
Asset 3 out of 5
Bild 3 von 5
```

assuming that

```
properties.assetName = 'Night Sky'
properties.current   = 3
properties.total     = 5
formatter 'Asset {0} out of {1}' will be translated to 'Bild {0} von {1}' for the 'de' locale by i18n
```

##### 1.2.2.2. Dates
Date formatting supports timezones and localisation. In case internationalisation is also specified ([`i18n`](#123-i18n)), it will be applied to the formatting pattern and the locale will be passed forward to formatting.

```html
<!--/* Formatting pattern: */-->
${'yyyy-MM-dd' @ format=myDate}
${'yyyy-MM-dd' @ format=myDate, type='date'}                <!--/* Forced formatting type */-->
${'yyyy-MM-dd HH:mm' @ format=myDate, timezone='GMT+00:30'} <!--/* Timezone */-->
${'EEEE, dd MMMM yyyy' @ format=obj.date, locale='de'}      <!--/* Locale */-->
```

The formatting pattern supports, at minimum, the following letters:

* `y` - Year. Variants: `yy`, `yyyy`
* `M` - Month in year. Variants: `MM`, `MMM`, `MMMM`
* `w` - Week in year. Variants: `ww`
* `D` - Day in year. Variants: `DD`, `DDD`
* `d` - Day in month. Variants: `dd`
* `E` - Day name in week. Variants: `EEEE`
* `a` - Am/pm marker
* `H` - Hour in day (0-23). Variants: `HH`
* `h` - Hour in am/pm. Variants: `hh`
* `m` - Minute in hour. Variants: `mm`
* `s` - Second in minute. Variants: `ss`
* `S` - Millisecond. Variants: `SSS`
* `z` - General time zone
* `Z` - RFC 822 time zone
* `X` - ISO 8601 time zone. Variants: `XX`, `XXX`

All other characters from 'A' to 'Z' and from 'a' to 'z' are reserved for future possible use; if needed, they can be escaped using single quotes. Single quotes are escaped as two in a row. Other characters are not interpreted.

##### Examples

```html
${'yyyy-MM-dd HH:mm:ss.SSSXXX' @ format=obj.date, timezone='UTC'}
${'yyyy-MM-dd HH:mm:ss.SSSXXX' @ format=obj.date, timezone='GMT+02:00'}
${'yyyy-MM-dd HH:mm:ss.SSS(z)' @ format=obj.date, timezone='GMT+02:00'}
${'yyyy-MM-dd HH:mm:ss.SSSZ' @ format=obj.date, timezone='GMT+02:00'}
${'dd MMMM \'\'yy hh:mm a; \'day in year\': D; \'week in year\': w' @ format=obj.date, timezone='UTC'}
${'EEEE, d MMM y' @ format=obj.date, timezone='UTC', locale='de'}
${'EEEE, d MMM y' @ format=obj.date, timezone='UTC', locale='en_US', i18n}
```

will generate the following output for the date `1918-12-01 00:00:00Z`

```
1918-12-01 00:00:00.000Z
1918-12-01 02:00:00.000+02:00
1918-12-01 02:00:00.000(GMT+02:00)
1918-12-01 02:00:00.000+0200
01 December '18 12:00 AM; day in year: 335; week in year: 49
Sonntag, 1 Dez 1918
Sunday, Dec 1, 1918  <!--/* assuming the formatter 'EEEE, d MMM y' will be translated to 'EEEE, MMM d, y' for the 'en_US' locale by i18n */--> 
```

##### 1.2.2.3. Numbers
Number formatting supports localisation. In case internationalisation is also specified ([`i18n`](#123-i18n)), it will be applied to the formatting pattern and the locale will be passed forward to formatting.

```html
<!--/* Formatting pattern: */-->
${'#.00' @ format=42}
${'#.00' @ format=myNumber, type='number'} <!--/* Forced formatting type */-->
${'#.00' @ format=myNumber, locale='de'}   <!--/* Locale */-->
```

The formatting pattern supports both a positive and negative pattern, separated by semicolon. Each sub-pattern can have a prefix, a numeric part and a suffix. The negative sub-pattern can only change the prefix or/and suffix. The following characters are supported, at minimum:

* `0` - digit, shows as 0 if absent
* `#` - digit, does not show if absent
* `.` - decimal separator
* `-` - minus sign
* `,` - grouping separator
* `E` - separator between mantissa and exponent
* `;` - sub-pattern boundary
* `%` - multiply by 100 and show as percentage

Characters can be escaped in prefix or suffix using single quotes. Single quotes are escaped as two in a row. 

##### Examples

```html
${'#,###.00' @ format=1000}
${'#.###;-#.###' @ format=obj.number}
${'#.00;(#.00)' @ format=obj.number}
${'#.000E00' @ format=obj.number}
${'#%' @ format=obj.number}
${ 'curr #,###.##' @ format=1000.14, locale='de_CH', i18n}
```

will generate the following output if `obj.number` evaluates to `-3.14`:

```
1,000.00
-3.14
(3.14)
-.314E01
-314%
CHF 1'000.14 <!--/* assuming the formatter 'curr #,###.##' will be translated to 'CHF #,###.##' for the 'de_CH' locale by i18n */-->
```

#### 1.2.3. i18n
This option internationalises strings.

```html
${'Assets' @ i18n} <!--/* Translates the string to the resource language */-->
```

When this option is used, two more options take a special meaning:

* `locale`: When set, it overrides the language from the source. For e.g.: `en_US` or `fr_CH`
* `hint`: Allows to provide some information about the context for the translators.

```html
${'Assets' @ i18n, locale='en-US', hint='Translation Hint'}
```


#### 1.2.4. Array Join
The `join` option allows to control the output of an array object by specifying the separator string.

```html
${['one', 'two'] @ join='; '} <!--/* outputs: one; two */-->
 
<!--/* This can for e.g. be useful for setting class-names */-->
<span class="${myListOfClassNames @ join=' '}"></span>
```

Applying the `join` option to simple strings should just output the string:

```html
${'test' @ join=', '} <!--/*  outputs: test */-->
```

#### 1.2.5. URI Manipulation
URI manipulation can be performed by adding any of the following options to an expression:

* `scheme` - allows adding or removing the scheme part for a URI
  
  ```html
  ${'//example.com/path/page.html' @ scheme='http'}
  <!-- outputs: http://example.com/path/page.html -->
  
  ${'http://example.com/path/page.html' @ scheme='https'}
  <!-- outputs: https://example.com/path/page.html -->
  
  ${'http://example.com/path/page.html' @ scheme=''}
  <!-- outputs: http://example.com/path/page.html -->
  
  ${'http://example.com/path/page.html' @ scheme}
  <!-- outputs: http://example.com/path/page.html -->
  ```
  
* `domain` - allows adding or replacing the host and port (domain) part for a URI
  
  ```html
  ${'///path/page.html' @ domain='example.org'}
  <!-- outputs: //example.org/path/page.html -->
  
  ${'http:///path/page.html' @ domain='example.org'}
  <!-- outputs: http://example.org/path/page.html -->
  
  ${'http://www.example.com/path/page.html' @ domain='www.example.org'}
  <!-- outputs: http://www.example.org/path/page.html -->
  ```
  
* `path` - modifies the path that identifies a resource
* `prependPath` - prepends its content to the path that identifies a resource
* `appendPath` - appends its content to the path that identifies a resource

  
  ```html
  ${'one' @ appendPath='two'}
  <!-- outputs: one/two -->
  
  ${'/one/' @ appendPath='/two/'}
  <!-- outputs: /one/two/ -->
  
  ${'path' @ prependPath='..'}
  <!-- outputs: ../path -->
  
  ${'path' @ prependPath='/', appendPath='/'}
  <!-- outputs: /path/ -->
  
  ${'http://example.com/path/page.html' @ prependPath='foo'}
  <!-- outputs: http://example.com/foo/path/page.html -->
  
  ${'path/page.selector.html/suffix?key=value#fragment' @ appendPath='appended'}
  <!-- outputs: path/page/appended.selector.html/suffix?key=value#fragment -->
  
  ${'http://example.com/this/one.selector.html/suffix?key=value#fragment' @ path='that/two'}
  <!-- outputs: http://example.com/that/two.selector.html/suffix?key=value#fragment -->
  
  ${'http://example.com/this/one.selector.html/suffix?key=value#fragment' @ path=''}
  <!-- outputs: http://example.com/this/one.selector.html/suffix?key=value#fragment -->
  
  ${'http://example.com/this/one.selector.html/suffix?key=value#fragment' @ path}
  <!-- outputs: http://example.com/this/one.selector.html/suffix?key=value#fragment -->
  ```
  
* `selectors` - modifies or removes the selectors from a URI; the selectors are the URI segments between the part that identifies a resource (the resource's path) and the extension used for representing the resource
* `addSelectors` - adds the provided selectors (selectors string or selectors array) to the URI
* `removeSelectors` - removes the provided selectors (selectors string or selectors array) from the URI

  ```html
  ${'path/page.woo.foo.html' @ selectors='foo.bar'}
  <!-- outputs: path/page.foo.bar.html -->
  
  ${'path/page.woo.foo.html' @ selectors=['foo', 'bar']}
  <!-- outputs: path/page.foo.bar.html -->
  
  ${'path/page.woo.foo.html' @ addSelectors='foo.bar'}
  <!-- outputs: path/page.woo.foo.foo.bar.html -->
  
  ${'path/page.woo.foo.html' @ addSelectors=['foo', 'bar']}
  <!-- outputs: path/page.woo.foo.foo.bar.html -->
  
  ${'path/page.woo.foo.html' @ removeSelectors='foo.bar'}
  <!-- outputs: path/page.woo.html -->
  
  ${'path/page.woo.foo.html' @ removeSelectors=['foo', 'bar']}
  <!-- outputs: path/page.woo.html -->
  
  ${'path/page.woo.foo.html' @ selectors}
  <!-- outputs: path/page.html -->
  
  ${'path/page.woo.foo.html' @ selectors=''}
  <!-- outputs: path/page.html -->
  ```

* `extension` - adds, modifies or removes the extension from a URI

  ```html
  ${'path/page' @ extension='html'}
  <!-- outputs: path/page.html -->
  
  ${'path/page.json' @ extension='html'}
  <!-- outputs: path/page.html -->
  
  ${'path/page.selector.json' @ extension='html'}
  <!-- outputs: path/page.selector.html -->
  
  ${'path/page.json/suffix' @ extension='html'}
  <!-- outputs: path/page.html/suffix -->
  
  ${'path/page.json?key=value' @ extension='html'}
  <!-- outputs: path/page.html?key=value -->
  
  ${'path/page.json#fragment' @ extension='html'}
  <!-- outputs: path/page.html#fragment -->
  
  ${'path/page.json' @ extension}
  <!-- outputs: path/page -->
  ```
  
* `suffix` - adds, modifies or removes the suffix part from a URI; the suffix is the URI segment between the extension and the query segment
* `prependSuffix` - prepends its content to the existing suffix
* `appendSuffix` - appends its content to the existing suffix

  ```html
  ${'path/page.html' @ suffix='my/suffix'}
  <!-- outputs: path/page.html/my/suffix -->
  
  ${'path/page.html/some/suffix' @ suffix='my/suffix'}
  <!-- outputs: path/page.html/my/suffix -->
  
  ${'path/page.html?key=value' @ suffix='my/suffix'}
  <!-- outputs: path/page.html/my/suffix?key=value -->
  
  ${'path/page.html#fragment' @ suffix='my/suffix'}
  <!-- outputs: path/page.html/my/suffix#fragment -->
  
  ${'path/page.html/suffix' @ prependSuffix='prepended'}
  <!-- outputs: path/page.html/prepended/suffix -->
  
  ${'path/page.html/suffix' @ appendSuffix='appended'}
  <!-- outputs: path/page.html/suffix/appended -->
  
  ${'path/page.html/suffix' @ suffix}
  <!-- outputs: path/page.html -->
  ```
  
* `query` - adds, replaces or removes the query segment of a URI, depending on the contents of its map value
* `addQuery` - adds or extends the query segment of a URI with the contents of its map value
* `removeQuery` - removes the identified parameters from an existing query segment of a URI; its value can be a string or a string array
  
  ```html
  <!--
      assuming that jsuse.query evaluates to:
      
      {
        "query": {
          "q" : "htl",
          "array" : [1, 2, 3]
        }
      }
  -->
  
  ${'http://www.example.org/search' @ query=jsuse.query, context='uri'}
  <!-- outputs: http://www.example.org/search?q=htl&array=1&array=2&array=3 -->

  ${'http://www.example.org/search?s=1' @ addQuery=jsuse.query, context='uri'}
  <!-- outputs: http://www.example.org/search?s=1&q=htl&array=1&array=2&array=3 -->

  ${'http://www.example.org/search?s=1&q=htl' @ removeQuery='q', context='uri'}
  <!-- outputs: http://www.example.org/search?s=1 -->

  ${'http://www.example.org/search?s=1&q=htl' @ removeQuery=['s', 'q'], context='uri'}
  <!-- outputs: http://www.example.org/search -->

  ${'http://www.example.org/search?s=1&q=htl' @ query, context='uri'}
  <!-- outputs: http://www.example.org/search -->
  ```
  
* `fragment` - adds, modifies or replaces the fragment segment of a URI

  ```html
  ${'path/page' @ fragment='fragment'}
  <!-- outputs: path/page#fragment -->
  
  ${'path/page#one' @ fragment='two'}
  <!-- outputs: path/page#two -->
  
  ${'path/page#one' @ fragment}
  <!-- outputs: path/page -->
  ```

## 2. Block Statements

### 2.1. Syntax
HTL block plugins are defined by `data-sly-*` attributes set on HTML elements. Elements can have a closing tag or be self-closing. Attributes can have values (which can be static strings or expressions), or simply be boolean attributes (without a value).

```html
<tag data-sly-BLOCK></tag>                                 <!--/* A block is simply consists in a data-sly attribute set on an element. */-->
<tag data-sly-BLOCK/>                                      <!--/* Empty elements (without a closing tag) should have the trailing slash. */-->
<tag data-sly-BLOCK="string value"/>                       <!--/* A block statement usually has a value passed, but not necessarily. */-->
<tag data-sly-BLOCK="${expression}"/>                      <!--/* The passed value can be an expression as well. */-->
<tag data-sly-BLOCK="${@ myArg='foo'}"/>                   <!--/* Or a parametric expression with arguments. */-->
<tag data-sly-BLOCKONE="value" data-sly-BLOCKTWO="value"/> <!--/* Several block statements can be set on a same element. */-->
```

All evaluated `data-sly-*` attributes are removed from the generated markup.

#### 2.1.1. Identifiers
A block statement can also be followed by an identifier:

```html
<tag data-sly-BLOCK.IDENTIFIER="value"></tag>
```

The identifier can be used by the block statement in various ways, here are some examples:

```html
<!--/* Example of statements that use the identifier to set a variable with their result: */-->
<div data-sly-use.navigation="MyNavigation">${navigation.title}</div>
<div data-sly-test.isEditMode="${wcmmode.edit}">${isEditMode}</div>
<div data-sly-list.child="${currentPage.listChildren}">${child.properties.jcr:title}</div>
<div data-sly-template.nav>Hello World</div>
 
<!--/* The attribute statement uses the identifier to know to which attribute it should apply it's value: */-->
<div data-sly-attribute.title="${properties.jcr:title}"></div> <!--/* This will create a title attribute */-->
```

Top top-level identifiers are case-insensitive (because they can be set through HTML attributes which are case-insensitive), but all their properties are case-sensitive.

### 2.2. Available Block Statements

#### 2.2.1. Use
**`data-sly-use`:**
* Exposes logic to the template.
* **Element:** always shown.
* **Attribute value:** required; evaluates to `String`; the object to instantiate.
* **Attribute identifier:** optional; customised identifier name to access the instantiated logic; if an identifier is not provided, the instantiated logic will be available under the `useBean` identifier name.
* **Scope:** The identifier set by the `data-sly-use` block element is global to the script and can be used anywhere after its declaration:

    ```html
    ${customPage.foo} <!--/* this fails */-->
    <div data-sly-use.customPage="CustomPage">Hello World</div>
    ${customPage.foo} <!--/* but this works */-->
    ```


Initialises the specified logic and makes it available to the current template:

```html
<div data-sly-use.page="customPage.js">${page.foo}</div>
```

The element on which a `data-sly-use` has been set as well as its content is rendered (simply removing the `data-sly-use` attribute from the output):

```html
<div class="foo" data-sly-use.customPage="CustomPage">Hello World</div>
<!--/* outputs: */-->
<div class="foo">Hello World</div>
```

Parameters can be passed to the Use-API by using expression options:

```html
<div data-sly-use.nav="${'Navigation' @ depth=1, showVisible=!wcmmode.edit}">${nav.foo}</div>
```

More informations about how the Use-API is working can be found in the [Use-API section](#4-use-api).

The use statement can also be used to load external templates. See the [Template & Call section](#2210-template--call) for this usage.

#### 2.2.2. Text
**`data-sly-text`:**
* Sets the content for the current element.
* **Element:** always shown.
* **Content of element:** replaced with evaluated result.
* **Attribute value:** required; evaluates to `String`; the element content.
* **Attribute identifier:** none.

Content can be written either simply by writing an expression, or by specifying a `data-sly-text` attribute. This allows to annotate a designer's HTML without modifying the mock content:

```html
<p data-sly-text="${properties.jcr:title}">This text would never be shown.</p>
```

The content of the `data-sly-text` attribute is automatically XSS-protected with the `text` context, unless stated otherwise:

```html
<p data-sly-text="${'<strong>Bold and Proud</strong>' @ context='html'}"></p>
<!--/* outputs: */-->
<p><strong>Bold and Proud</strong></p>
```

Falsy variables are not treated specially, they are simply cast to strings:

```html
<p data-sly-text="${''}"></p>    <!--/* outputs: */--> <p></p>
<p data-sly-text="${[]}"></p>    <!--/* outputs: */--> <p></p>
<p data-sly-text="${0}"></p>     <!--/* outputs: */--> <p>0</p>
<p data-sly-text="${false}"></p> <!--/* outputs: */--> <p>false</p>
```

#### 2.2.3. Attribute
**`data-sly-attribute`:**
* Sets an attribute or a group of attributes on the current element.
* **Element:** always shown.
* **Content of element:** always shown.
* **Attribute value:** optional; `String` for setting attribute content, or `Boolean` for setting boolean attributes, or `Object` for setting multiple attributes; removes the attribute if the value is omitted.
* **Attribute identifier:** optional; the attribute name; must be omitted only if attribute value is an `Object`.

Attributes can be written either simply by writing an expression, or by specifying a `data-sly-attribute.*` attribute. This allows to annotate a designer's HTML without modifying the mock content:

```html
<tag class="className" data-sly-attribute.class="${myVar}"></tag> <!--/* This will overwrite the content of the class attribute */-->
<tag data-sly-attribute.data-values="${myValues}"></tag>          <!--/* This will create a data-values attribute */-->
```

The `data-sly-attribute` block element (without specifying an attribute name) allows to inject at once several attributes that have been prepared in a map object that contains key-value pairs:
```html
<input data-sly-attribute="${foobar}" type="text"/>
<!--/* outputs for instance: */-->
<input id="foo" class="bar" type="text"/>
<!--
    assuming that foobar = {'id' : 'foo', 'class' : 'bar'}
-->
```

The attribute name and content are automatically XSS-protected accordingly, unless stated otherwise:
```html
<input type="number" name="quantity" min="${qttMin @ context='number'}" max="${qttMax @ context='number'}"/>
```

Event handler attributes (`on*`) and the `style` attribute cannot be generated with `data-sly-attribute` due to the fact that none of the available display contexts can fully protect against XSS attacks given the range of values that these attributes can contain.

##### 2.2.3.1. Detailed Examples
For all examples below, consider that following object is available in the context:
```javascript
foobar = {'id': 'foo', 'class': 'bar', 'lang': ''}
```

Attributes are processed left-to-right:

```html
<div class="bar1" data-sly-attribute.class="bar2" data-sly-attribute="${foobar}"></div>
<!--/* outputs: */-->
<div id="foo" class="bar"></div>
 
<div data-sly-attribute="${foobar}" data-sly-attribute.class="bar2" id="foo2"></div>
<!--/* outputs: */-->
<div id="foo2" class="bar2"></div>
```

Empty string values lead to the removal of the attribute:

```html
<div lang="${''}"></div>
<div lang="en" data-sly-attribute.lang></div>
<div lang="en" data-sly-attribute.lang=""></div>
<div lang="en" data-sly-attribute.lang="${''}"></div>
<!--/* All of the above output: */-->
<div></div>

<div lang="en" data-sly-attribute="${foobar}"></div>
<!--/* outputs: */-->
<div id="foo" class="bar"></div>
```

Still, empty attributes are left as they are if no data-sly-attribute applies to them

```html
<div title="" data-sly-attribute="${foobar}"></div>
<!--/* outputs: */-->
<div title="" id="foo" class="bar"></div>
```

Boolean values allow to control the display of boolean attributes:

```html
<input checked="${true}"/>
<input data-sly-attribute.checked="${true}"/>
<!--/* Both output: */-->
<input checked/>
 
<input checked="${false}"/>
<input data-sly-attribute.checked="${false}"/>
<!--/* Both output: */-->
<input/>
 
<!--/* But 'true' or 'false' strings don't work the same way: */-->
<input checked="${'true'}"/>  <!--/* outputs: */--> <input checked="true"/>
<input checked="${'false'}"/> <!--/* outputs: */--> <input checked="false"/>
 
<!--/* Consider having attrs={'checked': true} */-->
<input data-sly-attribute="${attrs}"/>
<!--/* outputs: */-->
<input checked/>
```

Arrays are cast to strings:

```html
<div title="${['one', 'two', 'three']}"></div>
<!--/* outputs: */-->
<div title="one,two,three"></div>
 
<!--/* Like empty strings, empty arrays remove the attribute: */-->
<div title="${[]}"></div>
<!--/* outputs: */-->
<div></div>
 
<!--/* But an array containing just an empty string doesn't get removed: */-->
<div title="${['']}"></div>
<!--/* outputs: */-->
<div title=""></div>
```

Numbers are cast to strings (i.e. zero doesn't remove the attribute):

```html
<div class="${0}"></div>
<!--/* outputs: */-->
<div class="0"></div>
```

#### 2.2.4. Element
**`data-sly-element`:**
* Replaces the element's tag name.
* **Element:** always shown.
* **Content of element:** always shown.
* **Attribute value:** required; `String`; the element's tag name.
* **Attribute identifier:** none.

Changes the element, mostly useful for setting element tags like `h1..h6`, `th`, `td`, `ol`, `ul`.

```html
<div data-sly-element="${'h1'}">Blah</div>
<!--/* outputs: */-->
<h1>Blah</h1>
```

The element name is automatically XSS-protected with the `elementName` context, which by the way doesn't allow elements like `<script>`, `<style>`, `<form>`, or `<input>` (see the [Display Context](#121-display-context) section for the exact list).

#### 2.2.5. Test
**`data-sly-test`:**
* Keeps or removes the element depending on the attribute value.
* **Element:** shown if test evaluates to `true`.
* **Content of element:** shown if test evaluates to `true`.
* **Attribute value:** optional; evaluated as `Boolean` (but not type-cased to `Boolean` when exposed in a variable); evaluates to `false` if the value is omitted.
* **Attribute identifier:** optional; identifier name to access the result of the test.
* **Scope:** The identifier set by the `data-sly-test` block element is global to the script and can be used anywhere after its declaration:

```html
<p data-sly-test.editOrDesign="${wcmmode.edit || wcmmode.design}">displays the content when in `edit` or `design` mode</p>
<p data-sly-test="${!editOrDesign && pageProperties.jcr:title}">displays the content when in `edit` or `design` mode and the `pageProperties` contain a non-empty `jcr:title` property</p>
```

Removes the whole element from the markup if the expression evaluates to `false`.

```html
<p data-sly-test="${wcmmode.edit}">You are in edit mode</p>
<p data-sly-test>This paragraph will never display</p>
```

Note that the identifier contains the value of the condition as it was (not casting it to a `Boolean` value):

```html
<p data-sly-test.myVar="${'foo'}">${myVar}</p>
<!--/* outputs: */-->
<p>foo</p>
```

#### 2.2.6. List
**`data-sly-list`:**
* Iterates over the content of each item in the attribute value, allowing to control the iteration through the following options:
  * `begin` - iteration begins at the item located at the specified index; first item of the collection has index 0
  * `step`  - iteration will only process every step items of the collection, starting with the first one
  * `end`   - iteration ends at the item located at the specified index (inclusive)
* **Element:** shown only if the number of items from the attribute value is greater than 0, or if the attribute value is a string or number;
  when the `begin` value is used the element will be shown only if the `begin` value is smaller than the collection's size.
* **Content of element:** repeated as many times as there are items in the attribute value.
* **Attribute value:** optional; the item to iterate over; if omitted the content will not be shown.
* **Attribute identifier:** optional; customised identifier name to access the item within the list element; if an identifier is not provided, the block element will implicitly make available an `item` identifier to access the element of the current iteration.
* **Scope:** The identifier set by the `data-sly-list` block element is available only in the element's content scope. The identifier will override other identifiers with the same name available in the scope, however their values will be restored once outside of the element's scope.

Repeats the content of the element for each item of the provided object (which can be an array, or any iterable object).

```html
<!--/* By default the 'item' identifier is defined within the loop. */-->
<ul data-sly-list="${currentPage.listChildren}">
    <li>${item.title}</li>
</ul>

<!--/* This is how the name of the 'item' identifier can be customised. */-->
<ul data-sly-list.childPage="${currentPage.listChildren}">
    <li>${childPage.title}</li>
</ul>

<!--/* Iteration control; start from the beginning, stop after the first 10 elements (index 9) */-->
<ul data-sly-list="${currentPage.listChildren @ begin = 0, end = 9}">
    <li>${item.title}</li>
</ul>

<!--/* Iteration control; start from the 11th element (index 10), stop after the next 10 elements (index 19) */-->
<ul data-sly-list="${currentPage.listChildren @ begin = 10, end = 19}">
    <li>${item.title}</li>
</ul>
```

An additional `itemList` (respectively `<variable>List` in case a custom identifier/variable was defined using `data-sly-list.<variable>`) identifier is also available within the scope, with the following members:

* `index`: zero-based counter (`0..length-1`);
* `count`: one-based counter (`1..length`);
* `first`: `true` for the first element being iterated;
* `middle`: `true` if element being iterated is neither the first nor the last;
* `last`: `true` for the last element being iterated;
* `odd`: `true` if `count` is odd;
* `even`: `true` if `count` is even.

When iterating over `Map` objects, the item variable contains the key of each map item:

```html
<dl data-sly-list="${myMap}">
    <dt>key: ${item}</dt>
    <dd>value: ${myMap[item]}</dd>
</dl>
```

#### 2.2.7. Repeat
**`data-sly-repeat`:**
* Iterates over the content of each item in the attribute value and displays the containing element as many times as items in the attribute
value, allowing to control the iteration through the following options:
  * `begin` - iteration begins at the item located at the specified index; first item of the collection has index 0
  * `step`  - iteration will only process every step items of the collection, starting with the first one
  * `end`   - iteration ends at the item located at the specified index (inclusive)
* **Element:** shown only if the number of items from the attribute value is greater than 0, or if the attribute value is a string or number.
* **Content of element:** repeated as many times as there are items in the attribute value.
* **Attribute value:** optional; the item to iterate over; if omitted the containing element and its content will not be shown.
* **Attribute identifier:** optional; customised identifier name to access the item within the repeat element; if an identifier is not provided, the block element will implicitly make available an `item` identifier to access the element of the current iteration.
* **Scope:** The identifier set by the `data-sly-repeat` block element is available only in the element's scope. The identifier will override other identifiers with the same name available in the scope, however their values will be restored once outside of the element's scope.

Repeats the content of the element for each item of the provided object (which can be an array, or any iterable object).

```html
<!--/* By default the 'item' identifier is defined within the loop. */-->
<p data-sly-repeat="${resource.listChildren}">${item.text}</p>

<!--/* This is how the name of the 'item' identifier can be customised. */-->
<p data-sly-repeat.childResource="${resource.listChildren}">${childResource.text}</p>

<!--/* The 'item' identifier can be used on the defining element. */-->
<div data-sly-repeat.article="${articlesCollection}" id="${article.id}">${article.excerpt}</div>

<!--/* Iteration control; start from the beginning, stop after the first 10 elements (index 9) */-->
<div data-sly-repeat.article="${articlesCollection @ begin = 0, end = 9}" id="${article.id}">${article.excerpt}</div>
```

An additional `itemList` (respectively `<variable>List` in case a custom identifier/variable was defined using `data-sly-repeat.<variable>`) identifier is also available within the scope, with the following members:

* `index`: zero-based counter (`0..length-1`);
* `count`: one-based counter (`1..length`);
* `first`: `true` for the first element being iterated;
* `middle`: `true` if element being iterated is neither the first nor the last;
* `last`: `true` for the last element being iterated;
* `odd`: `true` if `count` is odd;
* `even`: `true` if `count` is even.

When iterating over `Map` objects, the item variable contains the key of each map item:

```html
<p data-sly-repeat="${myMap}">
    <span>key: ${item}</span>
    <span>value: ${myMap[item]}</span>
</p>
```

#### 2.2.8. Include
**`data-sly-include`:**
* Includes the output of a rendering script run with the current context.
* **Element:** always shown.
* **Content of element:** replaced with the content of the included script.
* **Attribute value:** required; the file to include.
* **Attribute identifier:** none.

Includes the output of a rendering script run with the current request context, passing back control to the current HTL script.

> Note: this is comparable to the JSP `<%@ include file="" %>`.

```html
<div data-sly-include="template.html"></div>
<div data-sly-include="template.jsp"></div>
 
<!--/* Following statements are equivalent: */-->
<div data-sly-include="template.html"></div>
<div data-sly-include="${'template.html'}"></div>
```

With an expression more options can be specified:

```html
<!--/* Manipulating the path: */-->
<div data-sly-include="${'template.html' @ appendPath='appended/path'}"></div>
<div data-sly-include="${'template.html' @ prependPath='prepended/path'}"></div>
<div data-sly-include="${@ file='template.html', prependPath='prepended/path', appendPath='appended/path'}"></div>
```

The element on which a data-sly-include has been set is ignored and not displayed:

```html
<!--/* Following will simply output the rendered content of the template, the complete <div> element will be ignored */-->
<div id="test-one" class="test-two" data-sly-include="template.html">Foo</div>
<!--/* outputs only the result of template.html */-->
```

The scope of the `data-sly-include` statement isn't passed to the template of the included resource.

#### 2.2.9. Resource
**`data-sly-resource`:**
* Includes a rendered resource.
* **Element:** always shown.
* **Content of element:** replaced with the content of the resource.
* **Attribute value:** required; the path to include.
* **Attribute identifier:** none.

Includes a rendered resource from the same server, using an absolute or relative path. An implementation should create a new request context when retrieving the output.

> Note: this is comparable to a `<jsp:include page="" />`.

```html
<section data-sly-resource="./path"></section>
```

With an expression more options can be specified:

```html
<!--/* Following statements are equivalent: */-->
<section data-sly-resource="./path"></section>
<section data-sly-resource="${'./path'}"></section>
 
<!--/* Manipulating the path: */-->
<section data-sly-resource="${'my/path' @ appendPath='appended/path'}"></section>
<section data-sly-resource="${'my/path' @ prependPath='prepended/path'}"></section>
 
<!--/* Manipulating selectors: */-->
<section data-sly-resource="${'my/path' @ selectors='selector1.selector2'}"></section>
<section data-sly-resource="${'my/path' @ selectors=['selector1', 'selector2']}"></section>
<section data-sly-resource="${'my/path' @ addSelectors='selector1.selector2'}"></section>
<section data-sly-resource="${'my/path' @ addSelectors=['selector1', 'selector2']}"></section>
<section data-sly-resource="${'my/path' @ removeSelectors='selector1.selector2'}"></section>
<section data-sly-resource="${'my/path' @ removeSelectors=['selector1', 'selector2']}"></section>
<section data-sly-resource="${'my/path' @ removeSelectors}"></section>

<!--/* Forcing the type of the rendered resource: */-->
<section data-sly-resource="${'./path' @ resourceType='my/resource/type'}"></section>
```

The scope of the `data-sly-resource` statement isn't passed to the template of the included resource.

#### 2.2.10 Template & Call
Template blocks can be used like function calls: in their declaration they can get parameters, which can then be passed when calling them. They also allow recursion.

##### 2.2.10.1 Template
**`data-sly-template`:**
* Declares an HTML block, naming it with an identifier and defining the parameters it can get.
* **Element:** never shown.
* **Content of element**: shown upon calling the template with `data-sly-call`.
* **Attribute value:** optional; an expression with only options, defining the parameters it can get.
* **Attribute identifier:** required; the template identifier to declare.
* **Scope:** The identifier set by the `data-sly-template` block element is global and available no matter if it's accessed before or after the template's definition. An identically named identifier created with the help of another block element can override the value of the identifier set by `data-sly-template`.

##### 2.2.10.2. Call
**`data-sly-call`:**
* Calls a declared HTML block, passing parameters to it.
* **Element:** always shown.
* **Content of element:** replaced with the content of the called `data-sly-template` element.
* **Attribute value:** required; an expression defining the template identifier and the parameters to pass.
* **Attribute identifier:** none.

##### 2.2.10.3. Examples
Static template that has no parameters:

```html
<template data-sly-template.one>blah</template>
<div data-sly-call="${one}"></div>
```

The scope of the `data-sly-call` statement isn't inherited by the `data-sly-template` block. To pass variables, they must be passed as parameters:

```html
<template data-sly-template.two="${@ title, resource='The resource of the parent node'}"> <!--/* Notice the usage hint on the resource parameter. */-->
    <h1>${title}</h1>
    <p>Parent: ${resource.name}</p>
</template>
<div data-sly-call="${two @ title=properties.jcr:title, resource=resource.parent}"></div>
```

When templates are located in a separate file, they can be loaded with `data-sly-use`:

```html
<div data-sly-use.lib="templateLib.html" data-sly-call="${lib.one}"></div>
<div data-sly-call="${lib.two @ title=properties.jcr:title, resource=resource.parent}"></div>
```

When some parameters are missing in a template call, that parameter would be initialised to an empty string within the template.

#### 2.2.11. Unwrap
**`data-sly-unwrap`:**
* Unwraps the element.
* **Element:** shown if expression evaluates to `false`.
* **Content of element:** always shown.
* **Attribute value:** optional; an expression evaluated as `Boolean`; defaults to `true` if the value is omitted.
* **Attribute identifier:** optional; identifier name to access the result of the test.
* **Scope:** The identifier set by the `data-sly-unwrap` block element is global to the script and can be used anywhere after its declaration.

`data-sly-unwrap` can be used to hide the element itself, only showing its content:

```html
<!--/* This will only show "Foo" (without a <div> around) if the test is true: */-->
<div data-sly-test="${myTest}" data-sly-unwrap>Foo</div>
 
<!--/* This would show a <div> around "Foo" only if the test is false: */-->
<div data-sly-unwrap="${myTest}">Foo</div>

<!--/* Scope: script global */-->
<p data-sly-unwrap.richText="${configuration.text.isRich}">${sectionA}</p>
<p data-sly-unwrap="${richText}">${sectionB}</p>
```

#### 2.2.12. Set
**`data-sly-set`:**

* Defines a new identifier with a pre-defined value.
* **Element:** always shown.
* **Content of element:** always shown.
* **Attribute value:** optional; the value to store in the provided identifier.
* **Attribute identifier:** required; identifier name to access the stored value.
* **Scope:** The identifier set by the `data-sly-set` block element is global to the script and can be used anywhere after its declaration.

```html
<span data-sly-set.profile="${user.profile}">Hello, ${profile.firstName} ${profile.lastName}!</span>
<a class="profile-link" href="${profile.url}">Edit your profile</a>
```

### 2.3. Block Statements Priority
When used on the same element, the following priority list defines how block statements are evaluated:

1. `data-sly-template`
2. `data-sly-set`, `data-sly-test`, `data-sly-use`
3. `data-sly-call`
4. `data-sly-text`
5. `data-sly-element`, `data-sly-include`, `data-sly-resource`
6. `data-sly-unwrap`
7. `data-sly-list`, `data-sly-repeat`
8. `data-sly-attribute`

When two block statements have the same priority, their evaluation order is from left to right. 

## 3. Special HTML tags

### 3.1. `<sly>`
The `<sly>` HTML tag can be used to remove the current element, allowing only its children to be displayed. Its functionality is similar to the `data-sly-unwrap` block element:

```html
<!--/* This will display only the output of the 'header' resource, without the wrapping <sly> tag */-->
<sly data-sly-resource="./header"></sly>
```

Although not a valid HTML 5 tag, the `<sly>` tag can be displayed in the final output using `data-sly-unwrap`:

```html
<sly data-sly-unwrap="${false}"></sly> <!--/* outputs: <sly></sly> */-->
```

## 4. Use-API
HTL encourages separation of concerns by not allowing business logic to mix with markup. However, business logic can be implemented through the Use-API.

### 4.1. Java Use-API
The Java Use-API can be used for loading business logic objects to be used in HTL scripts through `data-sly-use`. A Java Use-API object can be a simple POJO, instantiated by a particular implementation through the POJO's default constructor.

The Use-API POJOs can also expose a public method, called `init`, with the following signature:

```java
    /**
     * Initialises the Use bean.
     *
     * @param bindings All bindings available to the HTL scripts.
     **/ 
    public void init(javax.script.Bindings bindings);
``` 

The `bindings` map can contain objects that provide context to the currently executed HTL script that the Use-API object can use for its processing.

### 4.2. JavaScript Use-API
Use objects can also be defined with JavaScript, using the following conventions:

```javascript
/**
 * In the following example '/libs/dep1.js' and 'dep2.js' are optional
 * dependencies needed for this script's execution. Dependencies can
 * be specified using an absolute path or a relative path to this
 * script's own path.
 *
 * If no dependencies are needed the dependencies array can be omitted.
 */
use(['dep1.js', 'dep2.js'], function (Dep1, Dep2) {
    // implement processing
    
    // define this Use object's behaviour
    return {
        propertyName: propertyValue
        functionName: function () {}
    }
});
```
