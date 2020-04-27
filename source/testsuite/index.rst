Test Suites
===============================

Overview
--------

The purpose of a test suite is to group test cases into a cohesive set and define the actors
that its test cases involve. In addition, test suites introduce metadata such as a version
number and a description to facilitate their identification and management.

The following is an example test suite that defines a single actor and two included test cases:

.. code-block:: xml

    <?xml version="1.0" encoding="UTF-8"?>
    <testsuite id="UBL_invoice_validation" xmlns="http://www.gitb.com/tdl/v1/" xmlns:gitb="http://www.gitb.com/core/v1/" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:schemaLocation="http://www.gitb.com/tdl/v1/ ../gitb_tdl.xsd">
        <metadata>
            <gitb:name>UBL_invoice_validation</gitb:name>
            <gitb:type>CONFORMANCE</gitb:type>
            <gitb:version>0.1</gitb:version>
            <gitb:description>A test suite to validate UBL invoices uploaded by a user</gitb:description>
        </metadata>
        <actors>
            <gitb:actor id="User">
                <gitb:name>User</gitb:name>
                <gitb:desc>User to upload a UBL invoice for validation</gitb:desc>
                <gitb:endpoint name="userInfo">
                    <gitb:config name="id" kind="SIMPLE"/>
                </gitb:endpoint>
            </gitb:actor>
        </actors>
        <testcase id="UBL_invoice_validation_test_1"/>
        <testcase id="UBL_invoice_validation_test_2"/>
    </testsuite>

A test suite is defined as the XML file's root element ``testsuite``. The following table defines its attributes and child elements:

.. csv-table::
    :stub-columns: 1
    :header: "Name", "Required?", "Description"

    @id, yes, A string to uniquely identify the test suite by.
    metadata, yes, A block containing the metadata used to describe the test suite.
    actors, yes, The list of actors that relate to the test suite's test cases.
    testcase, yes, One or more test cases that are included in the test suite.

.. note::
    **GITB software support:** Currently the ``id`` attribute of a test suite is ignored in favour of the test suite's name that is used to uniquelly identify
    the test suite within a specification. This is likely to be adapted in future versions so it is a good practice to provide meaningful values for the ``id``. 
    A good approach is to use the same value as for the test suite name.

Elements
--------

Here we will see how a test suite breaks down into its individual sections and discuss the purpose of each. 

.. index:: Metadata (Test suites)
.. _test-suite-metadata:

Metadata
~~~~~~~~

The purpose of the ``metadata`` element is to provide basic information about the test suite. This information is used both by administrators to better
manage existing test suites but also for end users to understand the test suite's purpose. The structure of the ``metadata`` element is as follows:

.. csv-table::
    :stub-columns: 1
    :header: "Name", "Required?", "Description"

    name, yes, The name of the test suite that is used to identify it to users.
    type, no, Either "CONFORMANCE" (the default) or "INTEROPERABILITY". "INTEROPERABILITY" is used when multiple systems under test are considered in the test suite's test cases.
    version, yes, A string that indicates the test suite's version.
    authors, no, A string to indicate the test suite's authors.
    description, no, A string to provide a user-friendly description of the test suite that is displayed to users.
    published, no, A string acting as an indication of the test suite's publishing time.
    lastModified, no, A string acting as an indication of the last modification time for the test suite.
    documentation, no, Rich text content that provides further information on the current test suite.

.. index:: documentation (test suite)

The ``documentation`` element complements the test suite's ``description`` by allowing the test suite's author to include extended rich text documentation. This documentation can 
provide further information on the context of the test suite, diagrams or reference information that are useful to understand how it is to be completed or its purpose within the
overall specification. The content supplied supports several HTML features:

    * Structure elements (e.g. headings, text blocks, lists).
    * In-line styling.
    * Tables.
    * Links.
    * Images.

The simplest way to provide such information is to enclose the HTML content in a CDATA section to ensure the XML remains well-formed. The
following sample provides an example of this approach:

.. code-block:: xml

    <testsuite id="TS1" xmlns="http://www.gitb.com/tdl/v1/" xmlns:gitb="http://www.gitb.com/core/v1/">
        <metadata>
            <gitb:name>TS1</gitb:name>
            <gitb:version>1.0</gitb:version>
            <gitb:description>A short description of the test suite to offer a short summary of its purpose.</gitb:description>
            <gitb:documentation><![CDATA[
                <p>Extended documentation for test suite <b>TS1</b></p>
                <p>This is an example to support the <a href="https://www.itb.ec.europa.eu/docs/tdl/latest">GITB TDL docs</a>.</p>
            ]]></gitb:documentation>
        </metadata>    
        ...
    </testsuite>

Note that documentation such as this is also supported for:

    * The :ref:`test cases<test-case-metadata>` included in the test suite.
    * Individual :ref:`test case steps<tdl-steps-common-documentation>`.

.. note::
    **GITB software support:** The ``name`` attribute is used to uniquely identify the test suite within a specification so ensure that it's unique 
    within a given specification. An uploaded test suite whose ``name`` matches that of an existing test suite will result in the existing test suite
    being updated. Furthermore, the ``version`` value is used only for display purposes whereas the ``authors``, ``published`` and ``lastModified`` 
    values are recorded but never used or displayed. Finally, the "INTEROPERABILITY" ``type`` (defined at test suite level) is currently ignored.

.. index:: Actors (Test suites)
.. _test-suite-actors:

Actors
~~~~~~

In the ``actors`` element we identify the actors that will be involved in the test suite's test cases. These actors may either be ones that will be tested 
as systems under test (i.e. the focus of test cases) or be simulated by the test bed. At least one actor needs to be defined here. The ``actors`` element 
contains one or more ``actor`` elements with structure as follows:

.. csv-table::
    :stub-columns: 1
    :header: "Name", "Required?", "Description"

    @id, yes, The unique identifier for the actor.
    @default, no, Whether or not the actor is to be considered as the specification's default actor (``false`` by default).
    @hidden, no, Whether or not the actor will be unavailable for use in conformance statements (``false`` by default).
    @displayOrder, no, A number indicating the relative positioning that needs to be respected when displaying the actor in test execution diagrams.
    name, yes, A user-friendly name for the actor.
    desc, no, A description to provide additional information on the purpose of this actor in the specification.
    endpoint, no, Zero or more ``endpoint`` elements that capture an actor's configuration. 

The value for the ``id`` attribute is very important as it is used internally to link the test suite and its test cases to the relevant actor in the specification.
The ``name`` and ``desc`` elements are present as metadata when displaying a test case to a user but are not important with respect to test case steps and logic. 
The ``default`` attribute can be useful in cases where a specification defines multiple actors but only one is ever expected to be used as the SUT. Setting this to 
``true`` indicates that this actor should be preselected when creating new conformance statements.

Another means of managing the actors that are to be used in
conformance statements is the ``hidden`` attribute which, if set to ``true``, will remove the actor from the ones available to create a conformance statement. This 
can useful when certain actors are not meant to be selected for testing (but there is no single default actor to otherwise set), or if an actor has been in use
but should now be deprecated. Setting an actor as ``hidden`` will not affect previous testing history but will make it unavailable for future conformance
statements.

The ``displayOrder`` attribute provides an indication on how the actor should be positioned in test execution diagrams relevant to other actors. This could be useful
to set if you want an actor to always appear first in diagrams regardless of the TDL steps that a test case defines (e.g. to always show the SUT first). When the 
``displayOrder`` attribute for an actor is smaller relevant to others, or if no ``displayOrder`` is specified for other actors, the actor will appear first. Note that
this ordering applies to actors defined in the specification, not special-purpose actor lifelines that could signify the test bed or the user. Finally, note that any 
setting that is made for an actor at test suite level is considered as a default and can be overridden at test case level (see :ref:`test-case-actors`).

Actors used in test suites may need to have configuration properties related to them that have meaning for the given specification or that are required to run 
test cases. Examples of such configuration properties could be a Member State country code, an IP address or a certificate used to produce and verify signatures
(i.e. these can be simple values or complete files). For an actor as a system under test (SUT), these are properties that would need to be provided before executing
a test case.

Such actor configuration is captured in configuration sets named "endpoints" with each one defining key-value pair configuration properties named
"parameters". In terms of their XML representation, an endpoint is defined using the ``endpoint`` element as follows:

.. csv-table::
    :stub-columns: 1
    :header: "Name", "Required?", "Description"

    @name, yes, The name of the endpoint that must be unique for the actor.
    @desc, no, A description to explain the purpose of this endpoint.
    config, yes, One or more elements to define each of the endpoint's parameters. 

The ``config`` elements defining an endpoint's parameters are structured as follows:

.. csv-table::
    :stub-columns: 1
    :header: "Name", "Required?", "Description"
    :delim: |

    @name| yes| The name of the parameter that must be unique for the endpoint.
    @desc| no| A description to explain the purpose of this parameter.
    @use| no| Whether this is a required (value "R" - the default) or optional ("O") parameter. 
    @kind| no| Whether this is a simple text (value "SIMPLE" - the default), file (value "BINARY") or a secret value (value "SECRET").
    @adminOnly| no| A boolean value (by default "false") indicating whether this parameter can only be edited by administrators.
    @notForTests| no| A boolean value (by default "false") indicating whether this parameter is included as a test session context variable.

In terms of the ``kind`` attribute, the values "SIMPLE" and "SECRET" both represent text values. The difference is that ones 
defined as "SECRET" are never presented to users nor are they ever transferred to client software. The two attributes ``adminOnly``
and ``notForTests`` are used to increase the options available in defining a flexible and complete conformance testing strategy.

Setting the ``adminOnly`` flag to "true" would typically be used for required parameters (i.e. ``use`` set to "R") as 
doing so allows community or test bed administrators to verify and adapt a system's setup before allowing it to
execute tests. This could either be done as a simple eligibility check or to actually provide required information
that is needed for the tests (e.g. a generated identifier or certificate). If a required parameter that is set as 
``adminOnly`` is not configured, the system in question is prevented from executing tests.

The ``notForTests`` flag on the other hand would be set to "true" for information that is strictly administrative
in nature and is not actually needed during test sessions. An example of this would be to record a flag set by administrators that, if
missing, would prevent test sessions to be started (i.e. combined with the ``adminOnly`` flag as explained above).
Otherwise such attributes could also be used to enable general data collection for testing organisations pertinent
to specific conformance statements. 

Endpoints and their parameters are used in two main scenarios:

* As simple sets of configuration values.
* As placeholders for SUT actors to hold simulated actor configuration (provided via their messaging handlers).

These two scenarios are explained in the following sections.

.. index:: Endpoints (simple configuration values)

Endpoints as simple configuration values
++++++++++++++++++++++++++++++++++++++++

For the most part you will be using endpoints to simply have users configure one or more parameters for an actor that 
you can then use in a test case. An example of this would be a "person" actor defined with required "firstName" and "lastName" properties:

.. code-block:: xml

    <gitb:actor id="person">
        <gitb:name>person</gitb:name>
        <gitb:endpoint name="personInfo">
            <gitb:config name="firstName" kind="SIMPLE" use="R"/>
            <gitb:config name="lastName" kind="SIMPLE" use="R"/>
        </gitb:endpoint>
    </gitb:actor>

This means that before running a test case that has the "person" actor with role SUT, the values for "firstName" and "lastName" would
have to be provided. With these values in place, the test case could then refer to them using the expressions 
``$person{firstName}`` and ``$person{lastName}``. As you see, in this scenario the actual name of the endpoint ("personInfo" in this case)
never actually figures when referencing the values. Given this you might wonder why the endpoint name is important to provide. The answer is to
cover the more complex scenario discussed next.

.. index:: Endpoints (simulated actor configuration)
.. _test-suite-actors-endpoints-simulated:

Endpoints to map simulated actor configuration
++++++++++++++++++++++++++++++++++++++++++++++

Simulated actors are handled in GITB through **messaging transactions** and **messaging handlers** (see :ref:`introduction-concepts-messaging-handlers`). 
Any messaging interaction between actors takes place by means of a messaging handler that actually implements the simulated actor for the purpose of the transaction's 
interaction. Before a test session starts, the existing messaging transactions are detected and the pairs of communicating actors are split in "from" and "to" roles.
Each simulated actor's messaging handler then receives an **initiate** call in which the configuration for the other actors is passed (if defined).
The messaging handler has now the opportunity to return a configuration per actor that can differ based on the current global testing state or the
configuration received. Different sets of configuration properties can be returned per actor and are mapped to each one on the basis of their ``endpoint``.

To better illustrate this, consider a sender and receiver example, defined in a test suite as follows:

.. code-block:: xml

    <gitb:actor id="sender">
        <gitb:name>sender</gitb:name>
        <gitb:endpoint name="expectedConfig"/>
    </gitb:actor>
    <gitb:actor id="receiver">
        <gitb:name>sender</gitb:name>
    </gitb:actor>

These actors can then be used in a test case by defining "sender" as the SUT and "receiver" as simulated. These are both involved in a messaging transaction.

.. code-block:: xml

    <actors>
        <gitb:actor id="sender" name="sender" role="SUT"/>
        <gitb:actor id="receiver" name="receiver" role="SIMULATED"/>
    </actors>
    <steps>
        <btxn from="sender" to="receiver" txnId="t1" handler="aHandler"/>
        <send desc="Call receiver" from="sender" to="receiver" txnId="t1">
            <input name="anInputValue">$sender{receiver}{configuredValue}</input>
        </send>
        <etxn txnId="t1"/>
    </steps>

The key point to note is the reference to the "configuredValue" parameter in ``$sender{receiver}{configuredValue}``. To be able to do this the following has happened:

1. The sender SUT actor defined an endpoint named "expectedConfig".
2. The receiver simulated actor, due to its presence in a messaging transaction (see the ``btxn`` element), was called during the test session'
   initiation phase.
3. The receiver responded by providing a configuration for endpoint "expectedConfig" with a parameter named "configuredValue".
4. The returned configuration was matched to the sender's "expectedConfig" endpoint and copied under the sender actor for this endpoint.
5. The returned parameter can now be referenced as ``$sender{receiver}{configuredValue}`` (i.e. ``$SUT_ACTOR_ID{SIMULATED_ACTOR_ID}{PARAMETER_NAME}``).

.. note::
    **GITB software support:** Only a single endpoint can currently be configured for an actor. Additional endpoints will be recorded for the actor but will be 
    ignored during test execution.

.. index:: Test cases (Test suites)

Test cases
~~~~~~~~~~

This section is used to reference the test cases contained in the test suite. One or more test case entries must be defined using the 
``testcase`` element whose structure is as follows:

.. csv-table::
    :stub-columns: 1
    :header: "Name", "Required?", "Description"

    @id, yes, The ID of the test case. This needs to match one defined in the test case's XML file (see :ref:`test-case`).
    prequisite, no, Zero or more elements each defining as text content a test case ID that should be considered as a prerequisite before running this one.
    option, no, Zero or more elements each defining as text content string values that match an option defined for the actor in the specification.

.. note::
    **GITB software support:** The ``prequisite`` and ``option`` values are currently ignored.

.. _test-suite-deploying:

Deploying a test suite in the GITB software
-------------------------------------------

A test suite is packaged as a compressed ZIP archive that contains:

* A single test suite XML file.
* One or more test case XML files.
* Any number of arbitrary files used as resources within test cases.

The names of the archive, the test suite and the test case XML files are not important. Neither is the folder structure defined within the archive.
What is important is that:

* A single test suite XML file is defined.
* The test case IDs defined in the test case XML files are referenced in the test suite XML.

Uploading a test suite to the GITB software the following has the following results:

* If it doesn't previously exist, the test suite is recorded along with its test cases and linked to the appropriate specification actors.
* If the test suite does exist the user selects whether this new version should invalidate previous conformance testing sessions (if the
  change is significant) or not. Choosing to replace the test suite results in the test suite being updated and its test cases being replaced
  with the ones contained in the new version. Matching of the test suite with an existing one is on the basis of their name within the specification.
* The actors that are defined in the test suite are created if they don't already exist along with their endpoints and endpoint parameters.
* Actors that already exist in the specification are updated based on the latest provided information. In this case new endpoints and parameters are added
  and existing ones are updated. Note that actors, endpoints and parameters that are not defined in the new test suite are not removed. The matching of
  actors is on the basis of their ID, whereas for endpoints and parameters their name is used.

As previously discussed the :ref:`test-suite-actors` section serves to define which actors are used within the test suite and to provide their details (their name, endpoints
and endpoint parameters). An alternative approach to avoid defining the complete actor details in the test suite is to simply refer to the actors used
in its test cases without providing their information. Referring to actors is on the basis of their ID and referred actors are assumed and required to be present in the
target specification (resulting in an upload error otherwise).

The following example shows a test suite in which a "User" actor is referred to.

.. code-block:: xml
    :emphasize-lines: 7,8,9

    <?xml version="1.0" encoding="UTF-8"?>
    <testsuite id="UBL_invoice_validation" xmlns="http://www.gitb.com/tdl/v1/" xmlns:gitb="http://www.gitb.com/core/v1/" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:schemaLocation="http://www.gitb.com/tdl/v1/ ../gitb_tdl.xsd">
        <metadata>
            <gitb:name>UBL_invoice_validation</gitb:name>
            <gitb:version>0.2</gitb:version>
        </metadata>
        <actors>
            <gitb:actor id="User"/>
        </actors>	
        <testcase id="UBL_invoice_validation_test_1"/>
        <testcase id="UBL_invoice_validation_test_2"/>
    </testsuite>

Specifying a test suite's actors in this way could be interesting if you want to be able to manage the actors' information through the GITB software's
user interface without being concerned with keeping their definitions up to date in test suites. This would avoid for example an unwanted case where an
actor's information is updated through the user interface but gets reset when a new version of a test suite gets uploaded where the change was not reflected.
The only points you need to ensure are that the specification's actors are already defined before you start uploading test suites and that you don't change their 
ID.