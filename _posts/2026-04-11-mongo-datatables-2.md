---
layout: post
title: "mongo-datatables 2.0"
date: 2026-04-11
description: Server-side jQuery DataTables processing with MongoDB — what changed in 2.0.
---

[mongo-datatables](https://github.com/pjosols/mongo-datatables) translates DataTables Ajax requests into MongoDB aggregation pipelines. Pagination, sorting, filtering, search, SearchPanes, SearchBuilder, and Editor (full CRUD).

```bash
pip install mongo-datatables
```

## Basic Usage

```python
from pymongo import MongoClient
from mongo_datatables import DataTables, DataField

db = MongoClient("mongodb://localhost:27017/")["mydb"]

data_fields = [
    DataField('title', 'string'),
    DataField('artist', 'string'),
    DataField('year', 'number'),
    DataField('genre', 'string'),
]

result = DataTables(db, 'albums', args, data_fields).get_rows()
```

`args` is the JSON body from a DataTables Ajax POST. Returns the standard server-side response:

```json
{
    "draw": 1,
    "recordsTotal": 1000000,
    "recordsFiltered": 4821,
    "data": [...]
}
```

## What Changed in 2.0

### Search

Global search now uses AND semantics — each word must independently match at least one searchable column. Previous versions used OR, which broadened results as you typed more words.

New field type: `keyword`. Exact equality match instead of regex. Uses a regular MongoDB index — better for categorical fields.

Comparison operators in search:

```
field:>100
field:>=2024-01-01
```

Pipe-delimited range syntax for column search:

```
2020|2024
```

Quoted phrase search via word-boundary regex.

### SearchPanes

Full server-side support with dual `total`/`count` per value. Array fields (e.g. `genre`) are unwound so individual elements appear as distinct options.

### SearchBuilder

Nested AND/OR criteria trees for string, number, date, and HTML-formatted column types.

### Editor

Full CRUD with pluggable file uploads, field-level validation, pre-hooks, dependent field handlers, and server-driven options for select/radio/checkbox fields.

```python
from mongo_datatables import Editor, DataField

data_fields = [
    DataField('title', 'string'),
    DataField('artist', 'string'),
]

editor = Editor(db, 'albums', args, data_fields)
result = editor.process()
```

### Pipeline Injection

Custom aggregation stages before `$match`:

```python
DataTables(db, 'albums', args, data_fields,
    pipeline_stages=[
        {"$lookup": {"from": "artists", "localField": "artist_id",
                     "foreignField": "_id", "as": "artist"}},
        {"$unwind": "$artist"},
    ]
).get_rows()
```

## Links

- [mongo-datatables.com](https://mongo-datatables.com) — project homepage
- [docs.mongo-datatables.com](https://docs.mongo-datatables.com) — full docs
- [Vinyl Archives](https://vinyl-archives.com) — showcase app with CRUD, SearchPanes, SearchBuilder, Editor
- [Flask Demo](https://flask-demo.net) · [Django Demo](https://django-demo.net) · [FastAPI Demo](https://fastapi-demo.net)
