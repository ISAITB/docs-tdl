Used to add simulated messaging steps to the test execution diagram without any actual message exchanges taking
place.

.. csv-table::
    :header: "Input name", "Input type", "Required?", "Type", "Description"
    :delim: |

    ``contentTypes``| Send/receive input| No| ``map``| An optional ``map`` including the content types (e.g. ``application/json``) to consider when displaying different parameters.
    ``delay``| Receive input| No| ``number``| An optional number of milliseconds to delay before presenting the :ref:`receive step<tdl-step-receive>` as completed.
    ``parameters``| Send/receive input| No| ``map``| An optional ``map`` of data to display in the step report.
    ``reportItems`` | Send/receive input | No | ``map`` | A ``map`` of report items to display as a detailed validation report.
    ``reportSteps`` | Send/receive input | No | ``list`` |A ``list`` of step identifiers from which to source the current step's detailed validation report.
    ``result``| Send/receive input| No| ``string``| Set to ``SUCCESS``, ``WARNING`` or ``FAILURE`` to specify the step's result (default is ``SUCCESS``).
    ``sortReportBySeverity`` | Send/receive input | No | ``boolean`` | A ``boolean`` flag (false by default) specifying whether report items should be sorted based on severity first and then location.

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

    <!--
        Define the data.
    -->
    <assign to="params{input}">$file1</assign>
    <assign to="params{output}">$file2</assign>
    <!--
        Define the content types.
    -->
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

    <!--
        Define the data.
    -->
    <assign to="params{aFile}">$file</assign>
    <assign to="params{countryCode}">$countryCode</assign>
    <assign to="params{message}">"Transformation was successful."</assign>
    <assign to="params{aSecondFile}">$secondFile</assign>
    <!--
        Define the content type only for the first file.
    -->
    <assign to="contentTypes{aFile}">'application/xml'</assign>
    <send desc="Send message" from="Actor1" to="Actor2" handler="SimulatedMessaging">
        <input name="parameters">$params</input>
        <input name="contentTypes">$contentTypes</input>
    </send>

Finally, the following example illustrates how content types can be provided when the parameters are defined within complex
structures (maps and lists, nested at different levels).

.. code-block:: xml

    <!--
        Define the data.
    -->
    <assign to="params{input}{file1}">$file1</assign>
    <assign to="params{input}{file2}">$file2</assign>
    <assign to="params{input}{messageId}">$messageIdentifier</assign>
    <assign to="params{input}{attachments}" append="true">$attachment1</assign>
    <assign to="params{input}{attachments}" append="true">$attachment2</assign>
    <assign to="params{output}{response}">$response</assign>
    <assign to="params{output}{message}">"Input processed successfully."</assign>
    <!--
        Define the content types.
    -->
    <assign to="types{input}{file1}">"application/xml"</assign>
    <assign to="types{input}{file2}">"application/xml"</assign>
    <assign to="types{input}{attachments}" append="true">"text/plain"</assign>
    <assign to="types{input}{attachments}" append="true">"application/pdf"</assign>
    <assign to="types{output}{response}">"application/json"</assign>
    <!--
        Call the send step.
    -->
    <send desc="Send message" from="Actor1" to="Actor2" handler="SimulatedMessaging">
        <input name="parameters">$params</input>
        <input name="contentTypes">$types</input>
    </send>

Besides presenting data, the ``SimulatedMessaging`` handler can also display a **detailed validation report**.
This can be constructed by using one or both of the ``reportItems`` and ``reportSteps`` inputs:

* Use ``reportItems`` to manually construct a report based on provided items.
* Use ``reportSteps`` to include the report items from other steps.

The ``reportItems`` input is used to provide a map of report items, mapped by severity level under keys named ``error``,
``warning`` and ``info``. Each of these map entries can either be set with a list of items, or with a single item.
Each item can be provided as a simple string, or as a map to include additional information. In this latter case
(item set as a map), the keys you can set are ``description``, ``location``, ``assertionId``, and ``test``.

The following example illustrates how a report can be produced with two information messages (provided as simple strings),
one warning (also provided as a string), and an error (provided as a map to include an assertion ID).

.. code-block:: xml

    <!--
        Include an XML document to display as the step's context data.
    -->
    <assign to="parameters{xml}">$xmlData</assign>
    <assign to="contentTypes{xml}">"application/xml"</assign>
    <!--
        Define information messages and a warning message to include in the report.
    -->
    <assign to="reportItems{info}" append="true">'First information message.'</assign>
    <assign to="reportItems{info}" append="true">'Second information message.'</assign>
    <assign to="reportItems{warning}" append="true">'A warning message.'</assign>
    <!--
        Define an error message as a map to highlight additional information.
    -->
    <assign to="xmlError{description}">'An error message.'</assign>
    <!--
        Set the 'location' to point to line 2, column 0 of the data named 'xml'.
        This is included in the parameters above.
    -->
    <assign to="xmlError{location}">'xml:2:0'</assign>
    <assign to="xmlError{assertionId}">'ITB01'</assign>
    <assign to="xmlError{test}">'X + Y = 10'</assign>
    <assign to="reportItems{error}" append="true">$xmlError</assign>
    <!--
        Call the send step.
    -->
    <send desc="Send message" from="Actor1" to="Actor2" handler="SimulatedMessaging">
        <input name="parameters">$parameters</input>
        <input name="contentTypes">$contentTypes</input>
        <input name="reportItems">$reportItems</input>
    </send>

The ``reportSteps`` input can be used instead of, or alongside, manually created report items, to include other steps'
reports. The steps in question are identified by means of their ``id`` attributes, and can be defined as a
comma-separated string or a list, provided inline or as a :ref:`variable reference <test-case-referring-to-variables>`.

Referring to other step's reports can be particularly useful if you want to present an **aggregated report** instead of multiple
individual ones. When reports are merged, this includes all report items as well as reported context data. In the example
below we are performing multiple individual checks as hidden steps, before presenting a single aggregated report with all
findings. The result is the same but results in a more concise report.

.. code-block:: xml

    <!--
        Carry out individual validations. We're doing this in a group to avoid the test session
        completing immediately if a step fails.
    -->
    <group hidden="true" stopOnError="false">
        <verify id="checkString" handler="StringValidator" desc="Check string" hidden="true">
            <input name="actual">$aString</input>
            <input name="expected">"expected_string"</input>
            <input name="successMessage">"The provided value is correct."</input>
            <input name="failureMessage">"The provided value does not match the requirements."</input>
        </verify>
        <assign to="stepsToReport" append="true">"checkString"</assign>
        <verify id="checkExpression" handler="ExpressionValidator" desc="Validate UUID">
            <input name="expression">$variable != "unwantedValue"</input>
            <input name="successMessage">"The provided UUID is correct."</input>
            <input name="failureMessage">"The provided UUID does not match the requirements."</input>
        </verify>
        <assign to="stepsToReport" append="true">"checkExpression"</assign>
    </group>
    <!--
        Show the aggregated items in the send step's report.
    -->
    <send desc="Send message" from="Actor1" to="Actor2" handler="SimulatedMessaging">
        <!--
            The steps to report could also have been provided inline as "checkString,checkExpression".
        -->
        <input name="reportSteps">$stepsToReport</input>
    </send>

When displaying a detailed validation report, using ``reportItems``, ``reportSteps`` or both, you can order its items
based on their severity level by setting the ``sortReportBySeverity`` input to ``true()``. Otherwise, the ordering
follows the items' location information. Regarding the step's result, this is calculated based on the report items. If an
explicit result is passed by means of the ``result`` input, this is respected as long as there are no report items with
a higher severity (in which case the overall result is adapted).

.. code-block:: xml

    <send desc="Send message" from="Actor1" to="Actor2" handler="SimulatedMessaging">
        <!--
            Include manually created report items.
        -->
        <input name="reportItems">$reportItems</input>
        <!--
            Include reports from other steps.
        -->
        <input name="reportSteps">$stepsToReport</input>
        <!--
            Sort the aggregated report's items by severity.
        -->
        <input name="sortReportBySeverity">true()</input>
    </process>