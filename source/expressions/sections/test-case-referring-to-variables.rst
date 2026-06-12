As you have seen in the previous examples, referring to variables is a very common use case in GITB TDL expressions. Variable
references are done as ``$VARIABLE``, i.e. using the ``$`` character, followed by the variable's name. Furthermore, when
an expression consists only of a variable reference without other XPath elements it is referred to as a **pure variable reference**.
As we have discussed in :ref:`test-case-types-type-conversions`, pure variable references are important when we need to convert 
variables from one type to another in ``assign`` steps.

The following examples show several variable reference cases used for illustration in a :ref:`log step <tdl-step-log>`:

.. code-block:: xml

    <!-- 
        Reference a simple variable from the current scope.
    -->
    <log>$myVariable</log>
    <!-- 
        Reference a map's entry from the current scope.
    -->
    <log>$myMap{key1}</log>
    <!-- 
        Reference a list's item from the current scope.
    -->
    <log>$myList{0}</log>
    <!-- 
        Reference a configuration properties at domain, organisation, system and statement (actor) level.
    -->
    <log>$DOMAIN{myDomainParameter}</log>
    <log>$ORGANISATION{myOrganisationParameter}</log>
    <log>$SYSTEM{mySystemParameter}</log>
    <log>$MyActor{myStatementParameter}</log>
    <!--
        Reference build-in test status information.
    -->
    <log>$STEP_STATUS{myStep}</log>
    <log>$STEP_SUCCESS{myStep}</log>
    <log>$TEST_SUCCESS</log>
    <!--
        Reference build-in test session metadata.
    -->
    <log>$SESSION{sessionId}</log>
    <log>$SESSION{testCaseId}</log>
    <log>$SESSION{testEngineVersion}</log>

.. _test-case-referring-to-variables_map:

Map elements
++++++++++++

Variables of type ``map`` represent special cases as they can contain additional variables and even additional ``map`` s. Referring to 
``map`` variables themselves is done as with any other variable (i.e. ``$myMap``), but referring to entries in the ``map`` are done
by specifying the key in curly brackets (i.e. ``$myMap{myKey}``). In the case a ``map`` contains a nested ``map`` its inner values
can be referenced by appending to the expression an additional set of curly braces with the inner ``map``'s key. There is no limit to 
the nesting that is possible in a ``map``. In addition note that when specifying the keys you may also use variable references
if the key is to be determined dynamically.

Assigning values to a ``map`` variable is achieved using the ``assign`` step (see :ref:`tdl-step-assign`) or as part of the variable's 
declaration in the test case (see :ref:`test-case-variables`). Using this we can either assign values to the ``map``
itself (i.e. point it to another ``map``) or one of its keys. Moreover, assignment to a ``map`` key may be done to an existing 
entry or by defining a new one. Note that when assigning a new ``map`` entry you also need to specify the ``type`` of this entry.
The following examples illustrate ``assign`` steps that showcase the possible assignment and reference scenarios:

.. code-block:: xml

    <!-- 
        Assign map "myMap" to another map named "anotherMap". 
    -->
    <assign to="myMap">$anotherMap</assign>
    <!-- 
        Assign the entry of "myMap" named "myKey" to the string "A value". 
    -->
    <assign to="myMap{myKey}" type="string">"A value"</assign>
    <!-- 
        Assign the entry of "myMap" named "myOtherKey" to an entry of "anotherMap" 
        named "anotherKey" as a string. 
    -->
    <assign to="myMap{myOtherKey}" type="string">$anotherMap{anotherKey}</assign>
    <!-- 
        Assign value "key1" to the string variable "k1". 
        Then use the "k1" variable to pick the target entry of "myMap" 
        to set as the string "A value".
    -->
    <assign to="k1">"key1"</assign>
    <assign to="myMap{k1}" type="string">"A value"</assign>
    <!-- 
        Assign value "key1" to the string variable "k1". 
        Assign value "key2" to the string variable "k2". 
        Then use the "k1" variable to pick the target entry of "myMap" to set as a string matching 
        the entry of "anotherMap" for the key matching the value of "k2".
    -->
    <assign to="k1">"key1"</assign>
    <assign to="k2">"key2"</assign>
    <assign to="myMap{k1}" type="string">$anotherMap{$k2}</assign>
    <!-- 
        Assume that "myMap" is a map that contains a nested map named "myNestedMap" that itself 
        contains a nested map named "myFurtherNestedMap". Set key "myKey" of "myFurtherNestedMap"
        a the string "A value".
    -->
    <assign to="myMap{myNestedMap}{myFurtherNestedMap}{myKey}" type="string">"A value"</assign>

.. _test-case-referring-to-variables_list:

List variable elements
++++++++++++++++++++++

Variables of type ``list`` contain a sequence of elements of a specific type. Given that their type is defined when they are declared
(see :ref:`test-case-variables`) you don't need to specify the ``type`` attribute when assigning values to them (in ``assign`` steps). Referencing a ``list``
element is done using its index which is 0-based. Adding values to a ``list`` is achieved using the ``assign`` step and can either
target an existing list element (identified by its index) or be appended to the list. The following examples illustrate how you can reference
and assign list values:

.. code-block:: xml

    <!-- 
        Assign list "myList" to another list named "anotherList". 
    -->
    <assign to="myList">$anotherList</assign>
    <!--
        Append an item to "myList" (declared as a list of strings). 
    -->
    <assign to="myList" append="true">"Value 1"</assign>
    <!-- 
        Append another item to "myList". 
    -->
    <assign to="myList" append="true">"Value 2"</assign>
    <!-- 
        Replace the first item of "myList" with "Value 3". 
    -->
    <assign to="myList{0}">"Value 3"</assign>
    <!-- 
        Assign to the number variables "index1" and "index2" values 1 and 2 respectivelly.
        Replace the item matching "index1" of "myList" with the item matching "index2" of "anotherList".
    -->
    <assign to="index1">1</assign>
    <assign to="index2">2</assign>
    <assign to="myList{$index1}">$anotherList{$index2}</assign>

.. _test-case-referring-to-variables_missing:

Missing variables
+++++++++++++++++

During a test session is it possible that you refer to a variable that is not defined. Doing so is not always an error
given that variables may be contributed to the session via external configuration or data received from messaging, processing
or validation steps. When an expression refers to a missing variable, the value that is assigned is an **empty string**.

A typical scenario where you may want to use this is when you check configuration or data for certain input and, if missing,
set a default value through the test case. This is illustrated in the following test case snippet:

.. code-block:: xml

    <if desc="Determine validation type">
        <!-- If "$SYSTEM{validationType}" cannot be determined it is considered as being an empty string. -->
        <cond>string-length($SYSTEM{validationType}) = 0</cond>
        <then>
            <!-- No value found - set a default. -->
            <assign to="typeToUse">'defaultValidationType'</assign>
        </then>
        <else>
            <!-- Value found - use it. -->
            <assign to="typeToUse">$SYSTEM{validationType}</assign>
        </else>
    </if>
    <log>'Using validation type: ' || $typeToUse</log>

.. note::
    To check more explicitly whether a given variable is defined you can also use the :ref:`VariableUtils <handlers-VariableUtils_exists>` ``exists`` operation.

.. _test-case-variables-from-expression-output:

Defining variables from expressions
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

When using :ref:`assign<tdl-step-assign>` steps the goal is to use an expression to produce an output value. This value
will be stored as a variable in the test session context and used in subsequent steps. The variables to hold such values 
can either be predetermined in the test case's :ref:`variables<test-case-variables>` section, or can be defined dynamically
during the test session execution. In the latter case, the type of the variable is determined from the output of the
expression.

As examples consider the following cases, where determining the output of expressions result in corresponding variables:

.. code-block:: xml

    <!-- 
        This results in a variable of type number named "result1".
    -->
    <assign to="result1"><![CDATA[string-length($input)]]></assign>
    <!-- 
        This results in a variable of type string named "result2".
    -->
    <assign to="result2">'This is a text value'</assign>

Automatic creation of variables also applies for container types (``map`` and ``list``) based on the target variable
expressions and additional TDL constructs that are used:

.. code-block:: xml

    <!-- 
        In this case the expression's result is a number, however the expression provided in the 
        assign step's "to" indicates that this is the value "aKey" of a map named "result3". Both the
        key and the map will be automatically created if not already defined.
    -->
    <assign to="result3{aKey}"><![CDATA[string-length($input)]]></assign>
    <!-- 
        In this case the expression results in a string that will be stored in a list named "result4".
        The fact that this is a list is determined from the "append" attribute that is used to signal that
        the value is to be added to a list. If the "result4" list is missing it will be created as a result
        of this step.
    -->
    <assign to="result4" append="true">'This is a text value'</assign>
