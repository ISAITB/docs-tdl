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
    ``parameters`` | No | A ``map`` including the values to display (labelled using the ``map`` keys).
    ``reportItems`` | No | A ``map`` of report items to display as a detailed validation report.
    ``reportSteps`` | No | A ``list`` of step identifiers from which to source the current step's detailed validation report.
    ``result`` | No | A ``string`` with the status (``SUCCESS``, ``FAILURE`` or ``WARNING``) to use for the relevant :ref:`process<tdl-step-process>` step (default is ``SUCCESS``).
    ``sortReportBySeverity`` | No | A ``boolean`` flag (false by default) specifying whether report items should be sorted based on severity first and then location.

The following example illustrates usage of the ``DisplayProcessor`` to create a step report for a given set of data that the user may
choose to view:

.. code-block:: xml

    <!--
        Display a report based on a set of parameters.
    -->
    <assign to="parameters{textValue}">'A sample value'</assign>
    <assign to="parameters{listValues}" append="true">'Value 1'</assign>
    <assign to="parameters{listValues}" append="true">'Value 2'</assign>
    <assign to="parameters{listValues}" append="true">`Value 3`</assign>
    <process desc="Show values" hidden="false" handler="DisplayProcessor" input="$parameters"/>
    <!--
        Display a report but also mark the step as failed if we have errors.
    -->
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
    <process desc="Process data" hidden="false" handler="DisplayProcessor">
        <input name="parameters">$params</input>
        <input name="contentTypes">$contentTypes</input>
    </process>

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
    <process desc="Process data" hidden="false" handler="DisplayProcessor">
        <input name="parameters">$params</input>
        <input name="contentTypes">$contentTypes</input>
    </process>

The following example illustrates how content types can be provided when the parameters are defined within complex
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
        Call the process step.
    -->
    <process desc="Process data" hidden="false" handler="DisplayProcessor">
        <input name="parameters">$params</input>
        <input name="contentTypes">$types</input>
    </process>

Besides sharing data, the ``DisplayProcessor`` can also be used to display a **detailed validation report**.
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
        Produce the report.
    -->
    <process desc="Show report" hidden="false" handler="DisplayProcessor">
        <input name="parameters">$parameters</input>
        <input name="contentTypes">$contentTypes</input>
        <input name="reportItems">$reportItems</input>
    </process>

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
        Show the aggregated report.
    -->
    <process desc="Show report" hidden="false" handler="DisplayProcessor">
        <!--
            The steps to report could also have been provided inline as "checkString,checkExpression".
        -->
        <input name="reportSteps">$stepsToReport</input>
    </process>

When displaying a detailed validation report, using ``reportItems``, ``reportSteps`` or both, you can order its items
based on their severity level by setting the ``sortReportBySeverity`` input to ``true()``. Otherwise, the ordering
follows the items' location information. Regarding the step's result, this is calculated based on the report items. If an
explicit result is passed by means of the ``result`` input, this is respected as long as there are no report items with
a higher severity (in which case the overall result is adapted).

.. code-block:: xml

    <process desc="Show report" hidden="false" handler="DisplayProcessor">
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

.. note::
    **DisplayProcessor in non-hidden steps:** :ref:`Process steps<tdl-step-process>` are by default set as ``hidden``, meaning
    that they execute but are not displayed and do not produce a visible report. When using the ``DisplayProcessor`` you need
    to ensure that ``hidden`` is set to ``false`` for its use to be meaningful.