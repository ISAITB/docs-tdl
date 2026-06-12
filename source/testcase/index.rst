.. index:: Test cases
.. _test-case:

Test cases
==========

Overview
--------

Test cases are the means by which a specific testing scenario is implemented. One or more test cases form the content of a test suite.
The following example represents a complete, simple test case for the validation of an invoice that is uploaded by a user.

.. code-block:: xml

    <?xml version="1.0" encoding="UTF-8"?>
    <testcase id="testCase1" xmlns="http://www.gitb.com/tdl/v1/" xmlns:gitb="http://www.gitb.com/core/v1/">
        <metadata>
            <gitb:name>UBL invoice validation 1</gitb:name>
            <gitb:version>1.0</gitb:version>
            <gitb:description>Test case to verify the correctness of a UBL invoice. The invoice is provided manually through user upload.</gitb:description>
        </metadata>
        <imports>
            <artifact name="schema">artifacts/UBL/maindoc/UBL-Invoice-2.1.xsd</artifact>
            <artifact name="schematron">artifacts/BII/BII_CORE/BIICORE-UBL-T10-V1.0.xsl</artifact>
        </imports>
        <actors>
            <gitb:actor id="User" name="User" role="SUT"/>
        </actors>
        <steps>
            <!-- 
                Step 1. Request the user to upload the UBL invoice.
            -->
            <interact id="userData" desc="UBL invoice upload">
                <request name="invoice" desc="Upload the UBL invoice to validate" inputType="UPLOAD"/>
            </interact>
            <!-- 
                Step 2. Validate the uploaded invoice.
            -->
            <verify handler="XmlValidator" desc="Validate invoice">
                <input name="xml">$userData{invoice}</input>
                <input name="xsd">$schema</input>
                <input name="schematron">$schematron</input>
            </verify>            
        </steps>
        <output>
            <failure>
                <default>"The test session resulted in a failure. Please check the validation reports and apply required corrections."</default>
            </failure>
        </output>
    </testcase>

.. index:: id (Test case)
.. index:: supportsParallelExecution
.. index:: metadata (Test case)
.. index:: namespaces (Test case)
.. index:: imports (Test case)
.. index:: preliminary (Test case)
.. index:: variables (Test case)
.. index:: actors (Test case)
.. index:: steps (Test case)
.. index:: output (Test case)
.. index:: scriptlets (Test case)
.. index:: optional (Test case)
.. index:: disabled (Test case)

The following table provides an overview of the attributes and child elements that a ``testcase`` may have. A more detailed discussion per case 
follows in the subsequent sections.

.. csv-table::
    :stub-columns: 1
    :header: "Name", "Required?", "Description"

    @disabled, no, A boolean flag indicating whether this test case is disabled (default is "false"). Disabled test cases cannot be executed and any existing test results don't count towards a conformance statement's status.
    @id, yes, A string to uniquely identify the test case by. This is referenced in the test suite XML.
    @optional, no, A boolean flag indicating whether this test case is optional (default is "false"). Optional test cases may be executed but their results don't count towards a conformance statement's status.
    @supportsParallelExecution, no, A boolean flag indicating whether this test case may be executed in parallel with other test cases for a given SUT (default is "true").
    actors, yes, The set of actors that this test case refers to.
    imports, no, An optional set of imports used to load additional resources from the test suite.
    metadata, yes, A block containing the metadata used to describe the test case.
    namespaces, no, An optional set of namespace declarations to define the namespace prefixes used in the test case's expressions.
    output, no, Definition of an output message to display for the overall test session.
    preliminary, no, An optional set of user interaction steps to display before the test session starts.
    scriptlets, no, Optional named groups of test steps which can be used within the test case.
    steps, yes, The sequence of steps that this test case foresees.
    variables, no, An optional set of variables that are used in the test case.

The ``id`` attribute is important in uniquely identifying the test case within a given test suite, and needs to be referenced by the 
:ref:`test suite<test-suite>` if it is to be considered. It is not presented to normal users, only administrators, and is used when 
:ref:`uploading a new version of a test suite<test-suite-deploying>` to determine whether the test case definition serves as an update to an existing 
test case.

The ``supportsParallelExecution`` attribute is important in determining how the test case is handled in batch background executions (i.e. not executions
that are interactively launched and followed by a tester). If this is set to "true", the default value considered if missing, the test case is assumed
to be able to correctly function while other test cases are being executed in parallel for the same SUT. This means that the design of the test case
caters for such concurrent sessions and is able to correctly map exchanged messages to sessions. This is not always possible to do, especially in 
scenarios where messaging is initiated from the SUT (not by the Test Bed) or are asynchronous in nature.

If the test case cannot correctly handle such concurrency, you need to set ``supportsParallelExecution`` to "true". Doing so instructs the test
engine to always execute the given test case in isolation. Any ongoing test session will first need to complete before the current test case is executed
and its own test session will itself need to complete before executing any other test cases. The order of execution of test cases when making such 
considerations is defined by their :ref:`declaration order<test-suite-test-cases>` in the :ref:`test suite<test-suite>`.

.. note::
    When ``supportsParallelExecution`` is set to "false", the test case's non-parallel execution is honoured only within the context of a single batch
    execution of a test suite. The flag becomes ineffective if the tester explicitly launches separate test sessions in parallel.

Elements
--------

We will now see how a test case breaks down into its individual sections and discuss the purpose of each.

.. index:: metadata (Test cases)
.. index:: name (Test case metadata)
.. index:: type (Test case metadata)
.. index:: version (Test case metadata)
.. index:: authors (Test case metadata)
.. index:: description (Test case metadata)
.. index:: published (Test case metadata)
.. index:: lastModified (Test case metadata)
.. index:: documentation (Test case metadata)
.. index:: update (Test case metadata)
.. index:: tags (Test case metadata)
.. _test-case-metadata:

Metadata
~~~~~~~~

.. include:: /testcase/sections/test-case-metadata.rst

.. index:: namespaces (Test case)
.. index:: ns (Test case namespaces)
.. index:: prefix (Test case namespaces)
.. _test-case-namespaces:

Namespaces
~~~~~~~~~~

.. include:: /testcase/sections/test-case-namespaces.rst

.. index:: imports (Test case)
.. index:: artifact (Test case imports)
.. index:: name (Test case imports)
.. index:: type (Test case imports)
.. index:: encoding (Test case imports)
.. index:: from (Test case imports)
.. _test-case-imports:

Imports
~~~~~~~

.. include:: /testcase/sections/test-case-imports.rst

.. index:: preliminary (Test case)
.. _test-case-preliminary:

Preliminary
~~~~~~~~~~~

.. include:: /testcase/sections/test-case-preliminary.rst

.. index:: actors (Test case)
.. index:: id (Test case actors)
.. index:: name (Test case actors)
.. index:: role (Test case actors)
.. index:: displayOrder (Test case actors)
.. index:: endpoint (Test case actors)
.. index:: adminDisplayOrder (Test case actors)
.. index:: adminName (Test case actors)
.. index:: engineDisplayOrder (Test case actors)
.. index:: engineName (Test case actors)
.. index:: userDisplayOrder (Test case actors)
.. index:: userName (Test case actors)

.. _test-case-actors:

Actors
~~~~~~

.. include:: /testcase/sections/test-case-actors.rst

.. index:: variables (Test case)
.. index:: name (Test case variables)
.. index:: type (Test case variables)
.. index:: value (Test case variables)
.. _test-case-variables:

Variables
~~~~~~~~~

.. include:: /testcase/sections/test-case-variables.rst

.. index:: steps (Test case)
.. index:: stopOnError (Test case)
.. index:: logLevel
.. index:: ERROR (Test case logLevel)
.. index:: WARNING (Test case logLevel)
.. index:: INFO (Test case logLevel)
.. index:: DEBUG (Test case logLevel)

.. _test-case-steps:

Steps
~~~~~

.. include:: /testcase/sections/test-case-steps.rst

.. index:: output (Test case)
.. index:: success (Test case output)
.. index:: failure (Test case output)
.. index:: undefined (Test case output)
.. index:: case (Test case output)
.. index:: default (Test case output)
.. index:: cond (Test case output)
.. index:: message (Test case output)
.. index:: match (Test case output)
.. index:: all (Test case output)
.. index:: first (Test case output)
.. index:: cascade (Test case output)
.. _test-case-output:

Output
~~~~~~

.. include:: /testcase/sections/test-case-output.rst

.. index:: scriptlets (Test case)
.. _test-case-scriptlets:

Scriptlets
~~~~~~~~~~

.. include:: /testcase/sections/test-case-scriptlets.rst
