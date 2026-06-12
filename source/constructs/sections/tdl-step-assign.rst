The ``assign`` step is a frequently used construct in GITB TDL. It is a step that is not visible to the user, used for the manipulation 
of data in the test session's context. It can be used to assign values to variables but also as a means of 
performing simple processing or :ref:`conversion between data types <test-case-types-type-conversions>`. 
The processing and assignment result is determined by an :ref:`expression <test-case-expressions>` provided as the text content of the ``assign`` element. 
The element's structure is as follows:

.. csv-table::
    :stub-columns: 1
    :header: "Name", "Required?", "Description"

    @append, no, Used if the ``to`` variable is a ``list`` to append the result to. Can be "true" or "false".
    @asTemplate, no, Whether or not the result will be considered as a :ref:`template for placeholder replacement <test-case-expressions-template-files>`. By default this is "false".
    @byValue, no, Whether adding a value to a ``map`` or ``list`` will be by-value as opposed to by-reference. By default this is "false" (i.e., by-reference).
    @lang, no, Not used currently (and defaulting to XPath as the built-in :ref:`expression language <test-case-expressions>`).
    @skipped, no, A boolean value or variable reference (default being "false") which will result in the step being skipped if "true". See also :ref:`tdl-steps-common-skipped`.
    @source, no, A variable reference to identify a source ``object`` variable upon which the expression should be evaluated.
    @stopOnError, no, A boolean flag determining whether the test session should end if this step fails (default is "false"). See also :ref:`tdl-steps-common-stoponerror`.
    @to, yes, The target variable to assign the result of the expression to.
    @type, no, Used to explicitly specify the type of variable to create (e.g. if the ``to`` is an entry in a ``map``).

The ``to`` attribute of an ``assign`` step determines the target variable to which the expression's output will be assigned to. This can be:

    * An **existing variable**, defined in the test case's :ref:`variables<test-case-variables>` section or from previous steps.
    * A **new variable** that will be created once this step completes.

You typically specify the ``to`` attribute with the target variable name (e.g. ``myVariable``) in which case the variable will either be set if it 
already exists, or created otherwise. You can make this behaviour explicit by prefixing the attribute name with a dollar sign (``$``),
in which case it becomes a :ref:`variable reference <test-case-referring-to-variables>` (e.g. ``$myVariable``). In this case, if the referenced
variable does not already exist the test case will fail. You will likely never need to use a variable reference in an assign step, unless you 
have very specific needs (such as :ref:`referencing a variable from a scriptlet's parent scope <scriptlets_scope>`).

When defining a new variable its type is determined based on the result of the expression. This can also be affected by additional context information
from the way the ``assign`` step is used, specifically the ``append`` attribute that would suggest a ``list``, as well as the ``to`` expression that 
could suggest a ``map`` (e.g. if this defines ``myMap{myKey}``). You can also explicitly define the variable's type by means of the
``type`` attribute.

Given that the GITB TDL's built-in :ref:`expression language <test-case-expressions>` is XPath, the ``assign`` step can also be leveraged
to complete more advanced tasks. Specifically:

    * Make **XPath lookups** in XML content (replacing use of the :ref:`XPathProcessor <handlers-XPathProcessor>`).
    * Make **conditional variable assignments** (replacing use of :ref:`if steps <tdl-step-if>`).

The following snippet illustrates simple and more advanced use cases of the ``assign`` step:

.. code-block:: xml

    <!-- 
        Assign a text to a string variable.
    -->
    <assign to="value1">"My value"</assign>
    <!--
        Assign a number to a number variable.
    -->
    <assign to="value2" type="number">1</assign>
    <!--
        Increment a number.
    -->
    <assign to="value2">$value2 + 1</assign>
    <!--
        Assign a variable's value conditionally (no need for an if step to do this).
    -->
    <assign to="value3">if ($flag) then "Value 1" else "Value 2"</assign>
    <!--
        Lookup a text value from an XML document (no need for a process step and XPathProcessor).
    -->
    <assign to="value4" source="$xmlContent">//po:shipTo[@country = "BE"]/po:name</assign>
    <!--
        Lookup an XML block from an XML document.
    -->
    <assign to="value5" source="$xmlContent" type="object">//po:shipTo[@country = "BE"]</assign>

Further examples can be found in the documentation on :ref:`expressions<test-case-expressions>`. Examples are also provided 
here on how variables are :ref:`dynamically created<test-case-variables-from-expression-output>` if not already defined.

.. index:: byValue (assign)

.. _tdl-step-assign-by-value:

Assignment by-reference and by-value
++++++++++++++++++++++++++++++++++++

The assignment carried out by the ``assign`` step is in almost all cases done **by-value**. This means that the step's expression is calculated
and the resulting value is assigned to the target variable. 

The exception to this rule is when the target value is a **container type**, specifically a ``map`` or ``list``, and the step's expression is a :ref:`variable reference <test-case-referring-to-variables>`.
In this case the referenced variable is looked up and added **by-reference** to the target ``map`` or ``list``. This means that if following the 
assignment the referenced variable changes value, this will also be reflected in the assigned value of the ``map`` or ``list``.
Although this may seem inconsistent it follows the typical practice in most programming languages. A ``map`` or ``list`` is a complex type that
wraps references to its contained objects, whereas other types are treated like primitives and are always assigned by value.

In case you are assigning to a ``map`` or a ``list`` you can override this default behaviour by using the ``byValue`` attribute. Setting this to
"true" in this case you can force the test engine to use a copy of the referenced variable's value instead of assigning the variable itself. Note 
that when the ``assign`` step targets a non-container variable, this attribute is ignored as the assignment is always by-value.

To summarise, the assignment behaviour of the ``assign`` step follows these rules (in sequence):

#. If the step's expression is not a :ref:`variable reference <test-case-referring-to-variables>`, the assignment is **by-value**.
#. Otherwise, if the the target variable is not a ``map`` or ``list``, the assignment is **by-value**.
#. Otherwise, if ``byValue`` is set to "true", the assignment is **by-value**.
#. Otherwise the assignment is **by-reference**.

The following example illustrates the above with various assignment cases:

.. code-block:: xml

    <assign to="variable1">"v1"</assign>
    <assign to="variable2">"v2"</assign>
    <!-- 
        Prints "Values: v1 v2".
    -->
    <log>"Values: " || $variable1 || " " || $variable2</log>

    <assign to="list" append="true">$variable1</assign>
    <assign to="list" append="true" byValue="true">$variable2</assign>
    <!--
        Prints "List: v1 v2".
    -->
    <log>"List: " || $list{0} || " " || $list{1}</log>

    <assign to="map{key1}">$variable1</assign>
    <assign to="map{key2}" byValue="true">$variable2</assign>
    <!--
        Prints "Map: v1 v2".
    -->
    <log>"Map: " || $map{key1} || " " || $map{key2}</log>

    <assign to="variable1">"v1_updated"</assign>
    <assign to="variable2">"v2_updated"</assign>
    <!--
        Prints "Values: v1_updated v2_updated".
    -->
    <log>"Values: " || $variable1 || " " || $variable2</log>
    <!--
        Prints "List: v1_updated v2".
        The second list item was not updated as the assignment was by-value.
    -->
    <log>"List: " || $list{0} || " " || $list{1}</log>
    <!--
        Prints "Map: v1_updated v2".
        The second map entry was not updated as the assignment was by-value.
    -->
    <log>"Map: " || $map{key1} || " " || $map{key2}</log>
