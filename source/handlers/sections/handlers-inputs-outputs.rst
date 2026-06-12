The ``input`` and ``output`` elements used with handlers are what GITB refers to as "Binding elements".
They share the following structure:

.. csv-table::
    :header: "Name", "Required?", "Description"

    ``@asTemplate``, no, Whether or not the result will be considered as a template for placeholder replacement (see :ref:`test-case-expressions-template-files`). By default this is "false".
    ``@name``, no, The name of the input or output element.
    ``@lang``, no, The expression language that should be considered when evaluating its contained expression (see :ref:`test-case-expressions`).
    ``@source``, no, A pure variable reference identifying a source variable. Used as the target upon which to evaluate the contained expression.

The text content of the element is considered to be an expression (see :ref:`test-case-expressions`). In the case a ``source`` attribute is provided
the contained expression is evaluated on the variable identified by ``source`` to produce the value. If no ``source`` attribute is present the value
is the result of the expression itself. For inputs of type ``object`` or ``schema`` (i.e. XML documents) the ``source`` attribute can also be used to pass
the complete document as the value. In this case use of the ``source`` attribute to reference the relevant variable is equivalent to specifying its
reference as the expression:

.. code-block:: xml

    <verify handler="XmlValidator" desc="Validate content">
        <!--
            Pass document through the expression.
        -->
        <input name="xml">$docToValidate</input>
        <!--
            Pass document through the source attribute.
        -->
        <input name="schematron" source="$schematronFile"/>
    </verify>

.. note::
    **Specifying a fixed value:** Considering that the default expression language is XPath, a fixed text value is provided by enclosing it in
    quotes. See :ref:`test-case-expressions` for further details.

The ``input`` and ``output`` options for service handlers are documented as part of their module definition. For handlers accessible
via remote web service calls this information is returned when calling the handler's ``getModuleDefinition`` operation. This is also used internally
by the Test Bed before calling a service handler to ensure that required parameters are provided by the test case.