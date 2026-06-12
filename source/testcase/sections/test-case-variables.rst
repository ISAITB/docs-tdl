.. note::

    **Implicit variables:** Variables are automatically created for new :ref:`assignments <tdl-step-assign>` and you should typically never need
    to declare them explicitly. The exception is test cases `executed via API <https://www.itb.ec.europa.eu/docs/itb-ou/latest/api/index.html#start>`__
    that are designed to work with client-provided inputs. Such inputs must map to test case variables.

The ``variables`` element can be defined to create one or more variables that will be used during the test case's execution. It contains one 
or more ``var`` elements, one per variable, with the following structure:

.. csv-table::
    :stub-columns: 1
    :header: "Name", "Required?", "Description"

    @name, yes, The name of the variable. It is with this name that the variable can be referenced.
    @type, yes, The type of the variable. One of the GITB data types can be used (see :ref:`test-case-types`).
    value, no, One or more values for the variable. More than one values are applicable in case of a ``map`` or ``list`` type.

Variables can be used to record arbitrary information for the duration of the test session. These can be fixed values defined along with the
variable's definition or dynamically produced values resulting from test steps. The way to reference variables is defined based on the expression
language in place. Using the default XPath 3.0 expression language a variable named ``myVar`` is referenced as ``$myVar``. More information on 
expressions to reference variable values is provided in :ref:`test-case-expressions`.

Definition of variables using the ``variables`` element is **optional** given that test steps resulting in output will automatically create 
variables as needed to store the output in the test session context. Such steps include:

* :ref:`Assign steps<tdl-step-assign>` that define new values or calculate expressions.
* :ref:`User interaction steps<tdl-step-interact>` that request data from the user.
* Messaging steps to record the output of a :ref:`send<tdl-step-send>` or a :ref:`receive<tdl-step-receive>`.
* :ref:`Processing steps<tdl-step-process>` to record the output of the process.

The type of the automatically created variables in the above cases is inferred from the type of the relevant data or expression result. For example,
when assigning a string to a variable, this will automatically be set with a ``string`` type. Considering this, you would use the ``variables`` element
to predefine variables in the following cases:

* To predefine all variables if you prefer this from the perspective of code organisation.
* To explicitly set the type of variables in cases where the automatic determination is not suitable (e.g. force a ``string`` type for a numeric value).
* To cover exceptional cases where automatic type determination is not possible.
* To provide initial values to variables.
* To cover inputs provided for test sessions started via `REST API <https://www.itb.ec.europa.eu/docs/itb-ou/latest/api/index.html#start>`__.

For examples of automatic variable definition refer to the corresponding steps as well as the documentation on :ref:`expressions<test-case-variables-from-expression-output>`.
Coming back to explicitly defined variables, the following example shows two such cases, one to store a user-uploaded file and another to store a part of it, 
extracted via XPath:

.. code-block:: xml

    <testcase>
        <imports>
            <artifact name="schemaFile">artifacts/UBL/maindoc/UBL-Invoice-2.1.xsd</artifact>
        </imports>
        <variables>
            <var name="fileContent" type="object"/>
            <var name="targetElement" type="object"/>
        </variables>
        <steps>
            <!-- 
                Store the uploaded result in the fileContent variable.
            -->
            <interact desc="UBL invoice upload">
                <request desc="Upload the UBL invoice to validate" contentType="BASE64">$fileContent</request>
            </interact>
            <!-- 
                Extract a part of it and store in the targetElement variable.
            -->
            <assign to="$targetElement" source="$fileContent">/*[local-name() = 'testcase']/*[local-name() = 'steps']</assign>
            <!-- 
                Pass the targetElement for validation.
            -->
            <verify handler="XmlValidator" desc="Validate content">
                <input name="xml">$targetElement</input>
                <input name="xsd">$schemaFile</input>
            </verify>
        </steps>
    </testcase>

Setting a variable's initial value is achieved using the ``value`` element, with one or more being used in case of a ``map`` or ``list``
type. The following example illustrates setting values for different variable types:

.. code-block:: xml

    <testcase>
        <variables>
            <var name="aList" type="list[string]">
                <value>List value 1</value>
                <value>List value 2</value>
            </var>
            <var name="aMap" type="map">
                <value name="key1" type="string">Map value 1</value>
                <value name="key2" type="string">Map value 2</value>
            </var>
            <var name="aString" type="string">
                <value>A string value</value>
            </var>
        </variables>
    </testcase>

.. note::
    **List variables:** When a ``list`` is defined as a variable it also needs to specify its internal element type. To do this you
    need to specify the ``type`` attribute as ``list[INTERNAL_TYPE]``. For example a ``list`` of ``string`` elements is defined as
    ``<var name="myList" type="list[string]"/>``.
