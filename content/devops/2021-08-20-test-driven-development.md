---
title: Test Driven Development
date: "2021-08-20T08:00:00Z"
categories:
- devops
---

So, I started learning [Golang](https://golang.org) recently, and decided that [Learn Go with Tests](https://quii.gitbook.io/learn-go-with-tests/) was a good way to go. However, there was a great 'buy-one-get-one-free' with this resource - it also taught me some of the benefits of Test Driven Development (TDD). There are four steps to TDD:

- Write the test
- Write a minimal amount of code for the test to recognise the code (e.g. write the function and just return - the test should still fail)
- Write the function correctly to return the expected output
- Refactor (change the code to better represent the test case)

I'm not using Go that much at work, with [Python](https://python.org) being  the dominant language (for now). This meant I had to learn how to use tests in Python. This blog post represents my learning and notes. 

### Install Pytest

Pytest does not come with the standard library, so does require an install. 

To install `pytest`, you just run `pip install -u pytest`. 

### Write the test

So, my initial test was to just add two numbers together:


    from src.add import add

    def test_add():
        assert add(5, 7) == 12

Here, the code asserts that if I pass `5` and `7` to my `add` function, the function will return `12`. If this returns anything different, then the test will fail. 

#### Directory Structure

This is within a `tests` directory. 

My project file structure is thus:

    project/
         | 
    	 |- src
         |
         |- tests

The initial problem that needs to be solved is that the test needs to import the add module. To help with this, we need to add `__init__.py` files to the src directory 

The initial problem that needs to be solved is that the test needs to import the add module. To help with this, we need to add `__init__.py` files to the `src` and `tests` directory. So this is how the file structure looks now:
    
    project/
         | 
    	 |- src
         |    |- __init__.py
         |- tests
              |- test_add.py
	          |- __init__.py

However, the import will still not work. You will need a `setup.py` file in the top directory to help with that:

    from setuptools import setup, find_packages

    setup(name="add", packages=find_packages())

This file needs to be at the project level: 

    project/
         |- setup.py
    	 |- src
         |    |- __init__.py
         |- tests
              |- test_add.py
	          |- __init__.py
The important part of the process now is that you run the test to prove that it fails. Run `pytest` in the project directory. Pytest automatically looks for directories and files that start with `test`:

    ================================================= test session starts =================================================
    platform win32 -- Python 3.9.6, pytest-6.2.4, py-1.10.0, pluggy-0.13.1
    rootdir: C:\Users\adamc\Documents\test
    plugins: anyio-3.3.0
    collected 0 items / 1 error
    
    ======================================================= ERRORS ========================================================
    _________________________________________ ERROR collecting tests/test_add.py __________________________________________
    importError while importing test module 'C:\Users\adamc\Documents\test\tests\test_add.py'.
    hint: make sure your test modules/packages have valid Python names.
    traceback:
    ..\..\appdata\local\programs\python\python39\lib\importlib\__init__.py:127: in import_module
        return _bootstrap._gcd_import(name[level:], package, level)
    tests\test_add.py:1: in <module>
        from src.add import add
    e   ModuleNotFoundError: No module named 'src.add'
    =============================================== short test summary info ===============================================
    eRROR tests/test_add.py
    !!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!! Interrupted: 1 error during collection !!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!
    ================================================== 1 error in 0.21s ===================================================


This proves that the error is failing. 

Then we need to create the file in the `src` directory to help this pass. Even at the moment, the import is not finding the `src` directory, which needs to be fixed.

### Write the minimal amount of code 

We need a file `add.py` in the `src` directory. The first iteration of the file we can add is just a function that returns:

    def add():
        return

This will change the nature of the test - it will run, but will now fail properly:
    
    ================================================= test session starts =================================================
    platform win32 -- Python 3.9.6, pytest-6.2.4, py-1.10.0, pluggy-0.13.1
    rootdir: C:\Users\adamc\Documents\test
    plugins: anyio-3.3.0
    collected 1 item
    
    tests\test_add.py F                                                                                              [100%]
    
    ====================================================== FAILURES =======================================================
    ______________________________________________________ test_add _______________________________________________________
    
        def test_add():
    >       assert add(5, 7) == 12
    E       TypeError: add() takes 0 positional arguments but 2 were given
    
    tests\test_add.py:4: TypeError
    =============================================== short test summary info ===============================================
    FAILED tests/test_add.py::test_add - TypeError: add() takes 0 positional arguments but 2 were given
    ================================================== 1 failed in 0.24s ==================================================

So, pytest is now importing the function, and passing in `5` and `7` parameters, but only getting a return (the function doesn't accept any values at the moment). 

Let's fix that:

    def add(x, y):
        return x + y

If we run `pytest` we now get:

    ================================================= test session starts =================================================
    platform win32 -- Python 3.9.6, pytest-6.2.4, py-1.10.0, pluggy-0.13.1
    rootdir: C:\Users\adamc\Documents\test
    plugins: anyio-3.3.0
    collected 1 item
    
    tests\test_add.py .                                                                                              [100%]
    
    ================================================== 1 passed in 0.06s ==================================================

The good news here is the `tests\test_add.py .` section - specifically the `.` character. This outlines that the test passed.       

### Refactor the code

This function is very simple and doesn't need any refactoring. It might be that my test needs refactoring, or my code needs refactoring. Here this isn't necessary, but it might be that there is some duplication in my code, or that I need to change a special case (for example, divide by zero). 

### Summary 

This is just an introduction to testing. There are many more ways tests can be 'named' and then run, but this just aims to get a simple test suite up and running. For more information, consult the [pytest documentation](https://docs.pytest.org/en/6.2.x/).
