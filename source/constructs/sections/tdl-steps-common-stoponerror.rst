By default a test session continues processing all steps regardless of failures. If you want to stop test execution upon a failure you can use
the boolean ``stopOnError`` flag. This flag can be set on:

* **Individual steps**, to stop the test for failures on the step in question, or any nested step.
* **Step sequences**, to stop the test for failures on any of the sequence's steps (or nested step). Examples of step sequence elements are the ``then`` or ``else`` blocks of an :ref:`if step<tdl-step-if>`.
* The **complete test case**, to stop the test on any failure. This is done by setting the ``stopOnError`` attribute on the test case's ``steps`` element.

The following GITB TDL snippets illustrates all these cases:

.. code-block:: xml

    <!-- Stop on any failure. -->
    <steps stopOnError="true">
        ...
    </steps>

    <!-- Stop if this step fails -->
    <verify stopOnError="true">
        ...
    </verify>

    <!-- Stop if any step within the "if" fails. -->
    <if stopOnError="true">
        ...
    </if>

    <!-- Stop if any step within the "then" block fails (but continue for failures under the else block). -->
    <if>
        <cond>...</cond>
        <then stopOnError="true">
            ...
        </then>
        <else>
            ...
        </else>
    </if>

It is interesting to note that stopping a test execution could also be achieved by means of the :ref:`if<tdl-step-if>` and 
:ref:`exit<tdl-step-exit>` steps. The following snippet illustrates such a case:

.. code-block:: xml

    <verify id="check" desc="Make an important validation">
    ...
    </verify>
    <if desc="check to stop the test">
        <cond>not($STEP_SUCCESS{check})</cond>
        <then>
            <exit desc="Stop the test"/>
        </then>
    </if>

Doing this simply to prevent subsequent test steps is overly verbose and, moreover is displayed as part of the test execution diagram. It
could still be interesting to follow this approach however if you want to include additional :ref:`processing<tdl-step-process>` or :ref:`user interaction<tdl-step-interact>` 
steps before the session ends.

.. index:: stopOnChildError
.. _tdl-steps-common-stoponchilderror:

Skip child steps in case of error
+++++++++++++++++++++++++++++++++

Several test steps are used to define **step sequences**, in that they include additional steps as children. The simplest of these is the
:ref:`group step <tdl-step-group>` that aggregates related steps to manage their common behavior and display them visually as a group.
For such sequence steps we may want to continue their execution until a failure occurs, and then skip remaining steps but without terminating
the test session as a whole. To achieve this effect you can use the boolean ``stopOnChildError`` flag.

The ``stopOnChildError`` flag can be an interesting alternative to the ``stopOnError`` flag, where you want to skip processing certain steps but
ensure overall processing continues. A good example of this is a test case that includes **setup** and **teardown phases** around the core testing
steps. This is illustrated in the following test case:

.. code-block:: xml

    <steps>
        <!--
            Setup steps making HTTP calls to the SUT to initialise it with test datasets.
        -->
        <group title="Setup phase" collapsed="true">
            <send desc="Store test dataset 1" handler="HttpMessagingV2">...</send>
            <send desc="Store test dataset 2" handler="HttpMessagingV2">...</send>
        </group>
        <!--
            Carry out the test case's message exchanges and validations. By setting here
            stopOnChildError to true any error will skip subsequent steps within the group
            but will not terminate the test session.
        -->
        <group hiddenContainer="true" stopOnChildError="true">
            <send desc="Update data" handler="HttpMessagingV2">...</send>
            <verify desc="Check update result">...</verify>
            <send desc="Create new data" handler="HttpMessagingV2">...</send>
            <verify desc="Check create result">...</verify>
        </group>
        <!--
            Teardown steps making HTTP calls to the SUT to remove test data. Note also
            how the severity level of the steps is set to WARNING to make sure that a
            failed teardown step can never itself result in a test session failure.
        -->
        <group title="Teardown phase" collapsed="true">
            <send desc="Delete test dataset 1" handler="HttpMessagingV2" level="WARNING">...</send>
            <send desc="Delete test dataset 2" handler="HttpMessagingV2" level="WARNING">...</send>
        </group>
    </steps>

Groups with different ``stopOnChildError`` values can also be nested to fine tune the test execution. A typical case where this
can be useful is grouping together verifications to ensure all checks are carried out and reported. The following example
shows how we skip messages following a failure, but still carry out all verifications per messaging step:

.. code-block:: xml

    <steps>
        <!--
            Setting stopOnChildError to true ensures failures prevent other child steps from continuing.
        -->
        <group hiddenContainer="true" stopOnChildError="true">
            <send desc="Update data" handler="HttpMessagingV2">...</send>
            <!-- 
                Here we override the stopOnChildError semantics so that we run through all verify steps
                regardless of their individual results.
            -->
            <group title="Validations" stopOnChildError="false">
                <verify desc="Check status code">...</verify>
                <verify desc="Check payload">...</verify>
            </group>
            <!-- 
                In case the previous group step failed, the following steps will be skipped.
            -->
            <send desc="Create new data" handler="HttpMessagingV2">...</send>
            <group title="Validations" stopOnChildError="false">
                <verify desc="Check status code">...</verify>
                <verify desc="Check payload">...</verify>
            </group>
        </group>
    </steps>

The ``stopOnChildError`` flag can be set on any step that defines a sequence of steps, specifically:

* The :ref:`group step <tdl-step-group>`.
* The :ref:`if step <tdl-step-if>`, as well as individual ``then`` and ``else`` blocks.
* The iteration steps (:ref:`while <tdl-step-while>`, :ref:`foreach <tdl-step-foreach>`, :ref:`repuntil <tdl-step-repuntil>`).
* The :ref:`flow step <tdl-step-flow>`, as well as individual ``thread`` blocks.
* The :ref:`call step <tdl-step-call>` in which case it applies to the referred scriptlet's steps.
