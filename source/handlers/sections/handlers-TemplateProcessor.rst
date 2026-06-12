Used to generate text data based on a provided template. Using this processing handler instead of the basic
:ref:`GITB TDL templating capabilities<test-case-expressions-template-files>` permits the decoupling of
information in the test session context and the template, and also generation of complex content based
on `FreeMarker templates <https://freemarker.apache.org/>`_. This processor should be used for any template-based
data generation that is not limited to simple placeholder replacements.

.. csv-table::
    :header: "Operation", "Description", "Input(s)", "Output(s)"
    :delim: |

    ``process`` | Process the provided template and parameters to produce the output. | Yes | Yes, a ``string`` named ``data`` in the resulting step's ``map``.

The input parameters expected by the ``process`` operation are as follows:

.. csv-table::
    :header: "Input name", "Required?", "Description"
    :delim: |

    ``parameters`` | No | A ``map`` with named inputs to use as the template's input.
    ``syntax`` | No | A ``string`` to specify what syntax the template uses. Accepted values are ``gitb`` (the default) and ``freemarker``.
    ``template`` | Yes | The template content to use (can be of any type that results in a ``string``).

The following example illustrates usage of the ``TemplateProcessor`` to create a message based on a FreeMarker template:

.. code-block:: xml

    <testcase>
        ...
        <imports>
            <artifact name="freemarkerTemplateFile">resources/template.xml</artifact>
        </imports>
        ...
        <steps>
            ...
            <assign to="parameters{value1}">'Value to use'</assign>
            <assign to="parameters{listValues}" append="true">1</assign>
            <assign to="parameters{listValues}" append="true">2</assign>
            <assign to="parameters{listValues}" append="true">3</assign>

            <process output="message" handler="TemplateProcessor">
                <input name="parameters">$parameters</input>
                <input name="template">$freemarkerTemplateFile</input>
                <input name="syntax">'freemarker'</input>
            </process>

            <log>$message</log>
            ...
        </steps>
    </testcase>

In this example the "freemarkerTemplateFile" variable is set via :ref:`import<test-case-imports>` to a template with the following content:

.. code-block:: none

    <?xml version="1.0" encoding="UTF-8"?>
    <data>
        <content>
            <value>${value1}</value>
            <items>
            <#list listValues as listValue>
                <item>${listValue}</item>
            </#list>
            </items>
        </content>
    </data>

.. note::
    **Importing the template as a non-binary variable:** When using the ``TemplateProcessor`` it is important to import the template as a ``binary`` variable
    (or to simply not specify the ``type`` attribute). If another type is used for the import you risk triggering the test engine's
    :ref:`built-in template processing <test-case-expressions-template-files>` that is not Freemarker-based.

Notice here how the template defines FreeMarker constructs (a list iteration) to go over the items of a collection named "listValues". This was passed in the
"parameters" ``map`` when calling the :ref:`process step<tdl-step-process>`. When executed, and considering our example, this step will produce data as follows:

.. code-block:: xml

    <?xml version="1.0" encoding="UTF-8"?>
    <data>
        <content>
            <value>Value to use</value>
            <items>
                <item>1</item>
                <item>2</item>
                <item>3</item>
            </items>
        </content>
    </data>