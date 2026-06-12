The ``preliminary`` element allows the test case to interact with the user before the test session begins. The purpose here is to allow the
user to provide preliminary input or be informed of certain actions that need to take place before the test session starts. In terms of structure 
and use, the ``preliminary`` element is a ``UserInteraction`` element (see :ref:`tdl-step-interact`). The difference is that the interaction takes place before the 
test session actually starts.

The following example shows a test case that prompts the user before starting to initialise their server and upload a configuration file.

.. code-block:: xml

    <testcase>
        <preliminary desc="Prepare your system">
            <instruct desc="Preparation instructions">"Make sure your system is up and running"</instruct>
            <request desc="Provide your configuration file" contentType="BASE64">$sutConfigFile</request>
        </preliminary>
        <variables>
            <var name="sutConfigFile" type="binary"/>
        </variables>
        <actors>
            <gitb:actor id="User" name="User" role="SUT"/>
        </actors>
        <steps>
            <!-- 
                The provided file can be referenced as $sutConfigFile 
            -->
        </steps>
    </testcase>
