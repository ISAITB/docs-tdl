.. index:: Service handlers
.. _handlers:

Service handlers
================

The architectural approach followed by GITB TDL is to capture in the test case the high level testing flow 
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

Another important distinction for handlers is whether they are **built-in** within the test bed software or **external**.
For handlers that relate to domain-specific operations, the norm is to externalise them as remotely callable services.
Nonetheless several common tasks that are frequently encountered in test cases are also available as built-in capabilities
of the test engine.

In the sections that follow you can find:

* The supported :ref:`built-in handlers<handlers-predefined-handlers>` covering common tasks encountered in test cases.
* The list of :ref:`reusable external services <handlers-reusable-handlers>` maintained by the Test Bed (also usable locally as components).
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
located. The test bed will automatically detect in this case that the handler is external and will internally replace local method
invocations with web service calls.

The value provided for the ``handler`` attribute can also be provided with a pure variable reference (see :ref:`test-case-referring-to-variables`)
allowing the actual value to be determined from configuration or even dynamically based on the test session context. In such a case the variable
reference is first evaluated to a ``string`` that is then considered to determine whether the handler is a remote or built-in one.

The following example shows three validation steps taking place, the first one using the built-in :ref:`handlers-XSDValidator`, the second one using 
an external validation service, and the third one using an external validation service whose address is configurable:

.. code-block:: xml

    <!-- 
        Call a local, built-in validation handler called "XSDValidator"
    -->
    <verify handler="XSDValidator" desc="Validate content local">
        <input name="xmldocument">$docToValidate</input>
        <input name="xsddocument">$schemaFile</input>
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
in the GITB test bed software.

.. index:: Built-in messaging handlers
.. _handlers-predefined-handlers-messaging:

Built-in messaging handlers
~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. warning::
    All built-in messaging handlers with the exception of the :ref:`SimulatedMessaging<handlers-simulatedmessaging>` handler are **deprecated**
    as they cannot be effectively customised and require the direct exposure of the internal test engine to systems under test. For any test
    cases involving the sending and receiving messages, you should define instead a custom :ref:`custom messaging handler<handlers-custom-handlers>`.

Each following section defines a table with the information expected by each messaging handler. The meaning of this information is
as follows:

* **Input:** These are the inputs provided for the ``send`` step.
* **Output:** These are the outputs returned from the ``receive`` step.
* **Actor configuration:** These are configuration properties that will be automatically set for simulated actors using this handler.
* **Receive configuration:** These are configuration properties expected by the ``receive`` step.
* **Send configuration:** These are configuration properties expected by the ``send`` step.
* **Transaction configuration:** These are configuration properties defined in the ``btxn`` or ``bptxn`` step.

The title of each section corresponds to the name of the handler that needs to be configured in the relevant step's ``handler`` attribute.

.. index:: HttpMessaging
.. index:: http_headers (HttpMessaging)
.. index:: http_body (HttpMessaging)
.. index:: http_parts (HttpMessaging)
.. index:: http_method (HttpMessaging)
.. index:: http_version (HttpMessaging)
.. index:: http_path (HttpMessaging)
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
  This messaging handler is **deprecated**. Define instead a :ref:`custom messaging handler<handlers-custom-handlers>` to cover your test case messaging needs.

Used to send or receive content over HTTP.

.. csv-table::
    :header: "Element name", "Element type", "Required?", "Type", "Description"

    ``http_version``, Input, No, ``string``, The HTTP version to consider.
    ``http_headers``, Input, No, ``map``, The ``map`` of HTTP headers to send.
    ``http_body``, Input, No, ``binary``, The HTTP request body's bytes.
    ``http_parts``, Input, No, ``map``, A ``map`` including the definition of the parts (see description below).
    ``http_method``, Output, No, ``string``, The HTTP method.
    ``http_version``, Output, No, ``string``, The HTTP version.
    ``http_path``, Output, No, ``string``, The HTTP request path.
    ``http_headers``, Output, No, ``map``, The ``map`` of received headers.
    ``http_body``, Output, No, ``binary``, The bytes of the received body.
    ``http_parts``, Output, No, ``map``, A ``map`` including the received parts (see description below).
    ``network.host``, Actor configuration, Yes, ``string``, The host of the actor.
    ``network.port``, Actor configuration, Yes, ``number``, The listen port for the actor.
    ``http.uri``, Actor configuration, No, ``string``, The request path for the request.
    ``status.code``, Receive configuration, No, ``string``, The status code for responses.
    ``http.method``, Send configuration, Yes, ``string``, The HTTP method to use when sending.
    ``http.uri``, Send configuration, No, ``string``, The request path URI to send to.
    ``http.uri.extension``, Send configuration, No, ``string``, HTTP URI extension for the address.
    ``status.code``, Send configuration, No, ``string``, Status for responses.
    ``http.ssl``, Transaction configuration, No, ``boolean``, Whether or not connections should be over HTTP (default) or HTTPS.

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
    **Isolating communications:** When using a ``HttpMessaging`` handler to receive communication from a SUT, the test bed dynamically starts listening on 
    a new port for incoming traffic. This port (along with the host) are presented to the test bed user upon test initiation so that he/she can configure
    the SUT accordingly. To avoid unwanted communication being received on this port that is unrelated to the test session, the test bed will only 
    listen to requests originating from the SUT's address, ignoring others originating from other sources. To achieve this, the test bed uses the 
    ``network.host`` parameter configured for the SUT that needs to be provided by the tester as part of the SUT's configuration before starting a test.

    The value for the ``network.host`` parameter must be set with the **public IP Address** of the SUT endpoint.

**Using HTTPS**

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
test SUT endpoints over which the test bed has no control over the underlying transport channel. In this case the "http.ssl" parameter could be set as part of 
the system's configuration, as in the following example (assuming an endpoint name of "sutInfo" and an endpoint parameter named "isHTTPS"):

.. code-block:: xml
    :emphasize-lines: 2

    <btxn from="sender" to="receiver" txnId="t1" handler="HttpMessaging">
        <config name="http.ssl">$sutInfo{isHTTPS}</config>
    </btxn>

**Support for sending and receiving multipart form data**

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
        <artifact type="schema" encoding="UTF-8" name="file1">testSuite1/artifacts/file1.xml</artifact>
        <artifact type="binary" encoding="UTF-8" name="file2">testSuite1/artifacts/file2.zip</artifact>
    </imports>
    <variables>
        <var name="parts" type="list[map]"/>
        <var name="filePartInfo1" type="map"/>
        <var name="filePartInfo2" type="map"/>
        <var name="textPartInfo1" type="map"/>
    </variables>
    <actors>
        ...
    </actors>
    <steps>
        <!--
            Define first file part.
        -->
        <assign to="$filePartInfo1{name}" type="string">"file1"</assign>
        <assign to="$filePartInfo1{content_type}" type="string">"text/xml"</assign>
        <assign to="$filePartInfo1{file_name}" type="string">"file1.xml"</assign>
        <assign to="$filePartInfo1{content}" type="binary">$file1</assign>
        <!--
            Define second file part.
        -->
        <assign to="$filePartInfo2{name}" type="string">"file2"</assign>
        <assign to="$filePartInfo2{content_type}" type="string">"application/zip"</assign>
        <assign to="$filePartInfo2{file_name}" type="string">"file2.zip"</assign>
        <assign to="$filePartInfo2{content}" type="binary">$file2</assign>
        <!--
            Define a third text part.
        -->
        <assign to="$textPartInfo1{name}" type="string">"text1"</assign>
        <assign to="$textPartInfo1{content_type}" type="string">"text/plain"</assign>
        <assign to="$textPartInfo1{content}" type="string">"A simple text value"</assign>
        <!--
            Put all parts in a list.
        -->
        <assign to="$parts" append="true">$filePartInfo1</assign>
        <assign to="$parts" append="true">$filePartInfo2</assign>
        <assign to="$parts" append="true">$textPartInfo1</assign>
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

.. index:: HttpsMessaging
.. index:: http_headers (HttpsMessaging)
.. index:: http_body (HttpsMessaging)
.. index:: http_method (HttpsMessaging)
.. index:: http_version (HttpsMessaging)
.. index:: http_path (HttpsMessaging)
.. index:: network.host (HttpsMessaging)
.. index:: network.port (HttpsMessaging)
.. index:: http.uri (HttpsMessaging)
.. index:: http.uri.extension (HttpsMessaging)
.. index:: http.ssl (HttpsMessaging)
.. index:: status.code (HttpsMessaging)
.. index:: http.method (HttpsMessaging)

HttpsMessaging
++++++++++++++

.. warning::
  This messaging handler is **deprecated**. Define instead a :ref:`custom messaging handler<handlers-custom-handlers>` to cover your test case messaging needs.

Used to send or receive content over HTTPS.

.. csv-table::
    :header: "Element name", "Element type", "Required?", "Type", "Description"

    ``http_headers``, Input, No, ``map``, The ``map`` of HTTP headers to send.
    ``http_body``, Input, No, ``binary``, The HTTP request body's bytes.
    ``http_method``, Output, No, ``string``, The HTTP method.
    ``http_version``, Output, No, ``string``, The HTTP version.
    ``http_uri``, Output, No, ``string``, The HTTP request path.
    ``http_headers``, Output, No, ``map``, The ``map`` of received headers.
    ``http_body``, Output, No, ``binary``, The bytes of the received body.
    ``network.host``, Actor configuration, Yes, ``string``, The host of the actor.
    ``network.port``, Actor configuration, Yes, ``number``, The listen port for the actor.
    ``http.uri``, Actor configuration, No, ``string``, The request path for the request.
    ``status.code``, Receive configuration, No, ``string``, The status code for responses.
    ``http.method``, Send configuration, Yes, ``string``, The HTTP method to use when sending.
    ``http.uri.extension``, Send configuration, No, ``string``, HTTP URI extension for the address.
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
  This messaging handler is **deprecated**. Define instead a :ref:`custom messaging handler<handlers-custom-handlers>` to cover your test case messaging needs.

Used to proxy HTTP requests and responses between two actors.

.. csv-table::
    :header: "Element name", "Element type", "Required?", "Type", "Description"
    :delim: ~

    ``request_data``~ Input~ No~ ``map``~ The ``map`` of data to consider. Contains the ``http_method``, ``http_path``, ``http_body``, ``http_headers`` inputs from the HttpMessaging handler.
    ``http_method``~ Output~ No~ ``string``~ The HTTP method.
    ``http_version``~ Output~ No~ ``string``~ The HTTP version.
    ``http_path``~ Output~ No~ ``string``~ The HTTP request path.
    ``network.host``~ Actor configuration~ Yes~ ``string``~ The host of the actor.
    ``network.port``~ Actor configuration~ Yes~ ``number``~ The listen port for the actor.
    ``proxy.address``~ Send configuration~ No~ ``string``~ Address of the proxied service.

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
    :header: "Element name", "Element type", "Required?", "Type", "Description"
    :delim: |

    ``parameters``| Send/receive input| No| ``map``| An optional ``map`` of data to display in the step report.
    ``contentTypes``| Send/receive input| No| ``map``| An optional ``map`` including the content types (e.g. ``application/json``) to consider when displaying different parameters.
    ``result``| Send/receive input| No| ``string``| Set to ``SUCCESS``, ``WARNING`` or ``FAILURE`` to specify the step's result (default is ``SUCCESS``).
    ``delay``| Receive input| No| ``number``| An optional number of milliseconds to delay before presenting the :ref:`receive step<tdl-step-receive>` as completed.

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

.. _handlers-soapmessaging:

SoapMessaging
+++++++++++++

.. warning::
  This messaging handler is **deprecated**. Define instead a :ref:`custom messaging handler<handlers-custom-handlers>` to cover your test case messaging needs.

Used to send or receive payloads via SOAP web service calls.

.. csv-table::
    :header: "Element name", "Element type", "Required?", "Type", "Description"

    ``http_headers``, Input, No, ``map``, A ``map`` of HTTP headers to include.
    ``soap_message``, Input, Yes, ``object``, The SOAP envelope to send.
    ``soap_attachments``, Input, No, ``map``, A ``map`` of ``binary`` attachments.
    ``http_headers``, Output, No, ``map``, The HTTP headers received.
    ``soap_header``, Output, Yes, ``object``, The received SOAP header.
    ``soap_body``, Output, Yes, ``object``, The received SOAP body.
    ``soap_message``, Output, Yes, ``object``, The received SOAP envelope.
    ``soap_content``, Output, Yes, ``object``, The XML content of the received SOAP body.
    ``soap_attachments``, Output, No, ``map``, A ``map`` of received ``binary`` attachments.
    ``soap_attachments_size``, Output, No, ``number``, The number of attachments received.
    ``network.host``, Actor configuration, Yes, ``string``, The host of the actor.
    ``network.port``, Actor configuration, Yes, ``number``, The listen port for the actor.
    ``http.uri``, Actor configuration, No, ``string``, The request path to send the SOAP request to.
    ``soap.version``, Receive configuration, Yes, ``string``, SOAP Version. Can be 1.1 or 1.2.
    ``soap.version``, Send configuration, Yes, ``string``, SOAP Version. Can be 1.1 or 1.2.
    ``soap.encoding``, Send configuration, No, ``string``, Character set encoding.
    ``http.uri.extension``, Send configuration, No, ``string``, HTTP URI extension for the address.
    ``http.ssl``, Transaction configuration, No, ``boolean``, Whether or not connections should be over HTTP (default) or HTTPS.

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

**Using HTTPS**

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
test SUT endpoints over which the test bed has no control over the underlying transport channel. In this case the "http.ssl" parameter could be set as part of 
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

.. index:: Built-in processing handlers
.. _handlers-predefined-handlers-processing:

Built-in processing handlers
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

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

    ``encode``, Receive a ``binary`` input and return a ``string`` with its Base64-encoded representation., Yes, A ``string`` named ``output`` in the resulting step's ``map``.
    ``decode``, Receive a ``string`` input that is Base64-encoded and return the ``binary`` output it corresponds to., Yes, A ``binary`` value named ``output`` in the resulting step's ``map``.

The input parameters expected by the different operations are as follows:

.. csv-table::
    :header: "Operation", "Input name", "Required?", "Description"

    ``encode``, ``input``, Yes, The ``binary`` value that will be encoded as a Base64 string.
    ``encode``, ``dataUrl``, No, A ``boolean`` flag that indicates whether or not the output should be formatted as a data URL (default is ``false``).
    ``decode``, ``input``, Yes, The ``string`` value (expected to be Base64-encoded or formatted as a data URL) that will be processed to return its corresponding ``binary`` value.

Base64 encoding is a technique often used to represent arbitrary byte sequences as text. Using this processing handler you can work with Base64 encoded texts
that need to be decoded in test cases, but also encode binary content where this is needed. In both the encoding and decoding steps there is support for Base64 
content and also data URLs. Data URLs are commonly used in web representations for the inline definition of binary resources. A data URL is essentially the 
Base64-encoded bytes prefixed with the content's mime type as ``data:[mime type],base64,[BASE64 encoded string]`` (e.g. ``data:application/xml;base64,YXNoZGl1cXcgaGRva...``).

The following examples illustrate use of this handler to work with Base64 encoding:

.. code-block:: xml

    <!--
        Encode the binary variable "aBinaryVariable" and return the encoded string as "data1{output}".
    -->
    <process id="data1" handler="Base64Processor">
        <operation>encode</operation>
        <input name="input">$aBinaryVariable</input>
    </process>
    <!--
        Encode the binary variable "aBinaryVariable" and return the encoded string as "data2{output}".
        The result in this case is formatted as a data URL.
    -->
    <process id="data2" handler="Base64Processor">
        <operation>encode</operation>
        <input name="input">$aBinaryVariable</input>
        <input name="dataUrl">'true'</input>
    </process>
    <!--
        Decode a Base-64 encoded string to return its binary equivalent as "data3{output}". In this case
        the result will be identical to the "aBinaryVariable" variable used in the first step.
    -->
    <process id="data3" handler="Base64Processor">
        <operation>decode</operation>
        <input name="input">$data1{output}</input>
    </process>    
    <!--
        Decode a Base-64 encoded string to return its binary equivalent as "data4{output}". In this example
        the handler is provided the data URL produced in the second step and will result in the same output
        as "data3" that matches the original input ("aBinaryVariable").
    -->
    <process id="data4" handler="Base64Processor">
        <operation>decode</operation>
        <input name="input">$data2{output}</input>
    </process>

.. index:: CollectionUtils
.. index:: size (CollectionUtils)
.. index:: clear (CollectionUtils)
.. index:: contains (CollectionUtils)
.. index:: randomKey (CollectionUtils)
.. index:: randomValue (CollectionUtils)
.. index:: remove (CollectionUtils)
.. index:: map (CollectionUtils)
.. index:: list (CollectionUtils)
.. index:: value (CollectionUtils)
.. index:: item (CollectionUtils)
.. _handlers-CollectionUtils:

CollectionUtils
+++++++++++++++

Used to process collections (maps and lists) in ways not possible otherwise with TDL expressions. This processing handler does not require
a processing transaction to be established. The following operations are supported:

.. csv-table::
    :header: "Operation", "Description", "Input(s)", "Output(s)"
    :delim: |

    ``clear`` | Receive a collection as input and empty it. | Yes | No.
    ``contains`` | Check to see whether a collection contains a given value. | Yes | Yes, a ``boolean`` representing the check result.
    ``randomKey`` | Return a random key from a map. | Yes | Yes, one of the map's ``string`` keys.
    ``randomValue`` | Return a random value from a collection. | Yes | Yes, the selected value (type varies depending on the content).
    ``remove`` | Remove an entry from a collection. | Yes | No.
    ``size`` | Receive a collection as input and return the number of elements it contains. | Yes | Yes, a ``number`` named ``output`` in the resulting step's ``map``.

The input parameters expected by the different operations are as follows:

.. csv-table::
    :header: "Operation", "Input name", "Required?", "Description"
    :delim: |

    ``clear`` | ``list`` | No | The ``list`` to be cleared (if the collection is a :ref:`list<test-case-types-lists>`). Either this or the ``map`` input must be provided.
    ``clear`` | ``map`` | No | The ``map`` to be cleared (if the collection is a :ref:`map<test-case-types-maps>`). Either this or the ``list`` input must be provided.
    ``contains`` | ``list`` | No | The ``list`` to be considered (if the collection is a :ref:`list<test-case-types-lists>`). Either this or the ``map`` input must be provided.
    ``contains`` | ``map`` | No | The ``map`` to be considered (if the collection is a :ref:`map<test-case-types-maps>`). Either this or the ``list`` input must be provided.
    ``contains`` | ``value`` | Yes | The value to look for.
    ``randomKey`` | ``map`` | No | The ``map`` to be considered (if the collection is a :ref:`map<test-case-types-maps>`). Either this or the ``list`` input must be provided.
    ``randomValue`` | ``list`` | No | The ``list`` to be considered (if the collection is a :ref:`list<test-case-types-lists>`). Either this or the ``map`` input must be provided.
    ``randomValue`` | ``map`` | No | The ``map`` to be considered (if the collection is a :ref:`map<test-case-types-maps>`). Either this or the ``list`` input must be provided.
    ``remove`` | ``list`` | No | The ``list`` to be considered (if the collection is a :ref:`list<test-case-types-lists>`). Either this or the ``map`` input must be provided.
    ``remove`` | ``map`` | No | The ``map`` to be considered (if the collection is a :ref:`map<test-case-types-maps>`). Either this or the ``list`` input must be provided.
    ``remove`` | ``item`` | Yes | In case of a ``list`` this is a ``number`` set with the zero-based index of the element to remove. For a ``map`` this is the ``string`` key of the entry to be removed.
    ``size`` | ``list`` | No | The ``list`` of which the elements are to be counted (if the collection is a :ref:`list<test-case-types-lists>`). Either this or the ``map`` input must be provided.
    ``size`` | ``map`` | No | The ``map`` of which the elements are to be counted (if the collection is a :ref:`map<test-case-types-maps>`). Either this or the ``list`` input must be provided.

Collection or *container* variables represent flexible means of recording arbitrary sequences of data or hierarchical data structures. In particular
``map`` variables are very common as these are used to store results of :ref:`processing<tdl-step-process>`, :ref:`messaging<tdl-messaging-steps>` and :ref:`validation<tdl-step-verify>` operations.
Adding new elements to collections or replacing existing values is achieved using the :ref:`assign<tdl-step-assign>` step, where the
expressions used may also :ref:`determine collections<test-case-variables-from-expression-output>` that don't previously exist.
The ``CollectionUtils`` processing handler complements such operations by allowing further manipulations that cannot be achieved
through simple :ref:`expressions<test-case-expressions>`.

The ``size`` operation allows a test case to determine a collection's size. This can be particularly useful in the case of
operations that return an arbitrary number of data items as a ``list`` which we need to iterate over. The following examples
illustrate how this operation can be used:

.. code-block:: xml

    <!-- Create a map with three elements -->
    <assign to="aMap{a}">'Value 1'</assign>
    <assign to="aMap{b}">'Value 2'</assign>
    <assign to="aMap{c}">'Value 3'</assign>
    <!-- Create a list with two elements -->
    <assign to="aList" append="true">'Value 1'</assign>
    <assign to="aList" append="true">'Value 2'</assign>
    <!-- Calculate the size of the map -->
    <process id="aMapSize" handler="CollectionUtils">
        <operation>size</operation>
        <input name="map">$aMap</input>
    </process>
    <!-- Prints "3" -->
    <log>$aMapSize{output}</log>
    <!-- Calculate the size of the list -->
    <process id="aListSize" handler="CollectionUtils">
        <operation>size</operation>
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

The ``clear`` operation on the other hand allows a test case to empty the contents of a given collection if this becomes
necessary. The following examples illustrate how this works for lists and maps:

.. code-block:: xml

    <!-- Create a map with three elements -->
    <assign to="aMap{a}">'Value 1'</assign>
    <assign to="aMap{b}">'Value 2'</assign>
    <assign to="aMap{c}">'Value 3'</assign>
    <!-- Create a list with two elements -->
    <assign to="aList" append="true">'Value 1'</assign>
    <assign to="aList" append="true">'Value 2'</assign>
    <!-- Empty the map -->
    <process handler="CollectionUtils">
        <operation>clear</operation>
        <input name="map">$aMap</input>
    </process>
    <!-- Empty the list -->
    <process handler="CollectionUtils">
        <operation>clear</operation>
        <input name="list">$aList</input>
    </process>

The ``contains`` operation allows for a simple lookup of a value with a collection. In case the collection is a ``map``, the lookup is
done on the basis of the entries' keys. Otherwise for a ``list`` the lookup considers the contained elements' values. The following examples
illustration the operation's use:

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

Using the ``randomKey`` and ``randomValue`` operations we can retrieve random entries from collections. The following
examples illustrate their usage:

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
    <!-- Get one of the map's values -->
    <process handler="CollectionUtils" output="value2" operation="randomValue">
        <input name="map">$aMap</input>
    </process>
    <!-- Prints either "Value 1" or "Value 2" -->
    <log>$value2</log>

    <!-- Create a list -->
    <assign to="aList" append="true">'Value 1'</assign>
    <assign to="aList" append="true">'Value 2'</assign>
    <!-- Get one of the list's values -->
    <process handler="CollectionUtils" output="value3" operation="randomValue">
        <input name="list">$aList</input>
    </process>
    <!-- Prints either "Value 1" or "Value 2" -->
    <log>$value3</log>    

The ``remove`` operation is used to remove specific entries from a collection. When using a map the removed entry is matched
based on its key. For lists, the entry to remove is identified by its zero-based index. The following examples illustrate the
operation's use:

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

.. index:: DelayProcessor
.. index:: delay (DelayProcessor)
.. index:: duration (DelayProcessor)
.. _handlers-DelayProcessor:

DelayProcessor
++++++++++++++

Used to pause a test session for a given duration. The following operations are supported:

.. csv-table::
    :header: "Operation", "Description", "Input(s)", "Output(s)"
    :delim: |

    ``delay`` | Delay the test session for a given duration. | Yes | No.

The input parameters expected by the ``delay`` operation are as follows:

.. csv-table::
    :header: "Operation", "Input name", "Required?", "Description"
    :delim: |

    ``delay`` | ``duration`` | Yes | A ``number`` representing the duration (expressed in milliseconds) to delay for.

The following examples illustrate how the ``DelayProcessor`` can be used to force the test session to wait.

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
    :header: "Operation", "Input name", "Required?", "Description"
    :delim: |

    ``display`` | ``result`` | No | A ``string`` with the status (``SUCCESS``, ``FAILURE`` or ``WARNING``) to use for the relevant :ref:`process<tdl-step-process>` step (default is ``SUCCESS``).
    ``display`` | ``parameters`` | No | A ``map`` including the values to display (labelled using the ``map`` keys).
    ``display`` | ``contentTypes`` | No | A ``map`` including the content types (e.g. ``application/json``) to consider when displaying different parameters.

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

.. index:: process (JSONPointerProcessor)
.. index:: content (JSONPointerProcessor)
.. index:: pointer (JSONPointerProcessor)
.. _handlers-JSONPointerProcessor:

JSONPointerProcessor
++++++++++++++++++++

Used to extract values or complete elements from JSON content based on a provided `JSON Pointer expression <https://datatracker.ietf.org/doc/html/rfc6901>`_.
The following operations are supported:

.. csv-table::
    :header: "Operation", "Description", "Input(s)", "Output(s)"
    :delim: |

    ``process`` | Use a JSON Pointer expression to extract a value or full elements from the provided JSON content. | Yes | A ``string`` named ``output`` in the resulting step's ``map``.

The input parameters expected by the ``process`` operation are as follows:

.. csv-table::
    :header: "Operation", "Input name", "Required?", "Description"
    :delim: |

    ``process`` | ``content`` | Yes | A ``string`` representing the JSON content to process.
    ``process`` | ``pointer`` | Yes | A ``string`` representing the JSON Pointer expression to use.

The following example illustrates how the ``JSONPointerProcessor`` can be used to extract a value from JSON content provided by the user via 
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
    <process handler="JSONPointerProcessor" operation="process" output="result">
        <input name="content">$data{json}</input>
        <input name="pointer">"/user/address/streetName"</input>
    </process>
    <!--
        Log the result.
    -->
    <log>$result</log>

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

The input parameters expected by the different operations are as follows:

.. csv-table::
    :header: "Operation", "Input name", "Required?", "Description"
    :delim: |

    ``check`` | ``input`` | Yes | The ``string`` to check.
    ``check`` | ``expression`` | Yes | A ``string`` with the expression that will be used to check the ``input``.
    ``collect`` | ``input`` | Yes | The ``string`` to process to collect data.
    ``collect`` | ``expression`` | Yes | A ``string`` with the expression to collect data with. The provided expression must define at least one capturing group.

Regular expressions offer a very powerful means of describing a text's content and extracting from it certain parts for further processing. They can be used
against any text content, offering a counterpart to the use of XPath in the :ref:`assign<tdl-step-assign>` step that is best adapted, but also limited, to XML structures.
The regular expressions are expected to be provided using the `syntax used by the Java language`_.

The ``check`` operation can be used to verify whether a given text matches a specific pattern. This may at first appear similar to the
:ref:`RegExpValidator<handlers-RegExpValidator>`, however there is a subtle difference: using the :ref:`RegExpValidator<handlers-RegExpValidator>`
constitutes an assertion made by the test case which, if failed, would likely mean that the test session itself is considered failed. The ``check``
operation doesn't presume anything for the test session's status, but is rather used as an internal check to e.g. determine
whether an optional set of steps should be followed. The following example illustrates its use:

.. code-block:: xml

    <!-- Check if a given text includes "test" in a case-insensitive manner -->
    <process id="check" handler="RegExpProcessor">
        <operation>check</operation>
        <input name="input">$someTextData</input>
        <!-- Flags are passed in embedded format (e.g. case insensitive match). -->
        <input name="expression">"(?i)test"</input>
    </process>
    <if desc="Optional steps">
        <cond>$check{output}</cond>
        <then>
            ...
        </then>
    </if>

The ``collect`` operation is used to process a provided text using an expression that defines one or more capturing groups. This
operation can be particularly powerful as it can collect data from both structured and unstructured data. Each matching group
is appended to a ``list`` of ``string`` elements in the sequence with which it was matched, otherwise resulting in an empty ``list``
if no matches were made. Consider the following example to see how this can be used:

.. code-block:: xml

    <!-- Define a firstname and lastname in an unstructured text block -->
    <assign to="aText">"My firstname is 'John' and my lastname is 'Doe'."</assign>
    <!-- Collect the data using an expression with two capturing groups -->
    <process id="personData" handler="RegExpProcessor">
        <operation>collect</operation>
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
    :header: "Operation", "Input name", "Required?", "Description"
    :delim: |

    ``process`` | ``template`` | Yes | The template content to use (can be of any type that results in a ``string``).
    ``process`` | ``parameters`` | No | A ``map`` with named inputs to use as the template's input.
    ``process`` | ``syntax`` | No | A ``string`` to specify what syntax the template uses. Accepted values are ``gitb`` (the default) and ``freemarker``.

The following example illustrates usage of the ``TemplateProcessor`` to create a message based on a FreeMarker template:

.. code-block:: xml

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

In this example the "freemarkerTemplateFile" variable is set (e.g. via :ref:`import<test-case-imports>`) to a template with the following content:

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

Notice here how the template defines FreeMarker constructs (a list iteration) to go over the items of a collection named "listValues". This was passed in the 
"parameters" ``map`` when calling the :ref:`process step<tdl-step-process>`. When executed, and considering the example's input, this step will produce data as follows:

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

    ``uuid``, Generate a random UUID text value matching a Java UUID (e.g. "123e4567-e89b-12d3-a456-556642440000"). This is a value that can be considered as unique for test purposes., No, A ``string`` named ``value`` in the resulting step's ``map``.
    ``timestamp``, Generate a timestamp for the current or a provided time based on a format string., Yes, A ``string`` named ``value`` in the resulting step's ``map``.
    ``string``, Generate a text token with potentially fixed and/or random parts to match a provided regular expression., Yes, A ``string`` named ``value`` in the resulting step's ``map``.
    ``random``, Generate a random integer of double precision number between optional minimum and maximum bounds., Yes, A ``number`` named ``value`` in the resulting step's ``map``.

The input parameters expected by the different operations are as follows:

.. csv-table::
    :header: "Operation", "Input name", "Required?", "Description"
    :delim: |

    ``uuid`` | ``prefix`` | No | An optional ``string`` to add as a prefix to the generated part of the UUID.
    ``uuid`` | ``postfix`` | No | An optional ``string`` to add as a postfix to the generated part of the UUID.
    ``timestamp`` | ``format`` | No | The formatting pattern to apply provided as a ``string`` matching the Java date/time formatting specifications (see `Formatting configuration`_). If unspecified the current Epoch milliseconds are returned.
    ``timestamp`` | ``zone`` | No | The timezone to consider when generating a formatted timestamp provided as a ``string``. Expected values are those defined by Java (see `Timezone codes`_). If unspecified the default consider is ``UTC``.
    ``timestamp`` | ``time`` |  No | A ``number`` representing the Epoch milliseconds to use as the date/time to format. If unspecified the current date/time is used.
    ``timestamp`` | ``date`` |  No | A ``string`` representing a date/time to use as the value to format. If specified along with ``time``, the ``time`` input takes precedence.
    ``timestamp`` | ``inputFormat`` |  No | The formatting pattern to use to interpret the ``date`` input (if provided), matching the Java date/time formatting specifications (see `Formatting configuration`_).
    ``timestamp`` | ``diff`` | No | A ``number`` representing the milliseconds to consider as a diff from the considered ``time`` or ``date``. This value (default 0) is added to the considered ``time`` or ``date`` before formatting (i.e. a negative value signals an earlier time).
    ``string`` | ``format`` | Yes | A regular expression acting as a template to determine the generated token's format.
    ``random`` | ``minimum`` | No | A ``number`` representing the minimum bound (inclusive) for the value to produce. If not provided the default considered is zero.
    ``random`` | ``maximum`` | No | A ``number`` representing the maximum bound (exclusive) for the value to produce.
    ``random`` | ``integer`` | No | A ``boolean`` determining whether the produced value will be an integer (when ``true``) or a double-precision number (when ``false``). By default a double-precision number is produced.

A typical use case for the ``TokenGenerator`` is to generate text tokens that can be used in test cases either as input parameters to
e.g. messaging calls (see :ref:`handlers-inputs-outputs`) or as values to replace in loaded text templates (see :ref:`test-case-expressions-template-files`).
The ``uuid`` operation provides a random and unique identifier where special formatting is not required (apart from an optional prefix and postfix), 
whereas the ``timestamp`` operation generates a timestamp string that includes date/time values but can also have fixed parts (e.g. if you need to generate
a text token with a fixed part and a variable part based on the current date/time). The ``string`` operation can be used to
generate any kind of text token with both fixed and random parts following a pattern based on a provided regular expression. Finally, the ``random`` operation
is used to generate random numbers that can be used as-is, or help in selecting random elements from a ``list``, or as part of XPath expressions.

.. note::
    **Default format for input dates:** If a ``date`` is provided without an ``inputFormat``, the pattern of ``dd/MM/yyyy'T'HH:mm:ss.SSSZ`` is assumed
    by default. Moreover, all parts are considered optional allowing you to specify only parts of a date, making use of the following defaults for those that are missing:

    * Day of year (``dd``): The 1st day of the month.
    * Month (``MM``): The 1st month of the year (January).
    * Year (``yyyy``): The current year.
    * Time elements (``HH``, ``mm``, ``ss`` and ``SSS``): A value of zero.
    * Time zone (``Z``): UTC.

The examples that follow illustrate use of these operations to generate a series of tokens that are then presented to the user by means
of an :ref:`tdl-step-interact` step. Note in all cases how the produced value is retrieved from the ``map`` resulting from the ``process`` step
that is named based on the steps' ``id``. The value itself is retrieved from within each ``map`` under the ``value`` key:

.. code-block:: xml

    <!--
        Generate a UUID (e.g. "971b4df9-4351-4cb8-9ba5-1f6373717ae0").
    -->
    <process id="uuid" handler="TokenGenerator">
        <operation>uuid</operation>
    </process>
    <!--
        Generate a UUID with a prefix and postfix (e.g. "message-971b4df9-4351-4cb8-9ba5-1f6373717ae0@my.org").
    -->
    <process id="uuid" handler="TokenGenerator">
        <operation>uuid</operation>
        <input name="prefix">"message-"</input>
        <input name="postfix">"@my.org"</input>
    </process>
    <!--
        Generate a timestamp for the current time without specifying formatting.
        Example output would be "1560238501040".
    -->
    <process id="defaultTimestamp" handler="TokenGenerator">
        <operation>timestamp</operation>
    </process>
    <!--
        Generate a timestamp for the current time with provided formatting.
        Example output would be "DATE[2019-05-22] TIME[11:48:06]".
    -->
    <process id="formattedTimestamp" handler="TokenGenerator">
        <operation>timestamp</operation>
        <input name="format">"'DATE['yyyy-MM-dd'] TIME['HH:mm:ss']'"</input>
    </process>
    <!--
        Generate an XML timestamp for the current time.
    -->
    <process id="formattedTimestamp" handler="TokenGenerator">
        <operation>timestamp</operation>
        <input name="format">"yyyy-MM-dd'T'HH:mm:ss.SSSXXX"</input>
    </process>
    <!--
        Generate an XML timestamp for the current time but expressed in the GMT+2 timezone.
    -->
    <process id="formattedTimestamp" handler="TokenGenerator">
        <operation>timestamp</operation>
        <input name="format">"yyyy-MM-dd'T'HH:mm:ss.SSSXXX"</input>
        <input name="zone">"GMT+2"</input>
    </process>
    <!-- 
        Generate a timestamp for the provided time and formatting.
        The output would be "2014-05-11".
    -->
    <process id="formattedTimestampProvidedTime" handler="TokenGenerator">
        <operation>timestamp</operation>
        <input name="time">'1399792366000'</input>
        <input name="format">"yyyy-MM-dd"</input>
    </process>
    <!-- 
        Generate a timestamp for the current time minus one minute (600000 milliseconds) using the provided formatting.
        Example output would be "2019-06-11 10:23:10".
    -->
    <process id="formattedTimestampDiff" handler="TokenGenerator">
        <operation>timestamp</operation>
        <input name="diff">-600000</input>
        <input name="format">"yyyy-MM-dd HH:mm:ss"</input>
    </process>    
    <!-- 
        Obtain the current time (T) and then generate two timestamps:
        - T minus one hour.
        - T plus one hour.
    -->
    <process id="now" handler="TokenGenerator">
        <operation>timestamp</operation>
    </process>    
    <process id="nowMinusOneHour" handler="TokenGenerator">
        <operation>timestamp</operation>
        <input name="time">$now{value}</input>
        <input name="diff">-3600000</input>
        <input name="format">"yyyy-MM-dd HH:mm:ss"</input>
    </process>
    <process id="nowPlusOneHour" handler="TokenGenerator">
        <operation>timestamp</operation>
        <input name="time">$now{value}</input>
        <input name="diff">3600000</input>
        <input name="format">"yyyy-MM-dd HH:mm:ss"</input>
    </process>
    <!--
        Generate a timestamp based on an existing date/time string plus one hour
    -->
    <process id="timestampFromFormattedString1" handler="TokenGenerator">
        <operation>timestamp</operation>
        <input name="date">'20-10-2021 13:30:00'</input>
        <input name="inputFormat">'dd-MM-yyyy HH:mm:ss'</input> <!-- Assumes UTC -->
        <input name="diff">3600000</input>
    </process>
    <!--
        Generate a timestamp based on an existing date/time string plus one hour (default formatting)
    -->
    <process id="timestampFromFormattedString2" handler="TokenGenerator">
        <operation>timestamp</operation>
        <input name="date">'20/10'</input> <!-- Assumes the current year, midnight, and a UTC timezone -->
        <input name="diff">3600000</input>
    </process>
    <!--
        Generate a random string with 2 characters followed by 10 digits.
        Example output would be "cD6723820231".
    -->
    <process id="stringRandom" handler="TokenGenerator">
        <operation>string</operation>
        <input name="format">"[a-zA-Z]{2}\d{10}"</input>
    </process>
    <!--
        Generate a random string:
        - Starting with "PREFIX" and ending with "POSTFIX".
        - With random parts of (a) 5 digits, (b) 5 occurences of 'a', 'b' or 'c', and (c) 2 digits.
        - With hyphens between all fixed and random parts.
        Example output would be "PREFIX-32145-abcaa-02-POSTFIX".
    -->
    <process id="stringRandomAndFixed" handler="TokenGenerator">
        <operation>string</operation>
        <input name="format">"PREFIX-\d{5}-[abc]{5}-\d{2}-POSTFIX"</input>
    </process>
    <!--
        Generate a random integer between 1 and 10 (exclusive).
    -->
    <process id="numberRandom" handler="TokenGenerator">
        <operation>random</operation>
        <input name="minimum">1</input>
        <input name="maximum">10</input>
        <input name="integer">true()</input>
    </process>
    <!--
        Display all generated tokens to the user.
    -->
    <interact desc="Generated tokens">
        <instruct desc="UUID:">$uuid{value}</instruct>
        <instruct desc="The default timestamp:">$defaultTimestamp{value}</instruct>
        <instruct desc="A formatted timestamp:">$formattedTimestamp{value}</instruct>
        <instruct desc="A formatted timestamp for provided time:">$formattedTimestampProvidedTime{value}</instruct>
        <instruct desc="A timestamp using a diff:">$formattedTimestampDiff{value}</instruct>
        <instruct desc="Now minus one hour:">$nowMinusOneHour{value}</instruct>
        <instruct desc="Now plus one hour:">$nowPlusOneHour{value}</instruct>
        <instruct desc="Plus one hour of formatted string:">$timestampFromFormattedString1{value}</instruct>
        <instruct desc="Plus one hour of default formatted string:">$timestampFromFormattedString2{value}</instruct>
        <instruct desc="A random string:">$stringRandom{value}</instruct>
        <instruct desc="A random string with fixed parts:">$stringRandomAndFixed{value}</instruct>
        <instruct desc="A random number:">$numberRandom{value}</instruct>
    </interact>

.. note:: 
    **Timestamps for use in XML content:** Formatted timestamps generated for use in XML content should match the formatting
    of the ISO 8601 version of the W3C XML Schema dateTime definition. The pattern to apply to get a XSD-valid timestamp is:
    ``yyyy-MM-dd'T'HH:mm:ss.SSSXXX``.

.. _Formatting configuration: https://docs.oracle.com/javase/8/docs/api/java/time/format/DateTimeFormatter.html
.. _Timezone codes: https://docs.oracle.com/javase/8/docs/api/java/time/ZoneId.html#of-java.lang.String-

.. index:: XSLTProcessor
.. index:: xml (XSLTProcessor)
.. index:: xslt (XSLTProcessor)
.. _handlers-XSLTProcessor:

XSLTProcessor
+++++++++++++

Used to transform XML content using an XSLT style sheet, both being provided as inputs, and output the result.

.. csv-table::
    :header: "Operation", "Description", "Input(s)", "Output(s)"
    :delim: |

    ``process`` | Process XML content using an XSLT style sheet and return the transformed result. | Yes | Yes, a ``string`` named ``output`` in the resulting step's ``map``.

The input parameters expected by the ``process`` operation are as follows:

.. csv-table::
    :header: "Operation", "Input name", "Required?", "Description"
    :delim: |

    ``process`` | ``xml`` | Yes | The XML content to transform.
    ``process`` | ``xslt`` | Yes | The XSLT style sheet to use for the transformation.

The following example illustrates usage of the ``XSLTProcessor`` to transform the provided "xmlContent" (the XML input) using the "xsltContent" (the XSLT style sheet).
These variables may be provided in any manner, for example the "xmlContent" could be uploaded via a :ref:`user interaction step<tdl-step-receive>` whereas the "xsltContent" could
be :ref:`imported<test-case-imports>` from the test suite's resources.

.. code-block:: xml

    <process output="result" handler="XSLTProcessor">
        <input name="xml">$xmlContent</input>
        <input name="xslt">$xsltContent</input>
    </process>
    <log>$result</log>

.. index:: Built-in validation handlers
.. _handlers-predefined-validation-handlers:

Built-in validation handlers
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. index:: ExpressionValidator
.. index:: expression (ExpressionValidator)
.. _handlers-ExpressionValidator:

ExpressionValidator
+++++++++++++++++++

Used to verify whether a provided :ref:`expression<test-case-expressions>` evaluates to ``true``. The 
``ExpressionValidator`` is the most generic validation handler as it can be used to check any arbitrary 
condition.

.. csv-table::
    :header: "Input name", "Required?", "Type", "Description"

    ``expression``, Yes, :ref:`Expression<test-case-expressions>`, The expression to evaluate.

.. code-block:: xml

    <verify handler="ExpressionValidator" desc="Validate UUID">
        <input name="expression">$variable != "unwantedValue"</input>
    </verify>

.. note::
    The ``expression`` input is not presented in the :ref:`verify step's<tdl-step-verify>` validation report as it
    would only ever display a "true" or "false".

.. index:: NumberValidator
.. index:: actualnumber (NumberValidator)
.. index:: expectednumber (NumberValidator)
.. _handlers-NumberValidator:

NumberValidator
+++++++++++++++

Used to verify that a provided ``number`` matches an expected value.

.. csv-table::
    :header: "Input name", "Required?", "Type", "Description"

    ``actualnumber``, Yes, ``number``, The value to check.
    ``expectednumber``, Yes, ``number``, The expected value.

.. code-block:: xml

    <verify handler="NumberValidator" desc="Check number">
        <input name="actualnumber">$aNumber</input>
        <input name="expectednumber">'10'</input>
    </verify>

.. index:: RegExpValidator
.. index:: input (RegExpValidator)
.. index:: expression (RegExpValidator)
.. _handlers-RegExpValidator:

RegExpValidator
+++++++++++++++

Used to verify that a provided ``string`` matches a regular expression.

.. csv-table::
    :header: "Input name", "Required?", "Type", "Description"

    ``input``, Yes, ``string``, The value to check.
    ``expression``, Yes, ``string``, The expression to match.

.. code-block:: xml

    <verify handler="RegExpValidator" desc="Check string">
        <input name="input">$aString</input>
        <input name="expression">'^REF\-\d+$'</input>
    </verify>

The regular expression provided for the ``expression`` input is expected to be provided using the `syntax used by the Java language`_.
This syntax also supports expression flags provided in an embedded manner, within an expression.

.. code-block:: xml

    <verify handler="RegExpValidator" desc="Check string">
        <input name="input">$aString</input>
        <!-- Same expression but executed in a case insensitive (?i) and multiline (?m) manner. -->
        <input name="expression">'(?im)^REF\-\d+$'</input>
    </verify>

.. index:: SchematronValidator
.. index:: schematron (SchematronValidator)
.. index:: xmldocument (SchematronValidator)
.. index:: type (SchematronValidator)
.. index:: showSchematron (SchematronValidator)
.. index:: sortBySeverity (SchematronValidator)
.. index:: showTests (SchematronValidator)
.. _handlers-SchematronValidator:

SchematronValidator
+++++++++++++++++++

Used to validate an XML document against a Schematron file.

.. csv-table::
    :header: "Input name", "Required?", "Type", "Description"

    ``schematron``, Yes, ``schema``, The Schematron file to use for the validation (XSTL or SCH).
    ``xmldocument``, Yes, ``object``, The XML document to validate.
    ``type``, No, ``string``, The type of Schematron file to consider (``xslt`` or ``sch``) in case this cannot be determined from the resource's file suffix. The overall default considered is ``sch``.
    ``showSchematron``, No, ``boolean``, Whether or not to include in the step's report the Schematron used for the validation (default is ``true``).
    ``sortBySeverity``, No, ``boolean``, Whether to sort findings by severity (``true``) or location in the input (``false`` - the default).
    ``showTests``, No, ``boolean``, Whether or not to include in the step's report the assertion performed for each finding (default is ``false``).

.. note::
    **XSLT vs SCH Schematron files:** XSLT versions of Schematron files are pre-processed files and offer significantly better
    performance for complex rule cases. In addition, if Schematron rules import other resources, use of XSLT files is required.

.. code-block:: xml

    <verify handler="SchematronValidator" desc="Validate content">
        <input name="xmldocument">$docToValidate</input>
        <input name="schematron">$schematronFile</input>
        <!-- Following inputs are optional. -->
        <input name="showSchematron">false()</input>
        <input name="sortBySeverity">true()</input>
        <input name="showTests">true()</input>
    </verify>

.. index:: StringValidator
.. index:: actualstring (StringValidator)
.. index:: expectedstring (StringValidator)
.. _handlers-StringValidator:

StringValidator
+++++++++++++++

Used to verify that a provided ``string`` matches an expected value.

.. csv-table::
    :header: "Input name", "Required?", "Type", "Description"

    ``actualstring``, Yes, ``string``, The value to check.
    ``expectedstring``, Yes, ``string``, The expected value.

.. code-block:: xml

    <verify handler="StringValidator" desc="Check string">
        <input name="actualstring">$aString</input>
        <input name="expectedstring">'expected_string'</input>
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

    ``xml``, Yes, ``object``, The XML file to validate.
    ``template``, Yes, ``object``, The XML file to consider as the validation's template.
    ``ignoredPaths``, No, ``list[string]``, An optional list of paths provided as XPath expressions identifying sections of the XML to ignore.

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
    <assign to="$pathsToSkip" append="true">"/x:Invoice/x:BillingInformation/y:Comments"</assign>
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

    ``xml``, Yes, ``object``, The XML document to validate.
    ``xsd``, No, ``schema``, The XSD to validate the document's structure against.
    ``schematron``, No, ``list[schema]``, The list of Schematron files to validate the document's content against.
    ``schematronType``, No, ``string``, The type of Schematron file to consider (``xslt`` or ``sch``) in case this cannot be determined from the files' suffix. The overall default considered is ``sch``.
    ``stopOnXsdErrors``, No, ``boolean``, Whether or not XSD errors should prevent validation from proceeding with Schematron validations (default is ``true``).
    ``sortBySeverity``, No, ``boolean``, Whether findings should be sorted by severity (``true``) or by location in the XML content (``false`` - the default).
    ``showValidationArtefacts``, No, ``boolean``, Whether or not the XSDs and/or Schematrons used for the validation should be included in the step's report (default is ``true``).
    ``showSchematronTests``, No, ``boolean``, Whether or not the Schematron assertions applied should be displayed for each reported finding (default is ``false``).

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
        Using the XSDValidator and SchematronValidator.
    -->
    <verify handler="XSDValidator" desc="Validate content">
        <input name="xmldocument">$docToValidate</input>
        <input name="xsddocument">$schemaFile</input>
    </verify>
    <verify handler="SchematronValidator" desc="Validate content">
        <input name="xmldocument">$docToValidate</input>
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
.. index:: xmldocument (XPathValidator)
.. index:: xpathexpression (XPathValidator)
.. _handlers-XPathValidator:

XPathValidator
++++++++++++++

Used to evaluate an XPath 3.0 expression against a provided XML document. The result of the expression
needs to evaluate to a boolean (i.e. true for success or false for failure).

.. csv-table::
    :header: "Input name", "Required?", "Type", "Description"

    ``xmldocument``, Yes, ``object``, The XML document upon which the XPath expression will be evaluated.
    ``xpathexpression``, Yes, ``string``, The XPath 3.0 expression passed as a string.

An important note here is that the XPath expression passed in ``xpathexpression`` is meant to be a string.
This means that to run an expression as-is you need to wrap it in quotes. This is because the content of
the ``input`` element can also be an expression that you want to evaluate to give you the final expression to
use. The following example illustrates both cases:

.. code-block:: xml

    <!-- 
        Pass a string as the expression to use.
    -->
    <verify handler="XPathValidator" desc="Check document">
        <input name="xmldocument">$myDocument</input>
        <input name="xpathexpression">"contains(/toc/text(), 'string to look for')"</input>
    </verify>
    <!-- 
        Evaluate an expression that will give you the final expression to use.
    -->
    <verify handler="XPathValidator" desc="Check document">
        <input name="xmldocument">$myDocument</input>
        <input name="xpathexpression">concat("contains(/toc/text()", ", 'string to look for')")</input>
    </verify>

In the expressions you use for the validations (attribute ``xpathexpression``) you may also make use of XML namespaces. Doing so is actually
a best practice to ensure that you don't have ambiguous results due to elements with the same local names. To use namespaces in expressions
you first need to define their prefixes in the test case's :ref:`namespaces section<test-case-namespaces>`. Moreover, keep in mind that the
provided input (attribute ``xmldocument``) also supports expressions with namespaces when determining the XML content to apply the XPath
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
                <input name="xmldocument">$myDocument</input>
                <input name="xpathexpression">"/ns1:Foo/ns2:Bar/text() = 'EXPECTED'"</input>
            </verify>  
        </steps>
    </testcase>


.. index:: XSDValidator
.. index:: xsddocument (XSDValidator)
.. index:: xmldocument (XSDValidator)
.. index:: showSchema (XSDValidator)
.. index:: sortBySeverity (XSDValidator)
.. _handlers-XSDValidator:

XSDValidator
++++++++++++

Used to validate an XML document against an XML Schema (XSD) instance.

.. csv-table::
    :header: "Input name", "Required?", "Type", "Description"

    ``xsddocument``, Yes, ``schema``, The XSD to validate the document against.
    ``xmldocument``, Yes, ``object``, The XML document to validate.
    ``showSchema``, No, ``boolean``, Whether to include in the step's report the XSD used for the validation (default is ``true``).
    ``sortBySeverity``, No, ``boolean``, Whether to sort findings by severity (``true``) or location in the input (``false`` - the default).

.. code-block:: xml

    <verify handler="XSDValidator" desc="Validate content">
        <input name="xmldocument">$docToValidate</input>
        <input name="xsddocument">$schemaFile</input>
        <!-- Following inputs are optional. -->
        <input name="showSchema">false()</input>        
        <input name="sortBySeverity">true()</input>        
    </verify>

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

* ``as4.receive.type``, set to "ack_check".
* ``as4.receive.messageId``, the ID of the message to check for (of type ``string``).

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

    ``initialize`` ~ Provide the ZIP archive to the service for subsequent extraction operations. ~ Yes ~ A ``map`` with two elements (``entries``, a ``number`` representing the count of entries included in the archive; ``entryPaths``, a ``string`` including a summary of the included paths, listing them one by one in square brackets).
    ``extract``~ Extract one or more files from the archive. ~ Yes ~ A ``map`` containing two entries (``entries``, a ``number`` representing the count of entries that were matched; ``entry``, a list with one item per matched entry). Each item in the entry list (corresponding to a matched entry) is a ``map`` with two further fields (``path``, a string with the file's precise path; ``content``, the binary content of the file).

The input parameters expected by the different operations are as follows:

.. csv-table::
    :header: "Operation", "Input name", "Required?", "Description"
    :delim: ~

    ``initialize`` ~ ``zip`` ~ Yes ~ A ``binary`` input corresponding to the archive to process.
    ``extract`` ~ ``path`` ~ Yes ~ A ``string`` with the path of the archive's entry (or entries) to return.
    ``extract`` ~ ``case`` ~ No ~ A ``string`` set as "true" or "false" (the default) specifying whether the path matching should be case sensitive.
    ``extract`` ~ ``match`` ~ No ~ A ``string`` set as "exact" or "regexp" (the default) specifying whether the path should be considered for an exact match or as a regular expression.

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
       <!--
          Use the extracted file (first match)
       -->
       <log>"Processing file " || $zip{entry}{0}{path} "..."</log>
       <assign to="file">$zip{entry}{0}{content}</assign>
       <!--
          Close the processing transaction to release the processed archive.
       -->
       <etxn txnId="t1"/>
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
* **As a service**, by setting your handler to ``https://www.itb.ec.europa.eu/csv/soap/any/validation?wsdl``.

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
* **As a service**, by setting your handler to ``https://www.itb.ec.europa.eu/json/soap/any/validation?wsdl``.

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
* **As a service**, by setting your handler to ``https://www.itb.ec.europa.eu/shacl/soap/any/validation?wsdl``.

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
* **As a service**, by setting your handler to ``https://www.itb.ec.europa.eu/xml/api/validation?wsdl``.

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

The starting point for the implementation is the Test Bed's set of `published template services <https://www.itb.ec.europa.eu/docs/services/latest/templates/index.html>`_.
These templates are **executable**, allowing you to create new services based on existing demo starting implementations. Although simple, the pre-existing
implementations fully cover the GITB service APIs and allow you to replace them with your own logic. Moreover, the documentation also includes a
`sample test case <https://www.itb.ec.europa.eu/docs/services/latest/templates/index.html#example-test-case>`_ that illustrates how the demo service
implementations can be used in test steps.

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

    ``auth.basic.username``, Any ``string``, The username to provide when prompted for basic HTTP authentication.
    ``auth.basic.password``, Any ``string``, The password to provide when prompted for basic HTTP authentication.
    ``auth.token.username``, Any ``string``, The username to include in the SOAP header as the UsernameToken's username.
    ``auth.token.password``, Any ``string``, The password to include in the SOAP header as the UsernameToken's password.
    ``auth.token.password``.type, 'DIGEST' (the default) or 'TEXT', The way the password is to be serialised in the header. 'DIGEST' includes it as a DIGEST whereas 'TEXT' adds it in plaintext.

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

    ``@name``, no, The name of the input or output element.
    ``@lang``, no, The expression language that should be considered when evaluating its contained expression (see :ref:`test-case-expressions`).
    ``@source``, no, A pure variable reference identifying a source variable. Used as the target upon which to evaluate the contained expression.
    ``@asTemplate``, no, Whether or not the result will be considered as a template for placeholder replacement (see :ref:`test-case-expressions-template-files`). By default this is "false".

The text content of the element is considered to be an expression (see :ref:`test-case-expressions`). In the case a ``source`` attribute is provided
the contained expression is evaluated on the variable identified by ``source`` to produce the value. If no ``source`` attribute is present the value
is the result of the expression itself. For inputs of type ``object`` or ``schema`` (i.e. XML documents) the ``source`` attribute can also be used to pass
the complete document as the value. In this case use of the ``source`` attribute to reference the relevant variable is equivalent to specifying its
reference as the expression:

.. code-block:: xml

    <verify handler="SchematronValidator" desc="Validate content">
        <!--
            Pass document through the expression.
        -->
        <input name="xmldocument">$docToValidate</input>
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
by the test bed before calling a service handler to ensure that required parameters are provided by the test case.

.. _syntax used by the Java language: https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/regex/Pattern.html