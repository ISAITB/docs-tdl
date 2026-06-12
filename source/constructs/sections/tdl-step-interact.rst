The ``interact`` step is used to exchange information with the user executing the test case by means of a popup dialog.
It can be used both to present information to the user, as well as request inputs, depending on the child elements it defines.
These child elements include:

* **Instructions:** Informative messages or data to be presented to a user.
* **Requests:** Prompts to a user to provide input.

The structure of the ``interact`` element is as follows:

.. csv-table::
    :stub-columns: 1
    :header: "Name", "Required?", "Description"

    @actor, no, The identifier of an :ref:`actor <test-case-actors>` to replace the "Test engine" in the step's display. See also :ref:`tdl-steps-common-step-actor`.
    @blocking, no, A boolean flag determining whether or not the interaction step will force the test session to wait until the interation completes (default is "true").
    @collapsed, no, A boolean flag determining whether or not the step is displayed as initially collapsed (default is "false"). See also :ref:`tdl-steps-common-collapsed`.
    @desc, no, A description for the user interaction to display to the user and to include in the test session log. Within scriptlets this can also be :ref:`a variable reference<scriptlets_dynamic_references>`.
    @handler, no, The endpoint address for a `GITB messaging service <https://www.itb.ec.europa.eu/docs/services/latest/messaging/index.html>`__ that will be delegated the handling of this interaction in case ``handlerEnabled`` is true. See also :ref:`tdl-step-interact_handler`.
    @handlerEnabled, no, A boolean flag (by default "false") provided as a constant or a :ref:`variable reference <test-case-referring-to-variables>` to determine whether the interaction should be delegated to an external service (specified via the ``handler`` attribute). See also :ref:`tdl-step-interact_handler`.
    @handlerTimeout, no, A number or variable reference with the maximum time (in milliseconds) to wait for the handler service call to complete (in case of an external test service being used as a handler). See also :ref:`tdl-steps-common-handlerTimeouts`.
    @handlerTimeoutFlag, no, A string value with the name of a boolean variable to set informing whether or not a handler timeout occurred. See also :ref:`tdl-steps-common-handlerTimeouts`.
    @hidden, no, A boolean flag determining whether or not the step is displayed to users (default is "false"). Note that within scriptlets this can also be :ref:`a variable reference<scriptlets_dynamic_references>`. See :ref:`tdl-steps-common-hidesteps` for further details.
    @id, no, Used as the name of a ``map`` variable that will be used to store provided input (if no per-input assignment is provided).
    @inputTitle, no, A custom text to display as the title of the user input popup (default is "Server interaction"). Within scriptlets this can also be :ref:`a variable reference<scriptlets_dynamic_references>`.
    @skipped, no, A boolean value or variable reference (default being "false") which will result in the step being skipped if "true". See also :ref:`tdl-steps-common-skipped`.
    @stopOnError, no, A boolean flag determining whether the test session should end if this step fails (default is "false"). See also :ref:`tdl-steps-common-stoponerror`.
    @timeout, no, An optional timeout (in milliseconds) on the time to wait for the interaction to be completed. This is provided as a ``number`` or a variable reference.
    @title, no, A short title to display for this step (default is "interact"). Within scriptlets this can also be :ref:`a variable reference<scriptlets_dynamic_references>`.
    @with, no, The ID of the actor this interaction refers to. If not specified is is assumed to be the test case actor defined as the SUT. Within scriptlets this can also be :ref:`a variable reference<scriptlets_dynamic_references>`.
    documentation, no, Rich text content that provides further information on the current step.
    handlerConfig, no, Contains a set of ``input`` elements (see :ref:`handlers-inputs-outputs`) to define the inputs passed to an external ``handler`` service in case ``handlerEnabled`` is "true". If ``handlerEnabled`` is "false" this element is ignored.
    instruct, no, Zero or more elements to appear as instructions to the user.
    request, no, Zero or more input requests for the user.

.. index:: instruct (interact)
.. index:: desc (instruct)
.. index:: with (instruct)
.. index:: name (instruct)
.. index:: type (instruct)
.. index:: source (instruct)
.. index:: asTemplate (instruct)
.. index:: mimeType (instruct)
.. index:: forceDisplay (instruct)
.. index:: showControls (instruct)
.. index:: level (instruct)
.. index:: ERROR (instruct)
.. index:: WARNING (instruct)
.. index:: INFO (instruct)
.. index:: SUCCESS (instruct)
.. index:: NONE (instruct)
.. index:: report (instruct)

The ``instruct`` elements define what is going to presented to the user. They have the following structure:

.. csv-table::
    :stub-columns: 1
    :delim: ~
    :header: "Name", "Required?", "Description"

    @asTemplate~ no~ Whether or not the result will be considered as a template for placeholder replacement (see :ref:`test-case-expressions-template-files`). By default this is "false".
    @desc~ yes~ The label to display to the user.
    @forceDisplay~ no~ Whether the content should be always displayed inline rather than in an editor. By default this is "false".
    @level~ no~ A severity level used to stylise the presentation of the displayed text. Can be set to ``ERROR``, ``WARNING``, ``INFO``, ``SUCCESS`` and ``NONE`` (the default), and also be provided via :ref:`variable reference <test-case-referring-to-variables>`.
    @mimeType~ no~ A `mime type`_ value (e.g. ``text/xml``) provided as a string or a variable reference, to hint how this value should be highlighted when displayed. In case an invalid or unsupported mime type is provided no such highlighting will be applied.
    @name~ no~ In case of ``instruct`` elements that used to share binary content, this is used as the name of the file presented for download.
    @report~ no~ Whether To include this element in the step's report. By default this is "false".
    @showControls~ no~ Whether or not to display user interface controls for this elements (copy, view and download). By default such controls are displayed.
    @source~ no~ A pure variable reference identifying a source variable. Used as the target upon which to evaluate the contained expression.
    @type~ no~ The ``type`` to consider for the displayed value. If this is not specified the ``type`` will be inferred from the referred variable (if defined) or default to ``string``.
    @with~ no~ The ID of the actor this interaction refers to. If not specified this is taken from the ``interact`` parent element (which itself defaults to the test case's SUT actor). Within scriptlets this can also be :ref:`a variable reference<scriptlets_dynamic_references>`.

.. index:: request (interact)
.. index:: desc (request)
.. index:: with (request)
.. index:: contentType (request)
.. index:: encoding (request)
.. index:: name (request)
.. index:: options (request)
.. index:: optionLabels (request)
.. index:: multiple (request)
.. index:: asTemplate (request)
.. index:: inputType (request)
.. index:: mimeType (request)
.. index:: report (request)
.. index:: fileName (request)
.. index:: required(request)
.. index:: size(request)
.. index:: accept(request)
.. index:: default(request)

The ``request`` elements define how information shall be requested from the user. Their structure is as follows:

.. csv-table::
    :stub-columns: 1
    :delim: ~
    :header: "Name", "Required?", "Description"

    @asTemplate~ no~ Whether or not the result will be considered as a :ref:`template for placeholder replacement <test-case-expressions-template-files>`. By default this is "false".
    @contentType~ no~ Defines how the specified variable's value is to be set ("STRING", "BASE64" or "URI"). The default is "STRING".
    @default~ no~ The default value to display in the presented input field. This can be provided inline or via :ref:`variable reference <test-case-referring-to-variables>`.
    @desc~ yes~ The label to display to the user.
    @encoding~ no~ Used in case of text binary input to specify the character encoding to consider. The default is "UTF-8".
    @fileName~ no~ The name of a variable or output map property that will record the file name for the uploaded file (in case of a file upload). This can be set as a fixed ``string`` or be defined dynamically via :ref:`variable reference<test-case-referring-to-variables>`.
    @inputType~ no~ The input control type to use when prompting users for the relevant value. By default a value of ``TEXT`` is assumed unless this input is mapped to an existing ``binary`` variable.
    @mimeType~ no~ In case the ``inputType`` is set as ``CODE`` (i.e. a code editor) this is the content's expected `mime type`_ (e.g. ``text/xml``), provided as a string or a variable reference, to be considered for presenting appropriate syntax highlighting.
    @multiple~ no~ A ``boolean`` value to determine whether the dropdown list (if the ``options`` attribute is defined) shall be a single or multiple selection list (default is "false" for single selection).
    @name~ no~ In case of ``request`` elements this name is the key to be used for the map entry to hold the provided data.
    @optionLabels~ no~ Used as the labels for the option values (comma-separated values, a reference to a string variable of comma-separated values, or a reference to a list variable of strings). If provided the number of values needs to match the options. If not provided the option values are used.
    @options~ no~ Used to render a dropdown list by providing the option values to consider (comma-separated values, a reference to a string variable of comma-separated values, or a reference to a list variable of strings).
    @required~ no~ Whether or not this input is mandatory (by default "false"). When set to "true" the relevant input control will appear as mandatory and the interaction will not be able to be completed unless a value is provided. Note that this flag can also be set as a :ref:`variable reference <test-case-referring-to-variables>`.
    @report~ no~ Whether or not this value will be included in the presentation of the test step's report (by default "true"). When set to "false" the requested value will be stored in the test session context but not displayed in the step's report.
    @size~ no~ A number defining the size of the input control in terms of its height. This is considered for the rows of ``MULTILINE_TEXT`` inputs, the pixels of ``CODE`` editors, and the presented items of ``SELECT_MULTIPLE`` selections.
    @with~ no~ The ID of the actor this interaction refers to. If not specified this is taken from the ``interact`` parent element (which itself defaults to the test case's SUT actor). Within scriptlets this can also be a :ref:`variable reference<scriptlets_dynamic_references>`.

Both instructions and requests can be included in the same ``interact`` step to display and/or request multiple sets of
information in one go. The sequence with which these elements are defined determine also their display sequence in the
``interact`` step's popup. The attributes for the ``interact``, ``instruct`` and ``request`` elements determine the precise
behaviour in presenting and requesting data to and from the user. Detailed information on their possible options as well as several
examples are provided in the sections that follow.

.. _tdl-step-interact_instruct_presentation:

Displaying data using instruct elements
+++++++++++++++++++++++++++++++++++++++

The ``instruct`` element is used to present data from the test session to the user. The presented data is labelled using
the ``desc`` attribute's value, whereas the data itself is determined by evaluating the element's content as an :ref:`expression <test-case-expressions>`,
ranging from a **constant value**, to a :ref:`variable reference <test-case-referring-to-variables>`, to a complete **XPath expression** to process. In case of
simple messages, you can also define the ``instruct`` element as empty in which case only the ``desc`` value is presented.

.. code-block:: xml

    <interact desc="User instructions">
        <!-- Display a simple inline message -->
        <instruct desc="Send the message to continue"/>
        <!-- Display a labelled value -->
        <instruct desc="Message type to use:">"Request message"</instruct>
        <!-- Display a labelled file for download -->
        <instruct desc="Message to send:">$file</instruct>
    </interact>

When the ``instruct`` element's content is defined, the way in which it is presented depends on the :ref:`type <test-case-types>`
of the resulting expression, which can also be forced through the ``type`` attribute. Depending on the value's type (inferred or explicitly set),
it is presented as follows:

* For a ``binary``, ``object`` or ``schema`` type, controls will be displayed allowing to **download** the value as a file
  and to view it in a **code editor** (if text-based).
* For all other types, the value is displayed **inline as text**. This is also the default approach if the value's type was not
  set and could not be inferred.

Instruction elements are considered as informational features and by default only figure in the step's presentation on
the user interface when the step is being executed. If you want to ensure certain ``instruct`` elements are presented
also for completed steps, both on the user interface and produced step reports, you can set their ``report`` attribute
to true.

When displaying the **value inline** it could be the case that the text is too long. In this case the user will instead
be provided with controls to download it as a file or open it in a code editor (as in the case of e.g. binary content).
You can override this behaviour by setting the ``forceDisplay`` attribute to true, which will result in an inline display
regardless of the value's size. In addition, you can set the ``showControls`` attribute to false, to hide the display of
user interface copy and view controls.

When conveying feedback, status or instructions, you may also find useful the ``level`` attribute. This is used to adapt
the display of ``instruct`` elements to convey severity by means of colour highlighting. It supports values ``ERROR``,
``WARNING``, ``INFO``, ``SUCCESS`` and ``NONE`` (the default), that result in the instruction being presented with a
coloured background (e.g. green background for ``SUCCESS``). The value for this attribute can also be determined at
runtime by providing it as a :ref:`variable reference <test-case-referring-to-variables>`. Using the ``level`` attribute
to display stylised messages, can be optionally complemented by the ``forceDisplay`` and ``showControls`` attributes to
ensure that the message is presented without truncation and without superfluous UI controls.

.. code-block:: xml

    <!--
        Display a blue information message ensuring that no UI controls are displayed.
    -->
    <interact inputTitle="Next step" desc="Show instructions">
        <instruct level="INFO" forceDisplay="true" showControls="false">"Please send a request referring to identifier " || $expectedIdentifier</instruct>
    </interact>

As a complement to the ``type``, you can also specify the ``mimeType`` attribute. This is meaningful for binary or large text content
as it serves two purposes: it allows you to specify the content type and **file extension** to use when the content is downloaded as a file, and it
provides a hint for appropriate **syntax highlighting** when displaying the content in a code editor. Considering file
downloads, you can also set the ``name`` attribute to specify the name of the downloaded file.

It is important to note that the ``mimeType`` attribute has no effect when the value is presented inline. In fact, an inline
presentation is always as simple text, with no additional formatting or highlighting. In case you are looking for an inline
display of something more elaborate (e.g. a rich text message styled as HTML content), you should consider defining this
as :ref:`documentation for the step <tdl-steps-common-documentation>` using a ``documentation`` element within the ``interact``
step.

In case the ``interact`` step only defines ``instruct`` elements you can also use the ``blocking`` attribute to specify that 
it **should not block the test session execution**. This could be interesting in case you want to allow subsequent test steps to
proceed without necessarily waiting for the user to complete the interaction. An example where this could be particularly
useful is in case you are using the ``interact`` step as an information popup, before expecting a message to be sent to the 
test engine through a :ref:`receive step <tdl-step-receive>`. In such a scenario, users may proceed with
sending the expected message before closing the interaction, which depending on the :ref:`handler implementation <handlers>`, might
result in the test engine ignoring the (prematurely sent) message.

The configuration to avoid this issue would be as follows:

.. code-block:: xml

    <!-- 
        Use an interact step to show an information popup informing the user of the expected identifier
        to use in a request. Setting this as not "blocking" will result in the popup being displayed
        but the test session proceeding to run the next "receive" step.

        We also set the interaction as "hidden" as we don't need this to be displayed on the test
        execution diagram.
    -->
    <log>"Expecting GET call quoting identifier " || $expectedIdentifier</log>
    <interact inputTitle="Instructions" desc="Show instructions" blocking="false" hidden="true">
        <instruct>"Please send a request referring to identifier " || $expectedIdentifier</instruct>
    </interact>
    <receive id="receive" desc="Receive message" handler="HttpMessagingV2">
        <input name="uriExtension">"/message/" || $expectedIdentifier</input>
    </receive>

Using the ``blocking`` attribute is optional, and if not specified defaults to "true". You can also set this dynamically 
as a :ref:`variable reference <test-case-referring-to-variables>` in case the blocking behaviour depends on configuration
or your current state. Note finally that in case the ``interact`` step includes ``request`` elements for user input, the
``blocking`` attribute is ignored as the interaction is always blocking.

.. _tdl-step-interact_form_inputs:

Requesting inputs using request elements
++++++++++++++++++++++++++++++++++++++++

The ``request`` element is used to request input from the user. The element's definition determines the two important aspects
related to such input:

* The way in which the provided input will be recorded for subsequent use.
* The form control used by the user to provide the input.

Regarding the first point, how to **record the provided input**, you can either set it as the value of an existing variable,
or record it in a map corresponding to the ``interact`` step's output. To set an existing variable you specify the ``request``
element's content as a :ref:`variable reference <test-case-referring-to-variables>`, referring to the variable that will
receive the value. In this case you need to ensure that the referenced variable already exists, for example through an earlier
:ref:`assign step <tdl-step-assign>` or a :ref:`variable declaration <test-case-variables>`.

.. code-block:: xml

    <assign to="aValue">"Initial value"</assign>
    ...
    <interact id="data" desc="Provide input">
        <!-- Stored as $aValue.  -->
        <request desc="Enter a text value:">$aValue</request>
    </interact>
    <!-- Log the provided value. -->
    <log>$aValue</log>

The alternative to this approach is to store received inputs in a map corresponding to the ``interact`` step's output. In
this case you leave the ``request`` element empty, specifying instead the ``name`` attribute that will be used as the value's
key in the step's output map. The output map added to the test session context is named after the ``interact`` step's ``id``.

.. code-block:: xml

    <interact id="data" desc="Provide input">
        <!-- Stored as $data{aValue}.  -->
        <request desc="Enter a text value:" name="aValue"/>
    </interact>
    <!-- Log the provided value. -->
    <log>$data{aValue}</log>

This second approach is considered the **better practice** as it is typically simpler to use. Firstly, there is no need to predefine
variables to receive inputs, and secondly, it becomes simpler to manage ``interact`` steps resulting in multiple inputs.

.. code-block:: xml

    <interact id="data" desc="Provide inputs">
        <request desc="Invoice number:" name="invoiceNumber"/>
        <request desc="Seller VAT number:" name="sellerVat"/>
        <request desc="Buyer VAT number:" name="buyerVAT"/>
    </interact>
    <!-- Log the provided values. -->
    <log>"Invoice "|| $data{invoiceNumber} || " sent from " || $data{sellerVat} || " to " || $data{buyerVAT}</log>

The second key aspect of a ``request`` element is configuring the **form control** to present to the user. The label for the
input is defined through the ``desc`` attribute, whereas the type of input presented is determined by the ``inputType`` attribute
that supports the following values:

.. csv-table::
    :delim: ~
    :header: "Input type", "Description"

    ``TEXT`` ~ A simple text field (the default if not specified).
    ``MULTILINE_TEXT`` ~ A textarea supporting input of multiple lines. You can also set in this case the ``size`` attribute to define the control's height (in rows).
    ``SECRET`` ~ A control to add a secret value such as a password.
    ``UPLOAD`` ~ A file upload control. You can also set in this case the ``fileName`` attribute to record the name of the uploaded file (the ``fileName`` value being the key to use in the step's output map). The ``accept`` attribute can also be used to limit the types of accepted files.
    ``CODE`` ~ A code editor. To complement this you can also specify the ``mimeType`` attribute with a `mime type`_ (e.g. ``text/xml``) to have appropriate syntax highlighting. The ``size`` attribute can also be used to define the editor's height (in pixels).
    ``SELECT_SINGLE`` ~ A single-select dropdown list, specifying the options via the ``options`` and ``optionLabels`` attributes.
    ``SELECT_MULTIPLE`` ~ A multi-select dropdown list using similarly the ``options`` and ``optionLabels`` attributes. You can also set in this case the ``size`` attribute to define the number of presented items.

.. note::
    The value received from a ``SELECT_MULTIPLE`` input will be a comma-separated string in which the individual parts match the selected values.

The following example produces a popup presenting a text field, textarea and file upload.

.. code-block:: xml

    <interact id="data" desc="Provide inputs">
        <request desc="Invoice number:" name="invoiceNumber"/>
        <request desc="Invoice comment:" name="invoiceComment" inputType="MULTILINE_TEXT"/>
        <request desc="Invoice file:" name="invoiceFile" fileName="invoiceFileName" inputType="UPLOAD"/>
    </interact>
    <!-- Log the provided values. -->
    <log>"Invoice "|| $data{invoiceNumber} || " uploaded as " || $data{invoiceFileName} || " with comment: " || $data{invoiceComment}</log>

In the example above, the types of uploaded files could also be controlled by using the ``accept`` attribute to specify the accepted file
types as `mime types <https://developer.mozilla.org/en-US/docs/Web/HTTP/Basics_of_HTTP/MIME_types/Common_types>`__. This can
be specified inline as a comma-separated string or provided dynamically by referencing a ``string`` or ``list`` variable.

.. code-block:: xml

    <assign to="acceptedImageTypes" append="true">"image/png"</assign>
    <assign to="acceptedImageTypes" append="true">"image/jpeg"</assign>
    <interact id="data" desc="Provide inputs">
        <request desc="Evidence screenshot" name="screenshot" fileName="evidenceFile" inputType="UPLOAD" accept="$acceptedImageTypes"/>
        <request desc="Invoice file" name="invoiceFile" fileName="invoiceFileName" inputType="UPLOAD" accept="application/pdf"/>
    </interact>

For certain input types, notably ``MULTILINE_TEXT``, ``CODE`` and ``SELECT_MULTIPLE``, you can specify the ``size`` attribute
to control the height of the resulting control. In all cases this needs to be a positive integer, provided inline or via
:ref:`variable reference <test-case-referring-to-variables>`. For ``MULTILINE_TEST`` inputs this is interpreted as the
number of rows displayed by default, for ``CODE`` inputs as the height in pixels of the editor, and for ``SELECT_MULTIPLE``
as the number of visible items in the selection list.

.. code-block:: xml

    <interact id="data" desc="Provide inputs">
        <request desc="Invoice comment:" name="invoiceComment" inputType="MULTILINE_TEXT" size="5"/>
        <request desc="Invoice content:" name="invoiceContent" inputType="CODE" size="500"/>
    </interact>

Where is is meaningful to present a default value in an input control you can use the ``default`` attribute. With this
you can specify a default value inline, or provide a :ref:`variable reference <test-case-referring-to-variables>` pointing
to a :ref:`configuration value <test-case-configuration>` or another value from the test session context.

.. code-block:: xml

    <interact id="data" desc="Provide inputs">
        <request desc="The name of your software" name="softwareName" inputType="TEXT" default="$SYSTEM{shortName}" required="true"/>
    </interact>

Prior to GITB TDL version 1.14.0, the way to determine the input control to use was the ``contentType`` attribute. Although less expressive, this approach is
still supported as follows:

* Specifying "BASE64" results in a file upload presented to the user.
* Specifying "STRING" (the default) or "URI" results in a simple text input. Note that only "STRING" can be used in case the request is defined as a dropdown list (i.e. the ``options`` attribute is defined).

It is interesting to note that any available context information is always considered to reduce the configuration you need to provide. For example, if for a ``request``
you are referencing an already defined ``binary`` variable, you can skip the ``inputType`` or ``contentType`` definitions as this will anyway result in a file upload.
Similarly, if for a ``request`` you define ``options`` and the ``multiple`` attribute, you don't need to define the ``inputType`` as well as this is considered to be
by default ``SELECT_MULTIPLE``.

Besides defining the type of input control to display, you can also specify whether the input should be presented as ``required``. This is achieved through the
similarly named attribute that is set either with a boolean value or with a :ref:`variable reference <test-case-referring-to-variables>`. The latter approach
might be interesting in case whether or not an input is mandatory depends on your configuration or test session state. Note that specifying the ``required``
attribute is optional, with a value of "false" (non-mandatory) being considered as the default.

.. code-block:: xml

    <interact id="data" desc="Provide inputs">
        <!-- The invoice number is required. -->
        <request desc="Invoice number:" name="invoiceNumber" required="true"/>
        <!-- The invoice comment is optional. -->
        <request desc="Invoice comment:" name="invoiceComment" inputType="MULTILINE_TEXT"/>
        <!-- The invoice file is required. -->
        <request desc="Invoice file:" name="invoiceFile" required="true" inputType="UPLOAD"/>
    </interact>

An ``interact`` step containing one or more ``request`` elements set as ``required`` will not be able to complete until at least the mandatory inputs are provided.
Presentation-wise, mandatory inputs are highlighted as such on the Test Bed's user interface with error messages displayed if the user attempts to complete the
interaction without specifying them. Note that it is always possible to minimise the interaction and completed it at a later point. If the interaction is
:ref:`delegated to an external service <tdl-step-interact_handler>`, ``required`` inputs must be returned otherwise the interaction will fail.

.. _tdl-step-interact_admin_interactions:

Interactions for administrators
+++++++++++++++++++++++++++++++

Interactions are by default presented to the tester, but can also be reserved for an administrator by setting the ``interact`` element's ``admin`` flag to true.
This could be useful in case you need to pause a test session while an administrator makes a manual verification, or to make a manual check on user-provided
evidence data that cannot be automatically verified.

In case the administrator needs to input internal information, you can fine-tune what gets reported to users. You can skip
the reporting of specific information by setting the ``report`` flag on the relevant ``request`` elements to false. You can
even :ref:`hide the interaction step <tdl-steps-common-hidesteps>` altogether by setting ``hidden`` to true.

.. code-block:: xml

    <interact id="adminData" desc="Confirm results" admin="true">
        <request desc="Comments" inputType="MULTILINE_TEXT" name="comments"/>
        <!-- The "code" input will be recorded but not added to reports. -->
        <request desc="Internal code" name="code" report="false"/>
    </interact>
    <log>$adminData{code}</log>

.. _tdl-step-interact_customise_display:

Customising the interact step's display
+++++++++++++++++++++++++++++++++++++++

The ``interact`` step's presentation can be customised using the element's attributes. Besides :ref:`commons options <tdl-steps-common>`
applicable to all TDL steps, you can adapt the step's description (``desc``) and ``title`` for the displayed step's boundary, as well as
provide a custom ``inputTitle`` for the popup used to present the step's instructions and form controls.

.. code-block:: xml

    <interact desc="User instructions" inputTitle="Instructions">
        <instruct desc="Send the message to continue"/>
    </interact>

For an ``interaction`` step that is not a key part of the test case, for example a popup to guide the user in next steps,
a typical configuration is to set it as :ref:`hidden <tdl-steps-common-hidesteps>`. This ensures the interaction is
executed but will not be displayed on the test execution diagram or included in test case reports. A lighter alternative
is to set the step as ``collapsed`` in which case it will be included in the execution diagram and reports, but will
be presented as initially collapsed. This approach could be interesting for example if the step in question requests inputs
that you still want to make available for later review by expanding the step's display and selecting the step's report.

.. code-block:: xml

    <interact desc="User instructions" inputTitle="Instructions" collapsed="true">
        <instruct desc="Send the message to continue"/>
    </interact>

.. _tdl-step-interact_timeout:

Background execution and timeouts
+++++++++++++++++++++++++++++++++

When the ``interact`` step is executed the test execution pauses until the step is completed by the user. This is meaningful
when the relevant test is executed interactively but could be a point to consider if the test is executed in the background.
Moreover, even if executed interactively, you may want to prevent the test session from blocking indefinitely due to a simple information
popup that has no bearing on the overall test.

To avoid blocking test sessions and to better control how interactions are completed in background executions, the ``interact``
step can be set with a configurable ``timeout``. The value of this is a number corresponding to the milliseconds to
wait for until automatically completing the interaction. It can be set as a fixed value or as a :ref:`variable reference <test-case-referring-to-variables>`,
set dynamically or referring to a :ref:`configuration property <test-case-configuration>` (similar to the timeouts of :ref:`receive steps <tdl-step-receive>`).

.. note::

    Background test sessions with pending user interactions, can be still be completed by both users and administrators
    through the GITB Test Bed's user interface (under the **active sessions** in the
    `session history <https://www.itb.ec.europa.eu/docs/itb-ta/latest/validateTestSetup/index.html#active-test-sessions>`__
    and `session dashboard <https://www.itb.ec.europa.eu/docs/itb-ta/latest/sessionDashboard/index.html#active-test-sessions>`__ screens).

For a test session executing in the background, the test engine treats ``interact`` steps in the following way:

1. If the step contains only ``instruct`` elements and ``blocking`` is set (or resolves) to "false", the interaction is skipped.
2. If the step is set with a ``timeout``, the interaction remains pending until completed by the user or until the timeout expires.
3. If no ``timeout`` is set but the ``interact`` step is set as an :ref:`administrator interaction <tdl-step-interact_admin_interactions>`
   (i.e. ``admin`` is set to true), the test session will pause indefinitely until completed by an administrator.
4. In all other cases the step is completed automatically with a log warning, effectively skipping it and resulting in empty inputs (if requested).

.. code-block:: xml

    <!-- Assume the test session is executing in the background. -->

    <!-- This interaction is automatically skipped. -->
    <interact desc="User instructions">
        <instruct desc="Send the message to continue"/>
    </interact>

    <!-- This interaction will remain pending until manually completed or until the timeout elapses. -->
    <interact id="userData" desc="User input" timeout="$DOMAIN{userInteractionTimeout}">
        <request desc="Identifier to check" name="identifier"/>
    </interact>

    <!-- This interaction will remain pending until manually completed. -->
    <interact id="adminData" desc="Confirm results" admin="true">
        <request desc="Comments" inputType="MULTILINE_TEXT" name="comments"/>
    </interact>

    <!-- This interaction will remain pending until manually completed or until the timeout elapses. -->
    <interact id="adminData" desc="Confirm results" admin="true" timeout="$DOMAIN{adminInteractionTimeout}">
        <request desc="Comments" inputType="MULTILINE_TEXT" name="comments"/>
    </interact>

Generally speaking, it is a **good practice** to always define timeouts for ``interact`` steps as this gives you full control over
their execution. This becomes even more important if you are expecting manual user inputs, but still want your test cases to
support background execution. In this case you can even foresee a fallback solution for user input that was not provided,
checking the ``interact`` step's resulting value(s) and assigning defaults as needed.

.. code-block:: xml

    <log>"Requesting user input..."</log>
    <interact id="userData" desc="Request user input" timeout="3000">
        <request desc="Input value" name="inputValue"/>
    </interact>
    <!-- If the step was skipped the requested value will be empty. -->
    <assign to="userData{inputValue}">if ($userData{inputValue} = "") then "Default value" else $userData{inputValue}</assign>
    <log>"Value to use: " || $userData{inputValue}</log>

.. note::

    When executing test sessions through the GITB Test Bed's `REST API <https://www.itb.ec.europa.eu/docs/itb-ta/latest/api/index.html#start>`__
    you can provide input data as part of the service call. Such data could serve to replace inputs that would otherwise be
    requested via ``interact`` steps. Note that for such headless test sessions you may also want to :ref:`delegate mandatory interactions <tdl-step-interact_handler>`
    to an external service.

.. _tdl-step-interact_handler:

Delegating to external handler services
+++++++++++++++++++++++++++++++++++++++

The default implementation for an ``interact`` step is to display a **popup dialog** to the user to :ref:`present instructions <tdl-step-interact_instruct_presentation>`
and :ref:`request inputs <tdl-step-interact_form_inputs>`. You may want to replace this default behaviour with a different implementation
that will delegate the handling of the interaction to an external service. Scenarios where you would want to do this are:

* When launching tests via the Test Bed's `REST API <https://www.itb.ec.europa.eu/docs/itb-ta/latest/api/index.html#start>`__, in which case
  instead of blocking the test, you want to provide requested inputs via a different approach.
* When the interaction should involve **external solutions** such as sending emails and collecting inputs via another interface.

Delegating an ``interact`` to an alternate implementation requires using an **external custom service** that will handle the provision
of any requested inputs. This service needs to implement the `GITB messaging service API <https://www.itb.ec.europa.eu/docs/services/latest/messaging/index.html>`__,
the same way that a :ref:`receive step <tdl-step-receive>` would do to receive a message asynchronously. Even though using a messaging
service API for a user interaction may seem initially unintuitive, semantically both represent the same scenario: the test session pauses until
data is received. The specific messaging operation that needs to be implemented by the service to handle ``interact`` steps is the
`receive operation <https://www.itb.ec.europa.eu/docs/services/latest/messaging/index.html#receive>`__.

To delegate an ``interact`` step to an external service you use the ``handler`` attribute. This is set exactly as in the case of a :ref:`receive step <tdl-step-receive>`,
and would typically be set via a :ref:`domain configuration parameter <test-case-expressions-domain>`. You also have available a ``boolean``
flag named ``handlerEnabled`` that determines whether the handler service should be used. This can be set to fixed "true" or "false" values
("false" being the default), but can also be a :ref:`variable reference <test-case-referring-to-variables>` allowing this decision
to be made at runtime based on your configuration or runtime state.

To provide context to the handler service, the ``interact`` step also supports a ``handlerConfig`` child element that is used
to define the inputs to send to the service. With this you can pass values and state collected from previous test steps to help
the service complete the interaction. If any values were requested by the interaction via ``request`` elements, these can be provided
as **similarly named service outputs**. Regarding inputs and outputs you need to keep in mind the following:

* If a handler service  is delegated the interaction, the step's ``instruct`` and ``request`` elements are ignored. You must pass
  expected information as ``input`` elements under a ``handlerConfig`` block.
* Any outputs returned by the service that do not correspond to ``request`` elements are ignored.
* Any missing outputs for ``request`` elements set as ``required`` will cause the ``interact`` step to fail.

.. note::
    When delegating an interaction to a handler service that is expected to returned inputs, it is best to set the step's
    ``request`` elements with ``name`` values. This allows **explicit name-based matching** to returned service outputs.

The following example illustrates how an ``interact`` step can delegate its implementation to a handler service:

.. code-block:: xml

    <!-- 
        The handler attribute defines an external service (via configuration) to manage the interaction.

        If the handlerEnabled flag evaluates to "true" when the step is executed, the delegation will
        take place. Otherwise the handler is ignored and the default UI-based approach to complete
        the step will be used.
    -->
    <interact id="review" handlerEnabled="$delegateInteractions" handler="$DOMAIN{serviceAddress}" desc="Review test data">
        <request name="outcome" desc="Review outcome" inputType="SELECT_SINGLE" options="OK,NOK"/>
        <request name="comments" desc="Comments" inputType="MULTILINE_TEXT"/>
        <!-- 
            Provide inputs to the handler service as context for the review, loaded from prior test steps.
            These are ignored if we're not delegating to a handler service.
        -->
        <handlerConfig>
            <input name="receivedMessage">$message</input>
            <input name="expectedCode">$expectedCode</input>
        </handlerConfig>
    </interact>
    <log>"Outcome: " || $review{outcome}</log>
    <log>"Comments: " || $review{comments}</log>

Using a handler to delegate interactions can be a powerful mechanism when required interactions are not possible to complete manually. This
is typically the case for tests launched through the Test Bed's `REST API <https://www.itb.ec.europa.eu/docs/itb-ta/latest/api/index.html#start>`__.
To allow such tests to work both when launched interactively and via REST call, we can have the ``handlerEnabled`` flag check 
a :ref:`predefined test case variable <test-case-variables>` that is :ref:`set accordingly through the REST call's inputs <tdl-step-interact_api>`:

.. code-block:: xml

    <variables>
        <!-- 
            Set the delegateInteractions flag to "false" by default to cover manual execution.
            This will be overriden when to "true" when the test is started via the REST API.
        -->
        <var name="delegateInteractions" type="boolean">
            <value>false</value>
        </var>
    </variables>
    <steps>
        ...
        <!-- 
            Check the delegateInteractions flag to see whether we are delegating or not.
        -->
        <interact id="review" handlerEnabled="$delegateInteractions" handler="$DOMAIN{serviceAddress}" desc="Review test data">
            ...
        </interact>
        ...
    </steps>

.. note::
    If an ``interact`` step is optional you can also :ref:`skip it <tdl-steps-common-skipped>` using the ``skipped`` attribute.

.. _tdl-step-interact_api:

Interactions in REST API-initiated tests
++++++++++++++++++++++++++++++++++++++++

The Test Bed provides a rich `REST API <https://www.itb.ec.europa.eu/docs/itb-ta/latest/api/>`__ to, among other operations, 
`launch test sessions <https://www.itb.ec.europa.eu/docs/itb-ta/latest/api/index.html#start>`__ as part of automated processes. Doing
so presents a challenge for tests relying on user interactions, given that in this case interactions will likely not be possible.

In case tests are launched via REST API but can later be interacted with by users, the typical approach would be to
:ref:`use timeouts <tdl-step-interact_timeout>` so that users have enough time to provide inputs. In case however such
interactions are not possible, the main options to manage ``interact`` steps are:

* Setting ``skipped`` to "true" to :ref:`skip interactions <tdl-steps-common-skipped>` that are not mandatory (e.g. instruction popups).
* Setting a ``handler`` service and ``handlerEnabled`` to "true" to :ref:`delegate to an external service <tdl-step-interact_handler>`.

To accommodate the same test cases being used regardless of whether they were launched manually or via REST call, the best approach
is to set ``skipped`` and ``handlerEnabled`` as :ref:`references <test-case-referring-to-variables>` to :ref:`predefined test case variables <test-case-variables>`.
When defining such variables you would set them by default to what you need to enable manual execution, while expecting them to be 
overriden when the test is executed via REST call.

The following example illustrates how you could configure this:

.. code-block:: xml

    <variables>
        <!-- 
            Define the flags to control the interactions. These are set to false to reflect the
            default behavior when executed via the UI. They will be replaced by inputs when the
            test session is triggered via the REST API.
        -->
        <var name="skipInteractions" type="boolean">
            <value>false</value>
        </var>
        <var name="delegateInteractions" type="boolean">
            <value>false</value>
        </var>
    </variables>
    <steps>
        ...
        <interact id="review" skipped="$skipInteractions" handlerEnabled="$delegateInteractions" handler="$DOMAIN{serviceAddress}" desc="Review test data">
            <request name="outcome" desc="Review outcome" inputType="SELECT_SINGLE" options="OK,NOK"/>
            <request name="comments" desc="Comments" inputType="MULTILINE_TEXT"/>
            <!-- 
                Provide inputs to the handler service as context for the review, loaded from prior
                test steps. These are ignored if we're not delegating to a handler service.
            -->
            <handlerConfig>
                <input name="receivedMessage">$message</input>
                <input name="expectedCode">$expectedCode</input>
            </handlerConfig>
        </interact>
        <log>"Outcome: " || $review{outcome}</log>
        <log>"Comments: " || $review{comments}</log>
        ...
    </steps>

When this test case is launched via the UI, both ``skipInteractions`` and ``delegateInteractions`` flags are set to their default "false"
values, resulting in an interaction that is not skipped and not delegated to the external service. When executed via the REST API however,
we can override the flags via inputs provided to the `start operation <https://www.itb.ec.europa.eu/docs/itb-ta/latest/api/index.html#start>`__
as follows:

.. code-block:: json

    {
        "system": "7039...E9B0",
        "actor": "7704...09E6",
        "testCase": [ "testCase1" ],
        "inputMapping": [
            { "input": { "name": "skipInteractions", "value": "false" } },
            { "input": { "name": "delegateInteractions", "value": "true" } }
        ]
    }

Not all REST-initiated tests that trigger interactions must have them skipped or delegated. You may have certain interactions
that you still expect to **complete manually**, such as manual :ref:`administrator verification <tdl-step-interact_admin_interactions>`
of collected evidences. In such cases you should define :ref:`interaction timeouts <tdl-step-interact_timeout>`
to set the maximum time tests will pause for waiting for such inputs to be provided.

.. _tdl-step-interact_examples:

Example interact steps
++++++++++++++++++++++

The following examples illustrate user interactions presenting instructions and also requesting information:

.. code-block:: xml

    <interact desc="Some information and inputs">
        <!-- type="string" omitted as default. Displays the text as a message to the user. -->
        <instruct desc="This is a simple message"/>
        <instruct desc="A text value:">"A text value " || $aTextValue</instruct>
        <!-- Present a download button and XML editor for file "schema.xsd" (not specifying a name would produce a "downloadedFile" file). -->
        <instruct name="schema.xsd" desc="A file to download:" mimeType="text/xml">$schemaFile</instruct>
        <!-- Present an instruction forcing an inline display rather than using an editor. -->
        <instruct desc="Message:" forceDisplay="true">"A long text such as detailed instructions, that would otherwise be displayed in an editor rather than follow an inline display."</instruct>
        <!-- Present a text input field storing the result in variable aStringInputValue. -->
        <request desc="Enter a text value:" inputType="TEXT">$aStringInputValue</request>
        <!-- Present a text area input storing the result in variable aLongStringInputValue. -->
        <request desc="Enter a long text value:" inputType="MULTILINE_TEXT">$aLongStringInputValue</request>
        <!-- Present a secret value input storing the result in variable aSecretValue. -->
        <request desc="Enter a secret value:" inputType="SECRET">$aSecretValue</request>
        <!-- Present a single selection dropdown list storing the result in variable aSelectedInputValue. -->
        <request desc="Enter a text value:" options="v1, v2" optionLabels="Value 1, Value 2">$aSelectedInputValue</request>
        <!-- Present a file upload storing the result in variable aBinaryVariable and its file name in aBinaryVariableFileName. -->
        <request desc="Upload a file:" fileName="aBinaryVariableFileName">$aBinaryVariable</request>
    </interact>

    <!-- Example storing all provided input in a map. This uses the "id" and "name" attributes. -->
    <interact id="userInput" desc="Some information and inputs">
        <!-- Present a text input field storing the result in variable userInput{text} (a type of "string" is assumed as the default). -->
        <request name="text" desc="Enter a text value:"/>
        <!-- Present a code editor for XML content, storing the result in variable userInput{xml} -->
        <request name="xml" desc="Enter XML content:" inputType="CODE" mimeType="text/xml"/>
        <!-- Present a code editor for JSON content, storing the result in variable userInput{json} -->
        <request name="json" desc="Enter JSON content:" inputType="CODE" mimeType="application/json"/>
        <!-- Present a file upload storing the result in variable userInput{file}. -->
        <request name="file" desc="Upload a file:" type="binary"/>
        <!-- Equivalent to the above but using the inputType. In addition, the file name is recorded and can be retrieved as userInput{anotherFileName}. -->
        <request name="anotherFile" fileName="anotherFileName" desc="Upload another file:" inputType="UPLOAD"/>
    </interact>

To better illustrate how dropdown selections can be defined, the following code sample presents the different ways to define them:

.. code-block:: xml

    <!-- Assign options and labels (you may predefine variables or create them on the fly as follows) -->
    <assign to="input3_options">"v1, v2, v3"</assign>
    <assign to="input3_labels">"Value 1, Value 2, Value 3"</assign>
    <assign to="input4_options" append="true">"x1"</assign>
    <assign to="input4_options" append="true">"x2"</assign>
    <assign to="input4_options" append="true">"x3"</assign>
    <assign to="input4_labels" append="true">"VAL 1"</assign>
    <assign to="input4_labels" append="true">"VAL 2"</assign>
    <assign to="input4_labels" append="true">"VAL 3"</assign>

    <interact id="data" desc="Enter data">
        <!-- Single selection with options provided in the attribute values (stored as data{input1}). -->
        <request desc="Select one" options="o1, o2, o3" optionLabels="Option 1, Option 2, Option 3" name="input1"/>
        <!-- Multiple selection with options provided in the attribute values (stored as data{input2}). -->
        <request desc="Select multiple" options="o1, o2, o3" optionLabels="Option 1, Option 2, Option 3" multiple="true" name="input2"/>
        <!-- Single selection with options provided by referring to string variables (stored as data{input3}). -->
        <request desc="Select one (use string reference)" options="$input3_options" optionLabels="$input3_labels" name="input3"/>
        <!-- Single selection with options provided by referring to list variables (stored as data{input4}). -->
        <request desc="Select one (use list reference)" options="$input4_options" optionLabels="$input4_labels" name="input4"/>
    </interact>

Finally the following code sample illustrates some of the more advanced interaction features, considering an information step to the tester
followed by an administrator-level verification:

.. code-block:: xml

    <!--
        Display an information prompt to the tester, closing it automatically after 10 seconds. The timeout could also be set
        via configuration using for example a domain parameter ($DOMAIN{infoTimeout}).

        The step is hidden as it is not interesting to include in the graphical execution diagram, and the message is set as being
        forced to display as-is, to avoid it being presented in a code editor if it exceeds the inline display limit.
    -->
    <interact hidden="true" desc="Test information" timeout="10000">
        <instruct desc="Message:" forceDisplay="true">"Please wait until the administrator validates your results."</instruct>
    </interact>

    <!--
        Display an interaction prompt to an administrator to provide inputs.

        The administrator is also expected to provide an internal code that will not be presented in the step's report but can
        be subsequently used in other test steps.
    -->
    <interact id="adminData" desc="Confirm results" admin="true">
        <request desc="Comments" inputType="MULTILINE_TEXT" name="comments"/>
        <!-- The "code" input will be recorded but not added to reports. -->
        <request desc="Internal code" name="code" report="false"/>
    </interact>

    <log>$adminData{code}</log>
