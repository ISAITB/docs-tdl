.. index:: Test suites
.. _test-suite:

Test suites
===========

Overview
--------

The purpose of a test suite is to group test cases into a cohesive set and define the actors
that its test cases involve. In addition, test suites introduce metadata such as a version
number and a description to facilitate their identification and management.

The following is an example test suite that defines a single actor and two included test cases:

.. code-block:: xml

    <?xml version="1.0" encoding="UTF-8"?>
    <testsuite id="invoiceValidation" xmlns="http://www.gitb.com/tdl/v1/" xmlns:gitb="http://www.gitb.com/core/v1/">
        <metadata>
            <gitb:name>UBL invoice validation</gitb:name>
            <gitb:version>1.0</gitb:version>
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
        <testcase id="testCase1"/>
        <testcase id="testCase2"/>
    </testsuite>

A test suite is defined as the XML file's root element ``testsuite``. The following table defines its attributes and child elements:

.. csv-table::
    :stub-columns: 1
    :header: "Name", "Required?", "Description"

    @id, yes, A string to uniquely identify the test suite by.
    metadata, yes, A block containing the metadata used to describe the test suite.
    actors, no, The list of actors that relate to the test suite's test cases. If not defined the test suite is assumed to be used only for :ref:`resource sharing<test-suite-sharing>`.
    testcase, no, A set of one or more test cases that are included in the test suite. If not defined the test suite is assumed to be used only for :ref:`resource sharing<test-suite-sharing>`.

Elements
--------

Here we will see how a test suite breaks down into its individual sections and discuss the purpose of each. 

.. index:: metadata (Test suite)
.. index:: name (Test suite metadata)
.. index:: type (Test suite metadata)
.. index:: version (Test suite metadata)
.. index:: authors (Test suite metadata)
.. index:: description (Test suite metadata)
.. index:: published (Test suite metadata)
.. index:: lastModified (Test suite metadata)
.. index:: documentation (Test suite metadata)
.. index:: update (Test suite metadata)
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
    update, no, Instructions determining the default choices when an update of this test suite is taking place.

.. note::
    **GITB software support:** The test suite's ``id`` attribute is used to uniquely identify the test suite within a specification so ensure that it's unique 
    within a given specification. An uploaded test suite whose ``id`` matches that of an existing test suite will result in the existing test suite
    being updated. Furthermore, the ``version`` value is used only for display purposes whereas the ``authors``, ``published`` and ``lastModified`` 
    values are recorded but never used or displayed. Finally, the "INTEROPERABILITY" ``type`` (defined at test suite level) is currently ignored.

.. index:: documentation (Test suite)
.. index:: import (Test suite documentation)
.. index:: from (Test suite documentation)
.. index:: encoding (Test suite documentation)
.. _test-suite-metadata-documentation:

documentation
+++++++++++++

The ``documentation`` element complements the test suite's ``description`` by allowing the test suite's author to include extended rich text documentation as HTML. The structure of this element is as follows:

.. csv-table::
    :stub-columns: 1
    :header: "Name", "Required?", "Description"

    import, no, A reference to a separate file within the test suite archive that defines the documentation content.
    from, no, The identifier of a test suite from which the ``import`` file will be loaded. If unspecified, the current test suite is assumed.
    encoding, no, In case an ``import`` reference is defined this can be used to specify the file's encoding. If not provided ``UTF-8`` is considered.

Using the above attributes to specify a reference to a separate file is not mandatory. The documentation's content can also be provided as the element's text content,
typically enclosed within a CDATA section if this includes HTML elements (in which case the ``from``, ``import`` and ``encoding`` attributes are omitted).
When loading documentation from a separate file, it is also possible to lookup this file from another test suite. This is
done by specifying as the value of the ``from`` attribute the ``id`` of the target test suite. This is used to lookup the
target test suite as follows:

#. Look for the test suite in the same **specification** as the current test case.
#. If not found in the same specification, look for the test suite in the other specifications of the test case's **domain**.
   If across specifications multiple matching test suites are found, one of them will be arbitrarily picked. To avoid such
   a scenario it is obvious that you should ensure test suites used to load shared resources can be uniquely identified.

This documentation can provide further information on the context of the test suite, diagrams or reference information that are useful to understand how it is to be completed or its purpose within the
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

.. index:: update (Test suite)
.. index:: updateMetadata (Test suite update)
.. index:: updateSpecification (Test suite update)
.. _test-suite-metadata-update:

update
++++++

The ``update`` element allows the test suite's developer to prescribe what should happen when this test suite is being uploaded and
an existing test suite with the same identifier is found. Through this you can define if the test suite's existing metadata 
(e.g. name, description and documentation) and the existing specification actors should be updated to match the definitions from
the new archive. Note that these choices represent the default selected options during the test suite upload, and can always be verified
and replaced by the Test Bed's operator.

It could be interesting to use the ``update`` element if the test developer is not the one performing the test suite upload. Doing so,
avoids providing detailed instruction to operations staff, by already encoding the relevant choices within the test suite archive itself.

The structure of the ``update`` element is as follows:

.. csv-table::
    :stub-columns: 1
    :header: "Name", "Required?", "Description"

    @updateMetadata, no, A boolean value determining whether the existing test suite's metadata should be updated based on the new archive (default is "false").
    @updateSpecification, no, A boolean value determining whether the existing test suite's actor information should be updated based on the new archive (default is "false").

The following example shows how you can specify that the test suite's metadata should be updated to reflect the new values in the archive
(see attribute ``updateMetadata``). Any existing definitions of actors are left unchanged (see attribute ``updateSpecification``).

.. code-block:: xml
    :emphasize-lines: 6

    <testsuite id="TS1" xmlns="http://www.gitb.com/tdl/v1/" xmlns:gitb="http://www.gitb.com/core/v1/">
        <metadata>
            <gitb:name>TS1</gitb:name>
            <gitb:version>1.0</gitb:version>
            <gitb:description>A short description of the test suite to offer a short summary of its purpose.</gitb:description>
            <gitb:update updateMetadata="true" updateSpecification="false"/>
        </metadata>    
        ...
    </testsuite>

Relevant options to manage updates for existing test cases are possible through a similar ``update`` element of the :ref:`test case <test-case-metadata-update>` definitions.

.. index:: specification (Test suite)
.. index:: reference (Test suite reference)
.. index:: description (Test suite reference)
.. index:: link (Test suite reference)
.. _test-suite-metadata-specification:

specification
+++++++++++++

The ``specification`` element is an optional part of a test suite's metadata, that allows you to record in a structured manner a normative specification
reference for the test suite. Besides being present in the test suite definition, this information will also be rendered appropriately in the test suite's
on-screen display and in reports.

The structure of the ``specification`` element is as follows:

.. csv-table::
    :stub-columns: 1
    :header: "Name", "Required?", "Description"

    reference, no, The reference identifier or code.
    description, no, A text describing the referred specification.
    link, no, A link to allow navigation to the referred specification's online documentation.

All the above elements are optional, meaning that you can choose to provide any documentation you see fit for the specification. Depending on what is provided,
this information will be displayed accordingly, presenting for example the reference as a link if both are provided, or presenting only a link icon if only the
link is present.

The following example illustrates how this metadata could be used to identify the specification section relevant to the test suite and point to its online
documentation.

.. code-block:: xml
    :emphasize-lines: 6-10

    <testsuite id="TS1" xmlns="http://www.gitb.com/tdl/v1/" xmlns:gitb="http://www.gitb.com/core/v1/">
        <metadata>
            <gitb:name>TS1</gitb:name>
            <gitb:version>1.0</gitb:version>
            <gitb:description>A short description of the test suite to offer a short summary of its purpose.</gitb:description>
            <gitb:specification>
                <gitb:reference>Section-1.2.A</gitb:reference>
                <gitb:description>Security requirements</gitb:description>
                <gitb:link>https://my.spec.wiki.org</gitb:link>
            </gitb:specification> 
        </metadata>    
        ...
    </testsuite>

.. note::
    Similar specification reference information can also be added to individual :ref:`test cases<test-case-metadata-specification>`.

.. index:: actors (Test suite)
.. index:: id (Test suite actors)
.. index:: default (Test suite actors)
.. index:: hidden (Test suite actors)
.. index:: displayOrder (Test suite actors)
.. index:: name (Test suite actors)
.. index:: desc (Test suite actors)
.. index:: endpoint (Test suite actors)
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
    @default, no, Whether or not the actor is to be considered as the specification's default actor ("false" by default).
    @hidden, no, Whether or not the actor will be unavailable for use in conformance statements ("false" by default).
    @displayOrder, no, A number indicating the relative positioning that needs to be respected when displaying the actor in test execution diagrams.
    name, yes, A user-friendly name for the actor.
    desc, no, A description to provide additional information on the purpose of this actor in the specification.
    endpoint, no, Zero or more ``endpoint`` elements that capture an actor's configuration. 

The value for the ``id`` attribute is very important as it is used internally to link the test suite and its test cases to the relevant actor in the specification.
The ``name`` and ``desc`` elements are present as metadata when displaying a test case to a user but are not important with respect to test case steps and logic. 
The ``default`` attribute can be useful in cases where a specification defines multiple actors but only one is ever expected to be used as the SUT. Setting this to 
"true" indicates that this actor should be preselected when creating new conformance statements.

Another means of managing the actors that are to be used in
conformance statements is the ``hidden`` attribute which, if set to "true", will remove the actor from the ones available to create a conformance statement. This 
can useful when certain actors are not meant to be selected for testing (but there is no single default actor to otherwise set), or if an actor has been in use
but should now be deprecated. Setting an actor as ``hidden`` will not affect previous testing history but will make it unavailable for future conformance
statements.

The ``displayOrder`` attribute provides an indication on how the actor should be positioned in test execution diagrams relevant to other actors. This could be useful
to set if you want an actor to always appear first in diagrams regardless of the TDL steps that a test case defines (e.g. to always show the SUT first). When the 
``displayOrder`` attribute for an actor is smaller relevant to others, or if no ``displayOrder`` is specified for other actors, the actor will appear first. Note that
this ordering applies to actors defined in the specification, not special-purpose actor lifelines that could signify the test bed or the user. Finally, note that any 
setting that is made for an actor at test suite level is considered as a default and can be overridden at test case level (see :ref:`test-case-actors`).

Besides defining the roles of parties in a specification, actors can also have **configuration properties**. The purpose of these properties is to record
information about a system under test (SUT) that are specific to conformance statements for this actor. They are the most fine-grained level of
configuration you can use, complementing configuration at :ref:`domain <test-case-expressions-domain>`, :ref:`organisation <test-case-expressions-organisation>`
and :ref:`system <test-case-expressions-system>` levels. Actor-level configuration is most appropriate for information that, considering the same system,
differs depending on the selected specification actor. An example of this would be an endpoint address for a "receiver" actor to expect incoming messages,
that would not be needed for a "sender" actor.

Similar to :ref:`other types of configuration <test-case-configuration>`, any **required properties** for an actor corresponding to the system under test (SUT)
must be provided before executing tests. In addition, properties that are set to be **included in tests** will be added to the session's context and made available
for use in :ref:`expressions <test-case-expressions>` during test execution.

.. index:: endpoint (Test suite actor)
.. index:: name (Test suite actor endpoint)
.. index:: desc (Test suite actor endpoint)
.. index:: config (Test suite actor endpoint)

Actor configuration is captured in configuration sets named "endpoints" with each one defining key-value pair configuration properties named
"parameters". In terms of their XML representation, an endpoint is defined using the ``endpoint`` element as follows:

.. csv-table::
    :stub-columns: 1
    :header: "Name", "Required?", "Description"

    @name, yes, The name of the endpoint that must be unique for the actor.
    @desc, no, A description to explain the purpose of this endpoint.
    config, yes, One or more elements to define each of the endpoint's parameters. 

.. index:: config (Test suite actor endpoint)
.. index:: label (Test suite actor endpoint parameter)
.. index:: desc (Test suite actor endpoint parameter)
.. index:: use (Test suite actor endpoint parameter)
.. index:: kind (Test suite actor endpoint parameter)
.. index:: adminOnly (Test suite actor endpoint parameter)
.. index:: notForTests (Test suite actor endpoint parameter)
.. index:: hidden (Test suite actor endpoint parameter)
.. index:: dependsOn (Test suite actor endpoint parameter)
.. index:: dependsOnValue (Test suite actor endpoint parameter)
.. index:: allowedValues (Test suite actor endpoint parameter)
.. index:: allowedValueLabels (Test suite actor endpoint parameter)
.. index:: defaultValue (Test suite actor endpoint parameter)

The ``config`` elements defining an endpoint's parameters are structured as follows:

.. csv-table::
    :stub-columns: 1
    :header: "Name", "Required?", "Description"
    :delim: |

    @name| yes| The name of the parameter that must be unique for the endpoint.
    @label| no| A user friendly name to display for the parameter. If not set this will be set to the value of the ``name`` attribute.
    @desc| no| A description to explain the purpose of this parameter.
    @use| no| Whether this is a required (value "R" - the default) or optional ("O") parameter. 
    @kind| no| Whether this is a simple text (value "SIMPLE" - the default), file (value "BINARY") or a secret value (value "SECRET").
    @adminOnly| no| A boolean value (by default "false") indicating whether this parameter can only be edited by administrators.
    @notForTests| no| A boolean value (by default "false") indicating whether this parameter is included as a test session context variable.
    @hidden| no| A boolean value (by default "false") indicating whether this parameter can only be viewed by administrators.
    @dependsOn| no| A string indicating the name of another parameter within the endpoint that is a prerequisite for the current one.
    @dependsOnValue| no| In case ``dependsOn`` is defined, this is the value that the prerequisite property should have in order for the current one to be enabled.
    @allowedValues| no| A comma-separated list of values that are allowed for this parameter.
    @allowedValueLabels| no| In case ``allowedValues`` is defined, this is a comma-separated list of labels for the provided values (their number must match the values). If not provided, the values themselves are used as labels.
    @defaultValue| no| An optional default value to set for new instances of this parameter (ignored if parameter is not of "SIMPLE" kind).

In terms of the ``kind`` attribute, the values "SIMPLE" and "SECRET" both represent text values. The difference is that ones 
defined as "SECRET" are never presented to users nor are they ever transferred to client software. The two attributes ``adminOnly``
and ``notForTests`` are used to increase the options available in defining a flexible and complete conformance testing strategy.

Setting the ``adminOnly`` flag to "true" would typically be used for required parameters (i.e. ``use`` set to "R") as 
doing so allows community or test bed administrators to verify and adapt a system's setup before allowing it to
execute tests. This could either be done as a simple eligibility check or to actually provide required information
that is needed for the tests (e.g. a generated identifier or certificate). If a required parameter that is set as 
``adminOnly`` is not configured, the system in question is prevented from executing tests. Furthermore, such a parameter
could be also set as ``hidden`` meaning that it is only visible to administrators.

The ``notForTests`` flag on the other hand would be set to "true" for information that is strictly administrative
in nature and is not actually needed during test sessions. An example of this would be to record a flag set by administrators that, if
missing, would prevent test sessions to be started (i.e. combined with the ``adminOnly`` flag as explained above).
Otherwise such attributes could also be used to enable general data collection for testing organisations pertinent
to specific conformance statements. 

The ``dependsOn`` attribute can be used to define prerequisites between parameters. Parameters whose prerequisites are not met are
considered as disabled. Consider for example a parameter named "size" that can set with values "large" or "small". In case "large"
is selected, a second parameter named "capacity" becomes applicable. These two concepts (providing a set of expected values and
defining prerequisites) can be achieved as follows:

.. code-block:: xml

    <gitb:actor id="system">
        <gitb:name>System</gitb:name>
        <gitb:endpoint name="systemInfo">
            <!-- Define the allowed values and provide user-friendly labels. -->
            <gitb:config name="size" label="Size" kind="SIMPLE" use="R" allowedValues="l,s" allowedValueLabels="Large,Small"/>
            <!-- Enable this parameter if "size" is set as "l". -->
            <gitb:config name="capacity" label="Capacity" kind="SIMPLE" use="R" dependsOn="size" dependsOnValue="l"/>
        </gitb:endpoint>
    </gitb:actor>

Note that a parameter whose prerequisite condition is not met is considered as inactive, even if set as required (i.e. ``use="R"``).
Such inactive parameters are not requested as input and are not included in test sessions as variables. Prerequisites can
also be chained, meaning that parameter A can depend on B that can depend on C. Even if a parameter's direct prerequisite is met,
it will still be considered as inactive if any prerequisites further up in its hierarchy are not.

Endpoints and their parameters are used in two main scenarios:

* As simple sets of configuration values.
* As placeholders for SUT actors to hold simulated actor configuration (provided via their messaging handlers).

These two scenarios are explained in the following sections.

.. index:: Endpoints (simple configuration values)

Endpoints for simple configuration values
+++++++++++++++++++++++++++++++++++++++++

.. note::

    You can also define configuration at :ref:`domain <test-case-expressions-domain>`, :ref:`organisation <test-case-expressions-organisation>`
    and :ref:`system <test-case-expressions-system>` levels. Actor configuration is best suited for information that differs across actors (conformance statements).

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

Endpoints for dynamic configuration values
++++++++++++++++++++++++++++++++++++++++++

The endpoint concept becomes more meaningful when certain configuration values need to be generated at runtime, and specifically before the test session begins.
An example of this is generating a unique endpoint address for a simulated "receiver" actor, that will be used by the "sender" actor as the SUT.
This takes place as part of the setup of :ref:`messaging exchanges <tdl-messaging-steps>` when using a :ref:`custom messaging handler <handlers-custom-handlers>`.

Before a test session starts, custom messaging services receive an **initiate** call for the simulated actors they cover. This call includes the actors'
predefined configuration values, as well as those configured for the SUT. The messaging handler now has the opportunity to return a set of values per
simulated actor that can differ based on the received inputs. These different sets of configuration are mapped on the basis of their ``endpoint``.

To better illustrate this, consider a sender and receiver example, defined in a test suite as follows:

.. code-block:: xml

    <gitb:actor id="sender">
        <gitb:name>sender</gitb:name>
        <gitb:endpoint name="expectedConfig"/>
    </gitb:actor>
    <gitb:actor id="receiver">
        <gitb:name>sender</gitb:name>
    </gitb:actor>

These actors can then be used in a test case by defining "sender" as the SUT and "receiver" as simulated.

.. code-block:: xml

    <actors>
        <gitb:actor id="sender" name="sender" role="SUT"/>
        <gitb:actor id="receiver" name="receiver" role="SIMULATED"/>
    </actors>
    <steps>
        <btxn from="sender" to="receiver" txnId="t1" handler="$DOMAIN{messagingServiceAddress}"/>
        <send desc="Call receiver" from="sender" to="receiver" txnId="t1">
            <input name="anInputValue">$sender{receiver}{configuredValue}</input>
        </send>
        <etxn txnId="t1"/>
    </steps>

The key point to note is the reference to the "configuredValue" parameter in ``$sender{receiver}{configuredValue}``. To be able to do this the following has happened:

1. The sender SUT actor defined an endpoint named "expectedConfig".
2. The receiver simulated actor, due to its presence in a messaging transaction (see the ``btxn`` element), was included in the ``initiate`` call to
   the messaging service listening at ``$DOMAIN{messagingServiceAddress}``.
3. The service responded by providing a configuration for endpoint "expectedConfig" with a parameter named "configuredValue".
4. The returned configuration was matched to the sender's "expectedConfig" endpoint and copied under the sender actor for this endpoint.
5. The returned parameter can now be referenced as ``$sender{receiver}{configuredValue}`` (i.e. ``$SUT_ACTOR_ID{SIMULATED_ACTOR_ID}{PARAMETER_NAME}``).

.. note::
    **GITB software support:** Only a single endpoint can currently be configured for an actor. Additional endpoints will be recorded for the actor but will be 
    ignored during test execution.

.. index:: testcase (Test suite)
.. index:: id (Test suite testcase)
.. index:: prequisite (Test suite testcase)
.. index:: option (Test suite testcase)
.. index:: supportsParallelExecution (test case order configuration in test suite)

.. _test-suite-test-cases:

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

The order with which the test case entries are defined is important as it defines their **execution order**. This ordering would apply when multiple test
cases are selected for execution at once, which typically occurs when launching a complete test suite as opposed to an individual test case. Executing a
complete test suite can be done in two ways:

    * **Interactively**, by presenting to the tester all test cases and launching each one in sequence while interacting with the tester and updating
      the test session diagram and log.
    * **In the background**, by launching all test cases as a batch, allowing also the tester to choose whether tests are executed in parallel or in 
      sequence. Note that test cases that do not :ref:`support parallel execution<test-case>` can be configured as such so that they are always executed
      in isolation (for a given SUT).

.. note::
    **GITB software support:** The ``prequisite`` and ``option`` values are currently ignored.

.. _test-suite-deploying:

Deploying a test suite in the GITB software
-------------------------------------------

.. note::

    **Deploying test suites during development:** When deploying test suites to a development test bed instance, the fastest way
    is to use the test bed's `REST API <https://www.itb.ec.europa.eu/docs/itb-ta/latest/api/index.html#deploy>`_. You can find
    instructions `here <https://www.itb.ec.europa.eu/docs/guides/latest/developingComplexTests/index.html#step-3-prepare-your-workspace>`_
    on enabling the API and creating a script to automate its use.

A test suite is packaged as a compressed ZIP archive that contains:

* A single test suite XML file.
* One or more test case XML files (unless this is only a :ref:`shared resource holder<test-suite-sharing-empty>`).
* Any number of arbitrary files used as resources within test cases.

The names of the archive, the test suite and the test case XML files are not important. Neither is the folder structure defined within the archive.
What is important is that:

* A single test suite XML file is defined.
* Test case IDs defined in the test case XML files are referenced in the test suite XML.

Uploading a test suite to the GITB software has the following results:

* If it doesn't previously exist, the test suite is recorded along with its test cases and linked to the appropriate specification actors.
* If the test suite does exist, the user selects whether this new version should invalidate previous conformance testing sessions (if the
  change is significant) or not. Choosing to replace the test suite results in the test suite being updated and its test cases being replaced
  with the ones contained in the new version. Matching of the test suite with an existing one is on the basis of their ID within the specification.
* The actors that are defined in the test suite are created if they don't already exist along with their endpoints and endpoint parameters.
* If the user chooses to, actors that already exist in the specification are updated based on the latest provided information. In this case new endpoints and parameters are added
  and existing ones are updated. Note that actors, endpoints and parameters that are not defined in the new test suite are not removed. The matching of
  actors is on the basis of their ID, whereas for endpoints and parameters their name is used.
* If the test suite does not include any test cases it is marked as hidden (See :ref:`test-suite-sharing-empty`).

As previously discussed, the :ref:`test-suite-actors` section serves to define which actors are used within the test suite and to provide their details (their name, endpoints
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

Specifying a test suite's actors in this way could be interesting if you want to manage their information fully through the GITB software's
user interface. The only thing to ensure is that the specification's actors are already defined before uploading test suites and that their 
IDs are correctly referenced.

.. _test-suite-sharing:

Sharing resources across test suites
------------------------------------

Test suites, and in particular the :ref:`test cases<test-case>` they include, may rely on additional file resources for
their documentation and test content. Such files may be made available to test suites in one of two ways:

* As binary :ref:`external configuration properties<test-case-configuration>`
* As files included in test suite archives.

When provided through configuration properties, files will be available in the test sessions' context from which they
can be used as ``binary`` variables. Although this approach is flexible, it does not cover all cases, cannot support
large files, and is not well adapted for resources with dependencies (e.g. an XML schema with imports). The approach of
providing files within test suite archives is an effective way of addressing such limitations.

The files included within test suites, apart from the test suite and test case definition files, can be:

* :ref:`Scriptlets<scriptlets>`, XML files containing blocks of test steps to be used across multiple test cases.
* Arbitrary files used as imports in :ref:`test cases<test-case-imports>` and :ref:`scriptlets<scriptlets_elements_imports>`.
* Documentation files, containing HTML content that is used to provide extended documentation for :ref:`test suites<test-suite-metadata>`,
  :ref:`test cases<test-case-metadata>`, and individual :ref:`test steps<tdl-steps-common-documentation>`.

Once such files are included in a test suite they can be used within itself but also shared with other test suites. The
approach to share files uses the test suites' ``id`` attribute to identify the test suite from which a resource will be
loaded. The constructs that support this all foresee a ``from`` attribute that is set with the target test suite ``id``,
defaulting to the current test suite if missing. Specifically:

* The ``call`` step to :ref:`call a scriptlet<tdl-step-call>`.
* The ``artifact`` element of a test case or scriptlet's :ref:`imports<test-case-imports>` block.
* The ``documentation`` element for :ref:`test suites<test-suite-metadata>`, :ref:`test cases<test-case-metadata>` and
  :ref:`test steps<tdl-steps-common-documentation>`.

The approach to lookup a test suite using the identifier specified in the ``from`` attribute is as follows:

#. Look for the test suite in the same **specification**.
#. If not found, look for the test suite in the other specifications of the **domain**.
   If across specifications multiple matching test suites are found, one of them will be arbitrarily picked. To avoid such
   a scenario it is obvious that you should ensure test suites used to load shared resources can be uniquely identified.

Once the target test suite has been located, the specific file is loaded using a file path relative to the test suite's
root. The approach to provide this path depends on the test construct in question:

* For ``call`` steps this is the value of the ``path`` attribute.
* For ``artifact`` import elements this is the element's value.
* For ``documentation`` elements this is the value of the ``import`` attribute.

.. note::
    **Resources with dependencies:** When resources such as XML schemas and Schematrons are imported from other test suites,
    resolution of dependent files (e.g. imported schemas) is carried out as expected based on the loaded file's location.

.. _test-suite-sharing-empty:

Test suites as shared resource holders
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

It is also possible to define a test suite as a pure resource holder for commonly used files (scriptlets, imports and
documentation). Such test suites can omit the definition of :ref:`actors<test-suite-actors>` and
:ref:`test cases<test-suite-test-cases>`, including only a :ref:`metadata<test-suite-metadata>` block to provide, at least,
the test suite's name and version.

Test suites that do not include test cases are considered as **hidden** in the GITB software and are not visible to testers.
They are visible to administrators for their management and can only be used by other test suites to load common resources.