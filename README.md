# json-kcql
This is a small utility library allowing you to translate the shape of a JSON document.
Let's say we have the following json (A description of a Pizza):

```json
{
  "ingredients": [
    {
      "name": "pepperoni",
      "sugar": 12.0,
      "fat": 4.4
    },
    {
      "name": "onions",
      "sugar": 1.0,
      "fat": 0.4
    }
  ],
  "vegetarian": false,
  "vegan": false,
  "calories": 98,
  "fieldName": "pepperoni"
}
```

using the library one can apply to types of queries:
* to flatten it
* to retain the structure while cherry-picking and/or rename fields
The difference between the two is marked by the **_withstructure_*** keyword.
If this is missing you will end up flattening the structure.
This library is dependant on **com.datamountaineer.kcql** hence you still have to provide a **'from topic'**

Let's take a look at the flatten first. There are cases when you are receiving a nested
json and you want to flatten the structure while being able to cherry pick the fields and rename them.
Imagine we have the following JSON:
```
{
  "name": "Rick",
  "address": {
    "street": {
      "name": "Rock St"
    },
    "street2": {
      "name": "Sunset Boulevard"
    },
    "city": "MtV",
    "state": "CA",
    "zip": "94041",
    "country": "USA"
  }
}
```
Applying this SQL like syntax
```
SELECT 
    name, 
    address.street.*, 
    address.street2.name as streetName2 
FROM topic
```
the projected new JSON is:
```
{
  "name": "Rick",
  "name_1": "Rock St",
  "streetName2": "Sunset Boulevard"
}
```

There are scenarios where you might want to rename fields and maybe reorder them.
By applying this SQL like syntax on the Pizza JSON

```
SELECT 
       name, 
       ingredients.name as fieldName, 
       ingredients.sugar as fieldSugar, 
       ingredients.*, 
       calories as cals 
FROM topic 
withstructure
```
we end up projecting the first structure into this one:

```json
{
  "name": "pepperoni",
  "ingredients": [
    {
      "fieldName": "pepperoni",
      "fieldSugar": 12.0,
      "fat": 4.4
    },
    {
      "fieldName": "onions",
      "fieldSugar": 1.0,
      "fat": 0.4
    }
  ],
  "cals": 98
}
```

## Flatten rules
* you can't flatten a json containing array fields
* when flattening and the column name has already been used it will get a index appended. For example if field *name* appears twice and you don't specifically
rename the second instance (*name as renamedName*) the new json will end up containing: *name* and *name_1*

## How to use it

```scala
import JsonKcql._
val json: JsonNode= ...
json.kcql("SELECT name, address.street.name as streetName FROM topic")
```
As simple as that!

## Query Examples
You can find more examples in the unit tests, however here are a few used:
* flattening
```
//rename and only pick fields on first level
SELECT calories as C ,vegan as V ,name as fieldName FROM topic

//Cherry pick fields on different levels in the structure
SELECT name, address.street.name as streetName FROM topic

//Select and rename fields on nested level
SELECT name, address.street.*, address.street2.name as streetName2 FROM topic
```
* retaining the structure
```
//you can select itself - obviousely no real gain on this
SELECT * FROM topic withstructure 

//rename a field 
SELECT *, name as fieldName FROM topic withstructure

//rename a complex field
SELECT *, ingredients as stuff FROM topic withstructure

//select a single field
SELECT vegan FROM topic withstructure

//rename and only select nested fields
SELECT ingredients.name as fieldName, ingredients.sugar as fieldSugar, ingredients.* FROM topic withstructure


```

## Release Notes


**0.1 (2017-03-15)**

* first release

### Building

***Requires gradle 3.4.1 to build.***

To build

```bash
gradle compile
```

To test

```bash
gradle test
```


You can also use the gradle wrapper

```
./gradlew build
```

To view dependency trees

```
gradle dependencies # 
```