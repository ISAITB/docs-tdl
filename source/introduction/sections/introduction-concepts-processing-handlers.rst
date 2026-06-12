**Processing handlers** are extensions to the Test Bed's capabilities to undertake actions that it does not natively support. The 
purpose of a processing handler is to receive a set of input parameters that it can process and produce a 
set of output values. These output values are stored in the test session context for subsequent use. 

Processing handlers implement the `GITB processing service API`_ that defines the operations needed by the Test Bed to request
processing for specific input and receive the results. Implementations can be either embedded components or external web service endpoints.

.. _GITB processing service API: https://www.itb.ec.europa.eu/specs/latest/gitb_ps.wsdl

The lifecycle of a test session
-------------------------------

The following diagram illustrates the steps involved in creating and using a test case. These steps occur within a Test Bed as part of overall
test management before and after the test's execution and also during the test session itself.

.. figure:: images/lifecycle.png
  :align: center
  :figclass: no-bottom-margin

  The test session lifecycle

Step 1: Create
~~~~~~~~~~~~~~

Before we can start testing we need to create a test suite and at least one contained test case. The test suite will define actors matching the
specification's requirements and the test cases will each define one actor from the ones configured as the SUT. Other actors will be defined 
as being simulated.

Step 2: Configure
~~~~~~~~~~~~~~~~~

The actor that is identified as the SUT may have one or more optional or required configuration parameters that need to be provided. These are 
verified before test session execution to ensure that all required parameters are defined. Configuration parameters are provided by administrators
of the organisation that is engaged in testing.

Step 3: Initiate
~~~~~~~~~~~~~~~~

During this step a new test session is created and the messaging handlers of any simulated actors present are notified to prepare. This is done 
by calling the simulated actors' messaging handlers passing the session information and also the configuration parameters of the SUT. Apart from 
preparing the messaging handlers for the upcoming session this notification may also result in specific configuration provided from the messaging 
handlers to the SUT (e.g. a dynamically created address to send messages to).

Step 4: Run preliminary
~~~~~~~~~~~~~~~~~~~~~~~

With all actors configured, this is the phase where notifications may be presented to the user to prepare for the upcoming test.
These notifications can either be instructions to follow or requests for data to be provided before the test session starts.

Step 5: Run test steps
~~~~~~~~~~~~~~~~~~~~~~

During this phase the test session proceeds to execute the steps defined in the test case's description. Any supported set of steps can take place
resulting in a test session context that is populated with variables and referenced according to the implemented test logic. The test steps may involve automatic 
processing and validation, interaction with messaging handlers to send and receive messages, user input requests, and any control logic constructs
that are needed to correctly express the expected flow. The test session eventually completes, resulting in one of three outcomes:

* **Success:** If all test steps are completed successfully.
* **Failure:** If one of the test steps failed.
* **Undefined:** If the session was terminated before completion.

Step 6: Finalise
~~~~~~~~~~~~~~~~

Upon finalisation the Test Bed cleans up the state relevant to the test session and also notifies external handler implementations to consider the 
session as closed.

Step 7: Consult
~~~~~~~~~~~~~~~

The results of the test session can be consulted either immediately following completion or by looking up the results in the organisation's test session
history. A test session records its output per step that can be used to produce detailed test step reports or an overview report for the complete
test session.

.. _introduction_spec_links:

Specification links
-------------------

The following table provides the links to access the latest version of the GITB specifications. These include XSDs defining the GITB
TDL constructs but also related specifications such as the WSDLs for messaging, processing and validation services.

.. csv-table::
  :header: "Specification", "Description", "Link"

  GITB core elements, The XSD defining core elements used in other GITB XSDs, https://github.com/ISAITB/gitb-types/blob/master/gitb-types-specs/src/main/resources/schema/gitb_core.xsd
  GITB test description language (TDL), The XSD defining the types used when defining TDL test suites and test cases, https://github.com/ISAITB/gitb-types/blob/master/gitb-types-specs/src/main/resources/schema/gitb_tdl.xsd
  GITB test presentation language (TPL), The XSD defining types used to report on the status of a test session, https://github.com/ISAITB/gitb-types/blob/master/gitb-types-specs/src/main/resources/schema/gitb_tpl.xsd
  GITB test reporting language (TRL), The XSD defining the types used to report step output, https://github.com/ISAITB/gitb-types/blob/master/gitb-types-specs/src/main/resources/schema/gitb_tr.xsd
  GITB messaging service API, The WSDL defining messaging service operations, https://github.com/ISAITB/gitb-types/blob/master/gitb-types-specs/src/main/resources/wsdl/gitb_ms.wsdl
  GITB messaging service types, The XSD defining the messaging service message types, https://github.com/ISAITB/gitb-types/blob/master/gitb-types-specs/src/main/resources/schema/gitb_ms.xsd
  GITB validation service API, The WSDL defining validation service operations, https://github.com/ISAITB/gitb-types/blob/master/gitb-types-specs/src/main/resources/wsdl/gitb_vs.wsdl
  GITB validation service types, The XSD defining the validation service message types, https://github.com/ISAITB/gitb-types/blob/master/gitb-types-specs/src/main/resources/schema/gitb_vs.xsd
  GITB processing service API, The WSDL defining processing service operations, https://github.com/ISAITB/gitb-types/blob/master/gitb-types-specs/src/main/resources/wsdl/gitb_ps.wsdl
  GITB processing service types, The XSD defining the processing service message types, https://github.com/ISAITB/gitb-types/blob/master/gitb-types-specs/src/main/resources/schema/gitb_ps.xsd
  GITB Test Bed service API, The WSDL defining Test Bed service operations, https://github.com/ISAITB/gitb-types/blob/master/gitb-types-specs/src/main/resources/wsdl/gitb_tbs.wsdl
  GITB Test Bed service types, The XSD defining the Test Bed service message types, https://github.com/ISAITB/gitb-types/blob/master/gitb-types-specs/src/main/resources/schema/gitb_tbs.xsd

The final GITB workgroup report can be downloaded here [:download:`CEN_WS_GITB3_CWA_Final.pdf`].
