## 2.1. Request Body

When you need to send data from a client (let's say, a browser) to your API, 
you send it as a request body.
A **request body** is data sent by the client to your API. 

A **response body** is the data your API sends to the client.

Your API almost always has to send a response body. 
But clients don't necessarily need to send request bodies all the time, 
sometimes they only request a path, maybe with some query parameters, 
but don't send a body.

To declare a request body, 
you use Pydantic models with all their power and benefits.

**Info**: 

To send data, you should use one of: 
`POST` (the more common), `PUT`, `DELETE` or `PATCH`.
    
Sending a body with a `GET` request has an 
undefined behavior in the specifications, 
nevertheless, it is supported by FastAPI, 
only for very complex/extreme use cases.

As it is discouraged, 
the interactive docs with Swagger UI won't show 
the documentation for the body when using GET, 
and proxies in the middle might not support it.

------------------------------------------------------------------------------------------

### Basics

#### Create your data model

First, you need to import `BaseModel` from `pydantic`:

Then you declare your data model as a class that inherits from `BaseModel`.

Use standard Python types for all the attributes:

```Python 3.10+
from pydantic import BaseModel

class Item(BaseModel):
    name: str
    description: str | None = None
    price: float
    tax: float | None = None
```

The same as when declaring query parameters, 
when a model attribute has a default value, it is not required. 
Otherwise, it is required. Use `None` to make it just optional.

For example, this model above declares a JSON "object" (or Python `dict`) like:

```
{
    "name": "Foo",
    "description": "An optional description",
    "price": 45.2,
    "tax": 3.5
}
```

as `description` and `tax` are optional (with a default value of `None`), 
this JSON "object" would also be valid:

```
{
    "name": "Foo",
    "price": 45.2
}
```


#### Declare it as a parameter

To add it to your path operation, declare it the same way you declared path and query parameters:

```Python 3.10+
from fastapi import FastAPI
from pydantic import BaseModel

class Item(BaseModel):
    name: str
    description: str | None = None
    price: float
    tax: float | None = None

app = FastAPI()

@app.post("/items/")
async def create_item(item: Item):
    return item
```

and declare its type as the model you created, Item.


#### Results

With just that Python type declaration, FastAPI will:

- Read the body of the request as JSON.
- Convert the corresponding types (if needed).
- Validate the data.
  * If the data is invalid, it will return a nice and clear error, 
  indicating exactly where and what was the incorrect data.
- Give you the received data in the parameter item.
  * As you declared it in the function to be of type Item, 
  you will also have all the editor support (completion, etc) 
  for all of the attributes and their types.
- Generate [JSON Schema](https://json-schema.org/) definitions for your model, 
  you can also use them anywhere else you like 
  if it makes sense for your project.
- Those schemas will be part of the generated OpenAPI schema, 
  and used by the automatic documentation UIs.


#### Automatic docs

The JSON Schemas of your models will be part of your OpenAPI generated schema, and will be shown in the interactive API docs:

<div style="text-align: center;">
    <img 
        src="../../imgs/image01.png" 
        style="width:800px"
    >
</div> 


And will also be used in the API docs inside each path operation that needs them:

<div style="text-align: center;">
    <img 
        src="../../imgs/image02.png" 
        style="width:800px"
    >
</div> 


#### Editor support

In your editor, inside your function you will get type hints and completion everywhere 
(this wouldn't happen if you received a `dict` instead of a Pydantic model):

<div style="text-align: center;">
    <img 
        src="../../imgs/image03 (1).png" 
        style="width:1024px"
    >
</div> 

You also get error checks for incorrect type operations:

<div style="text-align: center;">
    <img 
        src="../../imgs/image04.png" 
        style="width:800px"
    >
</div> 

This is not by chance, the whole framework was built around that design.

And it was thoroughly tested at the design phase, before any implementation, to ensure it would work with all the editors.

There were even some changes to Pydantic itself to support this.

The previous screenshots were taken with Visual Studio Code.

But you would get the same editor support with PyCharm and most of the other Python editors.


#### Use the model

Inside of the function, you can access all the attributes of the model object directly:

```Python 3.10+
from fastapi import FastAPI
from pydantic import BaseModel

class Item(BaseModel):
    name: str
    description: str | None = None
    price: float
    tax: float | None = None

app = FastAPI()

@app.post("/items/")
async def create_item(item: Item):
    item_dict = item.dict()
    if item.tax:
        price_with_tax = item.price + item.tax
        item_dict.update({"price_with_tax": price_with_tax})
    return item_dict
```


#### Request body + path parameters

You can declare path parameters and request body at the same time.

FastAPI will recognize that the function parameters that 
match path parameters should be taken from the path, 
and that function parameters that are declared to be Pydantic models should be taken from the request body.


```Python 3.10+
from fastapi import FastAPI
from pydantic import BaseModel

class Item(BaseModel):
    name: str
    description: str | None = None
    price: float
    tax: float | None = None

app = FastAPI()

@app.put("/items/{item_id}")
async def update_item(item_id: int, item: Item):
    return {"item_id": item_id, **item.dict()}
```


#### Request body + path + query parameters

You can also declare body, path and query parameters, all at the same time.

FastAPI will recognize each of them and take the data from the correct place.

```Python 3.10+
from fastapi import FastAPI
from pydantic import BaseModel

class Item(BaseModel):
    name: str
    description: str | None = None
    price: float
    tax: float | None = None

app = FastAPI()

@app.put("/items/{item_id}")
async def update_item(
    item_id: int, item: Item, q: str | None = None
):
    result = {"item_id": item_id, **item.dict()}
    if q:
        result.update({"q": q})
    return result
```

The function parameters will be recognized as follows:

  - If the parameter is also declared in the path, 
    it will be used as a path parameter.
  - If the parameter is of a singular type 
    (like `int`, `float`, `str`, `bool`, etc) 
    it will be interpreted as a query parameter.
  - If the parameter is declared to be of the type of a Pydantic model, 
    it will be interpreted as a request body.


#### Without Pydantic

If you don't want to use Pydantic models, 
you can also use Body parameters. 

See the docs for Body - [Multiple Parameters: Singular values in body](https://fastapi.tiangolo.com/tutorial/body-multiple-params/#singular-values-in-body).

------------------------------------------------------------------------------------------

### Body - Multiple Parameters


#### Mix `Path`, `Query` and body parameters

First, of course, you can mix `Path`, `Query` and request body parameter declarations freely and FastAPI will know what to do.

And you can also declare body parameters as optional, by setting the default to `None`:

```Python 3.10+
from typing import Annotated
from fastapi import FastAPI, Path
from pydantic import BaseModel

app = FastAPI()

class Item(BaseModel):
    name: str
    description: str | None = None
    price: float
    tax: float | None = None

@app.put("/items/{item_id}")
async def update_item(
    item_id: Annotated[int, Path(title="The ID of the item to get", ge=0, le=1000)],
    q: str | None = None,
    item: Item | None = None,
):
    results = {"item_id": item_id}
    if q:
        results.update({"q": q})
    if item:
        results.update({"item": item})
    return results
```

Notice that, in this case, 
the `item` that would be taken from the body is optional. 
As it has a `None` default value.


#### Multiple body parameters

In the previous example, the path operations would expect a JSON body with the attributes of an `Item`, like:

```python
{
    "name": "Foo",
    "description": "The pretender",
    "price": 42.0,
    "tax": 3.2
}
```

But you can also declare multiple body parameters, e.g. `item` and `user`:

```Python 3.10+
from fastapi import FastAPI
from pydantic import BaseModel

app = FastAPI()

class Item(BaseModel):
    name: str
    description: str | None = None
    price: float
    tax: float | None = None

class User(BaseModel):
    username: str
    full_name: str | None = None

@app.put("/items/{item_id}")
async def update_item(item_id: int, item: Item, user: User):
    results = {"item_id": item_id, "item": item, "user": user}
    return results
```

In this case, FastAPI will notice that 
there is more than one body parameter in the function 
(there are two parameters that are Pydantic models).

So, it will then use the parameter names as keys 
(field names) in the body, and expect a body like:

```python
{
    "item": {
        "name": "Foo",
        "description": "The pretender",
        "price": 42.0,
        "tax": 3.2
    },
    "user": {
        "username": "dave",
        "full_name": "Dave Grohl"
    }
}
```

Notice that even though the `item` was declared the same way as before, 
it is now expected to be inside of the body with a key `item`.

FastAPI will do the automatic conversion from the request, 
so that the parameter `item` receives its specific content and the same for user.

It will perform the validation of the compound data, 
and will document it like that for the OpenAPI schema and automatic docs.


#### Singular values in body

The same way there is a `Query` and `Path` 
to define extra data for query and path parameters, 
FastAPI provides an equivalent `Body`.

For example, extending the previous model, 
you could decide that you want to have another key importance in the same body, 
besides the `item` and `user`.

If you declare it as is, because it is a singular value, 
FastAPI will assume that it is a query parameter.

But you can instruct FastAPI to treat it as another body key using `Body`:

```Python 3.10+
from typing import Annotated
from fastapi import Body, FastAPI
from pydantic import BaseModel

app = FastAPI()

class Item(BaseModel):
    name: str
    description: str | None = None
    price: float
    tax: float | None = None

class User(BaseModel):
    username: str
    full_name: str | None = None

@app.put("/items/{item_id}")
async def update_item(
    item_id: int, item: Item, user: User, importance: Annotated[int, Body()]
):
    results = {"item_id": item_id, "item": item, "user": user, "importance": importance}
    return results
```

In this case, FastAPI will expect a body like:

```python
{
    "item": {
        "name": "Foo",
        "description": "The pretender",
        "price": 42.0,
        "tax": 3.2
    },
    "user": {
        "username": "dave",
        "full_name": "Dave Grohl"
    },
    "importance": 5
}
```

Again, it will convert the data types, validate, document, etc.


#### Multiple body params and query

Of course, you can also declare 
additional query parameters whenever you need, 
additional to any body parameters.

As, by default, singular values are interpreted as query parameters, 
you don't have to explicitly add a `Query`, you can just do:

```python
q: str | None = None
```

For example:

```Python 3.10+
from typing import Annotated
from fastapi import Body, FastAPI
from pydantic import BaseModel

app = FastAPI()

class Item(BaseModel):
    name: str
    description: str | None = None
    price: float
    tax: float | None = None

class User(BaseModel):
    username: str
    full_name: str | None = None

@app.put("/items/{item_id}")
async def update_item(
    *,
    item_id: int,
    item: Item,
    user: User,
    importance: Annotated[int, Body(gt=0)],
    q: str | None = None,
):
    results = {"item_id": item_id, "item": item, "user": user, "importance": importance}
    if q:
        results.update({"q": q})
    return results
```

Body also has all the same extra validation and metadata parameters as `Query`, `Path` and others you will see later.


#### Embed a single body parameter

Let's say you only have a single `item` body parameter from a Pydantic model `Item`.

By default, FastAPI will then expect its body directly.

But if you want it to expect a JSON with a key `item` and inside of it the model contents, 
as it does when you declare extra body parameters, 
you can use the special `Body` parameter `embed`:

```python
item: Item = Body(embed=True)
```

as in:

```Python 3.10+
from typing import Annotated
from fastapi import Body, FastAPI
from pydantic import BaseModel

app = FastAPI()

class Item(BaseModel):
    name: str
    description: str | None = None
    price: float
    tax: float | None = None

@app.put("/items/{item_id}")
async def update_item(item_id: int, item: Annotated[Item, Body(embed=True)]):
    results = {"item_id": item_id, "item": item}
    return results
```

In this case FastAPI will expect a body like:

```python
{
    "item": {
        "name": "Foo",
        "description": "The pretender",
        "price": 42.0,
        "tax": 3.2
    }
}
```

instead of:

```python
{
    "name": "Foo",
    "description": "The pretender",
    "price": 42.0,
    "tax": 3.2
}
```


#### Recap

You can add multiple body parameters to your *path operation function*, 
even though a request can only have a single body.

But FastAPI will handle it, 
give you the correct data in your function, 
and validate and document the correct schema in the path operation.

You can also declare singular values to be received as part of the body.

And you can instruct FastAPI to embed the body in a key 
even when there is only a single parameter declared.

------------------------------------------------------------------------------------------

### Body - Fields

The same way you can declare additional validation and metadata 
in path operation function parameters with `Query`, `Path` and `Body`, 
you can declare validation and metadata inside of Pydantic models using Pydantic's `Field`.


#### Import `Field`

First, you have to import it:

```Python 3.10+
from pydantic import BaseModel, Field
```

Notice that `Field` is imported directly from `pydantic`, not from `fastapi` as are all the rest (`Query`, `Path`, `Body`, etc).


#### Declare model attributes

You can then use `Field` with model attributes:

```Python 3.10+
from pydantic import BaseModel, Field

class Item(BaseModel):
    name: str
    description: str | None = Field(
        default=None, title="The description of the item", max_length=300
    )
    price: float = Field(gt=0, description="The price must be greater than zero")
    tax: float | None = None
```

`Field` works the same way as `Query`, `Path` and `Body`, it has all the same parameters, etc.

**Technical Details**:

Actually, `Query`, `Path` and others create objects of subclasses of a common `Param` class, 
which is itself a subclass of Pydantic's `FieldInfo` class.

And Pydantic's `Field` returns an instance of `FieldInfo` as well.

`Body` also returns objects of a subclass of `FieldInfo` directly. 
And there are others you will see later that are subclasses of the `Body` class.

Remember that when you import `Query`, `Path`, and others from `fastapi`, 
those are actually functions that return special classes.

**Tip**:

Notice how each model's attribute with a type, default value and `Field` 
has the same structure as a path operation function's parameter, 
with `Field` instead of `Path`, `Query` and `Body`.


#### Add extra information

You can declare extra information in `Field`, `Query`, `Body`, etc. 
And it will be included in the generated JSON Schema.

**Warning**

Extra keys passed to `Field` will also be present in the resulting OpenAPI schema for your application. 
As these keys may not necessarily be part of the OpenAPI specification, 
some OpenAPI tools, for example the OpenAPI validator, may not work with your generated schema.


#### Recap

You can use Pydantic's `Field` to declare extra validations and metadata for model attributes.

You can also use the extra keyword arguments to pass additional JSON Schema metadata.

------------------------------------------------------------------------------------------

### Body - Nested Models

With FastAPI, you can define, validate, document, and use arbitrarily deeply nested models (thanks to Pydantic).


#### List fields

You can define an attribute to be a subtype. For example, a Python `list`:

```Python 3.10+
from fastapi import FastAPI
from pydantic import BaseModel

app = FastAPI()

class Item(BaseModel):
    name: str
    description: str | None = None
    price: float
    tax: float | None = None
    tags: list = []

@app.put("/items/{item_id}")
async def update_item(item_id: int, item: Item):
    results = {"item_id": item_id, "item": item}
    return results
```

This will make `tags` be a list, although it doesn't declare the type of the elements of the list.


#### List fields with type parameter

But Python has a specific way to declare lists with internal types, or "type parameters":


##### Declare a list with a type parameter
To declare types that have type parameters (internal types), like `list`, `dict`, `tuple`:

  - If you are in a Python version lower than 3.9, import their equivalent version from the `typing` module
  - Pass the internal type(s) as "type parameters" using square brackets: `[` and `]`

In Python 3.9 it would be:

```Python 3.9
my_list: list[str]
```

In versions of Python before 3.9, it would be:

```Python 3.8
from typing import List

my_list: List[str]
```

That's all standard Python syntax for type declarations.

Use that same standard syntax for model attributes with internal types.

So, in our example, we can make `tags` be specifically a "list of strings":

```Python 3.10+
from fastapi import FastAPI
from pydantic import BaseModel

app = FastAPI()

class Item(BaseModel):
    name: str
    description: str | None = None
    price: float
    tax: float | None = None
    tags: list[str] = []

@app.put("/items/{item_id}")
async def update_item(item_id: int, item: Item):
    results = {"item_id": item_id, "item": item}
    return results
```


#### Set types
But then we think about it, and realize that tags shouldn't repeat, they would probably be unique strings.

And Python has a special data type for sets of unique items, the `set`.

Then we can declare `tags` as a set of strings:

```Python 3.10+
from fastapi import FastAPI
from pydantic import BaseModel

app = FastAPI()

class Item(BaseModel):
    name: str
    description: str | None = None
    price: float
    tax: float | None = None
    tags: set[str] = set()

@app.put("/items/{item_id}")
async def update_item(item_id: int, item: Item):
    results = {"item_id": item_id, "item": item}
    return results
```

With this, 
even if you receive a request with duplicate data, 
it will be converted to a set of unique items.

And whenever you output that data, 
even if the source had duplicates, 
it will be output as a set of unique items.

And it will be annotated / documented accordingly too.


#### Nested Models

Each attribute of a Pydantic model has a type.

But that type can itself be another Pydantic model.

So, you can declare deeply nested JSON "objects" with specific attribute names, types and validations.

All that, arbitrarily nested.


##### Define a submodel

For example, we can define an Image model:

```Python 3.10+
from pydantic import BaseModel

class Image(BaseModel):
    url: str
    name: str
```


##### Use the submodel as a type

And then we can use it as the type of an attribute:

```Python 3.10+
from fastapi import FastAPI
from pydantic import BaseModel

app = FastAPI()

class Image(BaseModel):
    url: str
    name: str

class Item(BaseModel):
    name: str
    description: str | None = None
    price: float
    tax: float | None = None
    tags: set[str] = set()
    image: Image | None = None

@app.put("/items/{item_id}")
async def update_item(item_id: int, item: Item):
    results = {"item_id": item_id, "item": item}
    return results
```

This would mean that FastAPI would expect a body similar to:

```python
{
    "name": "Foo",
    "description": "The pretender",
    "price": 42.0,
    "tax": 3.2,
    "tags": ["rock", "metal", "bar"],
    "image": {
        "url": "http://example.com/baz.jpg",
        "name": "The Foo live"
    }
}
```

Again, doing just that declaration, with FastAPI you get:
  - Editor support (completion, etc.), even for nested models
  - Data conversion
  - Data validation
  - Automatic documentation


#### Special types and validation

Apart from normal singular types like `str`, `int`, `float`, etc. 
you can use more complex singular types that inherit from `str`.

To see all the options you have, checkout [Pydantic's Type Overview](https://docs.pydantic.dev/latest/concepts/types/). 
You will see some examples in the next chapter.

For example, as in the `Image` model we have a `url` field, 
we can declare it to be an instance of Pydantic's `HttpUrl` instead of a `str`:

```Python 3.10+
from fastapi import FastAPI
from pydantic import BaseModel, HttpUrl

app = FastAPI()

class Image(BaseModel):
    url: HttpUrl
    name: str

class Item(BaseModel):
    name: str
    description: str | None = None
    price: float
    tax: float | None = None
    tags: set[str] = set()
    image: Image | None = None

@app.put("/items/{item_id}")
async def update_item(item_id: int, item: Item):
    results = {"item_id": item_id, "item": item}
    return results
```

The string will be checked to be a valid URL, and documented in JSON Schema / OpenAPI as such.


#### Attributes with lists of submodels

You can also use Pydantic models as subtypes of `list`, `set`, etc.:

```Python 3.10+
from fastapi import FastAPI
from pydantic import BaseModel, HttpUrl

app = FastAPI()

class Image(BaseModel):
    url: HttpUrl
    name: str

class Item(BaseModel):
    name: str
    description: str | None = None
    price: float
    tax: float | None = None
    tags: set[str] = set()
    images: list[Image] | None = None

@app.put("/items/{item_id}")
async def update_item(item_id: int, item: Item):
    results = {"item_id": item_id, "item": item}
    return results
```

This will expect (convert, validate, document, etc.) a JSON body like:

```py
{
    "name": "Foo",
    "description": "The pretender",
    "price": 42.0,
    "tax": 3.2,
    "tags": [
        "rock",
        "metal",
        "bar"
    ],
    "images": [
        {
            "url": "http://example.com/baz.jpg",
            "name": "The Foo live"
        },
        {
            "url": "http://example.com/dave.jpg",
            "name": "The Baz"
        }
    ]
}
```

Notice how the `images` key now has a list of `image` objects.


#### Deeply nested models

You can define arbitrarily deeply nested models:

```Python 3.10+
from fastapi import FastAPI
from pydantic import BaseModel, HttpUrl

app = FastAPI()

class Image(BaseModel):
    url: HttpUrl
    name: str

class Item(BaseModel):
    name: str
    description: str | None = None
    price: float
    tax: float | None = None
    tags: set[str] = set()
    images: list[Image] | None = None

class Offer(BaseModel):
    name: str
    description: str | None = None
    price: float
    items: list[Item]

@app.post("/offers/")
async def create_offer(offer: Offer):
    return offer
```

Notice how Offer has a list of Items, which in turn have an optional list of Images


#### Bodies of pure lists

If the top level value of the JSON body you expect is a JSON array (a Python `list`), 
you can declare the type in the parameter of the function, the same as in Pydantic models:

```py
images: List[Image]
```

or in Python 3.9 and above:

```py
images: list[Image]
```

as in:

```Python 3.9+
from fastapi import FastAPI
from pydantic import BaseModel, HttpUrl

app = FastAPI()

class Image(BaseModel):
    url: HttpUrl
    name: str

@app.post("/images/multiple/")
async def create_multiple_images(images: list[Image]):
    return images
```


#### Editor support everywhere¶

And you get editor support everywhere.

Even for items inside of lists:

<div style="text-align: center;">
    <img 
        src="../../imgs/image01 (2).png" 
        style="width:800px"
    >
</div> 

You couldn't get this kind of editor support 
if you were working directly with `dict` instead of Pydantic models.

But you don't have to worry about them either, incoming `dict`s are converted automatically 
and your output is converted automatically to JSON too.

#### Bodies of arbitrary `dict`s

You can also declare a body as a `dict` with keys of some type and values of some other type.

This way, you don't have to know beforehand what the valid field/attribute names are 
(as would be the case with Pydantic models).

This would be useful if you want to receive keys that you don't already know.

---

Another useful case is when you want to have keys of another type (e.g., `int`).

That's what we are going to see here.

In this case, you would accept any `dict` 
as long as it has `int` keys with `float` values:

```Python 3.9+
from fastapi import FastAPI

app = FastAPI()

@app.post("/index-weights/")
async def create_index_weights(weights: dict[int, float]):
    return weights
```

Keep in mind that JSON only supports `str` as keys.

But Pydantic has *automatic data conversion*. 
This means that, even though your API clients can only send strings as keys, 
as long as those strings contain pure integers, Pydantic will convert them and validate them.

And the `dict` you receive as `weights` will actually have `int` keys and `float` values.


#### Recap

With FastAPI you have the maximum flexibility provided by Pydantic models, while keeping your code simple, short and elegant.

But with all the benefits:
  - Editor support (completion everywhere!)
  - Data conversion (a.k.a. parsing / serialization)
  - Data validation
  - Schema documentation
  - Automatic docs

------------------------------------------------------------------------------------------

### Declare Request Example Data

You can declare examples of the data your app can receive.

Here are several ways to do it.


#### Extra JSON Schema data in Pydantic models
You can declare examples for a Pydantic model that will be added to the generated JSON Schema.


```Python 3.10+ - Pydantic v2
from fastapi import FastAPI
from pydantic import BaseModel

app = FastAPI()


class Item(BaseModel):
    name: str
    description: str | None = None
    price: float
    tax: float | None = None

    model_config = {
        "json_schema_extra": {
            "examples": [
                {
                    "name": "Foo",
                    "description": "A very nice Item",
                    "price": 35.4,
                    "tax": 3.2,
                }
            ]
        }
    }


@app.put("/items/{item_id}")
async def update_item(item_id: int, item: Item):
    results = {"item_id": item_id, "item": item}
    return results
```

That extra info will be added as-is to the output JSON Schema for that model, 
and it will be used in the API docs.

In Pydantic version 2, you would use the attribute `model_config`, 
that takes a `dict` as described in [Pydantic's docs: Configuration](https://docs.pydantic.dev/latest/api/config/).

You can set `"json_schema_extra"` with a dict containing any additional data 
you would like to show up in the generated JSON Schema, including examples.


#### `Field` additional arguments

When using `Field()` with Pydantic models, you can also declare additional examples:

```Python 3.10+
from fastapi import FastAPI
from pydantic import BaseModel, Field

app = FastAPI()

class Item(BaseModel):
    name: str = Field(examples=["Foo"])
    description: str | None = Field(default=None, examples=["A very nice Item"])
    price: float = Field(examples=[35.4])
    tax: float | None = Field(default=None, examples=[3.2])

@app.put("/items/{item_id}")
async def update_item(item_id: int, item: Item):
    results = {"item_id": item_id, "item": item}
    return results
```

#### examples in JSON Schema - OpenAPI

When using any of:

  - `Path()`
  - `Query()`
  - `Header()`
  - `Cookie()`
  - `Body()`
  - `Form()`
  - `File()`

you can also declare a group of examples with additional information 
that will be added to their JSON Schemas inside of OpenAPI.


### Reference Materials

  - [FastAPI > Learn > Tutorial - User Guide > Request Body](https://fastapi.tiangolo.com/tutorial/body/)

  - [FastAPI > Learn > Tutorial - User Guide > Body - Multiple Parameters](https://fastapi.tiangolo.com/tutorial/body-multiple-params)

  - [FastAPI > Learn > Tutorial - User Guide > Body - Fields](https://fastapi.tiangolo.com/tutorial/body-fields)

  - [FastAPI > Learn > Tutorial - User Guide > Body - Nested Models](https://fastapi.tiangolo.com/tutorial/body-nested-models)

  - [FastAPI > Learn > Tutorial - User Guide > Declare Request Example Data](https://fastapi.tiangolo.com/tutorial/schema-extra-example)
  