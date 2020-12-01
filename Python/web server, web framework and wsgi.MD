## Introduction

WSGI is an interface between a web server and a web framework/application. You can run any web framework/application on any web server as long as both of them comply with the WSGI interface, which introduces great flexibility when building a website with python.

### Example

A simple WSGI application is like the following:

```python
def application(environ, start_response):
    start_response('200 OK', [('Content-Type', 'text/html')])
    return [b'<h1>Hello, web!</h1>']
```

and the HTTP request processing flow is like the following:

![figure](F:\Notes\Python\web server, web framework and wsgi_1.png)

Let's look into details of each step:

1. The client sends the HTTP requests.
2. The WSGI parses the HTTP request and use the parsing result to build the `environ` object. `environ` is a dictionary which contains variables defined in the CGI standard. For more details about this object, you can refer [PEP 3333](https://www.python.org/dev/peps/pep-3333/#environ-variables). 
3. The WSGI call the WSGI application with two parameters, 