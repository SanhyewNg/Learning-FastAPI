## 1.4. Parameter Validations


### More Validations for Query Parameters

#### Approach

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


##### Import `Query` and `Annotated`

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


##### Use `Annotated` in the type for the `q` parameter

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


##### Add `Query` to Annotated in the `q` parameter

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


##### Required with Ellipsis (`...`)

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


##### Required, can be None

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


#### Query parameter list / multiple values

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


##### Query parameter list / multiple values with defaults

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


##### Using just list

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


#### Declare more metadata

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


#### Alias parameters

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


#### Deprecating parameters

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


#### Exclude parameters from OpenAPI

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


#### Recap

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

----------------------------------------------------------------------------

### Additional Validations for Path Parameters

In the same way that you can declare [more validations and metadata for query parameters with `Query`](../1.5__Query-Parameters/ReadMe.md#add-more-validations), 
you can declare the same type of validations and metadata for path parameters with `Path`.

#### Approach

##### Import `Path`
First, import `Path` from `fastapi`, and import `Annotated`:

```Python 3.10+
from typing import Annotated
```

FastAPI added support for `Annotated` (and started recommending it) in version 0.95.0.

If you have an older version, you would get errors when trying to use `Annotated`.

Make sure you [Upgrade the FastAPI version](https://fastapi.tiangolo.com/deployment/versions/#upgrading-the-fastapi-versions) 
to at least 0.95.1 before using `Annotated`.


##### Declare metadata

You can declare all the same parameters as for `Query`.

For example, to declare a `title` metadata value for the path parameter `item_id` you can type:

```Python 3.10+
from typing import Annotated
from fastapi import FastAPI, Path, Query

app = FastAPI()

@app.get("/items/{item_id}")
async def read_items(
    item_id: Annotated[int, Path(title="The ID of the item to get")],
    q: Annotated[str | None, Query(alias="item-query")] = None,
):
    results = {"item_id": item_id}
    if q:
        results.update({"q": q})
    return results
```

A path parameter is always required as it has to be part of the path. 
Even if you declared it with `None` or set a default value, 
it would not affect anything, it would still be always required.


#### Order the parameters as you need

This is probably not as important or necessary if you use `Annotated`.

Let's say that you want to declare the query parameter `q` as a required `str`.

And you don't need to declare anything else for that parameter, so you don't really need to use `Query`.

But you still need to use `Path` for the `item_id` path parameter. 
And you don't want to use `Annotated` for some reason.

Python will complain if you put a value with a "default" before a value that doesn't have a "default".

But you can re-order them, and have the value without a default (the query parameter `q`) first.

It doesn't matter for FastAPI. 
It will detect the parameters by their names, types and default declarations (`Query`, `Path`, etc), 
it doesn't care about the order.

So, you can declare your function as:

```Python 3.8+ - non-Annotated
from fastapi import FastAPI, Path

app = FastAPI()

@app.get("/items/{item_id}")
async def read_items(q: str, item_id: int = Path(title="The ID of the item to get")):
    results = {"item_id": item_id}
    if q:
        results.update({"q": q})
    return results
```

Prefer to use the `Annotated` version if possible.

But keep in mind that if you use `Annotated`, you won't have this problem, 
it won't matter as you're not using the function parameter default values for `Query()` or `Path()`.

```Python 3.9+
from typing import Annotated
from fastapi import FastAPI, Path

app = FastAPI()

@app.get("/items/{item_id}")
async def read_items(
    q: str, item_id: Annotated[int, Path(title="The ID of the item to get")]
):
    results = {"item_id": item_id}
    if q:
        results.update({"q": q})
    return results
```


##### Order the parameters as you need, tricks

This is probably not as important or necessary if you use `Annotated`.

Here's a small trick that can be handy, but you won't need it often.

If you want to:
  - declare the `q` query parameter without a `Query` nor any default value
  - declare the path parameter `item_id` using Path
  - have them in a different order
  - not use `Annotated`

Python has a little special syntax for that.

Pass `*`, as the first parameter of the function.

Python won't do anything with that `*`, 
but it will know that all the following parameters should be called as 
keyword arguments (key-value pairs), also known as `kwargs`. 
Even if they don't have a default value.

```Python 3.8+ - non-Annotated
from fastapi import FastAPI, Path

app = FastAPI()

@app.get("/items/{item_id}")
async def read_items(*, item_id: int = Path(title="The ID of the item to get"), q: str):
    results = {"item_id": item_id}
    if q:
        results.update({"q": q})
    return results
```

##### Better with `Annotated`
Keep in mind that if you use `Annotated`, 
as you are not using function parameter default values, 
you won't have this problem, and you probably won't need to use `*`.

```Python 3.9+
from typing import Annotated
from fastapi import FastAPI, Path

app = FastAPI()

@app.get("/items/{item_id}")
async def read_items(
    item_id: Annotated[int, Path(title="The ID of the item to get")], q: str
):
    results = {"item_id": item_id}
    if q:
        results.update({"q": q})
    return results
```


#### Number validations

##### greater than or equal

With `Query` and `Path` (and others you'll see later) 
you can declare number constraints.

Here, with `ge=1`, `item_id` will need to be an integer number "greater than or equal" to `1`.

```Python 3.9+
from typing import Annotated
from fastapi import FastAPI, Path

app = FastAPI()

@app.get("/items/{item_id}")
async def read_items(
    item_id: Annotated[int, Path(title="The ID of the item to get", ge=1)], q: str
):
    results = {"item_id": item_id}
    if q:
        results.update({"q": q})
    return results
```

##### greater than and less than or equal
The same applies for:
  - `gt`: greater than
  - `le`: less than or equal

```Python 3.9+
from typing import Annotated
from fastapi import FastAPI, Path

app = FastAPI()

@app.get("/items/{item_id}")
async def read_items(
    item_id: Annotated[int, Path(title="The ID of the item to get", gt=0, le=1000)],
    q: str,
):
    results = {"item_id": item_id}
    if q:
        results.update({"q": q})
    return results
```


##### floats, greater than and less than

Number validations also work for `float` values.

Here's where it becomes important to be able to declare `gt` and not just `ge`. 
As with it you can require, for example, 
that a value must be greater than `0`, even if it is less than `1`.

So, `0.5` would be a valid value. But `0.0` or `0` would not.

And the same for `lt`.

```Python 3.9+
from typing import Annotated
from fastapi import FastAPI, Path, Query

app = FastAPI()

@app.get("/items/{item_id}")
async def read_items(
    *,
    item_id: Annotated[int, Path(title="The ID of the item to get", ge=0, le=1000)],
    q: str,
    size: Annotated[float, Query(gt=0, lt=10.5)],
):
    results = {"item_id": item_id}
    if q:
        results.update({"q": q})
    if size:
        results.update({"size": size})
    return results
```


#### Recap

With `Path` (and others you haven't seen yet), 
you can declare metadata and string validations.

And you can also declare numeric validations:
  - `gt`: greater than
  - `ge`: greater than or equal
  - `lt`: less than
  - `le`: less than or equal

`Query`, `Path`, and other classes you will see later are subclasses of a common `Param` class.

All of them share the same parameters for additional validation and metadata you have seen.


### Technical Details

When you import `Query`, `Path` and others from `fastapi`, they are actually functions.

That when called, return instances of classes of the same name.

So, you import `Query`, which is a function. 
And when you call it, it returns an instance of a class also named `Query`.

These functions are there (instead of just using the classes directly) 
so that your editor doesn't mark errors about their types.

That way you can use your normal editor and coding tools 
without having to add custom configurations to disregard those errors.


### Reference Materials

  - [FastAPI > Learn > Tutorial - User Guide > Query Parameters and String Validations](https://fastapi.tiangolo.com/tutorial/query-params-str-validations)

  - [FastAPI > Learn > Tutorial - User Guide > Path Parameters and Numeric Validations](https://fastapi.tiangolo.com/tutorial/path-params-numeric-validations)