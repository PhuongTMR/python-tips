---
**Some internal Utils you need to know to avoid code duplication**
```python
# ai_api/services/utils.py
Utils.safe_*()
Utils.paginator()
Utils.query_with_pagination()
Utils.chunks()
Utils.do_raw_query()
Utils.intersect()
Utils.is_intersect()
Utils.merge_list()
Utils.list_diff()
Utils.fixture_safe_load()
Utils.clean_list()
Utils.clean_list_id()
# ... A lot more in this Utils class, MUST TO READ when join project.
```

> + [ai_api/odtutils.py](https://github.com/ParadoxAi/olivia-core/blob/master/src/ai_api/odtutils.py)
> + [ai_api/parallel.py](https://github.com/ParadoxAi/olivia-core/blob/master/src/ai_api/parallel.py)
> + [ai_core/patches.py](https://github.com/ParadoxAi/olivia-core/blob/master/src/aicore/patches.py)
> + [ai_core/cache.py(memorized)](https://github.com/ParadoxAi/olivia-core/blob/master/src/aicore/cache.py#L17-L32)
> + [aicore/models/custom_anotate.py](https://github.com/ParadoxAi/olivia-core/blob/master/src/aicore/models/custom_annotate.py)
> + [aicore/models/lookups.py](https://github.com/ParadoxAi/olivia-core/blob/master/src/aicore/models/lookups.py)
> + [aicore/models/manager.py](https://github.com/ParadoxAi/olivia-core/blob/master/src/aicore/models/manager.py)
> + [aicore/models/queryset.py](https://github.com/ParadoxAi/olivia-core/blob/master/src/aicore/models/queryset.py)
