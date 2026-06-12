The ``call`` step is used to invoke a set of steps defined as a ``scriptlet`` (see :ref:`scriptlets`). If we consider that a scriptlet resembles a function
with input, output and local variables, the ``call`` step can be considered as the function's invocation. Its purpose is to identify the ``scriptlet`` to call, pass
its required input parameters and receive its output. The structure of the ``call`` element is as follows:

.. csv-table::
    :stub-columns: 1
    :header: "Name", "Required?", "Description"

    @from, no, The identifier of the test suite from which the scriptlet will be loaded. If not provided this is assumed to be the current test suite.
    @hidden, no, A boolean flag determining whether or not the step is displayed to users (default is "false"). Note that within scriptlets this can also be :ref:`a variable reference<scriptlets_dynamic_references>`. See :ref:`tdl-steps-common-hidesteps` for further details.
    @id, no, The ID for the step. This is also the name of a ``map`` variable in the session context in which output will be stored.
    @input, no, An alternative to input elements to provide a single input when the scriptlet expects a single input or (if multiple) a single mandatory input. See also :ref:`tdl-step-call__simplified`.
    @output, no, The name to use for the session context variable to store the scriptlet output as an alternative to using the ``id``. See also :ref:`tdl-step-call__simplified`.
    @path, yes, The identifier scriptlet to call. The value provided here depends on the whether the scriptlet is :ref:`external to the test case<scriptlets>` or :ref:`defined within it<test-case-scriptlets>`.
    @skipped, no, A boolean value or variable reference (default being "false") which will result in the step being skipped if "true". See also :ref:`tdl-steps-common-skipped`.
    @stopOnError, no, A boolean flag determining whether the test session should end if this step fails (default is "false"). See also :ref:`tdl-steps-common-stoponerror`.
    input, no, Zero or more elements for the ``scriptlet``'s input parameters. See :ref:`handlers-inputs-outputs` for details.
    output, no, Zero or more elements for the ``scriptlet``'s output parameters to specify which outputs you require.

Scriptlets can be defined in :ref:`separate XML files<scriptlets>`, in which case they can be used by any test case, or as
:ref:`internal to a specific test case<test-case-scriptlets>`, in which case they are considered private. How a scriptlet
is looked up depends on its type, which defines how the ``path`` and ``from`` attributes are used. Specifically:

* **Internal scriptlet:** The ``path`` attribute is set with the ``id`` value of the scriptlet to call and the ``from`` attribute is omitted.
* **External scriptlet:** The ``from`` attribute is set with the ``id`` of the test suite to load the scriptlet from, and
  the ``path`` is set with the file path to the scriptlet's XML file (relative to the test suite's root).

If the ``from`` attribute is not specified, the test engine first attempts to load the scriptlet from the ones defined
within the test case, by matching the ``path`` value against the defined scriptlets' ``id``. If no match is found a
further lookup is made within the test case's containing test suite, in which case the ``path`` value is considered as
the path to the scriptlet's XML file (relative to the test suite root or, as a fallback, to the test suite definition file).
When the ``from`` attribute is specified the scriptlet is always considered to be
external to the test case, and its value is considered to be a test suite's ``id``. The lookup in this case proceeds as
follows:

#. If the value matches the current **test suite** ``id``, the lookup is made within the current test suite.
#. If not found, the lookup for a matching test suite continues within the current test suite's **specification**.
#. If not found, the lookup for a matching test suite continues within the current test suite's overall **domain**.

.. note::
    **Non-unique test suite IDs:** If multiple test suites are matched during a scriptlet's lookup, an arbitrary test suite
    will be considered. Ensure that test suites sharing common resources have a unique ``id``. A test suite's ``id`` is
    always unique within a specification but not necessarily across specifications (i.e. within the overall domain).

The following example ``call`` steps, illustrate different cases of scriptlet lookup:

.. code-block:: xml

    <!--
        Look for a scriptlet with id "script1" within the test case.
        If not found look for a file "script1" within the test case's test suite.
    -->
    <call id="call1" path="script1"/>
    <!--
        Look for the scriptlet in test suite "test_suite_1" and load it from file "scriptlets/script1.xml".
    -->
    <call id="call2" from="test_suite_1" path="scriptlets/script1.xml"/>

Once the target scriptlet has been located, the ``call`` step will calculate and pass any ``inputs`` it requires. The approach
to pass inputs is identical to the case of :ref:`inputs to handlers<handlers-inputs-outputs>`. Values can be provided as
constants or results of :ref:`expressions<test-case-expressions>`, and can optionally be considered as :ref:`templates<test-case-expressions-template-files>`
with placeholder substitutions. It is important to note that all scriptlet inputs are required; failure to provide one or
more inputs will result in a test session error.

Once a scriptlet completes, its :ref:`outputs<scriptlets_elements_output>` are recorded in a ``map`` stored in the
test session's context, that is named using the ``call`` step's ``id``. Individual outputs can then be referred
to from within this ``map`` using their name.

A ``call`` step may choose to ignore specific scriptlet outputs. This can be done by listing the specific outputs you are
interested in, naming them as part of the ``call`` step's ``output`` elements. Any outputs that don't match the listed ones
will then be discarded. Note that when the ``call`` step does not define specific ``output`` elements, all scriptlet outputs are
returned by default.

The following example illustrates potential uses of the ``call`` step:

.. code-block:: xml

    <!--
        Call a scriptlet defined within the test case and retrieve all its output.
    -->
    <call id="internalCall" path="script1">
        <input name="docToValidate">$fileContent1</input>
    </call>
    <!--
        Call a scriptlet defined in test suite "test_suite_1" and retrieve only its "outputMessage" output.
    -->
    <call id="externalCall" from="test_suite_1" path="scriptlets/script1.xml">
        <input name="docToValidate">$fileContent1</input>
        <output name="outputMessage"/>
    </call>

Further information on defining and using scriptlets is provided in the :ref:`scriptlet documentation<scriptlets>`. For
scriptlets specifically defined within test cases (i.e. private scriptlets) refer to the test case's
:ref:`scriptlets element<test-case-scriptlets>`.

.. _tdl-step-call__simplified:

Simplified call steps
+++++++++++++++++++++

Test cases often include scriptlets as utilities that don't need multiple inputs or produce only single output values. To reduce the 
verbosity of the ``call`` step in such cases, you can make use of two syntax alternatives:

    * The ``input`` attribute to provide a single input. This is possible when the scriptlet expects only a single input.
    * The ``output`` attribute to directly name the result rather than use an intermediate ``map``.

Defining an input both as an attribute and child element is superfluous. If nonetheless both are defined, the child elements take precedence.
On the other hand, the ``output`` attribute is complementary to the output child elements. When defining output child elements these result
in limiting the produced results to only the ones specified. The results are first filtered as such before using the ``output`` attribute's
value to name the resulting variable.

The following example illustrates how these alternatives can be used to simplify your test definitions. We consider here that we are 
generating two messages based on a template that includes a placeholder for a signature (named "signature"). For the first message
we use a verbose syntax whereas for the second one we use the simplifications discussed here. In both cases the signature value is created
through a scriptlet that expects an input named "valueToSign" and produces an output named "signedValue".

.. code-block:: xml

    <!--
        Verbose approach.
    -->
    <call id="signatureCall" path="signatureScript">
        <input name="valueToSign">$aValue</input>
    </call>
    <!-- 
        The output is stored in a map named using the step's id. As the template defines
        a "signature" placeholder we need to create such a variable from the result map.
    -->
    <assign to="signature">$signatureScript{signedValue}</assign>
    <assign to="message1" asTemplate="true">$messageTemplate</assign>

    <!--
        Simplified approach.
    -->
    <call output="signature" path="signatureScript" input="$aValue"/>
    <assign to="message2" asTemplate="true">$messageTemplate</assign>

.. note::
    The :ref:`tdl-step-process` step also offers :ref:`similar syntax simplifications<tdl-step-process__simplified>`. This simplified
    syntax is available for :ref:`tdl-step-process` and :ref:`tdl-step-call` steps as these typically represent utilities that are
    frequently used.
