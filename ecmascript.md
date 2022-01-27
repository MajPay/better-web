## configure basic import module resulution

```html
<script type="ecmascript">
    import.configure({
        paths: {
            foo: "/path/to/foo.hash.mjs",
            bar: "/path/to/bar.hash.mjs",
        },
        depcache:{
            foo: ["bar"],
        }
    });
</script>
```

## most simple loading for common entrypoints

```html
<!-- sync -->
<script type="ecmascript" import="bar"></script>
<!-- or async for example -->
<script type="ecmascript" import="bar" async></script>
```

## with manual module initialization

```html
<!-- sync -->
<script type="ecmascript">
    import {method} from "foo";
    // i would event prefer 'from "foo" import {method}'
    method();
</script>
<!-- async / whole script block is executed as async callback -->
<script type="ecmascript" async>
    import {method} from "foo";
    import {constValue} from "bar";
    method(constValue);
</script>
```

## with function call

```html
<!-- single module -->
<script type="ecmascript">
    import("foo").then(foo => foo.method());
</script>
<!-- multiple modules -->
<script type="ecmascript">
import("foo", "bar").then((foo, bar) => {
    foo.method(bar.constValue);
});
</script>
```

## other loaders / customization

```html
<!-- customize -->
<script type="ecmascript">
    import.addResolver('json', /\.json\??.*$/, function(path, alias) {
        return fetch(path).then(response => response.json()).then(json => {default: json});
    });
    import.addResolver('text', /\.txt\??.*$/, function(path, alias) {
        return fetch(path).then(response => response.text()).then(text => {default: text});
    });
    import.addResolver('css', /\.css\??.*$/, function(path, alias) {
        // example: dev-loader: avoid caching!
        const responseHandler = (response) => {
            response.headers.set('Cache-Control', 'no-cache');
            return response.text();
        };
        return fetch(path).then(responseHandler).then(css => {
            const style = document.createElement('style');
            style.innerText = css;
            return {default: style};
        });
    });
</script>
<!-- usage -->
<script type="ecmascript">
    import config from "/path/to/config.json";
    import style from "/path/to/style.css";
    import {method} from "bar";
    document.querySelector('head').insertAdjacentElement('beforeend', style);
    method(config);
</script>
```