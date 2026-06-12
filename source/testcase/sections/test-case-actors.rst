The ``actors`` element is where the test case defines the actors involved in its steps and, importantly, their role. It
defines the following attributes:

.. csv-table::
    :stub-columns: 1
    :delim: ~
    :header: "Name", "Required?", "Description"

    @adminDisplayOrder ~ no ~ A number indicating the relative display order of the predefined administrator actor used for :ref:`administrator interactions <tdl-step-interact_admin_interactions>`.
    @adminName ~ no ~ A custom name to display for the predefined administrator actor used for :ref:`administrator interactions <tdl-step-interact_admin_interactions>` (default is "Administrator").
    @engineDisplayOrder ~ no ~ A number indicating the relative display order of the predefined test engine actor used to display :ref:`verify <tdl-step-verify>`, :ref:`process <tdl-step-process>` and :ref:`exit <tdl-step-exit>` steps.
    @engineName ~ no ~  A custom name to display for the predefined test engine actor used to display :ref:`verify <tdl-step-verify>`, :ref:`process <tdl-step-process>` and :ref:`exit <tdl-step-exit>` steps (default is "Test engine").
    @userDisplayOrder ~ no ~  A number indicating the relative display order of the predefined user actor used for :ref:`user interactions <tdl-step-interact>`.
    @userName ~ no ~ A custom name to display for the predefined user actor used for :ref:`user interactions <tdl-step-interact>` (default is the ``SUT`` actor's name).

Besides these attributes, the ``actors`` element defines one or more ``actor`` children to list the specification actors
involved in the test case. Each ``actor`` element has the following structure:

.. csv-table::
    :stub-columns: 1
    :delim: ~
    :header: "Name", "Required?", "Description"

    @displayOrder ~ no ~ A number indicating the relative positioning that needs to be respected when displaying the actor in test case's execution diagram. Setting this here overrides any corresponding :ref:`setting at test suite level <test-suite-actors>`.
    @id ~ yes ~ The actor's unique (within the specification) ID. This must match an actor ID specified in the test suite.
    @name ~ no ~ The name to display for the actor. This can differ from the ID to display an actor name specific to the test case. Not specifying this will default to the name for actor provided in the test suite.
    @role ~ no ~ The actor's role in the test case. This is "SUT" if the actor is the focus of the test case, or "SIMULATED" (the default value) if the actor is simulated by the Test Bed.

The main purpose of the ``actors`` element is to identify which of the :ref:`actors defined in the test suite <test-suite-actors>`
is the SUT (System Under Test - the actor the target system is testing for). This is done by defining the ``role`` attribute as follows:

.. code-block:: xml

    <testcase>
        <actors>
          <gitb:actor id="sender" role="SUT"/>
          <!-- The "SIMULATED" role is considered by default. -->
          <gitb:actor id="receiver"/>
        </actors>
    </testcase>

When a test case is executed, it is presented as a series of :ref:`test steps <tdl-steps>` associated to the test case's
:ref:`actors <test-case-actors>`. The presentation resembles a sequence diagram with the actors displayed at the top as
the diagram's lifelines. The displayed actors include:

* The actors defined in the test case using the ``actor`` elements.
* Predefined actors to represent the **user** (for user inputs), the **administrator** (for administrator inputs), and the **test engine** (for validations and processing steps).

Actors are displayed as long as they are associated with a visible test step. For your defined actors this takes place
by referring to them in :ref:`send <tdl-step-send>` and :ref:`receive <tdl-step-receive>` steps, whereas for
predefined actors this involves the use of other displayed steps, notably :ref:`interact <tdl-step-interact>`,
:ref:`process <tdl-step-process>`, :ref:`verify <tdl-step-verify>` and :ref:`exit <tdl-step-exit>` steps.

.. note::
    You can also :ref:`explicitly set the actor <tdl-steps-common-step-actor>` under which to display each test step,
    overriding the Test Bed's default display.

The presentation of actors, both declared in the test case and predefined ones, can be managed by adapting
their name and relative display order. For test case actors declared in ``actor`` elements, you do this by setting the
``name`` and ``displayOrder`` attributes, whereas for predefined actors you use:

* ``adminName`` and ``adminDisplayOrder`` for the administrator actor.
* ``userName`` and ``userDisplayOrder`` for the user actor.
* ``engineName`` and ``engineDisplayOrder`` for the test engine actor.

The following example shows use of custom ordering and naming to manage the actors' display:

.. code-block:: xml

    <testcase>
        <!--
            Display first the SUT 'sender' actor, followed by the simulated 'receiver' actor.
            Following these present the user, administrator and test engine actors (as needed).
        -->
        <actors userName="User" userDisplayOrder="2" adminName="Administrator" adminDisplayOrder="3" engineName="Test Bed" engineDisplayOrder="4">
          <gitb:actor id="sender" role="SUT" name="Message sender" displayOrder="0"/>
          <gitb:actor id="receiver" name="Message receiver" displayOrder="1"/>
        </actors>
    </testcase>
