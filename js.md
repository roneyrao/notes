can route with plain link?
===============================================
>> ECMA(European Computer Manufacturers Association);
>> TC(Technical Committee)/TG(Task Group): working unit of ECMA, identified by number;
>> TC39: for ECMAScript;
>> "ECMA-no.": the standards issued by committees;
>> ECMA-262: for ECMAScript;
>> ISO/IEC 16262: equivalent to ECMA-262; usually these standards are proposed and approved in ISO/IEC(International Electrotechnical Commission) JTC(Joint Technical Committee) 1, which also give it a number;

 * when a new version is issued, the old one is withdrawn automatically;

 * stages:
stage-0 - Strawman: just an idea, possible Babel plugin.
stage-1 - Proposal: this is worth working on.
stage-2 - Draft: initial spec.
stage-3 - Candidate: complete spec and initial browser implementations.
stage-4 - Finished: will be added to the next yearly release.

===============================================
### optimization

- batch style into class

- batch new dom in container/fragment 

- reduce doms (paging, large number records: custom scrolling/empty hidden records + indexed DB (searching))

- simple selector

- cache dom reference

- restraint offsetWidth/getComputedStyle (flush changes) 

- requestAnimationFrame for manual animation 

- event delegation

- async loop

- lazy binding

- shorten scope chain (reduce closures) - wire to `this`

- array > object indexing > switch > if/then

## prototype
- base classes' prototype is a special instance of that class;
 * Function.prototype is function which accepts any arguments and returns undefined;

- shortcut to call method as function:
	(0, host.method)();
'host.method' treated as an expression;

- function.prototype defaults to { constructor: [itself], [[Prototype]]: Object }.

#	ES6
 http://es6-features.org/

- setPrototypeOf() to change __proto__ of an instance;

- no 'strict mode' in function with non-simple paramenter list(default/rest/destructure);
	because: function arguments and body should be parsed in the same mode; 'strict mode' in body will cause re-parsing;

- default param;
 * the previous param can be used for latter ones:
 	function add(first, second = getValue(first))
 * 'arguments' always reflect the parameters passed(its length don't count default values);
 * 'arguments' is a copy like in 'strict mode';
 * pass undefined to employ default;

- rest parameter(function(arg1, ...other)) isn't  counted in function's length(named parameters);
 * if no named arguments, 'other'.length==='arguments'.length, but 'other'!=='arguments';

- spread arguments:
	fun(...argsArr)	<==>	fun.apply(null, argsArr)

- block scope(let, const);
 * doesn't hoist declaration;
 * no duplicate declaration(different from variable shadowing);
 * TDZ(temporal dead zone): when a block scope variable is declared, the code in the same or within the scope of which, accessing it(even typeof) incurs error;
 	function(){
		alert(typeof var1); //error
		if(1){
			alert(typeof var1); //error
		}
		let var1=1;
	}
 * when in global scope, value is not assigned to global object property, instead it's shadowed;
 * 'let' in loop is handled specially, a new scope is created each time, like a function invocation;
let len='len1';		--scope 1
for(let i=0, len='len2'; i<3; i++, len+='a', console.log(len)){		--scope 2
	let len='len3';		--scope3
	let str=i+len;
	console.log(str);
}
 * the variable in iteration loop(for...in / for-of) is not modified, instead a new scope is created each time.

	this is valid: `for(const key in obj){}`
 	
- function:
 * each function has a 'name' for debugging;
 * when called with new, new.target===function
 * block level function is valid(not only function expression); 
 	.) when in 'strict mode', its hoisted to the top of block;
	.) to traditional scope top if not; 
 * arrow function(fun= arg1=>arg1)
 	.) when single statement **without curly brackets**, a 'return' is prepended to it;

		**`()=>{1}` : return nothing;**

 	.) no 'this', 'super', 'arguments', 'new.target' of its own; those accessed belong to the scope in which the function resides; setting to them has no effect(bind() has no effect);
	.) can't be used as constructor, so no 'prototype';
	.) arguments conform to 'strict mode' even not in it;
	.) return {}: ()=>({})
	.) **in object/prototype, 'this' is not the instance**
 * in strict mode, tail call is optimized;

- object literal:
 * shorthand: 
 	let name='tom';
	let per={name:name}		->		let per={name}
 * concise method(no 'function'):
 	{name:'tom', say(){}}
 * variable key:
 	let key1='key1', key2='key2';
	let obj={
		[key1]:'val'
		,[key2](){}
	}

- 'Object.is()'	<=> '==='; except:
 * +0===-0	-> true;	Object.is(+0, -1) -> false
 * NaN===NaN	-> false;	Object.is(NaN, NaN) -> true

- copy property:
 * Object.assign(), adopt '=' to copy, so no accessor property is copied;
 * Object.assign(), adopt 'for in' to enumerate, so non-enumerable property is not copied;


- own property enumerating order:
numeric(ascending) -> string(appear) -> symbol(appear);
 + only applied to Reflect.ownKeys(); except for 'for in'/Object.keys()/JSON.stringify();


- string (utf16 inside)
  + unicode
    * code point is the encoded number (index in the encoding table)
    * code unit is the basic size of storage for an encoding(16 bits for utf16);
    * one char (code point) may consists of multipe code units (double 16 in utf16)
      encoded as a surrogate pair (low + high), not the code piont directly.
        - low (D800-DBFF)
        - high (DC00-DFFF)
      '🙂' (code point: '1f642', code units: 'd83d' + 'de42'
    * String.length calc number of units instead of points; incorrect for char of two units, fix:
      - String.match(/./gu).length
      - `for..of`
      - `str.replace(/[\uD800-\uDBFF][\uDC00-\uDFFF]/g, '_').length`
    * subscript is also based on code unit, for+of fixes this;
    * `charCodeAt/codePointAt` (utf16)
      - index is code unit
      - `charCodeAt` only returns one code unit
      - `codePointAt(0)` returns the full code point
      - `codePointAt(1)` truncated
 * regex matching start point:
 	.) no flag: 0
	.) 'g': lastIndex and forward
	.) 'y': lastIndex (not to forward)
  ```
regex1.test(str1);
console.log(regex1.lastIndex);
// expected output: 9
regex1.test(str1);
console.log(regex1.lastIndex);
  ```
 * [tagged] template literal (backquotes):	
	 function filterFun(parts, ...substitutes){};
		+ parts always have more than one elements than substitutes;
	.) new line is allowed;
		```
		 let str1=filterFun`abc ${expr}
				 123`;
		```
	.) filterFun is a filter function; 
		=> filterfun('abc ', expr, '\n123')
	.) expr can be any valid R-value;
	.) to evaluate at runtime: wrapped in a function: 
		function reusable(a, b) {
			  return `a is ${a} and b is ${b}`;
		}
	.) filterFun can return whatever value instead of string;
	.) string creator accepting array/spreaded arguments:
		function creatorGenerator(parts, ...indexArray){
			console.log(parts.length+','+ indexArray.length);
			return function(...values){
				return parts.reduce((accumulated, part, i)=>{
					return accumulated+values[indexArray[i-1]]+part;
				});
			}
		}
		let strCreator=creatorGenerator`123${0}456${2}789${1}`;
		let str1=strCreator('A', 'B', 'C');		//123A456C789B
	.) string creator accepting object:
		function creatorGenerator(parts, ...keyArray){
			console.log(parts.length+','+ keyArray.length);
			return function(values){
				return parts.reduce((accumulated, part, i)=>{
					return accumulated+values[keyArray[i-1]]+part;
				});
			}
		}
		let strCreator=creatorGenerator`123${'f0'}456${'f2'}789${'f1'}`;
		let str1=strCreator({f0:'A', f1:'B', f2:'C'});	//123A456C789B



- super to access prototype methods in overriden:
Object.getPrototypeOf(this).method1.call(this)	=>	super.method1();
 * only works in concise style method;
 * works in multiple level inheritance(the traditional way is not, which cause stack overflow error);

- destructuring(also applied for parameters declaration):
let node = {
	type: "Identifier",
	name: "foo",
	loc: {
		start: {
			line: 1,
			column: 1
		},
		end: {
			line: 1,
			column: 4
		}
	},
	range: [0, 1, 2, 3, 4]
};
let {type, var1='none', loc: {start: startNew}, range: [, secIndex, ...restNums], nonexist:{fd}={}} = node;
                  |						          |				        |			            |							        |
          default value;			          |				      omit;		            |							  default value(if none, error occurs)
                                    new name;					              rest all elements;

 * initializer is mandatory:
 	let {var1, var2};		--error since no value assigned;
 * when assign values, parentheses is required for object:
 	({var1}=obj);
 * value assigned should not be non-existent, if then error occurs;
 	'node' should not be undefined; 

- symbol 
 * type can't be coerced to other type, if then error occurs(toString works, no '+' etc.);
 * Symbol is not a constructor
 * when used as key, not enumerable by default (defineProperty to specify);
 * Symbol.for/keyFor to create/retrieve global symbols/keys;

- Constructor.prototype[Symbol.toStringTag] determines what returned by Object.prototype.toString.call(inst); which is also the result of Constructor.prototype.toString if not overriden;

- Set
 * unique
 * no index
 * Object.is() to check duplicate;
 * duplicate value is dropped silently;

- WeakSet (used to track references to 'object value')
 * nonobject(including null) throws error;
 * when no references (**current** reference to the object in this is weakset/map is not taken into a account) to the object, garbage collector will reclaim it when it runs(not necessarily immediately);
 * no iterator;
 * no size;
 ```
let john = { name: "John" };

let weakMap = new WeakMap();
weakMap.set(john, "...");

john = null; // overwrite the reference

// john is removed from memory!
 ```

- Map
 * not to coerce key into string as object;
 * WeakMap only applies 'weak' to key; value still can be any type;

- WEAK Collection: for 'Tag' an object
 * only able to check if an object exists (tag the object with this set);
 * no way to check if is empty;
 * automate garbage collection;
 * to iterate:
   const it = keys()/entries();
   it.next();

- Promise
 * in executor, error thrown(implicit try/catch exists) or passed to reject is passed to 'catch';
  + value returned has nonsense.
 * the rejected value, if no reject handler, is passed to 'catch', if no 'catch', then passed to global promise handler, if still none, error unhandled;
 * resolve()/reject() creates a resolved/rejected promise, passed the parameter as result to corresponding handler;
	+ if an object with 'then' method is passed, 'then' is called as the new promise's executor;
	+ 'promise' as the parameter
    * resolve() relay it as the returned promise.
    * reject() still treat it as the result to reject handler.
 * then(onResolve, onReject) / catch(onReject) returns a new promise;
	.) when error is thrown, the new promise is rejected;
 	.) returned value in both handler is passed to the new promise resolve();
	.) if a promise is returned, its result is passed;
 * in promise chain, if a reject handler exists, the following promise is fulfilled, unless error thrown in that handler as well;
 * its always async, even already resolved.
 * promise.all 
  + receives non promise item
  + wait until all items comes to non-promise.

- `then/catch` of resolved promise is added in to execution queue immediately, or it's justed cached until resolve

@ use generator to convert async to sync syntax;
 * callback(call next() in callback):
   ```
function doAct(act, ...args){
	return function(cb){
		act(...args, cb);
	}
}
function run(gen, _cb){
	let t =gen(_cb);
	function cb(err, rs){
		if(err){
			t.throw(err);
		}else{
			step(t.next(rs));
		}
	}
	function step(rs){
		if(rs.done)return;
		rs.value(cb);
	}
	step(t.next());
}
function *tasks(cb){
	let s1=yield doAct(act1,'act1 data');
	console.log('s1 got:'+s1);
	try{
		let s2=yield doAct(act2);
		console.log('s2 got:'+s2)
	}catch(err){
		return cb(err);
	}
	cb(null, 'done');
}
run(tasks, function(err, rs){
	if(err){
		console.log('tasks fail:'+err);
	}else{
		console.log('tasks done:'+rs);
	}
});

   ```

 * promise(call next() in the previous promise fulfill handler):
  ** is es7, to handle promise (converted to promise): generator `*` is replaced with 'async', yield with 'await' **
  ```
function act1(str){
	console.log('act1 arg1 is:'+str);
	return new Promise(function(resolve, reject){
		setTimeout(function(){
			resolve('act1');
			//reject('act1');
		}, 500);
	});
}
function act2(){
	console.log('act2 run');
	return new Promise(function(resolve){
		setTimeout(function(){
			resolve('act2');
		}, 500);
	});
}
function run(gen){
	let t=gen();
	function step(rs){
		let p=Promise.resolve(rs.value);
		if(rs.done)return p;
		return p.then(function(_rs){
			return step(t.next(_rs));
		}, function(err){
			return step(t.throw(err)); //give business logic/generator a chance to catch/fix;
		});
	}
	try{// catch early error;
		return step(t.next());
	}catch(err){
		return Promise.reject(err);
	}
}
function *tasks(){
	throw new Error('early error before the first task');
	try{
		let s1=yield act1('act1 data');
	}catch(err){//ignore the error
		console.log(err);
	}
	let s2=yield act2();
	return 'all tasks';
}
run(tasks).then(function(rs){
	console.log('tasks done:'+rs);
},function(err){
	console.log('tasks failed:'+err);
});
  ```

- Class
 * derived class set Symbol.species to it by default;
 * extends accept expression;
 * typeof() defined class is 'function';
 * always call 'super(args)' as the first line in constructor of derived class;
 * return value in constrcutor works as es5;
 * derived class calls base constructor if no constructors defined.
 * class properties initialization is placed in the last of contructor.

##	ES7

- exponential operator(`**`);
- Array.prototype.includes
 * NaN can be found(indexOf can't, which uses '===');


##	MODULE

- static, e.g. load at compile time
 * import(), proposed dynamic import function.

- module/script are distinguished by loading method, instead of itself;
 * script tag with type='module', whether src is present or not(inline); 
 * new Worker(path, {type:'module'});
	.) should use other module by 'import' instead of 'importScript'
	.) can load from other origin with CORS;

- path should begin with specifier: ./ ../ /

- module codes execute after document being parsed(defer attribute of 'script' tag is implicitly set), in the order of presense, even inline one precedes external;
 * 'async' is still functioning, which causes module executing immediately after being parsed;

- runs in strict mode implicitly; 

- html style comment is not allowed, which is a historic feature;

- scope is not global, then 'this' is undefined;

- import/export should resides at the outmost level, determinable by compiler; 

- import commonjs module: `import * as name from ...`

- export:
 * when declare:
 	export let name;
 * after declare:
 	export {name};
 * default exported value is placed in a field named 'default';

 * export what imported:
   * default value
    `export { default as Name } from file`

   * import field, export field;
    export {name1 as Name1, name2} from path

   * `export * from ''`  
    imported default value is **discarded**.

   * `export name from ''`  
    imported default value is exported as **property** of 'name'.

   * `export default from ''`  
    imported default value is exported as default.
 * export default:
	.) export default val1=1; (**declare is invalid: `export default let a=1`**)
	.) export {val1 as default}; (**val1 is substituted with name 'default' when `import *`**)
  .) default exported value is placed in a field named 'default'; so `import sth from` is short for `import {default as sth} from`

- import:
 * only execute once no matter how many times it's imported;
 * imort default:
	.) import defName, {va1} from path;
	.) import {default as defName, va1} from path;
 * alias:
	.) import {Name1 as NAME1, name2} from path

 * import a module just to execute its code, without reference/binding;
	import path

 * import as a whole object: 
	import * as mod1 from path

 * imported name is readonly; but can be modified inside that module, which will be reflected to binding; 

- **default value is included in `import * from` but not in `export * from`**;


## GENERAL

- cookie is inaccessible if set 'httpOnly';

- support html style comment;
 
- XMLHttpRequest send only string converted from what ever type automatically if non-string passed;

- request data type is mandatory for server parsing, or it doesn't know what data format is which the body string represents;
 * json: Content-Type:application/json; charset=utf-8
 * urlencoded(form default): Content-Type:application/x-www-form-urlencoded
 * file input type(if not set, file content will not be sent): multipart/form-data
 * plain text(html5): text/plain

- function expression:
 varFun=function(){}

- instanceof will always fail if the two parts belongs to two environment;

- isPrototpyeOf/getPrototpyeOf

- string is an readonly array; to invoke array methods on it, it's necessary to convert to array first (split('')); 

- 'forEach' accepts a second param act as the host of callback function;

- Array:
 * Array.isArray
 * forEach/map/filter can't break;
 * find/every/some terminates when result got;
 * reduce/reduceRight accept an initial accumulated value passed in the first argument, if not, its the first element;
 * copyWithin will not extend the array;
 * of(el,el...);	from(array);
 * reduce((accumulated, part, i)=>{} [,initialValue]);
 	+ if initialValue is not supplied, 'accumulated' adopts the first element as its initial value, and 'i' starts from 1;

- TypedArray is not derived from Array;
 * slice clones the buffer, subarray doesn't; 

- Object:
 * Object.create: if null is passed (undefined is not allowed), the object has no builtin members; pass Object.prorotype to create a regular object;
 * builtin property is not enumerable;
 * keys()/getOwnPropertyNames()/getOwnPropertyDescriptor()/Object.prototype.properyIsEnumerable(prop) ;
 * accessor property:
 ```
 	var obj={
		_name:'',
		get name(){
			return this._name;
		},
		set name(n){
			return this._name=n;
		}
	};
  ```
 * Object.defineProperties(obj, {
	 	x:{value:1, configurable:true},
	 	y:{
			get:function(){},
			enumerable:true
		}
	});
 * enumerate own properties: Object.keys()/entries/values();
 	.) not instance object;
	.) no builtin Symbol.iterator;

- enumeration
  + 'for in'/keys()	--enumerable(inherited);
  + in/getOwnPropertyNames()	--enumerable+unenumerable (no symbols)
    - 'in' test also check prototype;

  + getOwnPropertySymbols()	--symbol properties
  + Reflect.ownKeys()	--string+symbol

- iterator generator is a function with name with '*' prefix;
 * default iterator generator used for 'for...of' has the function name '[Symbol.iterator]';
 * types with builtin generator:
	 .) collections(array,map,set): entries(),values(),keys(); default: map(entries), array/set(values)
 * the param passed to 'next()' is returned by the previous yield, which is the first command to execute:
 	function *iteratorCreator(){
		let nextParam=yield 1;
		let nextParam+=10
		yield nextParam;
	}
	let it=iteratorCreator();
	it.next()	-> 1
	it.next(5)	-> 15
  ** the second next() pass data to the first yield. **

 * if yield is not assigned to a variable, it.throw(new Error()) will not trigger error;
 * when no 'yield' left, 'next()' returns {value:returnValue, done:true}; 'returnValue' is returned by 'return';
 * when an iterator generator is 'yield'ed, this code line is replaced with codes in the generator;
 	.) the syntax '*gen()' is only valid when following 'yield';
	function *createNumberIterator() {
		yield 1;
		yield 2;
	}
	function *createCombinedIterator() {
		yield *createNumberIterator();
		yield true;
	}
	=>
	function *createCombinedIterator() {
		yield 1;
		yield 2;
		yield true;
	}

- accessor property (defineProperty/defineProperties):
 * getter/setter can be removed as a whole by deleting the accessor property;
 * getter/setter can not coexist with 'value'
 * default values of attributes are 'false' when the property is set by defineProperty, whereas 'true' when set in the traditional form;
 * when modify an spicific attribute, others stay untouched; 
 * non-configurable
  - CAN change its writable from true to false;
  - can NOT be deleted; 
 * non-writable but configurable property can change value by defineProperty;
 * setter/getter is controlled by 'configurable';
 * setter/getter can be changed only by defineProperty;
 * 'writable' only controls data property, incurs error if coexist with getter/setter when define;
 * 'value' can be a function;

- extensibility(three functions are one way operation/can't be reversed back):
  > extensibility applied to an object, configurable + writable applied to existing properties.
 * of nonextensible object, its prototype can have new property;
 * nonextensible object can delete its property, but can't add back;
 * preventExtensibility()	-> extensible
 * seal()	-> extensible + configurable
 * freeze()	-> extensible + configurable + writable (setter is not affected)


- strict mode:
  + characteristics
    * prevent dynamic scoping
    * less error prone;
    * as a string of 'directive', which should be placed in the beginning of 'script'/'function' with other directives if any;
    ```
     function(){
       var a=1;
       'use strict' //ignored;
     }
    ```
  + rules
    * 'eval'/'arguments' are keywords, not writable;

    * no 'with';
    * 'eval' runs in a new scope;
    * 'delete' non-member name throws error, instead of ignoring;
    * non-declared variable incurs error, instead of adding to global;
    * property attributes violation throws error, instead of ignored;

    * 'this' in function(not method) is undefined, instead of global;
     var hasStrictMode = (function() { "use strict"; return this===undefined}());
    * host passed to 'call'/'apply' stay as what it is, instead of being converted(null/undefined -> global; non-object -> object);
    * 'caller'/'callee' is inaccessible;
    * 'arguments' is a copy instead of an alias;
      change value of one aurgment, the original value in arguments array stay untouched.
    * parameters can't has duplicate names, instead of accessing the first one via 'arguments';

    * json can't has duplicate names, instead of overwritten(**removed in ES6**);
    * octal literal(leading zero of decimal) throws error;
    * `function f() {}` is in block scope


- console.log supports formatting as printf;

- check window.JSON instead of JSON in IE.

```
trace = function (a, c) {
    if ($("#TRACEWINDOW").length == 0) {
        $("<div id='TRACEWINDOW' style='position:absolute;border:1px black solid;background:#eee;color:Black;left:0;bottom:0;font-size:30px;z-index:9999999'><div id='traceWin_msg' style='padding:3px;'></div><div style='background:black;clear:both;'><input type='text'  id='traceWin_cmd' value='' style='width:300px'/><input type='button'  id='traceWin_btn' value='execute'></div></div>").appendTo("body");
        $("#traceWin_btn").click(function () {
            var e = "";
            try {
                e = eval($("#traceWin_cmd").val())
            } catch (b) {
                e = b.message
            }
            if (c) e = $("#traceWin_msg").text() + "|" + e;
            $("#traceWin_msg").text(e)
        });
        $("#traceWin_cmd").keyup(function (e) {
            e.keyCode === 13 && $("#traceWin_btn").click()
        })
    }
    if (a === null || a === undefined) a = (new Date).valueOf();
    if (c) a = $("#traceWin_msg").text() + "|" + a;
    $("#traceWin_msg").text(a)
};
```



- event coordinates:
*) pageX/Y gives the coordinates relative to the <html> element in CSS pixels.
*) clientX/Y gives the coordinates relative to the viewport in CSS pixels.
*) screenX/Y gives the coordinates relative to the screen in device pixels.

- im media query, width stands for the viewport's, divice-width stands for
screen's, which is hardly used on desktop.

- two viewports:
*) layout viewport: the full content box
*) visual viewport: the size of the content visible zooming into window


- visual viewport's dimension can be gotten from window.innerWidth/height.

- layout viewport dimension is arbitary varing between different browsers:
Safari iPhone uses 980px, Opera 850px, Android WebKit 800px, and IE 974px.

- scale
  screen width / visual viewport width
  window.outerWidth(screen.width) / window.innerWidth

- in meta tag 'viewport' to controll viewports:
  + width
    layout viewport
  + initial-scale
    - implies `width=device-width`
    - by default it's set to make visual == layout

*) if layout is less than visual, it's extended; because visual can't be larger than layout.

- XMLHttpRequest doesn't support 'CONNECT, TRACE, and TRACK';
withCredential
can not read even sent.


- web component
 * get owning document when being imported: document.currentScript.ownerDocument;

- angular2
 * ChangeDetectionStrategy.OnPush: detect only when input properties change(immutable is not used);

- jsx:
 * wrap doms in parens;

## React 

### Usage
 * HMR updated Component has a diffenrent type, so the old one will be deleted/unmount.
 * set value of input to null/undefined, equal to remove value property, thus it turns to uncontrolled.
 * it's drived by `setState`.
 * functional component (function) is called every time in render.
 * component created as function(no instance / 'this') or class;
 * component and dom tag are distinguished by first letter case(Upper/lower);
 	.) when lower case is recongized as native DOM node, only supported properties is allowed;
 * component returns a single root element;
 * component is functional: parameters won't be modified; same params with same returns;
 * props: params(immutable);
 * state only available to component defined by class;
 * state: variables(mutable), used in render() for model tracking by 'setState()';
 * setState is async, read 'state' when calling it may get a staled value(other setState is cached); instead pass function to setState: (prevState, props)=>({});
    * get mutated state in componentWillUpdate;
 * setState triggers render(), if shouldComponentUpdate() returns true(always by default);
 * setState accept json, which is mixined to previous one, the ones being not overridden are not touched;
 * PureComponent apply shadow comparison in shouldComponentUpdate(); pass new object when property changed;
 * parent re-render will trigger descendents re-render;
 * 6 lifecycle events: componentWillMount/componentWillReceiveProps/(shouldComponentUpdate)/componentWillUpdate/componentDidUpdate/componentDidMount/componentWillUnmount/;
 	- componentWillReceiveProps
		..) props may be the same(parent rerender);
		..) not triggered in mounting;
  - componentWillUpdate
    ..) can not call setState.
  - load data in componentDidMount
      + remind setting initial state
  - none is called except for `componentWillMount` in server side for SSR
    + data is loaded manually from static methods
    + may check if call again fired in componentWillMount in client
 * new lifecycles (commit phase)
  - getSnapshotBeforeUpdate (obsolete componentWillUpdate)
    called before DOM udpate, its result is passed to componentDidUpdate 
  - getDerivedStateFromProps (obsolete componentWillMount & componentWillReceiveProps) 
    called after initialized and before re-rendered.
  - why obsolete in async mode
    + only componentWillMount is called in SSR, no pairs
    + may be called multiple times
      > interrupted with higher priority task, so mounting restarts and calls componentWillMount again
      <https://github.com/reactjs/reactjs.org/issues/2566>
    + delays between render phase (Wills...) and commit phase (Dids...)
 * preventDefault instead of 'return false';
 * prop.children represents passed html;
 * fetch for ajax;
 * model in template: {}
 * <button onClick={this.handleClick}>:	not-bound to 'this' by default(arrow function to overcome);
 * list of elements with same type should have a 'key' attribute with unique value for comparison efficiency;
 * event object is reused: it's refreshed after this event loop; cause event.persist() to remove it from pooling;
 * <a href={this.state.url} >a</a>:	no '' in attributes; valid in text;
 * 'class' -> 'className'
 * 'for' -> 'htmlFor'
 * `<span dangerouslySetInnerHTML={{__html:'abc'}}></span>`: `{__html:'abc'}` to set innerHTML;
 * <span style={{color:'red'}}></span>: style accepts json only;
 * <button {...attrs}>MyLabel</button>: compose attributes;
 * use '&&'/'||'/'?:' for conditional rendering;
 * controled form field bind its value to state(textarea/select both has a 'value' prop for binding); uncontroled passes itself to code for value manipulating;
	<input type="text" ref={input => this._name = input} />
 * pass dom ref upwards: pass ref callback down;
 * mixin problem:
 	.) mutated, can't reuse;
	.) name clash;
	.) 
 * ways for data flow:
 	.) component hierachy;
	.) container component
 * react-router:
  .) hashRouter will not match publicPath
  .) router listen to history, then update `match`
 	.) when component doesn't update when url changes, pass location as prop to it(which will change to cause re-render); 
		..) applied to the descendents of component which is rendered by Route, which itself already has prop of location injected;
		..) when component is not in Route, wrap in withRouter to access the location;
	.) location.key is an id;
	.) react-router-redux: save route/url in store for replaying states(normally url is a separate source along with store for component props);
	.) Switch: first matched, exclusive (when not in it routes matched are all rendered);
    * what ever child with props of 'path' or 'from';
	.) react-router-dom for browser; react-router-native for server; react-router-server for server; react-router is fundametal; 
  .) Redirect should not be rendered if not matched, or it will update and cause error 'You tried to redirect to the same route you're currently on', so always place it in Switch.
  .) if path is empty, a slash is passed.
 * context:
 	.) childContextTypes and contextTypes are necessary;
 	.) provider: childContextTypes & getChildrenContext
  .) consumer (any descendants): contextTypes (if none, context is empty)
	?) can be relayed;
 * HOC(higher-order-comp)/composition: wrap a component in another, usually renders the same one with special props;
 * ReactDOM.render() will check if rendered HTML exists already, and match them if so;
 * custom DOM attribute added in `data-*` is reserved;
 * 'key' attr is reserved, always 'undefined';
 * change root element's property: call again ReactDOM.render;
 * <Com /> returns ReactElement(whose $$typeof is a Symbol of key 'react.element') by React.createElement(), which is a virtual DOM describing React Component(type==function)/DOM element(type==string);
 * setState cause rerender even values are not changed.
 * devServer publicPath has to ends with slash
 * cloneElement to assign extra props or replace children.

* mutate children without key is bad practice, or set deleted element to null to maintain correct children mapping.
  **fiber is reused only when key and type are both matched**
1.
```
  !removed ?
  <div>
    <div key='div1' > div1 </div>
    <div key='div2' > div2 </div>
  </div>
  :
  <div>
    <div key='div2' > div2 </div>
  </div>
```
2.
```
  <div>
{
    !removed ? <div key='div1' > div1 </div> : null
}
    <div key='div2' > div2 </div>
  </div>
```

### Flowing

#### Init

 DOMRenderer (ReactFiberReconciler$1) => ReactFiberScheduler (ReactFiberBeginWork/CompleteWork/CommitWork) => ReactFiberClassComponent

#### Render

 ReactDom.render -> renderSubtreeIntoContainer -> DOMRenderer.unbatchedUpdates/DOMRenderer.updateContainer(childrenElement, fiberRoot) ->  scheduleTopLevelUpdate(fiber, childrenElement) ->  scheduleWork/scheduleWorkImpl -> requestWork(fiberRoot) -> performWork

#### requestWork

1. add this requested `root` into scheduling roots chain.

2. in batching update?
  - true
    requested update is unbatching?
      - true: performWorkOnRoot
      - false: return

  - false
    Sych?
      - true: performWork
      - false: scheduleCallbackWithExpiration -> requestIdleCallback(performAsyncWork)

#### unbatchedUpdates
in baching updates && not in unbatching updates ?
  - true
    1. flag on unbatching update
    2. invoke
  - false
    invoke.

#### scheduleWork/scheduleWorkImpl
 - find root
 - assign expirationTime to every node above in the path

#### performWork
  -> findHighestPriorityRoot                                                <---
    (find a root with `remainingExpirationTime` set to other than `NoWork`)      |
                                                                                 |
                                                                                 | (looping)
  | [nextFlushedRoot]                                                            |
  |-> performWorkOnRoot                                                      ----

#### performWorkOnRoot (wrapped in `isRendering` lock) 
  -> renderRoot
      
      -> nextUnitOfWork = createWorkInProgress()
           
      -> workLoop (by invokeGuardedCallback$1) <----------------------------------------
          -> performUnitOfWork(nextUnitOfWork) (looping through nextUnitOfWork)         |
            -> beginWork                                                                | 
            | \/ [next = child/sibling fiber]                                           |                   
            | [no next]                                                                 |
            |-> completeUnitOfWork -> completeWork                                      |
            <- [next]                                                                ---
          <- [next]
     \/ root.current.alternate
      
  -> commitRoot(alternate) 
     -> prepareForCommit => disable events
     -> commitAllHostEffects -> commitPlacement
                            |
                             -> commitDeletion
                            |
                             -> Update: commitWork
     -> root.current = alternate
     -> commitAllLifeCycle -> commitLifeCycles -> componentDidMount/componentDidUpdate (ClassComponent & Update)
                                               -> commitCallbacks (ClassComponent, HostRoot)
                                               -> commitMount (HostComponent & Update)
                           -> commitAttachRef
                           -> commitErrorHandling -> componentDidCatch (catch errors of descendents only)
     \/ alternate.expirationTime
      
  -> root.remainingExpirationTime = <return>


##### performUnitOfWork

 -> beginWork
 | (no return value)
 -> completeUnitOfWork

##### bailoutOnAlreadyFinishedWork: when no need to update, return child.alternate for next loop
 -> cloneChildFibers: attach child.alternate to this.alternate.child

##### beginWork

 -> updateHostRoot => reconcileChildren 
|
 -> mountIndeterminateComponent -> adoptClassInstance -> mountClassInstance -> finishClassComponent 
|
 -> updateClassComponent 

    -> pushContextProvider
    
    current?
    
     -> null (mount) : constructClassInstance => adoptClassInstance -> mountClassInstance -> processUpdateQueue 
    |
     -> object (update) : -> updateClassInstance

    -> finishClassComponent -> render -> reconcileChildren -> invalidateContextProvider

##### mountClassInstance 
  -> getUnmaskedContext from contextStackCursor
  -> instance.context = getMaskedContext(unmasked);
  -> componentWillMount

##### updateClassInstance: shouldUpdate? 
  -> getMaskedContext 
  -> componentWillReceiveProps 
  -> processUpdateQueue
  -> shouldComponentUpdate 
  | [true]
  |-> componentWillUpdate 
  -> flag componentDidUpdate
  -> store props & state

##### effects

- Placement: when create, insert DOM
- ContentReset: clear text
- Deletion: no longer exists.
- Ref: clear ref.
- Update 
  - ClassComponent: componentDidMount/componentDidUpdate
  - HostComponent: commitUpdate; commitMount
  - HostText: commitTextUpdate

##### commitUpdate -> updateProperties;

##### commitMount -> focus();

##### unstable_AsyncComponent/AsyncComponent: flags `unstable_isAsyncReactComponent`.

##### processUpdateQueue is called after: callComponentWillMount, callComponentWillReceiveProps,  pushHostRootContext

##### createWorkInProgress: create/return alternate

##### reconcileChildren (reconcileChildrenAtExpirationTime): update fibers based on new elements.

  current?

    -> null : mountChildFibers (reconcileChildFibers (shouldTrackSideEffects: false))
   |
    -> object: reconcileChildFibers (shouldTrackSideEffects: true)

  <- workInProgress.child


##### reconcileChildFibers: flag modified fibers in effectTag and append to return's effect list.
 - react.element

    -> reconcileSingleElement  -> [new?]
                                  - true: createFiberFromElement
                                  - false: 
                                    -> [fiber matched?]
                                      - useFiber => createWorkInProgress
                                      - deleteChild -> createFiberFromElement

    -> placeSingleChild: flag Placement if new

##### completeUnitOfWork - looping to unwind up until sibling exists, then passing back to workLoop. When the last sibling finished, it continues unwinding to parent. The process runs until root reached.
  - attach to return's effects forward-list. 

##### completeWork

  - HostComponent

    -> createInstance -> document.createElement

    -> appendAllChildren

    -> finalizeInitialChildren: autoFocus? -> setInitialProperties$1 -> setInitialDOMProperties -> setValueForProperty

    <- null | CallComponent


#### Update

 ->  setState

 ->  updater().enqueueSetState (store partialState in fiber.updateQueue) -> scheduleWork -> requestWork -> performWork

##### root.current.alternate.expirationTime
  enqueueSetState
    -> computeExpirationForFiber
    -> scheduleWork/scheduleWorkImpl(fiber, expirationTime) (update host's expirationTime - then findHighestPriorityRoot will find it again)
  
### event flow

 setInitialDOMProperties (onClick) -> ensureListeningTo (listenTo) -> trapBubbledEvent : bind `dispatchEvent` to root.


 click:

 dispatchEvent -> batchedUpdates (handleTopLevelImpl) | performWork

### Tips

- Even shouldUpdate returns false and 'render' is not called, children are still checked for update.

- React defines abstract API leaving for other renderer to implement, such as Component.updater.

- expirationTime is 1000+200ms.

- fiber.firstEffect/lastEffect is forward-list of modified children fibers.

- fiber.nextEffect points to next fiber of modified sibling.

- element is dropped after fiber is created.

- batchedUpdates: if in batched updating & rendering process, `performWork` will not be invoked.

- pull update instead of push update

- DOM._reactRootContainer (fiber.stateNode): a special plain object, a virtual instance of Component, set on rendered root DOM container.
```
  var root = {
    current: uninitializedFiber (HostRoot),
    containerInfo: containerInfo,
    pendingChildren: null,
    remainingExpirationTime: NoWork,
    isReadyForCommit: false,
    finishedWork: null,
    context: null,
    pendingContext: null,
    hydrate: hydrate,
    nextScheduledRoot: null
  };
```

- NESTED_UPDATE_LIMIT = 1000: maximum updates to prevent infinite call `setState` in componentWillUpdate or componentDidUpdate.

- fiber.alternate is a copy (they point to each other), on which current update is working on (argument named `current`).

- a fiber corresponds to a component, representing a stack frame of function, which is a unit of work.
  There are a stack system internally, consisting of valueStack & fiberStack. Anything can be pushed into & poped out of the valueStack, with corresponding fiber into fiberStack.

- fiber
  - stateNode is instance of Component/DOM
  - type: element.type
  - tag: category of element.type

- Component instance

  - updater: set in ReactFiberClassComponent.adoptClassInstance.

  - `_reactInternalFiber`

- HostComponent: native DOM

- IndeterminateComponent:
  - FunctionalComponent
  -
    ```
      function () {
        return {
          render: function () {
            return <div/>;
          }
        }
      }
    ```

    helper: React.createClass/create-react-class

    ```
      createClass({
        render: function () {
          return <div/>;
        }
      });
    ```

- adopt postMessage to invoke requestAnimationFrame to ensure it runs after next repaint (before by default).

- max frame rates is 120hz 

- '__reactInternalMemoizedMergedChildContext' of instance of context provider is merged via 'assign' with result of getChildContext() and context in parent provider. 
  * so the context in each provider contains all contexts in the ancestry above.
  * The nearest context is pointed by `contextStackCursor` (**not the one with the same name in ReactFiberHostContext**)

- pass ref to render
```
  function forwardRef(props, ref) {
    return <Child {...props} ref={ref} />;
  }

  const Comp = React.forwardRef(forwardRef);

  const refComp = React.createRef(); // points to Child;
  <Comp ref={refComp} />
```

## requestAnimationFrame

  - runs in the same rate with display device refresh rate

  - will pause when invisible

  - the argument which callback receives is timestamp when this repaint starts. The same value is passed to all callbacks.

- requestIdleCallback accepts timeout to ensure callback is invoked before the deadline.

- performance.now() returns DOMHighResTimeStamp (monotonically increasing and sub-milisecond), which is duration from `time origin`

- time origin:
 
- browser context is created

- the confirming dialog when previous page leaves is confirmed.

- webworker is created.

## redux:
https://tonyhb.gitbooks.io/redux-without-profanity
 * read: getState, write: dispatch, watch: subscribe;
 * data/store is defined initially with each property with value of type of function(reducer), each is called in 'dispatch' to get the actual value;
 	xxxxxxxxxxxxx) init value returns when no actions; 
 	.) single global store;
 	.) reducer structure is corresponding to that of store(each returns a part of the store); 
 * container is wrapper component which passes state field as property to component;
 * action is a data structure passed to reducer to initiate an action to store;
 * middleware intercepts action when passed to reducer;
 * enhancer is to enhance `createStore`, which accepts `createStore` and whatever arguments createStore accepts. Then how to create store is totally delievered to the enhancer.
 * middleware is nested functions
    1. receive store
    2. receive next middleware (the last middleware receives dispatch: compose(middlewares)(store.dispatch))
    3. reducer handler
 * reducer:
 	.) contain biz logic;
	.) normalize/flatten the shape(reference structure by id), even nested is applicable(combineReducers({fieldA:combineReducers({fieldA_A, fieldA_B})}));
		..) array of structure => array_of_id + id_struct_object;
		..) normalizr.js(transforming for API)
		..) redux-orm.js(immutable for state)
	.) separate data from biz/ui state;
 	.) immutable: return new object, instead of mutate the original/passed state if changes happen;
		..) when mutate a linkage of nested field, the inner field mutation cause a copy of outer object, then each field in linkage is a new copy; 
		..) immutable.js, object-path-immutable(support batch)
	.) time-traveling requires functional/pure/predictable: don't touch anything outside; no unique/random data generated(pass in instead);
	.) combineReducers limit the state type(plain object), parameters passed(sliced state + action), and other things;
 * react-redux: Provider(store->react), connect((sliced state)/(store methods) -> comp);
 	.) mapStateToProps(state, [origProps]): state(select from the whole) <- subscribe; origProps <- componentWillReceiveProps(shallow compare);
	.) mapDispatchToProps(embed store methods into args or action sender function, and pass in as props; execute right away)
		..) mapDispatchToProps(dispatch, [ownProps]):{actionCreator(args){}}: only 'dispatch'; helper redux.bindActionCreators()
		xxx..) {actionCreator(args)=>((dispath, getState)=>{})}: 'dispatch' and 'getState';
	.) some array/immutable functions (reduce, map...) always return new object, deep compare before return (lodash.isEqual);
 * reduce-reducers.js: the whole state is passed to all reducers (instead of pass corresponding slice as combineReducer);
 * redux-immutable.js: make redux recongize its store as an Immutable.Collection;
 * Immutable.js
	.) toJs(): always returns new object(expensive);
	.) don't mix with plain object; 
	.) never in components;
 * redux-act/redux-actions: actionCreator generator to ensure actions adhering to standard (such as FSA: Flux Standard Action); 	
 * redux-thunk: when aync and sync action creators are mixed, it's hard to remember which one is async and pass 'dispatch' to it;
	 .) when action dispatched is a function, execute it, and returns its result; other middlewares are ignored;
 *  Redux Saga or Redux Loop
 * state depends on another:
 	 .) pass in action
	 .) combine as a single
	 .) whole state as a single (the last root reducer): reduceReducers(combineReducers(...), (rootState, action)=>{})
	 .) remove from store, calc in container, but with memoization (reselect); 

 * reselect
 	.) reuse between different instances of container can memoize, if 'props' is passed, since 'props' are different;
	.) if some arguments always are new even data is not modified, replace equalityCheck(defaultMemoize) with deep compare, **if efficient** 


  * compose(fun1, fun2, ...): args to the last, then result to the left, left, ...

  return funcs.reduce(function (a, b) {
    return function () {
      return a(b.apply(undefined, arguments));
    };
  });

-----------
    (function () {
      return (function () {
        return a(b.apply(undefined, arguments));
      })(c.apply(undefined, arguments));
    })(d.apply(undefined, arguments));
-----------

- fetch(https://developer.mozilla.org/en-US/docs/Web/API/Fetch_API/Using_Fetch);
  + mode
    - cors(default): dictate browser, whether to send, check access control header from server;
    - no-cors: one way, response even error inaccessible.
 * stream: response.body - ReadableStream
 * cache
 * include cookie: credentials:'include'
  `Access-Control-Allow-Credentials`
 * http error status is not rejected; 
 * redirect is not accessible in case private data is contained in target url.
 * shortcuts
  + abort
  + progress

- response type
- error: 

  if SOP is violated, this type is returned.

- Access-Control-Expose-Headers

- observable stream supply values and call handlers multiple times, whereas promise only once; 
 * promise:
	.) returns a single value
	.) not cancellable
 * observable
	.) works with multiple values over time
	.) cancellable
	.) supports map, filter, reduce and similar operators
	.) proposed feature for ES 2016
	.) use Reactive Extensions (RxJS)
	.) an array whose items arrive asynchronously over time

- immutable.js (Persistent data structures; structural sharing)
 * access fields via array: {fieldA:{fieldB:{fieldC:1}}}	-> ['fieldA','fieldB','fieldC']
 * employ 'Seq' or 'withMutations' to apply batch mutation to emit once;

- response can be read only once, so clone it before read for later use:
	cache.put(key, response.clone());

- File of input file is of type Blob;

- add polyfill as webpack entry as shortcut;

- tools:
 * watch(to execute cmd): https://github.com/mikeal/watch


- access `data-*`:
 * `dataset[*]`
 * `getAttribute('data-*')`

- 'switch' adopt '===';

- arguments validation in caller or callee;

- when convert to number implicitly, 'valueOf' is called;

	let n={valueOf:()=>1};
	n+1	=> 2;

- touchmove-preventDefault() to overcome that 'pointermove' is 'pointercancel'ed;	

- download file via ajax:
let xhr=new XMLHttpRequest();
xhr.open('GET', '/tshark.pdf');
xhr.responseType='arraybuffer';
xhr.onload=function(){
	if(xhr.response){
		let blob=new Blob([xhr.response], {type:'application/octet-stream'});
		let a= document.createElement('a');
		a.download='file.pdf';
		a.href=URL.createObjectURL(blob);
		document.body.appendChild(a);
	}
}
xhr.send();

- instanceof check if one of __proto__ in the chain is equal to that class/function's prorotype
  `__proto__` is not accessible anymore (removed)

- ToInt32: adopts `floor`

- stopImmediatePropagation, stop firing other handlers even on the same nodes (IE9+)

## regexp

- single line mode
  not supported. work around: `[\s\S]` instead of `.`.

- backreference, whole match `$&`

- regexp
 * lookahead:	?=, ?! 
 * lookbehind:	`?<=`, `?<! `
 * non-capture:	`(?:.*)`
 * non-greedy: ? after `+/*`

- `[1^23]` - `^` itself instead of negation when not at the beginning.


## react
- useEffect
  + componentWillMount (only once)
    `useEffect(fun, [])`
  + skip the first time
    ```
  const firstUpdate = useRef(true);
  useLayoutEffect(() => {
    if (firstUpdate.current) {
      firstUpdate.current = false;
      return;
    }

    console.log("componentDidUpdateFunction");
  });
    ```

- ref
  + won't rerender
  + call setState when mutated, or error occurs
    `if (this.state.value !== newValue) this.setState({value: newValue})`

## new Function
- scope is global

## throw
traditionally, it can only appear in a single statement as
`;throw 'abc';`
- a proposal to support placing in an expression
<https://github.com/tc39/proposal-throw-expressions>
`() => throw 'abc'`
`a ? b : throw 'abc'`

## onFocus/onBlur for non-default elements
  Elements who don't have these events by default can work when
- have `tabindex`
  `tabindex=-1` to prevent being really focused by tab key
- contentEditable 
