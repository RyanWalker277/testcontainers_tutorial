# Testcontainers_Tutorial

## What is testcontainer library for python?

Testcontainers for Python is a Python library that provides lightweight, disposable containers for running integration tests. With testcontainers, developers can easily create, manage and dispose of containers for various databases, message queues, and other services that their application relies on during testing. The library integrates seamlessly with popular testing frameworks such as Pytest and unittest, and can be used in both local and CI/CD environments. By using testcontainers, developers can ensure that their tests are reliable and isolated, without the need to manually set up and tear down dependencies for each test.

## Issues with testcontainers library

The issue with testcontainers is that it currently supports only a limited amount of containers, which means that if you need to use a container that is not supported out of the box, you will have to implement the container yourself.

## Now, how to add a container yourself?

In this tutorial we will be using FusionAuth as an example.

Firstly, Install the testcontainers and requests package

```
pip install testcontainers requests
```

Next, you need to install the Docker Python SDK. You can do this by running the following command:

```
pip install docker
```

Make a directory for your container in the project root. In this case we will be creating FusionAuth

```
mkdir FusionAuth
```

This directory is the directory for your container module. The directory structure for each container in the `testcontainers` looks liek this:

```
├── README.rst
├── setup.py
├── testcontainers
│   └── container-name
│       └── __init__.py
└── tests
    └── test_container-name.py
```

<dl>
  <dt><strong>Setup.py</strong></dt>
  <dd>setup.py is a script used to build, distribute, and install Python packages. It is used by the Python packaging tools such as pip, setuptools, and easy_install to install and manage Python packages.

  The setup.py file usually contains some metadata about the package, such as the name, version, author, license, and description. It also includes instructions on how to install and run the package.</dd>
  <dt><strong>__init__.py</strong></dt>
  <dd>The __init__.py file is a special file in Python that is executed when a package is imported. It can be used to define various package-level attributes, such as version information, author information, and documentation strings.

  In addition, the __init__.py file can contain import statements, which are executed when the package is imported. This allows you to organize your code into multiple modules and subpackages, and have them automatically loaded when the main package is imported.</dd>
</dl>

Now let's add a `_init_.py` file. Taking the example of FusionAuth, the _init_.py would be :

```
from testcontainers.core.container import DockerContainer

class FusionAuthContainer(DockerContainer):
    def __init__(self, version="latest"):
        super(FusionAuthContainer, self).__init__(
            "fusionauth/fusionauth-app:latest",
            version=version,
        )

    def is_healthy(self):
        response = self.exec(["curl", "http://localhost:9011/.well-known/jwks.json"])
        print(response)

__all__ = ["FusionAuthContainer"]

```

Now, let's understand what the above code does. 

Here we have defined a custom Docker container class called `FusionAuthContainer`, which inherits from `DockerContainer`. When an instance of `FusionAuthContainer` is created, it sets the Docker image to "fusionauth/fusionauth-app:latest" and the version to "latest".

Additionally, the `is_healthy` method is overridden to check if the container is healthy by executing a curl command to retrieve the JWKS (JSON Web Key Set) endpoint, which is expected to be running at http://localhost:9011/.well-known/jwks.json. The method returns True if the response is successful, otherwise it returns False.

The `__all__` variable is a list that specifies which symbols should be imported when using `from <module> import *` syntax. In this case, it specifies that only FusionAuthContainer should be imported.

Now let's add a `setup.py` file to FusionAuth directory.

```
from setuptools import setup, find_namespace_packages

description = "FusionAuth component of testcontainers-python."

setup(
    name="testcontainers-fusionauth",
    version="0.1.0",
    packages=find_namespace_packages(),
    description=description,
    long_description=description,
    long_description_content_type="text/x-rst",
    url="https://github.com/testcontainers/testcontainers-python",
    install_requires=[
        "testcontainers-core",
    ],
    python_requires=">=3.7",
)
```
Now, we just need to add a line `-e file:[service name]` to `requirements.in` and run `make requirements`. This command will find any new requirements and generate lock files to ensure reproducible builds. Then run `pip install -r requirements/[your python version].txt` to install the new requirements.

In this way, you can add your own containers to `testcontainers-python`. Keep in mind the above implementation was limited in it's scope. The custom container class defined by you can have much more attributes depending on the container added.