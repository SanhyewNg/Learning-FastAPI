## 1.1. FastAPI Introduction and Development Environment

### Introduction to FastAPI 

FastAPI is a modern, fast (high-performance), web framework for building APIs with Python based on standard Python type hints.

#### Key Features

- **Fast**: Very high performance, on par with NodeJS and Go (thanks to Starlette and Pydantic). 
  One of the fastest Python frameworks available (https://fastapi.tiangolo.com/#performance).
- **Fast to code**: Increase the speed to develop features by about 200% to 300% (estimation based on tests on an internal development team, building production applications). 
- **Fewer bugs**: Reduce about 40% of human (developer) induced errors. *
- **Intuitive**: Great editor support. Completion (also known as auto-complete, autocompletion, IntelliSense) everywhere. Less time debugging.
- **Easy**: Designed to be easy to use and learn. Less time reading docs.
- **Short**: Minimize code duplication. Multiple features from each parameter declaration. Fewer bugs.
- **Robust**: Get production-ready code. With automatic interactive documentation.
- **Standards-based**: Based on (and fully compatible with) the open standards for APIs: [OpenAPI](https://github.com/OAI/OpenAPI-Specification) (previously known as Swagger) and [JSON Schema](https://json-schema.org/).


#### FastAPI's Architecture

FastAPI's architecture is based on several key components: 
1. **ASGI (Asynchronous Server Gateway Interface)**: This is the foundation of FastAPI's high performance. 
   ASGI allows for handling of asynchronous requests, which is crucial for building scalable applications. 

2. **Starlette**: FastAPI is built on top of Starlette, 
   which provides the core functionality for handling HTTP requests and responses. 

3. **Pydantic**: FastAPI uses Pydantic for data validation, serialization, and documentation. 
   Pydantic leverages Python type annotations to define data models. 

4. **Dependency Injection System**: FastAPI includes a powerful dependency injection system 
   that makes it easy to write clean, modular code. 

5. **OpenAPI and JSON Schema**: FastAPI automatically generates 
   OpenAPI (Swagger) and JSON Schema documentation for your API. 


#### FastAPI vs Other Frameworks

FastAPI stands out from other Python web frameworks in several ways: 
1. **Performance**: FastAPI is one of the fastest Python frameworks available, 
   comparable to Node.js and Go in terms of performance. 

2. **Type Hints**: FastAPI makes extensive use of Python type hints, 
   which improves code quality and developer experience. 

3. **Automatic Documentation**: FastAPI automatically generates interactive API documentation 
   (Swagger UI) and OpenAPI schema. 

4. **Data Validation**: With Pydantic integration, FastAPI provides robust data validation out of the box. 

5. **Asynchronous Support**: FastAPI is built for asynchronous programming, 
   making it ideal for building high-performance, scalable applications. 


### Setting Up FastAPI Development Environment

#### System Requirements

FastAPI requires Python 3.6+. It's recommended to use the latest version of Python available, 
as FastAPI takes advantage of the latest Python features. 


#### Install Python Distribution 

[Install Python Standard Distribution on Windows](https://docs.python.org/3.9/using/windows.html#installation-steps)

[Install Anaconda Distribution on Windows]()


#### Install VS Code and the Python Extensions



#### Create a Virtual Environment

It's a good practice to create a virtual environment for your FastAPI project. 
This isolates your project dependencies from your system-wide Python installation.




#### Install FastAPI package

Once your virtual environment is activated, you can install FastAPI using pip: 
```bash
pip install fastapi
```

#### Development Tools

While not strictly necessary, the following tools can enhance your FastAPI development experience: 
1. Black: A code formatter that ensures consistent code style. 
2. Flake8: A linter that checks your code for style and programming errors. 
3. Mypy: A static type checker for Python. 

You can install these tools with: 
```cmd
pip install black flake8 mypy
```

#### Project Structure

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


### Reference Materials

  - [FastAPI Official Documentation - Home ppage](https://fastapi.tiangolo.com)
  - [[eBook] Mastering FastAPI: Building Modern, High-Performance APIs with Python by Laszlo Bocso, 2024.11.5](https://www.amazon.com/Mastering-FastAPI-Building-Modern-High-Performance-ebook/dp/B0DM6MDLRV)
