## 1.3. Path Parameters


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

### Path parameters with types
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

#### Data conversion
If you run this example and open your browser at http://127.0.0.1:8000/items/3, you will see a response of:

```
{"item_id":3}
```

Notice that the value your function received (and returned) is `3`, as a Python `int`, not a string "3".
So, with that type declaration, FastAPI gives you automatic request "parsing".

#### Data validation
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


#### Pydantic
All the data validation is performed under the hood by Pydantic, 
so you get all the benefits from it. And you know you are in good hands.

You can use the same type declarations with `str`, `float`, `bool` and many other complex data types.

Several of these are explored in the next chapters of the tutorial.


### Order matters
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


### Path parameters containing paths

Let's say you have a path operation with a path `/files/{file_path}`.

But you need `file_path` itself to contain a path, like `home/johndoe/myfile.txt`.

So, the URL for that file would be something like: `/files/home/johndoe/myfile.txt`.


#### OpenAPI support

OpenAPI doesn't support a way to declare a path parameter to contain a path inside, 
as that could lead to scenarios that are difficult to test and define.

Nevertheless, you can still do it in FastAPI, 
using one of the internal tools from Starlette.

And the docs would still work, 
although not adding any documentation telling that the parameter should contain a path.


#### Path convertor
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


### Predefined values
If you have a path operation that receives a path parameter, 
but you want the possible valid path parameter values to be predefined, 
you can use a standard Python `Enum`.


#### Create an `Enum` class
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


#### Declare a path parameter
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


#### Check the docs
Because the available values for the path parameter are predefined, 
the interactive docs can show them nicely:

<div style="text-align: center;">
    <img 
        src="../../imgs/image03.png" 
        style="width:800px"
    >
</div> 


#### Working with Python enumerations

The value of the path parameter will be an enumeration member.


##### Compare enumeration members
You can compare it with the enumeration member in your created enum ModelName:

```Python 3.8+
    if model_name is ModelName.alexnet:
```


##### Get the enumeration value

You can get the actual value (a `str` in this case) 
using `model_name.value`, or in general, `your_enum_member.value`:

```Python 3.8+
    if model_name.value == "lenet":
```

You could also access the value `"lenet"` with `ModelName.lenet.value`.


##### Return enumeration members

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


### Additional Validations

In the same way that you can declare [more validations and metadata for query parameters with `Query`](../1.5__Query-Parameters/ReadMe.md#add-more-validations), 
you can declare the same type of validations and metadata for path parameters with `Path`.


#### Import `Path`
First, import `Path` from `fastapi`, and import `Annotated`:

```Python 3.10+
from typing import Annotated
from fastapi import FastAPI, Path, Query
```

FastAPI added support for `Annotated` (and started recommending it) in version 0.95.0.

If you have an older version, you would get errors when trying to use `Annotated`.

Make sure you [Upgrade the FastAPI version](https://fastapi.tiangolo.com/deployment/versions/#upgrading-the-fastapi-versions) 
to at least 0.95.1 before using `Annotated`.


#### Declare metadata

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


### Recap
With FastAPI, by using short, intuitive and standard Python type declarations, you get:

- Editor support: error checks, autocompletion, etc.
- Data "parsing"
- Data validation
- API annotation and automatic documentation

And you only have to declare them once.

That's probably the main visible advantage of FastAPI 
compared to alternative frameworks (apart from the raw performance).

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

  - [FastAPI > Learn > Tutorial - User Guide > Path Parameters](https://fastapi.tiangolo.com/tutorial/path-params)
  
  - [FastAPI > Learn > Tutorial - User Guide > Path Parameters and Numeric Validations](https://fastapi.tiangolo.com/tutorial/path-params-numeric-validations)