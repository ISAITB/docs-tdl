Used to validate an XML document by matching it against a provided reference document or template.

.. csv-table::
    :header: "Input name", "Required?", "Type", "Description"

    ``ignoredPaths``, No, ``list[string]``, An optional list of paths provided as XPath expressions identifying sections of the XML to ignore.
    ``template``, Yes, ``object``, The XML file to consider as the validation's template.
    ``xml``, Yes, ``object``, The XML file to validate.

The ``template`` input can either be a complete XML reference document that you want the ``xml`` to match, or a match template.
The matching process takes place by normalising whitespace, ignoring comments and tolerating naming differences in namespace prefixes. In addition,
texts of elements or attributes in the provided ``template`` can be specified with the special value ``?``. This means that any value will be allowed
for this element or attribute and will be ignored as part of the matching (e.g. to ignore random tokens, timestamps, or texts with no expected value).

In case you want to ignore complete XML sections you may use the ``ignoredPaths`` attribute. This allows you to define one or more paths that identify
elements that will, themselves and for all children, be ignored. For each provided path the following constraints apply:

* It must be formed as a namespace-aware XPath expression considering the namespace prefixes of the provided ``template``.
* It must identify a specific element, rather than a set of elements, a text node or an attribute.
* It must be a simple element-based path with no functions, selectors or wildcards.

The following example illustrates how this validator can be used:

.. code-block:: xml

    <!--
        Validate an XML file based on a provided template.
    -->
    <verify handler="XmlMatchValidator" desc="Validate content">
        <input name="xml">$docToValidate</input>
        <input name="template">$templateFile</input>
    </verify>
    <!--
        Another validation that also defines a set of paths to ignore.
        Variable "pathsToSkip" is of type list[string].
    -->
    <assign to="pathsToSkip" append="true">"/x:Invoice/x:BillingInformation/y:Comments"</assign>
    <verify handler="XmlMatchValidator" desc="Validate content">
        <input name="xml">$docToValidate</input>
        <input name="template">$templateFile</input>
        <input name="ignoredPaths">$pathsToSkip</input>
    </verify>