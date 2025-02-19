LuaJS
=====

Lua VM running in Javascript (using emscripten)

Build instructions
------------------

Run `emmake make && emmake make install` then you have the compiled files in the dist folder.

Usage of Lua from JavaScript
----------------------------

```javascript
LuaJS.addEventListener("ready", () => {
    const L = new LuaJS.State();
    let value;
    value = L.run("return 1+2"); //value == [3]

    value = L.run("return {a = 1, b = 2}"); //value[0] instanceof LuaJS.Table, value[0] instanceof LuaJS.Reference
    value[0].get("a"); // == 1
    console.log(value);
    console.log(value[0].toObject(true, true)); //converts LuaJS.Table to JavaScript object (will drop all other LuaJS.Reference-s if unrefAll == true)
    value[0].unref();

    let func = L.run("return function(a,b) return a + b end"); //func[0] instanceof LuaJS.Function, func[0] instanceof LuaJS.Reference
    value = func[0].call(3,4); //value == [7]
    console.log(value);
    func[0].unref();
});
```

Using `.unref()` is optional as LuaJS does utilize Finalizers. However, those can be called at any time, so if you are memory constrained, manual unref might help!

Usage of JavaScript from Lua
----------------------------
Lua knows the library `js`, where the `js.global` table equals the JavaScript `window` object

When you call JS functions from Lua, the function parameters will always be automatically converted to JS equivalents (and internally .unref()'d, except functions).

You can convert JS objects/arrays to native Lua tables by using jsObject:toTable(recursive), however you can also directly index JS objects from Lua.

Warning: You need to call all JS functions either like `js.global:alert("testmessage")` or `local alert = js.global.alert; alert(nil, "testmessage")`. The first argument will be used as the "this" context in JavaScript.

Using Lua inline in HTML
------------------------

If you call `enableLuaScriptTags(document)`, you can specify `<script type="text/lua">` tags just like JavaScript tags with either inline scripts or a `src` attribute.

Below is an example HTML document that enables Lua scripts for the entire page:

```html
<!DOCTYPE html>
<html>
    <head>
        <script type="text/javascript" src="luajs.js"></script>
        <script type="text/javascript">
            LuaJS.addEventListener("ready", () => {
                const L = new LuaJS.State();
                L.enableLuaScriptTags(document);
            });
        </script>
        <script type="text/lua">
            js.global.console:log("Hello world")
        </script>
    </head>
</html>
```
