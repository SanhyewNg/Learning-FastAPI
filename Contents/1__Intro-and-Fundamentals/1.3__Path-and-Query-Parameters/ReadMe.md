## 1.3. Path and Query Parameters

### Path Parameters

You can declare path "parameters" or "variables" with the same syntax used by Python format strings:

```python
from fastapi import FastAPI

app = FastAPI()

@app.get("/items/{item_id}")
async def read_item(item_id):
    return {"item_id": item_id}
```

The value of the path parameter `item_id` will be passed to your function as the argument `item_id`.

So, if you run this example and go to http://127.0.0.1:8000/items/foo, you will see a response of:

```
{"item_id":"foo"}
```

#### Path parameters with types
You can declare the type of a path parameter in the function, using standard Python type annotations:

```python
from fastapi import FastAPI

app = FastAPI()

@app.get("/items/{item_id}")
async def read_item(item_id: int):
    return {"item_id": item_id}
```

In this case, item_id is declared to be an int.
This will give you editor support inside of your function, with error checks, completion, etc.

##### Data conversion
If you run this example and open your browser at http://127.0.0.1:8000/items/3, you will see a response of:

```
{"item_id":3}
```

Notice that the value your function received (and returned) is `3`, as a Python `int`, not a string "3".
So, with that type declaration, FastAPI gives you automatic request "parsing".

##### Data validation
But if you go to the browser at http://127.0.0.1:8000/items/foo, you will see a nice HTTP error of:

```
{
  "detail": [
    {
      "type": "int_parsing",
      "loc": [
        "path",
        "item_id"
      ],
      "msg": "Input should be a valid integer, unable to parse string as an integer",
      "input": "foo",
      "url": "https://errors.pydantic.dev/2.1/v/int_parsing"
    }
  ]
}
```
because the path parameter `item_id` had a value of "foo", which is not an `int`.

The same error would appear if you provided a `float` instead of an `int`, as in: http://127.0.0.1:8000/items/4.2

So, with the same Python type declaration, FastAPI gives you data validation. 

Notice that the error also clearly states exactly the point where the validation didn't pass.
This is incredibly helpful while developing and debugging code that interacts with your API.


##### Pydantic
All the data validation is performed under the hood by Pydantic, 
so you get all the benefits from it. And you know you are in good hands.

You can use the same type declarations with `str`, `float`, `bool` and many other complex data types.

Several of these are explored in the next chapters of the tutorial.


#### Order matters
When creating path operations, you can find situations where you have a fixed path.

Like `/users/me`, let's say that it's to get data about the current user.

And then you can also have a path `/users/{user_id}` to get data about a specific user by some user ID.

Because path operations are evaluated in order, 
you need to make sure that the path for `/users/me` is declared before the one for `/users/{user_id}`:

```Python 3.8+
from fastapi import FastAPI

app = FastAPI()

@app.get("/users/me")
async def read_user_me():
    return {"user_id": "the current user"}

@app.get("/users/{user_id}")
async def read_user(user_id: str):
    return {"user_id": user_id}
```

Otherwise, the path for `/users/{user_id}` would match also for `/users/me`, 
"thinking" that it's receiving a parameter `user_id` with a value of "me".

Similarly, you cannot redefine a path operation:

```Python 3.8+
from fastapi import FastAPI

app = FastAPI()

@app.get("/users")
async def read_users():
    return ["Rick", "Morty"]

@app.get("/users")
async def read_users2():
    return ["Bean", "Elfo"]
```

The first one will always be used since the path matches first.


#### Path parameters containing paths

Let's say you have a path operation with a path `/files/{file_path}`.

But you need `file_path` itself to contain a path, like `home/johndoe/myfile.txt`.

So, the URL for that file would be something like: `/files/home/johndoe/myfile.txt`.


##### OpenAPI support

OpenAPI doesn't support a way to declare a path parameter to contain a path inside, 
as that could lead to scenarios that are difficult to test and define.

Nevertheless, you can still do it in FastAPI, 
using one of the internal tools from Starlette.

And the docs would still work, 
although not adding any documentation telling that the parameter should contain a path.


##### Path convertor
Using an option directly from Starlette you can declare a path parameter containing a path using a URL like:

```python
/files/{file_path:path}
```

In this case, the name of the parameter is `file_path`, 
and the last part, `:path`, tells that the parameter should match any path.

So, you can use it with:

```Python 3.8+
from fastapi import FastAPI

app = FastAPI()

@app.get("/files/{file_path:path}")
async def read_file(file_path: str):
    return {"file_path": file_path}
```

You could need the parameter to contain `/home/johndoe/myfile.txt`, with a leading slash (`/`). 
In that case, the URL would be: `/files//home/johndoe/myfile.txt`, with a double slash (`//`) between `files` and `home`.


#### Predefined values
If you have a path operation that receives a path parameter, 
but you want the possible valid path parameter values to be predefined, 
you can use a standard Python `Enum`.


##### Create an `Enum` class
Import `Enum` and create a sub-class that inherits from `str` and from `Enum`.

By inheriting from `str` the API docs will be able to know 
that the values must be of type `string` and will be able to render correctly.

Then create class attributes with fixed values, which will be the available valid values:

```Python 3.8+
from enum import Enum

class ModelName(str, Enum):
    alexnet = "alexnet"
    resnet = "resnet"
    lenet = "lenet"
```

Enumerations (or enums) are available in Python since version 3.4.


##### Declare a path parameter
Then create a path parameter with a type annotation using the enum class you created (ModelName):

```Python 3.8+
from enum import Enum
from fastapi import FastAPI

class ModelName(str, Enum):
    alexnet = "alexnet"
    resnet = "resnet"
    lenet = "lenet"

app = FastAPI()

@app.get("/models/{model_name}")
async def get_model(model_name: ModelName):
    if model_name is ModelName.alexnet:
        return {"model_name": model_name, "message": "Deep Learning FTW!"}
    if model_name.value == "lenet":
        return {"model_name": model_name, "message": "LeCNN all the images"}
    return {"model_name": model_name, "message": "Have some residuals"}
```


##### Check the docs
Because the available values for the path parameter are predefined, 
the interactive docs can show them nicely:

<div style="text-align: center;">
    <img 
        src="../../imgs/image03.png" 
        style="width:800px"
    >
</div> 


##### Working with Python enumerations

The value of the path parameter will be an enumeration member.


###### Compare enumeration members
You can compare it with the enumeration member in your created enum ModelName:

```Python 3.8+
    if model_name is ModelName.alexnet:
```


###### Get the enumeration value

You can get the actual value (a `str` in this case) 
using `model_name.value`, or in general, `your_enum_member.value`:

```Python 3.8+
    if model_name.value == "lenet":
```

You could also access the value `"lenet"` with `ModelName.lenet.value`.


###### Return enumeration members

You can return enum members from your path operation, 
even nested in a JSON body (e.g. a `dict`).

They will be converted to their corresponding values (strings in this case) 
before returning them to the client:

```Python 3.8+
    return {"model_name": model_name, "message": "Have some residuals"}
```

In your client you will get a JSON response like:

```
{
  "model_name": "alexnet",
  "message": "Deep Learning FTW!"
}
```


#### Recap
With FastAPI, by using short, intuitive and standard Python type declarations, you get:

- Editor support: error checks, autocompletion, etc.
- Data "parsing"
- Data validation
- API annotation and automatic documentation

And you only have to declare them once.

That's probably the main visible advantage of FastAPI 
compared to alternative frameworks (apart from the raw performance).

-----------------------------------------------------------------------

### Query Parameters

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




### Reference Materials

  - [FastAPI > Learn > Tutorial - User Guide > Path Parameters](https://fastapi.tiangolo.com/tutorial/path-params)
  
  - [FastAPI > Learn > Tutorial - User Guide > Query Parameters](https://fastapi.tiangolo.com/tutorial/query-params)
