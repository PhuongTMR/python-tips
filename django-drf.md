#### 1. Be aware of QuerySet’s lazy evaluation.
Perhaps the most important aspect of the Django ORM to understand is how QuerySets work. Since QuerySets are lazily-evaluated, you can chain filter() and exclude() all day without actually hitting the database. Look out for this in order to evaluate QuerySets only when you actually need to.

- When QuerySets are evaluated:
```python
    # Iteration
    for person in Person.objects.all():
        # Some logic

    # Slicing/Indexing
    Person.objects.all()[0]

    # Pickling (i.e. serialization)
    pickle.dumps(Person.objects.all())

    # Evaluation functions
    repr(Person.objects.all())
    len(Person.objects.all())
    list(Person.objects.all())
    bool(Person.objects.all())

    # Other
    [person for person in Person.objects.all()]  # List comprehensions
    person in Person.objects.all()  # `in` checks
```

- When QuerySets are not cached:
```python
    # Not reusing evaluated QuerySets
    print([p.name for p in Person.objects.all()])  # QuerySet evaluated and cached
    print([p.name for p in Person.objects.all()])  # New QuerySet is evaluated and cached

    # Slicing/indexing unevaluated QuerySets
    queryset = Person.objects.all()
    print(queryset[0])  # Queries the database
    print(queryset[0])  # Queries the database again

    # Printing
    print(Person.objects.all())
```

- When QuerySets are cached:
```python
    # Reusing an evaluated QuerySet
    queryset = Person.objects.all()
    print([p.name for p in queryset])  # QuerySet evaluated and cached
    print([p.name for p in queryset])  # Cached results are used

    # Slicing/indexing evaluated QuerySets
    queryset = Person.objects.all()
    list(queryset)  # Queryset evaluated and cached
    print(queryset[0])  # Cache used
    print(queryset[0])  # Cache used
```

#### 2. Be aware of which model attributes are not cached.
- When Django evaluates a QuerySet, foreign-key relationships and reverse relationships are not included in the query, and thus not included in the cache, unless specified otherwise.
```python
    # Foreign-key related objects
    person = Person.objects.get(id=1)
    person.father  # foreign object is retrieved and cached
    person.father  # cached version is used
    ## Never cached
    # Callable attributes
    person = Person.objects.get(id=1)
    person.children.all()  # Database hit
    person.children.all()  # Another database hit
```

#### 3. Use select_related() and prefetch_related() when you will need foreign-key/reverse related objects.
- These tools tell Django that you actually will need these objects, so that it will go ahead and query and cache them for you. The common pitfall here is to not use these when they are needed. This results in a lot of unnecessary database queries.
```python
    # DON'T
    queryset = Person.objects.all()
    for person in queryset:
        person.father  # Foreign key relationship results in a database hit each iteration
    # DO
    queryset = Person.objects.all().select_related('father')  # Foreign key object is included in query and cached
    for person in queryset:
        person.father  # Hits the cache instead of the database
```

#### 4. Try to avoid database queries in a loop.
- This is something you will most likely run into, as trying to write clean code can often result in this pitfall. Using get() or evaluating a QuerySet in a loop can be very bad for performance. Instead, do what you can to do the database work before entering the loop.
```python
    # DON'T (contrived example)
    filtered = Person.objects.filter(first_name='Shallan', last_name='Davar')
    for age in range(18):
        person = filtered.get(age=age)  # Database query on each iteration
    # DO (contrived example)
    filtered = Person.objects.filter(  # Narrow down the QuerySet to only what you need
        first_name='Shallan',
        last_name='Davar',
        age_gte=0,
        age_lte=18,
    )
    lookup = {person.age: person for person in filtered}  # Evaluate the QuerySet and construct lookup
    for age in range(18):
        person = lookup[age]  # No database query
```

#### 5. Use iterator() to iterate through a very large QuerySet only once.
- If you know your QuerySet could be very large, and you only need to iterate over it once, it makes sense to eliminate usage of the cache in order to preserve memory and other overhead. iterator() provides just this ability.
```python
    # Save memory by not caching anything
    for person in Person.objects.iterator():
        # Some logic
```

#### 6. Do work in the database rather than in Python.
- Your database can do almost anything data-related much faster than Python can. If at all possible, do your work in the database. Django provides many tools to make this possible.
- Use filter() and exclude() for filtering:
```python
    # DON'T
    for person in Person.objects.all():
        if person.age >= 18:
            # Do something
    # DO
    for person in Person.objects.filter(age__gte=18):
        # Do something
```

- **Use F expressions:**
```python
    # DON'T
    for person in Person.objects.all():
        person.age += 1
        person.save()
    # DO
    Person.objects.update(age=F('age') + 1)
```

- **Do aggregation in the database:**
```python
# DON'T
    max_age = 0
    for person in Person.objects.all():
        if person.age > max_age:
            max_age = person.age
    # DO
    max_age = Person.objects.all().aggregate(Max('age'))['age__max']
```

#### 7. Use values() and values_list() to get only the things you need.
- values() and values_list() provide lists, dictionaries, or tuples evaluating only the fields you specify.
- **Use values():**
```python
    # DON'T
    age_lookup = {
        person.name: person.age
        for person in Person.objects.all()
    }
    # DO
    age_lookup = {
        person['name']: person['age']
        for person in Person.objects.values('name', 'age')
    }
```

- **Use values_list():**
```python
    # DON'T
    person_ids = [person.id for person in Person.objects.all()]
    # DO
    person_ids = Person.objects.values_list('id', flat=True)
```

#### 8. Use defer() and only() when you only need certain fields.
**Caveats:**
- Use these in favor of values() when you need a QuerySet instead of a list of dicts.
- May only make a difference if the fields you are excluding require a lot of processing to be converted to a Python object.

- **Use defer():**
```python
    queryset = Person.objects.defer('age')  # Imagine age is computationally expensive
    for person in queryset:
        print(person.id)
        print(person.name)
```

- **Use only():**
```python
    queryset = Person.objects.only('name')
    for person in queryset:
        print(person.name)
```

#### 9. Use count() and exists() when you don’t need the contents of the QuerySet.
Caveats:
- Only use these when you don’t need to evaluate the QuerySet for other reasons.
- **Use count():**
```python
    # DON'T
    count = len(Person.objects.all())  # Evaluates the entire queryset
    # DO
    count = Person.objects.count()  # Executes more efficient SQL to determine count
```

- **Use exists():**
```python
    # DON'T
    exists = len(Person.objects.all()) > 0
    # DO
    exists = Person.objects.exists()
```

#### 10. Use delete() and update() when possible.
- Instead of updating model instances one at a time, delete() and update() allow you to do this in bulk.
- **Use delete():**
```python
    # DON'T
    for person in Person.objects.all():
        person.delete()
    # DO
    Person.objects.all().delete()
```

- **Use update():**
```python
    # DON'T
    for person in Person.objects.all():
        person.age = 0
        person.save()
    # DO
    Person.objects.update(age=0)
```

#### 11. Use bulk_create() and bulk_update() when possible.
Caveats:
- This works a bit differently than calling create().
- Read more about it in the [Django docs](https://docs.djangoproject.com/en/3.1/ref/models/querysets/#django.db.models.query.QuerySet.bulk_create).
```python
    names = ['Jeff', 'Beth', 'Tim']
    creates = []
    for name in names:
        creates.append(
            Person(name=name, age=0)
        )
    Person.objects.bulk_create(creates)
```

- Similarly, bulk-add to many-to-many fields:
```python
    person = Person.objects.get(id=1)
    person.jobs.add(job1, job2, job3)
```

#### 12. Use foreign key values directly.
- The Django ORM automatically retrieves and caches foreign keys, so use them instead of causing a needless database query.
```python
    # DON'T
    father_id = Person.objects.get(id=1).father.id  # Causes a needless database query
    # DO
    father_id = Person.objects.get(id=1).father_id  # The foreign key is already
```

#### 13. Tips & Tricks
+ https://docs.djangoproject.com/en/3.1/
+ https://docs.djangoproject.com/en/3.1/ref/models/querysets/
+ https://www.django-rest-framework.org/
+ https://simpleisbetterthancomplex.com/tips/
+ https://www.designmycodes.com/python/useful-python-tricks.html
