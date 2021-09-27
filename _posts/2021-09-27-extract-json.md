---
layout: post
title:  "Extract JSON with PHP"
date:   2021-09-27 13:29:06 -0700
categories: coding php json
tag: coding
---

## How to extract data from JSON with PHP?

##### shamelessly copied from [stackoverflow](https://stackoverflow.com/questions/29308898/how-do-i-extract-data-from-json-with-php)
First off you have a string. JSON is not an array, an object, or a data structure. JSON is a text-based serialization format - so a fancy string, but still just a string. Decode it in PHP by using json_decode().

    $data = json_decode($json);
 
Therein you might find:

* scalars: strings, ints, floats, and bools
* nulls (a special type of its own)
* compound types: objects and arrays.

These are the things that can be encoded in JSON. Or more accurately, these are PHP's versions of the things that can be encoded in JSON.

There's nothing special about them. They are not "JSON objects" or "JSON arrays." You've decoded the JSON - you now have basic everyday PHP types.

Objects will be instances of stdClass, a built-in class which is just a generic thing that's not important here.

### Accessing object properties
You access the properties of one of these objects the same way you would for the public non-static properties of any other object, e.g. $object->property.

    $json = '
    {
        "type": "donut",
        "name": "Cake"
    }';

    $yummy = json_decode($json);

    echo $yummy->type; //donut

### Accessing array elements
You access the elements of one of these arrays the same way you would for any other array, e.g. $array[0].

    $json = '
    [
        "Glazed",
        "Chocolate with Sprinkles",
        "Maple"
    ]';

    $toppings = json_decode($json);

    echo $toppings[1]; //Chocolate with Sprinkles

Iterate over it with foreach.

    foreach ($toppings as $topping) {
        echo $topping, "\n";
    }


> Glazed
>  
> Chocolate with Sprinkles
> 
> Maple


Or mess about with any of the bazillion built-in array functions.

### Accessing nested items
The properties of objects and the elements of arrays might be more objects and/or arrays - you can simply continue to access their properties and members as usual, e.g. $object->array[0]->etc.

    $json = '
    {
        "type": "donut",
        "name": "Cake",
        "toppings": [
            { "id": "5002", "type": "Glazed" },
            { "id": "5006", "type": "Chocolate with Sprinkles" },
            { "id": "5004", "type": "Maple" }
        ]
    }';

    $yummy = json_decode($json);

    echo $yummy->toppings[2]->id; //5004

#### Passing true as the second argument to json_decode()
When you do this, instead of objects you'll get associative arrays - arrays with strings for keys. Again you access the elements thereof as usual, e.g. $array['key'].

    $json = '
    {
        "type": "donut",
        "name": "Cake",
        "toppings": [
            { "id": "5002", "type": "Glazed" },
            { "id": "5006", "type": "Chocolate with Sprinkles" },
            { "id": "5004", "type": "Maple" }
        ]
    }';

    $yummy = json_decode($json, true);

    echo $yummy['toppings'][2]['type']; //Maple

### Accessing associative array items
When decoding a JSON object to an associative PHP array, you can iterate both keys and values using the foreach (array_expression as $key => $value) syntax, eg

    $json = '
    {
        "foo": "foo value",
        "bar": "bar value",
        "baz": "baz value"
    }';

    $assoc = json_decode($json, true);
    foreach ($assoc as $key => $value) {
        echo "The value of key '$key' is '$value'", PHP_EOL;
    }

Prints

> The value of key 'foo' is 'foo value'
> 
> The value of key 'bar' is 'bar value'
>
> The value of key 'baz' is 'baz value'

### Don't know how the data is structured
Read the documentation for whatever it is you're getting the JSON from.

Look at the JSON - **where you see curly brackets {}** expect an *object*, where you see **square brackets []** expect an *array.*

Hit the decoded data with a print_r():

    $json = '
    {
        "type": "donut",
        "name": "Cake",
        "toppings": [
            { "id": "5002", "type": "Glazed" },
            { "id": "5006", "type": "Chocolate with Sprinkles" },
            { "id": "5004", "type": "Maple" }
        ]
    }';

    $yummy = json_decode($json);

    print_r($yummy);

and check the output:

    stdClass Object
    (
        [type] => donut
        [name] => Cake
        [toppings] => Array
            (
                [0] => stdClass Object
                    (
                        [id] => 5002
                        [type] => Glazed
                    )

                [1] => stdClass Object
                    (
                        [id] => 5006
                        [type] => Chocolate with Sprinkles
                    )

                [2] => stdClass Object
                    (
                        [id] => 5004
                        [type] => Maple
                    )

            )

    )

It'll tell you where you have objects, where you have arrays, along with the names and values of their members.

If you can only get so far into it before you get lost - go that far and hit that with print_r():

    print_r($yummy->toppings[0]);


    stdClass Object
    (
        [id] => 5002
        [type] => Glazed
    )

Take a look at it in this handy [interactive JSON explorer](http://array.include-once.org/).

Break the problem down into pieces that are easier to wrap your head around.

#### json_decode() returns null
This happens because either:

1. The JSON consists entirely of just that, null.
2. The JSON is invalid - check the result of json_last_error_msg or put it through something like JSONLint.
3. It contains elements nested more than 512 levels deep. This default max depth can be overridden by passing an integer as the third argument to json_decode().

If you need to change the max depth you're probably solving the wrong problem. Find out why you're getting such deeply nested data (e.g. the service you're querying that's generating the JSON has a bug) and get that to not happen.

### Object property name contains a special character
Sometimes you'll have an object property name that contains something like a hyphen - or at sign @ which can't be used in a literal identifier. Instead you can use a string literal within curly braces to address it.

    $json = '{"@attributes":{"answer":42}}';
    $thing = json_decode($json);

    echo $thing->{'@attributes'}->answer; //42

If you have an integer as property see: How to access object properties with names like integers? as reference.

### Someone put JSON in your JSON
It's ridiculous but it happens - there's JSON encoded as a string within your JSON. Decode, access the string as usual, decode that, and eventually get to what you need.

    $json = '
    {
        "type": "donut",
        "name": "Cake",
        "toppings": "[{ \"type\": \"Glazed\" }, { \"type\": \"Maple\" }]"
    }';

    $yummy = json_decode($json);
    $toppings = json_decode($yummy->toppings);

    echo $toppings[0]->type; //Glazed

### Data doesn't fit in memory
If your JSON is too large for json_decode() to handle at once things start to get tricky. See:

Processing large JSON files in PHP
How to properly iterate through a big json file

### How to sort it
See: [Reference: all basic ways to sort arrays and data in PHP.](https://stackoverflow.com/q/17364127/3942918)
