.. index:: Change log
.. _changelog:

Change history
==============

The current section provides an overview of new and notable changes in the latest GITB TDL documentation release
as well as access to all previous documentation versions.

What's new in release 1.12.0
----------------------------

The table that follows summarises the notable changes in the latest documentation release that matches the latest
GITB TDL and GITB software release. In each case the "type" indicates whether the change applies to the GITB TDL as 
a whole or specific improvements in the GITB software (or both) . In addition, references are provided to the relevant sections 
for more information.

.. csv-table::
    :header: "Description", "Type", "Relevant sections"
    :delim: |

    New embedded processing handler ``RegExpProcessor`` to process texts using regular expressions. | GITB TDL | :ref:`handlers-RegExpProcessor`
    New embedded processing handler ``CollectionUtils`` providing utility functions for maps and lists. | GITB TDL | :ref:`handlers-CollectionUtils`
    The ``TokenGenerator`` processing handler now allows manipulation of dates based on existing, formatted, date values. | GITB TDL | :ref:`handlers-TokenGenerator`
    The ``SchematronValidator`` validation handler now supports defining the Schematron file type (XSLT or SCH) as an input. | GITB TDL | :ref:`handlers-SchematronValidator`

Previous documentation versions
-------------------------------

The following table provides the main highlights introduced in each previous documentation release as well as its access link.

.. csv-table::
    :header: "Release", "Key highlights", "Documentation link"
    :stub-columns: 1
    :delim: |

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