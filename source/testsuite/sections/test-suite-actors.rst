In the ``actors`` element we identify the actors that will be involved in the test suite's test cases. These actors may either be ones that will be tested 
as systems under test (i.e. the focus of test cases) or be simulated by the Test Bed. At least one actor needs to be defined here. The ``actors`` element 
contains one or more ``actor`` elements with structure as follows:

.. csv-table::
    :stub-columns: 1
    :header: "Name", "Required?", "Description"

    @id, yes, The unique identifier for the actor.
    @default, no, Whether or not the actor is to be considered as the specification's default actor ("false" by default).
    @displayOrder, no, A number indicating the relative positioning that needs to be respected when displaying the actor in test execution diagrams.
    @hidden, no, Whether or not the actor will be unavailable for use in conformance statements ("false" by default).
    desc, no, A description to provide additional information on the purpose of this actor in the specification.
    endpoint, no, Zero or more ``endpoint`` elements that capture an actor's configuration. 
    name, yes, A user-friendly name for the actor.

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
this ordering applies to actors defined in the specification, not special-purpose actor lifelines that could signify the Test Bed or the user. Finally, note that any 
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

    @desc, no, A description to explain the purpose of this endpoint.
    @name, yes, The name of the endpoint that must be unique for the actor.
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

    @adminOnly| no| A boolean value (by default "false") indicating whether this parameter can only be edited by administrators.
    @allowedValues| no| A comma-separated list of values that are allowed for this parameter.
    @allowedValueLabels| no| In case ``allowedValues`` is defined, this is a comma-separated list of labels for the provided values (their number must match the values). If not provided, the values themselves are used as labels.
    @defaultValue| no| An optional default value to set for new instances of this parameter (ignored if parameter is not of "SIMPLE" kind).
    @dependsOn| no| A string indicating the name of another parameter within the endpoint that is a prerequisite for the current one.
    @dependsOnValue| no| In case ``dependsOn`` is defined, this is the value that the prerequisite property should have in order for the current one to be enabled.
    @desc| no| A description to explain the purpose of this parameter.
    @hidden| no| A boolean value (by default "false") indicating whether this parameter can only be viewed by administrators.
    @kind| no| Whether this is a simple text (value "SIMPLE" - the default), file (value "BINARY") or a secret value (value "SECRET").
    @label| no| A user friendly name to display for the parameter. If not set this will be set to the value of the ``name`` attribute.
    @name| yes| The name of the parameter that must be unique for the endpoint.
    @notForTests| no| A boolean value (by default "false") indicating whether this parameter is included as a test session context variable.
    @use| no| Whether this is a required (value "R" - the default) or optional ("O") parameter. 

In terms of the ``kind`` attribute, the values "SIMPLE" and "SECRET" both represent text values. The difference is that ones 
defined as "SECRET" are never presented to users nor are they ever transferred to client software. The two attributes ``adminOnly``
and ``notForTests`` are used to increase the options available in defining a flexible and complete conformance testing strategy.

Setting the ``adminOnly`` flag to "true" would typically be used for required parameters (i.e. ``use`` set to "R") as 
doing so allows community or Test Bed administrators to verify and adapt a system's setup before allowing it to
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
