The ``imports`` element allows the use of arbitrary resources from the same or another test suite. This can be very useful when a test case needs to send messages
based on a template or load a binary file that is needed as input by a messaging, processing or validation handler (e.g. a certificate). The ``imports`` element 
defines one or more ``artifact`` children with the following structure:

.. csv-table::
    :stub-columns: 1
    :header: "Name", "Required?", "Description"
    :delim: |

    @encoding | No | In case the artefact is to be treated as text, this is the character encoding to apply when reading its bytes (default is "UTF-8").
    @from | No | The identifier of another test suite from which this resource will be loaded. If unspecified the current test suite is assumed.
    @name | Yes | The name with which this artefact will be associated to the test session context for subsequent lookups.
    @type | No | The type as which the artefact needs to be loaded (default is ``binary``).

The text value of the ``artifact`` element is the path within the test suite from which the relevant resource will be loaded. This path may be provided as a
fixed value or as a :ref:`variable reference<test-case-referring-to-variables>` to determine the imported resource dynamically. In case a variable reference
is provided this should be one of the following:

* A reference to a configuration value (i.e. a :ref:`domain<test-case-expressions-domain>`, :ref:`organisation<test-case-expressions-organisation>`, :ref:`system<test-case-expressions-system>` or :ref:`actor<test-case-expressions-actor>` parameter).
* A reference to a :ref:`variable<test-case-variables>` defined in the test case. In this case the value of the variable can even be adapted during the course of the test session resulting in
  different resources depending on the point at which the import is referenced.

Importing resources is not limited to the current test suite. Using the ``from`` attribute it is possible to define another
test suite as the source from which to lookup the resource, specifying as the attribute's value the identifier of the target
test suite. The lookup of the test suite using the ``from`` value is carried out as follows:

#. Look for the test suite in the same **specification** as the test case being executed.
#. If not found in the same specification, look for the test suite in the other specifications of the test case's **domain**. If across
   specifications multiple matching test suites are found, one of them will be arbitrarily picked. To avoid such a scenario
   it is obvious that you should ensure test suites used to load shared resources can be uniquely identified.

The ``type`` attribute is optional and defaults to ``binary`` denoting a general-purpose file (regardless of whether it is text-based or not). You would
likely never need to set this explicitly, however if you choose to do so you can set it set as:

* ``binary``: To load the artefact as a set of bytes without additional processing.
* ``object``: To load the artefact as a XML Document Object Model.
* ``schema``: To load the artefact as a XML Schema or Schematron file.

In case the file is text-based you also have the option of setting the ``encoding`` attribute to consider (by default set as ``UTF-8``).

Regarding the path to the resource, this is considered first as relative to the test suite root, and if not found, as relative
to the test suite definition file. In both cases the resource path can be prefixed with the test suite identifier (done
for backwards compatibility and ideally avoided for new test suites).

As an example consider the following test case fragment where a XML schema is loaded and set in the session context as a variable named "ublSchema". The
path specified suggests that the file is named "UBL-Invoice-2.1.xsd" and exists in a folder within the test suite archive named "resources". This example also includes
another input whose referenced resource is defined dynamically based on an external configuration parameter (at organisation level in this case).

.. code-block:: xml

    <testcase>
        <imports>
            <!--
                The "ublSchema" is loaded from a fixed resource within the test suite.
            -->
            <artifact name="ublSchema">resources/UBL-Invoice-2.1.xsd</artifact>
            <!--
                The "organisationSpecificSchema" is loaded dynamically based on an organisation-level configuration property named "xsdToUseForOrganisation".
            -->
            <artifact name="organisationSpecificSchema">$ORGANISATION{xsdToUseForOrganisation}</artifact>
        </imports>
        <steps>
            <verify handler="XmlValidator" desc="Validate invoice against UBL 2.1 Invoice Schema">
                <!-- 
                    Variable $fileContent is loaded in another step.
                -->
                <input name="xml">$fileContent</input>
                <input name="xsd">$ublSchema</input>
            </verify>
        </steps>
    </testcase>

It is also possible to :ref:`import resources from other test suites <test-suite-sharing>`. To do this you use the ``from`` attribute identifying the
test suite that contains the resource, in which case the provided path is resolved in the context of the other test suite.

.. code-block:: xml

    <testcase>
        <imports>
            <!--
                The "ublSchema" is loaded from a fixed resource within a test suite with identifier "testSuite2".
            -->
            <artifact name="ublSchema" from="testSuite2">resources/UBL-Invoice-2.1.xsd</artifact>
        </imports>
    </testcase>

.. note::
    **Test module import:** The GITB TDL schema allows also for ``module`` elements to be defined for the import of test modules (validation, 
    messaging and processing handlers). This approach is no longer supported as it required the handler implementations to be bundled within 
    the Test Bed itself. The preferred and simpler approach now is to simply define the handler in the respective test step (e.g. the ``verify``
    step's ``handler`` attribute for validators) without previously importing it.
