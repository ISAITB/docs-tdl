Steps that perform validations support a ``level`` attribute to define their **severity level**. This attribute, by default considered as ``ERROR``,
can be set to ``WARNING`` so that a failure is presented as a warning, and importantly does not impact the overall result of the test session. Moreover,
this attribute can be set with a fixed value, or evaluated dynamically as a :ref:`variable reference<test-case-referring-to-variables>`.

The steps that support the ``level`` attribute are:

* The :ref:`verify <tdl-step-verify>` step.
* The :ref:`process <tdl-step-process>` step.
* The :ref:`send <tdl-step-send>`, :ref:`receive <tdl-step-receive>` and :ref:`listen <tdl-step-listen>` messaging steps.

Setting a :ref:`verify <tdl-step-verify>` step's severity level to ``WARNING`` can be useful if you want to carry out **optional checks**. These will be
appropriately displayed as warnings, leaving the overall test session outcome unaffected. This is illustrated in the following example, which goes
a step further defining the severity level through configuration:

.. code-block:: xml

    <!-- 
        Check required assertions (the severity level is implicitly "ERROR").
    -->
    <verify handler="XmlValidator" desc="Check core features">
        ...
    </verify>
    <!-- 
        Check experimental assertions. The severity level is determined via a domain
        configuration parameter, set either as "WARNING" or "ERROR".
    -->
    <verify handler="XmlValidator" desc="Check optional features" level="$DOMAIN{optionalLevel}">
        ...
    </verify>

Similarly, setting the severity level on messaging and processing steps is useful given that these steps can also be used to return validation
reports. Besides managing optional validations however, setting such steps at warning level ensures that they will never result in
test session failures. Scnearios where this could be useful include:

* Using a :ref:`process <tdl-step-process>` step to trigger **downstream actions** such as collecting statistics.
* Using one or more :ref:`send <tdl-step-send>` steps to perform **teardown operations** on the SUT.

The following example illustrates two teardown HTTP calls that we would want to execute at the end of a test session, ensuring 
that any failures they produce are ignored:

.. code-block:: xml

    <group title="Teardown phase" collapsed="true">
        <send desc="Delete test dataset 1" handler="HttpMessagingV2" level="WARNING">...</send>
        <send desc="Delete test dataset 2" handler="HttpMessagingV2" level="WARNING">...</send>
    </group>
