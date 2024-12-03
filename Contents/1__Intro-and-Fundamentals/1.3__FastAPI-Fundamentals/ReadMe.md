## 1.3. FastAPI Fundamentals

### How FastAPI Works

FastAPI is built on top of *Starlette*, a lightweight ASGI framework/toolkit, 
which is ideal for building high-performance asyncio services. 
It also uses *Pydantic* for data validation and serialization. 

When you create a FastAPI application, you define routes (endpoints) using Python functions. 
These functions are decorated with FastAPI decorators 
that specify the HTTP method and path for each endpoint. 
FastAPI then uses these decorations to generate OpenAPI (Swagger) documentation automatically. 


### The simplest FastAPI Application

Create a new file called `main.py` with the following content:

```python
from fastapi import FastAPI

app = FastAPI()

@app.get("/")
async def root():
    return {"message": "Hello World"}
```

This simple application does the following: 
1. Imports FastAPI
2. Creates an instance of the FastAPI class 
3. Defines a route for the root path ("/") that responds to GET requests
4. Returns a JSON object with a "Hello World" message

To run this application, use the following command:

```bash
uvicorn main:app --reload
```

The `--reload` flag enables auto-reloading, which is useful during development.

Now, if you open a web browser and navigate to http://localhost:8000, 
you should see the JSON response:
```
{'message’: “Hello World") .
```

**Reference**: [FastAPI > Learn > Tutorial - User Guide > First Steps](https://fastapi.tiangolo.com/tutorial/first-steps)


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









### Query Parameters




### Project Structure

Here's a recommended project structure for a FastAPI application: 
```
my_project/
|-- app /
|   |-- __init__.py
|   |-- main.py
|   |-- dependencies.py
|   |-- routers/
|       |-- __init__.py
|       |-- items.py
|       |-- users.py
|
|-- models /
|   |-- __init__.py
|   |-- item.py
|   |-- user.py
|
|-- tests/
|   |-- __init__.py
|   |-- test_main.py
|   |-- test_items.py
|   |-- test_users.py
|
|-- venv/
|-- .gitignore
|-- requirements.txt
|-- README.md
```