The ``log`` step is used to add information to the test session's log output at various severity levels. The step itself is not visible on a test case's
diagram but users can inspect its output in the recorded test session log. This step can be used both as a development utility
for test case developers and also as a means of providing additional information to testers. The latter case can be valuable
in providing e.g. technical details to complement a validation step if needed to inspect further details.

The log output is determined by an :ref:`expression <test-case-expressions>` provided as the text content of the ``log`` element.
The element's structure is as follows:

.. csv-table::
    :stub-columns: 1
    :delim: ~
    :header: "Name", "Required?", "Description"

    @asTemplate~ no~ Whether or not the result will be considered as a template for placeholder replacement (see :ref:`test-case-expressions-template-files`). By default this is "false".
    @lang~ no~ Not used currently (and defaulting to XPath as the built-in :ref:`expression language <test-case-expressions>`).
    @level~ no~ The severity level to consider for the log entry. This can be (in increasing severity) ``DEBUG``, ``INFO`` (the default level), ``WARNING`` or ``ERROR``. It can also be provided as a variable reference. See :ref:`tdl-steps-common-level` for further details.
    @skipped~ no~ A boolean value or variable reference (default being "false") which will result in the step being skipped if "true". See also :ref:`tdl-steps-common-skipped`.
    @source~ no~ A variable reference to identify a source ``object`` variable upon which the expression should be evaluated.
    @stopOnError~ no~ A boolean flag determining whether the test session should end if this step fails (default is "false"). See also :ref:`tdl-steps-common-stoponerror`.

The most typical usage of the ``log`` step is to print a message, potentially also referencing variables form the session's state. This can be
easily done using the XPath concatenation operator (``||``) or even by defining the ``log`` step's value as a template and having it processed as
such by setting ``asTemplate`` to "true":

.. code-block:: xml

    <!-- Generate a UUID. -->
    <process handler="TokenGenerator" output="uuid" operation="uuid"/>
    <!-- Log using string concatenation. -->
    <log>"Expecting UUID '" || $uuid || "' to be referenced in message."</log>
    <!-- Log using a string template. -->
    <log asTemplate="true">"Expecting UUID '${uuid}' to be referenced in message."</log>

The following example illustrates further ways the ``log`` step can be used, considering in this case input provided by the
user by means of a :ref:`user interaction step<tdl-step-interact>`:

.. code-block:: xml

    <!-- Add a static message to the log. -->
    <log>'Starting execution of test case'</log>
    <!-- Request certain information from the user. -->
    <interact id="input" desc="User input">
        <request desc="Provide a boolean flag" name="flag" options="true,false"/>
        <request desc="Provide an XML file" contentType="BASE64" name="file"/>
    </interact>
    <!-- Log the provided flag value. -->
    <log>$input{flag}</log>
    <!-- Log a message including the provided flag value. -->
    <log>'You selected: ' || $input{flag}</log>
    <!-- Print the id attribute of the XML file's root element. -->
    <log source="$input{file}">string(/*[local-name() = "myRootElement"]/@id)</log>
    <!-- Define a template text. -->
    <assign to="message">'A value of ${input{flag}} was provided.'</assign>
    <!-- Will process 'message' as a template to produce the log output. -->
    <log asTemplate="true">$message</log>
    <!-- Will process 'message' as a simple text and log its contents without replacing placeholders. -->
    <log>$message</log>
    <!-- Equivalent to the previous case (template processing is disabled by default). -->
    <log asTemplate="false">$message</log>
    <!-- Log a message at a different severity level (a warning in this case). -->
    <log level="WARNING">'The value should normally be received by your service directly.'</log>
    <!-- Log a message at a dynamically defined severity level. -->
    <assign to="logLevel">'WARNING'</assign>
    <log level="$logLevel">'The value should normally be received by your service directly.'</log>

Using the ``log`` step provides flexibility to test developers for conveying information to users that may be difficult to present on the test execution
diagram. When considering such log contributions, the ``log`` step is complemented by the `logging capabilities`_ of `custom test services`_ used as
:ref:`remote service handlers<handlers>` for messaging (:ref:`send<tdl-step-send>`, :ref:`receive<tdl-step-receive>`), processing (:ref:`process<tdl-step-process>`)
and validation (:ref:`verify<tdl-step-verify>`) steps. Such custom services can contribute to the test session log via service call to the Test Bed.

.. note::
    **Test case log level:** You can configure the :ref:`minimum log level for a test case<test-case-steps>` to control which log
    messages are included in the session log.
