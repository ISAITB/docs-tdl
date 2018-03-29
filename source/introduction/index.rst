Introduction
============

The purpose of the current documentation is to make it possible for you to understand and create test suites
using the GITB TDL. The approach followed herein is to document the available possibilities providing easy to 
follow examples but also to note where appropriate certain considerations on the use of TDL in the GITB test bed
software.

What is the GITB TDL?
---------------------

The `GITB project`_ represents a CEN standardisation initiative funded by the European Commission’s DG GROW 
to provide the specifications for a generic interoperability test bed and their implementation in the form 
of the GITB test bed software. The focus of these specifications and software is not any kind of testing 
(e.g. performance, regression, penetration) but rather conformance and interoperability testing. Simply
put, *conformance testing* verifies that the requirements of a given specification are met, whereas
*interoperability testing* comes as a second step to verify that two or more parties can interact as expected.
Furthermore the focus here is on technical specifications related to e.g. messaging protocols or content,
whereas the parties previously mentioned would typically be software components.

A key element of the GITB specifications is the **GITB Test Description Language** (GITB TDL) that is used to 
express how individual tests are defined in terms of involved actors and foreseen steps. The goal for GITB TDL
is to provide test descriptions that are executable, i.e. ready to be processed by a testing engine capable of
understanding GITB TDL, but also human readable. To achieve this, GITB TDL is based on XML, defining specific 
elements to match its foreseen testing constructs. 

.. _GITB project: http://www.cen.eu/work/areas/ict/ebusiness/pages/ws-gitb.aspx

Where is it used?
~~~~~~~~~~~~~~~~~

The short answer is that GITB TDL is used in the GITB test bed software to define and run tests.

To be more complete however, GITB TDL is not tied to the GITB software per se but can be processed by any test bed
implementation that conforms to the GITB specifications. In fact, the GITB specification foresees the concepts
of TDL compliance and splits this in *GITB compliant TDL processors* and *GITB TDL compliant producers*. Simply
put, a TDL processor is any software that can read GITB TDL content and understand it, typically being a
test bed implementation that can read and execute GITB TDL test cases when requested (without necessarily conforming
to other elements of the overall GITB specification). On the other hand, a TDL producer is software that can 
output TDL content, in which case we would typically be referring to test case editors or converter tools from
other test case representations. Finally, consider that compliance to GITB TDL is not necessarily an "all-or-nothing"
statement; compliance can be partial by supporting only a limited set of constructs. To support this last point, GITB
TDL concepts and constructs are formalised using a `published taxonomy`_ that can be referenced to understand the TDL 
constructs that are used (within a test case) or supported (by a test bed).

One of the goals of GITB TDL is also to help reuse existing work by ideally sharing test cases between test bed 
implementations. To this effect the `GITB Test Registry and Repository (TRR)`_ aims to act as a reuse portal for 
GITB TDL test cases by facilitating their discovery and listing expected and used constructs.

.. _published taxonomy: http://data.europa.eu/itw
.. _GITB Test Registry and Repository (TRR): https://joinup.ec.europa.eu/collection/gitb-trr

How is it maintained?
~~~~~~~~~~~~~~~~~~~~~

Following the initial work from the CEN GITB workgroup, the maintenance and evolution of GITB TDL has been taken over 
by the European Commission, specifically the `Interoperability Test Bed Action`_ of DIGIT ISA. The Action foresees the 
maintenance of the GITB software and specification based on the needs of the testing community and proceeds to regular
releases. Regarding GITB TDL in particular, introduced changes are always done in a backwards-compatible manner, adding 
capabilities rather than changing existing ones. Releases of GITB TDL and its version numbers currently follow the GITB
software versioning.

.. _Interoperability Test Bed Action: https://joinup.ec.europa.eu/solution/interoperability-test-bed/about

Core concepts
-------------

Before diving into GITB TDL it is important to be aware of certain core concepts.

Test case
~~~~~~~~~

A **test case** represents a set of steps and assertions that form a cohesive scenario for testing purposes. 
These are captured as as a single XML file authored using GITB TDL constructs.

Test suite
~~~~~~~~~~

A **test suite** is used to group together related test cases into a cohesive set. A test suite can include
metadata such a name and description that provide useful context to understand the purpose of its contained
tests. A test suite is expressed as an XML file and is bundled in a ZIP archive along with its contained 
test cases and the resources used in those.

Test session
~~~~~~~~~~~~

A **test session** represents a single execution of a test case.

Test session context
~~~~~~~~~~~~~~~~~~~~

The **test session context** can be considered a persistent store specific to a given test session that is used to record
values. These values can either be configuration parameters provided before the test session starts, configuration parameters
generated during the test session's initiation, or values that are added dynamically during the session as a result of the 
ongoing test steps. The test session context is very important in that it gives a level of control above individual test 
steps that allow the testing of complete conversations. In addition, its ability to store and lookup arbitrary content dynamically
makes it possible to implement complex flows and control logic.

The test session context can be viewed as a map or keys to values, where each key is a string identifier and the value can be 
any type supported by GITB, including additional nested maps.

Actor
~~~~~

An **actor** from the perspective of GITB represents a party involved in the overall process that the test cases
aim to test. What exactly is an actor depends fully on the tests or the specification they relate to. Here are
some examples:

* A specification that foresees sending a message from one party to another could define a "Sender" and "Receiver" 
  actor to represent the involved parties.
* A solution that foresees a central EU portal exchanging information with national endpoints could define a "Portal" 
  and "National endpoint".
* A content specification (e.g. XML-based) for which we simply want to allow users to upload samples for validation 
  could define a "User" actor to represent this role in test cases.

Once actors are defined they play an important role in the testing process as follows:

* They define properties relevant to the testing (think of them as actor-specific configuration elements).
* Each test case foresees that actors are either simulated by the test bed or are the focus of the test. In the latter case
  they are refered to as having a role of **SUT** (System Under Test).

.. _introduction-concepts-messaging-handlers:

Messaging handlers
~~~~~~~~~~~~~~~~~~

**Messaging handlers** are embedded or external components whose purpose is to allow 
messaging for simulated actors, defining how these receive and send messages. How this actually takes place can be
completely arbitrary and is not tied to a specific protocol. What is important is that the test bed can signal the handler that
it needs to send data or similarly signal that is is waiting to receive data. In this latter case, the handler notifies the 
test bed by means of a callback with an appropriate payload.

Given that the actual meaning and implementation of sending and receiving is decoupled from the test bed, it can be extended 
to handle any needed protocol, requiring only that an appropriate handler exists. If for example the test bed needs to send 
and receive email, it does not need to support e.g. SMTP natively, it just needs access to a handler that implements the GITB
messaging service API, that can send and receive emails on its behalf. When receiving an email this handler will use the 
callback approach to signal to the test bed that an email was received, passing back the received content as appropriate. Given this 
flexibility, it is important to note also that messaging handlers are not only used when an actor is fully simulated
but could also act as a adapter layer on top of existing software.

Messaging handlers receive in each case (when sending or receiving) input parameters and return a set of output parameters. These output
parameters are stored in the test session context and can be subsequently used in further test steps.

.. _introduction-concepts-validation-handlers:

Validation handlers
~~~~~~~~~~~~~~~~~~~

**Validation handlers** are similar in concept to messaging handlers but much simpler. Their purpose is limited to validating content and 
returning a report with the result. Validation is a fully decoupled process in that the content being validated as well as any other parameters 
are provided by the test bed as input without needing to be aware of how validation actually takes place.

The result of a validation handler is recorded in the test session context for subsequent use in the test case.

.. _introduction-concepts-processing-handlers:

Processing handlers
~~~~~~~~~~~~~~~~~~~

**Processing handlers** are extensions to the test bed's capabilities to undertake actions that it does not natively support. The 
purpose of a processing handler is to receive a set of input parameters that it can process and produce as a result an arbitrary
set of output values. These output values are stored in the test session context and can be subsequently used. 

The features exposed by processing services are captured as operations that take place within a processing transaction. As an example
consider a processing service to access ZIP archives. When a ZIP archive needs to be processed, a transaction can be initiated by the
test bed where as a first operation, the ZIP archive to handle is passed. Subsequent operations could check for the existence of entries,
get a content list, and extract a complete entry. Closing the transaction would allow the processing service to cleanup any open resources.

The lifecycle of a test session
-------------------------------

[TODO IMAGE]

Step 1: Test suite creation
~~~~~~~~~~~~~~~~~~~~~~~~~~~

Before we can start testing we need to create a test suite and at least one contained test case. The test suite will define actors matching the
specification's requirements and the test cases will each define one actor from the ones configured as the SUT. Other actors will be defined 
as simulated.

Step 2: Configuration
~~~~~~~~~~~~~~~~~~~~~

The actor that is identified as the SUT may have one or more optional or required configuration parameters that need to be provided. These are 
verified before test session execution to ensure that all required parameters are defined.

Step 3: Initiation
~~~~~~~~~~~~~~~~~~

During this step a new test session is created and any simulated actors present are notified to prepare. This is done by calling the simulated
actors' messaging handlers passing the session information and also the configuration parameters of the SUT. Apart from preparing the messaging
handlers for the upcoming session this notification may also result in specific configuration to provide to the SUT.

Step 4: Preliminary steps
~~~~~~~~~~~~~~~~~~~~~~~~~

With all actors configured, this is the phase where notifications may be presented to the user as instructions on any specific steps to follow.

Step 5: Test execution
~~~~~~~~~~~~~~~~~~~~~~

During this phase the test session proceeds to execute the steps defined in the test case's description. Any supported set of steps can take place
resulting in a test session context that is populated and referenced according to the implemented test logic. The test steps may involve automatic 
processing and validation, interaction with messaging handlers to send and receive messages, user input requests, and any control logic constructs
that are needed to correctly express the expected flow.

Step 6: Finalisation
~~~~~~~~~~~~~~~~~~~~

Upon finalisation the test bed cleans up the state relevant to the test session and also notifies external handler implementations to consider the 
session as closed.