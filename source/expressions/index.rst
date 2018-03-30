.. _test-case-expressions:

Expressions
-----------

Expressions are used in GITB TDL to perform arbitrary operations on context variables and to provide more control over the input and
output of specific steps. The approach to interpret and execute expressions is pluggable, meaning that a test bed implementation can
support any number of expression languages that a test case can then refer to. This is achieved by means of the declarations in a 
test case's  ``namespaces`` element (see :ref:`test-case-namespaces`) where a prefix is used to identify each language used. 
The default expression language assumed by GITB TDL is XPath 1.0 given that processing XML constructs is one of the more frequent needs 
when conformance testing for content specifications. However, the use of XPath does not restrict us to using XML documents as variables; 
XPath provides sufficient expressiveness to define most operation you would need to support (albeit not always in the most intuitive way).

The following ``assign`` operations illustrate some interesting examples:

.. code-block:: xml

    <!-- 
        Assign to text to a string variable.
    -->
    <assign to="$myString">"aText"</assign>
    <!-- 
        Assign a number to a number variable.
    -->
    <assign to="$myNumber">1</assign>
    <!-- 
        From an object variable fileContent (i.e. an XML document), extract part matching 
        /testcase/steps into another object variable named targetElement.
    -->
    <assign to="$targetElement" source="$fileContent">/*[local-name() = 'testcase']/*[local-name() = 'steps']</assign>
    <!-- 
        Assign to a number variable named result the result of adding 1 to another number 
        variable named counter.
    -->
    <assign to="$result">$counter + 1</assign>
    <!-- 
        Create a custom XML fragment from a string variable named value and assign it to 
        the tempXml string variable
    -->
    <assign to="$tempXml">concat('<temp>', $value, '</temp>')</assign>
    <!--
        Assign to the boolean result variable the result of checking that a string variable 
        named input has at least 10 characters (expression wrapped in CDATA block to use '>' 
        character without escaping it
    -->
    <assign to="$result"><![CDATA[string-length($input) >= 10]]></assign>
    <!-- 
        Assign to string variable result the value "result1" if the number var is 1, or 
        "result2" otherwise
    -->
    <assign to="$result">concat(substring('result1', 1 div number(boolean($var = 1))), substring('result2', 1 div number(not(boolean($var = 1)))))</assign>

As you can see the expressions you can use are limited only to the functions available in XPath 1.0. Using these there is typically always 
a way to express what is needed, potentially by first wrapping one or more values in a custom XML wrapper and then using XPath functions
on that:

.. code-block:: xml

    <!-- 
        Store custom content as a string in the string tempStr.
    -->
    <assign to="$tempStr"><![CDATA[concat('<toc>', $toc{tocEntries}, '</toc>')]]></assign>
    <!-- 
        Convert tempStr to an object tempXml.
    -->
    <assign to="$tempXml">$tempStr</assign>
    <!-- 
        Use object tempXml to evaluate an XPath expression and store the result in a boolean result.
    -->
    <assign to="$result" source="$tempXml">contains(/toc/text(), 'file.xml')</assign>

In the above example, we are using the value contained in a ``map`` variable named "toc" to construct a temporary ``string`` with 
XML content. We then assign this to an ``object`` variable named "tempXml" to convert it into an XML document (i.e. a variable
of type ``object``). We can then use the "tempXml" variable for any XPath manipulation that requires a source document.

Escaping XML in expressions
~~~~~~~~~~~~~~~~~~~~~~~~~~~

You need to always keep in mind that when you are writing an expression you are doing so within an XML document (the test case's). This means that
special characters such as ``<`` and ``>`` need to be escaped. There are two ways to handle this, matching how you would do this 
in any XML document.

**Approach 1: Escape entities**

Special XML characters can always be escaped using their corresponding entities. In the following example we use the ``&gt;``
entity to escape the ``>`` character:

.. code-block:: xml

    <assign to="$result">string-length($input) &gt;= 10</assign>

A similar approach can be taken to replace any other special character such as ``"`` (replaced by ``&quot;``) or ``'``
(replaced by ``&apos;``).

**Approach 2: CDATA block**

Using XML escape entities can result in expressions that are hard to read. In addition, they are insufficient when the text we
are using in the expression is unknown and may give unexpected results. In these cases you can use a ``CDATA`` block:

.. code-block:: xml

    <assign to="$result"><![CDATA[string-length($input) >= 10]]></assign>

Referring to variables
~~~~~~~~~~~~~~~~~~~~~~

As you have seen in the previous examples, referring to variables is a very common use case in GITB TDL expressions. Variable
references are done as ``$VARIABLE``, i.e. using the ``$`` character, followed by the variable's name. Furthermore, when
an expression consists only of a variable reference without other XPath elements it is referred to as a **pure variable reference**.
As we have discussed in :ref:`test-case-types-type-conversions`, pure variable references are important when we need to convert 
variables from one type to another in ``assign`` steps.

Map elements
++++++++++++

Variables of type ``map`` represent special cases as they can contain additional variables and even additional ``map``s. Referring to 
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
    <assign to="$myMap">$anotherMap</assign>
    <!-- 
        Assign the entry of "myMap" named "myKey" to the string "A value". 
    -->
    <assign to="$myMap{myKey}" type="string">"A value"</assign>
    <!-- 
        Assign the entry of "myMap" named "myOtherKey" to an entry of "anotherMap" 
        named "anotherKey" as a string. 
    -->
    <assign to="$myMap{myOtherKey}" type="string">$anotherMap{anotherKey}</assign>
    <!-- 
        Assign value "key1" to the string variable "k1". 
        Then use the "k1" variable to pick the target entry of "myMap" 
        to set as the string "A value".
    -->
    <assign to="$k1">"key1"</assign>
    <assign to="$myMap{$k1}" type="string">"A value"</assign>
    <!-- 
        Assign value "key1" to the string variable "k1". 
        Assign value "key2" to the string variable "k2". 
        Then use the "k1" variable to pick the target entry of "myMap" to set as a string matching 
        the entry of "anotherMap" for the key matching the value of "k2".
    -->
    <assign to="$k1">"key1"</assign>
    <assign to="$k2">"key2"</assign>
    <assign to="$myMap{$k1}" type="string">$anotherMap{$k2}</assign>

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
    <assign to="$myList">$anotherList</assign>
    <!--
        Append an item to "myList" (declared as a list of strings). 
    -->
    <assign to="$myList" append="true">"Value 1"</assign>
    <!-- 
        Append another item to "myList". 
    -->
    <assign to="$myList" append="true">"Value 2"</assign>
    <!-- 
        Replace the first item of "myList" with "Value 3". 
    -->
    <assign to="$myList{0}">"Value 3"</assign>
    <!-- 
        Assign to the number variables "index1" and "index2" values 1 and 2 respectivelly.
        Replace the item matching "index1" of "myList" with the item matching "index2" of "anotherList".
    -->
    <assign to="$index1">1</assign>
    <assign to="$index2">2</assign>
    <assign to="$myList{$index1}">$anotherList{$index2}</assign>

Referring to actor configuration parameters
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Configuration relating to specific actors is defined by means of endpoints and parameters. These can be declared in the following ways:

* **Externally:** Actors may be defined fully in the test bed. In this case the test suite simply references actors by their ID (see :ref:`test-suite-deploying`).
* **In the test suite:** Actors can be fully defined in a test suite, listing their endpoints and parameters (see :ref:`test-suite-actors`).
* **During the test session initiation:** Simulated actors participating in messaging transactions with SUTs have their messaging handlers 
  called during test suite initiation at which time they can return endpoints and parameters (see :ref:`test-suite-actors-endpoints-simulated`).
* **In the test case:** Simulated actors participating in messaging transactions with SUTs can define endpoints and parameters as fixed values 
  in the test case (unless already defined during the initiation step - see previous point) (see :ref:`test-case-actors`).

As discussed in :ref:`test-suite-actors-endpoints-simulated`, the latter two cases result in the configured parameters being set for the SUT actor 
by matching the SUTs endpoint name against the endpoint name of the simulated actor's configuration (defined during initiation or statically in the test case).

The resulting configuration from the above sources for all actors are recorded in the test session context as variables. Each actor's configuration
results in a ``map`` being created named using the actor's ID. Configuration for the SUT actor pertinent to simulated actors is stored as additional 
``map`` variables under the SUT actor's ``map``. Each such ``map`` is named using the ID of the corresponding simulated actor. Once in place, these
configuration variables can be referenced and manipulated in exactly the same way as regular variables.

The following example illustrates the creation of session context variables for different types of actor configuration. Consider a test suite defined as
follows:

.. code-block:: xml

    <testsuite>
        <actors>
            <gitb:actor id="Sender">
                <gitb:name>Sender</gitb:name>
                <gitb:endpoint name="info">
                    <gitb:config name="dataVersion" kind="SIMPLE"/>
                </gitb:endpoint>
            </gitb:actor>
            <gitb:actor id="Receiver">
                <gitb:name>Receiver</gitb:name>
            </gitb:actor>
        </actors>
        <testcase id="config_test_1"/>
    </testsuite>

And the "config_test_1" test case is defined as follows:

.. code-block:: xml

    <testcase id="config_test_1">
        <actors>
            <gitb:actor id="Sender" name="Sender" role="SUT"/>
            <gitb:actor id="Receiver" name="Receiver" role="SIMULATED">
                <gitb:endpoint name="info">
                    <gitb:config name="address" kind="SIMPLE">AN_ADDRESS</gitb:config>
                </gitb:endpoint>
            </gitb:actor>
        </actors>
        <variables>
            <var name="temp" type="string"/>
        </variables>
        <steps>
            <!-- 
                Lookup the "dataVersion" property configured for the "Sender". 
            -->
            <assign to="$temp">$Sender{dataVersion}</assign>
            <!-- 
                Lookup the "address" property configured by the simulated "Receiver" for the "Sender". 
                This is statically defined here but could also be received from the "Receiver" messaging
                handler as part of the test session's initiation phase.
            -->
            <assign to="$temp">$Sender{Receiver}{address}</assign>
        </steps>
    </testcase>

Where can expressions be used?
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The following table provides an overview of the places where expressions can be used:

.. csv-table::
    :widths: 30, 70
    :header: "Use", "Description"

    Step :ref:`tdl-step-interact`, Used in the values displayed to (``instruct``) or requested from (``request``) users.
    Step :ref:`tdl-step-send`, Used to determine ``input`` values.
    Step :ref:`tdl-step-receive`, Used to determine ``input`` values.
    Step :ref:`tdl-step-listen`, Used to determine ``input`` values.
    Step :ref:`tdl-step-process`, Used to determine ``input`` values.
    Step :ref:`tdl-step-if`, Used to define and evaluate the if condition (``cond``).
    Step :ref:`tdl-step-while`, Used to define and evaluate the loop condition (``cond``).
    Step :ref:`tdl-step-repuntil`, Used to define and evaluate the repeat condition (``cond``).
    Step :ref:`tdl-step-assign`, Used as the expression to apply. Also a pure variable reference is used in the ``to`` and ``source`` elements.
    Step :ref:`tdl-step-verify`, Used to determine ``input`` values.
    Step :ref:`tdl-step-call`, Used to determine ``input`` values.