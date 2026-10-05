# httpbin(1): HTTP Request & Response Service

<!-- project-guide:start -->
## Project guide

[Project architecture](PROJECT_ARCHITECTURE.md) · [Interview questions and answers](INTERVIEW_QA.md)

Use the architecture document for the component diagram, implementation boundaries, and verification entry points. The interview guide includes source-backed answers and project walkthroughs.

### Implementation map

| Component | Responsibility |
| --- | --- |
| [`httpbin/core.py`](httpbin/core.py) | HTTP handlers: `ROUTE /legacy`, `ROUTE /html`, `ROUTE /robots.txt`, `ROUTE /deny`, `ROUTE /ip` |
| [`httpbin/helpers.py`](httpbin/helpers.py) | Functions: `json_safe`, `get_files`, `get_headers`, `semiflatten`, `get_url`, `get_dict`, `status_code` |
| [`httpbin/filters.py`](httpbin/filters.py) | Functions: `x_runtime`, `gzip`, `deflate`, `brotli` |
| [`setup.py`](setup.py) | Implementation or supporting configuration |
| [`httpbin/__init__.py`](httpbin/__init__.py) | Implementation or supporting configuration |
| [`httpbin/structures.py`](httpbin/structures.py) | Functions: `_lower_keys`, `__contains__`, `__getitem__` |
| [`httpbin/utils.py`](httpbin/utils.py) | Functions: `weighted_choice` |
| [`Dockerfile`](Dockerfile) | Container build/service configuration |
| [`docker-compose.yml`](docker-compose.yml) | Container build/service configuration |
| [`test_httpbin.py`](test_httpbin.py) | Executable checks and regression examples |
| [`README.md`](README.md) | Project explanations or operating notes |

Setup and examples are described in the existing project notes below. Consult the component-specific manifests before assuming a single launch command.

<!-- project-guide:end -->

<!-- repository-summary -->
A fork of the Python and Flask HTTP request and response inspection service.
<!-- /repository-summary -->


A [Kenneth Reitz](http://kennethreitz.org/bitcoin) Project.

![ice cream](http://farm1.staticflickr.com/572/32514669683_4daf2ab7bc_k_d.jpg)

Run locally:
```sh
docker pull kennethreitz/httpbin
docker run -p 80:80 kennethreitz/httpbin
```

See http://httpbin.org for more information.

## Officially Deployed at:

- http://httpbin.org
- https://httpbin.org
- https://hub.docker.com/r/kennethreitz/httpbin/


## SEE ALSO

- http://requestb.in
- http://python-requests.org
- https://grpcb.in/

## Build Status

[![Build Status](https://travis-ci.org/requests/httpbin.svg?branch=master)](https://travis-ci.org/requests/httpbin)
