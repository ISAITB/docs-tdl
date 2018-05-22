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

Another important distinction for handlers is whether they are embedded within the test bed software or external.
Considering that handlers are typically used to extend the test bed for domain-specific operations, the norm
is to externalise them as remotely callable services. Embedded handlers are typically defined for generic and
simple use cases that are frequently encountered in test cases.

One thing that needs to be clear to test case authors is that the use of embedded handlers limits the 
portability of their test cases. Each embedded handler used needs to be implemented in exactly the same
way in another test bed and furthermore needs to be identified using the same name.

.. _handlers-implementation:

Specifying the handler implementation
-------------------------------------

Handlers are defined in the following steps:
 
* :ref:`tdl-step-btxn`: When beginning a messaging transaction.
* :ref:`tdl-step-bptxn`: When beginning a processing transaction.
* :ref:`tdl-step-verify`: When validating content.

The element corresponding to each of these steps defines a ``handler`` attribute to identify the handler implementation.
In case an embedded handler is to be used the value specified here is the name of the handler (see :ref:`handlers-predefined-handlers`). Using an external
handler implementation is achieved by specifying as the ``handler`` value the address where the service's WSDL file is 
located. The test bed will automatically detect in this case that the handler is external and will internally replace local method
invocations with web service calls.

The value provided for the ``handler`` attribute can also be provided with a pure variable reference (see :ref:`test-case-referring-to-variables`)
allowing the actual value to be determined from configuration or even dynamically based on the test session context. In such a case the variable
reference is first evaluated to a ``string`` that is then considered to determine whether the handler is a remote or embedded one.

The following example shows three validation steps taking place, the first one using an embedded :ref:`handlers-XSDValidator`, the second one using 
an external validation service, and the third one using an external validation service whose address is configurable:

.. code-block:: xml

    <!-- 
        Call a local, embedded validation handler called "XSDValidator"
    -->
    <verify handler="XSDValidator" desc="Validate content local">
        <input name="xmldocument">$docToValidate</input>
        <input name="xsddocument">$schemaFile</input>
    </verify>
    <!-- 
        Call a remote validation service handler
    -->
    <verify handler="https://serviceaddress?wsdl" desc="Validate content remote">
        <input name="xmldocument">$docToValidate</input>
        <input name="xsddocument">$schemaFile</input>
    </verify>
    <!-- 
        Call a remote validation service handler (address in configuration)
    -->
    <verify handler="$DOMAIN{validationHandlerAddress}" desc="Validate content remote">
        <input name="xmldocument">$docToValidate</input>
        <input name="xsddocument">$schemaFile</input>
    </verify>

Using remote service handlers is considered a best practice based on the benefits they offer:

* **Scalability:** Potentially heavy processing is handled by a dedicated service outside the test bed that can be 
  scaled appropriately.
* **Separation of concerns:** The test bed focuses on test orchestration whereas domain specific logic is captured only
  in the test case and the services it uses.
* **Extensibility:** New capabilities can be added to the test bed by simply making available a new service to call.
* **Maintenance:** Updates to service handlers can take place without impacting test bed operations or requiring new
  versions of the test bed software. Similarly external service updates would not require new test suite versions.
* **Better presentation:** Remote service handlers can encapsulate multiple custom actions leading to better test session
  presentation. If e.g. a document needs to be validated by one XSD and two Schematron files we would only show a single,
  concise validation step versus three separate validations.

.. index:: Embedded handlers
.. _handlers-predefined-handlers:

Embedded handlers
-----------------

The sections that follow list the handler implementations that already exist as predefined embedded implementations
in the GITB test bed software.

.. index:: Embedded messaging handlers

Embedded messaging handlers
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Each following section defines a table with the information expected by each messaging handler. The meaning of this information is
as follows:

* **Input:** These are the inputs provided for the ``send`` step.
* **Output:** These are the outputs returned from the ``receive`` step.
* **Actor configuration:** These are configuration properties that will be automatically set for simulated actors using this handler.
* **Receive configuration:** These are configuration properties expected by the ``receive`` step.
* **Send configuration:** These are configuration properties expected by the ``send`` step.
* **Transaction configuration:** These are configuration properties defined in the ``btxn`` or ``bptxn`` step.

The title of each section corresponds to the name of the handler that needs to be configured in the relevant step's ``handler`` attribute.

.. index:: TCPMessaging

TCPMessaging
++++++++++++

Used to send or receive an arbitrary byte stream over TCP.

.. csv-table::
    :stub-columns: 1
    :header: "Element name", "Element type", "Required?", "Type", "Description"

    content, Input, Yes, ``binary``, The stream of bytes to send.
    content, Output, Yes, ``binary``, The stream of bytes received.
    network.host, Actor configuration, Yes, ``string``, The host of the actor.
    network.port, Actor configuration, Yes, ``number``, The listen port for the actor.

.. code-block:: xml

    <btxn from="Actor1" to="Actor2" txnId="t1" handler="TCPMessaging"/>
    <send id="dataSend" desc="Send data" from="Actor1" to="Actor2" txnId="t1">
        <input name="content">$binaryContent</input>
    </send>
    <receive id="dataReceive" desc="Receive data" from="Actor2" to="Actor1" txnId="t1"/>
    <etxn txnId="t1"/>

.. index:: SoapMessaging

SoapMessaging
+++++++++++++

Used to send or receive payloads via SOAP web service calls.

.. csv-table::
    :stub-columns: 1
    :header: "Element name", "Element type", "Required?", "Type", "Description"

    http_headers, Input, No, ``map``, A ``map`` of HTTP headers to include.
    soap_message, Input, Yes, ``object``, The SOAP envelope to send.
    soap_attachments, Input, No, ``map``, A ``map`` of ``binary`` attachments.
    http_headers, Output, No, ``map``, The HTTP headers received.
    soap_header, Output, Yes, ``object``, The received SOAP header.
    soap_body, Output, Yes, ``object``, The received SOAP body.
    soap_message, Output, Yes, ``object``, The received SOAP envelope.
    soap_content, Output, Yes, ``object``, The XML content of the received SOAP body.
    soap_attachments, Output, No, ``map``, A ``map`` of received ``binary`` attachments.
    soap_attachments_size, Output, No, ``number``, The number of attachments received.
    network.host, Actor configuration, Yes, ``string``, The host of the actor.
    network.port, Actor configuration, Yes, ``number``, The listen port for the actor.
    http.uri, Actor configuration, No, ``string``, The request path to send the SOAP request to.
    soap.version, Receive configuration, Yes, ``string``, SOAP Version. Can be 1.1 or 1.2.
    soap.version, Send configuration, Yes, ``string``, SOAP Version. Can be 1.1 or 1.2.
    soap.encoding, Send configuration, No, ``string``, Character set encoding.
    http.uri.extension, Send configuration, No, ``string``, HTTP URI extension for the address.
    http.ssl, Transaction configuration, No, ``boolean``, Whether or not connections should be over HTTP (default) or HTTPS.

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

.. index:: HttpMessaging

HttpMessaging
+++++++++++++

Used to send or receive content over HTTP.

.. csv-table::
    :stub-columns: 1
    :header: "Element name", "Element type", "Required?", "Type", "Description"

    http_version, Input, No, ``string``, The HTTP version to consider.
    http_headers, Input, No, ``map``, The ``map`` of HTTP headers to send.
    http_body, Input, No, ``binary``, The HTTP request body's bytes.
    http_method, Output, No, ``string``, The HTTP method.
    http_version, Output, No, ``string``, The HTTP version.
    http_path, Output, No, ``string``, The HTTP request path.
    http_headers, Output, No, ``map``, The ``map`` of received headers.
    http_body, Output, No, ``binary``, The bytes of the received body.
    network.host, Actor configuration, Yes, ``string``, The host of the actor.
    network.port, Actor configuration, Yes, ``number``, The listen port for the actor.
    http.uri, Actor configuration, No, ``string``, The request path for the request.
    status.code, Receive configuration, No, ``string``, The status code for responses.
    http.method, Send configuration, Yes, ``string``, The HTTP method to use when sending.
    http.uri, Send configuration, No, ``string``, The request path URI to send to.
    http.uri.extension, Send configuration, No, ``string``, HTTP URI extension for the address.
    status.code, Send configuration, No, ``string``, Status for responses.

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

.. index:: HttpsMessaging

HttpsMessaging
++++++++++++++

Used to send or receive content over HTTPS.

.. csv-table::
    :stub-columns: 1
    :header: "Element name", "Element type", "Required?", "Type", "Description"

    http_headers, Input, No, ``map``, The ``map`` of HTTP headers to send.
    http_body, Input, No, ``binary``, The HTTP request body's bytes.
    http_method, Output, No, ``string``, The HTTP method.
    http_version, Output, No, ``string``, The HTTP version.
    http_uri, Output, No, ``string``, The HTTP request path.
    http_headers, Output, No, ``map``, The ``map`` of received headers.
    http_body, Output, No, ``binary``, The bytes of the received body.
    network.host, Actor configuration, Yes, ``string``, The host of the actor.
    network.port, Actor configuration, Yes, ``number``, The listen port for the actor.
    http.uri, Actor configuration, No, ``string``, The request path for the request.
    status.code, Receive configuration, No, ``string``, The status code for responses.
    http.method, Send configuration, Yes, ``string``, The HTTP method to use when sending.
    http.uri.extension, Send configuration, No, ``string``, HTTP URI extension for the address.
    status.code, Send configuration, No, ``string``, Status for responses.

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

.. index:: HttpProxyMessaging
.. _handlers-HttpProxyMessaging:

HttpProxyMessaging
++++++++++++++++++

Used to proxy HTTP requests and responses between two actors.

.. csv-table::
    :stub-columns: 1
    :header: "Element name", "Element type", "Required?", "Type", "Description"

    request_data, Input, No, ``map``, The ``map`` of data to consider. Contains the ``http_method``, ``http_path``, ``http_body``, ``http_headers`` inputs from the HttpMessaging handler.
    http_method, Output, No, ``string``, The HTTP method.
    http_version, Output, No, ``string``, The HTTP version.
    http_path, Output, No, ``string``, The HTTP request path.
    network.host, Actor configuration, Yes, ``string``, The host of the actor.
    network.port, Actor configuration, Yes, ``number``, The listen port for the actor.
    proxy.address, Send configuration, No, ``string``, Address of the proxied service.

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

.. index:: Embedded processing handlers

Embedded processing handlers
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

No processing handlers currently exist as predefined and embedded in the test bed software.

.. index:: Embedded validation handlers
.. _handlers-predefined-validation-handlers:

Embedded validation handlers
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. index:: NumberValidator

NumberValidator
+++++++++++++++

Used to verify that a provided ``number`` matches an expected value.

.. csv-table::
    :stub-columns: 1
    :header: "Input name", "Required?", "Type", "Description"

    actualnumber, Yes, ``number``, The value to check.
    expectednumber, Yes, ``number``, The expected value.

.. code-block:: xml

    <verify handler="NumberValidator" desc="Check number">
        <input name="actualnumber">$aNumber</input>
        <input name="expectednumber">'10'</input>
    </verify>

.. index:: StringValidator

StringValidator
+++++++++++++++

Used to verify that a provided ``string`` matches an expected value.

.. csv-table::
    :stub-columns: 1
    :header: "Input name", "Required?", "Type", "Description"

    actualstring, Yes, ``string``, The value to check.
    expectedstring, Yes, ``string``, The expected value.

.. code-block:: xml

    <verify handler="StringValidator" desc="Check string">
        <input name="actualstring">$aString</input>
        <input name="expectedstring">'expected_string'</input>
    </verify>

.. index:: XPathValidator
.. _handlers-XPathValidator:

XPathValidator
++++++++++++++

Used to evaluate an XPath 1.0 expression against a provided XML document. The result of the expression
needs to evaluate to a boolean (i.e. true for success or false for failure).

.. csv-table::
    :stub-columns: 1
    :header: "Input name", "Required?", "Type", "Description"

    xmldocument, Yes, ``object``, The XML document upon which the XPath expression will be evaluated.
    xpathexpression, Yes, ``string``, The XPath 1.0 expression passed as a string.

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

.. index:: XSDValidator
.. _handlers-XSDValidator:

XSDValidator
++++++++++++

Used to validate an XML document against an XML Schema (XSD) instance.

.. csv-table::
    :stub-columns: 1
    :header: "Input name", "Required?", "Type", "Description"

    xsddocument, Yes, ``schema``, The XSD to validate the document against.
    xmldocument, Yes, ``object``, The XML document to validate.

.. code-block:: xml

    <verify handler="XSDValidator" desc="Validate content">
        <input name="xmldocument">$docToValidate</input>
        <input name="xsddocument">$schemaFile</input>
    </verify>

.. index:: SchematronValidator
.. _handlers-SchematronValidator:

SchematronValidator
+++++++++++++++++++

Used to validate an XML document against a Schematron file.

.. csv-table::
    :stub-columns: 1
    :header: "Input name", "Required?", "Type", "Description"

    schematron, Yes, ``schema``, The Schematron file to use for the validation (XSTL or SCH).
    xmldocument, Yes, ``object``, The XML document to validate.

.. code-block:: xml

    <verify handler="SchematronValidator" desc="Validate content">
        <input name="xmldocument">$docToValidate</input>
        <input name="schematron">$schematronFile</input>
    </verify>

.. index:: Handler inputs and outputs
.. _handlers-inputs-outputs:

Handler inputs and outputs
--------------------------

The ``input`` and ``output`` elements used with handlers are what GITB refers to as "Binding elements".
They share the following structure:

.. csv-table::
    :stub-columns: 1
    :header: "Name", "Required?", "Description"

    @name, no, The name of the input or output element.
    @lang, no, The expression language that should be considered when evaluating its contained expression (see :ref:`test-case-expressions`).
    @source, no, A pure variable reference identifying a source variable. Used as the target upon which to evaluate the contained expression.

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
    **Specifying a fixed value:** Considering that the default expression language is XPath 1.0, a fixed text value is provided by enclosing it in
    quotes. See :ref:`test-case-expressions` for further details.

The ``input`` and ``output`` options for service handlers are documented as part of their module definition. For handlers accessible
via remote web service calls this information is returned when calling the handler's ``getModuleDefinition`` operation. This is also used internally
by the test bed before calling a service handler to ensure that required parameters are provided by the test case.