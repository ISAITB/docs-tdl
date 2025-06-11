.. index:: Service handlers
.. index:: Step handlers
.. _handlers:

Test step handlers
==================

The architectural approach followed by the GITB TDL is to capture in the test case the high level testing flow 
and delegate detailed domain-specific processing to separate services. These services can cover messaging
between actors, complex processing or content validation and implement APIs that are defined in the GITB
specification. The components implementing these services are termed generally **handlers** and, depending on
their purpose can be:

* :ref:`introduction-concepts-messaging-handlers` implementing the `GITB messaging service API`_.
* :ref:`introduction-concepts-processing-handlers` implementing the `GITB processing service API`_.
* :ref:`introduction-concepts-validation-handlers` implementing the `GITB validation service API`_.

.. _GITB messaging service API: https://www.itb.ec.europa.eu/specs/latest/gitb_ms.wsdl
.. _GITB processing service API: https://www.itb.ec.europa.eu/specs/latest/gitb_ps.wsdl
.. _GITB validation service API: https://www.itb.ec.europa.eu/specs/latest/gitb_vs.wsdl

Another important distinction for handlers is whether they are **built-in** within the Test Bed software or **external**.
For handlers that relate to domain-specific operations, the norm is to externalise them as remotely callable services.
Nonetheless several common tasks that are frequently encountered in test cases are also available as built-in test engine capabilities.

In the sections that follow you can find:

* The supported :ref:`built-in handlers<handlers-predefined-handlers>` covering common tasks encountered in test cases.
* The list of :ref:`reusable external services <handlers-reusable-handlers>` maintained by the Test Bed team (also usable locally as components).
* Guidelines to implement :ref:`custom external services <handlers-custom-handlers>` to cover project-specific needs.

.. _handlers-implementation:

Specifying the handler implementation
-------------------------------------

Handlers are defined in the following steps:
 
* :ref:`tdl-step-btxn`: When beginning a messaging transaction.
* :ref:`tdl-step-send`: When sending a message outside of a messaging transaction.
* :ref:`tdl-step-receive`: When receiving a message outside of a messaging transaction.
* :ref:`tdl-step-listen`: When proxying a call to another service outside of a messaging transaction.
* :ref:`tdl-step-bptxn`: When beginning a processing transaction.
* :ref:`tdl-step-process`: When making a processing operation outside of a processing transaction.
* :ref:`tdl-step-verify`: When validating content.

The element corresponding to each of these steps defines a ``handler`` attribute to identify the handler implementation.
In case an built-in handler is to be used the value specified here is the name of the handler (see :ref:`handlers-predefined-handlers`). Using an external
handler implementation is achieved by specifying as the ``handler`` value the address where the service's WSDL file is 
located. The Test Bed will automatically detect in this case that the handler is external and will internally replace local method
invocations with web service calls.

The value provided for the ``handler`` attribute can also be provided with a pure variable reference (see :ref:`test-case-referring-to-variables`)
allowing the actual value to be determined from configuration or even dynamically based on the test session context. In such a case the variable
reference is first evaluated to a ``string`` that is then considered to determine whether the handler is a remote or built-in one.

The following example shows three validation steps taking place, the first one using the built-in :ref:`handlers-XmlValidator`, the second one using 
an external validation service, and the third one using an external validation service whose address is configurable:

.. code-block:: xml

    <!-- 
        Call a local, built-in validation handler called "XmlValidator"
    -->
    <verify handler="XmlValidator" desc="Validate content local">
        <input name="xml">$docToValidate</input>
        <input name="xsd">$schemaFile</input>
    </verify>
    <!-- 
        Call a remote validation service handler
    -->
    <verify handler="https://serviceaddress?wsdl" desc="Validate content remote">
        <input name="xml">$docToValidate</input>
        <input name="schema">$schemaFile</input>
    </verify>
    <!-- 
        Call a remote validation service handler (address in configuration)
    -->
    <verify handler="$DOMAIN{validationHandlerAddress}" desc="Validate content remote">
        <input name="xml">$docToValidate</input>
        <input name="schema">$schemaFile</input>
    </verify>

.. index:: Built-in handlers
.. _handlers-predefined-handlers:

Built-in handlers
-----------------

The sections that follow list the handler implementations that already exist as predefined built-in implementations
in the GITB Test Bed software.

.. index:: Built-in messaging handlers
.. _handlers-predefined-handlers-messaging:

Built-in messaging handlers
~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. note::

    **Custom vs built-in messaging handlers:** Built-in messaging handlers offer simple, out-of-the-box support for sending and receiving content
    to and from external systems. However, for complex messaging or testing needs you may be better suited by a :ref:`custom implementation<handlers-custom-handlers>`
    that would allow you to:

    * Use **non-HTTP** based protocols.
    * Manage **security aspects** including authentication and use of client certificates.
    * Produce **customised step reports** rather than a listing of request and response data.
    * Manage **large payloads** not supported directly by the test engine.
    * For :ref:`receive <tdl-step-receive>` steps, control fully **exposed APIs** and dynamically **adapt responses** based on received request data.

    To start developing a custom messaging service you can use the `published template <https://www.itb.ec.europa.eu/docs/services/latest/templates/index.html>`_ 
    and follow the `step-by-step development guide <https://www.itb.ec.europa.eu/docs/guides/latest/developingComplexTests/index.html>`_.

The title of each section corresponds to the name of the handler that needs to be configured in the relevant step's ``handler`` attribute (in 
:ref:`send <tdl-step-send>`, :ref:`receive <tdl-step-receive>` or :ref:`btxn <tdl-step-btxn>` steps).

.. index:: HttpMessagingV2

.. _handlers-httpmessagingv2:

HttpMessagingV2
+++++++++++++++

Used to send or receive content over HTTP.

.. index:: HttpMessagingV2 (send)
.. index:: HttpMessagingV2 (send - uri)
.. index:: HttpMessagingV2 (send - method)
.. index:: HttpMessagingV2 (send - headers)
.. index:: HttpMessagingV2 (send - body)
.. index:: HttpMessagingV2 (send - parameters)
.. index:: HttpMessagingV2 (send - queryParameters)
.. index:: HttpMessagingV2 (send - parts)
.. index:: HttpMessagingV2 (send - followRedirects)
.. index:: HttpMessagingV2 (send - status)
.. index:: HttpMessagingV2 (send - requestTimeout)
.. index:: HttpMessagingV2 (send - connectionTimeout)

.. _handlers-httpmessagingv2-send:

Use in send steps
^^^^^^^^^^^^^^^^^

To use this handler to **send a HTTP message** to a external system, set it as the ``handler`` of a :ref:`send <tdl-step-send>` step 
(or the step's :ref:`transaction <tdl-step-btxn>`). In this case the supported step **inputs** are as follows:

.. csv-table::
    :header: "Input name", "Required?", "Type", "Description"
    :delim: |

    ``connectionTimeout`` | No | ``number`` | The number of milliseconds to wait for until a connection is established (by default 10000 - 10 seconds).
    ``body`` | No | ``binary`` | The HTTP body of the request.
    ``followRedirects`` | No | ``boolean`` | Whether redirect responses should be followed (default is "true").
    ``headers`` | No | ``map`` | The HTTP headers to include, provided as a map of either strings (for single values) or lists of strings (for multiple values).
    ``method`` | No | ``string`` | The HTTP method to use. Supported methods are GET (the default), POST, PUT, DELETE, HEAD, OPTIONS, PATCH and TRACE.
    ``queryParameters`` | No | ``map`` | The HTTP request parameters to include in the query string, provided as a map of strings (the map keys will be the parameter names).
    ``parameters`` | No | ``map`` | The HTTP request parameters to include, provided as a map of strings (the map keys will be the parameter names).
    ``parts`` | No | ``list`` | The parts to include for a multipart request, provided either as a list of maps or a single map. Each map corresponds to a part and includes keys *"name"* (required), *"content"* (required), *"fileName"* and *"contentType"*.
    ``requestTimeout`` | No | ``number`` | The number of milliseconds to wait for after a connection is established to read back the complete response (by default no timeout is applied).
    ``uri`` | Yes | ``string`` | The URI of the endpoint to be called.
    
In terms of implicit default values for missing inputs and the processing logic applied, the test engine bases itself on the overall inputs provided. Specifically:

* If no ``method`` is provided but a ``body`` or ``parts`` input is present, the request will be a POST.
* If ``parameters`` is used on a POST, PUT or PATCH these will be added as form-encoded parameters in the body, adding also the correct content type
  ("application/x-www-form-urlencoded") as a HTTP header. If a ``body`` is also provided then the parameters will be merged with any ``queryParameters``
  and added on the query string.
* If ``parameters`` is used on a GET, DELETE, HEAD, OPTIONS or TRACE they will be merged with any ``queryParameters`` and added on the query string.
* If ``parts`` are provided for a multipart request, these are added as parts named using the relevant map's "name" value, with content set to the map's
  "content" value, and a content type set to the "contentType" value (defaulting to "application/octet-stream" if missing). If a "fileName" value is also
  provided, the part is added as a file part (as opposed to a text part if missing).
  
Once the :ref:`send <tdl-step-send>` step completes, the resulting report will include two nested maps named ``request`` and ``response``, corresponding 
respectively to the submitted request and received response. The ``request`` map includes the following:

.. csv-table::
    :header: "Output name", "Always present?", "Type", "Description"
    :delim: |

    ``body`` | No | ``binary`` or ``map`` | The HTTP body of the request that will be a map in case of a multipart request, with one entry per part.
    ``headers`` | No | ``map`` | The HTTP headers explicitly added to the request, provided as a map of key (header name) to comma-separated string values (header values).
    ``method`` | Yes | ``string`` | The HTTP method used.
    ``uri`` | Yes | ``string`` | The URI of the endpoint that was called.

The ``response`` map includes the following:

.. csv-table::
    :header: "Output name", "Always present?", "Type", "Description"
    :delim: |

    ``body`` | No | ``binary`` or ``map`` | The HTTP body of the response that will be a map in case of a multipart request, with one entry per part.
    ``headers`` | No | ``map`` | The HTTP headers of the response, provided as a map of key (header name) to comma-separated string values (header values).
    ``status`` | Yes | ``number`` | The HTTP status code of the response.

The following GITB TDL snippets provide various examples using the ``HttpMessagingV2`` handler in :ref:`send <tdl-step-send>` steps, as well as accessing
outputs (see inline comments for details per case).

.. code-block:: xml

    <!--
        Make a GET call to https://my.sut.org/api/get.
    -->
    <send id="send1" desc="Call system" from="Actor1" to="Actor2" handler="HttpMessagingV2">
        <input name="uri">"https://my.sut.org/api/get"</input>
    </send>
    <log>$send1{response}{status}</log> <!-- Print returned status. -->

    <!--
        Make a POST call to https://my.sut.org/api/post with form-encoded parameters in the request body.
    -->
    <assign to="send2Params{key1}">"value1"</assign>
    <assign to="send2Params{key2}">"value 2"</assign> <!-- URL escaping is automatically handled. -->
    <send id="send2" desc="Call system with parameters" from="Actor1" to="Actor2" handler="HttpMessagingV2">
        <input name="uri">"https://my.sut.org/api/post"</input>
        <input name="method">"POST"</input>
        <input name="parameters">$send2Params</input>
    </send>
    <log>$send2{response}{body}</log> <!-- Print returned body. -->

    <!--
        Make a PUT call to https://my.sut.org/api/put with a body, headers and query string parameters.
    -->
    <assign to="send3Params{key1}">"value1"</assign>
    <assign to="send3Params{key2}">"value2"</assign>
    <assign to="send3Headers{Content-Type}">"application/xml"</assign>
    <send id="send3" desc="Call system" from="Actor1" to="Actor2" handler="HttpMessagingV2">
        <input name="uri">"https://my.sut.org/api/put"</input>
        <input name="method">"PUT"</input>
        <input name="body">$send3Body</input>
        <input name="headers">$send3Headers</input>
        <input name="parameters">$send3Params</input>
    </send>
    <log>$send3{request}{body}</log> <!-- Print request body (added by the test session). -->
    <log>$send3{response}{body}</log> <!-- Print response body (returned by the SUT). -->
    <log>$send3{response}{headers}{Content-Type}</log> <!-- Print response content type. -->

    <!--
        Make a DELETE call with a query string parameter.
    -->
    <assign to="send4Params{id}">"123"</assign>
    <send id="send4" desc="Call system" from="Actor1" to="Actor2" handler="HttpMessagingV2">
        <input name="uri">"https://my.sut.org/api/delete"</input>
        <input name="method">"DELETE"</input>
        <input name="parameters">$send4Params</input>
    </send>
    <log>$send4{response}{status}</log> <!-- Print returned status. -->

    <!--
        Make a DELETE call with an identifier in the URI as a path variable.
    -->
    <assign to="send5Identifier">"123"</assign>
    <send id="send5" desc="Call system" from="Actor1" to="Actor2" handler="HttpMessagingV2">
        <input name="uri">"https://my.sut.org/api/delete/" || $send5Identifier || "/"</input>
        <input name="method">"DELETE"</input>
    </send>
    <log>$send5{response}{status}</log> <!-- Print returned status. -->

    <!-- 
        Make a multipart POST submission to a variable URI with extra query string parameters.
    -->
    <assign to="send6Part1{name}">"file1"</assign> <!-- A file part. -->
    <assign to="send6Part1{contentType}">"text/xml"</assign>
    <assign to="send6Part1{fileName}">"file1.xml"</assign>
    <assign to="send6Part1{content}">$send6Part1Content</assign>

    <assign to="send6Part2{name}" type="string">"text1"</assign> <!-- A text part. -->
    <assign to="send6Part2{contentType}" type="string">"text/plain"</assign>
    <assign to="send6Part2{content}" type="string">"A simple text value"</assign>

    <assign to="send6Parts" append="true">$send6Part1</assign> <!-- Put parts together. -->
    <assign to="send6Parts" append="true">$send6Part2</assign>
    
    <assign to="send6Identifier">"123"</assign> <!-- Path variable. -->
    <assign to="send6Params{key}">"value"</assign> <!-- Query string parameter. -->

    <send id="send6" desc="Call system" from="Actor1" to="Actor2" handler="HttpMessagingV2">
        <input name="uri">"https://my.sut.org/api/multi/" || $send6Identifier || "/"</input>
        <input name="method">"POST"</input>
        <input name="parts">$send6Parts</input>
        <input name="parameters">$send6Params</input>
    </send>
    <log>$send6{response}{status}</log> <!-- Print returned status. -->

    <!--
        Make a GET call to https://my.sut.org/api/get and fail if the response has not been completely received within 1 second.
    -->
    <send id="send7" desc="Call time-sensitive system" from="Actor1" to="Actor2" handler="HttpMessagingV2">
        <input name="uri">"https://my.sut.org/api/get"</input>
        <input name="requestTimeout">1000</input>
    </send>
    <log>$send7{response}{status}</log> <!-- Print returned status. -->

.. index:: HttpMessagingV2 (receive)
.. index:: HttpMessagingV2 (receive - uri)
.. index:: HttpMessagingV2 (receive - uriExtension)
.. index:: HttpMessagingV2 (receive - method)
.. index:: HttpMessagingV2 (receive - headers)
.. index:: HttpMessagingV2 (receive - body)
.. index:: HttpMessagingV2 (receive - status)

.. _handlers-httpmessagingv2-receive:

Use in receive steps
^^^^^^^^^^^^^^^^^^^^

To use this handler to **receive a HTTP message** from an external system, set it as the ``handler`` of a :ref:`receive <tdl-step-receive>` step 
(or the step's :ref:`transaction <tdl-step-btxn>`). In this case the supported step **inputs** are as follows:

.. csv-table::
    :header: "Input name", "Required?", "Type", "Description"
    :delim: |

    ``body`` | No | ``binary`` | The HTTP body of the response.
    ``headers`` | No | ``map`` | The HTTP headers to include in the response.
    ``method`` | No | ``string`` | The HTTP method to expect. Supported methods are GET, POST, PUT, DELETE, HEAD, OPTIONS, PATCH, TRACE.
    ``status`` | No | ``number`` | The HTTP status to respond with (default is 200).
    ``uriExtension`` | No | ``string`` | A URI extension following the base endpoint path at which the request will be expected.

When the :ref:`receive <tdl-step-receive>` step executes, the test session will be suspended until a matching request is received from the system under test.
The listening endpoint where this request will be received is constructed by concatenating in sequence:

1. The test engine's **base messaging URL**.
2. The unique **API key** of the system under test.
3. If defined, the **URI extension** provided as the step's ``uriExtension`` input.

As an example, for a Test Bed installation running locally with default settings, a system with API key "MY_API_KEY", and a provided URI extension of "/test";
the listen endpoint will be:

``http://localhost:8080/itbsrv/api/http/MY_API_KEY/test``

Furthermore, if a ``method`` has been provided as an input, a received request will need to have the **same method** to complete the step. If not provided,
any received request will complete the step. Finally, received query string parameters are ignored when matching incoming requests, except if a query string
has been included in the ``uriExtension`` input. In any case, once the :ref:`receive <tdl-step-receive>` step executes, the listen endpoint and expected
method(s) will be added in the test session log.

.. code-block:: none

    ...
    [2024-06-14 16:31:35] INFO  - Waiting to receive POST at [http://localhost:8080/itbsrv/api/http/7039EC8EX4786X4103XB40FXC7242D11E9B0/search] for step [Receive message 1]

.. note::

    **Supporting parallel test sessions:** The listen endpoint for a :ref:`receive <tdl-step-receive>` step determines also whether test sessions can execute in parallel. If no ``uriExtension`` is
    provided, all test sessions for the system (at least the ones with :ref:`receive <tdl-step-receive>` steps) will need to execute sequentially. You can
    :ref:`configure the relevant test cases <test-case>` with ``supportsParallelExecution`` as "false" to enforce this.

    If possible, try to have ``uriExtension`` differ across test cases, and ideally vary per test session.
    
Once the :ref:`receive <tdl-step-receive>` step completes, the resulting report will include two nested maps named ``request`` and ``response``, corresponding 
respectively to the received request and provided response. The ``request`` map includes the following:

.. csv-table::
    :header: "Output name", "Always present?", "Type", "Description"
    :delim: |

    ``body`` | No | ``binary`` or ``map`` | The HTTP body of the request that will be a map in case of a multipart request, with one entry per part.
    ``headers`` | No | ``map`` | The HTTP request headers, provided as a map of key (header name) to comma-separated string values (header values).
    ``method`` | Yes | ``string`` | The HTTP method used.
    ``uri`` | Yes | ``string`` | The URI of the endpoint that was called.

The ``response`` map includes the following:

.. csv-table::
    :header: "Output name", "Always present?", "Type", "Description"
    :delim: |

    ``body`` | No | ``binary`` or ``map`` | The HTTP body of the response that will be a map in case of a multipart request, with one entry per part.
    ``headers`` | No | ``map`` | The HTTP headers of the response, provided as a map of key (header name) to comma-separated string values (header values).
    ``status`` | Yes | ``string`` | The HTTP status code of the response.

The following GITB TDL snippets provide various examples using the ``HttpMessagingV2`` handler in :ref:`receive <tdl-step-receive>` steps, as well as accessing
outputs (see inline comments for details per case).

.. code-block:: xml

    <!--
        Receive a GET call and respond with a 200 status.
        The step will complete for a received GET to http://localhost:8080/itbsrv/api/http/MY_API_KEY/.
    -->
    <receive id="receive1" desc="Receive call" from="Actor1" to="Actor2" handler="HttpMessagingV2">
        <input name="method">"GET"</input>
        <input name="status">"200"</input>
    </receive>
    <log>$receive1{request}{uri}</log> <!-- Print the full URI of the request. -->

    <!--
        Receive a POST call at a dynamically generated URI extension and respond with a 200 status and JSON body.
        The step will complete for a received POST to http://localhost:8080/itbsrv/api/http/MY_API_KEY/update/123/all.
    -->
    <assign to="receive2Identifier">"123"<assign>
    <assign to="receive2Headers{Content-Type}">"application/json"<assign>
    <receive id="receive2" desc="Receive call" from="Actor1" to="Actor2" handler="HttpMessagingV2">
        <input name="method">"POST"</input>
        <input name="uriExtension">"/update/" || $receive1Identifier || "/all"</input>
        <input name="status">"200"</input>
        <input name="headers">$receive1Headers</input>
        <input name="body">$receive2Body</input>
    </receive>
    <log>$receive2{request}{body}</log> <!-- Print the request body. -->
    <log>$receive2{request}{headers}{Content-Type}</log> <!-- Print the request content type header. -->
    <log>$receive2{response}{body}</log> <!-- Print the response body. -->

    <!--
        Receive a multipart PUT call and respond with a 500 status and text error message.
        The step will complete for a received PUT to http://localhost:8080/itbsrv/api/http/MY_API_KEY/multi.
    -->
    <assign to="receive3Headers{Content-Type}">"text/plain"<assign>
    <receive id="receive3" desc="Receive call" from="Actor1" to="Actor2" handler="HttpMessagingV2">
        <input name="method">"PUT"</input>
        <input name="uriExtension">"/multi"</input>
        <input name="status">"500"</input>
        <input name="headers">$receive3Headers</input>
        <input name="body">"Unsupported operation"</input>
    </receive>
    <log>$receive3{request}{uri}</log> <!-- Print the request URI. -->
    <log>$receive3{request}{body}</log> <!-- Print the request body. -->

.. index:: HttpsMessaging
.. index:: http_headers (HttpsMessaging)
.. index:: http_body (HttpsMessaging)
.. index:: http_method (HttpsMessaging)
.. index:: http_version (HttpsMessaging)
.. index:: http_path (HttpsMessaging)
.. index:: http_status (HttpsMessaging)
.. index:: network.host (HttpsMessaging)
.. index:: network.port (HttpsMessaging)
.. index:: http.uri (HttpsMessaging)
.. index:: http.uri.extension (HttpsMessaging)
.. index:: http.ssl (HttpsMessaging)
.. index:: status.code (HttpsMessaging)
.. index:: http.method (HttpsMessaging)

.. index:: SimulatedMessaging
.. index:: parameters (SimulatedMessaging)
.. index:: contentTypes (SimulatedMessaging)
.. index:: result (SimulatedMessaging)
.. index:: delay (SimulatedMessaging)

.. _handlers-simulatedmessaging:

SimulatedMessaging
++++++++++++++++++

Used to add simulated messaging steps to the test execution diagram without any actual message exchanges taking
place.

.. csv-table::
    :header: "Input name", "Input type", "Required?", "Type", "Description"
    :delim: |

    ``contentTypes``| Send/receive input| No| ``map``| An optional ``map`` including the content types (e.g. ``application/json``) to consider when displaying different parameters.
    ``delay``| Receive input| No| ``number``| An optional number of milliseconds to delay before presenting the :ref:`receive step<tdl-step-receive>` as completed.
    ``parameters``| Send/receive input| No| ``map``| An optional ``map`` of data to display in the step report.
    ``result``| Send/receive input| No| ``string``| Set to ``SUCCESS``, ``WARNING`` or ``FAILURE`` to specify the step's result (default is ``SUCCESS``).

The following example illustrates usage of the ``SimulatedMessaging`` handler to present a simulated exchange between actors, each
with its own report:

.. code-block:: xml

    <assign to="map1{valueFile}">$templateFile</assign>
    <assign to="map1{valueText}">'A text'</assign>
    <send id="dataSend" desc="Send data" from="Actor1" to="Actor2" handler="SimulatedMessaging">
        <input name="parameters">$map1</input>
    </send>
    <assign to="map2{valueFile}">$templateFile</assign>
    <assign to="map2{valueText}">'Another text'</assign>
    <receive id="dataReceive" desc="Receive data" from="Actor2" to="Actor1" reply="true" handler="SimulatedMessaging">
        <input name="parameters">$map2</input>
        <input name="result">'FAILURE'</input>
        <input name="delay">3000</input>
    </receive>

In case the ``SimulatedMessaging`` handler displays reports with **large content** or **complete files**, we can also provide a hint to the test engine
on how such data is to be displayed. This is done by means of the ``contentTypes`` input, an optional ``map`` that can be set with 
the content types (e.g. ``application/json``) to consider per parameter. When a content type is set for a given parameter this will
affect its syntax highlighting when displaying it within editors and also the type of file generated when it is downloaded.

The approach used to specify content types is to match fully, in terms of parameter names and structure, the corresponding
``parameters`` map. Matching of specific parameters is done on the basis of their map key and nesting level, whereas the 
content type values are of type ``string``.

To clarify this, consider the following example where a ``SimulatedMessaging`` handler is used for a :ref:`send<tdl-step-send>` step,
displaying a report with two files (named ``input`` and ``output``). Notice how the ``contentTypes`` input is defined in a manner identical
to the actual data to be displayed.

.. code-block:: xml

    <!-- Define the data. -->
    <assign to="params{input}">$file1</assign>
    <assign to="params{output}">$file2</assign>
    <!-- Define the content types. -->
    <assign to="contentTypes{input}">'application/json'</assign>
    <assign to="contentTypes{output}">'application/xml'</assign>
    <send desc="Send message" from="Actor1" to="Actor2" handler="SimulatedMessaging">
        <input name="parameters">$params</input>
        <input name="contentTypes">$contentTypes</input>
    </send>

Content types don't need to cover all parameters, only those for which they are relevant or known. For example in the following case
we only define a content type for the first displayed file, omitting it for simple strings and for the second file for which the 
content type is unknown.

.. code-block:: xml

    <!-- Define the data. -->
    <assign to="params{aFile}">$file</assign>
    <assign to="params{countryCode}">$countryCode</assign>
    <assign to="params{message}">"Transformation was successful."</assign>
    <assign to="params{aSecondFile}">$secondFile</assign>
    <!-- Define the content type only for the first file. -->
    <assign to="contentTypes{aFile}">'application/xml'</assign>
    <send desc="Send message" from="Actor1" to="Actor2" handler="SimulatedMessaging">
        <input name="parameters">$params</input>
        <input name="contentTypes">$contentTypes</input>
    </send>

Finally, the following example illustrates how content types can be provided when the parameters are defined within complex
structures (maps and lists, nested at different levels).

.. code-block:: xml

    <!-- Define the data. -->
    <assign to="params{input}{file1}">$file1</assign>
    <assign to="params{input}{file2}">$file2</assign>
    <assign to="params{input}{messageId}">$messageIdentifier</assign>
    <assign to="params{input}{attachments}" append="true">$attachment1</assign>
    <assign to="params{input}{attachments}" append="true">$attachment2</assign>
    <assign to="params{output}{response}">$response</assign>
    <assign to="params{output}{message}">"Input processed successfully."</assign>
    <!-- Define the content types. -->
    <assign to="types{input}{file1}">"application/xml"</assign>
    <assign to="types{input}{file2}">"application/xml"</assign>
    <assign to="types{input}{attachments}" append="true">"text/plain"</assign>
    <assign to="types{input}{attachments}" append="true">"application/pdf"</assign>
    <assign to="types{output}{response}">"application/json"</assign>
    <!-- Call the send step. -->
    <send desc="Send message" from="Actor1" to="Actor2" handler="SimulatedMessaging">
        <input name="parameters">$params</input>
        <input name="contentTypes">$types</input>
    </send>

.. index:: SoapMessagingV2

.. _handlers-soapmessagingv2:

SoapMessagingV2
+++++++++++++++

Used to send or receive payloads via SOAP web service calls.

.. index:: SoapMessagingV2 (send)
.. index:: SoapMessagingV2 (send - uri)
.. index:: SoapMessagingV2 (send - envelope)
.. index:: SoapMessagingV2 (send - version)
.. index:: SoapMessagingV2 (send - headers)
.. index:: SoapMessagingV2 (send - action)
.. index:: SoapMessagingV2 (send - attachments)
.. index:: SoapMessagingV2 (send - tolerateNonSoapResponse)

.. _handlers-soapmessagingv2-send:

Use in send steps
^^^^^^^^^^^^^^^^^

To use this handler to **call a SOAP web service** of an external system, set it as the ``handler`` of a :ref:`send <tdl-step-send>` step 
(or the step's :ref:`transaction <tdl-step-btxn>`). In this case the supported step **inputs** are as follows:

.. csv-table::
    :header: "Input name", "Required?", "Type", "Description"
    :delim: |

    ``action`` | No | ``string`` | The SOAPAction header to set. This overrides the value set in the HTTP headers (if present).
    ``attachments`` | No | ``list`` | The MTOM attachments to set, provided as a list of maps or a single map. Each map corresponds to an attachment and includes keys *"name"* (required), *"content"* (required), *"contentType"* and *"forceText"* (a boolean to force the attachment's inclusion as text).
    ``envelope`` | Yes | ``object`` | The SOAP envelope to send (as an XML object).
    ``headers`` | No | ``map`` | The HTTP headers to include in the request, provided as a map of key (header name) to comma-separated string values (header values). Note that the Content-Type header is automatically set based on the SOAP version used.
    ``tolerateNonSoapResponse`` | No | ``boolean`` | Whether a non-SOAP response will be tolerated (otherwise the step results in a failure).
    ``uri`` | Yes | ``string`` | The URI of the SOAP endpoint to be called.
    ``version`` | No | ``string`` | The SOAP protocol version (either "1.1" or "1.2"). If not provided this is calculated from the provided Content-Type HTTP header, otherwise "1.1" is used.

Once the :ref:`send <tdl-step-send>` step completes, the resulting report will include two nested maps named ``request`` and ``response``, corresponding 
respectively to the submitted request and received response. The ``request`` map includes the following:

.. csv-table::
    :header: "Output name", "Always present?", "Type", "Description"
    :delim: |

    ``attachments`` | No | ``map`` | The MTOM attachments sent with the envelope. The key of each entry is the attachment name, whereas the value is the content.
    ``body`` | Yes | ``object`` | The SOAP body extracted from the envelope.
    ``envelope`` | Yes | ``object`` | The SOAP envelope that was sent.
    ``headers`` | Yes | ``map`` | The HTTP headers added to the request, provided as a map of key (header name) to comma-separated string values (header values).
    ``uri`` | Yes | ``string`` | The URI of the endpoint that was called.

The ``response`` map includes the following:

.. csv-table::
    :header: "Output name", "Always present?", "Type", "Description"
    :delim: |

    ``attachments`` | No | ``map`` | The MTOM attachments sent with the envelope. The key of each entry is the attachment name, whereas the value is the content.
    ``body`` | No | ``object`` | The SOAP body extracted from the response envelope.
    ``envelope`` | No | ``object`` | The SOAP envelope of the response.
    ``error`` | No | ``binary`` | In case the response could not be parsed as a SOAP envelope this property includes the HTTP response's payload.
    ``headers`` | No | ``map`` | The HTTP headers from the response, provided as a map of key (header name) to comma-separated string values (header values).
    ``status`` | Yes | ``number`` | The HTTP status code of the response.

The following GITB TDL snippets provide various examples using the ``SoapMessagingV2`` handler in :ref:`send <tdl-step-send>` steps, as well as accessing
outputs (see inline comments for details per case).

.. code-block:: xml

    <!--
        Make a SOAP 1.1 call to https://my.sut.org/api/service.
    -->
    <send id="send1" desc="Send SOAP message" from="Actor1" to="Actor2" handler="SoapMessagingV2">
        <input name="uri">"https://my.sut.org/api/service"</input>
        <input name="envelope">$send1Envelope</input>
    </send>
    <log>$send1{request}{envelope}</log> <!-- Print the SOAP envelope that was sent. -->
    <log>$send1{response}{body}</log> <!-- Print the response's SOAP body. -->
    <log>$send1{response}{headers}{Content-Type}</log> <!-- Print the response HTTP Content-Type header. -->

    <!-- 
        Make a SOAP 1.2 call to https://my.sut.org/api/service2 setting the SOAP action.
    -->
    <send id="send2" desc="Send SOAP message" from="Actor1" to="Actor2" handler="SoapMessagingV2">
        <input name="uri">"https://my.sut.org/api/service2"</input>
        <input name="envelope">$send2Envelope</input>
        <input name="action">"http://my.sut.org/MyService/Operation1"</input>
        <input name="version">"1.2"</input>
    </send>
    <log>$send2{request}{envelope}</log> <!-- Print the SOAP envelope that was sent. -->
    <log>$send2{response}{body}</log> <!-- Print the response's SOAP body. -->
    <log>$send2{response}{headers}{Content-Type}</log> <!-- Print the response HTTP Content-Type header. -->

    <!--
        Make a SOAP 1.1 call to https://my.sut.org/api/service3 with MTOM attachments (a binary file and a JSON document).
    -->
    <assign to="send3File1{name}">"id1"</assign>
    <assign to="send3File1{content}">$send3Attachment1</assign>
    <assign to="send3File1{contentType}">"application/zip"</assign>

    <assign to="send3File2{name}">"id2"</assign>
    <assign to="send3File2{content}">$send3Attachment2</assign>
    <assign to="send3File2{contentType}">"application/json"</assign>
    <assign to="send3File2{forceText}">true()</assign> <!-- Add the attachment inline as text rather than Base64. -->

    <assign to="send3Attachments" append="true">$send3File1</assign>
    <assign to="send3Attachments" append="true">$send3File2</assign>

    <send id="send3" desc="Send SOAP message" from="Actor1" to="Actor2" handler="SoapMessagingV2">
        <input name="uri">"https://my.sut.org/api/service3"</input>
        <input name="envelope">$send3Envelope</input>
        <input name="attachments">$send3Attachments</input>
    </send>
    <log>$send3{request}{envelope}</log> <!-- Print the SOAP envelope that was sent. -->
    <log>$send3{request}{attachments}{id2}</log> <!-- Print the JSON file sent as an attachment. -->
    <log>$send3{request}{headers}{Content-Type}</log> <!-- Print the HTTP Content-Type header that was generated for the request. -->
    <log>$send3{response}{status}</log> <!-- Print the response status. -->
    <log>$send3{response}{body}</log> <!-- Print the response SOAP body. -->

.. index:: SoapMessagingV2 (receive)
.. index:: SoapMessagingV2 (receive - uriExtension)
.. index:: SoapMessagingV2 (receive - status)
.. index:: SoapMessagingV2 (receive - envelope)
.. index:: SoapMessagingV2 (receive - version)
.. index:: SoapMessagingV2 (receive - attachments)
.. index:: SoapMessagingV2 (receive - headers)

.. _handlers-soapmessagingv2-receive:

Use in receive steps
^^^^^^^^^^^^^^^^^^^^

To use this handler to **receive a SOAP web service call** from an external system, set it as the ``handler`` of a :ref:`receive <tdl-step-receive>` step 
(or the step's :ref:`transaction <tdl-step-btxn>`). In this case the supported step **inputs** are as follows:

.. csv-table::
    :header: "Input name", "Required?", "Type", "Description"
    :delim: |

    ``attachments`` | No | ``list`` | The MTOM attachments to include in the response, provided as a list of maps or a single map. Each map corresponds to an attachment and includes keys *"name"* (required), *"content"* (required), *"contentType"* and *"forceText"* (a boolean to force the attachment's inclusion as text).
    ``envelope`` | Yes | ``object`` | The SOAP envelope to respond with (as an XML object).
    ``headers`` | No | ``map`` | The HTTP headers to include in the response, provided as a map of key (header name) to comma-separated string values (header values).
    ``status`` | No | ``number`` | The HTTP status code to return (default is 200).
    ``uriExtension`` | No | ``string`` | A URI extension following the base endpoint path at which the request will be expected.
    ``version`` | No | ``string`` | The SOAP protocol version (either "1.1" or "1.2") to construct the response with. If not provided this is calculated from the provided Content-Type HTTP header, otherwise "1.1" is used.

When the :ref:`receive <tdl-step-receive>` step executes, the test session will be suspended until a matching request is received from the system under test.
The listening endpoint where this request will be received is constructed by concatenating in sequence:

1. The test engine's **base messaging URL**.
2. The unique **API key** of the system under test.
3. If defined, the **URI extension** provided as the step's ``uriExtension`` input.

As an example, for a Test Bed installation running locally with default settings, a system with API key "MY_API_KEY", and a provided URI extension of "/test";
the listen endpoint will be:

``http://localhost:8080/itbsrv/api/soap/MY_API_KEY/test``

Received query string parameters are ignored when matching incoming requests, except if a query string has been included in the ``uriExtension`` input.
In any case, once the :ref:`receive <tdl-step-receive>` step executes, the listen endpoint will be added in the test session log.

.. code-block:: none

    ...
    [2024-06-05 18:19:56] INFO  - Waiting to receive SOAP message at [http://localhost:8080/itbsrv/api/soap/7039EC8EX4786X4103XB40FXC7242D11E9B0/service] for step [Receive SOAP message]

.. note::

    **Supporting parallel test sessions:** The listen endpoint for a :ref:`receive <tdl-step-receive>` step determines also whether test sessions can execute in parallel. If no ``uriExtension`` is
    provided, all test sessions for the system (at least the ones with :ref:`receive <tdl-step-receive>` steps) will need to execute sequentially. You can
    :ref:`configure the relevant test cases <test-case>` with ``supportsParallelExecution`` as "false" to enforce this.

    If possible, try to have ``uriExtension`` differ across test cases, and ideally vary per test session.

Once the :ref:`receive <tdl-step-receive>` step completes, the resulting report will include two nested maps named ``request`` and ``response``, corresponding 
respectively to the received request and provided response. The ``request`` map includes the following:

.. csv-table::
    :header: "Output name", "Always present?", "Type", "Description"
    :delim: |

    ``attachments`` | No | ``map`` | The MTOM attachments received with the envelope. The key of each entry is the attachment name, whereas the value is the content.
    ``body`` | Yes | ``object`` | The SOAP body extracted from the envelope.
    ``envelope`` | Yes | ``object`` | The SOAP envelope that was received.
    ``headers`` | Yes | ``map`` | The request HTTP headers, provided as a map of key (header name) to comma-separated string values (header values).
    ``uri`` | Yes | ``string`` | The URI of the endpoint that was called.

The ``response`` map includes the following:

.. csv-table::
    :header: "Output name", "Always present?", "Type", "Description"
    :delim: |

    ``attachments`` | No | ``map`` | The MTOM attachments sent with the response envelope. The key of each entry is the attachment name, whereas the value is the content.
    ``body`` | No | ``object`` | The SOAP body extracted from the response envelope.
    ``envelope`` | No | ``object`` | The SOAP envelope provided as the response.
    ``headers`` | No | ``map`` | The response HTTP headers, provided as a map of key (header name) to comma-separated string values (header values).
    ``status`` | Yes | ``number`` | The HTTP status code of the response.

The following GITB TDL snippets provide various examples using the ``SoapMessagingV2`` handler in :ref:`send <tdl-step-send>` steps, as well as accessing
outputs (see inline comments for details per case).

.. code-block:: xml

    <!--
        Receive a SOAP call and respond with a success and specific envelope.
        The step will complete for a received SOAP call to http://localhost:8080/itbsrv/api/soap/MY_API_KEY/.
    -->
    <receive id="receive1" desc="Receive SOAP message" from="Actor1" to="Actor2" handler="SoapMessagingV2">
        <input name="envelope">$receive1Envelope</input>
    </receive>
    <log>$receive1{request}{body}</log> <!-- Print the received SOAP body. -->
    <log>$receive1{response}{envelope}</log> <!-- Print the SOAP envelope sent as the response. -->

    <!--
        Receive a SOAP call at a specific URI extension and respond with a 500 status and SOAP fault using SOAP version 1.2.
        The step will complete for a received SOAP call to http://localhost:8080/itbsrv/api/soap/MY_API_KEY/service.
    -->
    <receive id="receive2" desc="Receive SOAP message" from="Actor1" to="Actor2" handler="SoapMessagingV2">
        <input name="uriExtension">"/service"</input>
        <input name="status">"500"</input>
        <input name="version">"1.2"</input>
        <input name="envelope">$receive2FaultEnvelope</input>
    </receive>
    <log>$receive2{request}{body}</log> <!-- Print the received SOAP body. -->
    <log>$receive2{response}{status}</log> <!-- Print the status of the response. -->

    <!--
        Receive a SOAP call at a specific URI extension and respond with a success and envelope along with two MTOM attachments  (a binary file and a JSON document).
        The step will complete for a received SOAP call to http://localhost:8080/itbsrv/api/soap/MY_API_KEY/service.
    -->
    <assign to="receive3File1{name}">"id1"</assign>
    <assign to="receive3File1{content}">$receive3Attachment1</assign>
    <assign to="receive3File1{contentType}">"application/zip"</assign>

    <assign to="receive3File2{name}">"id2"</assign>
    <assign to="receive3File2{content}">$receive3Attachment2</assign>
    <assign to="receive3File2{contentType}">"application/json"</assign>
    <assign to="receive3File2{forceText}">true()</assign> <!-- Add the attachment inline as text rather than Base64. -->

    <assign to="receive3Attachments" append="true">$receive3File1</assign>
    <assign to="receive3Attachments" append="true">$receive3File2</assign>
    <receive id="receive3" desc="Receive SOAP message" from="Actor1" to="Actor2" handler="SoapMessagingV2">
        <input name="uriExtension">"/service"</input>
        <input name="envelope">$receive3Envelope</input>
        <input name="attachments">$receive3Attachments</input>
    </receive>

.. index:: Built-in processing handlers
.. _handlers-predefined-handlers-processing:

Built-in processing handlers
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The title of each section corresponds to the name of the handler that needs to be configured in the relevant step's ``handler`` attribute (in 
:ref:`process <tdl-step-process>` or :ref:`bptxn <tdl-step-bptxn>` steps).

.. index:: Base64Processor
.. index:: encode (Base64Processor)
.. index:: decode (Base64Processor)
.. index:: input (Base64Processor)
.. index:: dataUrl (Base64Processor)
.. _handlers-Base64Processor:

Base64Processor
+++++++++++++++

Used to manipulate Base64-encoded content for use in test cases. This processing handler supports but does not require a processing 
transaction to be established. The following operations are supported:

.. csv-table::
    :header: "Operation", "Description", "Input(s)", "Output(s)"

    ``decode``, Receive a ``string`` input that is Base64-encoded and return the ``binary`` output it corresponds to., Yes, A ``binary`` value named ``output`` in the resulting step's ``map``.
    ``encode``, Receive a ``binary`` input and return a ``string`` with its Base64-encoded representation., Yes, A ``string`` named ``output`` in the resulting step's ``map``.

Base64 encoding is a technique often used to represent arbitrary byte sequences as text. Using this processing handler you can work with Base64 encoded texts
that need to be decoded in test cases, but also encode binary content where this is needed. In both the encoding and decoding steps there is support for Base64 
content and also data URLs. Data URLs are commonly used in web representations for the inline definition of binary resources. A data URL is essentially the 
Base64-encoded bytes prefixed with the content's mime type as ``data:[mime type],base64,[BASE64 encoded string]`` (e.g. ``data:application/xml;base64,YXNoZGl1cXcgaGRva...``).

.. _handlers-Base64Processor_decode:

Base64Processor - decode
^^^^^^^^^^^^^^^^^^^^^^^^

The ``decode`` operation is used to decode a Base64-encoded string to its binary representation. The inputs it expects are as follows:

.. csv-table::
    :header: "Input name", "Required?", "Description"

    ``input``, Yes, The ``string`` value (expected to be Base64-encoded or formatted as a data URL) that will be processed to return its corresponding ``binary`` value.

The following example illustrates how to use the ``decode`` operation to decode and validate an image:

.. code-block:: xml

    <!--
        Decode a Base64-encoded string representing an image. The input could also be encoded as a data URL.
    -->
    <process id="decodeData" handler="Base64Processor" output="image" operation="decode">
        <input name="input">$myBase64EncodedText</input>
    </process>
    <!-- 
        Validate the image using an custom validator.
    -->
    <verify desc="Validate image" handler="$DOMAIN{imageValidator}">
        <input name="image">$image</input>
    </verify>

.. _handlers-Base64Processor_encode:

Base64Processor - encode
^^^^^^^^^^^^^^^^^^^^^^^^

The ``encode`` operation is used to encode a Base64-encoded string from binary data. The inputs it expects are as follows:

.. csv-table::
    :header: "Input name", "Required?", "Description"

    ``dataUrl``, No, A ``boolean`` flag that indicates whether or not the output should be formatted as a data URL (default is ``false``).
    ``input``, Yes, The ``binary`` value that will be encoded as a Base64 string.

The following example illustrates how to use the ``encode`` operation to process binary data:

.. code-block:: xml

    <!--
        Encode the binary variable "aBinaryVariable" and return the encoded string as "data1{output}".
    -->
    <process id="encode1" handler="Base64Processor" output="result" operation="encode">
        <input name="input">$aBinaryVariable</input>
    </process>
    <!--
        Encode the binary variable "aBinaryVariable" and return the encoded string as "data2{output}".
        The result in this case is formatted as a data URL.
    -->
    <process id="encode2" handler="Base64Processor" output="resultAsDataUrl" operation="encode">
        <input name="input">$aBinaryVariable</input>
        <input name="dataUrl">true()</input>
    </process>

.. index:: CollectionUtils
.. index:: append (CollectionUtils)
.. index:: size (CollectionUtils)
.. index:: clear (CollectionUtils)
.. index:: contains (CollectionUtils)
.. index:: find (CollectionUtils)
.. index:: randomKey (CollectionUtils)
.. index:: randomValue (CollectionUtils)
.. index:: remove (CollectionUtils)
.. index:: map (CollectionUtils)
.. index:: fromMap (CollectionUtils)
.. index:: toMap (CollectionUtils)
.. index:: list (CollectionUtils)
.. index:: fromList (CollectionUtils)
.. index:: toList (CollectionUtils)
.. index:: value (CollectionUtils)
.. index:: item (CollectionUtils)
.. index:: onlyMissing (CollectionUtils)
.. index:: ignoreCase (CollectionUtils)
.. _handlers-CollectionUtils:

CollectionUtils
+++++++++++++++

Used to process collections (maps and lists) in ways not possible otherwise with TDL expressions. This processing handler does not require
a processing transaction to be established. The following operations are supported:

.. csv-table::
    :header: "Operation", "Description", "Input(s)", "Output(s)"
    :delim: |

    ``append`` | Append the elements of one collection to another. | Yes | No.
    ``clear`` | Receive a collection as input and empty it. | Yes | No.
    ``contains`` | Check to see whether a collection contains a given value. | Yes | Yes, a ``boolean`` representing the check result.
    ``find`` | Search for and return a given value from the collection. | Yes | Yes, the matched item (if found).
    ``randomKey`` | Return a random key from a map. | Yes | Yes, one of the map's ``string`` keys.
    ``randomValue`` | Return a random value from a collection. | Yes | Yes, the selected value (type varies depending on the content).
    ``remove`` | Remove an entry from a collection. | Yes | No.
    ``size`` | Receive a collection as input and return the number of elements it contains. | Yes | Yes, a ``number`` named ``output`` in the resulting step's ``map``.

Collection or *container* variables represent flexible means of recording arbitrary sequences of data or hierarchical data structures. In particular
``map`` variables are very common as these are used to store results of :ref:`processing<tdl-step-process>`, :ref:`messaging<tdl-messaging-steps>` and :ref:`validation<tdl-step-verify>` operations.
Adding new elements to collections or replacing existing values is achieved using the :ref:`assign<tdl-step-assign>` step, where the
expressions used may also :ref:`determine collections<test-case-variables-from-expression-output>` that don't previously exist.
The ``CollectionUtils`` processing handler complements such operations by allowing further manipulations that cannot be achieved
through simple :ref:`expressions<test-case-expressions>`.

.. _handlers-CollectionUtils_append:

CollectionUtils - append
^^^^^^^^^^^^^^^^^^^^^^^^

The ``append`` operation allows you to merge two collections, by appending the values of one collection (the "from" collection) to
another one (the "to" collection). The inputs it expects are as follows:

.. csv-table::
    :header: "Input name", "Required?", "Description"
    :delim: |

    ``fromList`` | No | The ``list`` to read the values from (if the collection is a :ref:`list<test-case-types-lists>`). Either this or the ``fromMap`` input must be provided.
    ``fromMap`` | No | The ``map`` to read the values from (if the collection is a :ref:`list<test-case-types-maps>`). Either this or the ``fromList`` input must be provided.
    ``ignoreCase`` | No | When ``onlyMissing`` is set to "true", whether the matching of existing items will ignore casing (default is "false").
    ``onlyMissing`` | No | Whether only items missing from the target ``map`` or ``list`` should be added (default is "false"). For a ``list`` the check is made on values whereas for a ``map`` it is on keys.
    ``toList`` | No | The ``list`` to append the values to (if the collection is a :ref:`list<test-case-types-lists>`). Either this or the ``toMap`` input must be provided.
    ``toMap`` | No | The ``map`` to append the values to (if the collection is a :ref:`list<test-case-types-maps>`). Either this or the ``toList`` input must be provided.

The ``append`` operation does not return an output but rather modifies directly the target collection. Appended entries are added to the end
of the collection, maintaining their insertion order. Furthermore, you can leverage the ``onlyMissing`` and ``ignoreCase`` inputs to have only
missing and unique items appended to the target collection. The following examples illustrate the operation's use:

.. code-block:: xml

    <!-- Create a map -->
    <assign to="map1{a}">'Value A'</assign>
    <assign to="map1{b}">'Value B'</assign>

    <!-- Create a second map -->
    <assign to="map2{x}">'Value X'</assign>
    <assign to="map2{y}">'Value 2'</assign>

    <!-- Append map1 to the end of map2 -->
    <process handler="CollectionUtils" operation="append">
        <input name="fromMap">$map1</input>
        <input name="toMap">$map2</input>
    </process>

    <!-- Prints (the newly appended) 'Value A' -->
    <log>$map2{a}</log>

    <!-- Create two lists -->
    <assign to="list1" append="true">'A'</assign>
    <assign to="list1" append="true">'B'</assign>
    <assign to="list1" append="true">'C'</assign>

    <assign to="list2" append="true">'a'</assign>
    <assign to="list2" append="true">'X'</assign>
    <assign to="list2" append="true">'Z'</assign>

    <!-- Append list1 to list2 without duplicates (case-insensitive) -->
    <process handler="CollectionUtils" operation="append">
        <input name="fromList">$list1</input>
        <input name="toList">$list2</input>
        <input name="onlyMissing">true()</input>
        <input name="ignoreCase">true()</input>
    </process>
    <!-- Prints "a,X,Z,B,C" (having skipped the "A" element of list1) -->
    <log>$list2</log>

.. _handlers-CollectionUtils_clear:

CollectionUtils - clear
^^^^^^^^^^^^^^^^^^^^^^^

The ``clear`` operation allows a test case to empty the contents of a given collection if this becomes necessary. 
The inputs it expects are as follows:

.. csv-table::
    :header: "Input name", "Required?", "Description"
    :delim: |

    ``list`` | No | The ``list`` to be cleared (if the collection is a :ref:`list<test-case-types-lists>`). Either this or the ``map`` input must be provided.
    ``map`` | No | The ``map`` to be cleared (if the collection is a :ref:`map<test-case-types-maps>`). Either this or the ``list`` input must be provided.

The following examples illustrate how this works for lists and maps:

.. code-block:: xml

    <!-- Create a map with three elements -->
    <assign to="aMap{a}">'Value 1'</assign>
    <assign to="aMap{b}">'Value 2'</assign>
    <assign to="aMap{c}">'Value 3'</assign>
    <!-- Create a list with two elements -->
    <assign to="aList" append="true">'Value 1'</assign>
    <assign to="aList" append="true">'Value 2'</assign>
    <!-- Empty the map -->
    <process handler="CollectionUtils" operation="clear">
        <input name="map">$aMap</input>
    </process>
    <!-- Empty the list -->
    <process handler="CollectionUtils" operation="clear">
        <input name="list">$aList</input>
    </process>

.. _handlers-CollectionUtils_contains:

CollectionUtils - contains
^^^^^^^^^^^^^^^^^^^^^^^^^^

The ``contains`` operation allows checking whether a value is defined within a collection. In case the collection is a ``map``, the lookup is
done on the basis of the entries' keys. Otherwise for a ``list`` the lookup considers the contained elements' values.
The inputs it expects are as follows:

.. csv-table::
    :header: "Input name", "Required?", "Description"
    :delim: |

    ``ignoreCase`` | No | Whether the matching of items to determine the operation's result will ignore casing (default is "false").
    ``list`` | No | The ``list`` to be considered (if the collection is a :ref:`list<test-case-types-lists>`). Either this or the ``map`` input must be provided.
    ``map`` | No | The ``map`` to be considered (if the collection is a :ref:`map<test-case-types-maps>`). Either this or the ``list`` input must be provided.
    ``value`` | Yes | The value to look for.

The following examples illustrate the operation's use:

.. code-block:: xml

    <!-- Create a map -->
    <assign to="aMap{a}">'Value 1'</assign>
    <assign to="aMap{b}">'Value 2'</assign>
    <assign to="aMap{c}">'Value 3'</assign>
    <!-- Lookup an existing value -->
    <process handler="CollectionUtils" output="mapCheck1" operation="contains">
        <input name="map">$aMap</input>
        <input name="value">'b'</input>
    </process>
    <!-- Prints "true" -->
    <log>$mapCheck1</log>
    <!-- Lookup an non-existing value -->
    <process handler="CollectionUtils" output="mapCheck2" operation="contains">
        <input name="map">$aMap</input>
        <input name="value">'x'</input>
    </process>
    <!-- Prints "false" -->
    <log>$mapCheck2</log>
    <!-- Lookup an existing value in a non case-sensitive way -->
    <process handler="CollectionUtils" output="mapCheck3" operation="contains">
        <input name="map">$aMap</input>
        <input name="value">'A'</input>
        <input name="ignoreCase">true()</input>
    </process>
    <!-- Prints "true" -->
    <log>$mapCheck3</log>

    <!-- Create a list -->
    <assign to="aList" append="true">'Value 1'</assign>
    <assign to="aList" append="true">'Value 2'</assign>
    <assign to="aList" append="true">'Value 3'</assign>
    <!-- Lookup an existing value -->
    <process handler="CollectionUtils" output="listCheck1" operation="contains">
        <input name="map">$aList</input>
        <input name="value">'Value 1'</input>
    </process>
    <!-- Prints "true" -->
    <log>$listCheck1</log>
    <!-- Lookup an non-existing value -->
    <process handler="CollectionUtils" output="listCheck2" operation="contains">
        <input name="map">$aList</input>
        <input name="value">'Value X'</input>
    </process>
    <!-- Prints "false" -->
    <log>$listCheck2</log>
    <!-- Lookup an existing value in a non case-sensitive way -->
    <process handler="CollectionUtils" output="listCheck3" operation="contains">
        <input name="map">$aList</input>
        <input name="value">'value 1'</input>
        <input name="ignoreCase">true()</input>
    </process>
    <!-- Prints "true" -->
    <log>$listCheck3</log>

.. _handlers-CollectionUtils_find:

CollectionUtils - find
^^^^^^^^^^^^^^^^^^^^^^

The ``find`` operation is used to return a value from a collection. In case the collection is a ``map``, the lookup is
done on the basis of the entries' keys. Otherwise for a ``list`` the lookup considers the contained elements' values.
The inputs it expects are as follows:

.. csv-table::
    :header: "Input name", "Required?", "Description"
    :delim: |

    ``ignoreCase`` | No | Whether the matching of items to will ignore casing (default is "false").
    ``list`` | No | The ``list`` to be considered (if the collection is a :ref:`list<test-case-types-lists>`). Either this or the ``map`` input must be provided.
    ``map`` | No | The ``map`` to be considered (if the collection is a :ref:`map<test-case-types-maps>`). Either this or the ``list`` input must be provided.
    ``value`` | Yes | The value to look for.

The following examples illustrate the operation's use:

.. code-block:: xml

    <!--
        Map example. Look up a value from a map (case-sensitive lookup based on the key).
    -->
    <assign to="myMap{key1}">"Value1"</assign>
    <assign to="myMap{key2}">"Value2"</assign>
    <process handler="CollectionUtils" output="mapValue" operation="find">
        <input name="map">$myMap</input>
        <input name="value">'key1'</input>
    </process>
    <process handler="VariableUtils" operation="exists" input="mapValue" output="mapValueFound"/>
    <!-- Prints "true" -->
    <log>$mapValueFound</log>
    <!-- Prints "Value1" -->
    <log>$mapValue</log>

    <!--
        List example. Look up a value from a map (case-sensitive lookup).
    -->
    <assign to="myList" append="true">"Value1"</assign>
    <assign to="myList" append="true">"Value2"</assign>
    <process handler="CollectionUtils" output="listValue" operation="find">
        <input name="list">$myList</input>
        <input name="value">'value1'</input>
        <input name="ignoreCase">true()</input>
    </process>            
    <process handler="VariableUtils" operation="exists" input="listValue" output="listValueFound"/>
    <!-- Prints "true" -->
    <log>$listValueFound</log>
    <!-- Prints "Value1" -->
    <log>$listValue</log>

.. _handlers-CollectionUtils_randomKey:

CollectionUtils - randomKey
^^^^^^^^^^^^^^^^^^^^^^^^^^^

The ``randomKey`` operation works specifically with ``map`` variables and is used to randomly retrieve one of its keys.
The inputs it expects are as follows:

.. csv-table::
    :header: "Input name", "Required?", "Description"
    :delim: |

    ``map`` | Yes | The ``map`` to be considered.

The following example illustrates the operation's use:

.. code-block:: xml

    <!-- Create a map -->
    <assign to="aMap{a}">'Value 1'</assign>
    <assign to="aMap{b}">'Value 2'</assign>
    <!-- Get one of the map's keys -->
    <process handler="CollectionUtils" output="value1" operation="randomKey">
        <input name="map">$aMap</input>
    </process>
    <!-- Prints either "a" or "b" -->
    <log>$value1</log>

.. _handlers-CollectionUtils_randomValue:

CollectionUtils - randomValue
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

The ``randomValue`` operation works is used to randomly retrieve randomly one of the provided collection's values.
The inputs it expects are as follows:

.. csv-table::
    :header: "Input name", "Required?", "Description"
    :delim: |

    ``list`` | No | The ``list`` to be considered (if the collection is a :ref:`list<test-case-types-lists>`). Either this or the ``map`` input must be provided.
    ``map`` | No | The ``map`` to be considered (if the collection is a :ref:`map<test-case-types-maps>`). Either this or the ``list`` input must be provided.

The following example illustrates the operation's use:

.. code-block:: xml

    <!-- Create a map -->
    <assign to="aMap{a}">'Value 1'</assign>
    <assign to="aMap{b}">'Value 2'</assign>
    <!-- Get one of the map's values -->
    <process handler="CollectionUtils" output="value1" operation="randomValue">
        <input name="map">$aMap</input>
    </process>
    <!-- Prints either "Value 1" or "Value 2" -->
    <log>$value1</log>

    <!-- Create a list -->
    <assign to="aList" append="true">'Value 1'</assign>
    <assign to="aList" append="true">'Value 2'</assign>
    <!-- Get one of the list's values -->
    <process handler="CollectionUtils" output="value2" operation="randomValue">
        <input name="list">$aList</input>
    </process>
    <!-- Prints either "Value 1" or "Value 2" -->
    <log>$value2</log>    

.. _handlers-CollectionUtils_remove:

CollectionUtils - remove
^^^^^^^^^^^^^^^^^^^^^^^^

The ``remove`` operation is used to remove specific entries from a collection. When using a ``map`` the removed entry is matched
based on its key. For a ``list``, the entry to remove is identified by its zero-based index. The inputs it expects are as follows:

.. csv-table::
    :header: "Input name", "Required?", "Description"
    :delim: |

    ``list`` | No | The ``list`` to be considered (if the collection is a :ref:`list<test-case-types-lists>`). Either this or the ``map`` input must be provided.
    ``map`` | No | The ``map`` to be considered (if the collection is a :ref:`map<test-case-types-maps>`). Either this or the ``list`` input must be provided.
    ``item`` | Yes | In case of a ``list`` this is a ``number`` set with the zero-based index of the element to remove. For a ``map`` this is the ``string`` key of the entry to be removed.

The following examples illustrate the operation's use:

.. code-block:: xml

    <!-- Create a map -->
    <assign to="aMap{a}">'Value 1'</assign>
    <assign to="aMap{b}">'Value 2'</assign>
    <!-- Remove the entry with key "a" -->
    <process handler="CollectionUtils" operation="remove">
        <input name="map">$aMap</input>
        <input name="item">'a'</input>
    </process>

    <!-- Create a list -->
    <assign to="aList" append="true">'Value 1'</assign>
    <assign to="aList" append="true">'Value 2'</assign>
    <!-- Remove the second entry from the list -->
    <process handler="CollectionUtils" operation="remove">
        <input name="list">$aList</input>
        <!-- Providing "1" given that indexes are zero-based -->
        <input name="item">1</input>
    </process>

.. _handlers-CollectionUtils_size:

CollectionUtils - size
^^^^^^^^^^^^^^^^^^^^^^

The ``size`` operation allows a test case to determine a collection's size. This can be particularly useful in the case of
operations that return an arbitrary number of data items as a ``list`` which we need to iterate over.
The inputs it expects are as follows:

.. csv-table::
    :header: "Input name", "Required?", "Description"
    :delim: |

    ``list`` | No | The ``list`` of which the elements are to be counted (if the collection is a :ref:`list<test-case-types-lists>`). Either this or the ``map`` input must be provided.
    ``map`` | No | The ``map`` of which the elements are to be counted (if the collection is a :ref:`map<test-case-types-maps>`). Either this or the ``list`` input must be provided.

The following examples illustrate how this operation can be used:

.. code-block:: xml

    <!-- Create a map with three elements -->
    <assign to="aMap{a}">'Value 1'</assign>
    <assign to="aMap{b}">'Value 2'</assign>
    <assign to="aMap{c}">'Value 3'</assign>
    <!-- Create a list with two elements -->
    <assign to="aList" append="true">'Value 1'</assign>
    <assign to="aList" append="true">'Value 2'</assign>
    <!-- Calculate the size of the map -->
    <process id="aMapSize" handler="CollectionUtils" operation="size">
        <input name="map">$aMap</input>
    </process>
    <!-- Prints "3" -->
    <log>$aMapSize{output}</log>
    <!-- Calculate the size of the list -->
    <process id="aListSize" handler="CollectionUtils" operation="size">
        <input name="list">$aList</input>
    </process>
    <!-- Prints "2" -->
    <log>$aListSize{output}</log>
    <!-- Print each list element. -->
    <foreach desc="Iterate list" counter="index" start="0" end="$aListSize{output}">
        <do>
            <!-- Prints "Value 1" and then "Value 2" -->
            <log>aList{$index}</log>
        </do>
    </foreach>

.. note::
    **Nested collections:** If a collection structure contains itself further collection structures as elements, the
    ``size`` operation will only count the collection's top level elements.

.. index:: DelayProcessor
.. index:: delay (DelayProcessor)
.. index:: duration (DelayProcessor)
.. _handlers-DelayProcessor:

DelayProcessor
++++++++++++++

Used to pause a test session for a given duration. The following operation is supported:

.. csv-table::
    :header: "Operation", "Description", "Input(s)", "Output(s)"
    :delim: |

    ``delay`` | Delay the test session for a given duration. | Yes | No.

The input parameters expected by the ``delay`` operation are as follows:

.. csv-table::
    :header: "Input name", "Required?", "Description"
    :delim: |

    ``duration`` | Yes | A ``number`` representing the duration (expressed in milliseconds) to delay for.

The following examples illustrate how the ``DelayProcessor`` can be used to pause the test session.

.. code-block:: xml

    <!-- Wait for a fixed 5 seconds before proceeding. -->
    <process handler="DelayProcessor" operation="delay" input="5000"/> 

    <!-- Wait for a random duration between 5 and 10 seconds. -->
    <process handler="TokenGenerator" operation="random" output="randomDelay">
        <input name="minimum">5</input>
        <input name="maximum">10</input>
        <input name="integer">true()</input>
    </process>
    <process handler="DelayProcessor" operation="delay" output="$randomDelay"/> 

.. index:: DisplayProcessor
.. index:: result (DisplayProcessor)
.. index:: parameters (DisplayProcessor)
.. index:: contentTypes (DisplayProcessor)
.. _handlers-DisplayProcessor:

DisplayProcessor
++++++++++++++++

Used to display arbitrary content to users as a report. Using this instead of a :ref:`user interaction step<tdl-step-interact>` allows you
to display content when the user clicks the relevant step report, as opposed to always producing a popup. This makes it a useful mechanism
for including additional information in the test's output without distracting the user. The following operation is supported:

.. csv-table::
    :header: "Operation", "Description", "Input(s)", "Output(s)"
    :delim: |

    ``display`` | Include the provided data in a step report that the user may choose to view. | Yes | No.

The input parameters expected by the ``display`` operation are as follows:

.. csv-table::
    :header: "Input name", "Required?", "Description"
    :delim: |

    ``contentTypes`` | No | A ``map`` including the content types (e.g. ``application/json``) to consider when displaying different parameters.
    ``result`` | No | A ``string`` with the status (``SUCCESS``, ``FAILURE`` or ``WARNING``) to use for the relevant :ref:`process<tdl-step-process>` step (default is ``SUCCESS``).
    ``parameters`` | No | A ``map`` including the values to display (labelled using the ``map`` keys).

The following example illustrates usage of the ``DisplayProcessor`` to create a step report for a given set of data that the user may
choose to view:

.. code-block:: xml

    <!-- Display a report based on a set of parameters. -->
    <assign to="parameters{textValue}">'A sample value'</assign>
    <assign to="parameters{listValues}" append="true">'Value 1'</assign>
    <assign to="parameters{listValues}" append="true">'Value 2'</assign>
    <assign to="parameters{listValues}" append="true">`Value 3`</assign>
    <process desc="Show values" hidden="false" handler="DisplayProcessor" input="$parameters"/>

    <!-- Display a report but also mark the step as failed if we have errors. -->
    <assign to="result">if ($status = "OK") then "SUCCESS" else "FAILURE"</assign>
    <assign to="report{comments}">$errorDescription</assign>    
    <process desc="Show values" hidden="false" handler="DisplayProcessor">
        <input name="result">$result</input>
        <input name="parameters">$report</input>
    </process>

In case the ``DisplayProcessor`` is used to display **large content** or **complete files**, we can also provide a hint to the test engine
on how such data is to be displayed. This is done by means of the ``contentTypes`` input, an optional ``map`` that can be set with 
the content types (e.g. ``application/json``) to consider per parameter. When a content type is set for a given parameter this will
affect its syntax highlighting when displaying it within editors and also the type of file generated when it is downloaded.

The approach used to specify content types is to match fully, in terms of parameter names and structure, the corresponding
``parameters`` map. Matching of specific parameters is done on the basis of their map key and nesting level, whereas the 
content type values are of type ``string``.

To clarify this, consider the following example where a ``DisplayProcessor`` is used to display two files (named ``input`` and ``output``).
Notice how the ``contentTypes`` input is defined in a manner identical to the actual data to be displayed.

.. code-block:: xml

    <!-- Define the data. -->
    <assign to="params{input}">$file1</assign>
    <assign to="params{output}">$file2</assign>
    <!-- Define the content types. -->
    <assign to="contentTypes{input}">'application/json'</assign>
    <assign to="contentTypes{output}">'application/xml'</assign>
    <process desc="Process data" hidden="false" handler="DisplayProcessor">
        <input name="parameters">$params</input>
        <input name="contentTypes">$contentTypes</input>
    </process>

Content types don't need to cover all parameters, only those for which they are relevant or known. For example in the following case
we only define a content type for the first displayed file, omitting it for simple strings and for the second file for which the 
content type is unknown.

.. code-block:: xml

    <!-- Define the data. -->
    <assign to="params{aFile}">$file</assign>
    <assign to="params{countryCode}">$countryCode</assign>
    <assign to="params{message}">"Transformation was successful."</assign>
    <assign to="params{aSecondFile}">$secondFile</assign>
    <!-- Define the content type only for the first file. -->
    <assign to="contentTypes{aFile}">'application/xml'</assign>
    <process desc="Process data" hidden="false" handler="DisplayProcessor">
        <input name="parameters">$params</input>
        <input name="contentTypes">$contentTypes</input>
    </process>

Finally, the following example illustrates how content types can be provided when the parameters are defined within complex
structures (maps and lists, nested at different levels).

.. code-block:: xml

    <!-- Define the data. -->
    <assign to="params{input}{file1}">$file1</assign>
    <assign to="params{input}{file2}">$file2</assign>
    <assign to="params{input}{messageId}">$messageIdentifier</assign>
    <assign to="params{input}{attachments}" append="true">$attachment1</assign>
    <assign to="params{input}{attachments}" append="true">$attachment2</assign>
    <assign to="params{output}{response}">$response</assign>
    <assign to="params{output}{message}">"Input processed successfully."</assign>
    <!-- Define the content types. -->
    <assign to="types{input}{file1}">"application/xml"</assign>
    <assign to="types{input}{file2}">"application/xml"</assign>
    <assign to="types{input}{attachments}" append="true">"text/plain"</assign>
    <assign to="types{input}{attachments}" append="true">"application/pdf"</assign>
    <assign to="types{output}{response}">"application/json"</assign>
    <!-- Call the process step. -->
    <process desc="Process data" hidden="false" handler="DisplayProcessor">
        <input name="parameters">$params</input>
        <input name="contentTypes">$types</input>
    </process>

.. note::
    **DisplayProcessor in non-hidden steps:** :ref:`Process steps<tdl-step-process>` are by default set as ``hidden``, meaning
    that they execute but are not displayed and do not produce a visible report. When using the ``DisplayProcessor`` you need
    to ensure that ``hidden`` is set to ``false`` for its use to be meaningful.

.. index:: JsonPointerProcessor
.. index:: JSONPointerProcessor
.. index:: process (JsonPointerProcessor)
.. index:: content (JsonPointerProcessor)
.. index:: pointer (JsonPointerProcessor)
.. _handlers-JSONPointerProcessor:

JsonPointerProcessor
++++++++++++++++++++

Used to extract values or complete elements from JSON content based on a provided `JSON Pointer expression <https://datatracker.ietf.org/doc/html/rfc6901>`_.
The following operation is supported:

.. csv-table::
    :header: "Operation", "Description", "Input(s)", "Output(s)"
    :delim: |

    ``process`` | Use a JSON Pointer expression to extract a value or full elements from the provided JSON content. | Yes | A ``string`` named ``output`` in the resulting step's ``map``.

The input parameters expected by the ``process`` operation are as follows:

.. csv-table::
    :header: "Input name", "Required?", "Description"
    :delim: |

    ``content`` | Yes | A ``string`` representing the JSON content to process.
    ``pointer`` | Yes | A ``string`` representing the JSON Pointer expression to use.

The following example illustrates how the ``JsonPointerProcessor`` can be used to extract a value from JSON content provided by the user via 
an :ref:`interact<tdl-step-interact>` step:

.. code-block:: xml

    <!--
        Have the user enter the JSON content to process.
    -->
    <interact id="data" desc="Receive input">
        <request name="json" desc="Enter JSON content:" inputType="CODE" mimeType="application/json"/>
    </interact>
    <!--
        Retrieve a value from the provided JSON content. The content is expected to structured as follows:
        {
            "user": {
                "address": {
                    "streetName": "An address"
                }
            }
        }
    -->
    <process handler="JsonPointerProcessor" operation="process" output="result">
        <input name="content">$data{json}</input>
        <input name="pointer">"/user/address/streetName"</input>
    </process>
    <!--
        Log the result.
    -->
    <log>$result</log>

.. index:: RdfUtils
.. index:: convert (RdfUtils)
.. index:: merge (RdfUtils)
.. index:: ask (RdfUtils)
.. index:: select (RdfUtils)
.. index:: construct (RdfUtils)
.. index:: model (RdfUtils)
.. index:: models (RdfUtils)
.. index:: query (RdfUtils)
.. index:: inputContentType (RdfUtils)
.. index:: inputContentTypes (RdfUtils)
.. index:: outputContentType (RdfUtils)
.. index:: output (RdfUtils)

.. _handlers-RdfUtils:

RdfUtils
++++++++

The ``RdfUtils`` processor is used to perform different types of manipulations on RDF models. This processing handler
does not require a processing transaction to be established. The following operations are supported:

.. csv-table::
    :header: "Operation", "Description", "Input(s)", "Output(s)"
    :delim: |

    ``ask`` | Run a SPARQL ASK query on an RDF model. | Yes | Yes, a ``boolean`` value with the ask query's result.
    ``construct`` | Run a SPARQL CONSTRUCT query on an RDF model to produce a new RDF graph. | Yes | Yes, a ``string`` representation of the resulting RDF graph.
    ``convert`` | Convert a provided RDF model from its current RDF syntax to another. | Yes | Yes, a ``string`` representation of the resulting RDF model.
    ``merge`` | Merge multiple RDF models to a single aggregate one. | Yes | Yes, a ``string`` representation of the resulting RDF model.
    ``select`` | Run a SPARQL SELECT query on an RDF model to produce an RDF result set. | Yes | Yes, a ``string`` representation of the resulting RDF result set.

Manipulating RDF models using this processor's operations grants you the flexibility needed to handle any kind of RDF processing in test cases.

.. _handlers-RdfUtils_ask:

RdfUtils - ask
^^^^^^^^^^^^^^

The ``ask`` operation enables you to run `SPARQL ASK queries <https://www.w3.org/TR/rdf-sparql-query/#ask>`__ against a provided RDF model, to check whether
they produce a match (returned as a ``boolean`` result). The input parameters supported by this operation are as follows:

.. csv-table::
    :header: "Input name", "Required?", "Description"
    :delim: |

    ``inputContentType`` | Yes | The ``string`` content type of the provided model.
    ``model`` | Yes | The ``string`` RDF model to run the query against.
    ``query`` | Yes | The ``string`` SPARQL query to execute.

The following example illustrates the ``ask`` operation's use:

.. code-block:: xml

    <!--
        Define the ASK query to use.
    -->
    <assign to="query"><![CDATA["PREFIX ns1: <http://itb.ec.europa.eu/sample/po#>

    ASK WHERE {
      ?item a ns1:Item ;
              ns1:quantity ?q .
    FILTER(?q > 1000)
    }"]]></assign>
    <!-- 
        Run the query.
    -->
    <process output="askResult" handler="RdfUtils" operation="ask">
        <input name="model">$ttlSample</input>
        <input name="inputContentType">"text/turtle"</input>
        <input name="query">$query</input>
    </process>
    <log>$askResult</log>


.. _handlers-RdfUtils_construct:

RdfUtils - construct
^^^^^^^^^^^^^^^^^^^^

The ``construct`` operation enables you to run `SPARQL CONSTRUCT queries <https://www.w3.org/TR/rdf-sparql-query/#construct>`__ against a provided RDF model,
to generate from it a new graph based on the resulting triples (returned as a ``string`` result). The input parameters supported by this operation are as follows:
 
.. csv-table::
    :header: "Input name", "Required?", "Description"
    :delim: |

    ``inputContentType`` | Yes | The ``string`` content type of the provided model.
    ``model`` | Yes | The ``string`` RDF model to run the query against.
    ``outputContentType`` | No | The ``string`` content type to use for the produced graph. If not provided, the content type of the input model will be used.
    ``query`` | Yes | The ``string`` SPARQL query to execute.

The following example illustrates the ``construct`` operation's use:

.. code-block:: xml

    <!-- 
        Define the CONSTRUCT query.
    -->
    <assign to="query"><![CDATA["PREFIX ns1: <http://itb.ec.europa.eu/sample/po#>
    PREFIX : <http://my.sample.po/summary#>

    CONSTRUCT {
        ?item :summaryOf ?productName ;
        :totalCost ?cost .
    }
    WHERE {
        ?item a ns1:Item ;
        ns1:productName ?productName ;
        ns1:quantity ?q ;
        ns1:priceEUR ?p .
        BIND(?q * ?p AS ?cost)
    }"]]></assign>
    <!-- 
        Run the query.
    -->
    <process output="constructResult" handler="RdfUtils" operation="construct">
        <input name="model">$ttlSample</input>
        <input name="inputContentType">"text/turtle"</input>
        <!-- 
            Optional. If not provided defaults to the input model's content type.
        -->
        <input name="outputContentType">"application/rdf+xml"</input>
        <input name="query">$query</input>
    </process>
    <log>$constructResult</log>

.. _handlers-RdfUtils_convert:

RdfUtils - convert
^^^^^^^^^^^^^^^^^^

The ``convert`` operation enables you to convert an RDF model to another RDF syntax. The input parameters supported by this operation are as follows:

.. csv-table::
    :header: "Input name", "Required?", "Description"
    :delim: |

    ``inputContentType`` | Yes | The ``string`` content type of the provided model.
    ``model`` | Yes | The ``string`` RDF model to convert.
    ``outputContentType`` | Yes | The ``string`` content type to use for the output model.

The following example illustrates the ``convert`` operation's use:

.. code-block:: xml

    <!--
        Convert the provided Turtle model to RDF/XML.
    -->
    <process output="convertResult" handler="RdfUtils" operation="convert">
        <input name="model">$ttlSample</input>
        <input name="inputContentType">"text/turtle"</input>
        <input name="outputContentType">"application/rdf+xml"</input>
    </process>
    <!--
        Output the converted model.
    -->
    <log>$convertResult</log>

.. _handlers-RdfUtils_merge:

RdfUtils - merge
^^^^^^^^^^^^^^^^

The ``merge`` operation enables you to merge multiple RDF models to a new one, aggregating their combined triples into as a new graph.
The input parameters supported by this operation are as follows:

.. csv-table::
    :header: "Input name", "Required?", "Description"
    :delim: |

    ``inputContentType`` | No | The ``string`` content type for all provided models. Either this or the ``inputContentTypes`` must be provided.
    ``inputContentTypes`` | No | A ``list`` of ``string`` content types for the provided models. Either this or the ``inputContentType`` must be provided.
    ``models`` | Yes | The ``string`` RDF model to convert.
    ``outputContentType`` | No | The ``string`` content type to use for the output model. If not provided the first provided input content type will be used.

The following example illustrates the ``merge`` operation's use:

.. code-block:: xml

    <!-- 
        Merge models with different RDF syntaxes and return the resulting model in RDF/XML.
    -->
    <assign to="models" append="true">$ttlSample</assign>
    <assign to="models" append="true">$rdfXmlSample</assign>
    <assign to="inputContentTypes" append="true">"text/turtle"</assign>
    <assign to="inputContentTypes" append="true">"application/rdf+xml"</assign>
    <process output="mergeResult" handler="RdfUtils" operation="merge">
        <input name="models">$models</input>
        <input name="inputContentTypes">$inputContentTypes</input>
        <!--
            Optional. If not provided the output type will match the one of the first input model.
        -->
        <input name="outputContentType">"application/rdf+xml"</input> Optional
    </process>
    <log>$mergeResult</log>

    <!-- 
        Merge models that are all in Turtle format and return the resulting model in Turtle.
        We don't specify here the outputContentType as it defaults to the input type.
    -->
    <assign to="ttlModels" append="true">$ttlSample1</assign>
    <assign to="ttlModels" append="true">$ttlSample2</assign>
    <process output="mergeResult2" handler="RdfUtils" operation="merge">
        <input name="models">$models</input>
        <input name="inputContentType">"text/turtle"</input>
    </process>
    <log>$mergeResult2</log>

.. _handlers-RdfUtils_select:

RdfUtils - select
^^^^^^^^^^^^^^^^^

The ``select`` operation enables you to run `SPARQL SELECT queries <https://www.w3.org/TR/rdf-sparql-query/#select>`__ against a provided RDF model,
resulting in an RDF result set (returned as a ``string`` result). The input parameters supported by this operation are as follows:
 
.. csv-table::
    :header: "Input name", "Required?", "Description"
    :delim: |

    ``inputContentType`` | Yes | The ``string`` content type of the provided model.
    ``model`` | Yes | The ``string`` RDF model to run the query against.
    ``outputContentType`` | No | The ``string`` content type to use for the produced result set. If not provided, ``application/sparql-results+xml`` will be used as a default.
    ``query`` | Yes | The ``string`` SPARQL query to execute.

The following example illustrates the ``select`` operation's use:

.. code-block:: xml

    <!-- 
        Define the SELECT query.
    -->
    <assign to="query"><![CDATA["PREFIX ns1: <http://itb.ec.europa.eu/sample/po#>

    SELECT ?item ?productName ?quantity ?priceEUR ?totalCost WHERE {
      ?item a ns1:Item ;
            ns1:productName ?productName ;
            ns1:quantity ?quantity ;
            ns1:priceEUR ?priceEUR .
      BIND(?quantity * ?priceEUR AS ?totalCost)
    }"]]></assign>
    <!-- 
        Run the query.
    -->
    <process output="selectResult" handler="RdfUtils" operation="select">
        <input name="model">$ttlSample</input>
        <input name="inputContentType">"text/turtle"</input>
        <!-- 
            Optional. If not provided defaults to application/sparql-results+xml. 
        -->
        <input name="outputContentType">"text/csv"</input>
        <input name="query">$query</input>
    </process>
    <log>$selectResult</log>

.. index:: RegExpProcessor
.. index:: check (RegExpProcessor)
.. index:: collect (RegExpProcessor)
.. index:: input (RegExpProcessor)
.. index:: expression (RegExpProcessor)
.. index:: output (RegExpProcessor)
.. _handlers-RegExpProcessor:

RegExpProcessor
+++++++++++++++

Used to process texts using regular expressions, to verify whether they match a specific pattern or to extract data. This processing handler
does not require a processing transaction to be established. The following operations are supported:

.. csv-table::
    :header: "Operation", "Description", "Input(s)", "Output(s)"
    :delim: |

    ``check`` | Check to see if a ``string`` matches an expression. | Yes | Yes, a ``boolean`` named ``output`` in the resulting step's ``map``.
    ``collect`` | Use an expression to collect data from a provided ``string`` based on the expression's capturing groups. | Yes | A ``list`` of ``string`` values, one value per matched group.

Regular expressions offer a very powerful means of describing a text's content and extracting from it certain parts for further processing. They can be used
against any text content, offering a counterpart to the use of XPath in the :ref:`assign<tdl-step-assign>` step that is best adapted, but also limited, to XML structures.
The regular expressions are expected to be provided using the `syntax used by the Java language`_.

.. _handlers-RegExpProcessor_check:

RegExpProcessor - check
^^^^^^^^^^^^^^^^^^^^^^^

The ``check`` operation can be used to verify whether a given text matches a specific pattern. This may at first appear similar to the
:ref:`RegExpValidator<handlers-RegExpValidator>`, however there is a subtle difference: using the :ref:`RegExpValidator<handlers-RegExpValidator>`
constitutes an assertion made by the test case which, if failed, would likely mean that the test session itself is considered failed. The ``check``
operation doesn't presume anything for the test session's status, but is rather used as an internal check to e.g. determine
whether an optional set of steps should be followed. The input parameters expected by this operation are as follows:

.. csv-table::
    :header: "Input name", "Required?", "Description"
    :delim: |

    ``expression`` | Yes | A ``string`` with the expression that will be used to check the ``input``.
    ``input`` | Yes | The ``string`` to check.

The following example illustrates the ``check`` operation's use:

.. code-block:: xml

    <!-- Check if a given text includes "test" in a case-insensitive manner -->
    <process output="check" handler="RegExpProcessor" operation="check">
        <input name="input">$someTextData</input>
        <!-- Flags are passed in embedded format (e.g. case insensitive match). -->
        <input name="expression">"(?i)test"</input>
    </process>
    <if desc="Optional steps">
        <cond>$check</cond>
        <then>
            ...
        </then>
    </if>

.. _handlers-RegExpProcessor_collect:

RegExpProcessor - collect
^^^^^^^^^^^^^^^^^^^^^^^^^

The ``collect`` operation is used to process a provided text using an expression that defines one or more capturing groups. This
operation can be particularly powerful as it can collect data from both structured and unstructured data. Each matching group
is appended to a ``list`` of ``string`` elements in the sequence with which it was matched, otherwise resulting in an empty ``list``
if no matches were made. The input parameters expected by this operation are as follows:

.. csv-table::
    :header: "Input name", "Required?", "Description"
    :delim: |

    ``expression`` | Yes | A ``string`` with the expression to collect data with. The provided expression must define at least one capturing group.
    ``input`` | Yes | The ``string`` to process to collect data.

The following example illustrates the ``collect`` operation's use:

.. code-block:: xml

    <!-- Define a firstname and lastname in an unstructured text block -->
    <assign to="aText">"My firstname is 'John' and my lastname is 'Doe'."</assign>
    <!-- Collect the data using an expression with two capturing groups -->
    <process output="personData" handler="RegExpProcessor" operation="collect">
        <input name="input">$aText</input>
        <input name="expression">".+ firstname is '([\w]+)' .+ lastname is '([\w]+)'"</input>
    </process>
    <!-- Prints "John" -->
    <log>$personData{0}</log>
    <!-- Prints "Doe" -->
    <log>$personData{1}</log>

.. index:: TemplateProcessor
.. index:: template (TemplateProcessor)
.. index:: parameters (TemplateProcessor)
.. index:: syntax (TemplateProcessor)
.. _handlers-TemplateProcessor:

TemplateProcessor
+++++++++++++++++

Used to generate text data based on a provided template. Using this processing handler instead of the basic 
:ref:`GITB TDL templating capabilities<test-case-expressions-template-files>` permits the decoupling of 
information in the test session context and the template, and also generation of complex content based
on `FreeMarker templates <https://freemarker.apache.org/>`_. This processor should be used for any template-based
data generation that is not limited to simple placeholder replacements.

.. csv-table::
    :header: "Operation", "Description", "Input(s)", "Output(s)"
    :delim: |

    ``process`` | Process the provided template and parameters to produce the output. | Yes | Yes, a ``string`` named ``data`` in the resulting step's ``map``.

The input parameters expected by the ``process`` operation are as follows:

.. csv-table::
    :header: "Input name", "Required?", "Description"
    :delim: |

    ``parameters`` | No | A ``map`` with named inputs to use as the template's input.
    ``syntax`` | No | A ``string`` to specify what syntax the template uses. Accepted values are ``gitb`` (the default) and ``freemarker``.
    ``template`` | Yes | The template content to use (can be of any type that results in a ``string``).

The following example illustrates usage of the ``TemplateProcessor`` to create a message based on a FreeMarker template:

.. code-block:: xml

    <testcase>
        ...
        <imports>
            <artifact name="freemarkerTemplateFile">resources/template.xml</artifact>
        </imports>
        ...
        <steps>
            ...
            <assign to="parameters{value1}">'Value to use'</assign>
            <assign to="parameters{listValues}" append="true">1</assign>
            <assign to="parameters{listValues}" append="true">2</assign>
            <assign to="parameters{listValues}" append="true">3</assign>

            <process output="message" handler="TemplateProcessor">
                <input name="parameters">$parameters</input>
                <input name="template">$freemarkerTemplateFile</input>
                <input name="syntax">'freemarker'</input>
            </process>

            <log>$message</log>
            ...
        </steps>
    </testcase>

In this example the "freemarkerTemplateFile" variable is set via :ref:`import<test-case-imports>` to a template with the following content:

.. code-block:: none

    <?xml version="1.0" encoding="UTF-8"?>
    <data>
        <content>
            <value>${value1}</value>
            <items>
            <#list listValues as listValue>
                <item>${listValue}</item>
            </#list>
            </items>
        </content>
    </data>

.. note::
    **Importing the template as a non-binary variable:** When using the ``TemplateProcessor`` it is important to import the template as a ``binary`` variable
    (or to simply not specify the ``type`` attribute). If another type is used for the import you risk triggering the test engine's 
    :ref:`built-in template processing <test-case-expressions-template-files>` that is not Freemarker-based.

Notice here how the template defines FreeMarker constructs (a list iteration) to go over the items of a collection named "listValues". This was passed in the 
"parameters" ``map`` when calling the :ref:`process step<tdl-step-process>`. When executed, and considering our example, this step will produce data as follows:

.. code-block:: xml

    <?xml version="1.0" encoding="UTF-8"?>
    <data>
        <content>
            <value>Value to use</value>
            <items>
                <item>1</item>
                <item>2</item>
                <item>3</item>
            </items>
        </content>
    </data> 

.. index:: TokenGenerator
.. index:: zone (TokenGenerator)
.. index:: time (TokenGenerator)
.. index:: string (TokenGenerator)
.. index:: timestamp (TokenGenerator)
.. index:: uuid (TokenGenerator)
.. index:: format (TokenGenerator)
.. index:: diff (TokenGenerator)
.. index:: date (TokenGenerator)
.. index:: inputFormat (TokenGenerator)
.. index:: random (TokenGenerator)
.. index:: minimum (TokenGenerator)
.. index:: maximum (TokenGenerator)
.. index:: integer (TokenGenerator)
.. _handlers-TokenGenerator:

TokenGenerator
++++++++++++++

Used to generate tokens that can be used as data in test cases. This processing handler supports but does not require a processing 
transaction to be established. The following operations are supported:

.. csv-table::
    :header: "Operation", "Description", "Input(s)", "Output(s)"

    ``random``, Generate a random integer of double precision number between optional minimum and maximum bounds., Yes, A ``number`` named ``value`` in the resulting step's ``map``.
    ``string``, Generate a text token with potentially fixed and/or random parts to match a provided regular expression., Yes, A ``string`` named ``value`` in the resulting step's ``map``.
    ``timestamp``, Generate a timestamp for the current or a provided time based on a format string., Yes, A ``string`` named ``value`` in the resulting step's ``map``.
    ``uuid``, Generate a random UUID text value matching a Java UUID (e.g. "123e4567-e89b-12d3-a456-556642440000"). This is a value that can be considered as unique for test purposes., No, A ``string`` named ``value`` in the resulting step's ``map``.

A typical use case for the ``TokenGenerator`` is to generate text tokens that can be used in test cases either as input parameters to
e.g. messaging calls (see :ref:`handlers-inputs-outputs`) or as values to replace in loaded text templates (see :ref:`test-case-expressions-template-files`).

.. _handlers-TokenGenerator_random:

TokenGenerator - random
^^^^^^^^^^^^^^^^^^^^^^^

The ``random`` operation is used to generate random numbers that can be used as-is, or help in selecting random elements from a ``list``, or
as part of XPath expressions. The expected inputs are as follows:

.. csv-table::
    :header: "Input name", "Required?", "Description"
    :delim: |

    ``integer`` | No | A ``boolean`` determining whether the produced value will be an integer (when "true") or a double-precision number (when ``false``). By default a double-precision number is produced.
    ``maximum`` | No | A ``number`` representing the maximum bound (exclusive) for the value to produce.
    ``minimum`` | No | A ``number`` representing the minimum bound (inclusive) for the value to produce. If not provided the default considered is zero.

The following example illustrates the operation's usage:

.. code-block:: xml

    <!--
        Generate a random integer between 1 and 10 (exclusive).
    -->
    <process output="numberRandom" handler="TokenGenerator" operation="random">
        <operation>random</operation>
        <input name="minimum">1</input>
        <input name="maximum">10</input>
        <input name="integer">true()</input>
    </process>

.. _handlers-TokenGenerator_string:

TokenGenerator - string
^^^^^^^^^^^^^^^^^^^^^^^

The ``string`` operation can be used to generate any kind of text token with both fixed and random parts following a pattern based on a provided regular expression.
The expected input is as follows:

.. csv-table::
    :header: "Input name", "Required?", "Description"
    :delim: |

    ``format`` | Yes | A regular expression acting as a template to determine the generated token's format.

The following examples illustrate the operation's usage:

.. code-block:: xml

    <!--
        Generate a random string with 2 characters followed by 10 digits.
        Example output would be "cD6723820231".
    -->
    <process output="stringRandom" handler="TokenGenerator" operation="string">
        <input name="format">"[a-zA-Z]{2}\d{10}"</input>
    </process>
    <!--
        Generate a random string:
        - Starting with "PREFIX" and ending with "POSTFIX".
        - With random parts of (a) 5 digits, (b) 5 occurences of 'a', 'b' or 'c', and (c) 2 digits.
        - With hyphens between all fixed and random parts.
        Example output would be "PREFIX-32145-abcaa-02-POSTFIX".
    -->
    <process output="stringRandomAndFixed" handler="TokenGenerator" operation="string">
        <input name="format">"PREFIX-\d{5}-[abc]{5}-\d{2}-POSTFIX"</input>
    </process>

.. _handlers-TokenGenerator_timestamp:

TokenGenerator - timestamp
^^^^^^^^^^^^^^^^^^^^^^^^^^

The ``timestamp`` operation generates a timestamp string that includes date/time values but can also have fixed parts (e.g. if you need to generate
a text token with a fixed part and a variable part based on the current date/time). The inputs expected are as follows:

.. csv-table::
    :header: "Input name", "Required?", "Description"
    :delim: |

    ``date`` |  No | A ``string`` representing a date/time to use as the value to format. If specified along with ``time``, the ``time`` input takes precedence.
    ``diff`` | No | A ``number`` representing the milliseconds to consider as a diff from the considered ``time`` or ``date``. This value (default 0) is added to the considered ``time`` or ``date`` before formatting (i.e. a negative value signals an earlier time).
    ``format`` | No | The formatting pattern to apply provided as a ``string`` matching the Java date/time formatting specifications (see `Formatting configuration`_). If unspecified the current Epoch milliseconds are returned.
    ``inputFormat`` |  No | The formatting pattern to use to interpret the ``date`` input (if provided), matching the Java date/time formatting specifications (see `Formatting configuration`_).
    ``time`` |  No | A ``number`` representing the Epoch milliseconds to use as the date/time to format. If unspecified the current date/time is used.
    ``zone`` | No | The timezone to consider when generating a formatted timestamp provided as a ``string``. Expected values are those defined by Java (see `Timezone codes`_). If unspecified the default consider is ``UTC``.

If a ``date`` is provided without an ``inputFormat``, the pattern of ``dd/MM/yyyy'T'HH:mm:ss.SSSZ`` is assumed by default.
Moreover, all parts are considered optional allowing you to specify only parts of a date, making use of the following defaults
for those that are missing:

* Day of year (``dd``): The 1st day of the month.
* Month (``MM``): The 1st month of the year (January).
* Year (``yyyy``): The current year.
* Time elements (``HH``, ``mm``, ``ss`` and ``SSS``): A value of zero.
* Time zone (``Z``): UTC.

The following examples illustrate the operation's usage:

.. code-block:: xml

    <!--
        Generate a timestamp for the current time without specifying formatting.
        Example output would be "1560238501040".
    -->
    <process output="currentTimestamp" handler="TokenGenerator" operation="timestamp"/>
    <!--
        Generate a timestamp for the current time with provided formatting.
        Example output would be "DATE[2019-05-22] TIME[11:48:06]".
    -->
    <process output="formattedTimestampNow" handler="TokenGenerator" operation="timestamp">
        <input name="format">"'DATE['yyyy-MM-dd'] TIME['HH:mm:ss']'"</input>
    </process>
    <!--
        Generate an XML timestamp for the current time.
    -->
    <process output="currentXmlTimestamp" handler="TokenGenerator" operation="timestamp">
        <input name="format">"yyyy-MM-dd'T'HH:mm:ss.SSSXXX"</input>
    </process>
    <!--
        Generate an XML timestamp for the current time but expressed in the GMT+2 timezone.
    -->
    <process output="currentXmlTimestampInTimezone" handler="TokenGenerator" operation="timestamp">
        <input name="format">"yyyy-MM-dd'T'HH:mm:ss.SSSXXX"</input>
        <input name="zone">"GMT+2"</input>
    </process>
    <!-- 
        Generate a timestamp for the provided time and formatting.
        The output would be "2014-05-11".
    -->
    <process output="formattedTimestamp" handler="TokenGenerator" operation="timestamp">
        <input name="time">'1399792366000'</input>
        <input name="format">"yyyy-MM-dd"</input>
    </process>
    <!-- 
        Generate a timestamp for the current time minus one minute (600000 milliseconds) using the provided formatting.
        Example output would be "2019-06-11 10:23:10".
    -->
    <process output="formattedTimestampMinusOneMinute" handler="TokenGenerator" operation="timestamp">
        <input name="diff">-600000</input>
        <input name="format">"yyyy-MM-dd HH:mm:ss"</input>
    </process>    
    <!-- 
        Obtain the current time (T) and then generate two timestamps:
        - T minus one hour.
        - T plus one hour.
    -->
    <process output="now" handler="TokenGenerator" operation="timestamp"/>
    <process output="nowMinusOneHour" handler="TokenGenerator" operation="timestamp">
        <input name="time">$now</input>
        <input name="diff">-3600000</input>
        <input name="format">"yyyy-MM-dd HH:mm:ss"</input>
    </process>
    <process output="nowPlusOneHour" handler="TokenGenerator" operation="timestamp">
        <input name="time">$now{value}</input>
        <input name="diff">3600000</input>
        <input name="format">"yyyy-MM-dd HH:mm:ss"</input>
    </process>
    <!--
        Generate a timestamp based on an existing date/time string plus one hour
    -->
    <process output="timestampFromFormattedString1" handler="TokenGenerator" operation="timestamp">
        <input name="date">'20-10-2021 13:30:00'</input>
        <input name="inputFormat">'dd-MM-yyyy HH:mm:ss'</input> <!-- Assumes UTC -->
        <input name="diff">3600000</input>
    </process>
    <!--
        Generate a timestamp based on an existing date/time string plus one hour (default formatting)
    -->
    <process output="timestampFromFormattedString2" handler="TokenGenerator" operation="timestamp">
        <input name="date">'20/10'</input> <!-- Assumes the current year, midnight, and a UTC timezone -->
        <input name="diff">3600000</input>
    </process>

.. note:: 
    **Timestamps for use in XML content:** Formatted timestamps generated for use in XML content should match the formatting
    of the ISO 8601 version of the W3C XML Schema dateTime definition. The pattern to apply to get a XSD-valid timestamp is:
    ``yyyy-MM-dd'T'HH:mm:ss.SSSXXX``.

.. _Formatting configuration: https://docs.oracle.com/javase/8/docs/api/java/time/format/DateTimeFormatter.html
.. _Timezone codes: https://docs.oracle.com/javase/8/docs/api/java/time/ZoneId.html#of-java.lang.String-

.. _handlers-TokenGenerator_uuid:

TokenGenerator - uuid
^^^^^^^^^^^^^^^^^^^^^

The ``uuid`` operation provides a random and unique identifier where special formatting is not required (apart from an optional prefix and postfix).
The inputs expected are as follows:

.. csv-table::
    :header: "Input name", "Required?", "Description"
    :delim: |

    ``postfix`` | No | An optional ``string`` to add as a postfix to the generated part of the UUID.
    ``prefix`` | No | An optional ``string`` to add as a prefix to the generated part of the UUID.

The following examples illustrate the operation's usage:

.. code-block:: xml

    <!--
        Generate a UUID (e.g. "971b4df9-4351-4cb8-9ba5-1f6373717ae0").
    -->
    <process output="uuid1" handler="TokenGenerator" operation="uuid"/>
    <!--
        Generate a UUID with a prefix and postfix (e.g. "message-971b4df9-4351-4cb8-9ba5-1f6373717ae0@my.org").
    -->
    <process output="uuid2" handler="TokenGenerator" operation="uuid">
        <input name="prefix">"message-"</input>
        <input name="postfix">"@my.org"</input>
    </process>

.. index:: VariableUtils
.. index:: type (VariableUtils)
.. index:: exists (VariableUtils)
.. index:: name (VariableUtils)
.. _handlers-VariableUtils:

VariableUtils
+++++++++++++

Used for common operations on variables. This processing handler supports but does not require a processing 
transaction to be established. The following operations are supported:

.. csv-table::
    :header: "Operation", "Description", "Input(s)", "Output(s)"

    ``exists``, Check to see if a variable with a given name is defined in the session context., Yes, A ``boolean`` named ``output`` in the resulting step's ``map``.
    ``type``, Return the type of a variable with a given name., Yes, A ``string`` named ``output`` in the resulting step's ``map``. This will be empty if no variable was found for the provided name.

Besides using as part of your testing logic, the ``VariableUtils`` operations can also be quite useful while developing tests to debug
problems related to variables.

.. _handlers-VariableUtils_exists:

VariableUtils - exists
^^^^^^^^^^^^^^^^^^^^^^

The ``exists`` operation is used to check whether a variable exists in the current scope. An example of when this would be
interesting is to check within a :ref:`scriptlet <scriptlets>` whether an :ref:`optional parameter <scriptlets_elements_params>`
has been set. The expected input parameter is as follows:

.. csv-table::
    :header: "Input name", "Required?", "Description"
    :delim: |

    ``name`` | Yes | The name of the variable to check.

The following example illustrates how the ``exists`` operation could be used:

.. code-block:: xml

    <!-- Create a variable named "aVariable". -->
    <assign to="aVariable">"A value"</assign>
    <!-- Check to see if it exists. -->
    <process handler="VariableUtils" operation="exists" input="aVariable" output="variableExists"/>
    <!-- Prints "true". -->
    <log>$variableExists</log>

    <!-- Check to see if a map is defined (e.g. within a scriptlet) and do something with it. -->
    <process handler="VariableUtils" operation="exists" input="optionalMap" output="mapExists"/>
    <if>
        <cond>$mapExists</cond>
        <then>
            <!-- Do something with the map. -->
        </then>
    </if>

.. _handlers-VariableUtils_type:

VariableUtils - type
^^^^^^^^^^^^^^^^^^^^

The ``type`` operation on the other hand could be used in situations when you are not certain of a variable's type. This could
be the case for an element in a ``map`` that can take values of varying types. The expected input parameter is as follows:

.. csv-table::
    :header: "Input name", "Required?", "Description"
    :delim: |

    ``name`` | Yes | The name of the variable to check.

The following example illustrates how the ``type`` operation could be used:

.. code-block:: xml

    <!-- Create a variable named "aVariable". -->
    <assign to="aVariable">"A value"</assign>
    <!-- Check the variable's type. -->
    <process handler="VariableUtils" operation="type" input="aVariable" output="variableType"/>
    <!-- Prints "string". -->
    <log>$variableType</log>

.. index:: XPathProcessor
.. index:: input (XPathProcessor)
.. index:: expression (XPathProcessor)
.. index:: type (XPathProcessor)

.. _handlers-XPathProcessor:

XPathProcessor
++++++++++++++

Used to extract information from XML content using an XPath expression.

.. csv-table::
    :header: "Operation", "Description", "Input(s)", "Output(s)"
    :delim: |

    ``process`` | Process XML content with an XPath expression return the transformed result. | Yes | Yes, a ``string`` named ``output`` in the resulting step's ``map``.

The input parameters expected by the ``process`` operation are as follows:

.. csv-table::
    :header: "Input name", "Required?", "Description"
    :delim: |

    ``expression`` | Yes | The XPath expression to use.
    ``input`` | Yes | The XML content to process.
    ``type`` | No | The :ref:`GITB type <test-case-types>` for the expression's output (default is ``string`` - for XML output use ``object``).

The ``XPathProcessor`` leverages **XML namespaces** as defined by the test case's :ref:`namespaces element <test-case-namespaces>`, allowing
you to construct namespace-aware expressions. The following examples illustrate several data extraction scenarios from a provided XML document.

.. code-block:: xml

    <!-- 
        Define the namespace(s) used at the level of the test case (or scriptlet).
    -->
    <namespaces>
        <ns prefix="po">http://itb.ec.europa.eu/sample/po</ns>
    </namespaces>
    <steps>
        ...
        <!--
            Extract a text value.
        -->
        <process handler="XPathProcessor" output="result1">
            <input name="input">$xmlContent</input>
            <input name="expression">"/po:purchaseOrder/po:shipTo/po:name/text()"</input>
        </process>
        <log>"Extracted string: " || $result1</log>
        <!--
            Extract a value and force it to be considered as a number.
        -->
        <process handler="XPathProcessor" output="result2">
            <input name="input">$xmlContent</input>
            <input name="expression">"/po:purchaseOrder/po:amount/text()"</input>
            <input name="type">"number"</input>
        </process>
        <log>"Extracted number: " || $result2</log>
        <!--
            Extract an XML block for further processing.
        -->
        <process handler="XPathProcessor" output="result3">
            <input name="input">$xmlContent</input>
            <input name="expression">"/po:purchaseOrder/po:shipTo"</input>
            <input name="type">"object"</input>
        </process>
        <log>"Extracted XML block: " || $result3</log>
        <!--
            Extract a text value (alternative using namespace wildcards).
        -->
        <process handler="XPathProcessor" output="result4">
            <input name="input">$xmlContent</input>
            <input name="expression">"/*[local-name() = 'purchaseOrder']/*[local-name() = 'shipTo']/po:name/text()"</input>
        </process>
        <log>"Extracted string: " || $result4</log>
        ...
    </steps>

Using namespaces as illustrated in the above examples is not mandatory. You could also use expressions with **namespace wildcards**,
in which case however you would most probably need to check elements' local names. Using specific namespaces should be preferred 
when possible, as they result in more simple and understandable expressions, but also ensure you are matching precisely what you 
expect. The following examples illustrates a lookup using namespace wildcards:

.. code-block:: xml

    <!-- 
        Use wildcards in place of specific namespaces. Besides being more complex, the expression could have ambiguous
        results in case several namespaces in the XML document define similarly named elements.
    -->
    <process handler="XPathProcessor" output="result">
        <input name="input">$xmlContent</input>
        <input name="expression">"/*[local-name() = 'purchaseOrder']/*[local-name() = 'shipTo']/po:name/text()"</input>
    </process>
    <log>"Extracted string: " || $result</log>

You may have noticed in all examples above that the ``expression`` input is provided as a string and needs to be quoted. This contrasts
:ref:`other steps accepting expressions <test-case-expressions-where>` where XPath expressions can be provided directly. The reason for
this is because you may use the ``XPathProcessor`` also with expressions that are generated, for example by concatenating several strings.
Support for such generated XPath expressions is also what differentiates this handler from using a simple :ref:`assign step <tdl-step-assign>`.
The following examples illustrates how this handler is often equivalent to an :ref:`assign step <tdl-step-assign>`, as well as how it differs
when you need to generate the expression to use:

.. code-block:: xml

    <assign to="country">"BE"</assign>
    <!-- 
        Using the XPath processor to look up a value, with a variable used in a condition.
    -->
    <process handler="XPathProcessor" output="result1">
        <input name="input">$xmlContent</input>
        <input name="expression">"/po:purchaseOrder/po:shipTo[@country = $country]/po:name"</input>
    </process>
    <!--
        Using an assign step that gives the same result as the previous step.
    -->
    <assign to="result2" source="$xmlContent">/po:purchaseOrder/po:shipTo[@country = $country]/po:name</assign>
    <!--
        Using the XPath processor but generating the expression beforehand.
    -->
    <assign to="countryAttributeName">"country"</assign>
    <assign to="expressionToUse">"/po:purchaseOrder/po:shipTo[@" || $countryAttributeName || " = $country]/po:name"</assign>
    <process handler="XPathProcessor" output="result3">
        <input name="input">$xmlContent</input>
        <input name="expression">$expressionToUse</input>
    </process>

    <log>"Results are the same: [" || $result1 || "] [" || $result2 || "] [" || $result3 || "]"</log>

.. note::

    **Using assign for simple extractions:** Using an :ref:`assign step <tdl-step-assign>` is the simplest way to extract content from XML.
    Provide the XML content as the ``source``, the output value as the ``to``, and if needed the specific target type as the ``type``, with
    the XPath expression as the step's value:

    ``<assign to="result" source="$xmlContent">//po:shipTo[@country = $country]/po:name</assign>``

    The ``XPathProcessor`` should be preferred to create expressions dynamically or to reuse expressions across lookups.

.. index:: XsltProcessor
.. index:: XSLTProcessor
.. index:: xml (XsltProcessor)
.. index:: xslt (XsltProcessor)
.. _handlers-XSLTProcessor:

XsltProcessor
+++++++++++++

Used to transform XML content using an XSLT style sheet, both being provided as inputs, and output the result.

.. csv-table::
    :header: "Operation", "Description", "Input(s)", "Output(s)"
    :delim: |

    ``process`` | Process XML content using an XSLT style sheet and return the transformed result. | Yes | Yes, a ``string`` named ``output`` in the resulting step's ``map``.

The input parameters expected by the ``process`` operation are as follows:

.. csv-table::
    :header: "Input name", "Required?", "Description"
    :delim: |

    ``xml`` | Yes | The XML content to transform.
    ``xslt`` | Yes | The XSLT style sheet to use for the transformation.

The following example illustrates usage of the ``XsltProcessor`` to transform the provided "xmlContent" (the XML input) using the "xsltContent" (the XSLT style sheet).
These variables may be provided in any manner, for example the "xmlContent" could be uploaded via a :ref:`user interaction step<tdl-step-receive>` whereas the "xsltContent" could
be :ref:`imported<test-case-imports>` from the test suite's resources.

.. code-block:: xml

    <process output="result" handler="XsltProcessor">
        <input name="xml">$xmlContent</input>
        <input name="xslt">$xsltContent</input>
    </process>
    <log>$result</log>

.. index:: Built-in validation handlers
.. _handlers-predefined-validation-handlers:

Built-in validation handlers
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The title of each section corresponds to the name of the handler that needs to be configured in the relevant :ref:`verify <tdl-step-verify>` step's ``handler`` attribute.

.. index:: ExpressionValidator
.. index:: expression (ExpressionValidator)
.. index:: successMessage (ExpressionValidator)
.. index:: failureMessage (ExpressionValidator)
.. _handlers-ExpressionValidator:

ExpressionValidator
+++++++++++++++++++

Used to verify whether a provided :ref:`expression<test-case-expressions>` evaluates to "true". The 
``ExpressionValidator`` is the most generic validation handler as it can be used to check any arbitrary 
condition.

.. csv-table::
    :header: "Input name", "Required?", "Type", "Description"

    ``expression``, Yes, :ref:`Expression<test-case-expressions>`, The expression to evaluate.
    ``failureMessage``, No, ``string``, An optional message to display in case the check fails.
    ``successMessage``, No, ``string``, An optional message to display in case the check succeeds.

.. code-block:: xml

    <verify handler="ExpressionValidator" desc="Validate UUID">
        <input name="expression">$variable != "unwantedValue"</input>
        <input name="successMessage">'The provided UUID is correct.'</input>
        <input name="failureMessage">'The provided UUID does not match the requirements.'</input>        
    </verify>

.. note::
    The ``expression`` input is not presented in the :ref:`verify step's<tdl-step-verify>` validation report as it
    would only ever display a "true" or "false".

.. index:: JsonValidator
.. index:: json (JsonValidator)
.. index:: schema (JsonValidator)
.. index:: schemaCombinationApproach (JsonValidator)
.. index:: supportYaml (JsonValidator)
.. index:: showSchema (JsonValidator)
.. index:: sharedSchemaPaths (JsonValidator)
.. _handlers-JsonValidator:

JsonValidator
+++++++++++++

Used to validate a JSON document (with support also for YAML) against JSON Schema(s).

.. csv-table::
    :header: "Input name", "Required?", "Type", "Description"
    :delim: ~

    ``json`` ~ Yes ~ ``string`` ~ The JSON content to validate.
    ``schema`` ~ No ~ ``string`` or ``list[string]`` ~ One or more JSON schemas to use for the validation. If no schema is provided the validation will by default succeed.
    ``schemaCombinationApproach`` ~ No ~ ``string`` ~ The combined validation approach when multiple schemas are used. This can be ``allOf`` (the default), ``anyOf``, or ``oneOf``.
    ``showSchema`` ~ No ~ ``boolean`` ~ Whether or not the schema(s) used will be displayed in the resulting step report. By default this is "true".
    ``supportYaml`` ~ No ~ ``boolean`` ~ Whether or not the provided content for the ``json`` input can also be YAML. By default this is "false".

Regarding inputs, if you need to supply a single schema file you don't need to create a ``list`` and pass it as such. You can
simply pass the file as-is and the test engine will automatically convert it to a single-element ``list``.

The following examples illustrate how the ``JsonValidator`` can be used in various scenarios:

.. code-block:: xml

    <!-- 
        Validate JSON content against a single schema.
    -->
    <verify desc="Validate content" handler="JsonValidator">
        <input name="json">$jsonContent</input>
        <input name="schema">$schema</input>
    </verify>
    <!-- 
        Validate JSON content against multiple schemas.
    -->
    <assign to="schemas" append="true">$jsonSchema1</assign>
    <assign to="schemas" append="true">$jsonSchema2</assign>
    <verify desc="Validate content" handler="JsonValidator">
        <input name="json">$jsonContent</input>
        <input name="schema">$schemas</input>
        <!-- This could have been skipped as "allOf" is the default. -->
        <input name="schemaCombinationApproach">"allOf"</input>
    </verify>
    <!--
        Validate content that can be either JSON or YAML against a single schema.
        In addition, don't show the schema that was used for the validation.
    -->
    <verify desc="Validate content" handler="JsonValidator">
        <input name="json">$jsonContent</input>
        <input name="schema">$schema</input>
        <input name="supportYaml">true()</input>
        <input name="showSchema">false()</input>
    </verify>

When validation needs are complex it can be useful to **split schemas into separate ones** and reuse them as needed. In case such schemas
are published online you can reference through their published URI. In case schemas are not published,
the `JSON Schema specification <https://json-schema.org/draft/2020-12/json-schema-core>`__ leaves the handling of local schema references
up to each implementation, which in the case of the Test Bed is addressed as described below.

The first step is to define your schemas as test suite resources. As an example consider a ``PurchaseOrder.schema.json`` that reuses
an ``Address.schema.json`` schema, which could be defined as follows in the test suite:

.. code-block:: none

    testSuite
    ├── schemas
    │   ├── common
    │   │   └── Address.schema.json
    │   └── PurchaseOrder.schema.json
    ├── tests
    │   └── testCase1.xml 
    └── testSuite.xml 

The common schema defines (as all schemas do) its ``$id`` element as a unique identifier. For example, ``Address.schema.json`` could define this
as follows:

.. code-block:: none

    {
        "$id": "http://itb.ec.europa.eu/sample/Address.schema.json",
        ...
    }    

This identifier allows its reference from ``PurchaseOrder.schema.json``, which does so as follows:

.. code-block:: none

    {
        "$id": "http://itb.ec.europa.eu/sample/PurchaseOrder.schema.json",
        ...
        "shipTo": { "$ref": "Address.schema.json" },
        ...
    }

The reference is achieved through the ``$ref`` property which can either be set with an absolute URI (``http://itb.ec.europa.eu/sample/Address.schema.json``),
or a relative one (``Address.schema.json``) as seen above. In the latter case, the full URI will be determined considering the current schema's
``$id`` property. Note that all that matters are the schema ``$id`` properties, not their specific folder placement. In other words, the fact
that ``Address.schema.json`` is under a ``common`` folder does not need to be reflected in the ``$id`` or ``$ref`` properties.

On the side of the test case, you would then need to import the main ``PurchaseOrder.schema.json`` schema but also tell the test engine where
to look for any dependent schemas. To achieve this, you provide the schema to the ``JsonValidator`` not as a ``string`` but as a ``map``
that contains two keys:

* ``schema``, a ``string`` element with the main schema's content.
* ``sharedSchemaPaths``, a ``list`` of ``string`` elements with the **paths**, not the actual contents, of the referred-to schemas.

You may wonder why there is a need for the ``sharedSchemaPaths`` information, as opposed to simply loading schemas from a location
relative to the current one. The reason is that schema URIs (specified via the ``$id`` property) do not represent necessarily 
the physical location of schemas. Making a comparison with XML Schema where namespaces and schema locations are distinct, there
is no such distinction in JSON Schema, at least when working with local schemas.

.. note::
    
    When :ref:`importing schemas from other test suites <test-suite-sharing>`, the ``sharedSchemaPaths`` are resolved with respect to the target test suite.

The following examples cover various scenarios of validating against schemas with dependencies:

.. code-block:: xml

    <testcase>
        ...
        <imports>
            <!-- 
                Import the main schema to use.
            -->
            <artifact name="jsonSchema">schemas/PurchaseOrder.schema.json</artifact>
            ...
        </imports>
        ...
        <steps>
            <!--
                Example with a schema having a single dependent schema.
                Define the schema as a map and under the "schema" key set the schema content
                (imported from the test suite).
            -->
            <assign to="schema{schema}">$jsonSchema</assign>
            <!-- 
                Under "sharedSchemaPaths" define the path from which dependent schemas should be read from.
            -->
            <assign to="schema{sharedSchemaPaths}" append="true">"schemas/common/Address.schema.json"</assign>
            <verify desc="Validate JSON" handler="JsonValidator">
                <input name="json">$jsonContent</input>
                <input name="schema">$schema</input>
            </verify>
            <!--
                Example with a schema having a multiple dependent schemas.
            -->
            <assign to="schemaWithDependencies{schema}">$jsonSchema</assign>
            <assign to="schemaWithDependencies{sharedSchemaPaths}" append="true">"schemas/common/Address.schema.json"</assign>
            <assign to="schemaWithDependencies{sharedSchemaPaths}" append="true">"schemas/common/OrderItem.schema.json"</assign>
            <verify desc="Validate JSON" handler="JsonValidator">
                <input name="json">$jsonContent</input>
                <input name="schema">$schemaWithDependencies</input>
            </verify>
            <!-- 
                Example with multiple schemas, each with dependent schemas.
            -->
            <assign to="example1Schema1{schema}">$schema1</assign>
            <assign to="example1Schema1{sharedSchemaPaths}" append="true">"schemas/common/Address.schema.json"</assign>
            <assign to="example1Schema2{schema}">$schema2</assign>
            <assign to="example1Schema2{sharedSchemaPaths}" append="true">"schemas/common/OrderItem.schema.json"</assign>
            <assign to="example1Schemas" append="true">$example1Schema1</assign>
            <assign to="example1Schemas" append="true">$example1Schema2</assign>
            <verify desc="Validate JSON" handler="JsonValidator">
                <input name="json">$jsonContent</input>
                <input name="schema">$example1Schemas</input>
                <input name="schemaCombinationApproach">"allOf"</input>
            </verify>
            <!-- 
                Example with multiple schemas, only one of which has dependencies.
                For schemas without dependencies you don't need to define them as a map.
            -->
            <assign to="example2Schema1{schema}">$schema1</assign>
            <assign to="example2Schema1{sharedSchemaPaths}" append="true">"schemas/common/Address.schema.json"</assign>
            <assign to="example2Schemas" append="true">$example2Schema1</assign>
            <!-- Add the second schema directly. -->
            <assign to="example2Schemas" append="true">$schema2</assign>
            <verify desc="Validate JSON" handler="JsonValidator">
                <input name="json">$jsonContent</input>
                <input name="schema">$schemasToUse</input>
                <input name="schemaCombinationApproach">"allOf"</input>
            </verify>
            ...
        </steps>
    </testcase>

.. index:: NumberValidator
.. index:: actual (NumberValidator)
.. index:: actualnumber (NumberValidator)
.. index:: expected (NumberValidator)
.. index:: expectednumber (NumberValidator)
.. index:: successMessage (NumberValidator)
.. index:: failureMessage (NumberValidator)
.. _handlers-NumberValidator:

NumberValidator
+++++++++++++++

Used to verify that a provided ``number`` matches an expected value.

.. csv-table::
    :header: "Input name", "Required?", "Type", "Description"

    ``actual``, Yes, ``number``, The value to check.
    ``expected``, Yes, ``number``, The expected value.
    ``failureMessage``, No, ``string``, An optional message to display in case the check fails.
    ``successMessage``, No, ``string``, An optional message to display in case the check succeeds.

.. code-block:: xml

    <verify handler="NumberValidator" desc="Check number">
        <input name="actual">$aNumber</input>
        <input name="expected">'10'</input>
        <input name="successMessage">'The provided value is correct.'</input>
        <input name="failureMessage">'The provided value does not match the requirements.'</input>
    </verify>

.. index:: RegExpValidator
.. index:: input (RegExpValidator)
.. index:: expression (RegExpValidator)
.. index:: successMessage (RegExpValidator)
.. index:: failureMessage (RegExpValidator)
.. _handlers-RegExpValidator:

RegExpValidator
+++++++++++++++

Used to verify that a provided ``string`` matches a regular expression.

.. csv-table::
    :header: "Input name", "Required?", "Type", "Description"

    ``expression``, Yes, ``string``, The expression to match.
    ``failureMessage``, No, ``string``, An optional message to display in case the check fails.
    ``input``, Yes, ``string``, The value to check.
    ``successMessage``, No, ``string``, An optional message to display in case the check succeeds.

.. code-block:: xml

    <verify handler="RegExpValidator" desc="Check string">
        <input name="input">$aString</input>
        <input name="expression">'^REF\-\d+$'</input>
        <input name="successMessage">'The provided value is correct.'</input>
        <input name="failureMessage">'The provided value does not match the requirements.'</input>        
    </verify>

The regular expression provided for the ``expression`` input is expected to be provided using the `syntax used by the Java language`_.
This syntax also supports expression flags provided in an embedded manner, within an expression.

.. code-block:: xml

    <verify handler="RegExpValidator" desc="Check string">
        <input name="input">$aString</input>
        <!-- Same expression but executed in a case insensitive (?i) and multiline (?m) manner. -->
        <input name="expression">'(?im)^REF\-\d+$'</input>
        <input name="successMessage">'The provided value is correct.'</input>
        <input name="failureMessage">'The provided value does not match the requirements.'</input>        
    </verify>

.. index:: SchematronValidator
.. index:: schematron (SchematronValidator)
.. index:: xmldocument (SchematronValidator)
.. index:: xml (SchematronValidator)
.. index:: type (SchematronValidator)
.. index:: showSchematron (SchematronValidator)
.. index:: sortBySeverity (SchematronValidator)
.. index:: showTests (SchematronValidator)
.. _handlers-SchematronValidator:

SchematronValidator
+++++++++++++++++++

.. note::
    
  For Schematron validation consider using the :ref:`handlers-XmlValidator` handler which offering more features and flexibility.

Used to validate an XML document against a Schematron file.

.. csv-table::
    :header: "Input name", "Required?", "Type", "Description"

    ``schematron``, Yes, ``schema``, The Schematron file to use for the validation (XSTL or SCH).
    ``showSchematron``, No, ``boolean``, Whether or not to include in the step's report the Schematron used for the validation (default is "true").
    ``showTests``, No, ``boolean``, Whether or not to include in the step's report the assertion performed for each finding (default is ``false``).
    ``sortBySeverity``, No, ``boolean``, Whether to sort findings by severity ("true") or location in the input (``false`` - the default).
    ``type``, No, ``string``, The type of Schematron file to consider (``xslt`` or ``sch``) in case this cannot be determined from the resource's file suffix. The overall default considered is ``sch``.
    ``xml``, Yes, ``object``, The XML document to validate.

.. note::
    **XSLT vs SCH Schematron files:** XSLT versions of Schematron files are pre-processed files and offer significantly better
    performance for complex rule cases. In addition, if Schematron rules import other resources, use of XSLT files is required.

.. code-block:: xml

    <verify handler="SchematronValidator" desc="Validate content">
        <input name="xml">$docToValidate</input>
        <input name="schematron">$schematronFile</input>
        <!-- Following inputs are optional. -->
        <input name="showSchematron">false()</input>
        <input name="sortBySeverity">true()</input>
        <input name="showTests">true()</input>
    </verify>

.. index:: ShaclValidator
.. index:: contentType (ShaclValidator)
.. index:: loadImports (ShaclValidator)
.. index:: model (ShaclValidator)
.. index:: reportContentType (ShaclValidator)
.. index:: shapes (ShaclValidator)
.. index:: showReport (ShaclValidator)
.. index:: showShapes (ShaclValidator)
.. index:: sortBySeverity (ShaclValidator)
.. _handlers-ShaclValidator:

ShaclValidator
++++++++++++++

Used to validate an RDF model against SHACL shape files.

.. csv-table::
    :header: "Input name", "Required?", "Type", "Description"
    :delim: ~

    ``contentType`` ~ Yes ~ ``string`` ~ The content type of the RDF model provided for validation.
    ``loadImports`` ~ No ~ ``boolean`` ~ Whether or not ``owl:import`` statements found in the input model will be evaluated and included for the validation. By default this is "false".
    ``model`` ~ Yes ~ ``string`` ~ The RDF model to validate.
    ``reportContentType`` ~ No ~ ``string`` ~ The content type to use for the `SHACL validation report <https://www.w3.org/TR/shacl/#validation-report>`__ in case ``showReport`` is set to "true". By default the input model's ``contentType`` will be used.
    ``shapes`` ~ No ~ ``map`` or ``list[map]`` ~ One or more files with SHACL shapes to use for the validation (provided along with their respective content types). If no shape files are provided the validation will by default succeed.
    ``showReport`` ~ No ~ ``boolean`` ~ Whether or not the step's report will display the resulting `SHACL validation report <https://www.w3.org/TR/shacl/#validation-report>`__. By default this is "false".
    ``showShapes`` ~ No ~ ``boolean`` ~ Whether or not the step's report will display the shapes used for the validation (using the first shape file's content type in case of multiple). By default this is "true".
    ``sortBySeverity`` ~ No ~ ``boolean`` ~ Whether or not the resulting report items will be sorted by severity. By default this is "false".

Regarding providing the shape file to consider for the ``shapes`` input, this is provided as a ``map`` containing two keys:

* ``content``, for the shape file content.
* ``contentType``, for the content type to consider when reading the content.

In case multiple shape files are needed, you provide the ``shapes`` input as a ``list`` where each element is a ``map`` corresponding
to a shape file as described above.

The following examples illustrate how the ``ShaclValidator`` can be used in various scenarios:

.. code-block:: xml

    <!--
        Validate an RDF model with single shape file.
    -->
    <assign to="shapeFile{content}">$myShapes1</assign>
    <assign to="shapeFile{contentType}">"text/turtle"</assign>
    <verify desc="Validate model" handler="ShaclValidator">
        <input name="model">$ttlSample</input>
        <input name="shapes">$shapeFile</input>
        <input name="contentType">"text/turtle"</input>
    </verify>
    <!-- 
        Validate an RDF model with multiple shape files.
    -->
    <assign to="shape1{content}">$myShapes1</assign>
    <assign to="shape1{contentType}">"text/turtle"</assign>
    <assign to="shape2{content}">$myShapes2</assign>
    <assign to="shape2{contentType}">"text/turtle"</assign>
    <assign to="shapes" append="true">$shape1</assign>
    <assign to="shapes" append="true">$shape2</assign>
    <verify desc="Validate model" handler="ShaclValidator">
        <input name="model">$ttlSample</input>
        <input name="contentType">"text/turtle"</input>
        <input name="shapes">$shapes</input>
    </verify>
    <!-- 
        Validate an RDF model and for the step's report:
        - Hide the shapes used for the validation.
        - Include the SHACL validation report in RDF/XML format.
        - Sort reported items by severity.
    -->
    <assign to="shapeFileExample{content}">$myShapes1</assign>
    <assign to="shapeFileExample{contentType}">"text/turtle"</assign>
    <verify desc="Validate model" handler="ShaclValidator">
        <input name="model">$ttlSample</input>
        <input name="shapes">$shapeFileExample</input>
        <input name="contentType">"text/turtle"</input>
        <input name="showShapes">false()</input>
        <input name="showReport">false()</input>
        <input name="reportContentType">"application/rdf+xml"</input>
        <input name="sortBySeverity">true()</input>
    </verify>

.. index:: StringValidator
.. index:: actual (StringValidator)
.. index:: expected (StringValidator)
.. index:: actualstring (StringValidator)
.. index:: expectedstring (StringValidator)
.. index:: successMessage (StringValidator)
.. index:: failureMessage (StringValidator)
.. _handlers-StringValidator:

StringValidator
+++++++++++++++

Used to verify that a provided ``string`` matches an expected value.

.. csv-table::
    :header: "Input name", "Required?", "Type", "Description"

    ``actual``, Yes, ``string``, The value to check.
    ``expected``, Yes, ``string``, The expected value.
    ``failureMessage``, No, ``string``, An optional message to display in case the check fails.
    ``successMessage``, No, ``string``, An optional message to display in case the check succeeds.

.. code-block:: xml

    <verify handler="StringValidator" desc="Check string">
        <input name="actual">$aString</input>
        <input name="expected">'expected_string'</input>
        <input name="successMessage">'The provided value is correct.'</input>
        <input name="failureMessage">'The provided value does not match the requirements.'</input>
    </verify>

.. index:: XmlMatchValidator
.. index:: xml (XmlMatchValidator)
.. index:: template (XmlMatchValidator)
.. index:: ignoredPaths (XmlMatchValidator)
.. _handlers-XmlMatchValidator:

XmlMatchValidator
+++++++++++++++++

Used to validate an XML document by matching it against a provided template.

.. csv-table::
    :header: "Input name", "Required?", "Type", "Description"

    ``ignoredPaths``, No, ``list[string]``, An optional list of paths provided as XPath expressions identifying sections of the XML to ignore.
    ``template``, Yes, ``object``, The XML file to consider as the validation's template.
    ``xml``, Yes, ``object``, The XML file to validate.

The matching process takes place by normalising whitespace, ignoring comments and tolerating naming differences in namespace prefixes. In addition,
texts of elements or attributes in the provided ``template`` can be specified with the special value ``?``. This means that any value will be allowed
for this element or attribute and will be ignored as part of the matching (e.g. to ignore random tokens, timestamps, or texts with no expected value).

In case you want to ignore complete XML sections you may use the ``ignoredPaths`` attribute. This allows you to define one or more paths that identify 
elements that will, themselves and for all children, be ignored. For each provided path the following constraints apply:

* It must be formed as a namespace-aware XPath expression considering the namespace prefixes of the provided ``template``.
* It must identify a specific element, rather than a set of elements, a text node or an attribute.
* It must be a simple element-based path with no functions, selectors or wildcards.

The following example illustrates how this validator can be used:

.. code-block:: xml

    <!--
        Validate an XML file based on a provided template.
    -->
    <verify handler="XmlMatchValidator" desc="Validate content">
        <input name="xml">$docToValidate</input>
        <input name="template">$templateFile</input>
    </verify>
    <!--
        Another validation that also defines a set of paths to ignore.
        Variable "pathsToSkip" is of type list[string].
    -->
    <assign to="pathsToSkip" append="true">"/x:Invoice/x:BillingInformation/y:Comments"</assign>
    <verify handler="XmlMatchValidator" desc="Validate content">
        <input name="xml">$docToValidate</input>
        <input name="template">$templateFile</input>
        <input name="ignoredPaths">$pathsToSkip</input>
    </verify>


.. index:: XmlValidator
.. index:: xml (XmlValidator)
.. index:: xsd (XmlValidator)
.. index:: schematron (XmlValidator)
.. index:: schematronType (XmlValidator)
.. index:: stopOnXsdErrors (XmlValidator)
.. index:: sortBySeverity (XmlValidator)
.. index:: showValidationArtefacts (XmlValidator)
.. index:: showSchematronTests (XmlValidator)
.. _handlers-XmlValidator:

XmlValidator
++++++++++++

Used to validate an XML document against an XML Schema (XSD) and/or zero or more Schematron files.

.. csv-table::
    :header: "Input name", "Required?", "Type", "Description"

    ``schematron``, No, ``list[schema]``, The list of Schematron files to validate the document's content against.
    ``schematronType``, No, ``string``, The type of Schematron file to consider (``xslt`` or ``sch``) in case this cannot be determined from the files' suffix. The overall default considered is ``sch``.
    ``showValidationArtefacts``, No, ``boolean``, Whether or not the XSDs and/or Schematrons used for the validation should be included in the step's report (default is "true").
    ``showSchematronTests``, No, ``boolean``, Whether or not the Schematron assertions applied should be displayed for each reported finding (default is ``false``).
    ``sortBySeverity``, No, ``boolean``, Whether findings should be sorted by severity ("true") or by location in the XML content (``false`` - the default).
    ``stopOnXsdErrors``, No, ``boolean``, Whether or not XSD errors should prevent validation from proceeding with Schematron validations (default is "true").
    ``xml``, Yes, ``object``, The XML document to validate.
    ``xsd``, No, ``schema``, The XSD to validate the document's structure against.

Regarding inputs, if you need to supply a single Schematron file you don't need to create a ``list`` and pass it as such. You can
simply pass the Schematron file as-is and the test engine will automatically convert it to a single-element ``list``. Note that considering
that both the ``xsd`` and ``schematron`` inputs are optional, if you provide neither, the validation will simply succeed with an empty report.

The following examples illustrate how the ``XmlValidator`` can be used in various scenarios:

.. code-block:: xml

    <!--
        Validate against an XSD.
    -->
    <verify handler="XmlValidator" desc="XML validation">
        <input name="xml">$content</input>
        <input name="xsd">$schema</input>
    </verify> 
    <!--
        Validate against a single Schematron file.
    -->
    <verify handler="XmlValidator" desc="XML validation">
        <input name="xml">$content</input>
        <input name="schematron">$schematron</input>
    </verify>
    <!--
        Validate against two Schematron files.
    -->
    <assign to="schematrons" append="true">$schematron1</assign>
    <assign to="schematrons" append="true">$schematron2</assign>
    <verify handler="XmlValidator" desc="XML validation">
        <input name="xml">$content</input>
        <input name="schematron">$schematrons</input>
    </verify>
    <!--
        Validate against an XSD and two Schematron files:
        - Without stopping for XSD errors.
        - Sorting findings by severity.
        - Hiding the XSD and Schematrons used.
    -->
    <assign to="schematrons" append="true">$schematron1</assign>
    <assign to="schematrons" append="true">$schematron2</assign>
    <verify handler="XmlValidator" desc="XML validation">
        <input name="xml">$content</input>
        <input name="schematron">$schematrons</input>
        <input name="stopOnXsdErrors">false()</input>
        <input name="sortBySeverity">true()</input>        
        <input name="showValidationArtefacts">false()</input>        
    </verify>

When comparing with the similar :ref:`handlers-XSDValidator` and :ref:`handlers-SchematronValidator`, the  ``XmlValidator`` is more 
flexible and simple to use. In addition, it allows a better fine-tuning of how validation steps are presented. If for example validating
XML content requires validation against an XSD and two Schematron files, using the :ref:`handlers-XSDValidator` and :ref:`handlers-SchematronValidator`
will present three distinct validation steps in the session execution diagram. Using the ``XmlValidator`` you may still display each such
validation separately but you also have the option of making a single validation for all artefacts. Doing so is typically preferred because:

* It presents a **single logical step**, rather than expose the different resources involved.
* It aggregates all findings in a **single report**.

For the sake of comparison, the following examples illustrate how two distinct validations carried out with the :ref:`handlers-XSDValidator` and :ref:`handlers-SchematronValidator`
can be replicated via a single use of the ``XmlValidator``:

.. code-block:: xml

    <!--
        Using the XsdValidator and SchematronValidator.
    -->
    <verify handler="XsdValidator" desc="Validate content">
        <input name="xml">$docToValidate</input>
        <input name="xsd">$schemaFile</input>
    </verify>
    <verify handler="SchematronValidator" desc="Validate content">
        <input name="xml">$docToValidate</input>
        <input name="schematron">$schematronFile</input>
    </verify>
    <!--
        Equivalent validation using the XmlValidator.
    -->
    <verify handler="XmlValidator" desc="Validate content">
        <input name="xml">$docToValidate</input>
        <input name="xsd">$schemaFile</input>
        <input name="schematron">$schematronFile</input>
        <input name="stopOnXsdErrors">false()</input>
    </verify>

.. index:: XPathValidator
.. index:: xml (XPathValidator)
.. index:: expression (XPathValidator)
.. index:: xmldocument (XPathValidator)
.. index:: xpathexpression (XPathValidator)
.. index:: successMessage (StringValidator)
.. index:: failureMessage (StringValidator)
.. _handlers-XPathValidator:

XPathValidator
++++++++++++++

Used to evaluate an XPath 3.0 expression against a provided XML document. The result of the expression
needs to evaluate to a boolean (i.e. true for success or false for failure).

.. csv-table::
    :header: "Input name", "Required?", "Type", "Description"

    ``expression``, Yes, ``string``, The XPath 3.0 expression passed as a string.
    ``failureMessage``, No, ``string``, An optional message to display in case the check fails.
    ``successMessage``, No, ``string``, An optional message to display in case the check succeeds.
    ``xml``, Yes, ``object``, The XML document upon which the XPath expression will be evaluated.

An important note here is that the XPath expression passed in ``expression`` is meant to be a string.
This means that to run an expression as-is you need to wrap it in quotes. This is because the content of
the ``input`` element can also be an expression that you want to evaluate to give you the final expression to
use. The following example illustrates both cases:

.. code-block:: xml

    <!-- 
        Pass a string as the expression to use.
    -->
    <assign to="expectedValue">EXPECTED"</assign>
    <verify handler="XPathValidator" desc="Check document">
        <input name="xml">$myDocument</input>
        <!-- Define expression with variable reference -->
        <input name="expression">"contains(/toc/text(), $expectedValue)"</input>
        <input name="successMessage">"The provided XML is correct."</input>
        <input name="failureMessage">"The provided XML does not match the requirements."</input>
    </verify>
    <!-- 
        Evaluate an expression that will give you the final expression to use.
    -->
    <verify handler="XPathValidator" desc="Check document">
        <input name="xml">$myDocument</input>
        <!-- Define expression without a variable reference using string concatenation -->
        <input name="expression">"contains(/toc/text(), '" || $expectedValue || "')"</input>
        <input name="successMessage">"The provided XML is correct."</input>
        <input name="failureMessage">"The provided XML does not match the requirements."</input>
    </verify>

In the expressions you use for the validations (attribute ``expression``) you may also make use of XML namespaces. Doing so is actually
a best practice to ensure that you don't have ambiguous results due to elements with the same local names. To use namespaces in expressions
you first need to define their prefixes in the test case's :ref:`namespaces section<test-case-namespaces>`. Moreover, keep in mind that the
provided input (attribute ``xml``) also supports expressions with namespaces when determining the XML content to apply the XPath
expression to (if e.g. you want to validate only a part of an XML document).

The following example illustrates how you can use namespace prefixes with your XPath expressions:

.. code-block:: xml

    <testcase>
        <!-- 
            Declare the namespaces to be used.
        -->
        <namespaces>
           <ns prefix="ns1">urn:specification:foo</ns>
           <ns prefix="ns2">urn:specification:bar</ns>
        </namespaces>
        <steps>
            <!--
                Use the defined namespaces.
            -->
            <verify handler="XPathValidator" desc="Check document">
                <input name="xml">$myDocument</input>
                <input name="expression">"/ns1:Foo/ns2:Bar/text() = 'EXPECTED'"</input>
                <input name="successMessage">"The provided XML is correct."</input>
                <input name="failureMessage">"The provided XML does not match the requirements."</input>
            </verify>
        </steps>
    </testcase>

.. index:: XsdValidator
.. index:: xsd (XsdValidator)
.. index:: xml (XsdValidator)
.. index:: XSDValidator
.. index:: xsddocument (XsdValidator)
.. index:: xmldocument (XsdValidator)
.. index:: showSchema (XsdValidator)
.. index:: sortBySeverity (XsdValidator)
.. _handlers-XSDValidator:

XsdValidator
++++++++++++

.. note::
    
  For XSD validation consider using the :ref:`handlers-XmlValidator` handler which offering more features and flexibility.

Used to validate an XML document against an XML Schema (XSD) instance.

.. csv-table::
    :header: "Input name", "Required?", "Type", "Description"

    ``showSchema``, No, ``boolean``, Whether to include in the step's report the XSD used for the validation (default is "true").
    ``sortBySeverity``, No, ``boolean``, Whether to sort findings by severity ("true") or location in the input (``false`` - the default).
    ``xml``, Yes, ``object``, The XML document to validate.
    ``xsd``, Yes, ``schema``, The XSD to validate the document against.

.. code-block:: xml

    <verify handler="XsdValidator" desc="Validate content">
        <input name="xml">$docToValidate</input>
        <input name="xsd">$schemaFile</input>
        <!-- Following inputs are optional. -->
        <input name="showSchema">false()</input>        
        <input name="sortBySeverity">true()</input>        
    </verify>

.. index:: YamlValidator
.. index:: yaml (YamlValidator)
.. index:: schema (YamlValidator)
.. index:: schemaCombinationApproach (YamlValidator)
.. index:: showSchema (YamlValidator)
.. index:: supportJson (YamlValidator)
.. index:: sharedSchemaPaths (YamlValidator)
.. _handlers-YamlValidator:

YamlValidator
+++++++++++++

Used to validate a YAML document (with support also for JSON) against JSON Schema(s).

.. csv-table::
    :header: "Input name", "Required?", "Type", "Description"
    :delim: ~

    ``yaml`` ~ Yes ~ ``string`` ~ The YAML content to validate.
    ``schema`` ~ No ~ ``string`` or ``list[string]`` ~ One or more JSON schemas to use for the validation. If no schema is provided the validation will by default succeed.
    ``schemaCombinationApproach`` ~ No ~ ``string`` ~ The combined validation approach when multiple schemas are used. This can be ``allOf`` (the default), ``anyOf``, or ``oneOf``.
    ``showSchema`` ~ No ~ ``boolean`` ~ Whether or not the schema(s) used will be displayed in the resulting step report. By default this is "true".
    ``supportJson`` ~ No ~ ``boolean`` ~ Whether or not the provided content for the ``yaml`` input can also be JSON. By default this is "false".

Regarding inputs, if you need to supply a single schema file you don't need to create a ``list`` and pass it as such. You can
simply pass the file as-is and the test engine will automatically convert it to a single-element ``list``.

The following examples illustrate how the ``YamlValidator`` can be used in various scenarios:

.. code-block:: xml

    <!-- 
        Validate YAML content against a single schema.
    -->
    <verify desc="Validate content" handler="YamlValidator">
        <input name="yaml">$yamlContent</input>
        <input name="schema">$schema</input>
    </verify>
    <!-- 
        Validate YAML content against multiple schemas.
    -->
    <assign to="schemas" append="true">$jsonSchema1</assign>
    <assign to="schemas" append="true">$jsonSchema2</assign>
    <verify desc="Validate content" handler="YamlValidator">
        <input name="yaml">$yamlContent</input>
        <input name="schema">$schemas</input>
        <!-- This could have been skipped as "allOf" is the default. -->
        <input name="schemaCombinationApproach">"allOf"</input>
    </verify>
    <!--
        Validate content that can be either YAML or JSON against a single schema.
        In addition, don't show the schema that was used for the validation.
    -->
    <verify desc="Validate content" handler="YamlValidator">
        <input name="yaml">$yamlContent</input>
        <input name="schema">$schema</input>
        <input name="supportYaml">true()</input>
        <input name="showSchema">false()</input>
    </verify>

When validation needs are complex it can be useful to **split schemas into separate ones** and reuse them as needed. In case such schemas
are published online you can reference through their published URI. In case schemas are not published,
the `JSON Schema specification <https://json-schema.org/draft/2020-12/json-schema-core>`__ leaves the handling of local schema references
up to each implementation, which in the case of the Test Bed is addressed as described below.

The first step is to define your schemas as test suite resources. As an example consider a ``PurchaseOrder.schema.json`` that reuses
an ``Address.schema.json`` schema, which could be defined as follows in the test suite:

.. code-block:: none

    testSuite
    ├── schemas
    │   ├── common
    │   │   └── Address.schema.json
    │   └── PurchaseOrder.schema.json
    ├── tests
    │   └── testCase1.xml 
    └── testSuite.xml 

The common schema defines (as all schemas do) its ``$id`` element as a unique identifier. For example, ``Address.schema.json`` could define this
as follows:

.. code-block:: none

    {
        "$id": "http://itb.ec.europa.eu/sample/Address.schema.json",
        ...
    }    

This identifier allows its reference from ``PurchaseOrder.schema.json``, which does so as follows:

.. code-block:: none

    {
        "$id": "http://itb.ec.europa.eu/sample/PurchaseOrder.schema.json",
        ...
        "shipTo": { "$ref": "Address.schema.json" },
        ...
    }

The reference is achieved through the ``$ref`` property which can either be set with an absolute URI (``http://itb.ec.europa.eu/sample/Address.schema.json``),
or a relative one (``Address.schema.json``) as seen above. In the latter case, the full URI will be determined considering the current schema's
``$id`` property. Note that all that matters are the schema ``$id`` properties, not their specific folder placement. In other words, the fact
that ``Address.schema.json`` is under a ``common`` folder does not need to be reflected in the ``$id`` or ``$ref`` properties.

On the side of the test case, you would then need to import the main ``PurchaseOrder.schema.json`` schema but also tell the test engine where
to look for any dependent schemas. To achieve this, you provide the schema to the ``JsonValidator`` not as a ``string`` but as a ``map``
that contains two keys:

* ``schema``, a ``string`` element with the main schema's content.
* ``sharedSchemaPaths``, a ``list`` of ``string`` elements with the **paths**, not the actual contents, of the referred-to schemas.

You may wonder why there is a need for the ``sharedSchemaPaths`` information, as opposed to simply loading schemas from a location
relative to the current one. The reason is that schema URIs (specified via the ``$id`` property) do not represent necessarily 
the physical location of schemas. Making a comparison with XML Schema where namespaces and schema locations are distinct, there
is no such distinction in JSON Schema, at least when working with local schemas.

.. note::
    
    When :ref:`importing schemas from other test suites <test-suite-sharing>`, the ``sharedSchemaPaths`` are resolved with respect to the target test suite.

The following examples cover various scenarios of validating against schemas with dependencies:

.. code-block:: xml

    <testcase>
        ...
        <imports>
            <!-- 
                Import the main schema to use.
            -->
            <artifact name="jsonSchema">schemas/PurchaseOrder.schema.json</artifact>
            ...
        </imports>
        ...
        <steps>
            <!--
                Example with a schema having a single dependent schema.
                Define the schema as a map and under the "schema" key set the schema content
                (imported from the test suite).
            -->
            <assign to="schema{schema}">$jsonSchema</assign>
            <!-- 
                Under "sharedSchemaPaths" define the path from which dependent schemas should be read from.
            -->
            <assign to="schema{sharedSchemaPaths}" append="true">"schemas/common/Address.schema.json"</assign>
            <verify desc="Validate YAML" handler="YamlValidator">
                <input name="yaml">$yamlContent</input>
                <input name="schema">$schema</input>
            </verify>
            <!--
                Example with a schema having a multiple dependent schemas.
            -->
            <assign to="schemaWithDependencies{schema}">$jsonSchema</assign>
            <assign to="schemaWithDependencies{sharedSchemaPaths}" append="true">"schemas/common/Address.schema.json"</assign>
            <assign to="schemaWithDependencies{sharedSchemaPaths}" append="true">"schemas/common/OrderItem.schema.json"</assign>
            <verify desc="Validate YAML" handler="YamlValidator">
                <input name="yaml">$yamlContent</input>
                <input name="schema">$schemaWithDependencies</input>
            </verify>
            <!-- 
                Example with multiple schemas, each with dependent schemas.
            -->
            <assign to="example1Schema1{schema}">$schema1</assign>
            <assign to="example1Schema1{sharedSchemaPaths}" append="true">"schemas/common/Address.schema.json"</assign>
            <assign to="example1Schema2{schema}">$schema2</assign>
            <assign to="example1Schema2{sharedSchemaPaths}" append="true">"schemas/common/OrderItem.schema.json"</assign>
            <assign to="example1Schemas" append="true">$example1Schema1</assign>
            <assign to="example1Schemas" append="true">$example1Schema2</assign>
            <verify desc="Validate YAML" handler="YamlValidator">
                <input name="yaml">$yamlContent</input>
                <input name="schema">$example1Schemas</input>
                <input name="schemaCombinationApproach">"allOf"</input>
            </verify>
            <!-- 
                Example with multiple schemas, only one of which has dependencies.
                For schemas without dependencies you don't need to define them as a map.
            -->
            <assign to="example2Schema1{schema}">$schema1</assign>
            <assign to="example2Schema1{sharedSchemaPaths}" append="true">"schemas/common/Address.schema.json"</assign>
            <assign to="example2Schemas" append="true">$example2Schema1</assign>
            <!-- Add the second schema directly. -->
            <assign to="example2Schemas" append="true">$schema2</assign>
            <verify desc="Validate YAML" handler="YamlValidator">
                <input name="yaml">$yamlContent</input>
                <input name="schema">$schemasToUse</input>
                <input name="schemaCombinationApproach">"allOf"</input>
            </verify>
            ...
        </steps>
    </testcase>

Deprecated built-in messaging handlers
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. warning::

    The built-in messaging handlers listed here have been deprecated as they cannot be effectively customised and require the direct exposure of the internal
    test engine to systems under test.

Each following section defines a table with the information expected by each messaging handler. The meaning of this information is as follows:

* **Input:** These are the inputs provided for the send step.
* **Output:** These are the outputs returned from the receive step.
* **Actor configuration:** These are configuration properties that will be automatically set for simulated actors using this handler.
* **Receive configuration:** These are configuration properties expected by the receive step.
* **Send configuration:** These are configuration properties expected by the send step.
* **Transaction configuration:** These are configuration properties defined in the btxn or bptxn step.

The title of each section corresponds to the name of the handler that needs to be configured in the relevant step's ``handler`` attribute (in 
:ref:`send <tdl-step-send>`, :ref:`receive <tdl-step-receive>` or :ref:`btxn <tdl-step-btxn>` steps).

.. index:: HttpMessaging
.. index:: http_headers (HttpMessaging)
.. index:: http_body (HttpMessaging)
.. index:: http_parts (HttpMessaging)
.. index:: http_method (HttpMessaging)
.. index:: http_version (HttpMessaging)
.. index:: http_path (HttpMessaging)
.. index:: http_status (HttpMessaging)
.. index:: network.host (HttpMessaging)
.. index:: network.port (HttpMessaging)
.. index:: http.uri (HttpMessaging)
.. index:: http.uri.extension (HttpMessaging)
.. index:: http.ssl (HttpMessaging)
.. index:: status.code (HttpMessaging)
.. index:: http.method (HttpMessaging)

.. _handlers-httpmessaging:

HttpMessaging
+++++++++++++

.. warning::
  This messaging handler is **deprecated**. Use instead the :ref:`handlers-httpmessagingv2` handler or a 
  :ref:`custom messaging handler<handlers-custom-handlers>` if you have complex messaging needs.

Used to send or receive content over HTTP.

.. csv-table::
    :header: "Element name", "Element type", "Required?", "Type", "Description"

    ``http_body``, Input, No, ``binary``, The HTTP request body's bytes.
    ``http_body``, Output, No, ``binary``, The bytes of the received body.
    ``http_headers``, Input, No, ``map``, The ``map`` of HTTP headers to send.
    ``http_headers``, Output, No, ``map``, The ``map`` of received headers.
    ``http_method``, Output, No, ``string``, The HTTP method.
    ``http_parts``, Input, No, ``map``, A ``map`` including the definition of the parts (see description below).
    ``http_parts``, Output, No, ``map``, A ``map`` including the received parts (see description below).
    ``http_path``, Output, No, ``string``, The HTTP request path.
    ``http_status``, Output, No, ``string``, The HTTP status code from the received response.
    ``http_version``, Input, No, ``string``, The HTTP version to consider.
    ``http_version``, Output, No, ``string``, The HTTP version.
    ``http.method``, Send configuration, Yes, ``string``, The HTTP method to use when sending.
    ``http.ssl``, Transaction configuration, No, ``boolean``, Whether or not connections should be over HTTP (default) or HTTPS.
    ``http.uri``, Actor configuration, No, ``string``, The request path for the request.
    ``http.uri``, Send configuration, No, ``string``, The request path URI to send to.
    ``http.uri.extension``, Send configuration, No, ``string``, HTTP URI extension for the address.
    ``network.host``, Actor configuration, Yes, ``string``, The host of the actor.
    ``network.port``, Actor configuration, Yes, ``number``, The listen port for the actor.
    ``status.code``, Receive configuration, No, ``string``, The status code for responses.
    ``status.code``, Send configuration, No, ``string``, Status for responses.

.. code-block:: xml

    <btxn from="Actor1" to="Actor2" txnId="t1" handler="HttpMessaging"/>
    <send id="dataSend" desc="Send data" from="Actor1" to="Actor2" txnId="t1">
        <config name="http.method">"POST"</input>
        <config name="http.uri">"/path/to/service"</input>
        <input name="http_body">$binaryContent</input>
    </send>
    <receive id="dataReceive" desc="Receive data" from="Actor2" to="Actor1" txnId="t1">
        <config name="status.code">"200"</input>
    </receive>
    <etxn txnId="t1"/>

.. note::
    **Isolating communications:** When using a ``HttpMessaging`` handler to receive communication from a SUT, the Test Bed dynamically starts listening on 
    a new port for incoming traffic. This port (along with the host) are presented to the Test Bed user upon test initiation so that he/she can configure
    the SUT accordingly. To avoid unwanted communication being received on this port that is unrelated to the test session, the Test Bed will only 
    listen to requests originating from the SUT's address, ignoring others originating from other sources. To achieve this, the Test Bed uses the 
    ``network.host`` parameter configured for the SUT that needs to be provided by the tester as part of the SUT's configuration before starting a test.

    The value for the ``network.host`` parameter must be set with the **public IP Address** of the SUT endpoint.

Using HTTPS
^^^^^^^^^^^

The ``HttpMessaging`` handler can be used both for HTTP and (one-way) HTTPS connection. The default setting is connection over HTTP. Switching to 
HTTPS is done at the level of the handler's enclosing transaction and applies to all subsequent :ref:`tdl-step-send` or :ref:`tdl-step-receive` steps. Enabling HTTPS
is achieved by passing a configuration parameter named "http.ssl" with a value of true or false (case insensitive) as part of the begin transaction
step (step :ref:`tdl-step-btxn`). This must be provided at this point because it is needed when creating the sender and receiver implementation.

The following example illustrates its use:

.. code-block:: xml
    :emphasize-lines: 2

    <btxn from="sender" to="receiver" txnId="t1" handler="HttpMessaging">
        <config name="http.ssl">true</config>
    </btxn>
    <send id="dataSend" desc="Send data" from="sender" to="receiver" txnId="t1">
        <config name="http.method">POST</config>
        <input name="http_body">$content</input>
    </send>

Note that the value "true" in this example could also have been provided as a variable reference (e.g. ``$isHTTPS``) allowing a test case to remain unaffected
if the underlying communication needs to be over HTTP or HTTPS. This could be especially interesting in cases where the ``SoapMessaging`` handler is used to 
test SUT endpoints over which the Test Bed has no control over the underlying transport channel. In this case the "http.ssl" parameter could be set as part of 
the system's configuration, as in the following example (assuming an endpoint name of "sutInfo" and an endpoint parameter named "isHTTPS"):

.. code-block:: xml
    :emphasize-lines: 2

    <btxn from="sender" to="receiver" txnId="t1" handler="HttpMessaging">
        <config name="http.ssl">$sutInfo{isHTTPS}</config>
    </btxn>

Support for sending and receiving multipart form data
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

When **receiving**, a multipart message is detected if the ``ContentType`` header contains a boundary part. The ``http_parts`` output is a ``map`` that contains:

    * ``http_parts{parts}``: A list of all parts in sequence.
    * ``http_parts{parts}{0}{header}``: The part's header as a string.
    * ``http_parts{parts}{0}{content}``: The part's content as a binary.
    * ``http_parts{partsByName}``: A map of parts by name (for easy lookup of named parts):
    * ``http_parts{partsByName}{NAME}{header}``: The part's header as a string.
    * ``http_parts{partsByName}{NAME}{content}``: The part's content as a binary.

When **sending**, if a ``http_body`` input is present this takes precedence. If not, and a ``http_parts`` input is provided, then a multipart request is created. The 
``http_parts`` input is a ``list`` of ``maps`` (one map per part). To send a part as a file the ``file_name`` property needs to be passed. Specifically the information 
on a part is as follows:

    * ``http_parts{0}{name}``: The name of the part.
    * ``http_parts{0}{content_type}``: The mime type of the part (text/plain for simple text).
    * ``http_parts{0}{file_name}``: The name of the file to set for the part if this is a file/binary p

The following TDL example illustrates how to populate and send a multipart request with three parts (two file parts and one
test part):

.. code-block:: xml

    <imports>
        <artifact name="file1">testSuite1/artifacts/file1.xml</artifact>
        <artifact name="file2">testSuite1/artifacts/file2.zip</artifact>
    </imports>
    <actors>
        ...
    </actors>
    <steps>
        <!--
            Define first file part.
        -->
        <assign to="filePartInfo1{name}" type="string">"file1"</assign>
        <assign to="filePartInfo1{content_type}" type="string">"text/xml"</assign>
        <assign to="filePartInfo1{file_name}" type="string">"file1.xml"</assign>
        <assign to="filePartInfo1{content}" type="binary">$file1</assign>
        <!--
            Define second file part.
        -->
        <assign to="filePartInfo2{name}" type="string">"file2"</assign>
        <assign to="filePartInfo2{content_type}" type="string">"application/zip"</assign>
        <assign to="filePartInfo2{file_name}" type="string">"file2.zip"</assign>
        <assign to="filePartInfo2{content}" type="binary">$file2</assign>
        <!--
            Define a third text part.
        -->
        <assign to="textPartInfo1{name}" type="string">"text1"</assign>
        <assign to="textPartInfo1{content_type}" type="string">"text/plain"</assign>
        <assign to="textPartInfo1{content}" type="string">"A simple text value"</assign>
        <!--
            Put all parts in a list.
        -->
        <assign to="parts" append="true">$filePartInfo1</assign>
        <assign to="parts" append="true">$filePartInfo2</assign>
        <assign to="parts" append="true">$textPartInfo1</assign>
        <!--
            Send the request.
        -->
        <btxn from="Sender" to="Receiver" txnId="t1" handler="HttpMessaging"/>
        <send desc="Send file" from="Sender" to="Receiver" txnId="t1">
            <config name="http.method">POST</config>
            <input name="http_parts">$parts</input>
        </send>
        <etxn txnId="t1"/>
    </steps>

.. _handlers-httpsmessaging:

HttpsMessaging
++++++++++++++

.. warning::
  This messaging handler is **deprecated**. Use instead the :ref:`handlers-httpmessagingv2` handler or a 
  :ref:`custom messaging handler<handlers-custom-handlers>` if you have complex messaging needs.

Used to send or receive content over HTTPS.

.. csv-table::
    :header: "Element name", "Element type", "Required?", "Type", "Description"

    ``http_body``, Input, No, ``binary``, The HTTP request body's bytes.
    ``http_body``, Output, No, ``binary``, The bytes of the received body.
    ``http_headers``, Input, No, ``map``, The ``map`` of HTTP headers to send.
    ``http_headers``, Output, No, ``map``, The ``map`` of received headers.
    ``http_method``, Output, No, ``string``, The HTTP method.
    ``http_status``, Output, No, ``string``, The HTTP status code from the received response.
    ``http_uri``, Output, No, ``string``, The HTTP request path.
    ``http_version``, Output, No, ``string``, The HTTP version.
    ``http.method``, Send configuration, Yes, ``string``, The HTTP method to use when sending.
    ``http.uri``, Actor configuration, No, ``string``, The request path for the request.
    ``http.uri.extension``, Send configuration, No, ``string``, HTTP URI extension for the address.
    ``network.host``, Actor configuration, Yes, ``string``, The host of the actor.
    ``network.port``, Actor configuration, Yes, ``number``, The listen port for the actor.
    ``status.code``, Receive configuration, No, ``string``, The status code for responses.
    ``status.code``, Send configuration, No, ``string``, Status for responses.

.. code-block:: xml

    <btxn from="Actor1" to="Actor2" txnId="t1" handler="HttpsMessaging"/>
    <send id="dataSend" desc="Send data" from="Actor1" to="Actor2" txnId="t1">
        <config name="http.method">"POST"</input>
        <config name="http.uri.extension">"/path/to/service"</input>
        <input name="http_body">$binaryContent</input>
    </send>
    <receive id="dataReceive" desc="Receive data" from="Actor2" to="Actor1" txnId="t1">
        <config name="status.code">"200"</input>
    </receive>
    <etxn txnId="t1"/>

.. note::
    **Isolating communications:** Handler ``HttpsMessaging`` builds upon the mechanism of :ref:`handlers-httpmessaging` to isolate test 
    session communications when receiving data. Check it's documentation on what is needed to achieve this.

.. index:: HttpProxyMessaging
.. index:: request_data (HttpProxyMessaging)
.. index:: http_method (HttpProxyMessaging)
.. index:: http_version (HttpProxyMessaging)
.. index:: http_path (HttpProxyMessaging)
.. index:: network.host (HttpProxyMessaging)
.. index:: network.port (HttpProxyMessaging)
.. index:: proxy.address (HttpProxyMessaging)

.. _handlers-HttpProxyMessaging:

HttpProxyMessaging
++++++++++++++++++

.. warning::
  This messaging handler is **deprecated**. Use instead the :ref:`handlers-httpmessagingv2` handler or a 
  :ref:`custom messaging handler<handlers-custom-handlers>` if you have complex messaging needs.

Used to proxy HTTP requests and responses between two actors.

.. csv-table::
    :header: "Element name", "Element type", "Required?", "Type", "Description"
    :delim: ~

    ``http_method``~ Output~ No~ ``string``~ The HTTP method.
    ``http_path``~ Output~ No~ ``string``~ The HTTP request path.
    ``http_version``~ Output~ No~ ``string``~ The HTTP version.
    ``network.host``~ Actor configuration~ Yes~ ``string``~ The host of the actor.
    ``network.port``~ Actor configuration~ Yes~ ``number``~ The listen port for the actor.
    ``proxy.address``~ Send configuration~ No~ ``string``~ Address of the proxied service.
    ``request_data``~ Input~ No~ ``map``~ The ``map`` of data to consider. Contains the ``http_method``, ``http_path``, ``http_body``, ``http_headers`` inputs from the HttpMessaging handler.

In this case the ``request_data`` input ``map`` is defined as a convenience considering that we will always be receiving
a call that we want to proxy to a final destination. The HTTP-related parameters to send to the destination need to match
the initial parameters received.

.. code-block:: xml

    <btxn from="Actor1" to="Actor2" txnId="t1" handler="HttpProxyMessaging"/>
    <receive id="receiveData" desc="Receive call" from="Actor1" to="Actor2" txnId="t1" />
    <send desc="Send call" from="Actor2" to="Actor1" txnId="t1">
        <config name="proxy.address">http://PROXIED_SERVICE_ADDRESS</config>
        <input name="request_data" source="$receiveData" />
    </send>
    <etxn txnId="t1"/>

.. index:: SoapMessaging
.. index:: http_headers (SoapMessaging)
.. index:: soap_message (SoapMessaging)
.. index:: soap_attachments (SoapMessaging)
.. index:: soap_header (SoapMessaging)
.. index:: soap_body (SoapMessaging)
.. index:: soap_message (SoapMessaging)
.. index:: soap_content (SoapMessaging)
.. index:: soap_attachments (SoapMessaging)
.. index:: soap_attachments_size (SoapMessaging)
.. index:: network.host (SoapMessaging)
.. index:: network.port (SoapMessaging)
.. index:: http.uri (SoapMessaging)
.. index:: soap.version (SoapMessaging)
.. index:: soap.encoding (SoapMessaging)
.. index:: http.uri.extension (SoapMessaging)
.. index:: http.ssl (SoapMessaging)
.. index:: http_status (SoapMessaging)

.. _handlers-soapmessaging:

SoapMessaging
+++++++++++++

.. warning::
  This messaging handler is **deprecated**. Use instead the :ref:`handlers-soapmessagingv2` handler or a 
  :ref:`custom messaging handler<handlers-custom-handlers>` if you have complex messaging needs.

Used to send or receive payloads via SOAP web service calls.

.. csv-table::
    :header: "Element name", "Element type", "Required?", "Type", "Description"

    ``http_headers``, Input, No, ``map``, A ``map`` of HTTP headers to include.
    ``http_headers``, Output, No, ``map``, The HTTP headers received.
    ``http_status``, Output, No, ``string``, The HTTP status code from the received response.
    ``http.uri``, Actor configuration, No, ``string``, The request path to send the SOAP request to.
    ``http.uri.extension``, Send configuration, No, ``string``, HTTP URI extension for the address.
    ``http.ssl``, Transaction configuration, No, ``boolean``, Whether or not connections should be over HTTP (default) or HTTPS.
    ``network.host``, Actor configuration, Yes, ``string``, The host of the actor.
    ``network.port``, Actor configuration, Yes, ``number``, The listen port for the actor.
    ``soap_attachments``, Input, No, ``map``, A ``map`` of ``binary`` attachments.
    ``soap_attachments``, Output, No, ``map``, A ``map`` of received ``binary`` attachments.
    ``soap_attachments_size``, Output, No, ``number``, The number of attachments received.
    ``soap_body``, Output, Yes, ``object``, The received SOAP body.
    ``soap_message``, Input, Yes, ``object``, The SOAP envelope to send.
    ``soap_header``, Output, Yes, ``object``, The received SOAP header.
    ``soap_message``, Output, Yes, ``object``, The received SOAP envelope.
    ``soap_content``, Output, Yes, ``object``, The XML content of the received SOAP body.
    ``soap.encoding``, Send configuration, No, ``string``, Character set encoding.
    ``soap.version``, Receive configuration, Yes, ``string``, SOAP Version. Can be 1.1 or 1.2.
    ``soap.version``, Send configuration, Yes, ``string``, SOAP Version. Can be 1.1 or 1.2.

.. code-block:: xml

    <btxn from="Actor1" to="Actor2" txnId="t1" handler="SoapMessaging"/>
    <send id="dataSend" desc="Send data" from="Actor1" to="Actor2" txnId="t1">
        <config name="soap.version">1.2</config>
        <input name="soap_message">$soapMessage</input>
    </send>
    <receive id="dataReceive" desc="Receive data" from="Actor2" to="Actor1" txnId="t1">
        <config name="soap.version">1.2</config>
    </receive>
    <etxn txnId="t1"/>

Using HTTPS
^^^^^^^^^^^

The ``SoapMessaging`` handler can be used both over an HTTP and (one-way) HTTPS connection. The default setting is connection over HTTP. Switching to 
HTTPS is done at the level of the handler's enclosing transaction and applies to all subsequent :ref:`tdl-step-send` or :ref:`tdl-step-receive` steps. Enabling HTTPS
is achieved by passing a configuration parameter named "http.ssl" with a value of true or false (case insensitive) as part of the begin transaction
step (step :ref:`tdl-step-btxn`). This must be provided at this point because it is needed when creating the sender and receiver implementation.

The following example illustrates its use:

.. code-block:: xml
    :emphasize-lines: 2

    <btxn from="sender" to="receiver" txnId="t1" handler="SoapMessaging">
        <config name="http.ssl">true</config>
    </btxn>
    <send id="dataSend" desc="Send data" from="sender" to="receiver" txnId="t1">
        <config name="soap.version">$soapVersion</config>
        <input name="soap_message">$soapMessage</input>
    </send>

Note that the value "true" in this example could also have been provided as a variable reference (e.g. ``$isHTTPS``) allowing a test case to remain unaffected
if the underlying communication needs to be over HTTP or HTTPS. This could be especially interesting in cases where the ``SoapMessaging`` handler is used to 
test SUT endpoints over which the Test Bed has no control over the underlying transport channel. In this case the "http.ssl" parameter could be set as part of 
the system's configuration, as in the following example (assuming an endpoint name of "sutInfo" and an endpoint parameter named "isHTTPS"):

.. code-block:: xml
    :emphasize-lines: 2

    <btxn from="sender" to="receiver" txnId="t1" handler="SoapMessaging">
        <config name="http.ssl">$sutInfo{isHTTPS}</config>
    </btxn>

.. note::
    **Isolating communications:** Handler ``HttpsMessaging`` builds upon the mechanism of :ref:`handlers-httpmessaging` to isolate test 
    session communications when receiving data. Check it's documentation on what is needed to achieve this.

.. index:: TCPMessaging
.. index:: content (TCPMessaging)
.. index:: network.host (TCPMessaging)
.. index:: network.port (TCPMessaging)

TCPMessaging
++++++++++++

.. warning::
  This messaging handler is **deprecated**. Define instead a :ref:`custom messaging handler<handlers-custom-handlers>` to cover your test case messaging needs.

Used to send or receive an arbitrary byte stream over TCP.

.. csv-table::
    :header: "Element name", "Element type", "Required?", "Type", "Description"

    ``content``, Input, Yes, ``binary``, The stream of bytes to send.
    ``content``, Output, Yes, ``binary``, The stream of bytes received.
    ``network.host``, Actor configuration, Yes, ``string``, The host of the actor.
    ``network.port``, Actor configuration, Yes, ``number``, The listen port for the actor.

.. code-block:: xml

    <btxn from="Actor1" to="Actor2" txnId="t1" handler="TCPMessaging"/>
    <send id="dataSend" desc="Send data" from="Actor1" to="Actor2" txnId="t1">
        <input name="content">$binaryContent</input>
    </send>
    <receive id="dataReceive" desc="Receive data" from="Actor2" to="Actor1" txnId="t1"/>
    <etxn txnId="t1"/>

.. index:: UDPMessaging
.. index:: content (UDPMessaging)
.. index:: network.host (UDPMessaging)
.. index:: network.port (UDPMessaging)
.. _handlers-udpmessaging:

UDPMessaging
++++++++++++

.. warning::

  This messaging handler is **deprecated**. Define instead a :ref:`custom messaging handler<handlers-custom-handlers>` to cover your test case messaging needs.

Used to send or receive arbitrary bytes over UDP.

.. csv-table::
    :header: "Element name", "Element type", "Required?", "Type", "Description"

    ``content``, Input, Yes, ``binary``, The stream of bytes to send.
    ``content``, Output, Yes, ``binary``, The stream of bytes received.
    ``network.host``, Actor configuration, Yes, ``string``, The host of the actor.
    ``network.port``, Actor configuration, Yes, ``number``, The listen port for the actor.

.. code-block:: xml

    <btxn from="Actor1" to="Actor2" txnId="t1" handler="UDPMessaging"/>
    <send id="dataSend" desc="Send data" from="Actor1" to="Actor2" txnId="t1">
        <input name="content">$binaryContent</input>
    </send>
    <receive id="dataReceive" desc="Receive data" from="Actor2" to="Actor1" txnId="t1"/>
    <etxn txnId="t1"/>

.. index:: Reusable external handlers
.. _handlers-reusable-handlers:

Reusable external handlers
--------------------------

The sections that follow list handler implementations that are maintained by the Test Bed team but that are not :ref:`built-into<handlers-predefined-handlers>`
the test engine itself. In all cases such handlers are defined as **service references** in the test steps that support them, and are available in two ways:

* As a **reusable service** hosted on Test Bed infrastructure that can be used as-is.
* As a **Docker image** registered on the `Docker Hub <https://hub.docker.com/>`_ that can be used to install a local instance of the service.

.. _handlers-reusable-handlers_messaging:

Messaging services
~~~~~~~~~~~~~~~~~~

The following sections summarise reusable messaging services that can be used as handler implementations for :ref:`messaging transactions<tdl-step-btxn>`,
as well as directly in :ref:`send<tdl-step-send>` and :ref:`receive<tdl-step-receive>` steps.

.. index:: AS4 (Reusable messaging services)
.. _handlers-reusable-handlers_messaging_as4:

AS4 messaging
+++++++++++++

The AS4 messaging component allows integration with a `Domibus eDelivery access point <https://ec.europa.eu/digital-building-blocks/wikis/display/DIGITAL/Domibus>`_
to send and receive messages over eDelivery using the AS4 protocol. This component allows you to:

* **Send** a message using a prepared header and payload as inputs.
* **Check** for an acknowledgement received for a given message (previously sent) based on a message identifier.
* **Receive** a message based on an expected message identifier. This will poll the Domibus backend API until the specific message is received.

To use this component, pull the `isaitb/asx-messaging-v4 <https://hub.docker.com/r/isaitb/asx-messaging-v4>`_ Docker image to install it on your
environment. The resulting service must be accessible by the Test Bed, and also have access to the relevant Domibus backend API that will handle the messaging.

To **send** a message use a :ref:`send<tdl-step-send>` step with the following inputs:

* ``as4.send.header``, the AS4 message XML header (of type ``string``, ``object`` or ``binary``).
* ``as4.send.payload``, the list of payloads to send (of type ``list[binary]``).

.. code-block:: xml
    :emphasize-lines: 5-8

    <steps>
       <btxn from="sender" to="received" txnId="t1" handler="http://localhost:8080/ms/api/as4messaging?wsdl"/>
       <assign to="header">$headerToUse</assign>
       <assign to="payloads" append="true">$payloadToInclude</assign>
       <send id="data" desc="Send message" from="sender" to="receiver" txnId="t1">
          <input name="as4.send.header">$header</input>
          <input name="as4.send.payload">$payloads</input>
       </send>
       <!-- Assigned message ID returned as "as4.send.messageId" -->
       <log>$data{as4.send.messageId}</log>
       <!--
          Sent content returned as a map named "sentContent" including entries:
          - "as4.send.header": The header of the message.
          - "as4.send.payload": The list of payloads.
       -->
       <assign to="headerSent">$data{sentContent}{as4.send.header}</assign>
       <assign to="firstPayloadSent">$data{sentContent}{as4.send.payload}{0}</assign>
       <etxn txnId="t1"/>
    </steps>    

To check for a message's **acknowledgement** use a :ref:`receive<tdl-step-receive>` step with the following inputs:

* ``as4.receive.messageId``, the ID of the message to check for (of type ``string``).
* ``as4.receive.type``, set to "ack_check".

.. code-block:: xml
    :emphasize-lines: 9-12

    <steps>
       <btxn from="sender" to="received" txnId="t1" handler="http://localhost:8080/ms/api/as4messaging?wsdl"/>
       ...
       <send id="data" desc="Send message" from="sender" to="receiver" txnId="t1">
          <input name="as4.send.header">$header</input>
          <input name="as4.send.payload">$payloads</input>
       </send>
       ...
       <receive id="ackData" desc="Receive acknowledgement" from="receiver" to="sender" txnId="t1">
          <input name="as4.receive.messageId">$data{as4.send.messageId}</input>
          <input name="as4.receive.type">'ack_check'</input>
       </receive>   
       <!-- 
          The receive step returns a map of two entries:
          - "as4.send.messageId": The message ID that was acknowledged.
          - "as4.message.send.reason": The acknowledgement text.
          Whether the step is a success or failure depends on the state returned and the service's "messaging.ackFailureStates" environment variable.
       -->
       <log>$ackData{as4.send.messageId}</log>
       <log>$ackData{as4.message.send.reason}</log>
       <etxn txnId="t1"/>
    </steps>

To **receive** a message use a :ref:`receive<tdl-step-receive>` step with the following input:

* ``as4.receive.messageId``, the ID of the message to lookup (of type ``string``).

.. code-block:: xml
    :emphasize-lines: 3-5

    <steps>
       <btxn from="sender" to="receiver" txnId="t1" handler="http://localhost:8080/ms/api/as4messaging?wsdl"/>
       <receive id="data" desc="Receive message" from="sender" to="receiver" txnId="t1">
          <input name="as4.receive.messageId">$messageId</input>
       </receive>
       <!-- 
          The receive step returns a map of the following entries:
          - "header": The message header (of type "object").
          - "payload": A map containing the payload data. Each separate payload received is added in a separate map named "payload.N" (where N is a 1-based index). Each of these includes the following entries:
             - "payload.id": The ID of the payload (of type string).
             - "payload.contentType": The payload's content/mime type (of type string).
             - "payload.content": The payload's data (of type binary).
       -->
       <assign to="firstReceivedPayload">$data{payload}{payload.1}{payload.content}</assign>
       <etxn txnId="t1"/> 
    </steps>

.. _handlers-reusable-handlers_processing:

Processing services
~~~~~~~~~~~~~~~~~~~

The following sections summarise reusable processing services that can be used as handler implementations for :ref:`process<tdl-step-process>` steps.

.. index:: ZIP (Reusable processing services)
.. _handlers-reusable-handlers_processing_zip:

ZIP processing
++++++++++++++

The ZIP processing service allows you to extract files from received ZIP archives, or ZIP-like archives such as ASiC containers. Using this service you can:

* Obtain the **table of contents** of a provided archive.
* **Extract** one or more files from the archive based on provided search criteria.

This component functions as a **stateful processing service**, with extraction operations carried out within the scope of a
:ref:`processing transaction<tdl-step-bptxn>`. This allows the ZIP archive to be provided once to the service and then maintained as state across calls to
efficiently carry out several extraction operations. The archive in question is removed once the processing transaction or the overall test session ends.
You can use this component:

* **Locally**, by pulling the `isaitb/zip-processing <https://hub.docker.com/r/isaitb/zip-processing>`_ Docker image.
* **As a service**, by setting your handler to ``https://www.itb.ec.europa.eu/zip/api/processing?wsdl``.

The operations supported by the service are listed in the following table:

.. csv-table::
    :header: "Operation", "Description", "Input(s)", "Output(s)"
    :delim: ~

    ``extract``~ Extract one or more files from the archive. ~ Yes ~ A ``map`` containing three entries (``matched``, a ``boolean`` representing if matches were made; ``entries``, a ``number`` representing the count of entries that were matched; ``entry``, a ``list`` with one item per matched entry). Each item in the ``entry`` list (corresponding to a matched entry) is a ``map`` with two further fields (``path``, a ``string`` with the file's precise path; ``content``, the ``binary`` content of the file).
    ``initialize`` ~ Provide the ZIP archive to the service for subsequent extraction operations. ~ Yes ~ A ``map`` with two elements (``entries``, a ``number`` representing the count of entries included in the archive; ``entryPaths``, a ``string`` including a summary of the included paths, listing them one by one in square brackets).

The input parameters expected by the different operations are as follows:

.. csv-table::
    :header: "Operation", "Input name", "Required?", "Description"
    :delim: ~

    ``extract`` ~ ``case`` ~ No ~ A ``string`` set as "true" or "false" (the default) specifying whether the path matching should be case sensitive.
    ``extract`` ~ ``match`` ~ No ~ A ``string`` set as "exact" or "regexp" (the default) specifying whether the path should be considered for an exact match or as a regular expression.
    ``extract`` ~ ``path`` ~ Yes ~ A ``string`` with the path of the archive's entry (or entries) to return.
    ``initialize`` ~ ``zip`` ~ Yes ~ A ``binary`` input corresponding to the archive to process.

The following test case sample illustrates how to use the service to extract a file from a ZIP archive:

.. code-block:: xml

    <steps>
       <!-- 
          As a first step create processing transaction pointing to the service
       -->
       <bptxn txnId="t1" handler="https://www.itb.ec.europa.eu/zip/api/processing?wsdl"/>
       <!--
          Call the 'initialize' operation to pass the binary archiveContent as an input named 'zip'
       -->
       <process id="toc" txnId="t1" operation="initialize">
          <input name="zip">$archiveContent</input>
       </process>		
       <!--
          Call the 'extract' operation to retrieve a file with an exact but not case-sensitive match
       -->
       <process output="zip" txnId="t1" operation="extract">
          <input name="path">'META-INF/manifest.xml'</input>
          <input name="match">'exact'</input>
          <input name="case">'false'</input>
       </process>
       <!--
          Use if needed the number of returned entries
       -->
       <log>"Extracted " || $zip{entries} || " file(s)"</log>
       <if hidden="true">
          <cond>$zip{matched}</cond>
          <then>
            <!--
              Use the extracted file (first match)
            -->
            <log>"Processing file " || $zip{entry}{0}{path} || "..."</log>
            <assign to="file">$zip{entry}{0}{content}</assign>
          </then>
       </if>
       <assign to="file">$zip{entry}{0}{content}</assign>
       <!--
          Close the processing transaction to release the processed archive.
       -->
       <eptxn txnId="t1"/>
    </steps>

.. _handlers-reusable-handlers_validation:

Validation services
~~~~~~~~~~~~~~~~~~~

The following sections summarise reusable validation services that can be used as handler implementations for :ref:`verify<tdl-step-verify>` steps.

.. index:: ASiC (Reusable validation services)
.. _handlers-reusable-handlers_validation_asic:

ASiC validator
++++++++++++++

The ASiC validation service allows you to validate `ASiC containers <https://en.wikipedia.org/wiki/Associated_Signature_Containers>`_. The validator
currently supports two validation profiles you can consider in your tests:

* **base**: The base ASiC container definition (considered as the default).
* **etendering**: The rules relevant to the `PEPPOL Business Interoperability Specifications (BIS) <https://peppol.org/learn-more/peppol-interoperability-framework>`_.

You can use this component:

* **Locally**, by pulling the `isaitb/asic-validator <https://hub.docker.com/r/isaitb/asic-validator>`_ Docker image.
* **As a service**, by setting your handler to ``https://www.itb.ec.europa.eu/asic/api/validation?wsdl``.

When used in a :ref:`verify<tdl-step-verify>` step this validator expects the following inputs:

.. csv-table::
    :header: "Input name", "Required?", "Description"
    :delim: ~

    ``file`` ~ Yes ~ A ``binary`` input corresponding to the container to validate.
    ``profile`` ~ No ~ A ``string`` identifying the profile to consider that can be "base" (the default) or "etendering".

The following test case sample illustrates how to use the validator to verify an ASiC container using the default profile:

.. code-block:: xml

    <steps>
       <verify id="asicValidation" desc="Validate ASiC container" handler="https://www.itb.ec.europa.eu/asic/api/validation?wsdl">
          <!-- The binary container to validate -->
          <input name="file">$container</name>
          <!-- An optional profile to apply. -->
          <input name="profile">"base"</name>
       </verify>
    </steps>

.. index:: CSV (Reusable validation services)
.. index:: TableSchema (Reusable validation services)
.. _handlers-reusable-handlers_validation_csv:

CSV validator
+++++++++++++

The CSV validation service allows you to validate CSV content by means of one or more `Table Schema <https://specs.frictionlessdata.io/table-schema/>`_
definitions. It is the default, generic configuration of the Test Bed's `CSV validator component <https://hub.docker.com/r/isaitb/csv-validator>`_ that
expects the schemas to apply as inputs alongside the content to validate.

.. note::

    The generic CSV validator is also available for standalone use via `user interface <https://www.itb.ec.europa.eu/csv/any/upload>`__,
    `REST API <https://www.itb.ec.europa.eu/csv/swagger-ui/index.html>`__ and `SOAP API <https://www.itb.ec.europa.eu/csv/soap/any/validation?wsdl>`__.
    Furthermore, a custom validator with a predefinined configuration and specific settings can be defined following the Test Bed's 
    `CSV validation guide <https://www.itb.ec.europa.eu/docs/guides/latest/validatingCSV/index.html>`_. The API of such a custom instance is identical to
    the generic instance presented here.

You can use the CSV validator by one of two approaches:

* **Locally**, by pulling the `isaitb/csv-validator <https://hub.docker.com/r/isaitb/csv-validator>`_ Docker image.
* **As a service**, by setting the handler to ``https://www.itb.ec.europa.eu/csv/soap/any/validation?wsdl``.

The validator supports several inputs to customise the validation to take place. The available inputs are listed in the service's
`SOAP API documentation <https://www.itb.ec.europa.eu/docs/guides/latest/validatingCSV/index.html#validation-via-soap-web-service-api>`__,
where all listed inputs match exactly those that can be used in test cases through :ref:`verify<tdl-step-verify>` steps.

The following test case sample illustrates how to use the validator for the most common use case of validating JSON content against a schema:

.. code-block:: xml

    <steps>
        <!-- 
            You can validate against any number of schemas in one go. In this case we use one schema (defined in $schema)
            that is typically provided as an import but could also be loaded from configuration or even generated on the
            fly in a previous test case step.
         -->
        <assign to="schema1{content}">$schema</assign>
        <!-- Set embeddingMethod to "STRING" if the content is defined as a string ("BASE64" corresponds to binary). -->
        <assign to="schema1{embeddingMethod}">"BASE64"</assign>
        <assign to="schemasToUse" append="true">$schema1</assign>
        <!-- 
            Call the validator.
        -->
        <verify handler="https://www.itb.ec.europa.eu/csv/soap/any/validation?wsdl" desc="Validate CSV file">
            <input name="contentToValidate">$fileToValidate</input>
            <input name="schema">$schemasToUse</input>
            <!-- Set embeddingMethod to "STRING" if the contentToValidate is defined as a string ("BASE64" corresponds to binary). -->
            <input name="embeddingMethod">"BASE64"</input>
        </verify>
    </steps>

.. index:: JSON (Reusable validation services)
.. index:: JSON Schema (Reusable validation services)
.. _handlers-reusable-handlers_validation_json:

JSON validator
++++++++++++++

.. note::
    The built-in :ref:`JsonValidator <handlers-JsonValidator>` allows the validation of JSON without needing an external service.

The JSON validation service allows you to validate JSON content by means of one or more `JSON Schema <https://json-schema.org/>`_
definitions. It is the default, generic configuration of the Test Bed's `JSON validator component <https://hub.docker.com/r/isaitb/json-validator>`_ that
expects the schemas to apply as inputs alongside the content to validate.

.. note::

    The generic JSON validator is also available for standalone use via `user interface <https://www.itb.ec.europa.eu/json/any/upload>`__,
    `REST API <https://www.itb.ec.europa.eu/json/swagger-ui/index.html>`__ and `SOAP API <https://www.itb.ec.europa.eu/json/soap/any/validation?wsdl>`__.
    Furthermore, a custom validator with a predefinined configuration and specific settings can be defined following the Test Bed's 
    `JSON validation guide <https://www.itb.ec.europa.eu/docs/guides/latest/validatingJSON/index.html>`_. The API of such a custom instance is identical to
    the generic instance presented here.

You can use the JSON validator by one of two approaches:

* **Locally**, by pulling the `isaitb/json-validator <https://hub.docker.com/r/isaitb/json-validator>`_ Docker image.
* **As a service**, by setting the handler to ``https://www.itb.ec.europa.eu/json/soap/any/validation?wsdl``.

The validator supports several inputs to customise the validation to take place. The available inputs are listed in the service's
`SOAP API documentation <https://www.itb.ec.europa.eu/docs/guides/latest/validatingJSON/index.html#validation-via-soap-web-service-api>`__,
where all listed inputs match exactly those that can be used in test cases through :ref:`verify<tdl-step-verify>` steps.

The following test case sample illustrates how to use the validator for the most common use case of validating JSON content against a schema:

.. code-block:: xml

    <steps>
        <!-- 
            You can validate against any number of schemas in one go. In this case we use one schema (defined in $schema)
            that is typically provided as an import but could also be loaded from configuration or even generated on the
            fly in a previous test case step.
         -->
        <assign to="schema1{schema}">$schema</assign>
        <!-- Set embeddingMethod to "STRING" if the content is defined as a string ("BASE64" corresponds to binary). -->
        <assign to="schema1{embeddingMethod}">"BASE64"</assign>
        <assign to="schemasToUse" append="true">$schema1</assign>
        <!-- 
            Call the validator.
        -->
        <verify handler="https://www.itb.ec.europa.eu/json/soap/any/validation?wsdl" desc="Validate JSON file">
            <input name="contentToValidate">$fileToValidate</input>
            <input name="externalSchemas">$schemasToUse</input>
            <!-- Set embeddingMethod to "STRING" if the contentToValidate is defined as a string ("BASE64" corresponds to binary). -->
            <input name="embeddingMethod">"BASE64"</input>
        </verify>
    </steps>

.. index:: RDF (Reusable validation services)
.. index:: SHACL (Reusable validation services)
.. _handlers-reusable-handlers_validation_rdf:

RDF validator
+++++++++++++

.. note::
    The built-in :ref:`ShaclValidator <handlers-ShaclValidator>` allows validating RDF data without needing an external service.

The RDF validation service allows you to validate RDF content via `SHACL shapes <https://www.w3.org/TR/shacl/>`_
definitions. It is the default, generic configuration of the Test Bed's `RDF validator component <https://hub.docker.com/r/isaitb/shacl-validator>`_ that
expects the shapes to apply as inputs alongside the content to validate.

.. note::

    The generic RDF validator is also available for standalone use via `user interface <https://www.itb.ec.europa.eu/shacl/any/upload>`__,
    `REST API <https://www.itb.ec.europa.eu/shacl/swagger-ui/index.html>`__ and `SOAP API <https://www.itb.ec.europa.eu/shacl/soap/any/validation?wsdl>`__.
    Furthermore, a custom validator with a predefinined configuration and specific settings can be defined following the Test Bed's 
    `RDF validation guide <https://www.itb.ec.europa.eu/docs/guides/latest/validatingRDF/index.html>`_. The API of such a custom instance is identical to
    the generic instance presented here.

You can use the RDF validator by one of two approaches:

* **Locally**, by pulling the `isaitb/shacl-validator <https://hub.docker.com/r/isaitb/shacl-validator>`_ Docker image.
* **As a service**, by setting the handler to ``https://www.itb.ec.europa.eu/shacl/soap/any/validation?wsdl``.

The validator supports several inputs to customise the validation to take place. The available inputs are listed in the service's
`SOAP API documentation <https://www.itb.ec.europa.eu/docs/guides/latest/validatingRDF/index.html#validation-via-soap-web-service-api>`__,
where all listed inputs match exactly those that can be used in test cases through :ref:`verify<tdl-step-verify>` steps.

The following test case sample illustrates how to use the validator for the most common use case of validating RDF content against a shape graph:

.. code-block:: xml

    <steps>
        <!-- 
            You can validate against any number of shape graph files in one go. In this case we use one file (defined in $shapes)
            that is typically provided as an import but could also be loaded from configuration or even generated on the
            fly in a previous test case step.
         -->
        <assign to="shapes1{ruleSet}">$shapes</assign>
        <assign to="shapes1{ruleSyntax}">"application/turtle"</assign>
        <!-- Set embeddingMethod to "STRING" if the content is defined as a string ("BASE64" corresponds to binary). -->
        <assign to="shapes1{embeddingMethod}">"BASE64"</assign>
        <assign to="shapesToUse" append="true">$shapes1</assign>
        <!-- 
            Call the validator.
        -->
        <verify handler="https://www.itb.ec.europa.eu/shacl/soap/any/validation?wsdl" desc="Validate RDF file">
            <input name="contentToValidate">$fileToValidate</input>
            <input name="contentSyntax">"application/rdf+xml"</input>
            <input name="externalRules">$shapesToUse</input>
            <!-- Set embeddingMethod to "STRING" if the contentToValidate is defined as a string ("BASE64" corresponds to binary). -->
            <input name="embeddingMethod">"BASE64"</input>
        </verify>
    </steps>

.. index:: XML (Reusable validation services)
.. index:: XML Schema (Reusable validation services)
.. index:: Schematron (Reusable validation services)
.. _handlers-reusable-handlers_validation_xml:

XML validator
+++++++++++++

.. note::
    Built-in validators for XML are also available, notably the :ref:`handlers-XmlValidator` for validation against XML Schema and Schematron, as
    well as the :ref:`handlers-XmlMatchValidator` for validation against expected templates.

The XML validation service allows you to validate XML content by means of one or more `XML Schemas <https://www.w3.org/standards/xml/schema>`_ 
and `Schematron <https://schematron.com/>`_. It is the default, generic configuration of the Test Bed's 
`XML validator component <https://hub.docker.com/r/isaitb/xml-validator>`_ that expects the schemas and Schematrons to apply as inputs alongside
the content to validate.

.. note::

    The generic XML validator is also available for standalone use via `user interface <https://www.itb.ec.europa.eu/xml/upload>`__,
    `REST API <https://www.itb.ec.europa.eu/xml/swagger-ui/index.html>`__ and `SOAP API <https://www.itb.ec.europa.eu/xml/api/validation?wsdl>`__.
    Furthermore, a custom validator with a predefinined configuration and specific settings can be defined following the Test Bed's 
    `XML validation guide <https://www.itb.ec.europa.eu/docs/guides/latest/validatingXML/index.html>`_. The API of such a custom instance is identical to
    the generic instance presented here.

You can use the XML validator by one of two approaches:

* **Locally**, by pulling the `isaitb/xml-validator <https://hub.docker.com/r/isaitb/xml-validator>`_ Docker image.
* **As a service**, by setting the handler to ``https://www.itb.ec.europa.eu/xml/api/validation?wsdl``.

The validator supports several inputs to customise the validation to take place. The available inputs are listed in the service's
`SOAP API documentation <https://www.itb.ec.europa.eu/docs/guides/latest/validatingXML/index.html#validation-via-soap-web-service-api>`__,
where all listed inputs match exactly those that can be used in test cases through :ref:`verify<tdl-step-verify>` steps.

The following test case sample illustrates how to use the validator for the most common use case of validating XML content against an XML Schema and a
Schematron rule file:

.. code-block:: xml

    <steps>
        <!-- 
            You can validate against any number of schemas in one go. In this case we use one schema (defined in $schema)
            that is typically provided as an import but could also be loaded from configuration or even generated on the
            fly in a previous test case step.
         -->
        <assign to="schema1{content}">$schema</assign>
        <!-- Set embeddingMethod to "STRING" if the content is defined as a string ("BASE64" corresponds to binary). -->
        <assign to="schema1{embeddingMethod}">"BASE64"</assign>
        <assign to="schemasToUse" append="true">$schema1</assign>
        <!-- 
            Prepare also the Schematron rules to use (defined in $schematron) that could similarly be imported, loaded from
            configuration or generated on the fly.
        -->
        <assign to="schematron1{content}">$schematron</assign>
        <!-- 
            The Schematron type could be "xsl" (for Schematron transformed to XSLT), or "sch" for the raw Schematron format. Besides
            experimentation or very simple cases, XSLT rules (so a "xsl" type value) should always be preferred.
        -->
        <assign to="schematron1{type}">"xsl"</assign>
        <!-- Set embeddingMethod to "STRING" if the content is defined as a string ("BASE64" corresponds to binary). -->
        <assign to="schematron1{embeddingMethod}">"BASE64"</assign>
        <assign to="schematronsToUse" append="true">$schematron1</assign>
        <!-- 
            Call the validator.
        -->
        <verify handler="https://www.itb.ec.europa.eu/xml/api/validation?wsdl" desc="Validate XML file">
            <input name="xml">$fileToValidate</input>
            <input name="externalSchema">$schemasToUse</input>
            <input name="externalSchematron">$schematronsToUse</input>
            <!-- Set embeddingMethod to "STRING" if the contentToValidate is defined as a string ("BASE64" corresponds to binary). -->
            <input name="embeddingMethod">"BASE64"</input>
        </verify>
    </steps>

.. note::
    When using the generic XML validator you don't need to always validate using XML Schema and Schematron. For example you could skip schema
    validation and only validate against a set of generated Schematron rules.

.. index:: YAML (Reusable validation services)
.. index:: JSON Schema (Reusable validation services - YAML validator)
.. _handlers-reusable-handlers_validation_yaml:

YAML validator
++++++++++++++

.. note::
    The built-in :ref:`YamlValidator <handlers-YamlValidator>` allows the validation of YAML without needing an external service.

The YAML validation service allows you to validate YAML content by means of one or more `JSON Schema <https://json-schema.org/>`_
definitions. It is the default, generic configuration of the Test Bed's `JSON validator component <https://hub.docker.com/r/isaitb/json-validator>`_ 
that is configured to force the input of YAML content instead of JSON, and that expects the schemas to apply as inputs alongside
the content to validate.

.. note::

    The generic YAML validator is also available for standalone use via `user interface <https://www.itb.ec.europa.eu/json/yaml/upload>`__,
    `REST API <https://www.itb.ec.europa.eu/json/swagger-ui/index.html>`__ and `SOAP API <https://www.itb.ec.europa.eu/json/soap/yaml/validation?wsdl>`__.
    Furthermore, a custom validator with a predefinined configuration and specific settings can be defined following the Test Bed's 
    `YAML validation guide <https://www.itb.ec.europa.eu/docs/guides/latest/validatingYAML/index.html>`_. The API of such a custom instance is identical to
    the generic instance presented here.

You can use the YAML validator by one of two approaches:

* **Locally**, by pulling the `isaitb/json-validator <https://hub.docker.com/r/isaitb/json-validator>`_ Docker image and
  `configuring it for YAML usage <https://www.itb.ec.europa.eu/docs/guides/latest/validatingYAML/>`__.
* **As a service**, by setting the handler to ``https://www.itb.ec.europa.eu/json/soap/yaml/validation?wsdl``.

The validator supports several inputs to customise the validation to take place. The available inputs are listed in the service's
`SOAP API documentation <https://www.itb.ec.europa.eu/docs/guides/latest/validatingJSON/index.html#validation-via-soap-web-service-api>`__,
where all listed inputs match exactly those that can be used in test cases through :ref:`verify<tdl-step-verify>` steps.

The following test case sample illustrates how to use the validator for the most common use case of validating YAML content against a schema:

.. code-block:: xml

    <steps>
        <!-- 
            You can validate against any number of schemas in one go. In this case we use one schema (defined in $schema)
            that is typically provided as an import but could also be loaded from configuration or even generated on the
            fly in a previous test case step.
         -->
        <assign to="schema1{schema}">$schema</assign>
        <!-- Set embeddingMethod to "STRING" if the content is defined as a string ("BASE64" corresponds to binary). -->
        <assign to="schema1{embeddingMethod}">"BASE64"</assign>
        <assign to="schemasToUse" append="true">$schema1</assign>
        <!-- 
            Call the validator.
        -->
        <verify handler="https://www.itb.ec.europa.eu/json/soap/yaml/validation?wsdl" desc="Validate YAML file">
            <input name="contentToValidate">$fileToValidate</input>
            <input name="externalSchemas">$schemasToUse</input>
            <!-- Set embeddingMethod to "STRING" if the contentToValidate is defined as a string ("BASE64" corresponds to binary). -->
            <input name="embeddingMethod">"BASE64"</input>
        </verify>
    </steps>

.. index:: Custom external handlers
.. _handlers-custom-handlers:

Custom external handlers
------------------------

Custom service handlers are meaningful when you have project-specific testing needs that cannot be addressed by the test engine's 
:ref:`built-in capabilities<handlers-predefined-handlers>` or the :ref:`existing reusable services<handlers-reusable-handlers>` offered by the Test Bed.
In practice any non-trivial test setup would usually require at least a **custom messaging service implementation** to handle the messaging protocol foreseen
by the project. This holds true even if a seemingly suitable built-in messaging handler is available, as you will most likely need to add customisations
when making or receiving calls, but also adapt the reports produced by your messaging steps.

Custom service handler implementations would be defined in a **custom web application** that complements your test suites. This application would include
implementations (as needed) of the **GITB test service SOAP APIs** that allow it to be orchestrated by the Test Bed. To guide you in the implementation of
these APIs you can refer to the `GITB test services documentation <https://www.itb.ec.europa.eu/docs/services/latest/index.html>`_ for:

* `Validation services <https://www.itb.ec.europa.eu/docs/services/latest/validation/index.html>`_, to validate content.
* `Messaging services <https://www.itb.ec.europa.eu/docs/services/latest/messaging/index.html>`_, to send and receive messages.
* `Processing services <https://www.itb.ec.europa.eu/docs/services/latest/processing/index.html>`_, to implement supporting utility functions.

The starting point for the implementation is the Test Bed's `published template service <https://www.itb.ec.europa.eu/docs/services/latest/templates/index.html>`_.
This is an **executable** template, allowing you to create new services based on existing demo starting implementations. Although simple, the pre-existing
implementations fully cover the GITB service APIs and allow you to replace them with your own logic. Moreover, the documentation also includes a
`sample test case <https://www.itb.ec.europa.eu/docs/services/latest/templates/index.html#example-test-case>`_ that illustrates how the demo service
implementations can be used in test steps. For a guided, **step-by-step tutorial** on how to develop custom test services you can also check out the
`complex test development guide <https://www.itb.ec.europa.eu/docs/guides/latest/developingComplexTests/index.html>`_.

.. index:: Handler authentication
.. index:: HTTP Basic
.. index:: UsernameToken
.. index:: WS-Security
.. index:: auth.basic.username
.. index:: auth.basic.password
.. index:: auth.token.username
.. index:: auth.token.password
.. index:: auth.token.password.type
.. _handlers-authentication:

Authentication for external handlers
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Handlers defined as external service implementations may need to be protected with access control. To support such protected services,
the GITB software foresees the possibility to authenticate as part of each service call. Authentication information needs to be configured
before any exchanges take place with the service and as such, cannot use the ``config`` and ``input`` elements otherwise used to pass information. 
Authentication configuration is handled with ``property`` elements that are used as part of the handler setup in:

* The :ref:`tdl-step-btxn` step for transactional messaging services.
* The :ref:`tdl-step-send`, :ref:`tdl-step-receive` and :ref:`tdl-step-listen` step for non-transactional messaging services.
* The :ref:`tdl-step-bptxn` step for transactional processing services.
* The :ref:`tdl-step-process` step for non-transactional processing services.
* The :ref:`tdl-step-verify` step for validation services.

The authentication possibilities currently supported are:

* **Basic HTTP authentication** for all calls to the service's HTTP/HTTPS endpoint. This is authentication at the transport layer.
* Authentication using the **WS-Security UsernameToken profile** (see `here`_), supporting text and digest password transmission with timestamps and nonces. This is authentication at the SOAP application layer.

.. _here: https://www.oasis-open.org/committees/download.php/13392/wss-v1.1-spec-pr-UsernameTokenProfile-01.htm

The properties that are supported in the ``property`` elements are listed in the following table:

.. csv-table::
    :header: "Property name", "Value", "Description"

    ``auth.basic.password``, Any ``string``, The password to provide when prompted for basic HTTP authentication.
    ``auth.basic.username``, Any ``string``, The username to provide when prompted for basic HTTP authentication.
    ``auth.token.password``, Any ``string``, The password to include in the SOAP header as the UsernameToken's password.
    ``auth.token.password.type``, 'DIGEST' (the default) or 'TEXT', The way the password is to be serialised in the header. 'DIGEST' includes it as a DIGEST whereas 'TEXT' adds it in plaintext.
    ``auth.token.username``, Any ``string``, The username to include in the SOAP header as the UsernameToken's username.

Note that use of HTTP basic authentication and the UsernameToken are not necessarily exclusive. A case where both are provided would be
where a service protects access to its WSDL using HTTP basic authentication and adds additional protection for SOAP service calls by means
of a UsernameToken. Combining both approaches is rare but possible. The following example illustrates use of these authentication properties
calling various test services:

.. code-block:: xml

    <!--
        Transactional messaging service authentication with UsernameToken (DIGEST).
    -->
    <btxn from="Sender" to="Receiver1" txnId="t1" handler="$messagingServiceURL">
        <property name="auth.token.username">$DOMAIN{serviceUsername1}</property>
        <property name="auth.token.password">$DOMAIN{servicePassword1}</property>
        <property name="auth.token.password.type">DIGEST</property>
    </btxn>
    <send id="dataSend" desc="Send message" from="Sender" to="Receiver1" txnId="t1"/>
    <etxn txnId="t1"/>
    <!--
        Validation service authentication with UsernameToken (DIGEST - the default) and HTTP basic authentication.
    -->
    <verify handler="$validationService1" desc="Validate content">
        <property name="auth.basic.username">$DOMAIN{serviceUsername2}</property>
        <property name="auth.basic.password">$DOMAIN{servicePassword2}</property>
        <property name="auth.token.username">$DOMAIN{serviceUsername3}</property>
        <property name="auth.token.password">$DOMAIN{servicePassword3}</property>
        <input name="content">$contentToValidate</input>
    </verify>
    <!--
        Transactional processing service authentication with HTTP basic authentication.
    -->
    <bptxn txnId="t1" handler="$processingServiceURL">
        <property name="auth.basic.username">$DOMAIN{serviceUsername4}</property>
        <property name="auth.basic.password">$DOMAIN{servicePassword4}</property>
    </bptxn>
    <process id="result" txnId="t1">
        <operation>action</operation>
        <input name="anInput">$aValue</input>
    </process>
    <eptxn txnId="t1"/>
    <!--
        Non-transactional processing service authentication with HTTP basic authentication.
    -->
    <process id="result" handler="$otherProcessingServiceURL">
        <property name="auth.basic.username">$DOMAIN{serviceUsername5}</property>
        <property name="auth.basic.password">$DOMAIN{servicePassword5}</property>
        <operation>action</operation>
        <input name="anInput">$aValue</input>
    </process>
    <!--
        Validation service authentication with UsernameToken (TEXT) authentication.
    -->
    <verify handler="$validationService2" desc="Validate content">
        <property name="auth.token.username">$DOMAIN{serviceUsername6}</property>
        <property name="auth.token.password">$DOMAIN{servicePassword6}</property>
        <property name="auth.token.password.type">TEXT</property>
        <input name="content">$contentToValidate</input>
    </verify>
    <!--
        Non-transactional messaging service with UsernameToken (TEXT) authentication.
    -->
    <send id="dataSend" desc="Send message" from="Sender" to="Receiver" handler="$messagingServiceURL">
        <property name="auth.token.username">$DOMAIN{serviceUsername7}</property>
        <property name="auth.token.password">$DOMAIN{servicePassword7}</property>
        <property name="auth.token.password.type">TEXT</property>
        <input name="message">$messageToSend</input>
    </send>
    <etxn txnId="t1"/>

.. index:: Handler inputs and outputs
.. index:: name (Handler inputs and outputs)
.. index:: lang (Handler inputs and outputs)
.. index:: source (Handler inputs and outputs)
.. index:: asTemplate (Handler inputs and outputs)
.. _handlers-inputs-outputs:

Handler inputs and outputs
--------------------------

The ``input`` and ``output`` elements used with handlers are what GITB refers to as "Binding elements".
They share the following structure:

.. csv-table::
    :header: "Name", "Required?", "Description"

    ``@asTemplate``, no, Whether or not the result will be considered as a template for placeholder replacement (see :ref:`test-case-expressions-template-files`). By default this is "false".
    ``@name``, no, The name of the input or output element.
    ``@lang``, no, The expression language that should be considered when evaluating its contained expression (see :ref:`test-case-expressions`).
    ``@source``, no, A pure variable reference identifying a source variable. Used as the target upon which to evaluate the contained expression.

The text content of the element is considered to be an expression (see :ref:`test-case-expressions`). In the case a ``source`` attribute is provided
the contained expression is evaluated on the variable identified by ``source`` to produce the value. If no ``source`` attribute is present the value
is the result of the expression itself. For inputs of type ``object`` or ``schema`` (i.e. XML documents) the ``source`` attribute can also be used to pass
the complete document as the value. In this case use of the ``source`` attribute to reference the relevant variable is equivalent to specifying its
reference as the expression:

.. code-block:: xml

    <verify handler="XmlValidator" desc="Validate content">
        <!--
            Pass document through the expression.
        -->
        <input name="xml">$docToValidate</input>
        <!--
            Pass document through the source attribute.
        -->
        <input name="schematron" source="$schematronFile"/>
    </verify>

.. note::
    **Specifying a fixed value:** Considering that the default expression language is XPath, a fixed text value is provided by enclosing it in
    quotes. See :ref:`test-case-expressions` for further details.

The ``input`` and ``output`` options for service handlers are documented as part of their module definition. For handlers accessible
via remote web service calls this information is returned when calling the handler's ``getModuleDefinition`` operation. This is also used internally
by the Test Bed before calling a service handler to ensure that required parameters are provided by the test case.

.. _syntax used by the Java language: https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/regex/Pattern.html