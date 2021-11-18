.. index:: Change log
.. _changelog:

Change history
==============

The current section provides an overview of new and notable changes in the latest GITB TDL documentation release
as well as access to all previous documentation versions.

What's new in release 1.15.0
----------------------------

The table that follows summarises the notable changes in the latest documentation release that matches the latest
GITB TDL and GITB software release. In each case the "type" indicates whether the change applies to the GITB TDL as 
a whole or specific improvements in the GITB software (or both) . In addition, references are provided to the relevant sections 
for more information.

.. csv-table::
    :header: "Description", "Type", "Relevant sections"
    :delim: |

    Messaging steps now support the ``reply`` attribute to indicate that they should be presented as reply messages. | GITB TDL | :ref:`tdl-step-send`, :ref:`tdl-step-receive`, :ref:`tdl-step-listen`
    Support for automatically converting non-list values to lists (as single-item lists containing the value). | GITB TDL | :ref:`test-case-types-type-conversions`
    Support for automatically converting lists and maps to strings. | GITB TDL | :ref:`test-case-types-type-conversions`
    The ``log`` step now supports a ``level`` attribute to define the message's severity level. | GITB TDL | :ref:`tdl-step-log`
    Test cases now support the ``logLevel`` attribute to control which messages are included in test session logs. | GITB TDL | :ref:`test-case-steps`
    The severity level of ``verify`` steps can be now be provided dynamically via variable reference. | GITB TDL | :ref:`tdl-step-verify`
    The ``process`` step now enables simplified usage through the ``input``, ``operation`` and ``output`` attributes. | GITB TDL | :ref:`tdl-step-process__simplified`
    The ``call`` step now enables simplified usage through the ``input`` and ``output`` attributes. | GITB TDL | :ref:`tdl-step-call__simplified`

Previous documentation versions
-------------------------------

The following table provides the main highlights introduced in each previous documentation release as well as its access link.

.. csv-table::
    :header: "Release", "Key highlights", "Documentation link"
    :stub-columns: 1
    :delim: |

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