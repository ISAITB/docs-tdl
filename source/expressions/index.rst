.. index:: Expressions
.. _test-case-expressions:

Expressions
-----------

Expressions are used in GITB TDL to perform arbitrary operations on context variables and to provide more control over the input and
output of specific steps. The expression language assumed by GITB TDL is **XPath 3.0** given that processing XML constructs is one of the more
frequent needs when conformance testing for content specifications. However, the use of XPath does not restrict us to using XML documents as variables; 
XPath provides sufficient expressiveness to define most operation you would need to support (albeit not always in the most intuitive way).

The following ``assign`` operations illustrate some interesting examples:

.. code-block:: xml

    <!-- 
        Assign text to a string variable.
    -->
    <assign to="myString">"aText"</assign>
    <!-- 
        Assign a number to a number variable.
    -->
    <assign to="myNumber">1</assign>
    <!--
        Assign a value to a boolean variable. Considering the assign step's content is an XPath expression, this would
        be achieved using the true() or false() functions.
    -->
    <assign to="myBoolean">true()</assign>
    <!--
        From an object variable fileContent (i.e. an XML document), extract part matching 
        /testcase/steps into another object variable named targetElement.
    -->
    <assign to="targetElement" source="$fileContent">/*[local-name() = 'testcase']/*[local-name() = 'steps']</assign>
    <!-- 
        From an object variable fooContent (i.e. an XML document), extract part matching 
        /ns1:foo/ns2:bar into another object variable named barElement. Using "ns1" and "ns2" in the 
        expression assumes they are defined in the test case's "namespaces" section.
    -->
    <assign to="barElement" source="$fooContent">/ns1:foo/ns2:bar</assign>
    <!-- 
        Assign to a number variable named result the result of adding 1 to another number 
        variable named counter.
    -->
    <assign to="result">$counter + 1</assign>
    <!-- 
        Create a custom XML fragment from a string variable named value and assign it to 
        the tempXml string variable
    -->
    <assign to="tempXml"><![CDATA['<temp>' || $value || '</temp>']]></assign>
    <!--
        Assign to the boolean result variable the result of checking that a string variable 
        named input has at least 10 characters (expression wrapped in CDATA block to use '>' 
        character without escaping it
    -->
    <assign to="result"><![CDATA[string-length($input) >= 10]]></assign>
    <!-- 
        Assign to string variable result the value "result1" if the number var is 1, or 
        "result2" otherwise
    -->
    <assign to="result">if ($var = 1) then "result1" else "result2"</assign>
    <!--
        Create a map with two entries set to strings.
    -->
    <assign to="myMap{key1}">"Value 1"</assign>
    <assign to="myMap{key2}">"Value 2"</assign>
    <!--
        Create a list with two strings (note the use of 'append').
    -->
    <assign to="myList" append="true">"Value 1"</assign>
    <assign to="myList" append="true">"Value 2"</assign>
    <!-- 
        Create a map containing another map and a list.
    -->
    <assign to="myMap{myInnerMap}{value1}">"Value 1"</assign>
    <assign to="myMap{myInnerMap}{value2}">"Value 2"</assign>
    <assign to="myMap{myInnerList}" append="true">"Value 1"</assign>
    <assign to="myMap{myInnerList}" append="true">"Value 2"</assign>


As you can see the expressions you can use are limited only to the functions available in XPath 3.0. Using these there is typically always 
a way to express what is needed, potentially by first wrapping one or more values in a custom XML wrapper and then using XPath functions
on that:

.. code-block:: xml

    <!-- 
        Store custom text content as an XML object in the string tempXml.
    -->
    <assign to="tempXml" type="object"><![CDATA['<toc>' || $toc{tocEntries} || '</toc>']]></assign>
    <!-- 
        Use object tempXml to evaluate an XPath expression and store the result in a boolean result.
    -->
    <assign to="result" source="$tempXml">contains(/toc/text(), 'file.xml')</assign>

In the above example, we are using the value contained in a ``map`` variable named "toc" to construct a temporary XML content.
We can then use the "tempXml" variable for any XPath manipulation that requires a source document.

When manipulating XML content through expressions we most likely want to use **namespaces** to ensure we correctly identify our target elements.
To directly use namespaces in XPath expressions via their declared prefixes we need to first define them in the test case's :ref:`namespaces section<test-case-namespaces>`.

Escaping XML in expressions
~~~~~~~~~~~~~~~~~~~~~~~~~~~

You need to always keep in mind that when you are writing an expression you are doing so within an XML document (the test case's). This means that
special characters such as ``<`` and ``>`` need to be escaped. There are two ways to handle this, matching how you would do this 
in any XML document.

**Approach 1: Escape entities**

Special XML characters can always be escaped using their corresponding entities. In the following example we use the ``&gt;``
entity to escape the ``>`` character:

.. code-block:: xml

    <assign to="result">string-length($input) &gt;= 10</assign>

A similar approach can be taken to replace any other special character such as ``"`` (replaced by ``&quot;``) or ``'``
(replaced by ``&apos;``).

**Approach 2: CDATA block**

Using XML escape entities can result in expressions that are hard to read. In addition, they are insufficient when the text we
are using in the expression is unknown and may give unexpected results. In these cases you can use a ``CDATA`` block:

.. code-block:: xml

    <assign to="result"><![CDATA[string-length($input) >= 10]]></assign>

.. _test-case-referring-to-variables:

Referring to variables
~~~~~~~~~~~~~~~~~~~~~~

.. include:: /expressions/sections/test-case-referring-to-variables.rst

.. _test-case-configuration:

Referring to configuration parameters
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. include:: /expressions/sections/test-case-configuration.rst

.. index:: DOMAIN
.. _test-case-expressions-domain:

Domain configuration parameters
+++++++++++++++++++++++++++++++

.. include:: /expressions/sections/test-case-expressions-domain.rst

.. index:: ORGANISATION
.. index:: shortName (ORGANISATION)
.. index:: fullName (ORGANISATION)

.. _test-case-expressions-organisation:

Organisation configuration parameters
+++++++++++++++++++++++++++++++++++++

.. include:: /expressions/sections/test-case-expressions-organisation.rst

.. index:: SYSTEM
.. index:: shortName (SYSTEM)
.. index:: fullName (SYSTEM)
.. index:: version (SYSTEM)
.. index:: apiKey (SYSTEM)

.. _test-case-expressions-system:

System configuration parameters
+++++++++++++++++++++++++++++++

.. include:: /expressions/sections/test-case-expressions-system.rst

.. index:: actor, endpoint
.. _test-case-expressions-actor:

Actor configuration parameters
++++++++++++++++++++++++++++++

.. include:: /expressions/sections/test-case-expressions-actor.rst
