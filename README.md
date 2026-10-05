
# umapper

[![Documentation Status](https://readthedocs.org/projects/umapper/badge/?version=stable)](http://umapper.readthedocs.io/?badge=latest)
[![codecov](https://codecov.io/github/vottini/umapper/graph/badge.svg?token=N2H2WJ0SC5)](https://codecov.io/github/vottini/umapper)

A small Python utility for wrangling dictionary keys: translate between snake_case, camelCase and PascalCase, turn nested dicts into attribute-accessible objects, and merge multiple dicts in one shot.

---

## Installation

```bash
pip install umapper
```

---

## Features

- **Case translation** — recursively remap all keys in a dict (or list of dicts) to snake, camel, or pascal case
- **Object conversion** — turn nested dicts into plain objects so you can write `response.userId` instead of `response["userId"]`
- **Dict assembly** — merge several dicts at once, translate their keys, strip `None` values, and inject extra fields, all in one call

---

## Usage

### Case translation

```python
from umapper import Case, translate_case

data = {
    'OutterField': {'inner_field': 123, 'sibling_inner_field': 321},
    'siblingField': 'Hello',
}

translate_case(data, Case.SNAKE)
# {'outter_field': {'inner_field': 123, 'sibling_inner_field': 321}, 'sibling_field': 'Hello'}

translate_case(data, Case.CAMEL)
# {'outterField': {'innerField': 123, 'siblingInnerField': 321}, 'siblingField': 'Hello'}

translate_case(data, Case.PASCAL)
# {'OutterField': {'InnerField': 123, 'SiblingInnerField': 321}, 'SiblingField': 'Hello'}
```

Translation works on lists too — each element is converted independently.

Non-dict, non-list values (strings, numbers, etc.) are returned as-is.

---

### Object conversion

```python
from umapper import Case, translate_case, convert_to_object

data = {'user_id': 42, 'address': {'city': 'São Paulo'}}
obj = convert_to_object(data)

print(obj.user_id)        # 42
print(obj.address.city)   # São Paulo
```

Combine with `translate_case` when the source dict uses a different case:

```python
obj = convert_to_object(translate_case(data, Case.SNAKE))
```

---

### Assembling dicts

`assemble_dicts` merges any number of positional dicts, translates all keys to a target case (camelCase by default), and lets you inject extra fields as keyword arguments.

```python
from umapper import assemble_dicts

user   = {'first_name': 'Ada', 'last_name': 'Lovelace'}
coords = {'lat': -23.5, 'lng': -46.6}

result = assemble_dicts(user, coords, active=True)
# {
#   'firstName': 'Ada',
#   'lastName': 'Lovelace',
#   'lat': -23.5,
#   'lng': -46.6,
#   'active': True,
# }
```

**Options:**

| Parameter | Default | Description |
|-----------|---------|-------------|
| `mapping_case` | `Case.CAMEL` | Target case for all keys. Pass `None` to skip case translation. |
| `include_nones` | `True` | When `False`, keys whose value is `None` are dropped from the result. |

```python
# Strip None values and use snake_case output
result = assemble_dicts(user, coords, mapping_case=Case.SNAKE, include_nones=False)
```

---

### Registering custom mapping classes

Some libraries provide dict-like objects that don't inherit from `collections.abc.Mapping`. Register them so umapper traverses them correctly:

```python
from umapper import register_mapping_class
register_mapping_class(MyCustomDictLike)
```

---

## API reference

Full API docs are available on [Read the Docs](http://umapper.readthedocs.io/).
