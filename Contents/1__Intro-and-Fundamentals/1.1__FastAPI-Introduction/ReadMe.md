## 1.1. FastAPI Introduction


FastAPI is a modern, fast (high-performance), web framework for building APIs with Python based on standard Python type hints.

### Key Features

- **Fast**: Very high performance, on par with NodeJS and Go (thanks to Starlette and Pydantic). 
  One of the fastest Python frameworks available (https://fastapi.tiangolo.com/#performance).
- **Fast to code**: Increase the speed to develop features by about 200% to 300% (estimation based on tests on an internal development team, building production applications). 
- **Fewer bugs**: Reduce about 40% of human (developer) induced errors. *
- **Intuitive**: Great editor support. Completion (also known as auto-complete, autocompletion, IntelliSense) everywhere. Less time debugging.
- **Easy**: Designed to be easy to use and learn. Less time reading docs.
- **Short**: Minimize code duplication. Multiple features from each parameter declaration. Fewer bugs.
- **Robust**: Get production-ready code. With automatic interactive documentation.
- **Standards-based**: Based on (and fully compatible with) the open standards for APIs: [OpenAPI](https://github.com/OAI/OpenAPI-Specification) (previously known as Swagger) and [JSON Schema](https://json-schema.org/).


### FastAPI's Architecture

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


### FastAPI vs Other Frameworks

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


### Reference Materials

  - [FastAPI Official Documentation - Home ppage](https://fastapi.tiangolo.com)
  - [[eBook] Mastering FastAPI: Building Modern, High-Performance APIs with Python by Laszlo Bocso, 2024.11.5](https://www.amazon.com/Mastering-FastAPI-Building-Modern-High-Performance-ebook/dp/B0DM6MDLRV)