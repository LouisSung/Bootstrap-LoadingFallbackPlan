# Fallback Plan for Loading CSS and JS of Bootstrap 4  
## Shows how to load Bootstrap from local if that from remote failed  
### For Bootstrap 4.0.0 ~ 4.3.1(at least)
### Some discussions  
1. [Stack Overflow](https://stackoverflow.com/questions/25748112/load-local-bootstrap-css-and-js-if-load-fail-from-remote): load local bootstrap css and js if load fail from remote  
2. [Edd Mann](https://eddmann.com/posts/providing-local-js-and-css-resources-for-cdn-fallbacks/): Providing Local JS and CSS Resources for CDN Fallbacks  
3. [freeCodeCapm](https://www.freecodecamp.org/forum/t/using-a-fallback-code-in-case-bootstraps-cdn-is-down/160753): Using a fallback code in case bootstrap’s cdn is down
### Discussions for this project
By trial and error, I finally found out a better way to check and reload files when failed to load from remote.  
  1. It use [defer](https://stackoverflow.com/a/17914854) function to make sure jQuery is loaded.
  2. It'll try only `999 times` on loading jQuery, so that you won't waste resource on keep trying. (modify as needed)  
  3. You cannot use `document.write()` as what jQuery does to Poper.js and Bootstrap.js since it'll [delete](https://www.w3schools.com/jsref/met_doc_write.asp) all existing HTML.  
  4. Also, you cannot use `$("body").append()` as what bootstrap css does, since it [won't works](https://stackoverflow.com/questions/610995/cant-append-script-element)  
  5. And the `<script></script>` tag should be in the DOM for debugging, so use [`appendChild()`](https://stackoverflow.com/questions/610995/cant-append-script-element#comment17105918_611016) instead.  
  6. Sample HTML code is avalible in [repository](/LoadBootstrap.html), while the key part (JS code) is shown below.
```html
<script>
  // MODIFY ALL URLS FOR SRC TO YOUR OWN PATH ON LOCALHOST
  // Test if jQuery is loaded. If not, load it locally.
  window.jQuery ||
  console.log("Local jQuery") ||
  document.write('<script type="text/javascript" src="/static/js/jquery-3.3.1.slim.min.js"><\/script>');

  // First, make sure jQuery is loaded.
  function defer(method, i = 0) {   // defer method 50ms until jQuery is ready or counter exceed 999
    if (window.jQuery) { method(); }
    else if (i < 999) { setTimeout(function () { defer(method, ++i) }, 50); }
  }
  function externalJS(url, log) {
    let script = document.createElement("script"); script.type = 'text/javascript'; script.src = url;
    $("body")[0].appendChild(script);   // use appendChild() rather than append()
    if (log) { console.log(log); }
  }
  defer(function () {
      // Second, test Bootstrap.css by its body color.
      // Should be "rgb(33, 37, 41)" (#212529) if loaded. (For Bootstrap 4.0.0 ~ 4.3.1 (at least))
      $("body").css("color") === "rgb(33, 37, 41)" ||
      console.log("Local Bootstrap CSS") ||
      $("head").append('<link rel="stylesheet" href="/static/css/bootstrap.min.css">');

      // Last, test if Proper.js, Bootstrap.js and Bootstrap.css are loaded.
      window.Popper || externalJS("/static/js/popper.min.js", "Local Popper");
      window.jQuery.fn.modal || externalJS("/static/js/bootstrap.min.js", "Local Bootstrap JS");
  });
</script>
```

