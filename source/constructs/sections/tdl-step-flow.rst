The ``flow`` step is used to perform sequences of steps in parallel rather that sequentially as is the default. This can be useful
in scenarios where you want to process data in parallel or trigger messaging to actors concurrently. The flow of execution will be 
joined at the end of the ``flow`` step to continue sequential execution. The structure of the ``flow`` element is as follows:

.. csv-table::
    :stub-columns: 1
    :header: "Name", "Required?", "Description"

    @collapsed, no, A boolean flag determining whether or not the step is displayed as initially collapsed (default is "false"). See also :ref:`tdl-steps-common-collapsed`.
    @desc, no, A description for this thread fork to display to the user and to include in the test session log. Within scriptlets this can also be :ref:`a variable reference<scriptlets_dynamic_references>`.
    @hidden, no, A boolean flag determining whether or not the step is displayed to users (default is "false"). Note that within scriptlets this can also be :ref:`a variable reference<scriptlets_dynamic_references>`. See :ref:`tdl-steps-common-hidesteps` for further details.
    @skipped, no, A boolean value or variable reference (default being "false") which will result in the step being skipped if "true". See also :ref:`tdl-steps-common-skipped`.
    @stopOnError, no, A boolean flag determining whether the test session should end if this step fails (default is "false"). See also :ref:`tdl-steps-common-stoponerror`.
    @title, no, A short title to display for this step (default is "flow"). Within scriptlets this can also be :ref:`a variable reference<scriptlets_dynamic_references>`.
    documentation, no, Rich text content that provides further information on the current step.
    thread, yes, One or more elements containing as children any sequence of steps to execute in the thread (including other ``flow`` steps).

The following example makes a HTTP GET to two API endpoints in parallel and proceeds to call a third one when both requests have completed.

.. code-block:: xml

    <flow desc="Contact endpoints api1 and api2 in parallel">
        <thread>
            <!--
                Call api1 and wait for response.
            -->
            <send id="dataSend1" desc="Call API 1" from="Sender" to="Receiver" handler="HttpMessagingV2">
                <input name="uri">"https://my.sut.org/api1/get"</input>
                <input name="method">"GET"</input>
            </send>
        </thread>
        <thread>
            <!--
                Call api2 and wait for response.
            -->
            <send id="dataSend1" desc="Call API 2" from="Sender" to="Receiver" handler="HttpMessagingV2">
                <input name="uri">"https://my.sut.org/api2/get"</input>
                <input name="method">"GET"</input>
            </send>
        </thread>
    </flow>
    <!-- 
        After both requests have completed make a new HTTP GET to api3.
    -->
    <send id="dataSend" desc="Call API 3"  from="Sender" to="Receiver" handler="HttpMessagingV2">
        <input name="uri">"https://my.sut.org/api3/get"</input>
        <input name="method">"GET"</input>
    </send>

A ``flow`` step's separate threads can also be **individually hidden** through use of their ``hidden`` attribute. This could be useful 
if you need to execute and display a set of parallel test branches, but at the same time carry out a parallel operation that
shouldn't be displayed. The following example illustrates a scenario where two messages are sent in parallel, with an additional
notification that is hidden.

.. code-block:: xml
    :emphasize-lines: 10

    <flow desc="Send messages">
        <thread>
            <log>"Calling API 1"</log>
            <send id="dataSendA" desc="Call API 1" from="Sender" to="Receiver" handler="HttpMessagingV2">...</send>
        </thread>
        <thread>
            <log>"Calling API 2"</log>
            <send id="dataSendB" desc="Call API 2" from="Sender" to="Receiver" handler="HttpMessagingV2">...</send>
        </thread>
        <thread hidden="true">
            <log>"Calling clean-up API"</log>
            <send id="cleanUp" from="Sender" to="Receiver" handler="HttpMessagingV2">...</send>
        </thread>
    </flow>

.. note::
    **Parallel receives:** In case you use the :ref:`receive<tdl-step-receive>` step within a ``flow`` step's threads and a
    :ref:`custom messaging service<handlers>`, you need to make sure your service manages the specific receive call's identifier.
    Check the `messaging service documentation`_ for details on how to do this.
