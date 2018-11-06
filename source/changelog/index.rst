.. index:: Change log
.. _changelog:

Change history
==============

The current section provides an overview of new and notable changes in the latest GITB TDL documentation release
as well as access to all previous documentation versions.

What's new in release 1.5.0
---------------------------

The table that follows summarises the notable changes in the latest documentation release that matches the latest
GITB TDL and GITB software release. In each case the "type" indicates whether the change applies to the GITB TDL as 
a whole or specific improvements in the GITB software (or both) . In addition, references are provided to the relevant sections 
for more information.

.. csv-table::
    :header: "Description", "Type", "Relevant sections"
    :delim: ~

    The ``XPathValidator`` is now updated to support XPath 3.0 expressions (from the previous limitation to XPath 1.0)~ GITB software~ :ref:`handlers-XPathValidator`
    Support for any type of input that can be converted to ``string``s can now be passed to the ``XPathValidator`` and ``StringValidator``~ GITB software~ :ref:`handlers-XPathValidator`, :ref:`handlers-StringValidator`
    Added a new ``RegExpValidator`` to test text against a regular expression~ GITB software~ :ref:`handlers-RegExpValidator`
    Improved default handling of the ``with``, ``type`` and ``contentType`` properties linked to user interactions~ GITB software~ :ref:`tdl-step-interact`, :ref:`test-case-preliminary`
    Allowed user interactions to present binary content for download~ GITB software~ :ref:`tdl-step-interact`, :ref:`test-case-preliminary`
    The test suite name can now be ommitted in the value provided for test case ``import`` elements~ GITB software~ :ref:`test-case-imports`
    It is now possible to define an actor as the specification's default to have it automatically selected for new conformance statements~ GITB TDL~ :ref:`test-suite-actors`
    The display order of actors in test execution diagrams can now be set at test suite and test case level~ GITB TDL~ :ref:`test-suite-actors` (test suite), :ref:`test-case-actors` (test case)
    The ``exit`` step is now correctly reflected on the user interface and can be set to be either a success or a failure~ GITB TDL~ :ref:`tdl-step-exit`
    The ``if`` step now considers the ``else`` block as optional~ GITB TDL~ :ref:`tdl-step-if`

Previous documentation versions
-------------------------------

The following table provides the main highlights introduced in each previous documentation release as well as its access link.

.. csv-table::
    :header: "Release", "Key highlights", "Documentation link"
    :stub-columns: 1
    :delim: ~

    1.4.1~ Extensions to HttpMessaging capabilities~ https://www.itb.ec.europa.eu/docs/tdl/1.4.1/
    1.4.0~ Significant configuration and parameterisation extensions, improved SOAP support and introduced timeouts~ https://www.itb.ec.europa.eu/docs/tdl/1.4.0/
    1.3.0~ Release 1.3.0 for the GITB TDL documentation~ https://www.itb.ec.europa.eu/docs/tdl/1.3.0/
    1.2.0~ First release for the GITB TDL documentation~ https://www.itb.ec.europa.eu/docs/tdl/1.2.0/