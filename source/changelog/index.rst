.. index:: Change log
.. _changelog:

Change history
==============

The current section provides an overview of new and notable changes in the latest GITB TDL documentation release
as well as access to all previous documentation versions.

What's new in release 1.18.1
----------------------------

.. note::
    **Maintenance release:** Release 1.18.1 is a global maintenance release that did not change the GITB TDL. The latest features presented
    in this section correspond to the latest release that did introduce changes, i.e. release 1.18.0.

The table that follows summarises the notable changes in the latest documentation release that matches the latest
GITB TDL and GITB software release. In each case the "type" indicates whether the change applies to the GITB TDL as 
a whole or specific improvements in the GITB software (or both) . In addition, references are provided to the relevant sections 
for more information.

.. csv-table::
    :header: "Description", "Type", "Relevant sections"
    :delim: |

    New ``XmlValidator`` validation handler to validate XML against both XSDs and Schematrons. | GITB TDL | :ref:`handlers-XmlValidator`
    New ``ExpressionValidator`` validation handler to check arbitrary expressions. | GITB TDL | :ref:`handlers-ExpressionValidator`
    The ``TokenGenerator`` now accepts an optional prefix and postfix to add to generated UUIDs. | GITB TDL | :ref:`handlers-TokenGenerator`
    The ``SimulatedMessaging`` and ``DisplayProcessor`` handlers can now also specify the content types of data in their reports. | GITB TDL | :ref:`handlers-simulatedmessaging`, :ref:`handlers-DisplayProcessor`
    The ``reply`` attribute of messaging steps can now be set dynamically within scriptlets. | GITB TDL | :ref:`scriptlets_dynamic_references`
    The ``call`` step can now also be set as ``hidden`` to hide all steps of the referenced scriptlet. | GITB TDL | :ref:`tdl-step-call`
    The ``SchematronValidator`` can now show the assertions made for each finding. | GITB TDL | :ref:`handlers-SchematronValidator`
    The ``XSDValidator`` and ``SchematronValidator`` can now present findings sorted by their severity. | GITB TDL | :ref:`handlers-XSDValidator`, :ref:`handlers-SchematronValidator`
    The ``XSDValidator`` and ``SchematronValidator`` can now hide from their report the XSD or Schematron used for the validation. | GITB TDL | :ref:`handlers-XSDValidator`, :ref:`handlers-SchematronValidator`
    Scriptlet parameters can now be set with default values. | GITB TDL | :ref:`scriptlets_elements_params`
    ``flow`` steps can now have individually hidden ``thread`` blocks. | GITB TDL | :ref:`tdl-step-flow`
    Steps' ``hidden`` attribute can now be set in scriptlets via variable reference. | GITB TDL | :ref:`scriptlets_dynamic_steps`
    ``if`` steps can be set as ``static`` for dynamic inclusion of steps within scriptlets. | GITB TDL | :ref:`tdl-step-if_hide_boundary`, :ref:`scriptlets_dynamic_steps`
    ``if`` steps can now be set as ``hidden`` with visible child steps. | GITB TDL | :ref:`tdl-step-if_hide_boundary`
    New ``SESSION`` map allowing access to test session metadata (session identifier, test case identifier, TDL version). | GITB TDL | :ref:`test-case-expressions-session-metadata`
    Actor configuration (endpoint) parameters can now be set with default values and user friendly names. | GITB TDL | :ref:`test-suite-actors`
    Using a ``process`` step via its simplified syntax now supports mapping of the input to a parameter based on the defined types. | GITB TDL | :ref:`tdl-step-process__simplified`
    Allow the ``assign`` step to create on-the-fly maps and lists at any nesting level. | GITB TDL | :ref:`tdl-step-assign`
    New ``TestCaseOverviewReport`` root element of type ``TestCaseOverviewReportType`` in the ``gitb_tr.xsd`` for an XML test case overview report. | GITB TDL | :ref:`introduction_spec_links`

Previous documentation versions
-------------------------------

The following table provides the main highlights introduced in each previous documentation release as well as its access link.

.. csv-table::
    :header: "Release", "Key highlights", "Documentation link"
    :stub-columns: 1
    :delim: |

    1.18.0| New ``XmlValidator`` and ``ExpressionValidator``; updates to ``TokenGenerator``, ``SimulatedMessaging``, ``DisplayProcessor``, ``XSDValidator`` and ``SchematronValidator``; static ``if`` steps, dynamic ``hidden`` settings in scriptlets, ``SESSION`` map with test session metadata.|https://www.itb.ec.europa.eu/docs/tdl/1.18.0/
    1.17.0| Dynamic information display within scriptlets, new ``TemplateProcessor``, ``XSLTProcessor``, ``DisplayProcessor`` and ``SimulatedMessaging`` handlers, non-transactional messaging steps, upgrade to XPath 3.0.|https://www.itb.ec.europa.eu/docs/tdl/1.17.0/
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