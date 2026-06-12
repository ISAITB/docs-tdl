Domain configuration parameters apply to all its specifications and actors, as well as the organisations and their systems that define
relevant conformance statements. Such parameters typically relate to cross-cutting values or configuration of a much more technical 
nature. As an example consider a :ref:`tdl-step-verify` step that makes use of an external validation service (see :ref:`handlers`)
to validate content. It could be interesting to define the address of the validation service endpoint in configuration rather than
in each test case. This approach allows for more flexibility in terms of:

* **Portability:** Allowing test cases to automatically switch from local development settings to production settings or to point to a different
  service instance.
* **Implementation flexibility:** Allowing the actual implementation of a service handler to change without impacting test cases.
* **Maintainability:** Specifying and reusing values that can be changed transparently to test cases.
* **Sensitivity:** Allowing sensitive properties such as passwords to be used without including them in the GITB TDL content.

Using such domain-level configuration properties finds an obvious use case in service handler definitions but is not 
restricted to that. Such configuration can be used anywhere a value needs to be used that is common across all test cases.

Domain-level configuration properties are made available to test sessions in a ``map`` named **DOMAIN** that contains key-value pairs matching
the configured parameters. This ``map`` can be used in expressions in exactly the same way other variables and configuration entries are used
as illustrated in the following examples:

.. code-block:: xml
    :emphasize-lines: 2,6

    <!-- Validation service URL configured at domain level as "handlerURL". -->
    <verify desc="Validate content" handler="$DOMAIN{handlerURL}">
        <input name="content">$theContentToValidate</input>
    </verify>
    <!-- Use the "domainLevelValue" constant configured at domain level in calculations. -->
    <assign to="result">$aValue + $DOMAIN{domainLevelValue}</assign>

.. note::
    **GITB software support:** Use of the **DOMAIN** map is specific to the GITB software. If running on different 
    software it would need to be defined and populated within the test case itself.
