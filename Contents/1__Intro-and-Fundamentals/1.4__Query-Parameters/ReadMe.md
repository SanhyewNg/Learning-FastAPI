## 1.4. Query Parameters

### Query Parameters Basics

When you declare other function parameters that are not part of the path parameters, 
they are automatically interpreted as "query" parameters.

```Python 3.8+
from fastapi import FastAPI

app = FastAPI()

fake_items_db = [
    {"item_name": "Foo"}, 
    {"item_name": "Bar"}, 
    {"item_name": "Baz"},
]

@app.get("/items/")
async def read_item(skip: int = 0, limit: int = 10):
    return fake_items_db[skip : skip + limit]
```

The query is the set of key-value pairs 
that go after the `?` in a URL, separated by `&` characters.

For example, in the URL:
```
http://127.0.0.1:8000/items/?skip=0&limit=10
```
...the query parameters are:
- `skip`: with a value of `0`
- `limit`: with a value of `10`

As they are part of the URL, they are "naturally" strings.

But when you declare them with Python types (in the example above, as `int`), 
they are converted to that type and validated against it.

All the same process that applied for path parameters also applies for query parameters:

- Editor support (obviously)
- Data "parsing"
- Data validation
- Automatic documentation


#### Defaults

As query parameters are not a fixed part of a path, 
they can be optional and can have default values.

In the example above they have default values of `skip=0` and `limit=10`.

So, going to the URL:
```
http://127.0.0.1:8000/items/
```
would be the same as going to:
```
http://127.0.0.1:8000/items/?skip=0&limit=10
```

But if you go to, for example:
```
http://127.0.0.1:8000/items/?skip=20
```

the parameter values in your function will be:
- `skip=20`: because you set it in the URL
- `limit=10`: because that was the default value


#### Optional parameters

The same way, you can declare optional query parameters, by setting their default to `None`:

```Python 3.10+
from fastapi import FastAPI

app = FastAPI()

@app.get("/items/{item_id}")
async def read_item(item_id: str, q: str | None = None):
    if q:
        return {"item_id": item_id, "q": q}
    return {"item_id": item_id}
```

In this case, the function parameter `q` will be optional, and will be `None` by default.

**Check**: 
    Also notice that FastAPI is smart enough to notice that 
    the path parameter `item_id` is a path parameter and `q` is not, so, it's a query parameter.


#### Query parameter type conversion

You can also declare bool types, and they will be converted:

```Python 3.10+
from fastapi import FastAPI

app = FastAPI()

@app.get("/items/{item_id}")
async def read_item(item_id: str, q: str | None = None, short: bool = False):
    item = {"item_id": item_id}
    if q:
        item.update({"q": q})
    if not short:
        item.update(
            {"description": "This is an amazing item that has a long description"}
        )
    return item
```

In this case, if you go to:
```
http://127.0.0.1:8000/items/foo?short=1
```
or
```
http://127.0.0.1:8000/items/foo?short=True
```
or
```
http://127.0.0.1:8000/items/foo?short=true
```
or
```
http://127.0.0.1:8000/items/foo?short=on
```
or
```
http://127.0.0.1:8000/items/foo?short=yes
```
or any other case variation (uppercase, first letter in uppercase, etc), 
your function will see the parameter short with a bool value of `True`. Otherwise as `False`.


#### Multiple path and query parameters

You can declare multiple path parameters and query parameters at the same time, 
FastAPI knows which is which.

And you don't have to declare them in any specific order.

They will be detected by name:

```Python 3.10+
from fastapi import FastAPI

app = FastAPI()

@app.get("/users/{user_id}/items/{item_id}")
async def read_user_item(
    user_id: int, item_id: str, q: str | None = None, short: bool = False
):
    item = {"item_id": item_id, "owner_id": user_id}
    if q:
        item.update({"q": q})
    if not short:
        item.update(
            {"description": "This is an amazing item that has a long description"}
        )
    return item
```


#### Required query parameters

When you declare a default value for non-path parameters 
(for now, we have only seen query parameters), then it is not required.

If you don't want to add a specific value but just make it optional, set the default as `None`.

But when you want to make a query parameter required, you can just not declare any default value:

```Python 3.8+
from fastapi import FastAPI

app = FastAPI()

@app.get("/items/{item_id}")
async def read_user_item(item_id: str, needy: str):
    item = {"item_id": item_id, "needy": needy}
    return item
```

Here the query parameter `needy` is a required query parameter of type `str`.

If you open in your browser a URL like:
```
http://127.0.0.1:8000/items/foo-item
```
...without adding the required parameter `needy`, you will see an error like:
```
{
  "detail": [
    {
      "type": "missing",
      "loc": [
        "query",
        "needy"
      ],
      "msg": "Field required",
      "input": null,
      "url": "https://errors.pydantic.dev/2.1/v/missing"
    }
  ]
}
```
As `needy` is a required parameter, you would need to set it in the URL:
```
http://127.0.0.1:8000/items/foo-item?needy=sooooneedy
```
...this would work:
```
{
    "item_id": "foo-item",
    "needy": "sooooneedy"
}
```
And of course, you can define some parameters as required, 
some as having a default value, and some entirely optional:

```Python 3.10+
from fastapi import FastAPI

app = FastAPI()

@app.get("/items/{item_id}")
async def read_user_item(
    item_id: str, needy: str, skip: int = 0, limit: int | None = None
):
    item = {"item_id": item_id, "needy": needy, "skip": skip, "limit": limit}
    return item
```

In this case, there are 3 query parameters:
- `needy`, a required `str`.
- `skip`, an `int` with a default value of `0`.
- `limit`, an optional `int`.

**Tip**: 
    You could also use `Enum`s the same way as with [Path Parameters](../1.4__Path-Parameters/ReadMe.md#predefined-values).


### Additional validation

Let's take this application as example:

```Python 3.10+
from fastapi import FastAPI

app = FastAPI()

@app.get("/items/")
async def read_items(q: str | None = None):
    results = {"items": [{"item_id": "Foo"}, {"item_id": "Bar"}]}
    if q:
        results.update({"q": q})
    return results
```

The query parameter `q` is of type `str | None`, 
that means that it's of type `str` but could also be `None`, 
and indeed, the default value is `None`, so FastAPI will know it's not required.

We are going to enforce that even though `q` is optional, 
whenever it is provided, its length doesn't exceed 50 characters.


#### Import `Query` and `Annotated`

To achieve that, first import:
  - `Query` from `fastapi`
  - `Annotated` from `typing` (or from `typing_extensions` in Python below 3.9)

In Python 3.9 or above, `Annotated` is part of the standard library, so you can import it from `typing`.

```Python 3.10+
from typing import Annotated
from fastapi import FastAPI, Query
```

**Info**

FastAPI added support for Annotated (and started recommending it) in version 0.95.0.

If you have an older version, you would get errors when trying to use Annotated.

Make sure you Upgrade the FastAPI version to at least 0.95.1 before using Annotated.


#### Use `Annotated` in the type for the `q` parameter

`Annotated` can be used to add metadata to your parameters 
([Python Types Intro](https://fastapi.tiangolo.com/python-types/#type-hints-with-metadata-annotations))

We had this type annotation:

```Python 3.10+
q: str | None = None
```

What we will do is to wrap this with `Annotated`, so it becomes:

```Python 3.10+
q: Annotated[str | None] = None
```

Both of those versions mean the same thing, 
`q` is a parameter that can be a `str` or `None`, 
and by default, it is `None`.

Now let's jump to the fun stuff. 🎉


#### Add `Query` to Annotated in the `q` parameter

Now that we have this `Annotated` where we can put more information (in this case some additional validation), add `Query` inside of `Annotated`, and set the parameter `max_length` to `50`:

```Python 3.10+
from typing import Annotated
from fastapi import FastAPI, Query

app = FastAPI()

@app.get("/items/")
async def read_items(q: Annotated[str | None, Query(max_length=50)] = None):
    results = {"items": [{"item_id": "Foo"}, {"item_id": "Bar"}]}
    if q:
        results.update({"q": q})
    return results
```

Notice that the default value is still `None`, 
so the parameter is still optional.

But now, having `Query(max_length=50)` inside of `Annotated`, 
we are telling FastAPI that we want it to have additional validation for this value, 
we want it to have maximum 50 characters. 😎

**Tip**:

Here we are using `Query()` because this is a query parameter. 
We will see others like `Path()`, `Body()`, `Header()`, and `Cookie()`, 
that also accept the same arguments as `Query()`.

FastAPI will now:

  - Validate the data making sure that the max length is 50 characters
  - Show a clear error for the client when the data is not valid
  - Document the parameter in the OpenAPI schema path operation 
    (so it will show up in the automatic docs UI)


#### Add more validations
You can also add a parameter `min_length`:

```Python 3.10+
from typing import Annotated
from fastapi import FastAPI, Query

app = FastAPI()

@app.get("/items/")
async def read_items(
    q: Annotated[str | None, Query(min_length=3, max_length=50)] = None,
):
    results = {"items": [{"item_id": "Foo"}, {"item_id": "Bar"}]}
    if q:
        results.update({"q": q})
    return results
```


#### Add regular expressions

You can define a regular expression pattern that the parameter should match:

```Python 3.10+
from typing import Annotated
from fastapi import FastAPI, Query

app = FastAPI()

@app.get("/items/")
async def read_items(
    q: Annotated[
        str | None, Query(min_length=3, max_length=50, pattern="^fixedquery$")
    ] = None,
):
    results = {"items": [{"item_id": "Foo"}, {"item_id": "Bar"}]}
    if q:
        results.update({"q": q})
    return results
```

This specific regular expression pattern checks that the received parameter value:

  - `^`: starts with the following characters, doesn't have characters before.
  - `fixedquery`: has the exact value `fixedquery`.
  - `$`: ends there, doesn't have any more characters after `fixedquery`.

If you feel lost with all these "regular expression" ideas, don't worry. 
They are a hard topic for many people. 
You can still do a lot of stuff without needing regular expressions yet.

But whenever you need them and go and learn them, 
know that you can already use them directly in FastAPI.


#### Default values

You can, of course, use default values other than `None`.

Let's say that you want to declare the `q` query parameter 
to have a `min_length` of `3`, and to have a default value of `"fixedquery"`:

```Python 3.9+
from typing import Annotated
from fastapi import FastAPI, Query

app = FastAPI()

@app.get("/items/")
async def read_items(q: Annotated[str, Query(min_length=3)] = "fixedquery"):
    results = {"items": [{"item_id": "Foo"}, {"item_id": "Bar"}]}
    if q:
        results.update({"q": q})
    return results
```

Having a default value of any type, including `None`, makes the parameter optional (not required).


#### Required parameters¶

When we don't need to declare more validations or metadata, 
we can make the `q` query parameter required just by not declaring a default value, like:

```python
q: str
```
instead of:
```python
q: Union[str, None] = None
```

But we are now declaring it with `Query`, for example like:

```python
q: Annotated[Union[str, None], Query(min_length=3)] = None
```

So, when you need to declare a value as required while using `Query`, 
you can simply not declare a default value:

```Python 3.9+
from typing import Annotated
from fastapi import FastAPI, Query

app = FastAPI()

@app.get("/items/")
async def read_items(q: Annotated[str, Query(min_length=3)]):
    results = {"items": [{"item_id": "Foo"}, {"item_id": "Bar"}]}
    if q:
        results.update({"q": q})
    return results
```


#### Required with Ellipsis (`...`)

There's an alternative way to explicitly declare that a value is required. 
You can set the default to the literal value `...`:

```Python 3.9+
from typing import Annotated
from fastapi import FastAPI, Query

app = FastAPI()

@app.get("/items/")
async def read_items(q: Annotated[str, Query(min_length=3)] = ...):
    results = {"items": [{"item_id": "Foo"}, {"item_id": "Bar"}]}
    if q:
        results.update({"q": q})
    return results
```

**Info**: 

If you hadn't seen that `...` before: 
it is a special single value, 
it is part of Python and is called "Ellipsis".

It is used by Pydantic and FastAPI to explicitly declare that a value is required.

This will let FastAPI know that this parameter is required.


#### Required, can be None

You can declare that a parameter can accept `None`, but that it's still required. 
This would force clients to send a value, even if the value is `None`.

To do that, you can declare that `None` is a valid type but still use `...` as the default:

```Python 3.10+
from typing import Annotated
from fastapi import FastAPI, Query

app = FastAPI()

@app.get("/items/")
async def read_items(q: Annotated[str | None, Query(min_length=3)] = ...):
    results = {"items": [{"item_id": "Foo"}, {"item_id": "Bar"}]}
    if q:
        results.update({"q": q})
    return results
```

Tip

Remember that in most of the cases, 
when something is required, you can simply omit the default, 
so you normally don't have to use `...`.


### Query parameter list / multiple values

When you define a query parameter explicitly with `Query`, 
you can also declare it to receive a list of values, 
or said in another way, to receive multiple values.

For example, to declare a query parameter `q` 
that can appear multiple times in the URL, you can write:

```Python 3.10+
from typing import Annotated
from fastapi import FastAPI, Query

app = FastAPI()

@app.get("/items/")
async def read_items(q: Annotated[list[str] | None, Query()] = None):
    query_items = {"q": q}
    return query_items
```

Then, with a URL like http://localhost:8000/items/?q=foo&q=bar 
you would receive the multiple `q` query parameters' values (`foo` and `bar`) 
in a Python list inside your path operation function, in the function parameter `q`.

So, the response to that URL would be:

```
{
  "q": [
    "foo",
    "bar"
  ]
}
```

To declare a query parameter with a type of list, like in the example above, 
you need to explicitly use `Query`, 
otherwise it would be interpreted as a request body.

The interactive API docs will update accordingly, to allow multiple values:

<div style="text-align: center;">
    <img 
        src="../../imgs/image02 (1).png" 
        style="width:800px"
    >
</div> 


#### Query parameter list / multiple values with defaults

And you can also define a default list of values if none are provided:

```Python 3.9+
from typing import Annotated
from fastapi import FastAPI, Query

app = FastAPI()

@app.get("/items/")
async def read_items(q: Annotated[list[str], Query()] = ["foo", "bar"]):
    query_items = {"q": q}
    return query_items
```

If you go to http://localhost:8000/items/ 
the default of `q` will be: `["foo", "bar"]` and your response will be:

```
{
  "q": [
    "foo",
    "bar"
  ]
}
```


#### Using just list

You can also use `list` directly instead of `List[str]` (or `list[str]` in Python 3.9+):

```Python 3.9+
from typing import Annotated
from fastapi import FastAPI, Query

app = FastAPI()

@app.get("/items/")
async def read_items(q: Annotated[list, Query()] = []):
    query_items = {"q": q}
    return query_items
```

Keep in mind that in this case, FastAPI won't check the contents of the list.

For example, `List[int]` would check (and document) 
that the contents of the list are integers. 
But `list` alone wouldn't.


### Declare more metadata

You can add more information about the parameter.

That information will be included in the generated OpenAPI and used by the documentation user interfaces and external tools.

Note: 
    
    Keep in mind that different tools might have different levels of OpenAPI support.

    Some of them might not show all the extra information declared yet, although in most of the cases, the missing feature is already planned for development.

You can add a title:

```Python 3.10+
from typing import Annotated
from fastapi import FastAPI, Query

app = FastAPI()

@app.get("/items/")
async def read_items(
    q: Annotated[str | None, Query(title="Query string", min_length=3)] = None,
):
    results = {"items": [{"item_id": "Foo"}, {"item_id": "Bar"}]}
    if q:
        results.update({"q": q})
    return results
```

And a description:

```Python 3.10+
from typing import Annotated
from fastapi import FastAPI, Query

app = FastAPI()

@app.get("/items/")
async def read_items(
    q: Annotated[
        str | None,
        Query(
            title="Query string",
            description="Query string for the items to search in the database that have a good match",
            min_length=3,
        ),
    ] = None,
):
    results = {"items": [{"item_id": "Foo"}, {"item_id": "Bar"}]}
    if q:
        results.update({"q": q})
    return results
```


### Alias parameters

Imagine that you want the parameter to be `item-query`.

Like in:
```
http://127.0.0.1:8000/items/?item-query=foobaritems
```

But `item-query` is not a valid Python variable name.

The closest would be `item_query`.

But you still need it to be exactly `item-query`...

Then you can declare an **alias**, and that alias is what will be used to find the parameter value:

```Python 3.10+
from typing import Annotated
from fastapi import FastAPI, Query

app = FastAPI()

@app.get("/items/")
async def read_items(q: Annotated[str | None, Query(alias="item-query")] = None):
    results = {"items": [{"item_id": "Foo"}, {"item_id": "Bar"}]}
    if q:
        results.update({"q": q})
    return results
```


### Deprecating parameters

Now let's say you don't like this parameter anymore.

You have to leave it there a while because there are clients using it, 
but you want the docs to clearly show it as deprecated.

Then pass the parameter `deprecated=True` to `Query`:

```Python 3.10+
from typing import Annotated
from fastapi import FastAPI, Query

app = FastAPI()

@app.get("/items/")
async def read_items(
    q: Annotated[
        str | None,
        Query(
            alias="item-query",
            title="Query string",
            description="Query string for the items to search in the database that have a good match",
            min_length=3,
            max_length=50,
            pattern="^fixedquery$",
            deprecated=True,
        ),
    ] = None,
):
    results = {"items": [{"item_id": "Foo"}, {"item_id": "Bar"}]}
    if q:
        results.update({"q": q})
    return results
```

The docs will show it like this:

<div style="text-align: center;">
    <img 
        src="../../imgs/image01 (1).png" 
        style="width:800px"
    >
</div> 


### Exclude parameters from OpenAPI

To exclude a query parameter from the generated OpenAPI schema 
(and thus, from the automatic documentation systems), 
set the parameter `include_in_schema` of `Query` to `False`:

```Python 3.10+
from typing import Annotated
from fastapi import FastAPI, Query

app = FastAPI()

@app.get("/items/")
async def read_items(
    hidden_query: Annotated[str | None, Query(include_in_schema=False)] = None,
):
    if hidden_query:
        return {"hidden_query": hidden_query}
    else:
        return {"hidden_query": "Not found"}
```


### Recap

You can declare additional validations and metadata for your parameters.

Generic validations and metadata:
  - `alias`
  - `title`
  - `description`
  - `deprecated`

Validations specific for strings:
  - `min_length`
  - `max_length`
  - `pattern`

In these examples you saw how to declare validations for `str` values.





### Reference Materials

  - [FastAPI > Learn > Tutorial - User Guide > Query Parameters](https://fastapi.tiangolo.com/tutorial/query-params)

  - [FastAPI > Learn > Tutorial - User Guide > Query Parameters and String Validations](https://fastapi.tiangolo.com/tutorial/query-params-str-validations)

