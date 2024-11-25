.. index:: Change log
.. _changelog:

Change history
==============

The current section provides an overview of new and notable changes in the latest GITB TDL documentation release
as well as access to all previous documentation versions.

What's new in release 1.24.4
----------------------------

.. note::
    **Maintenance release:** Release 1.24.4 is a global maintenance release that did not change the GITB TDL. The latest features presented
    in this section correspond to the latest release that did introduce changes, i.e. release 1.24.0.

The table that follows summarises the notable changes in the latest documentation release that matches the latest
GITB TDL and GITB software release. In each case the "type" indicates whether the change applies to the GITB TDL as 
a whole or specific improvements in the GITB software (or both) . In addition, references are provided to the relevant sections 
for more information.

.. csv-table::
    :header: "Description", "Type", "Relevant sections"
    :delim: |

    Support for optional scriptlet parameters and parameters set as empty by default. | GITB TDL | :ref:`scriptlets_elements_params`
    New ``VariableUtils`` built-in processor for common operations (``exists``, ``type``) on variables. | GITB TDL | :ref:`handlers-VariableUtils`
    New operation for the ``CollectionUtils`` built-in processor to append collections. | GITB TDL | :ref:`handlers-CollectionUtils`
    New inputs for the ``HttpMessagingV2`` built-in messaging handler to set maximum timeouts (connection and request) when calling a remote service. | GITB TDL | :ref:`handlers-httpmessagingv2-send`

Previous documentation versions
-------------------------------

The following table provides the main highlights introduced in each previous documentation release as well as its access link.

.. csv-table::
    :header: "Release", "Key highlights", "Documentation link"
    :stub-columns: 1
    :delim: |

    1.24.3| Maintenance release with no changes. | https://www.itb.ec.europa.eu/docs/tdl/1.24.3/
    1.24.2| Maintenance release with no changes. | https://www.itb.ec.europa.eu/docs/tdl/1.24.2/
    1.24.1| Maintenance release with no changes. | https://www.itb.ec.europa.eu/docs/tdl/1.24.1/
    1.24.0| New ``VariableUtils`` processor, new ``CollectionUtils`` operations, new ``HttpMessagingV2`` timeout options, and optional scriptlet parameters. | https://www.itb.ec.europa.eu/docs/tdl/1.24.0/
    1.23.1| Maintenance release with no changes. | https://www.itb.ec.europa.eu/docs/tdl/1.23.1/
    1.23.0| New ``HttpMessagingV2`` and ``SoapMessagingV2`` messaging handlers, new ``$SYSTEM{apiKey}`` usable in test cases, and ``INFO`` logging level set as the default. | https://www.itb.ec.europa.eu/docs/tdl/1.23.0/
    1.22.0| Extended ``interact`` step for admin-only input, minimisation, asynchronous completion, and timeouts; normative references for test suites and test cases; configurable result for ``DisplayProcessor``. | https://www.itb.ec.europa.eu/docs/tdl/1.22.0/
    1.21.1| Maintenance release with no changes.| https://www.itb.ec.europa.eu/docs/tdl/1.21.1/
    1.21.0| New ``forceDisplay`` attribute for interact step, new embedded processing handlers and operations (``DelayProcessor``, ``JSONPointerProcessor``, ``random`` operation of ``TokenGenerator``), test case tags, test suite update metadata, optional and disabled test cases.| https://www.itb.ec.europa.eu/docs/tdl/1.21.0/
    1.20.1| Maintenance release with no changes.| https://www.itb.ec.europa.eu/docs/tdl/1.20.1/
    1.20.0| Maintenance release with no changes.| https://www.itb.ec.europa.eu/docs/tdl/1.20.0/
    1.19.0| New operations for the ``CollectionUtils`` embedded processor, extensions to the ``STEP_SUCCESS``, ``STEP_STATUS`` maps to refer to steps within scriptlets.| https://www.itb.ec.europa.eu/docs/tdl/1.19.0/
    1.18.1| Maintenance release with no changes.| https://www.itb.ec.europa.eu/docs/tdl/1.18.1/
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