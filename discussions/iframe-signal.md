Iframe signal
===============
2019-11-25


Sometimes, we want to communicate with an iframe.

If so, we can use the **iframe signal technique** to do so.

The **iframe signal technique** involves a parent page hosting the iframe, and an iframe.

The idea is that the iframe signals some custom message to the parent page when the iframe is reloaded.

So for instance if the iframe contains an html form, and the user posts the form, then the iframe is reloaded.



The example below shows just that.

The trick is the **div#iframe-signal** that holds the custom message and is the communication medium between the two pages.


The parent page: **test.php**
-----------

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1">
    <title>Title</title>

    <script
            src="https://code.jquery.com/jquery-3.3.1.min.js"
            integrity="sha256-FgpCb/KJQlLNfOu91ta32o/NMZxltwRo8QtmkMRdAu8="
            crossorigin="anonymous"></script>


</head>

<body>

<h1>Hello</h1>


<iframe
        width="100%"
        height="600"
        src="/test2.php"
        id="myFrame"
>

</iframe>

<script>
    $(document).ready(function () {


        // var jFrameSignal = $('#iframe-signal');

        $("#myFrame").on("load", function () {
            var jFrameSignal = $(this).contents().find('#iframe-signal');
            if(jFrameSignal.length){
                var iframeSignalValue = jFrameSignal.attr("data-value");
                console.log(iframeSignalValue);
            }
        });
    });
</script>


</body>
</html>

```


The iframe content: **test2.php**
---------------------


```php
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1">
    <title>Title</title>

</head>

<body>

<?php


$iframeSignalValue = 'none';
if (array_key_exists("magic_value", $_POST)) {
    if ("13" === $_POST['magic_value']) {
        $iframeSignalValue = "ok";
    }
}

?>


<h1>Hey buddy, my name is Jean-Pierre</h1>

<div id="iframe-signal" data-value="<?php echo htmlspecialchars($iframeSignalValue); ?>"></div>


<form method="post">
    <input type="text" value="10" name="magic_value"/>
    <input type="submit" value="Submit with value of 13"/>
</form>

</body>
</html>

```

