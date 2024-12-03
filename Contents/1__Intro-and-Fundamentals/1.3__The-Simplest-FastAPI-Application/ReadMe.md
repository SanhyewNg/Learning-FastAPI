## 1.3. The Simplest FastAPI Application

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
{'message’: “Hello World"} .
```

**Reference**: [FastAPI > Learn > Tutorial - User Guide > First Steps](https://fastapi.tiangolo.com/tutorial/first-steps)


