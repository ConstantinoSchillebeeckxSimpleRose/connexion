Connexion
=========

.. image:: https://badges.gitter.im/zalando/connexion.svg
   :alt: Join the chat at https://gitter.im/zalando/connexion
   :target: https://gitter.im/zalando/connexion?utm_source=badge&utm_medium=badge&utm_campaign=pr-badge&utm_content=badge

.. image:: https://travis-ci.org/zalando/connexion.svg?branch=master
   :target: https://travis-ci.org/zalando/connexion
   :alt: Travis CI build status

.. image:: https://coveralls.io/repos/zalando/connexion/badge.svg?branch=master
   :target: https://coveralls.io/r/zalando/connexion?branch=master
   :alt: Coveralls status

.. image:: https://img.shields.io/pypi/v/connexion.svg
   :target: https://pypi.python.org/pypi/connexion
   :alt: Latest Version

.. image:: https://img.shields.io/pypi/status/connexion.svg
   :target: https://pypi.python.org/pypi/connexion
   :alt: Development Status

.. image:: https://img.shields.io/pypi/pyversions/connexion.svg
   :target: https://pypi.python.org/pypi/connexion
   :alt: Python Versions

.. image:: https://img.shields.io/pypi/l/connexion.svg
   :target: https://github.com/zalando/connexion/blob/master/LICENSE
   :alt: License

Connexion is a framework that automagically handles HTTP requests based on `OpenAPI Specification`_
(formerly known as Swagger Spec) of your API described in `YAML format`_. Connexion allows you to
write an OpenAPI specification, then maps the endpoints to your Python functions; this makes it
unique, as many tools generate the specification based on your Python code. You can describe your
REST API in as much detail as you want; then Connexion guarantees that it will work as you
specified.

We built Connexion this way in order to:

- simplify the development process
- confirm expectations about what your API will look like

Connexion Features:
-------------------

- Validates requests and endpoint parameters automatically, based on
  your specification
- Provides a Web Swagger Console UI so that the users of your API can
  have live documentation and even call your API's endpoints
  through it
- Handles OAuth 2 token-based authentication
- Supports API versioning
- Supports automatic serialization of payloads. If your
  specification defines that an endpoint returns JSON, Connexion will
  automatically serialize the return value for you and set the right
  content type in the HTTP header.

Why Connexion
-------------

With Connexion, you write the spec first. Connexion then calls your Python
code, handling the mapping from the specification to the code. This
incentivizes you to write the specification so that all of your
developers can understand what your API does, even before you write a
single line of code.

If multiple teams depend on your APIs, you can use Connexion to easily send them the documentation of your API. This guarantees that your API will follow the specification that you wrote. This is a different process from that offered by frameworks such as Hug_, which generates a specification *after* you've written the code. Some disadvantages of generating specifications based on code is that they often end up lacking details or mix your documentation with the code logic of your application.

Other Sources/Mentions
----------------------

- Zalando Tech blog post `API First`_
- Connexion listed on Swagger_'s website
- Blog post: `Crafting effective Microservices in Python`_

New in Connexion 2.0:
---------------------
- App and Api options must be provided through the "options" argument (``old_style_options`` have been removed).
- You must specify a form content-type in 'consumes' in order to consume form data.
- The `Operation` interface has been formalized in the `AbstractOperation` class.
- The `Operation` class has been renamed to `Swagger2Operation`.
- Array parameter deserialization now follows the Swagger 2.0 spec more closely.
  In situations when a query parameter is passed multiple times, and the collectionFormat is either csv or pipes, the right-most value will be used.
  For example, `?q=1,2,3&q=4,5,6` will result in `q = [4, 5, 6]`.
  The old behavior is available by setting the collectionFormat to `multi`, or by importing `decorators.uri_parsing.AlwaysMultiURIParser` and passing `parser_class=AlwaysMultiURIParser` to your Api.
- The spec validator library has changed from `swagger-spec-validator` to `openapi-spec-validator`.
- Errors that previously raised `SwaggerValidationError` now raise the `InvalidSpecification` exception.
  All spec validation errors should be wrapped with `InvalidSpecification`.
- Support for nullable/x-nullable, readOnly and writeOnly/x-writeOnly has been added to the standard json schema validator.
- Custom validators can now be specified on api level (instead of app level).
- Added support for basic authentication and apikey authentication
- If unsupported security requirements are defined or ``x-tokenInfoFunc``/``x-tokenInfoUrl`` is missing, connexion now denies requests instead of allowing access without security-check.
- Accessing ``connexion.request.user`` / ``flask.request.user`` is no longer supported, use ``connexion.context['user']`` instead

How to Use
==========

Prerequisites
-------------

Python 2.7 or Python 3.5+

Installing It
-------------

In your command line, type:

.. code-block:: bash

    $ pip install connexion

Running It
----------

Place your API YAML inside a folder in the root
path of your application (e.g ``swagger/``). Then run:

.. code-block:: python

    import connexion

    app = connexion.App(__name__, specification_dir='swagger/')
    app.add_api('my_api.yaml')
    app.run(port=8080)

See the `Connexion Pet Store Example Application`_ for a sample
specification.

Now you're able to run and use Connexion!


OAuth 2 Authentication and Authorization
----------------------------------------

Connexion supports one of the three OAuth 2 handling methods. (See
"TODO" below.) With Connexion, the API security definition **must**
include a 'x-tokenInfoUrl' or 'x-tokenInfoFunc (or set ``TOKENINFO_URL``
or ``TOKENINFO_FUNC`` env var respectively). 'x-tokenInfoUrl' must contain an
URL to validate and get the `token information`_ and 'x-tokenInfoFunc must
contain a reference to a function used to obtain the token info. When both 'x-tokenInfoUrl'
and 'x-tokenInfoFunc' are used, Connexion will prioritize the function method. Connexion expects to
receive the OAuth token in the ``Authorization`` header field in the
format described in `RFC 6750 <rfc6750_>`_ section 2.1. This aspect
represents a significant difference from the usual OAuth flow.

Dynamic Rendering of Your Specification
---------------------------------------

Connexion uses Jinja2_ to allow specification parameterization through the ``arguments`` parameter. You can define specification arguments for the application either globally (via the ``connexion.App`` constructor) or for each specific API (via the ``connexion.App#add_api`` method):

.. code-block:: python

    app = connexion.App(__name__, specification_dir='swagger/',
                        arguments={'global': 'global_value'})
    app.add_api('my_api.yaml', arguments={'api_local': 'local_value'})
    app.run(port=8080)

When a value is provided both globally and on the API, the API value will take precedence.

Endpoint Routing to Your Python Views
-------------------------------------

Connexion uses the ``operationId`` from each `Operation Object`_ to
identify which Python function should handle each URL.

**Explicit Routing**:

.. code-block:: yaml

    paths:
      /hello_world:
        post:
          operationId: myapp.api.hello_world

If you provide this path in your specification POST requests to
``http://MYHOST/hello_world``, it will be handled by the function
``hello_world`` in the ``myapp.api`` module. Optionally, you can include
``x-swagger-router-controller`` (or ``x-openapi-router-controller``) in your
operation definition, making ``operationId`` relative:

.. code-block:: yaml

    paths:
      /hello_world:
        post:
          x-swagger-router-controller: myapp.api
          operationId: hello_world

Keep in mind that Connexion follows how `HTTP methods work in Flask`_ and therefore HEAD requests will be handled by the ``operationId`` specified under GET in the specification. If both methods are supported, ``connexion.request.method`` can be used to determine which request was made.

Automatic Routing
-----------------

To customize this behavior, Connexion can use alternative
``Resolvers``--for example, ``RestyResolver``. The ``RestyResolver``
will compose an ``operationId`` based on the path and HTTP method of
the endpoints in your specification:

.. code-block:: python

    from connexion.resolver import RestyResolver

    app = connexion.App(__name__)
    app.add_api('swagger.yaml', resolver=RestyResolver('api'))

.. code-block:: yaml

   paths:
     /:
       get:
          # Implied operationId: api.get
     /foo:
       get:
          # Implied operationId: api.foo.search
       post:
          # Implied operationId: api.foo.post

     '/foo/{id}':
       get:
          # Implied operationId: api.foo.get
       put:
          # Implied operationId: api.foo.put
       copy:
          # Implied operationId: api.foo.copy
       delete:
          # Implied operationId: api.foo.delete

``RestyResolver`` will give precedence to any ``operationId`` encountered in the specification. It will also respect
``x-router-controller``. You can import and extend ``connexion.resolver.Resolver`` to implement your own ``operationId``
(and function) resolution algorithm.

Automatic Parameter Handling
----------------------------

Connexion automatically maps the parameters defined in your endpoint specification to arguments of your Python views as named parameters, and, whenever possible, with value casting. Simply define the endpoint's parameters with the same names as your views arguments.

As an example, say you have an endpoint specified as:

.. code-block:: yaml

    paths:
      /foo:
        get:
          operationId: api.foo_get
          parameters:
            - name: message
              description: Some message.
              in: query
              type: string
              required: true

And the view function:

.. code-block:: python

    # api.py file

    def foo_get(message):
        # do something
        return 'You send the message: {}'.format(message), 200

In this example, Connexion automatically recognizes that your view
function expects an argument named ``message`` and assigns the value
of the endpoint parameter ``message`` to your view function.

.. note:: In the OpenAPI 3.x.x spec, the requestBody does not have a name.
          By default it will be passed in as 'body'. You can optionally
          provide the x-body-name parameter in your requestBody schema
          to override the name of the parameter that will be passed to your
          handler function.

.. warning:: When you define a parameter at your endpoint as *not* required, and
    this argument does not have default value in your Python view, you will get
    a "missing positional argument" exception whenever you call this endpoint
    WITHOUT the parameter. Provide a default value for a named argument or use
    ``**kwargs`` dict.

Type casting
^^^^^^^^^^^^

Whenever possible, Connexion will try to parse your argument values and
do type casting to related Python native values. The current
available type castings are:

+--------------+-------------+
| OpenAPI Type | Python Type |
+==============+=============+
| integer      | int         |
+--------------+-------------+
| string       | str         |
+--------------+-------------+
| number       | float       |
+--------------+-------------+
| boolean      | bool        |
+--------------+-------------+
| array        | list        |
+--------------+-------------+
| null         | None        |
+--------------+-------------+
| object       | dict        |
+--------------+-------------+

If you use the ``array`` type In the Swagger definition, you can define the
``collectionFormat`` so that it won't be recognized. Connexion currently
supports collection formats "pipes" and "csv". The default format is "csv".

Connexion is opinionated about how the URI is parsed for ``array`` types.
The default behavior for query parameters that have been defined multiple
times is to use the right-most value. For example, if you provide a URI with
the the query string ``?letters=a,b,c&letters=d,e,f``, connexion will set
``letters = ['d', 'e', 'f']``.

You can override this behavior by specifying the URI parser in the app or
api options.

.. code-block:: python

   from connexion.decorators.uri_parsing import AlwaysMultiURIParser
   options = {'uri_parser_class': AlwaysMultiURIParser}
   app = connexion.App(__name__, specification_dir='swagger/', options=options)

You can implement your own URI parsing behavior by inheriting from
``connexion.decorators.uri_parsing.AbstractURIParser``.

There are a handful of URI parsers included with connection.

+----------------------+---------------------------------------------------------------------------+
| OpenAPIURIParser     | This parser adheres to the OpenAPI 3.x.x spec, and uses the ``style``     |
| default: OpenAPI 3.0 | parameter. Query parameters are parsed from left to right, so if a query  |
|                      | parameter is defined twice, then the right-most definition will take      |
|                      | precedence. For example, if you provided a URI with the query string      |
|                      | ``?letters=a,b,c&letters=d,e,f``, and ``style: simple``, then connexion   |
|                      | will set ``letters = ['d', 'e', 'f']``. For additional information see    |
|                      | `OpenAPI 3.0 Style Values`_.                                              |
+----------------------+---------------------------------------------------------------------------+
| Swagger2URIParser    | This parser adheres to the Swagger 2.0 spec, and will only join together  |
| default: OpenAPI 2.0 | multiple instance of the same query parameter if the ``collectionFormat`` |
|                      | is set to ``multi``. Query parameters are parsed from left to right, so   |
|                      | if a query parameter is defined twice, then the right-most definition     |
|                      | wins. For example, if you provided a URI with the query string            |
|                      | ``?letters=a,b,c&letters=d,e,f``, and ``collectionFormat: csv``, then     |
|                      | connexion will set ``letters = ['d', 'e', 'f']``                          |
+----------------------+---------------------------------------------------------------------------+
| FirstValueURIParser  | This parser behaves like the Swagger2URIParser, except that it prefers    |
|                      | the first defined value. For example, if you provided a URI with the query|
|                      | string ``?letters=a,b,c&letters=d,e,f`` and ``collectionFormat: csv``     |
|                      | hen connexion will set ``letters = ['a', 'b', 'c']``                      |
+----------------------+---------------------------------------------------------------------------+
| AlwaysMultiURIParser | This parser is backwards compatible with Connexion 1.x. It joins together |
|                      | multiple instances of the same query parameter.                           |
+----------------------+---------------------------------------------------------------------------+


Parameter validation
^^^^^^^^^^^^^^^^^^^^

Connexion can apply strict parameter validation for query and form data
parameters.  When this is enabled, requests that include parameters not defined
in the swagger spec return a 400 error.  You can enable it when adding the API
to your application:

.. code-block:: python

    app.add_api('my_apy.yaml', strict_validation=True)

API Versioning and basePath
---------------------------

Setting a base path is useful for versioned APIs. An example of
a base path would be the ``1.0`` in ``http://MYHOST/1.0/hello_world``.

If you are using OpenAPI 3.x.x, you set your base URL path in the
servers block of the specification. You can either specify a full
URL, or just a relative path.

.. code-block:: yaml

    servers:
      - url: https://MYHOST/1.0
        description: full url example
      - url: /1.0
        description: relative path example

    paths:
      ...

If you are using OpenAPI 2.0, you can define a ``basePath`` on the top level
of your OpenAPI 2.0 specification.

.. code-block:: yaml

    basePath: /1.0

    paths:
      ...

If you don't want to include the base path in your specification, you
can provide it when adding the API to your application:

.. code-block:: python

    app.add_api('my_api.yaml', base_path='/1.0')

Swagger JSON
------------
Connexion makes the OpenAPI/Swagger specification in JSON format
available from either ``swagger.json`` (for OpenAPI 2.0) or
``openapi.json`` (for OpenAPI 3.x.x) at the base path of the API.
For example, if your base path was ``1.0``, then your spec would be
available at ``/1.0/openapi.json``.

You can disable serving the spec JSON at the application level:

.. code-block:: python

    options = {"serve_spec": False}
    app = connexion.App(__name__, specification_dir='openapi/',
                        options=options)
    app.add_api('my_api.yaml')

You can also disable it at the API level:

.. code-block:: python

    options = {"serve_spec": False}
    app = connexion.App(__name__, specification_dir='openapi/')
    app.add_api('my_api.yaml', options=options)

HTTPS Support
-------------

When specifying HTTPS as the scheme in the API YAML file, all the URIs
in the served Swagger UI are HTTPS endpoints. The problem: The default
server that runs is a "normal" HTTP server. This means that the
Swagger UI cannot be used to play with the API. What is the correct
way to start a HTTPS server when using Connexion?

One way, `described by Flask`_, looks like this:

.. code-block:: python

   from OpenSSL import SSL
   context = SSL.Context(SSL.SSLv23_METHOD)
   context.use_privatekey_file('yourserver.key')
   context.use_certificate_file('yourserver.crt')

   app.run(host='127.0.0.1', port='12344',
           debug=False/True, ssl_context=context)

However, Connexion doesn't provide an ssl_context parameter. This is
because Flask doesn't, either--but it uses ``**kwargs`` to send the
parameters to the underlying `werkzeug`_ server.

The Swagger UI Console
----------------------

The Swagger UI for an API is available through pip extras.
You can install it with ``pip install connexion[swagger-ui]``.
It will be served up at ``{base_path}/ui/`` where ``base_path`` is the
base path of the API.

You can disable the Swagger UI at the application level:

.. code-block:: python

    app = connexion.App(__name__, specification_dir='openapi/',
                        options={"swagger_ui": False})
    app.add_api('my_api.yaml')


You can also disable it at the API level:

.. code-block:: python

    app = connexion.App(__name__, specification_dir='openapi/')
    app.add_api('my_api.yaml', options={"swagger_ui": False})

If necessary, you can explicitly specify the path to the directory with
swagger-ui to not use the connexion[swagger-ui] distro.
In order to do this, you should specify the following option:

.. code-block:: python

   options = {'swagger_path': '/path/to/swagger_ui/'}
   app = connexion.App(__name__, specification_dir='openapi/', options=options)

If you wish to provide your own swagger-ui distro, note that connexion
expects a jinja2 file called ``swagger_ui/index.j2`` in order to load the
correct ``swagger.json`` by default. Your ``index.j2`` file can use the
``openapi_spec_url`` jinja variable for this purpose:

.. code-block::

    const ui = SwaggerUIBundle({ url: "{{ openapi_spec_url }}"})

Additionally, if you wish to use swagger-ui-3.x.x, it is also provided by
installing connexion[swagger-ui], and can be enabled like this:

.. code-block:: python

   from swagger_ui_bundle import swagger_ui_3_path
   options = {'swagger_path': swagger_ui_3_path}
   app = connexion.App(__name__, specification_dir='swagger/', options=options)


Server Backend
--------------

By default Connexion uses the Flask_ server. For asynchronous
applications, you can also use Tornado_ as the HTTP server. To do
this, set your server to ``tornado``:

.. code-block:: python

    import connexion

    app = connexion.App(__name__, specification_dir='swagger/')
    app.run(server='tornado', port=8080)

You can use the Flask WSGI app with any WSGI container, e.g. `using
Flask with uWSGI`_ (this is common):

.. code-block:: python

    app = connexion.App(__name__, specification_dir='swagger/')
    application = app.app # expose global WSGI application object

You can use the ``aiohttp`` framework as server backend as well:

.. code-block:: python

    import connexion

    app = connexion.AioHttpApp(__name__, specification_dir='swagger/')
    app.run(port=8080)

.. note:: Also check aiohttp handler examples_.

Set up and run the installation code:

.. code-block:: bash

    $ sudo pip3 install uwsgi
    $ uwsgi --http :8080 -w app -p 16  # use 16 worker processes

See the `uWSGI documentation`_ for more information.

.. _using Flask with uWSGI: http://flask.pocoo.org/docs/latest/deploying/uwsgi/
.. _uWSGI documentation: https://uwsgi-docs.readthedocs.org/
.. _examples: https://docs.aiohttp.org/en/stable/web.html#handler


Documentation
=============
Additional information is available at `Connexion's Documentation Page`_.

Changes
=======

A full changelog is maintained on the `GitHub releases page`_.

.. _GitHub releases page: https://github.com/zalando/connexion/releases

Contributing to Connexion/TODOs
===============================

We welcome your ideas, issues, and pull requests. Just follow the
usual/standard GitHub practices.

Unless you explicitly state otherwise in advance, any non trivial
contribution intentionally submitted for inclusion in this project by you
to the steward of this repository (Zalando SE, Berlin) shall be under the
terms and conditions of Apache License 2.0 written below, without any
additional copyright information, terms or conditions.

TODOs
-----


If you'd like to become a more consistent contributor to Connexion, we'd love your help working on
these we have a list of `issues where we are looking for contributions`_.

Thanks
===================

We'd like to thank all of Connexion's contributors for working on this
project, and to Swagger/OpenAPI for their support.

License
===================

Copyright 2015 Zalando SE

Licensed under the Apache License, Version 2.0 (the "License"); you may not use this file except in compliance with the License. You may obtain a copy of the License at http://www.apache.org/licenses/LICENSE-2.0.

Unless required by applicable law or agreed to in writing, software distributed under the License is distributed on an "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied. See the License for the specific language governing permissions and limitations under the License.

.. _Flask: http://flask.pocoo.org/
.. _issues waffle board: https://waffle.io/zalando/connexion
.. _API First: https://tech.zalando.com/blog/on-apis-and-the-zalando-api-guild/
.. _Hug: https://github.com/timothycrosley/hug
.. _Swagger: http://swagger.io/open-source-integrations/
.. _Jinja2: http://jinja.pocoo.org/
.. _rfc6750: https://tools.ietf.org/html/rfc6750
.. _OpenAPI Specification: https://www.openapis.org/
.. _OpenAPI 3.0 Style Values: https://github.com/OAI/OpenAPI-Specification/blob/master/versions/3.0.2.md#style-values
.. _Operation Object: https://github.com/swagger-api/swagger-spec/blob/master/versions/2.0.md#operation-object
.. _swager.spec.security_definition: https://github.com/swagger-api/swagger-spec/blob/master/versions/2.0.md#security-definitions-object
.. _swager.spec.security_requirement: https://github.com/swagger-api/swagger-spec/blob/master/versions/2.0.md#security-requirement-object
.. _YAML format: https://github.com/OAI/OpenAPI-Specification/blob/master/versions/2.0.md#format
.. _token information: https://tools.ietf.org/html/rfc6749
.. _Tornado: http://www.tornadoweb.org/en/stable/
.. _Connexion Pet Store Example Application: https://github.com/hjacobs/connexion-example
.. _described by Flask: http://flask.pocoo.org/snippets/111/
.. _werkzeug: http://werkzeug.pocoo.org/
.. _Connexion's Documentation Page: http://connexion.readthedocs.org/en/latest/
.. _Crafting effective Microservices in Python: https://jobs.zalando.com/tech/blog/crafting-effective-microservices-in-python/
.. _issues where we are looking for contributions: https://github.com/zalando/connexion/issues?q=is%3Aissue+is%3Aopen+label%3A%22help+wanted%22
.. _HTTP Methods work in Flask: http://flask.pocoo.org/docs/1.0/quickstart/#http-methods
