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
        return response.exit_code == 0

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

## How to define hooks for container life cycle management?

The container can use predefined `DockerContainer` or `DockerCompose` class methods for managing the the life cycle. Ther are a lot of pre-defined methods there which can be used to create hooks based on the use case of the specefic container. Here is a small description of each of them: 

DockerContainer:

- `with_env()`: Sets environment variables for the Docker container.
- `with_bind_ports()`: Binds container ports to host ports.
- `with_exposed_ports()`: Exposes container ports to the network.
- `with_kwargs()`: Sets additional keyword arguments for the Docker container.
- `maybe_emulate_amd64()`: Sets platform to "linux/amd64" if running on ARM architecture.
- `start()`: Starts the Docker container.
- `stop()`: Stops the Docker container.
- `enter()`: Starts the Docker container when used as a context manager.
- `exit()`: Stops the Docker container when used as a context manager.
- `del()`: Tries to remove the Docker container in all circumstances.
- `get_container_host_ip()`: Returns the host IP address of the Docker container.
- `get_exposed_port()`: Returns the exposed port for a specified container port.
- `with_command()`: Sets the command to be run in the Docker container.
- `with_name()`: Sets the name of the Docker container.
- `with_volume_mapping()`: Sets a host and container directory mapping for the Docker container.
- `get_wrapped_container()`: Returns the wrapped Docker container object.
- `get_docker_client()`: Returns the Docker client.
- `get_logs()`: Returns the logs of the Docker container.
- `exec()`: Executes a command in the Docker container.

DockerCompose:

- `docker_compose_command`: Returns a list of command parts for running docker-compose commands.
- `start`: Starts the docker-compose environment, with the option to pull and/or build images.
- `stop`: Stops the docker-compose environment and removes associated volumes.
- `get_logs`: Returns all log output from stdout and stderr.
- `exec_in_container`: Executes a command in the container of one of the services.
- `get_service_port`: Returns the mapped port for one of the services.
- `get_service_host`: Returns the hostname for one of the services.
- `_get_service_info`: Returns a list with the hostname and mapped port for one of the services.
- `_call_command`: Calls a given command with the option to specify a working directory.
- `wait_for`: Waits for a response from a given URL, typically used to block until a service is ready.

## So, why are they called "test" containers?

Since they are largely employed in software testing, test containers are referred to as "Test" containers. They offer a simple method for conducting tests in a setting that closely resembles a production environment. Test containers are perfect for running tests repeatedly and fast, which is a crucial requirement in software testing. They are lightweight and can be spun up and broken down quickly. Developers may make sure their code operates in a consistent and reproducible environment by using test containers, which helps to find flaws and failures before they are published to production.

## Are there any limitations?

Test-container tests can be slow because they require starting and stopping Docker containers, which can take several seconds or even minutes depending on the complexity of the container and the resources available on the testing machine. Additionally, if multiple containers need to be started and configured for a single test case, the time required for setup and teardown can be significant. This can result in longer feedback loops, which can slow down the overall development and testing process. However, the benefits of using test-containers, such as increased test reliability and ease of use, often outweigh the added time required for testing.

## Can the speed limitations be conquered?

Fortunately, there are a few ways to speed up the tests.

- Use the Singleton Pattern
- Speed-Up Start Time with Multiple Containers:
- Reuse the testcontainers
- Running the tests in a Docker Container (Docker in Docker)

[Refrence](https://callistaenterprise.se/blogg/teknik/2020/10/09/speed-up-your-testcontainers-tests/)

## Why does it only support only a limited number of containers?

The reason behind the limited support of containers by the testcontainers library is attributed to the relatively small size of its contributor-base. However, the library offers the advantage of being highly adaptable to modifications, enabling it to effectively cater to specific container requirements. Despite the limited number of supported containers, the library maintains its flexibility and versatility, which can prove to be advantageous in certain situations. Additionally, the testcontainers library is continuously evolving, and with the growth of its contributor-base, it is expected to expand its support for a wider range of containers in the future.