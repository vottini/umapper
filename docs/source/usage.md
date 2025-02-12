# Usage

The main functionalities of umapper are:

## Recursively uniformalize case of keys

```py
>>> from umapper import Case, translate_case
>>> orig = { 'OutterField': { 'inner_field': 123, 'sibling_inner_field': 321 }, 'siblingField': "Some Text" }
>>> snake = translate_case(orig, Case.SNAKE)
>>> camel = translate_case(orig, Case.CAMEL)
>>>
>>> snake
{'outter_field': {'inner_field': 123, 'sibling_inner_field': 321}, 'sibling_field': 'Some Text'}
>>>
>>> camel
{'outterField': {'innerField': 123, 'siblingInnerField': 321}, 'siblingField': 'Some Text'}
```

## Recursively turn dictionaries into objects

```py
>>> from umapper import convert_to_object
>>> obj = umapper.convert_to_object(snake)
>>> obj.outter_field.inner_field
123
```

## Join dictionaries while translating their keys's case

```py
>>> from umapper import convert_to_object
>>> a = { 'level_one': { 'level_two': "Hello" }}
>>> b = { 'level_zero': { 'level_one': "World" }}
>>> umapper.assemble_dicts(a, b, thirty=30)
{'levelOne': {'levelTwo': 'Hello'}, 'levelZero': {'levelOne': 'World'}, 'thirty': 30}
```
