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
.. index:: HttpMessagingV2 (send - showRequestUri)
.. index:: HttpMessagingV2 (send - showRequestMethod)
.. index:: HttpMessagingV2 (send - showRequestBody)
.. index:: HttpMessagingV2 (send - showRequestHeaders)
.. index:: HttpMessagingV2 (send - requestHeadersToShow)
.. index:: HttpMessagingV2 (send - requestHeadersToHide)
.. index:: HttpMessagingV2 (send - showResponseBody)
.. index:: HttpMessagingV2 (send - showResponseStatus)
.. index:: HttpMessagingV2 (send - showResponseHeaders)
.. index:: HttpMessagingV2 (send - responseHeadersToShow)
.. index:: HttpMessagingV2 (send - responseHeadersToHide)
.. index:: HttpMessagingV2 (send - version)

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
    ``requestHeadersToHide`` | No | ``list[string]`` | When request headers are displayed (see ``showRequestHeaders``), this is the ``list`` of headers to explicitly hide (can also be provided as a single ``string``).
    ``requestHeadersToShow`` | No | ``list[string]`` | When request headers are displayed (see ``showRequestHeaders``), this is the ``list`` of headers to explicitly show (can also be provided as a single ``string``).
    ``requestTimeout`` | No | ``number`` | The number of milliseconds to wait for after a connection is established to read back the complete response (by default no timeout is applied).
    ``responseHeadersToHide`` | No | ``list[string]`` | When response headers are displayed (see ``showResponseHeaders``), this is the ``list`` of headers to explicitly hide (can also be provided as a single ``string``).
    ``responseHeadersToShow`` | No | ``list[string]`` | When response headers are displayed (see ``showResponseHeaders``), this is the ``list`` of headers to explicitly show (can also be provided as a single ``string``).
    ``showRequestBody`` | No | ``boolean`` | Whether to display the request's body in the step's report (default is true).
    ``showRequestHeaders`` | No | ``boolean`` | Whether to display the request's headers in the step's report (default is true).
    ``showRequestMethod`` | No | ``boolean`` | Whether to display the request's HTTP method in the step's report (default is true).
    ``showRequestUri`` | No | ``boolean`` | Whether to display the request's URI in the step's report (default is true).
    ``showResponseBody`` | No | ``boolean`` | Whether to display the response's body in the step's report (default is true).
    ``showResponseHeaders`` | No | ``boolean`` | Whether to display the response's headers in the step's report (default is true).
    ``showResponseStatus`` | No | ``boolean`` | Whether to display the response's status in the step's report (default is true).
    ``uri`` | Yes | ``string`` | The URI of the endpoint to be called.
    ``version`` | No | ``string`` | The HTTP protocol version to use. Supported valued are "2" (the default, for HTTP v2) and "1.1" (for HTTP version 1.1).

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
.. index:: HttpMessagingV2 (receive - showRequestUri)
.. index:: HttpMessagingV2 (receive - showRequestMethod)
.. index:: HttpMessagingV2 (receive - showRequestBody)
.. index:: HttpMessagingV2 (receive - showRequestHeaders)
.. index:: HttpMessagingV2 (receive - requestHeadersToShow)
.. index:: HttpMessagingV2 (receive - requestHeadersToHide)
.. index:: HttpMessagingV2 (receive - showResponseBody)
.. index:: HttpMessagingV2 (receive - showResponseStatus)
.. index:: HttpMessagingV2 (receive - showResponseHeaders)
.. index:: HttpMessagingV2 (receive - responseHeadersToShow)
.. index:: HttpMessagingV2 (receive - responseHeadersToHide)

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
    ``requestHeadersToHide`` | No | ``list[string]`` | When request headers are displayed (see ``showRequestHeaders``), this is the ``list`` of headers to explicitly hide (can also be provided as a single ``string``).
    ``requestHeadersToShow`` | No | ``list[string]`` | When request headers are displayed (see ``showRequestHeaders``), this is the ``list`` of headers to explicitly show (can also be provided as a single ``string``).
    ``responseHeadersToHide`` | No | ``list[string]`` | When response headers are displayed (see ``showResponseHeaders``), this is the ``list`` of headers to explicitly hide (can also be provided as a single ``string``).
    ``responseHeadersToShow`` | No | ``list[string]`` | When response headers are displayed (see ``showResponseHeaders``), this is the ``list`` of headers to explicitly show (can also be provided as a single ``string``).
    ``showRequestBody`` | No | ``boolean`` | Whether to display the request's body in the step's report (default is true).
    ``showRequestHeaders`` | No | ``boolean`` | Whether to display the request's headers in the step's report (default is true).
    ``showRequestMethod`` | No | ``boolean`` | Whether to display the request's HTTP method in the step's report (default is true).
    ``showRequestUri`` | No | ``boolean`` | Whether to display the request's URI in the step's report (default is true).
    ``showResponseBody`` | No | ``boolean`` | Whether to display the response's body in the step's report (default is true).
    ``showResponseHeaders`` | No | ``boolean`` | Whether to display the response's headers in the step's report (default is true).
    ``showResponseStatus`` | No | ``boolean`` | Whether to display the response's status in the step's report (default is true).
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

.. _handlers-httpmessagingv2-reportdisplay:

Tuning reported data
^^^^^^^^^^^^^^^^^^^^

You can fine-tune the data displayed by the ``HttpMessagingV2`` handler in test step reports through a set inputs
that apply both when used in :ref:`send <handlers-httpmessagingv2-send>` and :ref:`receive <handlers-httpmessagingv2-receive>` steps.
The display of request and response data is addressed by the following ``boolean`` flags:

* Flags ``showRequestBody``, ``showRequestHeaders``, ``showRequestMethod`` and ``showRequestUri`` for requests.
* Flags ``showResponseBody``, ``showResponseHeaders`` and ``showResponseStatus`` for responses.

These flags determine whether reports will display their respective data. They are optional and by default considered as
true, meaning that all data is displayed. Regarding HTTP headers in particular further control is enabled by listing
specific headers to be considered, specifically:

* Inputs ``requestHeadersToHide`` and ``requestHeadersToShow`` to manage specific request headers.
* Inputs ``responseHeadersToHide`` and ``responseHeadersToShow`` to manage specific response headers.

These inputs are considered only in case HTTP headers are displayed, meaning that flags ``showRequestHeaders`` or
``showResponseHeaders`` are not set, or are set explicitly to true. They are provided with either a ``list`` of header
names or a single header, and disregard casing differences, meaning that for example, a "Content-Type" value will also
match "content-type" headers. The headers listed in ``requestHeadersToShow`` and ``responseHeadersToShow`` serve as a
**whitelist**, meaning that only the matching ones are displayed. In contrast, ``requestHeadersToHide`` and
``responseHeadersToHide`` serve as a **blacklist**, meaning that the matching headers are hidden. In the uncommon
scenario where a header is specified both as being explicitly displayed and hidden, the header will be hidden.

Note that not specifying these inputs means they are full disregarded. In other words if you want to display all request
headers you needn't list them explicitly as ``requestHeadersToShow``. Similarly, when hiding all headers you don't need
to explicitly list them as ``requestHeadersToHide``. What you do achieve in listing them is ensuring that only certain
headers are managed. The following examples show how header display is managed through these inputs:

.. code-block:: xml

    <!--
      Hide all headers.
    -->
    <send id="send1" desc="Send data" handler="HttpMessagingV2">
        ...
        <input name="showRequestHeaders">false()</input>
        <input name="showResponseHeaders">false()</input>
    </send>
    <!--
      Show all headers (the inputs could also be skipped in this case as this is the default).
    -->
    <send id="send2" desc="Send data" handler="HttpMessagingV2">
        ...
        <input name="showRequestHeaders">true()</input>
        <input name="showResponseHeaders">true()</input>
    </send>
    <!--
      Hide a specific response header while showing all others.
    -->
    <send id="send3" desc="Send data" handler="HttpMessagingV2">
        ...
        <input name="responseHeadersToHide">"HEADER-TO-HIDE"</input>
    </send>
    <!--
      Show only a specific response header while hiding all others.
    -->
    <send id="send4" desc="Send data" handler="HttpMessagingV2">
        ...
        <input name="responseHeadersToShow">"HEADER-TO-SHOW"</input>
    </send>

The following example illustrates a more complete scenario where you are making multiple requests to a SUT
and want to ensure that a sensitive API key header is not displayed. Notice here how hidden data can still
be leveraged in subsequent test steps:

.. code-block:: xml

    <!--
      Call the SUT and hide the X-API-KEY header received in the response.
    -->
    <send id="getApiKey" desc="Get API key" handler="HttpMessagingV2">
        <input name="uri">"https://my.sut.org/api/key"</input>
        <input name="responseHeadersToHide">"X-API-KEY"</input>
    </send>
    <!--
      Call the SUT and hide only the X-API-KEY header included in the request.
    -->
    <assign to="submitHeaders{Content-Type}">"application/json"</assign>
    <!--
      The header received from the previous step was hidden in the report but it is still
      stored in the test session context and can be used in subsequent steps.
    -->
    <assign to="submitHeaders{X-API-KEY}">$getApiKey{response}{headers}{X-API-KEY}</assign>
    <send id="submit" desc="Submit data" handler="HttpMessagingV2">
        <input name="uri">"https://my.sut.org/api/submit"</input>
        <input name="method">"PUT"</input>
        <input name="headers">$submitHeaders</input>
        <!--
          Hide (only) the X-API-KEY header from the request.
        -->
        <input name="requestHeadersToHide">"X-API-KEY"</input>
        <!--
          Hide the response's body and headers (i.e. show only the status).
        -->
        <input name="showResponseBody">false()</input>
        <input name="showResponseHeaders">false()</input>
    </send>

Showing and hiding request and response data can be useful in removing **superfluous** information that is not needed in
the step's report. It can also be important in not including information that might be **sensitive** in nature, while still
keeping it available for use in further test steps.
