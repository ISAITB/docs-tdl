.. index:: Scriptlets
.. _scriptlets:

Scriptlets
==========

.. _scriptlets_overview:

Overview
--------

Scriptlets are reusable blocks of test steps that can be called during a test case's execution. They are similar to
function blocks in programming languages considering that:

* They can receive inputs and may produce outputs.
* They can define their own imports.
* They operate in their own isolated scope.

Extracting reusable sections of test steps is a common need in test suites where certain set of tasks are frequently
encountered. Consider for example a test suite that contains numerous test cases that at some point need to validate
certain received data against the same common validation artefacts. Rather than repeating the same set of steps and
artefact imports across all test cases, these validation steps can be defined in a scriptlet that is called wherever
necessary. Scriptlets are defined as separate XML documents within a test suite, but can also be defined
:ref:`embedded within specific test cases<scriptlets_embedded>`.

The following example is a complete scriptlet that is used to validate a provided XML document against an XML Schema
and a Schematron file. As output the scriptlet returns the name of the file's root element:

.. code-block:: xml

    <?xml version="1.0" encoding="UTF-8"?>
    <scriptlet id="validateDocument" xmlns="http://www.gitb.com/tdl/v1/">
        <imports>
            <artifact type="schema" encoding="UTF-8" name="schemaToUse">resources/aSchemaFile.xsd</artifact>
            <artifact type="schema" encoding="UTF-8" name="schematronToUse">resources/aSchematronFile.sch</artifact>
        </imports>
        <params>
            <var name="contentToValidate" type="object"/>
        </params>
        <steps>
            <verify handler="XmlValidator" desc="Validate XML structure">
                <input name="xml">$contentToValidate</input>
                <input name="xsd">$schemaToUse</input>
            </verify>
            <verify handler="XmlValidator" desc="Validate business rules">
                <input name="xml">$contentToValidate</input>
                <input name="schematron">$schematronToUse</input>
            </verify>
            <assign to="rootName" source="$contentToValidate">name(/*)</assign>
        </steps>
        <output name="rootName"/>
    </scriptlet>

Scriptlets are used in test cases or other scriptlets by means of the :ref:`call step<tdl-step-call>`. Considering the
previous example, an example call from a test case would be as follows:

.. code-block:: xml

    <!--
        Assume the scriptlet is defined in the same test suite in file "scriptlets/validateDocument.xml".
    -->
    <call id="call1" path="scriptlets/validateDocument.xml">
        <!--
            The variable "anXmlFile" contains the XML content to validate.
        -->
        <input name="contentToValidate">$anXmlFile</input>
    </call>
    <!--
        Log the XML file's root element name.
    -->
    <log>$call1{rootName}</log>

The following table provides an overview of the attributes and child elements that a ``scriptlet`` may have. A more detailed discussion per case
follows in the subsequent sections.

.. csv-table::
    :stub-columns: 1
    :header: "Name", "Required?", "Description"

    @id, Yes, A unique identifier for the scriptlet.
    metadata, No, A block containing the metadata used to describe the scriptlet.
    namespaces, No, An optional set of namespaces to define the expression languages used in the scriptlet.
    imports, No, An optional set of imports used to load additional resources.
    params, No, An optional set of parameters that the scriptlet expects as input when called.
    variables, No, An optional set of variables that are used locally within the scriptlet.
    steps, Yes, The sequence of steps that this scriptlet foresees.
    output, No, An optional set of output values that the scriptlet will return to its caller.

.. _scriptlets_elements:

Elements
--------

The following sections discuss the purpose and use of each element contained within the ``scriptlet`` definition.

.. index:: metadata (Scriptlets)
.. _scriptlets_elements_metadata:

Metadata
~~~~~~~~

The structure and content of the ``metadata`` element is identical to the one defined for the :ref:`test suite<test-suite-metadata>`
and its :ref:`test cases<test-case-metadata>`. In the case of scriptlets however, no such metadata is displayed to users
meaning that any information provided here is purely for test developers. In contrast to test suites and test cases,
the ``metadata`` block for scriptlets is optional and can be fully skipped.

For further information on this element check the :ref:`test case metadata documentation<test-case-metadata>`.

.. index:: namespaces (Scriptlets)
.. _scriptlets_elements_namespaces:

Namespaces
~~~~~~~~~~

The ``namespaces`` element is identical in structure and purpose to the one defined for :ref:`test cases<test-case-namespaces>`.
It is used to declare one or more namespace mappings (prefix to value), allowing the declared prefixes to be used in the scriptlet's
XPath expressions.

If the scriptlet is called by a test case or another scriptlet that already defines namespaces, these definitions are inherited. In case
the current scriptlet defines a namespace prefix that has already been defined, then the scriptlet's own definition overrides the inherited
one.

For further information on this element check the :ref:`test case namespaces documentation<test-case-namespaces>`.

.. index:: imports (Scriptlets)
.. _scriptlets_elements_imports:

Imports
~~~~~~~

The ``imports`` element allows the use of arbitrary resources from the same or another test suite. The purpose and structure
of this element is identical to the one defined for :ref:`test cases<test-case-imports>`. One notable difference in the case
of scriptlets however, is that imports lacking an explicit ``from`` attribute are loaded from the test suite containing the
scriptlet, which is not necessarily the same as the test suite containing the currently executing test case.

For further information on this element check the :ref:`test case imports documentation<test-case-imports>`.

.. index:: params
.. index:: name (params)
.. index:: type (params)
.. index:: optional (params)
.. index:: defaultEmpty (params)
.. _scriptlets_elements_params:

Params
~~~~~~

The ``params`` element is used to define the inputs that a scriptlet expects. Each such input is defined as a ``var``
element with the following structure:

.. csv-table::
    :stub-columns: 1
    :header: "Name", "Required?", "Description"

    @defaultEmpty, No, Whether the parameter should be set with an empty value if no corresponding input is provided when the scriptlet is called.
    @name, Yes, The name of the parameter. It is with this name that the parameter can be referenced within the scriptlet.
    @optional, No, A boolean value defining whether the parameter is optional (true) or not (false - the default if missing).
    @type, Yes, The type of the parameter. One of the GITB data types can be used (see :ref:`test-case-types`).
    value, No, One or more values for the parameter acting as the parameter's default value. More than one values are applicable in case of a ``map`` or ``list`` type.

Whenever a scriptlet is called using a :ref:`call<tdl-step-call>` step, each of its declared parameters must be matched by provided inputs.
The exception are parameters that are:

* Set with a specific default value. The parameter's value will be the provided default value.
* Defined as ``defaultEmpty``. The parameter will be defined as a variable and will be set with an empty value. This is particularly interesting
  for optional ``list`` or ``map`` parameters.
* Defined as ``optional``. 

Parameters for which defaults are specified may also be provided via inputs of a :ref:`call<tdl-step-call>` step, in which case the input overrides the default value.
Parameters are available in the scope of the scriptlet and can be used in the same way as other variables. For scriptlets that are not
:ref:`embedded within test cases<scriptlets_embedded>`, using parameters provides state from the test session's context to the scriptlet.

The following snippet illustrates a scriptlet that defines a series of parameters using the available option (see the inline comments for explanations on each case):

.. code-block:: xml

    <scriptlet id="myScriptlet" xmlns="http://www.gitb.com/tdl/v1/">
        <params>
            <!-- Required parameter that must be provided when calling the scriptlet. -->
            <var name="requiredParam" type="string"/>
            <!-- Optional parameter that if not provided when calling the scriptlet will be missing. -->
            <var name="optionalParam" type="string" optional="true"/>
            <!-- Optional parameter with a default value. -->
            <var name="optionalParamWithDefault" type="string"><value>Default</value></var>
            <!-- Optional map parameter with two default entries. -->
            <var name="optionalMapParamWithDefault" type="map">
                <value name="key1" type="string">Value1</value>
                <value name="key2" type="string">Value2</value>
            </var>
            <!-- Optional list parameter with two default entries. -->
            <var name="optionalListParamWithDefault" type="list[string]">
                <value>Value1</value>
                <value>Value2</value>
            </var>
            <!-- Optional map parameter that will be empty if not provided. -->
            <var name="optionalEmptyMapParam" type="map" defaultEmpty="true"/>
        </params>
        ...
    </scriptlet>

.. note::

    When a parameter is provided with a default value or is defined as ``defaultEmpty``, it is forcibly considered as optional.

.. index:: variables (Scriptlets)
.. _scriptlets_elements_variables:

Variables
~~~~~~~~~

The ``variables`` element can be defined to create one or more variables that will be used during the scriptlet's execution.
The purpose and structure of this element is identical to the one defined for :ref:`test cases<test-case-variables>`.
One notable point is that a scriptlet's variables, as well as all other data in its scope, is isolated from the test
case or scriptlet that is calling it, unless this is a scriptlet :ref:`embedded within a test case<scriptlets_embedded>`.

For further information on this element check the :ref:`test case variables documentation<test-case-variables>`.

.. index:: steps (Scriptlets)
.. _scriptlets_elements_steps:

Steps
~~~~~

The ``steps`` element is where the scriptlet's testing logic is implemented. It consists of a sequence of test steps, each realised by means
of a GITB TDL step construct. The purpose and structure of this element is identical to the one defined for :ref:`test cases<test-case-steps>`,
with the only notable difference being how **actors** are considered in :ref:`messaging steps<tdl-messaging-steps>`.

A test case's :ref:`actors<test-case-actors>` represent the parties involved in the test. Each actor is assigned an ``id``
which is then referenced in :ref:`messaging steps<tdl-messaging-steps>` to determine the flow of communications. A scriptlet
does not include such an actor definition block given that it is not the scriptlet's role to determine which actors
are simulated and which is the system under test (SUT). If a scriptlet includes :ref:`messaging steps<tdl-messaging-steps>`
the test developer needs to ensure that the actor IDs that are referenced will match those defined by its calling test cases.

For further information on the scriptlet's ``steps`` element and the different constructs it can include check the :ref:`test case steps documentation<test-case-steps>`.

.. index:: output (Scriptlets)
.. index:: name (Scriptlet output)
.. index:: lang (Scriptlet output)
.. index:: source (Scriptlet output)
.. index:: asTemplate (Scriptlet output)
.. _scriptlets_elements_output:

Output
~~~~~~

A scriptlet's results are returned by the ``output`` elements that it defines. These outputs are the only means a scriptlet
has to communicate information back to its caller, be it a test case or another scriptlet, but can be omitted in case
no results are expected. The structure of each ``output`` element is as follows:

.. csv-table::
    :header: "Name", "Required?", "Description"
    :stub-columns: 1
    :delim: |

    @name | Yes | A name with which the output can be referenced.
    @lang | No | The expression language prefix to use to evaluate the contained expression (see :ref:`test-case-namespaces` and :ref:`test-case-expressions`).
    @source | No | A variable reference to identify a source ``object`` variable upon which the contained expression should be evaluated.
    @asTemplate | No | Whether or not the result will be considered as a template for placeholder replacement (see :ref:`test-case-expressions-template-files`). By default this is "false".

The content of the ``output`` element is an :ref:`expression<test-case-expressions>` that is used once the scriptlet's processing
is complete to calculate the output's value. In the special case where a result needs to be calculated from an XPath expression
on an XML document, the ``source`` attribute can be used to define the document to consider. The value returned by an ``output``
element is calculated as follows:

#. If the ``output`` element is non-empty, its content is used as an expression to calculate the return value.
#. If the ``output`` element is empty the ``name`` of the ``output`` element must match the name of a variable present
   in the scriptlet's scope. If matched, the value of the variable is returned.

Taking the sample scriptlet presented previously, we want to return from the scriptlet a single output named "rootName"
which is set with the validated document's root element name. A first way to achieve this is to reference a variable
from the scriptlet's scope that contains the calculated value.

.. code-block:: xml

    <scriptlet id="validateDocument" xmlns="http://www.gitb.com/tdl/v1/">
        <steps>
            ...
            <!--
                Calculate the document's root name and assign it to a variable named "rootName".
            -->
            <assign to="rootName" source="$contentToValidate">name(/*)</assign>
        </steps>
        <!--
            Reference the "rootName" variable from the scriptlet's scope.
        -->
        <output name="rootName"/>
    </scriptlet>

Using this approach we treat the ``output`` element simply as a definition for which we need to ensure that, once complete,
the scriptlet's scope will include a similarly named variable. Note how the ``output`` element is itself empty given that
it should not evaluate any expression to return its value. An alternative approach would be to use this expression to
define the value as follows:

.. code-block:: xml

    <scriptlet id="validateDocument" xmlns="http://www.gitb.com/tdl/v1/">
        <steps>
            ...
            <assign to="documentRootName" source="$contentToValidate">name(/*)</assign>
        </steps>
        <!--
            Reference the "documentRootName" variable from the scriptlet's scope.
        -->
        <output name="rootName">$documentRootName</output>
    </scriptlet>

This uses a simple :ref:`variable expression<test-case-referring-to-variables>` and is largely equivalent. We could of course
provide a more elaborate :ref:`expression<test-case-expressions>` here to e.g. enclose the value in square brackets:

.. code-block:: xml

    <output name="rootName">concat('[', $documentRootName, ']')</output>

Finally, when using the ``output`` element's expression to calculate its return value we could in our example skip the previous
:ref:`assign<tdl-step-assign>` step altogether as follows:

.. code-block:: xml

    <scriptlet id="validateDocument" xmlns="http://www.gitb.com/tdl/v1/">
        <steps>
            ...
        </steps>
        <!--
            Calculate the value to return in the output element itself.
        -->
        <output name="rootName" source="$contentToValidate">name(/*)</output>
    </scriptlet>

Keep in mind that you may return any number of ``output`` elements you want. As an example, we could adapt our scriptlet
to return also information on the result of the individual checks it made, in case the caller wants to take specific actions
per case.

.. code-block:: xml

    <scriptlet id="validateDocument" xmlns="http://www.gitb.com/tdl/v1/">
        <steps>
            <verify id="xsdCheck" handler="XmlValidator" desc="Validate XML structure">
                ...
            </verify>
            <verify id="schCheck" handler="XmlValidator" desc="Validate business rules">
                ...
            </verify>
        </steps>
        <!--
            Return the name of the document's root element and the individual validation steps' results.
        -->
        <output name="rootName" source="$contentToValidate">name(/*)</output>
        <output name="isValidForSchema">$xsdCheck</output>
        <output name="isValidForSchematron">$schCheck</output>
    </scriptlet>

In this example note how the :ref:`verify<tdl-step-verify>` steps are set with a specific ``id``. Once each validation is
completed the overall results are recorded as ``boolean`` variables which are then used to return the scriptlet's outputs
(see the :ref:`verify<tdl-step-verify>` step documentation for further details on this).

.. note::
    **Selecting outputs:** When a scriptlet returns multiple outputs its caller may choose to select only a subset of these to use. For details
    on this and how returned outputs can be used check the :ref:`call<tdl-step-call>` step's documentation.

.. _scriptlets_dynamic_references:

Dynamic presentation within scriptlets
--------------------------------------

Certain information relative to presenting :ref:`test steps<tdl-steps>` are expected to have fixed values that are known before the a test session is
executed. Such information includes step descriptions, titles and actor references (e.g. used in :ref:`messaging steps<tdl-messaging-steps>`)
which remain unchanged during a test session to allow a consistent execution diagram to be presented to the user.

This rule is relaxed in the case of scriptlets given that they can be used in various contexts. They can be used by multiple different 
test cases, potentially spanning several test suites, or even used from within other scriptlets (essentially anywhere you can have a 
:ref:`call step<tdl-step-call>`). To enable this flexibility, the rule for otherwise constant values in scriptlets is that these can be
set via :ref:`variable references<test-case-referring-to-variables>` as long as their value can be determined when a test case's definition
is initially loaded.

The cases where otherwise fixed values can be set via variable reference are:

* The ``desc`` attribute of all steps.
* The ``title`` attribute of all steps with child steps (:ref:`group<tdl-step-group>`, :ref:`if<tdl-step-if>`, :ref:`flow<tdl-step-flow>`,
  :ref:`foreach<tdl-step-foreach>`, :ref:`repuntil<tdl-step-repuntil>`, :ref:`while<tdl-step-while>`).
* The ``inputTitle`` of user interaction steps (:ref:`interact<tdl-step-interact>`).
* The ``from`` and ``to`` actor references in all messaging and interaction steps (:ref:`btxn<tdl-step-btxn>`, :ref:`send<tdl-step-send>`,
  :ref:`receive<tdl-step-receive>`, :ref:`interact<tdl-step-interact>`).
* The ``reply`` attribute of messaging steps (:ref:`tdl-step-send`, :ref:`tdl-step-receive`, :ref:`tdl-step-listen`).
* The ``hidden`` attribute of all steps (see also how this can be used to :ref:`dynamically adapt the content of scriptlets<scriptlets_dynamic_steps>`).

Values set in this way need to be provided as **inputs** to the scriptlet and resolve to constants before the test begins. In practical terms this
means that you will need to:

1. Define in the scriptlet's :ref:`params section<scriptlets_elements_params>` the relevant input(s).
2. Reference the parameters within the scriptlet where you want to use them.
3. Pass as fixed values or :ref:`configuration value references<test-case-configuration>` the values for the parameters when you 
   :ref:`call the scriptlet<tdl-step-call>`.

As an example of this consider the following scriptlet:

.. code-block:: xml

    <scriptlet id="receiveData" xmlns="http://www.gitb.com/tdl/v1/">
        <params>
            <var name="descriptionToUse" type="string"/>
            <var name="from" type="string"/>
            <var name="to" type="string"/>
        </params>
        <steps>
            ...
            <receive id="receiveStep" desc="$descriptionToUse" from="$from" to="$to" txnId="t1">
                <input name="countryCode">$ORGANISATION{countryCode}</input>
            </receive>
            ...
        </steps>
    </scriptlet>

The ``desc``, ``from`` and ``to`` in this case are set dynamically based on the values passed as parameters. In a test case that
will make use of this scriptlet we then add a :ref:`call<tdl-step-call>` step as follows:

.. code-block:: xml

    <call path="scriptlets/receiveData.xml">
        <input name="descriptionToUse">'Receive a message'</input>
        <input name="from">'Actor1'</input>
        <input name="to">'Actor2'</input>
    </call>

Notice here how the parameters defined in the scriptlet are supplied with constant values. This allows the test engine to calculate a
specific test execution graph when the test case is loaded but provides the flexibility for the scriptlet to be used in various
scenarios.

.. index:: hidden
.. index:: static
.. _scriptlets_dynamic_steps:

Dynamic steps within scriptlets
-------------------------------

Similar to :ref:`dynamically changing labels and actors<scriptlets_dynamic_references>`, a scriptlet can also have its contained
steps be dynamically defined. This is achieved via specific ``boolean`` flags, covered in this section, that in the case of
scriptlets can be set as :ref:`variable references<test-case-referring-to-variables>` as long as their values can be determined
at test case load time and remain constant during execution. Such flags, defining the visual representation of a test case, need
to remain fixed given that a test case's presentation cannot change during test execution.

Dynamically changing a scriptlet's contained steps can be achieved via the following flags:

* The ``hidden`` attribute :ref:`supported by all test steps<tdl-steps-common-hidesteps>`. This allows you to hide steps while
  ensuring they **will still be executed**.
* The ``static`` attribute of :ref:`if steps<tdl-step-if>`. This allows you to conditionally include steps, ensuring that non-included
  steps are **not executed**.

In the case of the ``hidden`` attribute, setting this to "true" will result in a step being executed but not displayed. 
Setting this to a :ref:`variable reference<test-case-referring-to-variables>` is allowed only within scriptlets
in which case the variable needs to match one of the scriptlet's :ref:`parameters<scriptlets_elements_params>`. The variable reference
is evaluated when the test case is loaded, with the resulting value being determined either from the 
scriptlet's inputs (provided via its :ref:`call step<tdl-step-call>`) or the default value defined for the parameter.

The :ref:`if step's<tdl-step-if>` ``static`` attribute goes further, allowing you to fully skip sets of steps. In this
case, when ``static`` is set to "true" the test engine will expect to find the step's condition (its ``cond`` element) set
with a :ref:`variable reference<test-case-referring-to-variables>`. It is this variable reference that is then evaluated at load
time to determine whether to include the steps in the ``then`` block (if "true"), or the ``else`` block (if "false" and if defined).
One important additional point here is that a statically evaluated :ref:`if step<tdl-step-if>` **does not display a boundary box and title**,
but rather directly displays the steps of the selected branch. This means that you can use a static ``if`` as other languages use
"include" and "import" constructs.

.. note::
    A similar result to an :ref:`if step<tdl-step-if>` defined as ``static`` can be achieved by using a hidden ``if`` step with an
    explicitly visible ``then`` block. Check the :ref:`if step's documentation<tdl-step-if>` for more details.

The cases described above are presented in the sample scriptlet below to highlight their use.

.. code-block:: xml

    <scriptlet id="receiveData" xmlns="http://www.gitb.com/tdl/v1/">
        <params>
            <!-- Parameter that must be specified as an input. -->
            <var name="hideMessage" type="boolean"/>
            <!-- Parameter that may be optionally specified as an input. -->
            <var name="validateAsExpression" type="boolean">
                <value>true</value>
            </var>
        </params>
        <steps>
            ...
            <!-- If 'hideMessage' is set to true, this message exchange will take place but will not be displayed. -->
            <receive hidden="$hideMessage" id="receiveStep" desc="Receive message" from="Actor1" to="Actor2" txnId="t1">
                <input name="countryCode">$ORGANISATION{countryCode}</input>
            </receive>
            <!-- 
                Static 'if' to determine the steps used to process the received message.
                No 'if' boundary will be displayed and the steps of the rejected branch will be excluded.
            -->
            <if static="true">
                <cond>$validateAsExpression</cond>
                <then>
                    <log>"Validating input as a TDL expression"</log>
                    <verify handler="ExpressionValidator" desc="Validate value">
                        <input name="expression">$value = $expectedValue</input>
                    </verify>
                </then>
                <else>
                    <log>"Validating input as a regular expression"</log>
                    <verify handler="RegExpValidator" desc="Validate value">
                        <input name="input">$value</input>
                        <input name="expression">'^REF\-\d+$'</input>
                    </verify>    
                </else>
            </if>             
            ...
        </steps>
    </scriptlet>

As you see the scriptlet expects two parameters, ``hideMessage`` and ``validateAsExpression``, that determine the display
and content of the scriptlet. They are used respectively in the :ref:`receive step's<tdl-step-receive>` ``hidden`` attribute,
and the condition of the :ref:`if step<tdl-step-if>` that is marked as ``static``.

When calling this scriptlet we need to ensure that both these variables can be evaluated at load time. The ``validateAsExpression``
parameter already has a default value, so a :ref:`call step<tdl-step-call>` needs to only override it if needed. The following
example illustrates this (note how ``false()`` is used as opposed to "false" given that this is an :ref:`expression<test-case-expressions>`).

.. code-block:: xml

    <call path="scriptlets/receiveData.xml">
        <input name="hideMessage">false()</input>
        <!--
            No need to specify 'validateAsExpression' as true given
            that this is already the default.

            <input name="validateAsExpression">true()</input>
        -->
    </call>

.. _scriptlets_embedded:

Scriptlets embedded in test cases
---------------------------------

Scriptlets are typically defined as separate XML documents within a test suite that can be used across its test cases or
externally from other test suites. It is also possible however, to define scriptlets within a specific test case in its
:ref:`scriptlets<test-case-scriptlets>` element. Such embedded scriptlets are processed in the same way but have certain
key differences:

* **Scope inheritance:** The scope of an embedded scriptlet extends, without replacing, the scope of its included test case.
  This means that looking up a variable within the scriptlet will first check the scriptlet's scope and, if not found, will
  check the test case's scope. Across multiple embedded scriptlets however, scopes remain isolated.
* **Private access:** Embedded scriptlets are only usable from the test case that contains them and from its other embedded
  scriptlets.
* **Identification:** Embedded scriptlets rely on their ``id`` attribute to be identified and referenced in :ref:`call<tdl-step-call>`
  steps. As such, this must be unique within the scope of their containing test case.

For more details on defining embedded scriptlets check the documentation of the test case's :ref:`scriptlets<test-case-scriptlets>`
element. For information on how a scriptlet, embedded or not, is called check the documentation of the :ref:`call<tdl-step-call>`
step.