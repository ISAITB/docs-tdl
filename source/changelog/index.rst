.. index:: Change log
.. _changelog:

Change history
==============

The current section provides an overview of new and notable changes in the latest GITB TDL documentation release
as well as access to all previous documentation versions.

What's new in release 1.4.0 
---------------------------

.. note::
    **Maintenance release:** Release 1.4.0 is a global maintenance release that did not change the GITB TDL. The latest features presented
    in this section correspond to the latest release that did introduce changes, i.e. release 1.3.0.

The table that follows summarises the notable changes in the latest documentation release that matches the latest
GITB TDL and GITB software release. In each case the "type" indicates whether the change applies to the GITB TDL as 
a whole or specific improvements in the GITB software (or both) . In addition, references are provided to the relevant sections 
for more information.

.. csv-table::
    :header: "Description", "Type", "Relevant sections"
    :delim: ~

    Transactions (messaging and processing) now support ``config`` elements to customise transaction creation~ GITB TDL~ :ref:`tdl-step-btxn`, :ref:`tdl-step-bptxn`
    Configuration elements (``config``) now support variable references~ GITB TDL~ :ref:`tdl-step-btxn`, :ref:`tdl-step-send`, :ref:`tdl-step-receive`, :ref:`tdl-step-listen`, :ref:`tdl-step-bptxn`, :ref:`tdl-step-verify`
    User instructions now consider "string" as the default type and can be fully empty in case of simple messages~ GITB TDL~ :ref:`tdl-step-interact`
    User requests now consider "STRING" as the default content type~ GITB TDL~ :ref:`tdl-step-interact`
    The ``foreach`` step now allows its boundaries (start and end) to be set using variable references~ GITB TDL~ :ref:`tdl-step-foreach`
    Actors can now be defined in test suites without endpoints~ GITB TDL~ :ref:`test-suite-actors`
    Service handler implementations can now be specified with variable references~ GITB TDL~ :ref:`handlers-implementation`
    The ``receive`` step now allows a timeout to be specified~ GITB TDL~ :ref:`tdl-step-receive`
    Communication with service handlers now supports authentication (HTTP Basic and UsernameToken)~ GITB TDL and software~ :ref:`handlers-implementation`
    The ``SoapMessaging`` embedded messaging handler now supports HTTPS~ GITB software~ :ref:`handlers-soapmessaging`
    Value 'CONFORMANCE' is now correctly considered as the default test case type~ GITB software~ :ref:`test-case-metadata`
    The actor name defined in a test case does not have to match the actor ID~ GITB software~ :ref:`test-case-actors`
    Domain-level configuration parameters can now be used in expressions~ GITB software~ :ref:`test-case-expressions-domain`

Previous documentation versions
-------------------------------

The following table provides the main highlights introduced in each previous documentation release as well as the 
link its access link.

.. csv-table::
    :header: "Release", "Key highlights", "Documentation link"
    :stub-columns: 1

    1.3.0, Release 1.3.0 for the GITB TDL documentation, https://www.itb.ec.europa.eu/docs/tdl/1.3.0/
    1.2.0, First release for the GITB TDL documentation, https://www.itb.ec.europa.eu/docs/tdl/1.2.0/