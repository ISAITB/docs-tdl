The ``group`` step is a construct used to visually and behaviourally group together a sequence of steps. By itself it has no effect on
the test execution but it can be used to apply a common behaviors to all child steps. Its structure is as follows:

.. csv-table::
    :stub-columns: 1
    :header: "Name", "Required?", "Description"

    @collapsed, no, A boolean flag determining whether or not the step is displayed as initially collapsed (default is "false"). See also :ref:`tdl-steps-common-collapsed`.
    @desc, no, A description for this group to display to the user and to include in the test session log. Within scriptlets this can also be :ref:`a variable reference<scriptlets_dynamic_references>`.
    @hidden, no, A boolean flag determining whether or not the step is displayed to users (default is "false"). Note that within scriptlets this can also be :ref:`a variable reference<scriptlets_dynamic_references>`. See :ref:`tdl-steps-common-hidesteps` for further details.
    @hiddenContainer, no, A boolean flag determining whether or not the group's boundary will be displayed or not (default is "false").
    @skipped, no, A boolean value or variable reference (default being "false") which will result in the step being skipped if "true". See also :ref:`tdl-steps-common-skipped`.
    @stopOnChildError, no, A boolean flag determining whether the execution of this step's children should stop in case of a failure (default is "false"). See also :ref:`tdl-steps-common-stoponerror`.
    @stopOnError, no, A boolean flag determining whether the test session should end if this step fails (default is "false"). See also :ref:`tdl-steps-common-stoponerror`.
    @title, no, A short title to display for this step (default is "group"). Within scriptlets this can also be :ref:`a variable reference<scriptlets_dynamic_references>`.
    documentation, no, Rich text content that provides further information on the current step.

The children of the ``group`` element can be any number of steps supported by GITB TDL. The following example creates a group around a set of 
related validations.

.. code-block:: xml

    <group desc="Validate payload">
        <verify handler="XmlValidator" desc="Validate body">
            <input name="xml">$body</input>
            <input name="xsd">$bodySchema"</input>
        </verify>
        <verify handler="XmlValidator" desc="Validate header">
            <input name="xml">$header</input>
            <input name="xsd">$headerSchema"</input>
        </verify>
    </group>

Using a ``group`` can provide a useful means of structuring a test case's presentation. In addition, it allows several steps to be considered
together and determine how they are processed and presented. Specifically:

* Used with the :ref:`hidden<tdl-steps-common-hidesteps>` attribute, to completely hide a set of steps.
* Used with the :ref:`collapsed<tdl-steps-common-collapsed>` attribute, to define the group's display as initially collapsed.
* Used with the :ref:`stopOnError <tdl-steps-common-stoponerror>` and :ref:`stopOnChildError <tdl-steps-common-stoponchilderror>` attributes, to
  define how errors in child steps are managed.
* Used with the ``hiddenContainer`` attribute, to display the group's child steps but not its own boundary. This flag is particularly
  useful when you want to apply behavior to a group of steps but don't want to otherwise adapt the display.

Use of these attributes is illustrated in the following TDL snippet:

.. code-block:: xml

    <!-- 
        Hide both validations. This could be interesting to make an internal check to drive subsequent control flow.
    -->
    <group id="checkResult" hidden="true">
        <verify handler="XmlValidator" desc="Validate body" level="WARNING">...</verify>
        <verify handler="XmlValidator" desc="Validate header" level="WARNING">...</verify>
    </group>
    <!-- 
        Show the two validations in a group and present as initially collapsed.
    -->
    <group desc="Validate document" collapsed="true">
        <verify handler="XmlValidator" desc="Validate body">...</verify>
        <verify handler="XmlValidator" desc="Validate header">...</verify>
    </group>
    <!-- 
        Show the two validations in a group and present it fully (the default).
    -->
    <group desc="Validate document">
        <verify handler="XmlValidator" desc="Validate body">...</verify>
        <verify handler="XmlValidator" desc="Validate header">...</verify>
    </group>
    <!--
        Ensure all validations within the group take place regardless of failures.
    -->
    <group desc="Validate document" stopOnChildError="false">
        <verify handler="XmlValidator" desc="Validate body">...</verify>
        <verify handler="XmlValidator" desc="Validate header">...</verify>
    </group>
    <!--
        Ensure validations immediately stop execution of the group's steps, and
        hide the group's own boundary.
    -->
    <group desc="Validate document" hiddenContainer="true" stopOnChildError="true">
        <verify handler="XmlValidator" desc="Validate body">...</verify>
        <verify handler="XmlValidator" desc="Validate header">...</verify>
    </group>
