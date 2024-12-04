## Declaring Request Example Data

You can declare examples of the data your app can receive.

Here are several ways to do it.


### Extra JSON Schema data in Pydantic models
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


### `Field` additional arguments

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

### examples in JSON Schema - OpenAPI

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

  - [FastAPI > Learn > Tutorial - User Guide > Declare Request Example Data](https://fastapi.tiangolo.com/tutorial/schema-extra-example)
  