# Porting from jQuery to umbrellaJS

"*Why should I port from jQuery to umbrellaJS?*"
Simply because [you might not need jQuery](http://youmightnotneedjquery.com/) to do all these fancy things.

But if you like the easy and readable syntax, **but** don't like how fat and slow jQuery is, you should

- use **zepto.js** as [drop in replacement](http://zeptojs.com/)
- port to **umbrellaJS**

**umbrellaJS** is small (3kb gzipped) and has a similar syntax as jQuery. it use native DOM elements instead of objects, this makes it fast. you can simply replace `$(...)` with `u(...)` in most cases.

## Before porting to umbrellaJS

zepto.js, umbreallyJS does not support jQuery extensions! if your scripts make heavy use of extensions you should stop reading here.

before porting an existing script to a jQuery alternative I strongly recommend to check your jQuery code to be correct.

**Check for usage of `$('.subclass', $('myclass'))`**

even if `$('.subclass', $('myclass'))` and `$('myclass').find('.subclass')` are equivalent, you should use the latter. it's more readable and easyer to port.

**Check for correct `$(element)` selection**

even if your script work, there are some pitfalls where jQuery works when you are using it in the wrong way.

**example:** you need excatly one element by class

`$('#myID')` always returns one element, more exactly the first matching element. all other selectors return ALL elements. if you want to select the first element of `$('.myclass')` you must use `$('.myclass').first()` or `$('.myclass').eq(0)`. avoid using of `$('.myclass')[0]`

### First steps

umbrellaJS can be used in parallel with jQuery, so you can start porting to umbrellaJs step by step. simply include `<script src="https://unpkg.com/umbrellajs"></script>` in your HMTL file or `// @require https://unpkg.com/umbrellajs` in your script, if you are writing a userscript.

now start with changing a simple function or statement from jQuery to umbrella by replacing `$(...)` with `u(...)`, it so simple!

## Porting tips

while porting my [enstlyer script](https://greasyfork.org/de/scripts/24244-enstyler-develop/code) from jQuery (more precise from zepto.js) to umbrellaJS I discoverd some pitfalls I want to share with you. nevertheless it was easy and its always helpfull to have the excellent [umbrellaJS documentation](https://umbrellajs.com/documentation) in a browser tab.

**Why does `.replaceWith()` not exist in umbrella?**

this should be very simple, use the umbrella `.replace()` method instead :-)
even this may confuse you because of the native javascript .replace() method, it works ...

If someone wants to stay with` .replaceWith` like in jquery and does not care about an extra function call, adding this to your script may help:

```
u.prototype.replaceWith = function(replace){
  return this.replace(replace);
};
```

**Why does `.css()` not exist in umbrella?**

I general I avoid using `.css()` to set CSS properties because it's better and faster handled by CSS rules. a valid use case for `.css()` is to read the effective CSS properties of an element. 

How ca I do this with umbrellaJS? my solution is a `getStyle()` function:
```
// alternative for jquery .css() get method
function getStyle(oElm, css3Prop){
    // FF, Chrome etc.
    if(window.getComputedStyle){
        try { return getComputedStyle(oElm).getPropertyValue(css3Prop); }
        catch (e) {}
    } else {
        // IE
        if (oElm.currentStyle){
            try { return oElm.currentStyle[css3Prop]; }
            catch (e) {}
        }
    }
    return "";
}
```
``` 
//usage examples
// umbrella
getStyle(u('.myClass').nodes[n], "border-radius");

// native DOM
getStyle(getElementsByClassName('.myClass')[n], "border-radius");
getStyle(getElementById('myID'), "border-radius");

```
see: https://www.htmlgoodies.com/html5/css/referencing-css3-properties-using-javascript.html#fbid=b2-TgWC-yGY


**`u(this)` works different like in jquery**

in contrast to jquery unmbrella does  not change the scope of the javascript `this` property in `each().` But its easy to emulate it.

simply modify jquery `.each()` loops from:

```js
$('.myremove').each(function () {
    ...
    $(this).dosomething();
    ...
});
```

to someting like this:

```js
u('.myclass').each(function (that) {
    ...
    u(that).dosomething();
    ...
});
```

umbrella provides the actually processed node as first argument to the called function, [see also .each()](https://umbrellajs.com/documentation#each). you can use any varibale name instead of `that`, but I like it because its easy to remember for jquery programmers :-)

as a bonus you get the node count as second argument, so you don't have count it yourself in case you need to know:

```js
u('.myclass').each(function (that, count) {
    ...
    alert('This is iteration # ' + count + ' of myclass'); 
    ...
});
```

**umbrella `.first()` returns native DOM element**

in jquery `.first()` returns a jquery object, but umbrella it returns a native DOM object. this has pro and con:

- pro: you can use faster native javascript DOM manipulation
- con: you can't chain an other umbrella method like in jquery
- con: be careful to select the correct DOM property/method!

**example:** how to get inner html:
```
// jquery:
$('.myclass').first().html();

// umbrella: direct access to DOM property innerHTML 
u('.myclass').first().innerHTML;
```

**power tip from umbrealla creator:** For the .first() example you can just wrap the whole thing into another u() and use .before() conveniently.

**example:** how to add html before element:
```
// jquery:
$('.myclass').first().before('<div>this is inserted before</div>');

// umbrella: use of .insertAdjacentHTML() method, DOM method  .before() requires DOM node!
u('.myclass').first().insertAdjacentHTML('beforebegin', '<div>this is inserted before</div>');

// wrong: works not as you may expect!
// insert TEXT => "&lt;div&gt,this is inserted before&lt;/div&gt,"
$('.myclass').first().before('<div>this is inserted before</div>');
```

for more information see  [.insertAdjacentHTML() ](https://developer.mozilla.org/docs/Web/API/Element/insertAdjacentHTML)

**example:** how to add DOM before element:
```
// correct use of .first().before(): create native DOM element
var enMenuButton = document.createElement('input');
enMenuButton.type = 'button';
enMenuButton.setAttribute(enID, 'myID');
enMenuButton.onclick = showmyConfig;
enMenuButton.value = 'Config ';
// append native DOM
u('.subNavMenu').first().before(enMenuButton);
```

**`return false`; does not stop iterations in umbrella `.each()` loop**

in jquery you can `return false;` to stop the iteration of an `.each()` loop
this seems to work different in umbrella, the remaining iterations are done instead of skipped

```
u('article').each(function (that,i) {
     dosometing(that);
     if (i==5) return false;
})
```

Additonal `.each()` hints from umbrella creator:
For instance, the looping is purposefully like this to follow the native elements API closer. With Arrow Functions:

```
// Native and Umbrella
['a', 'b', 'c'].forEach(item => console.log(item));
u('a').each(item => console.log(item));

// jQuery
$('a').each((i, item) => console.log(item));
```

For the .first() example you can just wrap the whole thing into another u() and use .before() conveniently.

Please feel free to put these things into a .md and put it into the root of the project and then link it either through the README.md or the documentation (or both).

**DONE**