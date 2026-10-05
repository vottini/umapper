# Usage

## Translating the case of dictionary keys

`translate_case` recursively converts all keys in a dict to a target case style.
The three supported styles are `Case.SNAKE`, `Case.CAMEL`, and `Case.PASCAL`.

```python
from umapper import Case, translate_case

orig = {
    'OuterField': {'inner_field': 123, 'sibling_inner_field': 321},
    'siblingField': 'Hello',
}

translate_case(orig, Case.SNAKE)
# {'outer_field': {'inner_field': 123, 'sibling_inner_field': 321}, 'sibling_field': 'Hello'}

translate_case(orig, Case.CAMEL)
# {'outerField': {'innerField': 123, 'siblingInnerField': 321}, 'siblingField': 'Hello'}

translate_case(orig, Case.PASCAL)
# {'OuterField': {'InnerField': 123, 'SiblingInnerField': 321}, 'SiblingField': 'Hello'}
```

Passing a list (or tuple) translates each element independently:

```python
translate_case([orig, orig], Case.CAMEL)
# [{'outerField': ...}, {'outerField': ...}]
```

Scalar values (numbers, strings, etc.) pass through unchanged, so it is safe to call
`translate_case` on a value whose type you don't know in advance.

---

## Turning dictionaries into objects

`convert_to_object` converts a dict into a plain object whose attributes mirror the
dictionary keys. Nested dicts and lists are converted recursively.

```python
from umapper import Case, translate_case, convert_to_object

data = {'userId': 42, 'homeAddress': {'city': 'São Paulo'}}
obj = convert_to_object(translate_case(data, Case.SNAKE))

print(obj.user_id)            # 42
print(obj.home_address.city)  # São Paulo
```

Use `translate_case` beforehand when you want a specific attribute naming style.
Without it the attribute names will match the original key casing exactly.

The resulting objects also support dict-like iteration via `.keys()`, `.values()`,
and `.items()`, so they can be passed back to umapper functions if needed.

---

## Assembling and merging dictionaries

`assemble_dicts` merges any number of positional dicts into one, translates all keys
to a target case (camelCase by default), and lets you inject extra fields as keyword
arguments.

```python
from umapper import assemble_dicts

user   = {'first_name': 'Ada', 'last_name': 'Lovelace'}
coords = {'lat': -23.5, 'lng': -46.6}

assemble_dicts(user, coords, active=True)
# {
#   'firstName': 'Ada',
#   'lastName': 'Lovelace',
#   'lat': -23.5,
#   'lng': -46.6,
#   'active': True,
# }
```

When the same key appears in multiple dicts, later arguments win.

### Controlling case translation

Pass `mapping_case` to change the target style, or `None` to skip translation entirely:

```python
assemble_dicts(user, coords, mapping_case=Case.SNAKE)
# {'first_name': 'Ada', 'last_name': 'Lovelace', 'lat': -23.5, ...}

assemble_dicts(user, coords, mapping_case=None)
# keys are left exactly as they appear in the source dicts
```

### Dropping None values

By default, keys whose value is `None` are kept. Pass `include_nones=False` to drop
them:

```python
partial = {'name': 'Ada', 'nickname': None}
assemble_dicts(partial, include_nones=False)
# {'name': 'Ada'}
```

---

## Registering custom mapping classes

By default umapper traverses standard dicts and anything that inherits from
`collections.abc.Mapping`. Some libraries provide dict-like objects that don't fit
either category. Register them with `register_mapping_class` so umapper treats them
the same as a regular dict:

```python
from umapper import register_mapping_class
register_mapping_class(MyCustomDictLike)
```

The class must expose an `.items()` method that yields `(key, value)` pairs.
