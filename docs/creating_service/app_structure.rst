.. _creating_app:

Service App Code
================

In this section we exaplin how to write you code to deploy it as a microservice - we have endeavoured to make this part as simple as possible, so you can get on with the business of writing your business logic normally and not worry about infrastructure, servers, or hosting.


.. _creating_app_idl:

Interface Definition (``api.idl``)
----------------------------------

**TODO**

See :ref:`tutorial_create`.

This is based on the `Barrister RPC project <http://barrister.bitmechanic.com/>`_, the format of which is described in the `project documentation <http://barrister.bitmechanic.com/docs.html>`_.



App Code
--------

**TODO**




.. _creating_app_auth:

Authorisation
^^^^^^^^^^^^^

By default if an ``auth`` object exists in the request (see :ref:`using_general_auth`) it's checked by the API layer and only routed to the service if it's valid for **any** StackHut user, regardless if the service is public or private. Authentication is handled at the API layer.

A service must determine if the authenticated request should be allowed or not, and the StackHut runtime (see :ref:`creating_runtime`) has several functions to aid this authorisation,

 * ``stackhut.get_stackhut_user()`` returns the username user to make the request,
 * ``stackhut.is_author()`` returns a boolean indicating if the author's Stackhut account was used to make the request. This can help to restrict a private service's interfaces to the service author only and any associated API tokens.

We have separated authentication from authorisation is to aid deploying internal StackHut clusters where there will be many users owning services, in this case we believe that individual service authorisation is better handled within the service itself according to internal policies. 

.. note:: However we are working on simplying this scheme of users on the hosted platform

.. _creating_app_lang:

Language Specific Notes
-----------------------


Python 3.x
^^^^^^^^^^

As mentioned above all calls to the server are blocking (although handled in multiple threads).

There are two methods to deal with this on the Python side.
Firstly Python has a good threading library that works very well when used with blocking IO calls. 
Secondly Python 3.5 will introduce ``async`` and ``await`` type-functionality as seen in C# that can be used to interleave these calls.


Node.js / ES6
^^^^^^^^^^^^^

The Node.js story is more complex as StackHut is primarily a request->response system exhibited through functions as entrypoints. This conflicts with the callback-based model of Node at present.

However things are looking much better with both ES6 and ES7 on the horizon.
StackHut's Node support is based on the latest io.js with support for ES6, and promises in particular (`here's <http://www.html5rocks.com/en/tutorials/es6/promises/>`_ is a good intro).

.. note:: We currently utilise `io.js v3 <https://iojs.org/>`_ to provide a compatible version of Node.js with ES6 features. These projects have now remerged and we will follow io.js in moving to the Node.js project accordingly.

The StackHut runtime is promise-based on Node, with each call returning a promise than resolves on completion.

Similarly the main entrypoints to a service are also promise-based. The StackHut runner expects each entrypoint to return a promise that resolves on completion of the service request.
This gives you two choices when implementing a service function depending on if it's a synchronous or a callback-based asynchronous service.


Synchronous Services
""""""""""""""""""""

Write your function as normal and simply wrap the result in a Promise.resolve().

.. code-block:: js

    add(x, y) {
        let res = x + y;
        return Promise.resolve(res);
    }


Asynchronous Services
"""""""""""""""""""""

Wrap your service block in a single promise that is returned to the system. Within this block write your normal code and call with ``resolve`` or ``reject`` as required on completion. This method interacts nicely with new promise-based and legacy callback-based async code.

.. code-block:: js

    asyncAdd(x, y) {
        return new Promise(function(resolve, reject) {
            someAsyncCall(x, y)
            .then(function (res) {
                resolve(res);
            
            })   
        })
    }


.. note:: As we support regular ES6 with node packages, feel free to add any helpers libraries to your ``package.json`` to ease writing async services, i.e. `co <https://github.com/tj/co>`_.

Similar to Python 3.5, ``async`` and ``await`` are coming with ES7 and will provide a better model for async code that will be easier to integrate with StackHut.


