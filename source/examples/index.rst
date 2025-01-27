.. _examples:

Example test suites
===================

The following section provides downloadable complete examples of GITB TDL :ref:`test suites <test-suite>`, illustrating common steps and
interesting practices. These are designed to be **self-contained** and work with any Test Bed installation, meaning that
you can deploy and try them out of the box.

To remove all external dependencies from the test suites, they lack certain concepts that are typically found in conformance tests,
such as :ref:`configuration parameters <test-case-configuration>` and use of :ref:`custom supporting services <handlers-custom-handlers>`.
Doing so however results in test suites that can be used as-is and that should be straightforward to understand.

Each test suite is covered in a separate section below, providing per case:

* The **download link** for the test suite (ready to :ref:`deploy to a Test Bed instance <test-suite-deploying>`).
* A **summary** of the test suite's purpose and main features.
* An overview of the test suite's included **resources**.
* A listing of all the **TDL features** used in the test suite.

.. index:: assign (examples)
.. index:: call (examples)
.. index:: HttpMessagingV2 (examples)
.. index:: import (examples)
.. index:: interact (examples)
.. index:: log (examples)
.. index:: output (examples)
.. index:: scriptlet (examples)
.. index:: send (examples)
.. index:: verify (examples)
.. index:: XmlValidator (examples)
.. _examples_example1:

Example 1: Validation of XML content via various inputs
-------------------------------------------------------

**Download test suite:** :download:`example1.zip`

This example showcases validation of XML content using the Test Bed's built-in **XML validation** capabilities. The test suite includes
two test cases covering the different ways in which the XML to validate is provided, specifically through **manual upload** and **HTTP request**.
The validation logic is defined once and **reused** across both test cases.

.. code-block:: none

  <archive root>
  ├── resources
  │   ├── LargePurchaseOrder.sch
  │   ├── PurchaseOrder.xsd
  │   ├── sample.xml
  │   └── sampleInvalid.xml
  ├── scriptlets
  │   └── validate.xml
  ├── testCases
  │   ├── testCase1.xml
  │   └── testCase2.xml
  └── testSuite.xml

.. csv-table::
    :header: "Concept used", "Type", "Purpose", "Resource(s)"
    :delim: |

    :ref:`assign <tdl-step-assign>` | :ref:`Test step <tdl-steps>` | Prepare step inputs. | ``validate.xml``
    :ref:`call <tdl-step-call>` | :ref:`Test step <tdl-steps>` | Call the scriptlet to carry out the common validation steps. | ``testCase1.xml`` ``testCase2.xml``
    :ref:`HttpMessagingV2 <handlers-httpmessagingv2>` | :ref:`Test step handler <handlers>` | Carry out the content retrieval using HTTP. | ``testCase2.xml``
    :ref:`import <test-case-imports>` | :ref:`Test case section <test-case>` | Import the validation artefacts to use and samples to use when providing manual input. | ``testCase1.xml`` ``validate.xml``
    :ref:`interact <tdl-step-interact>` | :ref:`Test step <tdl-steps>` | Display popup for manual input. | ``testCase1.xml``
    :ref:`log <tdl-step-log>` | :ref:`Test step <tdl-steps>` | Log progress messages in the test session log. | ``validate.xml``
    :ref:`output <test-case-output>` | :ref:`Test case section <test-case>` | Display user-friendly messages upon test completion. | ``testCase1.xml`` ``testCase2.xml``
    :ref:`scriptlet <scriptlets>` | :ref:`Scriptlet <scriptlets>` | Share common validation logic across test cases. | ``validate.xml``
    :ref:`send <tdl-step-send>` | :ref:`Test step <tdl-steps>` | Perform a HTTP GET to retrieve content. | ``testCase2.xml``
    :ref:`verify <tdl-step-verify>` | :ref:`Test step <tdl-steps>` | Validate the retrieved content. | ``validate.xml``
    :ref:`XmlValidator <handlers-XmlValidator>` | :ref:`Test step handler <handlers>` | Validate the retrieved content using XML Schema and Schematron. | ``validate.xml``

.. index:: assign (examples)
.. index:: call (examples)
.. index:: group (examples)
.. index:: hidden (examples)
.. index:: HttpMessagingV2 (examples)
.. index:: import (examples)
.. index:: interact (examples)
.. index:: log (examples)
.. index:: output (examples)
.. index:: process (examples)
.. index:: scriptlet (examples)
.. index:: send (examples)
.. index:: stopOnError (examples)
.. index:: TemplateProcessor (examples)
.. index:: TokenGenerator (examples)
.. index:: verify (examples)
.. _examples_example2:

Example 2: Validation of JSON content with fixed and dynamic schemas
--------------------------------------------------------------------

**Download test suite:** :download:`example2.zip`

This example showcases validation of **JSON content** retrieved over HTTP, using a **remote validator** (the Test Bed's generic JSON validator).
Besides checking the content's structure, the included test case also uses a **dynamically generated schema** to make session-specific tests,
ensuring that referred dates and item counts match the test case's expectations.

.. code-block:: none

  <archive root>
  ├── resources
  │   ├── schema.json
  │   └── schemaTemplate.txt
  ├── scriptlets
  │   └── validate.xml
  ├── testCases
  │   └── testCase1.xml
  └── testSuite.xml

.. csv-table::
    :header: "Concept used", "Type", "Purpose", "Resource(s)"
    :delim: |

    :ref:`assign <tdl-step-assign>` | :ref:`Test step <tdl-steps>` | Define variable and prepare validator inputs. | ``testCase1.xml`` ``validate.xml``
    :ref:`call <tdl-step-call>` | :ref:`Test step <tdl-steps>` | Call the scriptlet to carry out a schema's validation. | ``testCase1.xml``
    :ref:`Default scriptlet parameters <scriptlets_elements_params>` | :ref:`Scriptlet <scriptlets>` | Use a default value for the displayed validation step description. | ``validate.xml``
    :ref:`Dynamic scriptlet presentation <scriptlets_dynamic_references>` | :ref:`Scriptlet <scriptlets>` | Use different step descriptions within the scriptlet based on test case inputs. | ``validate.xml``
    :ref:`group <tdl-step-group>` | :ref:`Test step <tdl-steps>` | Group together visually the validation steps. | ``testCase1.xml``
    :ref:`hidden <tdl-steps-common-hidesteps>` | :ref:`Common concepts <tdl-steps-common>` | Hide the instruction popup from the displayed execution diagram. | ``testCase1.xml``
    :ref:`HttpMessagingV2 <handlers-httpmessagingv2>` | :ref:`Test step handler <handlers>` | Carry out the content retrieval using HTTP. | ``testCase1.xml``
    :ref:`import <test-case-imports>` | :ref:`Test case section <test-case>` | Import the schema to use and the template to generate the dynamic schema. | ``testCase1.xml``
    :ref:`interact <tdl-step-interact>` | :ref:`Test step <tdl-steps>` | Display popup with instructions for the user. | ``testCase1.xml``
    :ref:`JSON validator <handlers-reusable-handlers_validation_json>` | :ref:`Test step handler <handlers>` | Validate the retrieved JSON content. | ``validate.xml``
    :ref:`log <tdl-step-log>` | :ref:`Test step <tdl-steps>` | Log progress messages in the test session log. | ``validate.xml``
    :ref:`output <test-case-output>` | :ref:`Test case section <test-case>` | Display user-friendly messages upon test completion. | ``testCase1.xml``
    :ref:`process <tdl-step-process>` | :ref:`Test step <tdl-steps>` | Create the expected timestamp and generate the dynamic schema to use. | ``testCase1.xml``
    :ref:`scriptlet <scriptlets>` | :ref:`Scriptlet <scriptlets>` | Encapsulate the validation logic and steps. | ``validate.xml``
    :ref:`send <tdl-step-send>` | :ref:`Test step <tdl-steps>` | Perform a HTTP GET to retrieve content. | ``testCase1.xml``
    :ref:`stopOnError <tdl-steps-common-stoponerror>` | :ref:`Common concepts <tdl-steps-common>` | Skip subsequent steps if the content retrieval fails. | ``testCase1.xml``
    :ref:`TemplateProcessor <handlers-TemplateProcessor>` | :ref:`Test step handler <handlers>` | Use a template to generate the dynamic schema to use based on session-specific data. | ``testCase1.xml``
    :ref:`TokenGenerator <handlers-TokenGenerator>` | :ref:`Test step handler <handlers>` | Generate a timestamp for the current date with the expected format. | ``testCase1.xml``
    :ref:`verify <tdl-step-verify>` | :ref:`Test step <tdl-steps>` | Validate the retrieved content. | ``validate.xml``

.. index:: assign (examples)
.. index:: call (examples)
.. index:: documentation (examples)
.. index:: groups (examples)
.. index:: HttpMessagingV2 (examples)
.. index:: import (examples)
.. index:: log (examples)
.. index:: output (examples)
.. index:: scriptlet (examples)
.. index:: send (examples)
.. index:: SimulatedMessaging (examples)
.. index:: tags (examples)
.. index:: verify (examples)
.. index:: XmlValidator (examples)
.. _examples_example3:

Example 3: Test suite presentation, test case groups and simulated exchanges
----------------------------------------------------------------------------

**Download test suite:** :download:`example3.zip`

This example focuses on features that can be used to enhance a test suite's **presentation**, enhancing testers' **user experience**. 
Presentation features in use include test case **tags**, rich **documentation** and specific outcome **messages** for different test session
results. In addition, this test suite defines test cases with **simulated exchanges** for illustration, which are moreover **grouped** to
require that at least is executed, but allowing other simulation test cases to fail or be skipped. Although not the focus of this test suite,
each test case proceeds to **validate XML content** using XML Schema and Schematron, the content either being retrieved via HTTP GET or, for the
test cases based on simulated exchanges, provided as fixed inputs.

.. code-block:: none

  <archive root>
  ├── resources
  │   ├── LargePurchaseOrder.sch
  │   ├── PurchaseOrder.xsd
  │   ├── sample.xml
  │   ├── sampleInvalidContent.xml
  │   └── sampleInvalidStructure.xml
  ├── scriptlets
  │   └── validate.xml
  ├── testCases
  │   ├── testCase1.xml
  │   ├── testCase2.xml
  │   ├── testCase3.xml
  │   └── testCase4.xml
  └── testSuite.xml

.. csv-table::
    :header: "Concept used", "Type", "Purpose", "Resource(s)"
    :delim: |

    :ref:`assign <tdl-step-assign>` | :ref:`Test step <tdl-steps>` | Prepare step inputs. | ``testCase2.xml`` ``testCase3.xml`` ``testCase4.xml`` ``validate.xml``
    :ref:`call <tdl-step-call>` | :ref:`Test step <tdl-steps>` | Call the scriptlet to carry out validation. | ``testCase1.xml`` ``testCase2.xml`` ``testCase3.xml`` ``testCase4.xml``
    :ref:`documentation <test-case-metadata-documentation>` | :ref:`Test case section <test-case>` | Show rich documentation per test case importing from test suite HTML files. | ``testCase1.xml`` ``testCase2.xml`` ``testCase3.xml`` ``testCase4.xml``
    :ref:`groups <handlers-XmlValidator>` | :ref:`Test suite section <test-suite-groups>` | Group together test cases so that only one is needed to complete successfully. | ``testSuite.xml``
    :ref:`HttpMessagingV2 <handlers-httpmessagingv2>` | :ref:`Test step handler <handlers>` | Carry out the content retrieval using HTTP. | ``testCase1.xml``
    :ref:`import <test-case-imports>` | :ref:`Test case section <test-case>` | Import validation artefacts and sample files to use with simulated exchanges. | ``testCase2.xml`` ``testCase3.xml`` ``testCase4.xml`` ``validate.xml``
    :ref:`log <tdl-step-log>` | :ref:`Test step <tdl-steps>` | Log progress messages in the test session log. | ``validate.xml``
    :ref:`output <test-case-output>` | :ref:`Test case section <test-case>` | Display user-friendly messages upon test completion for different step outcomes. | ``testCase1.xml`` ``testCase2.xml`` ``testCase3.xml`` ``testCase4.xml``
    :ref:`scriptlet <scriptlets>` | :ref:`Scriptlet <scriptlets>` | Share common validation logic across test cases. | ``validate.xml``
    :ref:`send <tdl-step-send>` | :ref:`Test step <tdl-steps>` | Perform a HTTP GET to retrieve content or display a simulated exchange. | ``testCase1.xml`` ``testCase2.xml`` ``testCase3.xml`` ``testCase4.xml``
    :ref:`SimulatedMessaging <handlers-simulatedmessaging>` | :ref:`Test step handler <handlers>` | Display a simulated exchange between specification actors. | ``testCase2.xml`` ``testCase3.xml`` ``testCase4.xml``
    :ref:`tags <test-case-metadata-tags>` | :ref:`Test case section <test-case>` | Visually highlight the test cases using simulation. | ``testCase2.xml`` ``testCase3.xml`` ``testCase4.xml``
    :ref:`verify <tdl-step-verify>` | :ref:`Test step <tdl-steps>` | Validate the retrieved content. | ``validate.xml``
    :ref:`XmlValidator <handlers-XmlValidator>` | :ref:`Test step handler <handlers>` | Validate content using XML Schema and Schematron. | ``validate.xml``

.. index:: DisplayProcessor (examples)
.. index:: group (examples)
.. index:: output (examples)
.. index:: process (examples)
.. index:: send (examples)
.. index:: SimulatedMessaging (examples)
.. index:: stopOnChildError (examples)
.. index:: level (examples)
.. _examples_example4:

Example 4: Setup and teardown phases with fine-tuned error management
---------------------------------------------------------------------

**Download test suite:** :download:`example4.zip`

This test suite illustrates the use of **step groups** and **error management flags** to split a test case into sections with different
execution behaviours. The included test case splits its steps into setup, main and teardown phases, ensuring that setup and teardown
steps always execute and never fail test sessions, whereas main steps fail eagerly. Besides managing step failures, **group display features**
are used such as collapsing the display of setup and teardown steps, and hiding the group boundary for the main steps' group.

.. code-block:: none

  <archive root>
  ├── testCases
  │   └── testCase1.xml
  └── testSuite.xml

.. csv-table::
    :header: "Concept used", "Type", "Purpose", "Resource(s)"
    :delim: |

    :ref:`DisplayProcessor <handlers-simulatedmessaging>` | :ref:`Test step handler <handlers>` | Display an arbitrary step report. | ``testCase1.xml``
    :ref:`group <tdl-step-group>` | :ref:`Test step <tdl-steps>` | Group together visually and behaviourally test steps. | ``testCase1.xml``
    :ref:`output <test-case-output>` | :ref:`Test case section <test-case>` | Display user-friendly messages upon test completion. | ``testCase1.xml``
    :ref:`process <tdl-step-process>` | :ref:`Test step <tdl-steps>` | Make a processing step to display step reports simulating validations. | ``testCase1.xml``
    :ref:`send <tdl-step-send>` | :ref:`Test step <tdl-steps>` | Perform a messaging call to display a simulated exchange. | ``testCase1.xml``
    :ref:`SimulatedMessaging <handlers-simulatedmessaging>` | :ref:`Test step handler <handlers>` | Display a simulated exchange between specification actors. | ``testCase1.xml``
    :ref:`stopOnChildError <tdl-steps-common-stoponchilderror>` | :ref:`Common concepts <tdl-steps-common>` | Manage how step failures are managed within a specific group. | ``testCase1.xml``
    :ref:`warning level <tdl-steps-common-level>` | :ref:`Common concepts <tdl-steps-common>` | Treat teardown step failures as warnings to ensure they never cause session failures. | ``testCase1.xml``

.. index:: assign (examples)
.. index:: HttpMessagingV2 (examples)
.. index:: import (examples)
.. index:: log (examples)
.. index:: output (examples)
.. index:: send (examples)
.. index:: specification (examples)
.. index:: tags (examples)
.. index:: verify (examples)
.. index:: XmlValidator (examples)
.. _examples_example5:

Example 5: Linking test cases with specification references
-----------------------------------------------------------

**Download test suite:** :download:`example5.zip`

This test suite illustrates how test cases can be displayed with **specification references**, when it is important to have direct
traceability between test cases and normative requirements. The included tests cover **XML and JSON validation** using **built-in** and
**remote** validators, for XML and JSON content received via **HTTP GET** requests. The different syntaxes handled by each test case are
further highlighted using **tags**.

.. code-block:: none

  <archive root>
  ├── resources
  │   ├── PurchaseOrder.json
  │   └── PurchaseOrder.xsd
  ├── testCases
  │   ├── testCase1.xml
  │   ├── testCase2.xml
  └── testSuite.xml

.. csv-table::
    :header: "Concept used", "Type", "Purpose", "Resource(s)"
    :delim: |

    :ref:`assign <tdl-step-assign>` | :ref:`Test step <tdl-steps>` | Prepare validator inputs. | ``testCase2.xml``
    :ref:`HttpMessagingV2 <handlers-httpmessagingv2>` | :ref:`Test step handler <handlers>` | Carry out content retrieval using HTTP. | ``testCase1.xml`` ``testCase2.xml``
    :ref:`import <test-case-imports>` | :ref:`Test case section <test-case>` | Import validation artefacts. | ``testCase1.xml`` ``testCase2.xml``
    :ref:`JSON validator <handlers-reusable-handlers_validation_json>` | :ref:`Test step handler <handlers>` | Validate JSON content using JSON Schema. | ``testCase2.xml``
    :ref:`log <tdl-step-log>` | :ref:`Test step <tdl-steps>` | Log progress messages in the test session log. | ``testCase1.xml`` ``testCase2.xml``
    :ref:`output <test-case-output>` | :ref:`Test case section <test-case>` | Display user-friendly messages upon test completion for different step outcomes. | ``testCase1.xml`` ``testCase2.xml``
    :ref:`send <tdl-step-send>` | :ref:`Test step <tdl-steps>` | Perform HTTP GETs to retrieve the content to validate. | ``testCase1.xml`` ``testCase2.xml``
    :ref:`specification <test-case-metadata-specification>` | :ref:`Test case section <test-case>` | Display specification links for each test case. | ``testCase1.xml`` ``testCase2.xml``
    :ref:`tags <test-case-metadata-tags>` | :ref:`Test case section <test-case>` | Visually highlight the syntax covered by each test case. | ``testCase1.xml`` ``testCase2.xml``
    :ref:`verify <tdl-step-verify>` | :ref:`Test step <tdl-steps>` | Validate the retrieved content. | ``testCase1.xml`` ``testCase2.xml``
    :ref:`XmlValidator <handlers-XmlValidator>` | :ref:`Test step handler <handlers>` | Validate XML content using XML Schema. | ``testCase1.xml``

.. index:: assign (examples)
.. index:: call (examples)
.. index:: hidden (examples)
.. index:: import (examples)
.. index:: interact (examples)
.. index:: log (examples)
.. index:: namespace (examples)
.. index:: output (examples)
.. index:: process (examples)
.. index:: scriptlet (examples)
.. index:: send (examples)
.. index:: SimulatedMessaging (examples)
.. index:: stopOnError (examples)
.. index:: TemplateProcessor (examples)
.. index:: TokenGenerator (examples)
.. index:: verify (examples)
.. index:: while (examples)
.. _examples_example6:

Example 6: Request and asynchronous polling for status update
-------------------------------------------------------------

**Download test suite:** :download:`example6.zip`

This is a more complex scenario that considers sending a request and then using a **polling approach** to check for its
processing status. It involves reusing steps via **scriptlets**, generating messages using **templates**, **iterating** to 
make poll attempts following **delays**, making **conditional processing**, and extracting values from XML content using 
**XPath** expressions.

.. code-block:: none

  <archive root>
  ├── resources
  │   ├── ackRequestTemplate.xml
  │   ├── ackResponseTemplate.xml
  │   ├── messageRequest.xml
  │   └── messageResponse.xml
  ├── scriptlets
  │   └── createSimulatedStatusResponse.xml
  ├── testCases
  │   └── testCase1.xml
  └── testSuite.xml

.. csv-table::
    :header: "Concept used", "Type", "Purpose", "Resource(s)"
    :delim: |

    :ref:`assign <tdl-step-assign>` | :ref:`Test step <tdl-steps>` | Prepare step inputs, assign variables and check results. | ``testCase1.xml`` ``createSimulatedStatusResponse.xml``
    :ref:`call <tdl-step-call>` | :ref:`Test step <tdl-steps>` | Call the scriptlet to carry out status request steps. | ``testCase1.xml``
    :ref:`hidden <tdl-steps-common-hidesteps>` | :ref:`Common concepts <tdl-steps-common>` | Hide the user interaction popup from the displayed execution diagram. | ``createSimulatedStatusResponse.xml``
    :ref:`import <test-case-imports>` | :ref:`Test case section <test-case>` | Import validation artefacts. | ``testCase1.xml`` ``createSimulatedStatusResponse.xml``
    :ref:`interact <tdl-step-interact>` | :ref:`Test step <tdl-steps>` | Display popup to request the simulated status to return. | ``createSimulatedStatusResponse.xml``
    :ref:`log <tdl-step-log>` | :ref:`Test step <tdl-steps>` | Log progress messages in the test session log. | ``testCase1.xml`` ``createSimulatedStatusResponse.xml``
    :ref:`namespace <test-case-imports>` | :ref:`Test case section <test-case>` | Declare namespaces for XPath lookups. | ``testCase1.xml``
    :ref:`output <test-case-output>` | :ref:`Test case section <test-case>` | Display user-friendly messages upon test completion. | ``testCase1.xml``
    :ref:`process <tdl-step-process>` | :ref:`Test step <tdl-steps>` | Use templates and generate timestamps. | ``testCase1.xml`` ``createSimulatedStatusResponse.xml``
    :ref:`scriptlet <scriptlets>` | :ref:`Scriptlet <scriptlets>` | Encapsulate the status request steps. | ``createSimulatedStatusResponse.xml``
    :ref:`send <tdl-step-send>` | :ref:`Test step <tdl-steps>` | Perform messaging request. | ``testCase1.xml``
    :ref:`SimulatedMessaging <handlers-simulatedmessaging>` | :ref:`Test step handler <handlers>` | Display a simulated exchange between specification actors. | ``testCase1.xml``
    :ref:`stopOnError <tdl-steps-common-stoponerror>` | :ref:`Common concepts <tdl-steps-common>` | Immediately stop the test session if a step fails. | ``testCase1.xml``
    :ref:`TemplateProcessor <handlers-TemplateProcessor>` | :ref:`Test step handler <handlers>` | Use a template to generate messages based on parameters. | ``testCase1.xml`` ``createSimulatedStatusResponse.xml``
    :ref:`TokenGenerator <handlers-TokenGenerator>` | :ref:`Test step handler <handlers>` | Generate a timestamp for the polling responses. | ``createSimulatedStatusResponse.xml``
    :ref:`verify <tdl-step-verify>` | :ref:`Test step <tdl-steps>` | Validate the retrieved status value. | ``testCase1.xml``
    :ref:`while <tdl-step-while>` | :ref:`Test step <tdl-steps>` | Iterate through the polling attempts until polling should complete. | ``testCase1.xml``