.. index:: Change log
.. _changelog:

Release history
===============

The current section provides an overview of the changes in each GITB TDL release.

Release numbers follow **global numbering** covering all Test Bed components, meaning that certain releases may have not
actually introduced changes to the GITB TDL. The current page lists only the releases that introduced changes, whereas
those not included are global maintenance releases that made no changes to the GITB TDL.

The latest GITB TDL release is **1.27.0**.

Release 1.27.0 - 24/06/2025
---------------------------

.. csv-table::
    :header: "Description of changes", "Relevant sections"
    :delim: |

    New inputs for the ``HttpMessagingV2`` handler to control the display of information in step reports. | :ref:`handlers-httpmessagingv2` :ref:`handlers-httpmessagingv2-reportdisplay`
    New ``showLocationPaths`` input for the ``XmlValidator`` and ``SchematronValidator`` handlers to show reported item's location paths. | :ref:`handlers-XmlValidator` :ref:`handlers-SchematronValidator`

Release 1.26.0 - 12/06/2025
---------------------------

The documentation for GITB TDL release 1.26.0 is available `here <https://www.itb.ec.europa.eu/docs/tdl/1.26.0/>`__.

.. csv-table::
    :header: "Description of changes", "Relevant sections"
    :delim: |

    New ``JsonValidator`` handler for ``verify`` steps to validate JSON against JSON Schema. | :ref:`handlers-JsonValidator`
    New ``ShaclValidator`` handler for ``verify`` steps to validate RDF models against SHACL shapes. | :ref:`handlers-ShaclValidator`
    New ``YamlValidator`` handler for ``verify`` steps to validate YAML against JSON Schema. | :ref:`handlers-YamlValidator`
    Support the delegation of an ``interact`` step to an external handler service instead of the default UI-based approach, that will be conditionally triggered to complete the interaction. | :ref:`tdl-step-interact_handler`
    Extend all test steps with a ``skipped`` attribute to allow them being conditionally skipped during test execution. | :ref:`tdl-steps-common-skipped`
    Introduce a ``RdfUtils`` handler for ``process`` steps to manipulate (``merge``, ``convert``) and query (``ask``, ``construct``, ``select``) RDF models. | :ref:`handlers-RdfUtils`
    Add a ``find`` operation to ``CollectionUtils`` to support searching for and returning a value from a ``list`` or ``map`` (with case-sensitive matching or not). | :ref:`handlers-CollectionUtils_find`
    Extend the ``contains`` operation of ``CollectionUtils`` to support lookups that ignore casing. | :ref:`handlers-CollectionUtils_contains`
    Extend the ``append`` operation of ``CollectionUtils`` to support appending only missing items (case-sensitive or not) to the target ``map`` or ``list``. | :ref:`handlers-CollectionUtils_append`
    Support the generation of multiple test case ``output`` messages based on condition matching logic defined by the ``match`` attribute (set as ``first``, ``all`` or ``cascade``). | :ref:`test-case-output`
    Consider the type of test case and scriptlet ``imports`` optional, with ``binary`` as the default. | :ref:`test-case-imports`
    Extended the ``StringValidator``, ``NumberValidator``, ``ExpressionValidator``, ``RegExpValidator`` and ``XPathValidator`` with optional ``successMessage`` and ``failureMessage`` inputs to add user-friendly messages in the resulting report. | :ref:`handlers-StringValidator`, :ref:`handlers-NumberValidator`, :ref:`handlers-ExpressionValidator`, :ref:`handlers-RegExpValidator`, :ref:`handlers-XPathValidator`
    Applied consistent naming (with backwards compatibility through aliases) for the ``NumberValidator``, ``StringValidator``, ``XPathValidator``, ``XsdValidator``, ``SchematronValidator``, ``XsltProcessor`` and ``JsonPointerProcessor`` built-in step handlers. | :ref:`handlers-predefined-handlers`
    Allow **scriptlet inputs** for flags used at test case load time to refer to **configuration properties**.  | :ref:`scriptlets_dynamic_references`, :ref:`scriptlets_dynamic_steps`
    Extend the ``interact`` step with a ``blocking`` attribute (by default "false") to automatically proceed to the next step in case only ``instruct`` elements are defined. | :ref:`tdl-step-interact_instruct_presentation`
    Extend the ``request`` elements of ``interact`` steps to allow specifying inputs as ``required`` (optional remaining the default). | :ref:`tdl-step-interact_form_inputs`
    Consider as optional the ``from`` and ``to`` of ``receive`` steps defaulting, respectively, to the SUT and non-SUT actors. | :ref:`tdl-step-receive_actors`
    Consider as optional the ``from`` and ``to`` of ``send`` steps defaulting, respectively, to the non-SUT and SUT actors. | :ref:`tdl-step-send_actors`
    Extend the ``assign`` step with the ``byValue`` attribute to allow choosing by-value or by-reference assignment for maps and lists. | :ref:`tdl-step-assign-by-value`
    Remove the **scope isolation** requirement for remotely loaded **scriptlets**, making them identical behaviour-wise to local ones. | :ref:`scriptlets`

Release 1.25.0 - 31/01/2025
---------------------------

The documentation for GITB TDL release 1.25.0 is available `here <https://www.itb.ec.europa.eu/docs/tdl/1.25.0/>`__.

.. csv-table::
    :header: "Description of changes", "Relevant sections"
    :delim: |

    Support **test case groups** within which only a single successful result is needed to ensure conformance. | :ref:`test-suite-groups`
    Allow the ``send``, ``receive`` and ``process`` steps to be configured with their severity level in case of failures via the ``level`` attribute. | :ref:`tdl-step-receive`, :ref:`tdl-step-send`, :ref:`tdl-step-process`
    Allow the ``send``, ``receive``, ``process`` and ``verify`` steps to have their outcome inverted via the ``invert`` attribute. | :ref:`tdl-step-receive`, :ref:`tdl-step-send`, :ref:`tdl-step-process`, :ref:`tdl-step-verify`
    The ``DisplayProcessor`` can now be used without parameters to only display an overall result. | :ref:`handlers-DisplayProcessor`
    Support **test case output messages** also for undefined outcomes. | :ref:`test-case-output`
    Optionally record the original file name for files uploaded via ``interact`` steps. | :ref:`tdl-step-interact`
    Support ``exit`` steps terminating test sessions with an undefined outcome. | :ref:`tdl-step-exit`
    Steps defining children support the ``stopOnChildError`` flag to stop processing upon child error but not terminate the test session. | :ref:`tdl-steps-common-stoponchilderror`
    The ``group`` step's boundary can now be hidden by means of the ``hiddenContainer`` flag. | :ref:`tdl-step-group`
    New ``XPathProcessor`` processing handler to extract content from XML using XPath. | :ref:`handlers-XPathProcessor`

Release 1.24.0 - 15/10/2024
---------------------------

The documentation for GITB TDL release 1.24.0 is available `here <https://www.itb.ec.europa.eu/docs/tdl/1.24.0/>`__.

.. csv-table::
    :header: "Description of changes", "Relevant sections"
    :delim: |

    Support for optional scriptlet parameters and parameters set as empty by default. | :ref:`scriptlets_elements_params`
    New ``VariableUtils`` built-in processor for common operations (``exists``, ``type``) on variables. | :ref:`handlers-VariableUtils`
    New operation for the ``CollectionUtils`` built-in processor to append collections. | :ref:`handlers-CollectionUtils`
    New inputs for the ``HttpMessagingV2`` built-in messaging handler to set maximum timeouts (connection and request) when calling a remote service. | :ref:`handlers-httpmessagingv2-send`

Release 1.23.0 - 20/06/2024
---------------------------

The documentation for GITB TDL release 1.23.0 is available `here <https://www.itb.ec.europa.eu/docs/tdl/1.23.0/>`__.

.. csv-table::
    :header: "Description of changes", "Relevant sections"
    :delim: |

    New ``HttpMessagingV2`` handler for HTTP messaging, replacing and improving upon the deprecated ``HttpMessaging`` and ``HttpsMessaging``.  | :ref:`handlers-httpmessagingv2`
    New ``SoapMessagingV2`` handler for SOAP messaging, replacing and improving upon the deprecated ``SoapMessaging``.  | :ref:`handlers-soapmessagingv2`
    New ``$SYSTEM{apiKey}`` variable available in test sessions with the unique API key of the system under test. | :ref:`test-case-expressions-system`
    The ``HttpMessaging``, ``HttpsMessaging`` and ``SoapMessaging`` handlers now return a ``http_status`` entry in the ``receive`` step's report. | :ref:`handlers-httpmessaging`, :ref:`handlers-httpsmessaging`, :ref:`handlers-soapmessaging`
    Test session logging level set to ``INFO`` by default. | :ref:`test-case-steps__logging`

Release 1.22.0 - 08/04/2024
---------------------------

The documentation for GITB TDL release 1.22.0 is available `here <https://www.itb.ec.europa.eu/docs/tdl/1.22.0/>`__.

.. csv-table::
    :header: "Description of changes", "Relevant sections"
    :delim: |

    Interaction steps now support the ``admin`` attribute to specify that they should be completed by an administrator. | :ref:`tdl-step-interact`
    Interaction steps now support the ``timeout`` attribute to automatically complete them after the timeout period. | :ref:`tdl-step-interact`
    Interaction steps can now be minimised and also completed asynchronously. | :ref:`tdl-step-interact`
    The interaction step's ``mimeType`` attribute can now be set via variable reference. | :ref:`tdl-step-interact`
    Test suites and test cases support the definition of structured normative reference information. | :ref:`reference (test suite)<test-suite-metadata-specification>`, :ref:`reference (test case)<test-case-metadata-specification>`,
    The ``DisplayProcessor`` can now be set with a configurable result to mark its report as a success, failure or warning. | :ref:`handlers-DisplayProcessor`
    The documentation on service handlers for test steps was extended to list **reusable external services** available to test developers. | :ref:`handlers-reusable-handlers`

Release 1.21.0 - 06/10/2023
---------------------------

The documentation for GITB TDL release 1.21.0 is available `here <https://www.itb.ec.europa.eu/docs/tdl/1.21.0/>`__.

.. csv-table::
    :header: "Description of changes", "Relevant sections"
    :delim: |

    Interaction steps now support the ``forceDisplay`` attribute to ensure an inline display of their content. | :ref:`tdl-step-interact`
    The ``TokenGenerator`` now allows the generation of random numbers via the new ``random`` operation. | :ref:`handlers-TokenGenerator`
    New ``DelayProcessor`` to make it simpler to pause test sessions for a given duration. | :ref:`handlers-DelayProcessor`
    New ``JSONPointerProcessor`` to extract data from JSON content using JSON Pointer expressions. | :ref:`handlers-JSONPointerProcessor`
    Test cases can now be defined as ``optional`` and ``disabled`` restricting whether they can be executed and how their results are considered. | :ref:`test-case`
    Test suite and test case definitions now support an ``update`` metadata element to define how to carry out the update of existing test suites and test cases. | :ref:`update (test suite) <test-suite-metadata-update>`, :ref:`update (test case) <test-case-metadata-update>`
    Test cases can now define ``tags`` as part of their metadata to classify them and visually distinguish them for users. | :ref:`test-case-metadata-tags`

Release 1.19.0 - 17/03/2023
---------------------------

The documentation for GITB TDL release 1.19.0 is available `here <https://www.itb.ec.europa.eu/docs/tdl/1.19.0/>`__.

.. csv-table::
    :header: "Description of changes", "Relevant sections"
    :delim: |

    Extensions to step status checks (``STEP_SUCCESS``, ``STEP_STATUS``) to uniquely refer to steps within scriptlets. | :ref:`test-case-expressions-step-status-scriptlets`
    New operations (``contains``, ``randomKey``, ``randomValue``, ``remove``) for the ``CollectionUtils`` processor. | :ref:`handlers-CollectionUtils`

Release 1.18.0 - 17/10/2022
---------------------------

The documentation for GITB TDL release 1.18.0 is available `here <https://www.itb.ec.europa.eu/docs/tdl/1.18.0/>`__.

.. csv-table::
    :header: "Description of changes", "Relevant sections"
    :delim: |

    New ``XmlValidator`` validation handler to validate XML against both XSDs and Schematrons. | :ref:`handlers-XmlValidator`
    New ``ExpressionValidator`` validation handler to check arbitrary expressions. | :ref:`handlers-ExpressionValidator`
    The ``TokenGenerator`` now accepts an optional prefix and postfix to add to generated UUIDs. | :ref:`handlers-TokenGenerator`
    The ``SimulatedMessaging`` and ``DisplayProcessor`` handlers can now also specify the content types of data in their reports. | :ref:`handlers-simulatedmessaging`, :ref:`handlers-DisplayProcessor`
    The ``reply`` attribute of messaging steps can now be set dynamically within scriptlets. | :ref:`scriptlets_dynamic_references`
    The ``call`` step can now also be set as ``hidden`` to hide all steps of the referenced scriptlet. | :ref:`tdl-step-call`
    The ``SchematronValidator`` can now show the assertions made for each finding. | :ref:`handlers-SchematronValidator`
    The ``XSDValidator`` and ``SchematronValidator`` can now present findings sorted by their severity. | :ref:`handlers-XSDValidator`, :ref:`handlers-SchematronValidator`
    The ``XSDValidator`` and ``SchematronValidator`` can now hide from their report the XSD or Schematron used for the validation. | :ref:`handlers-XSDValidator`, :ref:`handlers-SchematronValidator`
    Scriptlet parameters can now be set with default values. | :ref:`scriptlets_elements_params`
    ``flow`` steps can now have individually hidden ``thread`` blocks. | :ref:`tdl-step-flow`
    Steps' ``hidden`` attribute can now be set in scriptlets via variable reference. | :ref:`scriptlets_dynamic_steps`
    ``if`` steps can be set as ``static`` for dynamic inclusion of steps within scriptlets. | :ref:`tdl-step-if_hide_boundary`, :ref:`scriptlets_dynamic_steps`
    ``if`` steps can now be set as ``hidden`` with visible child steps. | :ref:`tdl-step-if_hide_boundary`
    New ``SESSION`` map allowing access to test session metadata (session identifier, test case identifier, TDL version). | :ref:`test-case-expressions-session-metadata`
    Actor configuration (endpoint) parameters can now be set with default values and user friendly names. | :ref:`test-suite-actors`
    Using a ``process`` step via its simplified syntax now supports mapping of the input to a parameter based on the defined types. | :ref:`tdl-step-process__simplified`
    Allow the ``assign`` step to create on-the-fly maps and lists at any nesting level. | :ref:`tdl-step-assign`
    New ``TestCaseOverviewReport`` root element of type ``TestCaseOverviewReportType`` in the ``gitb_tr.xsd`` for an XML test case overview report. | :ref:`introduction_spec_links`

Release 1.17.0 - 20/07/2022
---------------------------

The documentation for GITB TDL release 1.17.0 is available `here <https://www.itb.ec.europa.eu/docs/tdl/1.17.0/>`__.

.. csv-table::
    :header: "Description of changes", "Relevant sections"
    :delim: |

    Steps' ``desc``, ``from``, ``to``, ``title`` and ``inputTitle`` values can be set as constant variable references when used within scriptlets. | :ref:`scriptlets_dynamic_references`
    New ``TemplateProcessor`` embedded processing handler for complex generation of data based on templates. | :ref:`handlers-TemplateProcessor`
    New ``XSLTProcessor`` embedded processing handler for XSLT transformations of XML content. | :ref:`handlers-XSLTProcessor`
    New ``DisplayProcessor`` embedded processing handler for displaying arbitrary content to users in a non-obtrusive manner. | :ref:`handlers-DisplayProcessor`
    New ``SimulatedMessaging`` embedded messaging handler to add simulated message exchanges between actors. | :ref:`handlers-simulatedmessaging`
    Messaging transactions (defined via ``btxn``) are no longer mandatory for ``send`` and ``receive`` messaging steps. | :ref:`tdl-step-send`, :ref:`tdl-step-receive`
    Non-transactional, non-embedded (external) ``process``, ``send`` and ``receive`` steps can now also specify authentication properties for the handler service call. | :ref:`handlers-authentication`
    The GITB TDL's expression language is now upgraded from XPath 1.0 to XPath 3.0. | :ref:`test-case-expressions`

Release 1.16.0 - 18/03/2022
---------------------------

The documentation for GITB TDL release 1.16.0 is available `here <https://www.itb.ec.europa.eu/docs/tdl/1.16.0/>`__.

.. csv-table::
    :header: "Description of changes", "Relevant sections"
    :delim: |

    Namespaces can now be defined to simplify XPath expressions in GITB TDL steps and the XPathValidator. | :ref:`test-case-namespaces`
    Actors defined in test cases are considered by default as SIMULATED. | :ref:`test-case-actors`
    Introduction of the ``STEP_STATUS`` variable to record and query the specific status of each step. | :ref:`test-case-expressions-step-status`
    Reuse of steps' ``desc`` attribute to identify them in the test session log to make log entries more meaningful. | :ref:`tdl-steps`
    Support for test session log contributions from custom test services used as remote service handlers. | :ref:`tdl-step-log`
    Support for the parallel or sequential execution of a test suite's test cases when a test suite is executed in the background. | :ref:`test-suite-test-cases`
    Test cases now feature the ``supportsParallelExecution`` attribute that can be used to enforce a test case's execution in isolation. | :ref:`test-case`

Release 1.15.0 - 29/11/2021
---------------------------

The documentation for GITB TDL release 1.15.0 is available `here <https://www.itb.ec.europa.eu/docs/tdl/1.15.0/>`__.

.. csv-table::
    :header: "Description of changes", "Relevant sections"
    :delim: |

    Messaging steps now support the ``reply`` attribute to indicate that they should be presented as reply messages. | :ref:`tdl-step-send`, :ref:`tdl-step-receive`, :ref:`tdl-step-listen`
    Support for automatically converting non-list values to lists (as single-item lists containing the value). | :ref:`test-case-types-type-conversions`
    Support for automatically converting lists and maps to strings. | :ref:`test-case-types-type-conversions`
    The ``log`` step now supports a ``level`` attribute to define the message's severity level. | :ref:`tdl-step-log`
    Test cases now support the ``logLevel`` attribute to control which messages are included in test session logs. | :ref:`test-case-steps`
    The severity level of ``verify`` steps can be now be provided dynamically via variable reference. | :ref:`tdl-step-verify`
    The ``process`` step now enables simplified usage through the ``input``, ``operation`` and ``output`` attributes. | :ref:`tdl-step-process__simplified`
    The ``call`` step now enables simplified usage through the ``input`` and ``output`` attributes. | :ref:`tdl-step-call__simplified`

Release 1.14.0 - 17/08/2021
---------------------------

The documentation for GITB TDL release 1.14.0 is available `here <https://www.itb.ec.europa.eu/docs/tdl/1.14.0/>`__.

.. csv-table::
    :header: "Description of changes", "Relevant sections"
    :delim: |

    The ``receive`` elements of ``interact`` steps support the ``inputType`` and ``mimeType`` attributes to determine the type of input control to use. | :ref:`tdl-step-interact`
    The ``instruct`` elements of ``interact`` steps support the ``mimeType`` attribute to determine downloaded file extensions and editor syntax highlighting. | :ref:`tdl-step-interact`

Release 1.13.0 - 01/07/2021
---------------------------

The documentation for GITB TDL release 1.13.0 is available `here <https://www.itb.ec.europa.eu/docs/tdl/1.13.0/>`__.

.. csv-table::
    :header: "Description of changes", "Relevant sections"
    :delim: |

    The ``process`` step can now be displayed in the test case's presentation. | :ref:`tdl-step-process`
    Support for the ``hidden`` attribute to define steps that are not included in the test case's presentation. | :ref:`tdl-steps-common-hidesteps`
    Support for the ``collapsed`` attribute on group steps to have them displayed as initially collapsed. | :ref:`tdl-steps-common-collapsed`
    Extended ``group`` steps to allow them to be included in the test case's presentation. | :ref:`tdl-step-group`

Release 1.12.0 - 03/03/2021
---------------------------

The documentation for GITB TDL release 1.12.0 is available `here <https://www.itb.ec.europa.eu/docs/tdl/1.12.0/>`__.

.. csv-table::
    :header: "Description of changes", "Relevant sections"
    :delim: |

    Support for test suites without actor definitions nor test cases, as shared resource holders. | :ref:`test-suite-sharing-empty`
    Support for sharing resources (scriptlets, artifacts, documentation) across test suites. | :ref:`test-suite-sharing`
    Support for sharing scriptlets across test cases, scriptlet output expressions, optional parameters, variables and outputs. | :ref:`Scriptlets (standalone)<scriptlets>`, :ref:`Scriptlets (test case)<test-case-scriptlets>`, :ref:`tdl-step-call`
    New embedded processing handler ``RegExpProcessor`` to process texts using regular expressions. | :ref:`handlers-RegExpProcessor`
    New embedded processing handler ``CollectionUtils`` providing utility functions for maps and lists. | :ref:`handlers-CollectionUtils`
    The ``TokenGenerator`` processing handler now allows manipulation of dates based on existing, formatted, date values. | :ref:`handlers-TokenGenerator`
    The ``SchematronValidator`` validation handler now supports defining the Schematron file type (XSLT or SCH) as an input. | :ref:`handlers-SchematronValidator`

Release 1.11.1 - 11/12/2020
---------------------------

The documentation for GITB TDL release 1.11.1 is available `here <https://www.itb.ec.europa.eu/docs/tdl/1.11.1/>`__.

.. csv-table::
    :header: "Description of changes", "Relevant sections"
    :delim: |

    Referencing a missing variable evaluates as an empty string. | :ref:`test-case-referring-to-variables_missing`

Release 1.11.0 - 13/11/2020
---------------------------

The documentation for GITB TDL release 1.11.0 is available `here <https://www.itb.ec.europa.eu/docs/tdl/1.11.0/>`__.

.. csv-table::
    :header: "Description of changes", "Relevant sections"
    :delim: |

    Support for stopping a test session immediately upon a step's failure. | :ref:`tdl-steps-common-stoponerror`, :ref:`test-case-steps`
    Support for custom output messages as the overall result of a test session's execution. | :ref:`test-case-output`
    Allow a ``verify`` step to optionally store its report's context data. | :ref:`tdl-step-verify`
    New ``TEST_SUCCESS`` variable used to track the overall status of the test session at any point. | :ref:`test-case-expressions-step-results`
    Support for custom titles for the user input popups linked to the ``interact`` step. | :ref:`tdl-step-interact`

Release 1.10.0 - 07/09/2020
---------------------------

The documentation for GITB TDL release 1.10.0 is available `here <https://www.itb.ec.europa.eu/docs/tdl/1.10.0/>`__.

.. csv-table::
    :header: "Description of changes", "Relevant sections"
    :delim: |

    Test case imports can now be defined dynamically using variable references. | :ref:`test-case-imports`, :ref:`test-case-expressions-where`
    Support for a custom ``title`` for ``if``, ``interact``, ``while``, ``repuntil``, ``foreach`` and ``flow`` steps. | :ref:`tdl-step-if`, :ref:`tdl-step-interact`, :ref:`tdl-step-while`, :ref:`tdl-step-repuntil`, :ref:`tdl-step-foreach`, :ref:`tdl-step-flow`
    Actors' endpoint parameters can now be defined as ``hidden`` to have them visible only to administrators. | :ref:`test-suite-actors`
    Actors' endpoint parameters now support the definition of allowed values with optional labels (``allowedValues`` and ``allowedValueLabels``). | :ref:`test-suite-actors`
    Actors' endpoint parameters now support defining other parameters as prerequisites (``dependsOn`` and ``dependsOnValue``). | :ref:`test-suite-actors`
    New ``log`` step to add arbitrary information to the test session's log output. | :ref:`tdl-step-log`
    The ``documentation`` content for test suites, test cases and test steps can now be provided via imported resource. | :ref:`test-suite-metadata`, :ref:`test-case-metadata`, :ref:`tdl-steps-common-documentation`
    The ID of test suites and test cases is now considered in the GITB software for referencing and matching. | :ref:`test-suite`, :ref:`test-case`

Release 1.9.0 - 30/04/2020
--------------------------

The documentation for GITB TDL release 1.9.0 is available `here <https://www.itb.ec.europa.eu/docs/tdl/1.9.0/>`__.

.. csv-table::
    :header: "Description of changes", "Relevant sections"
    :delim: |

    Rich text documentation can now also be set for test suites via the ``documentation`` element. | :ref:`test-suite-metadata`
    Rich text documentation can now also be set for test cases via the ``documentation`` element. | :ref:`test-case-metadata`
    ``Assign`` steps can now automatically create variables to store expression output. | :ref:`tdl-step-assign`, :ref:`test-case-variables`, :ref:`test-case-variables-from-expression-output`
    New ``Base64Processor`` for the handling of Base64 and data URL encoding in test cases. | :ref:`handlers-Base64Processor`

Release 1.8.0 - 20/01/2020
--------------------------

The documentation for GITB TDL release 1.8.0 is available `here <https://www.itb.ec.europa.eu/docs/tdl/1.8.0/>`__.

.. csv-table::
    :header: "Description of changes", "Relevant sections"
    :delim: |

    Actors can now be set as ``hidden`` to prevent them from being used in new conformance statements.| :ref:`test-suite-actors`
    Support for different severity levels for the validation carried out by ``verify`` steps.| :ref:`tdl-step-verify`
    Rich text documentation can now be set for test steps via the ``documentation`` element to provide further information or instructions. | :ref:`tdl-steps`, :ref:`tdl-steps-common-documentation`

Release 1.7.2 - 11/12/2019
--------------------------

The documentation for GITB TDL release 1.7.2 is available `here <https://www.itb.ec.europa.eu/docs/tdl/1.7.2/>`__.

.. csv-table::
    :header: "Description of changes", "Relevant sections"
    :delim: |

    Any expression can now be set with ``asTemplate`` to be used as a template for placeholder replacement.| :ref:`test-case-expressions-template-files`

Release 1.7.0 - 07/10/2019
--------------------------

The documentation for GITB TDL release 1.7.0 is available `here <https://www.itb.ec.europa.eu/docs/tdl/1.7.0/>`__.

.. csv-table::
    :header: "Description of changes", "Relevant sections"
    :delim: |

    Support for organisation and system level configuration parameters.| :ref:`test-case-configuration`
    Support for ``binary`` values in all levels of configuration parameters.| :ref:`test-case-configuration`
    Support for endpoint configuration parameters to be defined as editable only by administrators (``adminOnly``) and make their inclusion in test sessions optional (``notForTests``).| :ref:`test-suite-actors`
    Endpoint configuration parameters can now have secret values.| :ref:`test-suite-actors`

Release 1.6.1 - 14/06/2019
--------------------------

The documentation for GITB TDL release 1.6.1 is available `here <https://www.itb.ec.europa.eu/docs/tdl/1.6.1/>`__.

.. csv-table::
    :header: "Description of changes", "Relevant sections"
    :delim: ~

    Additional documentation for the use of templates.~ :ref:`test-case-expressions-template-files`
    Extensions to the ``TokenGenerator`` handler to fine-tune timestamp generation.~ :ref:`handlers-TokenGenerator`

Release 1.6.0 - 29/05/2019
--------------------------

The documentation for GITB TDL release 1.6.0 is available `here <https://www.itb.ec.europa.eu/docs/tdl/1.6.0/>`__.

.. csv-table::
    :header: "Description of changes", "Relevant sections"
    :delim: ~

    Documented the embedded ``UDPMessaging`` messaging handler.~ :ref:`handlers-udpmessaging`
    Introduced a new ``STEP_SUCCESS`` ``map`` variable to record the status of executed steps.~ :ref:`test-case-expressions-step-success`
    Step attributes now allow definition of text values including spaces, and numbers that are signed or floating point.~ :ref:`tdl-steps`
    Support for single and multiple selection lists to receive input in user interaction steps.~ :ref:`tdl-step-interact`
    Processing transactions are now optional allowing for simpler use of stateless processing steps.~ :ref:`tdl-step-bptxn`, :ref:`tdl-step-process`
    New embedded ``TokenGenerator`` processing handler to generate text tokens.~ :ref:`handlers-TokenGenerator`
    New embedded ``XmlMatchValidator`` validation handler for template-based matching of XML documents.~ :ref:`handlers-XmlMatchValidator`
    Test case variable values now correctly allow the definition only of a type and name.~ :ref:`test-case-variables`

Release 1.5.0 - 06/11/2018
--------------------------

The documentation for GITB TDL release 1.5.0 is available `here <https://www.itb.ec.europa.eu/docs/tdl/1.5.0/>`__.

.. csv-table::
    :header: "Description of changes", "Relevant sections"
    :delim: ~

    The ``XPathValidator`` is now updated to support XPath 3.0 expressions (from the previous limitation to XPath 1.0)~ :ref:`handlers-XPathValidator`
    Support for any type of input that can be converted to ``string``s can now be passed to the ``XPathValidator`` and ``StringValidator``~ :ref:`handlers-XPathValidator`, :ref:`handlers-StringValidator`
    Added a new ``RegExpValidator`` to test text against a regular expression~ :ref:`handlers-RegExpValidator`
    Improved default handling of the ``with``, ``type`` and ``contentType`` properties linked to user interactions~ :ref:`tdl-step-interact`, :ref:`test-case-preliminary`
    Allowed user interactions to present binary content for download~ :ref:`tdl-step-interact`, :ref:`test-case-preliminary`
    The test suite name can now be ommitted in the value provided for test case ``import`` elements~ :ref:`test-case-imports`
    It is now possible to define an actor as the specification's default to have it automatically selected for new conformance statements~ :ref:`test-suite-actors`
    The display order of actors in test execution diagrams can now be set at test suite and test case level~ :ref:`test-suite-actors` (test suite), :ref:`test-case-actors` (test case)
    The ``exit`` step is now correctly reflected on the user interface and can be set to be either a success or a failure~ :ref:`tdl-step-exit`
    The ``if`` step now considers the ``else`` block as optional~ :ref:`tdl-step-if`

Release 1.4.1 - 28/09/2018
--------------------------

The documentation for GITB TDL release 1.4.1 is available `here <https://www.itb.ec.europa.eu/docs/tdl/1.4.1/>`__.

.. csv-table::
    :header: "Description of changes", "Relevant sections"
    :delim: ~

    The ``HttpMessaging`` handler now supports HTTPS via configuration as well as handling of multipart forms~ :ref:`handlers-httpmessaging`

Release 1.3.0 - 25/05/2018
--------------------------

The documentation for GITB TDL release 1.3.0 is available `here <https://www.itb.ec.europa.eu/docs/tdl/1.3.0/>`__.

.. csv-table::
    :header: "Description of changes", "Relevant sections"
    :delim: ~

    Transactions (messaging and processing) now support ``config`` elements to customise transaction creation~ :ref:`tdl-step-btxn`, :ref:`tdl-step-bptxn`
    Configuration elements (``config``) now support variable references~ :ref:`tdl-step-btxn`, :ref:`tdl-step-send`, :ref:`tdl-step-receive`, :ref:`tdl-step-listen`, :ref:`tdl-step-bptxn`, :ref:`tdl-step-verify`
    User instructions now consider "string" as the default type and can be fully empty in case of simple messages~ :ref:`tdl-step-interact`
    User requests now consider "STRING" as the default content type~ :ref:`tdl-step-interact`
    The ``foreach`` step now allows its boundaries (start and end) to be set using variable references~ :ref:`tdl-step-foreach`
    Actors can now be defined in test suites without endpoints~ :ref:`test-suite-actors`
    Service handler implementations can now be specified with variable references~ :ref:`handlers-implementation`
    The ``receive`` step now allows a timeout to be specified~ :ref:`tdl-step-receive`
    Communication with service handlers now supports authentication (HTTP Basic and UsernameToken)~ :ref:`handlers-implementation`
    The ``SoapMessaging`` embedded messaging handler now supports HTTPS~ :ref:`handlers-soapmessaging`
    Value 'CONFORMANCE' is now correctly considered as the default test case type~ :ref:`test-case-metadata`
    The actor name defined in a test case does not have to match the actor ID~ :ref:`test-case-actors`
    Domain-level configuration parameters can now be used in expressions~ :ref:`test-case-expressions-domain`

Release 1.2.0 - 18/03/2018
--------------------------

Initial release of the documentation for the GITB TDL (release 1.2.0), available `here <https://www.itb.ec.europa.eu/docs/tdl/1.2.0/>`__.