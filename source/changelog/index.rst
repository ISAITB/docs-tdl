.. index:: Change log
.. _changelog:

Change history
==============

The current section provides an overview of new and notable changes in the latest GITB TDL documentation release
as well as access to all previous documentation versions.

What's new in release 1.26.0
----------------------------

The table that follows summarises the notable changes in the latest documentation release that matches the latest
GITB TDL and GITB software release. In each case the "type" indicates whether the change applies to the GITB TDL as 
a whole or specific improvements in the GITB software (or both) . In addition, references are provided to the relevant sections 
for more information.

.. csv-table::
    :header: "Description", "Type", "Relevant sections"
    :delim: |

    New ``JsonValidator`` handler for ``verify`` steps to validate JSON against JSON Schema. | GITB TDL | :ref:`handlers-JsonValidator`
    New ``ShaclValidator`` handler for ``verify`` steps to validate RDF models against SHACL shapes. | GITB TDL | :ref:`handlers-ShaclValidator`
    New ``YamlValidator`` handler for ``verify`` steps to validate YAML against JSON Schema. | GITB TDL | :ref:`handlers-YamlValidator`
    Support the delegation of an ``interact`` step to an external handler service instead of the default UI-based approach, that will be conditionally triggered to complete the interaction. | GITB TDL | :ref:`tdl-step-interact_handler`
    Extend all test steps with a ``skipped`` attribute to allow them being conditionally skipped during test execution. | GITB TDL | :ref:`tdl-steps-common-skipped`
    Introduce a ``RdfUtils`` handler for ``process`` steps to manipulate (``merge``, ``convert``) and query (``ask``, ``construct``, ``select``) RDF models. | GITB TDL | :ref:`handlers-RdfUtils`
    Add a ``find`` operation to ``CollectionUtils`` to support searching for and returning a value from a ``list`` or ``map`` (with case-sensitive matching or not). | GITB TDL | :ref:`handlers-CollectionUtils_find`
    Extend the ``contains`` operation of ``CollectionUtils`` to support lookups that ignore casing. | GITB TDL | :ref:`handlers-CollectionUtils_contains`
    Extend the ``append`` operation of ``CollectionUtils`` to support appending only missing items (case-sensitive or not) to the target ``map`` or ``list``. | GITB TDL | :ref:`handlers-CollectionUtils_append`
    Support the generation of multiple test case ``output`` messages based on condition matching logic defined by the ``match`` attribute (set as ``first``, ``all`` or ``cascade``). | GITB TDL | :ref:`test-case-output`
    Consider the type of test case and scriptlet ``imports`` optional, with ``binary`` as the default. | GITB TDL | :ref:`test-case-imports`
    Extended the ``StringValidator``, ``NumberValidator``, ``ExpressionValidator``, ``RegExpValidator`` and ``XPathValidator`` with optional ``successMessage`` and ``failureMessage`` inputs to add user-friendly messages in the resulting report. | GITB TDL | :ref:`handlers-StringValidator`, :ref:`handlers-NumberValidator`, :ref:`handlers-ExpressionValidator`, :ref:`handlers-RegExpValidator`, :ref:`handlers-XPathValidator`
    Applied consistent naming (with backwards compatibility through aliases) for the ``NumberValidator``, ``StringValidator``, ``XPathValidator``, ``XsdValidator``, ``SchematronValidator``, ``XsltProcessor`` and ``JsonPointerProcessor`` built-in step handlers. | GITB TDL | :ref:`handlers-predefined-handlers`
    Allow **scriptlet inputs** for flags used at test case load time to refer to **configuration properties**.  | GITB TDL | :ref:`scriptlets_dynamic_references`, :ref:`scriptlets_dynamic_steps`
    Extend the ``interact`` step with a ``blocking`` attribute (by default "false") to automatically proceed to the next step in case only ``instruct`` elements are defined. | GITB TDL | :ref:`tdl-step-interact_instruct_presentation`
    Extend the ``request`` elements of ``interact`` steps to allow specifying inputs as ``required`` (optional remaining the default). | GITB TDL | :ref:`tdl-step-interact_form_inputs`
    Consider as optional the ``from`` and ``to`` of ``receive`` steps defaulting, respectively, to the SUT and non-SUT actors. | GITB TDL | :ref:`tdl-step-receive_actors`
    Consider as optional the ``from`` and ``to`` of ``send`` steps defaulting, respectively, to the non-SUT and SUT actors. | GITB TDL | :ref:`tdl-step-send_actors`
    Extend the ``assign`` step with the ``byValue`` attribute to allow choosing by-value or by-reference assignment for maps and lists. | GITB TDL | :ref:`tdl-step-assign-by-value`
    Remove the **scope isolation** requirement for remotely loaded **scriptlets**, making them identical behaviour-wise to local ones. | GITB TDL | :ref:`scriptlets`

Previous documentation versions
-------------------------------

The following table provides the main highlights introduced in each previous documentation release as well as its access link.

.. csv-table::
    :header: "Release", "Key highlights", "Documentation link"
    :stub-columns: 1
    :delim: |

    1.25.2| Maintenance release with no changes. | https://www.itb.ec.europa.eu/docs/tdl/1.25.2/
    1.25.1| Maintenance release with no changes. | https://www.itb.ec.europa.eu/docs/tdl/1.25.1/
    1.25.0| Test case groups; severity levels and inverted outcomes for ``send``, ``receive``, ``process`` and ``verify`` steps; ``exit`` step and output messages for undefined results; ``stopOnChildError`` and ``hiddenContainer`` flags; new ``XPathProcessor`` processing handler. | https://www.itb.ec.europa.eu/docs/tdl/1.25.0/
    1.24.4| Maintenance release with no changes. | https://www.itb.ec.europa.eu/docs/tdl/1.24.4/
    1.24.3| Maintenance release with no changes. | https://www.itb.ec.europa.eu/docs/tdl/1.24.3/
    1.24.2| Maintenance release with no changes. | https://www.itb.ec.europa.eu/docs/tdl/1.24.2/
    1.24.1| Maintenance release with no changes. | https://www.itb.ec.europa.eu/docs/tdl/1.24.1/
    1.24.0| New ``VariableUtils`` processor, new ``CollectionUtils`` operations, new ``HttpMessagingV2`` timeout options, and optional scriptlet parameters. | https://www.itb.ec.europa.eu/docs/tdl/1.24.0/
    1.23.1| Maintenance release with no changes. | https://www.itb.ec.europa.eu/docs/tdl/1.23.1/
    1.23.0| New ``HttpMessagingV2`` and ``SoapMessagingV2`` messaging handlers, new ``$SYSTEM{apiKey}`` usable in test cases, and ``INFO`` logging level set as the default. | https://www.itb.ec.europa.eu/docs/tdl/1.23.0/
    1.22.0| Extended ``interact`` step for admin-only input, minimisation, asynchronous completion, and timeouts; normative references for test suites and test cases; configurable result for ``DisplayProcessor``. | https://www.itb.ec.europa.eu/docs/tdl/1.22.0/
    1.21.1| Maintenance release with no changes.| https://www.itb.ec.europa.eu/docs/tdl/1.21.1/
    1.21.0| New ``forceDisplay`` attribute for interact step, new embedded processing handlers and operations (``DelayProcessor``, ``JsonPointerProcessor``, ``random`` operation of ``TokenGenerator``), test case tags, test suite update metadata, optional and disabled test cases.| https://www.itb.ec.europa.eu/docs/tdl/1.21.0/
    1.20.1| Maintenance release with no changes.| https://www.itb.ec.europa.eu/docs/tdl/1.20.1/
    1.20.0| Maintenance release with no changes.| https://www.itb.ec.europa.eu/docs/tdl/1.20.0/
    1.19.0| New operations for the ``CollectionUtils`` embedded processor, extensions to the ``STEP_SUCCESS``, ``STEP_STATUS`` maps to refer to steps within scriptlets.| https://www.itb.ec.europa.eu/docs/tdl/1.19.0/
    1.18.1| Maintenance release with no changes.| https://www.itb.ec.europa.eu/docs/tdl/1.18.1/
    1.18.0| New ``XmlValidator`` and ``ExpressionValidator``; updates to ``TokenGenerator``, ``SimulatedMessaging``, ``DisplayProcessor``, ``XsdValidator`` and ``SchematronValidator``; static ``if`` steps, dynamic ``hidden`` settings in scriptlets, ``SESSION`` map with test session metadata.|https://www.itb.ec.europa.eu/docs/tdl/1.18.0/
    1.17.0| Dynamic information display within scriptlets, new ``TemplateProcessor``, ``XsltProcessor``, ``DisplayProcessor`` and ``SimulatedMessaging`` handlers, non-transactional messaging steps, upgrade to XPath 3.0.|https://www.itb.ec.europa.eu/docs/tdl/1.17.0/
    1.16.1| Maintenance release with no changes.| https://www.itb.ec.europa.eu/docs/tdl/1.16.1/
    1.16.0| Namespace support, introduction of ``STEP_STATUS`` variable, and ``supportsParallelExecution`` setting for test cases.| https://www.itb.ec.europa.eu/docs/tdl/1.16.0/
    1.15.1| Maintenance release with no changes.| https://www.itb.ec.europa.eu/docs/tdl/1.15.1/
    1.15.0| Logging levels, simplified ``process`` and call ``steps``, and styling of replies in messaging steps.| https://www.itb.ec.europa.eu/docs/tdl/1.15.0/
    1.14.1| Maintenance release with no changes.| https://www.itb.ec.europa.eu/docs/tdl/1.14.1/
    1.14.0| Syntax-aware processing on ``interact`` steps via the ``mimeType`` attribute and support for several input types via the ``inputType`` attribute.| https://www.itb.ec.europa.eu/docs/tdl/1.14.0/
    1.13.0| Support for steps to be defined as ``hidden`` and ``collapsed``, display of ``group`` steps, and option to display ``process`` steps.| https://www.itb.ec.europa.eu/docs/tdl/1.13.0/
    1.12.0| Support for sharing resources and scriptlets across test suites, new ``RegExpProcessor`` and ``CollectionUtils`` processors, extensions to the ``TokenGenerator`` and ``SchematronValidator``.| https://www.itb.ec.europa.eu/docs/tdl/1.12.0/
    1.11.1| Referencing a missing variable evaluates as an empty string.| https://www.itb.ec.europa.eu/docs/tdl/1.11.1/
    1.11.0| Stop on errors, test case output messages, support for ``verify`` output data, new ``TEST_SUCCESS`` variable, and custom ``interact`` titles.| https://www.itb.ec.europa.eu/docs/tdl/1.11.0/
    1.10.2| Maintenance release with no changes.| https://www.itb.ec.europa.eu/docs/tdl/1.10.2/
    1.10.1| Maintenance release with no changes.| https://www.itb.ec.europa.eu/docs/tdl/1.10.1/
    1.10.0| Dynamic test case imports, custom titles, new ``log`` step, imported documentation and extensions to endpoint parameters.| https://www.itb.ec.europa.eu/docs/tdl/1.10.0/
    1.9.1| Maintenance release with no changes.| https://www.itb.ec.europa.eu/docs/tdl/1.9.1/
    1.9.0| Automatic variable creation from ``assign`` steps, rich ``documentation`` for test cases and test suites and new ``Base64Processor``.| https://www.itb.ec.europa.eu/docs/tdl/1.9.0/
    1.8.0| Support for hidden actors, warning-level verifications and extended test step documentation.| https://www.itb.ec.europa.eu/docs/tdl/1.8.0/
    1.7.2| Support for any expression to be used as a template (``asTemplate``).| https://www.itb.ec.europa.eu/docs/tdl/1.7.2/
    1.7.1| Maintenance release with no changes.| https://www.itb.ec.europa.eu/docs/tdl/1.7.1/
    1.7.0| Organisation and system properties and extended configuration options for parameters.| https://www.itb.ec.europa.eu/docs/tdl/1.7.0/
    1.6.1| Extensions to the ``TokenGenerator`` to better handle timestamps and documentation on template file use.| https://www.itb.ec.europa.eu/docs/tdl/1.6.1/
    1.6.0| New ``TokenGenerator`` and ``XmlMatchValidator`` handlers, optional processing transactions, and selection lists for the ``interact`` step.| https://www.itb.ec.europa.eu/docs/tdl/1.6.0/
    1.5.0| Improved and new embedded validation handlers (``XPathValidator``, ``RegExpValidator``), improvements to steps (``exit``, ``if``) and management of reasonable defaults.| https://www.itb.ec.europa.eu/docs/tdl/1.5.0/
    1.4.1| Extensions to HttpMessaging capabilities.| https://www.itb.ec.europa.eu/docs/tdl/1.4.1/
    1.4.0| Significant configuration and parameterisation extensions, improved SOAP support and introduced timeouts.| https://www.itb.ec.europa.eu/docs/tdl/1.4.0/
    1.3.0| Release 1.3.0 for the GITB TDL documentation.| https://www.itb.ec.europa.eu/docs/tdl/1.3.0/
    1.2.0| First release for the GITB TDL documentation.| https://www.itb.ec.europa.eu/docs/tdl/1.2.0/