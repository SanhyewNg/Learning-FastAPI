## 2.4. Request Parameter Models

### Query Parameter Models¶
If you have a group of query parameters that are related, you can create a Pydantic model to declare them.

This would allow you to re-use the model in multiple places and also to declare validations and metadata for all the parameters at once. 😎

Note

This is supported since FastAPI version 0.115.0. 🤓

Query Parameters with a Pydantic Model¶
Declare the query parameters that you need in a Pydantic model, and then declare the parameter as Query:


Python 3.10+

from typing import Annotated, Literal

from fastapi import FastAPI, Query
from pydantic import BaseModel, Field

app = FastAPI()


class FilterParams(BaseModel):
    limit: int = Field(100, gt=0, le=100)
    offset: int = Field(0, ge=0)
    order_by: Literal["created_at", "updated_at"] = "created_at"
    tags: list[str] = []


@app.get("/items/")
async def read_items(filter_query: Annotated[FilterParams, Query()]):
    return filter_query

🤓 Other versions and variants
FastAPI will extract the data for each field from the query parameters in the request and give you the Pydantic model you defined.

Check the Docs¶
You can see the query parameters in the docs UI at /docs:


Forbid Extra Query Parameters¶
In some special use cases (probably not very common), you might want to restrict the query parameters that you want to receive.

You can use Pydantic's model configuration to forbid any extra fields:


Python 3.10+

from typing import Annotated, Literal

from fastapi import FastAPI, Query
from pydantic import BaseModel, Field

app = FastAPI()


class FilterParams(BaseModel):
    model_config = {"extra": "forbid"}

    limit: int = Field(100, gt=0, le=100)
    offset: int = Field(0, ge=0)
    order_by: Literal["created_at", "updated_at"] = "created_at"
    tags: list[str] = []


@app.get("/items/")
async def read_items(filter_query: Annotated[FilterParams, Query()]):
    return filter_query

🤓 Other versions and variants
If a client tries to send some extra data in the query parameters, they will receive an error response.

For example, if the client tries to send a tool query parameter with a value of plumbus, like:


https://example.com/items/?limit=10&tool=plumbus
They will receive an error response telling them that the query parameter tool is not allowed:


{
    "detail": [
        {
            "type": "extra_forbidden",
            "loc": ["query", "tool"],
            "msg": "Extra inputs are not permitted",
            "input": "plumbus"
        }
    ]
}
Summary¶
You can use Pydantic models to declare query parameters in FastAPI. 😎

Tip

Spoiler alert: you can also use Pydantic models to declare cookies and headers, but you will read about that later in the tutorial. 🤫


### Cookie Parameter Models¶
If you have a group of cookies that are related, you can create a Pydantic model to declare them. 🍪

This would allow you to re-use the model in multiple places and also to declare validations and metadata for all the parameters at once. 😎

Note

This is supported since FastAPI version 0.115.0. 🤓

Tip

This same technique applies to Query, Cookie, and Header. 😎

Cookies with a Pydantic Model¶
Declare the cookie parameters that you need in a Pydantic model, and then declare the parameter as Cookie:


Python 3.10+

from typing import Annotated

from fastapi import Cookie, FastAPI
from pydantic import BaseModel

app = FastAPI()


class Cookies(BaseModel):
    session_id: str
    fatebook_tracker: str | None = None
    googall_tracker: str | None = None


@app.get("/items/")
async def read_items(cookies: Annotated[Cookies, Cookie()]):
    return cookies

🤓 Other versions and variants
FastAPI will extract the data for each field from the cookies received in the request and give you the Pydantic model you defined.

Check the Docs¶
You can see the defined cookies in the docs UI at /docs:


Info

Have in mind that, as browsers handle cookies in special ways and behind the scenes, they don't easily allow JavaScript to touch them.

If you go to the API docs UI at /docs you will be able to see the documentation for cookies for your path operations.

But even if you fill the data and click "Execute", because the docs UI works with JavaScript, the cookies won't be sent, and you will see an error message as if you didn't write any values.

Forbid Extra Cookies¶
In some special use cases (probably not very common), you might want to restrict the cookies that you want to receive.

Your API now has the power to control its own cookie consent. 🤪🍪

You can use Pydantic's model configuration to forbid any extra fields:


Python 3.9+

from typing import Annotated, Union

from fastapi import Cookie, FastAPI
from pydantic import BaseModel

app = FastAPI()


class Cookies(BaseModel):
    model_config = {"extra": "forbid"}

    session_id: str
    fatebook_tracker: Union[str, None] = None
    googall_tracker: Union[str, None] = None


@app.get("/items/")
async def read_items(cookies: Annotated[Cookies, Cookie()]):
    return cookies

🤓 Other versions and variants
If a client tries to send some extra cookies, they will receive an error response.

Poor cookie banners with all their effort to get your consent for the API to reject it. 🍪

For example, if the client tries to send a santa_tracker cookie with a value of good-list-please, the client will receive an error response telling them that the santa_tracker cookie is not allowed:


{
    "detail": [
        {
            "type": "extra_forbidden",
            "loc": ["cookie", "santa_tracker"],
            "msg": "Extra inputs are not permitted",
            "input": "good-list-please",
        }
    ]
}
Summary¶
You can use Pydantic models to declare cookies in FastAPI. 😎


### Header Parameter Models¶
If you have a group of related header parameters, you can create a Pydantic model to declare them.

This would allow you to re-use the model in multiple places and also to declare validations and metadata for all the parameters at once. 😎

Note

This is supported since FastAPI version 0.115.0. 🤓

Header Parameters with a Pydantic Model¶
Declare the header parameters that you need in a Pydantic model, and then declare the parameter as Header:


Python 3.10+

from typing import Annotated

from fastapi import FastAPI, Header
from pydantic import BaseModel

app = FastAPI()


class CommonHeaders(BaseModel):
    host: str
    save_data: bool
    if_modified_since: str | None = None
    traceparent: str | None = None
    x_tag: list[str] = []


@app.get("/items/")
async def read_items(headers: Annotated[CommonHeaders, Header()]):
    return headers

🤓 Other versions and variants
FastAPI will extract the data for each field from the headers in the request and give you the Pydantic model you defined.

Check the Docs¶
You can see the required headers in the docs UI at /docs:


Forbid Extra Headers¶
In some special use cases (probably not very common), you might want to restrict the headers that you want to receive.

You can use Pydantic's model configuration to forbid any extra fields:


Python 3.10+

from typing import Annotated

from fastapi import FastAPI, Header
from pydantic import BaseModel

app = FastAPI()


class CommonHeaders(BaseModel):
    model_config = {"extra": "forbid"}

    host: str
    save_data: bool
    if_modified_since: str | None = None
    traceparent: str | None = None
    x_tag: list[str] = []


@app.get("/items/")
async def read_items(headers: Annotated[CommonHeaders, Header()]):
    return headers

🤓 Other versions and variants
If a client tries to send some extra headers, they will receive an error response.

For example, if the client tries to send a tool header with a value of plumbus, they will receive an error response telling them that the header parameter tool is not allowed:


{
    "detail": [
        {
            "type": "extra_forbidden",
            "loc": ["header", "tool"],
            "msg": "Extra inputs are not permitted",
            "input": "plumbus",
        }
    ]
}
Summary¶
You can use Pydantic models to declare headers in FastAPI. 😎





### Reference Materials 

  - [FastAPI > Learn > Tutorial - User Guide > Query Parameter Models](https://fastapi.tiangolo.com/tutorial/query-param-models/)

  - [FastAPI > Learn > Tutorial - User Guide > Cookie Parameter Models](https://fastapi.tiangolo.com/tutorial/cookie-param-models/)

  - [FastAPI > Learn > Tutorial - User Guide > Header Parameter Models](https://fastapi.tiangolo.com/tutorial/header-param-models/)

