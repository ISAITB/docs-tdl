The ``scriptlets`` element is meant to define reusable blocks of steps that can be called during the test case's execution.
They resemble function blocks in programming languages considering that they can be called multiple times with different
inputs and produce different outputs.

Scriptlets are typically defined as separate XML documents that can be used across test cases or even across test suites.
Defining a scriptlet within a test case results in it being private to its containing test case. To define such private
scriptlets, include the ``scriptlets`` element with one or more ``scriptlet`` children.

Details on how each ``scriptlet`` element is defined are provided in the :ref:`scriptlet documentation<scriptlets>`. This
includes the :ref:`differences to consider<scriptlets_embedded>` when comparing scriptlets embedded in test cases and ones
that are defined as standalone XML documents.

Calling a scriptlet from a test case is achieved through the :ref:`call<tdl-step-call>` step. The following example
illustrates the definition of a scriptlet within a test case to validate XML documents. This is called twice for each
of the inputs provided by the user.

.. code-block:: xml

    <testcase>
        <steps>
            <!-- 
                Request two files to be uploaded.
            -->
            <interact id="userData" desc="Upload files">
                <request desc="Upload the first file" name="file1" contentType="BASE64"/>
                <request desc="Upload the second file" name="file2" contentType="BASE64"/>
            </interact>
            <!-- 
                Call the scriptlet for the first file and store the result under variable "call1".
            -->
            <call id="call1" path="validateDocument">
                <input name="contentToValidate">$userData{file1}</input>
            </call>
            <!-- 
                Call the scriptlet for the second file and store the result under variable "call2".
            -->
            <call id="call2" path="validateDocument">
                <input name="contentToValidate">$userData{file2}</input>
            </call>
            <!--
                Log the root element names of the validated files.
            -->
            <log>"File 1: " || $call1{rootName}</log>
            <log>"File 2: " || $call2{rootName}</log>
        </steps>
        <scriptlets>
            <scriptlet id="validateDocument">
                <imports>
                    <artifact name="schemaToUse">resources/aSchemaFile.xsd</artifact>
                    <artifact name="schematronToUse">resources/aSchematronFile.sch</artifact>
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
                </steps>
                <output name="rootName" source="$contentToValidate">name(/*)</output>
            </scriptlet>
        </scriptlets>
    </testcase>

.. note::
    **Using scriptlets across test cases:** Scriptlets defined within test cases are private to that test case. If you want
    to use a scriptlet across several test cases, within the test suite or across test suites, you need to define it in
    its own XML document. See the :ref:`scriptlet documentation<scriptlets>` for details on this.
