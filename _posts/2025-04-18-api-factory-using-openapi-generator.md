---
title: FastAPI Factory Using OpenAPI Generator
layout: post
post-image: "/assets/images/blog/20250418/openapi_gen_helloworld_fastapi.png"
description: Generating FastAPI server using OpenAPI Generator
tags:
- project
- api
- fastapi
- openapi
- generator
- factory
- blog
---

# Introduction

This blog post discusses the use of the [OpenAPI Generator](https://github.com/OpenAPITools/openapi-generator) tool developed by the [OpenAPI Generator](https://openapi-generator.tech/) project to generate Python FastAPI server code from prepared OpenAPI specs. It  allows generation of API client libraries (SDK generation), server stubs, documentation and configuration automatically given an OpenAPI Spec (v2, v3).

**NOTE**: Currently it only supports up to Python `3.12`.

## Local Development Setup

### Install Pyenv

`pyenv` is needed to managed multiple Python versions.

```bash
brew update
brew install pyenv
```

Once installed, update your .zshrc with the following lines:

```bash
echo 'export PYENV_ROOT="$HOME/.pyenv"' >> ~/.zshrc
echo 'command -v pyenv >/dev/null || export PATH="$PYENV_ROOT/bin:$PATH"' >> ~/.zshrc
echo 'eval "$(pyenv init -)"' >> ~/.zshrc
source ~/.zshrc
```

Now you can install and run another version of python in your shell. The following will install Python 3.12 and set your current terminal window to use it.

```bash
pyenv install 3.12.10
pyenv shell 3.12.10
```

### Install OpenAPI Generator

Follow the instructions for [Homebrew](https://github.com/OpenAPITools/openapi-generator?tab=readme-ov-file#15---homebrew).

```bash
brew install openapi-generator
```

### Usage

#### HelloWorld FastAPI Server Generation

Follow the following instructions to generate the FastAPI HelloWorld API server code from the [HelloWorld OpenAPI specs](/assets/yaml/openapi.yaml) prepared in advance as the input.

```bash
cd api/generator/helloworld/server
openapi-generator generate -i openapi.yaml -g python-fastapi -o .
```

OpenAPI Generator will populate the `server` directory with the FastAPI server code.

#### Update Generated Code with Business Logic

The generated code is just a skeleton. Business logic needs to be added. As an example, in the `api/generator/helloworld/server/src/openapi_server/apis/default_api.py`, replace the following code:

```python
async def hello_get(
) -> str:
    """Returns a simple \&quot;Hello World\&quot; message."""
    if not BaseDefaultApi.subclasses:
        raise HTTPException(status_code=500, detail="Not implemented")
    return await BaseDefaultApi.subclasses[0]().hello_get()
```

with the following:

```python
async def hello_get(
) -> str:
    """Returns a simple \&quot;Hello World\&quot; message."""
    return "Hello World!"
```

### Local Deployment & Testing

#### Install Rust and Cargo

OpenAPI Generator requires Rust and Cargo, the Rust package manager, to compile extensions. To install Cargo and Rust using the following command:

```bash
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
```

#### Fix Pytest Code

There is a bug in the function `test_person_post` of the test module `tests/test_default_api.py`. Here is the buggy code:

```python
person_post_request = openapi_server.PersonPostRequest()
```

Replace it with the following code:

```python
person_post_request = PersonPostRequest()
```

#### Update Dockerfile

Update auto-generated `Dockerfile` to use internal Docker registry and image.

#### Deployment and Running via CLI

To run the server, please execute the following from the root directory:

```bash
cd api/generator/helloworld/server
python -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
PYTHONPATH=src uvicorn openapi_server.main:app --host localhost --port 8080
```

#### Deployment and Running via Docker

To run the server on a Docker container, please execute the following from the root directory:

```bash
docker-compose up --build
```

# References
1. [OpenAPI Generator Documentation](https://openapi-generator.tech/)
2. [OpenAPI Generator GitHub](https://github.com/OpenAPITools/openapi-generator)
