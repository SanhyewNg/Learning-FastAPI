## 1.2. Setting Up FastAPI Development Environment

### System Requirements

FastAPI requires Python 3.6+. It's recommended to use the latest version of Python available, 
as FastAPI takes advantage of the latest Python features. 


### Install Python Distribution 

[Install Python Standard Distribution on Windows](https://docs.python.org/3.9/using/windows.html#installation-steps)

[Install Anaconda Distribution on Windows]()


### Install VS Code and the Python Extensions



### Create a Virtual Environment

It's a good practice to create a virtual environment for your FastAPI project. 
This isolates your project dependencies from your system-wide Python installation.




### Install FastAPI package

Once your virtual environment is activated, you can install FastAPI using pip: 
```bash
pip install fastapi
```

### Development Tools

While not strictly necessary, the following tools can enhance your FastAPI development experience: 
1. Black: A code formatter that ensures consistent code style. 
2. Flake8: A linter that checks your code for style and programming errors. 
3. Mypy: A static type checker for Python. 

You can install these tools with: 
```cmd
pip install black flake8 mypy
```



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