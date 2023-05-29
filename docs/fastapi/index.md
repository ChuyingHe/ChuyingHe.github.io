# Bug recording

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

