.. index:: Service handlers
.. index:: Step handlers
.. _handlers:

Test step handlers
==================

The architectural approach followed by the GITB TDL is to capture in the test case the high level testing flow 
and delegate detailed domain-specific processing to separate services. These services can cover messaging
between actors, complex processing or content validation and implement APIs that are defined in the GITB
specification. The components implementing these services are termed generally **handlers** and, depending on
their purpose can be:

* :ref:`introduction-concepts-messaging-handlers` implementing the `GITB messaging service API`_.
* :ref:`introduction-concepts-processing-handlers` implementing the `GITB processing service API`_.
* :ref:`introduction-concepts-validation-handlers` implementing the `GITB validation service API`_.

.. _GITB messaging service API: https://www.itb.ec.europa.eu/specs/latest/gitb_ms.wsdl
.. _GITB processing service API: https://www.itb.ec.europa.eu/specs/latest/gitb_ps.wsdl
.. _GITB validation service API: https://www.itb.ec.europa.eu/specs/latest/gitb_vs.wsdl

Another important distinction for handlers is whether they are **built-in** within the Test Bed software or **external**.
For handlers that relate to domain-specific operations, the norm is to externalise them as remotely callable services.
Nonetheless several common tasks that are frequently encountered in test cases are also available as built-in test engine capabilities.

In the sections that follow you can find:

* The supported :ref:`built-in handlers<handlers-predefined-handlers>` covering common tasks encountered in test cases.
* The list of :ref:`reusable external services <handlers-reusable-handlers>` maintained by the Test Bed team (also usable locally as components).
* Guidelines to implement :ref:`custom external services <handlers-custom-handlers>` to cover project-specific needs.

.. _handlers-implementation:

Specifying the handler implementation
-------------------------------------

.. include:: /handlers/sections/handlers-implementation.rst

.. index:: Built-in handlers
.. _handlers-predefined-handlers:

Built-in handlers
-----------------

.. include:: /handlers/sections/handlers-predefined-handlers.rst

.. index:: Built-in messaging handlers
.. _handlers-predefined-handlers-messaging:

Built-in messaging handlers
~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. include:: /handlers/sections/handlers-predefined-handlers-messaging.rst

.. index:: DomibusMessaging
.. _handlers-DomibusMessaging:

DomibusMessaging
++++++++++++++++

.. include:: /handlers/sections/handlers-DomibusMessaging.rst

.. index:: HttpMessagingV2
.. _handlers-httpmessagingv2:

HttpMessagingV2
+++++++++++++++

.. include:: /handlers/sections/handlers-httpmessagingv2.rst

.. index:: SimulatedMessaging
.. index:: parameters (SimulatedMessaging)
.. index:: contentTypes (SimulatedMessaging)
.. index:: result (SimulatedMessaging)
.. index:: delay (SimulatedMessaging)
.. index:: reportItems (SimulatedMessaging)
.. index:: reportSteps (SimulatedMessaging)
.. index:: sortReportBySeverity (SimulatedMessaging)
.. _handlers-simulatedmessaging:

SimulatedMessaging
++++++++++++++++++

.. include:: /handlers/sections/handlers-simulatedmessaging.rst

.. index:: SoapMessagingV2
.. _handlers-soapmessagingv2:

SoapMessagingV2
+++++++++++++++

.. include:: /handlers/sections/handlers-soapmessagingv2.rst

.. index:: Built-in processing handlers
.. _handlers-predefined-handlers-processing:

Built-in processing handlers
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The title of each section corresponds to the name of the handler that needs to be configured in the relevant step's ``handler`` attribute (in 
:ref:`process <tdl-step-process>` or :ref:`bptxn <tdl-step-bptxn>` steps).

.. index:: Base64Processor
.. index:: encode (Base64Processor)
.. index:: decode (Base64Processor)
.. index:: input (Base64Processor)
.. index:: dataUrl (Base64Processor)
.. _handlers-Base64Processor:

Base64Processor
+++++++++++++++

.. include:: /handlers/sections/handlers-Base64Processor.rst

.. index:: CollectionUtils
.. index:: append (CollectionUtils)
.. index:: size (CollectionUtils)
.. index:: clear (CollectionUtils)
.. index:: contains (CollectionUtils)
.. index:: find (CollectionUtils)
.. index:: randomKey (CollectionUtils)
.. index:: randomValue (CollectionUtils)
.. index:: remove (CollectionUtils)
.. index:: map (CollectionUtils)
.. index:: fromMap (CollectionUtils)
.. index:: toMap (CollectionUtils)
.. index:: list (CollectionUtils)
.. index:: fromList (CollectionUtils)
.. index:: toList (CollectionUtils)
.. index:: value (CollectionUtils)
.. index:: item (CollectionUtils)
.. index:: onlyMissing (CollectionUtils)
.. index:: ignoreCase (CollectionUtils)
.. index:: entries (CollectionUtils)
.. index:: keys (CollectionUtils)
.. index:: values (CollectionUtils)
.. _handlers-CollectionUtils:

CollectionUtils
+++++++++++++++

.. include:: /handlers/sections/handlers-CollectionUtils.rst

.. index:: DelayProcessor
.. index:: delay (DelayProcessor)
.. index:: duration (DelayProcessor)
.. _handlers-DelayProcessor:

DelayProcessor
++++++++++++++

.. include:: /handlers/sections/handlers-DelayProcessor.rst

.. index:: DisplayProcessor
.. index:: result (DisplayProcessor)
.. index:: parameters (DisplayProcessor)
.. index:: contentTypes (DisplayProcessor)
.. index:: reportItems (DisplayProcessor)
.. index:: reportSteps (DisplayProcessor)
.. index:: sortReportBySeverity (DisplayProcessor)
.. _handlers-DisplayProcessor:

DisplayProcessor
++++++++++++++++

.. include:: /handlers/sections/handlers-DisplayProcessor.rst

.. index:: JsonPathProcessor
.. index:: content (JsonPathProcessor)
.. index:: expression (JsonPathProcessor)
.. index:: outputType (JsonPathProcessor)
.. index:: asYaml (JsonPathProcessor)
.. _handlers-JsonPathProcessor:

JsonPathProcessor
+++++++++++++++++

.. include:: /handlers/sections/handlers-JsonPathProcessor.rst

.. index:: JsonPointerProcessor
.. index:: JSONPointerProcessor
.. index:: process (JsonPointerProcessor)
.. index:: content (JsonPointerProcessor)
.. index:: pointer (JsonPointerProcessor)
.. index:: asYaml (JsonPointerProcessor)
.. _handlers-JSONPointerProcessor:

JsonPointerProcessor
++++++++++++++++++++

.. include:: /handlers/sections/handlers-JsonPointerProcessor.rst

.. index:: RdfUtils
.. index:: convert (RdfUtils)
.. index:: merge (RdfUtils)
.. index:: ask (RdfUtils)
.. index:: select (RdfUtils)
.. index:: construct (RdfUtils)
.. index:: model (RdfUtils)
.. index:: models (RdfUtils)
.. index:: query (RdfUtils)
.. index:: inputContentType (RdfUtils)
.. index:: inputContentTypes (RdfUtils)
.. index:: outputContentType (RdfUtils)
.. index:: output (RdfUtils)
.. _handlers-RdfUtils:

RdfUtils
++++++++

.. include:: /handlers/sections/handlers-RdfUtils.rst

.. index:: RegExpProcessor
.. index:: check (RegExpProcessor)
.. index:: collect (RegExpProcessor)
.. index:: input (RegExpProcessor)
.. index:: expression (RegExpProcessor)
.. index:: output (RegExpProcessor)
.. _handlers-RegExpProcessor:

RegExpProcessor
+++++++++++++++

.. include:: /handlers/sections/handlers-RegExpProcessor.rst

.. index:: TemplateProcessor
.. index:: template (TemplateProcessor)
.. index:: parameters (TemplateProcessor)
.. index:: syntax (TemplateProcessor)
.. _handlers-TemplateProcessor:

TemplateProcessor
+++++++++++++++++

.. include:: /handlers/sections/handlers-TemplateProcessor.rst

.. index:: TokenGenerator
.. index:: zone (TokenGenerator)
.. index:: time (TokenGenerator)
.. index:: string (TokenGenerator)
.. index:: timestamp (TokenGenerator)
.. index:: uuid (TokenGenerator)
.. index:: format (TokenGenerator)
.. index:: diff (TokenGenerator)
.. index:: date (TokenGenerator)
.. index:: inputFormat (TokenGenerator)
.. index:: random (TokenGenerator)
.. index:: minimum (TokenGenerator)
.. index:: maximum (TokenGenerator)
.. index:: integer (TokenGenerator)
.. _handlers-TokenGenerator:

TokenGenerator
++++++++++++++

.. include:: /handlers/sections/handlers-TokenGenerator.rst

.. index:: VariableUtils
.. index:: type (VariableUtils)
.. index:: exists (VariableUtils)
.. index:: name (VariableUtils)
.. _handlers-VariableUtils:

VariableUtils
+++++++++++++

.. include:: /handlers/sections/handlers-VariableUtils.rst

.. index:: XPathProcessor
.. index:: input (XPathProcessor)
.. index:: expression (XPathProcessor)
.. index:: type (XPathProcessor)
.. _handlers-XPathProcessor:

XPathProcessor
++++++++++++++

.. include:: /handlers/sections/handlers-XPathProcessor.rst

.. index:: XsltProcessor
.. index:: XSLTProcessor
.. index:: xml (XsltProcessor)
.. index:: xslt (XsltProcessor)
.. _handlers-XSLTProcessor:

XsltProcessor
+++++++++++++

.. include:: /handlers/sections/handlers-XSLTProcessor.rst

.. index:: YamlConverter
.. index:: jsonToYaml (YamlConverter)
.. index:: yamlToJson (YamlConverter)
.. index:: content (YamlConverter)
.. _handlers-YamlConverter:

YamlConverter
+++++++++++++

.. include:: /handlers/sections/handlers-YamlConverter.rst

.. index:: ZipProcessor
.. index:: initialize (ZipProcessor)
.. index:: extract (ZipProcessor)
.. _handlers-ZipProcessor:

ZipProcessor
++++++++++++

.. include:: /handlers/sections/handlers-ZipProcessor.rst

.. index:: Built-in validation handlers
.. _handlers-predefined-validation-handlers:

Built-in validation handlers
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The title of each section corresponds to the name of the handler that needs to be configured in the relevant :ref:`verify <tdl-step-verify>` step's ``handler`` attribute.

.. index:: ExpressionValidator
.. index:: expression (ExpressionValidator)
.. index:: successMessage (ExpressionValidator)
.. index:: failureMessage (ExpressionValidator)
.. _handlers-ExpressionValidator:

ExpressionValidator
+++++++++++++++++++

.. include:: /handlers/sections/handlers-ExpressionValidator.rst

.. index:: JsonValidator
.. index:: json (JsonValidator)
.. index:: schema (JsonValidator)
.. index:: schemaCombinationApproach (JsonValidator)
.. index:: supportYaml (JsonValidator)
.. index:: showSchema (JsonValidator)
.. index:: sharedSchemaPaths (JsonValidator)
.. _handlers-JsonValidator:

JsonValidator
+++++++++++++

.. include:: /handlers/sections/handlers-JsonValidator.rst

.. index:: NumberValidator
.. index:: actual (NumberValidator)
.. index:: actualnumber (NumberValidator)
.. index:: expected (NumberValidator)
.. index:: expectednumber (NumberValidator)
.. index:: successMessage (NumberValidator)
.. index:: failureMessage (NumberValidator)
.. _handlers-NumberValidator:

NumberValidator
+++++++++++++++

.. include:: /handlers/sections/handlers-NumberValidator.rst

.. index:: RegExpValidator
.. index:: input (RegExpValidator)
.. index:: expression (RegExpValidator)
.. index:: successMessage (RegExpValidator)
.. index:: failureMessage (RegExpValidator)
.. _handlers-RegExpValidator:

RegExpValidator
+++++++++++++++

.. include:: /handlers/sections/handlers-RegExpValidator.rst

.. index:: SchematronValidator
.. index:: schematron (SchematronValidator)
.. index:: xmldocument (SchematronValidator)
.. index:: xml (SchematronValidator)
.. index:: type (SchematronValidator)
.. index:: showSchematron (SchematronValidator)
.. index:: sortBySeverity (SchematronValidator)
.. index:: showTests (SchematronValidator)
.. index:: showLocationPaths (SchematronValidator)
.. _handlers-SchematronValidator:

SchematronValidator
+++++++++++++++++++

.. include:: /handlers/sections/handlers-SchematronValidator.rst

.. index:: ShaclValidator
.. index:: contentType (ShaclValidator)
.. index:: loadImports (ShaclValidator)
.. index:: model (ShaclValidator)
.. index:: reportContentType (ShaclValidator)
.. index:: shapes (ShaclValidator)
.. index:: showReport (ShaclValidator)
.. index:: showShapes (ShaclValidator)
.. index:: sortBySeverity (ShaclValidator)
.. _handlers-ShaclValidator:

ShaclValidator
++++++++++++++

.. include:: /handlers/sections/handlers-ShaclValidator.rst

.. index:: StringValidator
.. index:: actual (StringValidator)
.. index:: expected (StringValidator)
.. index:: actualstring (StringValidator)
.. index:: expectedstring (StringValidator)
.. index:: successMessage (StringValidator)
.. index:: failureMessage (StringValidator)
.. _handlers-StringValidator:

StringValidator
+++++++++++++++

.. include:: /handlers/sections/handlers-StringValidator.rst

.. index:: XmlMatchValidator
.. index:: xml (XmlMatchValidator)
.. index:: template (XmlMatchValidator)
.. index:: ignoredPaths (XmlMatchValidator)
.. _handlers-XmlMatchValidator:

XmlMatchValidator
+++++++++++++++++

.. include:: /handlers/sections/handlers-XmlMatchValidator.rst

.. index:: XmlValidator
.. index:: xml (XmlValidator)
.. index:: xsd (XmlValidator)
.. index:: schematron (XmlValidator)
.. index:: schematronType (XmlValidator)
.. index:: stopOnXsdErrors (XmlValidator)
.. index:: sortBySeverity (XmlValidator)
.. index:: showValidationArtefacts (XmlValidator)
.. index:: showSchematronTests (XmlValidator)
.. index:: showLocationPaths (XmlValidator)
.. index:: schemaVersion (XmlValidator)
.. _handlers-XmlValidator:

XmlValidator
++++++++++++

.. include:: /handlers/sections/handlers-XmlValidator.rst

.. index:: XPathValidator
.. index:: xml (XPathValidator)
.. index:: expression (XPathValidator)
.. index:: xmldocument (XPathValidator)
.. index:: xpathexpression (XPathValidator)
.. index:: successMessage (StringValidator)
.. index:: failureMessage (StringValidator)
.. _handlers-XPathValidator:

XPathValidator
++++++++++++++

.. include:: /handlers/sections/handlers-XPathValidator.rst

.. index:: XsdValidator
.. index:: xsd (XsdValidator)
.. index:: xml (XsdValidator)
.. index:: XSDValidator
.. index:: xsddocument (XsdValidator)
.. index:: xmldocument (XsdValidator)
.. index:: showSchema (XsdValidator)
.. index:: sortBySeverity (XsdValidator)
.. index:: schemaVersion (XsdValidator)
.. _handlers-XSDValidator:

XsdValidator
++++++++++++

.. include:: /handlers/sections/handlers-XSDValidator.rst

.. index:: YamlValidator
.. index:: yaml (YamlValidator)
.. index:: schema (YamlValidator)
.. index:: schemaCombinationApproach (YamlValidator)
.. index:: showSchema (YamlValidator)
.. index:: supportJson (YamlValidator)
.. index:: sharedSchemaPaths (YamlValidator)
.. _handlers-YamlValidator:

YamlValidator
+++++++++++++

.. include:: /handlers/sections/handlers-YamlValidator.rst

.. _handlers-deprecated-handlers:

Deprecated built-in messaging handlers
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. include:: /handlers/sections/handlers-deprecated-handlers.rst

.. index:: Reusable external handlers
.. _handlers-reusable-handlers:

Reusable external handlers
--------------------------

The sections that follow list handler implementations that are maintained by the Test Bed team but that are not :ref:`built-into<handlers-predefined-handlers>`
the test engine itself. In all cases such handlers are defined as **service references** in the test steps that support them, and are available in two ways:

* As a **reusable service** hosted on Test Bed infrastructure that can be used as-is.
* As a **Docker image** registered on the `Docker Hub <https://hub.docker.com/>`_ that can be used to install a local instance of the service.

.. _handlers-reusable-handlers_messaging:

Messaging services
~~~~~~~~~~~~~~~~~~

The following sections summarise reusable messaging services that can be used as handler implementations for :ref:`messaging transactions<tdl-step-btxn>`,
as well as directly in :ref:`send<tdl-step-send>` and :ref:`receive<tdl-step-receive>` steps.

.. index:: AS4 (Reusable messaging services)
.. _handlers-reusable-handlers_messaging_as4:

AS4 messaging
+++++++++++++

.. include:: /handlers/sections/handlers-reusable-handlers_messaging_as4.rst

.. _handlers-reusable-handlers_processing:

Processing services
~~~~~~~~~~~~~~~~~~~

The following sections summarise reusable processing services that can be used as handler implementations for :ref:`process<tdl-step-process>` steps.

.. index:: ZIP (Reusable processing services)
.. _handlers-reusable-handlers_processing_zip:

ZIP processing
++++++++++++++

.. include:: /handlers/sections/handlers-reusable-handlers_processing_zip.rst

.. _handlers-reusable-handlers_validation:

Validation services
~~~~~~~~~~~~~~~~~~~

The following sections summarise reusable validation services that can be used as handler implementations for :ref:`verify<tdl-step-verify>` steps.

.. index:: ASiC (Reusable validation services)
.. _handlers-reusable-handlers_validation_asic:

ASiC validator
++++++++++++++

.. include:: /handlers/sections/handlers-reusable-handlers_validation_asic.rst

.. index:: CSV (Reusable validation services)
.. index:: TableSchema (Reusable validation services)
.. _handlers-reusable-handlers_validation_csv:

CSV validator
+++++++++++++

.. include:: /handlers/sections/handlers-reusable-handlers_validation_csv.rst

.. index:: JSON (Reusable validation services)
.. index:: JSON Schema (Reusable validation services)
.. _handlers-reusable-handlers_validation_json:

JSON validator
++++++++++++++

.. include:: /handlers/sections/handlers-reusable-handlers_validation_json.rst

.. index:: RDF (Reusable validation services)
.. index:: SHACL (Reusable validation services)
.. _handlers-reusable-handlers_validation_rdf:

RDF validator
+++++++++++++

.. include:: /handlers/sections/handlers-reusable-handlers_validation_rdf.rst

.. index:: XML (Reusable validation services)
.. index:: XML Schema (Reusable validation services)
.. index:: Schematron (Reusable validation services)
.. _handlers-reusable-handlers_validation_xml:

XML validator
+++++++++++++

.. include:: /handlers/sections/handlers-reusable-handlers_validation_xml.rst

.. index:: YAML (Reusable validation services)
.. index:: JSON Schema (Reusable validation services - YAML validator)
.. _handlers-reusable-handlers_validation_yaml:

YAML validator
++++++++++++++

.. include:: /handlers/sections/handlers-reusable-handlers_validation_yaml.rst

.. index:: Custom external handlers
.. _handlers-custom-handlers:

Custom external handlers
------------------------

.. include:: /handlers/sections/handlers-custom-handlers.rst

.. index:: Handler inputs and outputs
.. index:: name (Handler inputs and outputs)
.. index:: lang (Handler inputs and outputs)
.. index:: source (Handler inputs and outputs)
.. index:: asTemplate (Handler inputs and outputs)
.. _handlers-inputs-outputs:

Handler inputs and outputs
--------------------------

.. include:: /handlers/sections/handlers-inputs-outputs.rst
