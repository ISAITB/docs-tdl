The ``steps`` element is where the test case's testing logic is implemented. It consists of a sequence of test steps realised by means
of a GITB TDL step construct. The structure of the element is as follows:

.. csv-table::
    :stub-columns: 1
    :delim: ~
    :header: "Name", "Required?", "Description"

    @logLevel~ no~ The minimum logging level that this test case should produce. This can be (in increasing severity) ``DEBUG``, ``INFO`` (the default level), ``WARNING`` or ``ERROR``, but can also be set dynamically as a variable reference. See also the :ref:`tdl-step-log` step.
    @stopOnError~ no~ A boolean flag determining whether the test session should stop if any step fails (default is "false"). See also :ref:`tdl-steps-common-stoponerror`.

.. _test-case-steps__logging:

Test case logging
+++++++++++++++++

The test case's **logging level** affects log statements produced automatically by the Test Bed or :ref:`added explicitly by the test case<tdl-step-log>`.
While executing a test session, the Test Bed automatically produces the following log output:

* At ``DEBUG`` level, information on each step's start, end and latest status.
* At ``INFO`` level, information on key lifecycle points such as the start and end of the session.
* At ``WARNING`` level, detected issues that although not blocking for the test session could be signs of problems.
* At ``ERROR`` level, information on unexpected errors that forced the test session to fail.

When using the ``logLevel`` attribute to set the test case's log level, this defines the minimum level of messages to be added to the
session's log. A good example is setting the ``logLevel`` to ``WARN`` which will exclude all ``DEBUG`` and ``INFO`` output while including all output 
of levels ``WARNING`` and ``ERROR``. This could be interesting if you want to only share test engine errors and problematic issues signalled by
using the :ref:`tdl-step-log` step while ignoring status updates.

In certain cases you may prefer to set the test case logging level dynamically. This is achieved by using a variable reference as the 
``logLevel`` value, referring either to a :ref:`configuration property<test-case-configuration>` or a :ref:`predefined test case variable<test-case-variables>`.
Interestingly, when referring to a variable and given that the provided expression is calculated every time, you can adapt the test case's
logging level during the course of the test session. You may for example start with a ``WARNING`` level but switch to ``INFO``
for a specific set of steps. An example of this is illustrated below:

.. code-block:: xml

    <testcase>
        <variables>
            <var name="loggingLevel" type="string">
                <value>WARNING</value>
            </var>
        </variables>
        <steps logLevel="$loggingLevel">
            <!-- 
                The following log entry is ignored as we only log at WARNING level and above.
            -->
            <log level="INFO">'An info message'</log>
            <!-- 
                For the group that follows switch to INFO level.
            -->
            <assign to="loggingLevel">'INFO'</assign>
            <group>
                ...
            </group>
            <!-- 
                Switch back to WARNING level.
            -->
            <assign to="loggingLevel">'WARNING'</assign>
            ...
        </steps>
    </testcase>

.. _test-case-steps__steps:

Available steps
+++++++++++++++

The test case's steps are defined as children of the ``steps`` element. The available test steps that can be defined are:

.. csv-table::
    :widths: 30, 70
    :header: "Step name", "Description"

    :ref:`tdl-step-send`, Send a message to an actor
    :ref:`tdl-step-receive`, Receive a message from an actor
    :ref:`tdl-step-listen`, Listen for exchanged messages between actors
    :ref:`tdl-step-btxn`, Begin a messaging transaction
    :ref:`tdl-step-etxn`, End a messaging transaction
    :ref:`tdl-step-process`, Process a set of inputs to get an output
    :ref:`tdl-step-bptxn`, Begin a processing transaction
    :ref:`tdl-step-eptxn`, End a processing transaction
    :ref:`tdl-step-if`, Apply if-else logic to conditionally execute test steps
    :ref:`tdl-step-while`, Loop over a set of steps while a condition is true
    :ref:`tdl-step-repuntil`, Repeat a set of steps while a condition is true (executing at least once)
    :ref:`tdl-step-foreach`, Execute a set of steps a fixed number of times
    :ref:`tdl-step-flow`, Execute sets of steps concurrently
    :ref:`tdl-step-exit`, Immediately terminate the test session
    :ref:`tdl-step-assign`, Process an expression and assign its output to a variable
    :ref:`tdl-step-log`, Log a message in the test session log.
    :ref:`tdl-step-group`, Display a set of steps as a logical group
    :ref:`tdl-step-verify`, Validate content
    :ref:`tdl-step-call`, Call a scriptlet
    :ref:`tdl-step-interact`, Trigger an interaction with the user
