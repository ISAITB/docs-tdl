Examples
========

The following section provides downloadable complete examples of GITB TDL test suites for simple and complex cases. The sections below 
provide the download links, a brief description of the test suite's purpose and context and an overview of the TDL constructs used.

Example 1: Invoice validation via user upload
---------------------------------------------

**Context:** Upload an UBL electronic invoice for subsequent validations using the UBL XSD and Schematron files.

This example illustrates use of :ref:`tdl-step-interact` to request the file to validate. It then proceeds to use :ref:`tdl-step-verify`
to validate the uploaded file against the UBL XSD and two Schematron files that are bundled as artefacts in the test suite. Validations
use the :ref:`handlers-XSDValidator` and :ref:`handlers-SchematronValidator`.

**Download:** :download:`example1.zip`

Example 2: Invoice validation via direct AS2 messaging
------------------------------------------------------

**Context:** Validation of a UBL or CII invoice provided to the test bed via AS2 messaging using direct endpoint-to-endpoint communication.

This example illustrates use of the :ref:`tdl-step-receive` and :ref:`tdl-step-send` messaging steps to receive an AS2 message and provide
an appropriate receipt response. Messaging is handled using a custom embedded messaging handler. The messaging handler extracts the invoice
from the message payload and using the :ref:`tdl-step-assign` step determines the syntax to validate for (UBL or CII). The :ref:`tdl-step-verify`
step is then used to validate in a single step the invoice against the appropriate set of rules given the invoice's type.

**Download:** :download:`example2.zip`

Example 3: Invoice validation via direct AS4 messaging
------------------------------------------------------

**Context:** Validation of a UBL or CII invoice provided to the test bed via AS4 messaging using direct endpoint-to-endpoint communication.

This example illustrates use of the :ref:`tdl-step-receive` and :ref:`tdl-step-send` messaging steps to receive an AS4 message that is sent
to the test bed's AS4 access point. The lookup of the relevant AS4 message is done after prompting the user for its ID using an :ref:`tdl-step-interact`
along with the target syntax to validate for. The message ID is then provided to the messaging transaction's remote validation handler that
does the actual lookup and returns the message. Once the invoice is extracted from the messaging handler it is validated in one go using the 
:ref:`tdl-step-verify` step with an external validator.

**Download:** :download:`example3.zip`

Example 4: Invoice validation via AS2 with automatic discovery
--------------------------------------------------------------

**Context:** Validation of a UBL or CII invoice provided to the test bed via AS2 messaging using automatic participant discovery.

This example illustrates configuration of a simulated actor's properties for a SUT and their subsequent use through an :ref:`tdl-step-assign`
step to construct the receiver address. These values are used with a template response artefact to provide responses specific to each test session.
The :ref:`tdl-step-receive` and :ref:`tdl-step-send` messaging steps are used conditionally using an :ref:`tdl-step-if` that checks a configuration 
property provided by the user before the test's execution. Once the message payload is extracted an :ref:`tdl-step-assign` step is used to 
conditionally determine the syntax to validate for, with the validation taking place via external validator using a :ref:`tdl-step-verify` step.

**Download:** :download:`example4.zip`

Example 5: ESPD content validation via user upload
--------------------------------------------------

**Context:** Validation of an ESPD request or response document via user upload.

This is a simple example illustrating upload of the content to validate by the user along with the target syntax using a :ref:`tdl-step-interact`
step. Validation of the uploaded content takes place using a :ref:`tdl-step-verify` step with an external validator.

**Download:** :download:`example5.zip`

Example 6: WMS client request validation
----------------------------------------

**Context:** Validation of the requests of a WMS client to a WMS server to request a map. A client of the WMS server is expected to first query 
its capabilities and, based on the returned values, request a supported map layer. In this test case the map box expected to be requested 
should match the maximum supported coordinates.

This example uses the :ref:`tdl-step-receive` and :ref:`tdl-step-send` messaging steps with the :ref:`handlers-HttpProxyMessaging` handler that 
is used as a proxy to monitor the exchange. A series of :ref:`tdl-step-assign` steps are used to extract information from the request that
are subsequently used to verify the correctness of the complete conversation using multiple :ref:`tdl-step-verify` steps with the 
:ref:`handlers-XPathValidator`.

**Download:** :download:`example6.zip`

Example 7: ASiC container validation for eTendering processes
-------------------------------------------------------------

**Context:** Validation of an ASiC container that is uploaded for validation to ensure that it is valid with respect to eTendering requirements.

This is a complex example that begins with the upload of the container to validate using an :ref:`tdl-step-interact` step. Initial validation of
the archive as an ASiC container takes place using a :ref:`tdl-step-verify` step with a remote validation service. To extract the content of the
archive for further validation, the :ref:`tdl-step-process` step is used with an external ZIP archive processing service to perform multiple operations
on the archive. Validation of the contents occurs in a stepped approach using multiple :ref:`tdl-step-assign` steps and progressive :ref:`tdl-step-verify` 
steps using remote validators and the embedded :ref:`handlers-XPathValidator`. This example also shows a number of complex conditional
variable assignments using the :ref:`tdl-step-assign` step. Finally, the :ref:`tdl-step-if` and :ref:`tdl-step-while` steps are used to construct
the appropriate control flow logic to ensure that all the container's applicable files get validated accordingly.

**Download:** :download:`example7.zip`

