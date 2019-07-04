bigint-json-native
===========

[![NPM](https://nodei.co/npm/bigint-json-native.png?downloads=true&stars=true)](https://nodei.co/npm/bigint-json-native/)

JSON.parse/stringify with bigints support. Based on Douglas Crockford [JSON.js](https://github.com/douglascrockford/JSON-js) package and [bignumber.js](https://github.com/MikeMcl/bignumber.js) library.

While most JSON parsers assume numeric values have same precision restrictions as IEEE 754 double, JSON specification _does not_ say anything about number precision. Any floating point number in decimal (optionally scientific) notation is valid JSON value. It's a good idea to serialize values which might fall out of IEEE 754 integer precision as strings in your JSON api, but `{ "value" : 9223372036854775807}`, for example, is still a valid RFC4627 JSON string, and in most JS runtimes the result of `JSON.parse` is this object: `{ value: 9223372036854776000 }`

==========

example:

```js
var JSONbig = require('bigint-json-native');

var json = '{ "value" : 9223372036854775807, "v2": 123 }';
console.log('Input:', json);
console.log('');

console.log('node.js bult-in JSON:')
var r = JSON.parse(json);
console.log('JSON.parse(input).value : ', r.value.toString());
console.log('JSON.stringify(JSON.parse(input)):', JSON.stringify(r));

console.log('\n\nbig number JSON:');
var r1 = JSONbig.parse(json);
console.log('JSON.parse(input).value : ', r1.value.toString());
console.log('JSON.stringify(JSON.parse(input)):', JSONbig.stringify(r1));
```

Output:

```
Input: { "value" : 9223372036854775807, "v2": 123 }

node.js bult-in JSON:
JSON.parse(input).value :  9223372036854776000
JSON.stringify(JSON.parse(input)): {"value":9223372036854776000,"v2":123}


big number JSON:
JSON.parse(input).value :  9223372036854775807
JSON.stringify(JSON.parse(input)): {"value":9223372036854775807,"v2":123}
```
### Options
The behaviour of the parser is somewhat configurable through 'options'

#### options.strict, boolean, default false
Specifies the parsing should be "strict" towards reporting duplicate-keys in the parsed string.
The default follows what is allowed in standard json and resembles the behavior of JSON.parse, but overwrites any previous values with the last one assigned to the duplicate-key.

Setting options.strict = true will fail-fast on such duplicate-key occurances and thus warn you upfront of possible lost information.

example:
```js
var JSONbig = require('bigint-json-native');
var JSONstrict = require('bigint-json-native')({"strict": true});

var dupkeys = '{ "dupkey": "value 1", "dupkey": "value 2"}';
console.log('\n\nDuplicate Key test with both lenient and strict JSON parsing');
console.log('Input:', dupkeys);
var works = JSONbig.parse(dupkeys);
console.log('JSON.parse(dupkeys).dupkey: %s', works.dupkey);
var fails = "will stay like this";
try {
    fails = JSONstrict.parse(dupkeys);
    console.log('ERROR!! Should never get here');
} catch (e) {
    console.log('Succesfully catched expected exception on duplicate keys: %j', e);
}
```

Output
```
Duplicate Key test with big number JSON
Input: { "dupkey": "value 1", "dupkey": "value 2"}
JSON.parse(dupkeys).dupkey: value 2
Succesfully catched expected exception on duplicate keys: {"name":"SyntaxError","message":"Duplicate key \"dupkey\"","at":33,"text":"{ \"dupkey\": \"value 1\", \"dupkey\": \"value 2\"}"}

```

#### options.storeAsString, boolean, default false
Specifies if BigInts should be stored in the object as a string, rather than the default BigNumber.

Note that this is a dangerous behavior as it breaks the default functionality of being able to convert back-and-forth without data type changes (as this will convert all BigInts to be-and-stay strings).

example:
```js
var JSONbig = require('bigint-json-native');
var JSONbigString = require('bigint-json-native')({"storeAsString": true});
var key = '{ "key": 1234567890123456789 }';
console.log('\n\nStoring the BigInt as a string, instead of a BigNumber');
console.log('Input:', key);
var withInt = JSONbig.parse(key);
var withString = JSONbigString.parse(key);
console.log('Default type: %s, With option type: %s', typeof withInt.key, typeof withString.key);

```

Output
```
Storing the BigInt as a string, instead of a BigNumber
Input: { "key": 1234567890123456789 }
Default type: object, With option type: string

```


#### options.forceBig, boolean, default false
Specifies if all numbers should be stored as BigNumber.

Note that this is a dangerous behavior as it breaks the default functionality of being able to convert back-and-forth without data type changes (as this will convert all Numbers to be-and-stay BigNumbers).


#### options.BigNumber, function, default BigInteger
Specifies a function which is used to convert strings to big numbers.

Note that this is a dangerous behavior as it is applied only to **parsing**,
not to **stringifying**.
If you want to stringify your objects back, you must provide them with `.toJSON()` method.


#### options.noNew, boolean, default false
Toggles whether a function, which is used to convert strings to big numbers,
should be called with `new`.
It is needed for native `BigInt`s support since `BigInt()` function cannot be called with `new`.


### Links:
- [RFC4627: The application/json Media Type for JavaScript Object Notation (JSON)](http://www.ietf.org/rfc/rfc4627.txt)
- [Re: \[Json\] Limitations on number size?](http://www.ietf.org/mail-archive/web/json/current/msg00297.html)
- [Is there any proper way to parse JSON with large numbers? (long, bigint, int64)](http://stackoverflow.com/questions/18755125/node-js-is-there-any-proper-way-to-parse-json-with-large-numbers-long-bigint)
- [What is JavaScript's Max Int? What's the highest Integer value a Number can go to without losing precision?](http://stackoverflow.com/questions/307179/what-is-javascripts-max-int-whats-the-highest-integer-value-a-number-can-go-t)
- [Large numbers erroneously rounded in Javascript](http://stackoverflow.com/questions/1379934/large-numbers-erroneously-rounded-in-javascript)

### Native BigInt support

#### Stringifying
Full support out-of-the-box, stringifies BigInts as pure numbers (no quotes, no `n`).

#### Parsing
```js
var JSONbigString = require('bigint-json-native')({"BigNumber": BigInt, "noNew": true});
```
If you want to force all numbers to be parsed as `BigInt`s
(you probably do! Otherwise any calulations become a real headache):
```js
var JSONbigString = require('bigint-json-native')({"BigNumber": BigInt, "noNew": true, "forceBig": true});
```
You should take care of `BigInt` to be defined.
No built-in checks for this are implemented.
