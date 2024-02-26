# Oblivion

# 422 unprocessable entity
the order of the routers in the controller matters!

for example:
```python
# wrong
/{item_id}/details
/all
# here in the /all endpoint, fastapi will also ask for a {item_id} even though its not written

# correct
/all
/{item_id}/details
```



# Object is not JSON serializable

```python
json.dumps(result)
```

json.dumps() function will convert a subset of **Python objects** into a **json string**. Not all objects are convertible and you may need to create a dictionary of data you wish to expose before serializing to JSON.

# Object not iterable

Error msgs:

```bash
ValueError: [TypeError("'EntityMention' object is not iterable"), TypeError('vars() argument must have __dict__ attribute')]
```

Solution:

```python
json.dumps(result)
```

## Iterable?

```python
# Testing: check wether an Object is iterable of not: If you can see the magic method __iter__, then the data are iterable. If not, then the data are not iterable and you shouldn’t bother looping through them.
print("is watson_nlp.data_model.text_primitives.Span iterable?",dir(Span))
print("is watson_nlp.data_model.entities iterable?",dir(EntityMention))
```

# no validator found for <class 'watson_nlp.data_model.entities.EntityMention'>, see `arbitrary_types_allowed` in Config

class NLPResponse(BaseModel):
results: List[EntityMention]

--> doesnt work

# How to load local file in source code using relative path?

```python
# file location:            project/src/app/asset/location_search.csv
# current code location:    project/src/app/core/services/watson_nlp_service.py
import os

script_path = os.path.abspath(__file__)
project_root = os.path.dirname(os.path.dirname(os.path.dirname(script_path)))

file_path = os.path.join(project_root, "asset/location_search.csv")
```
