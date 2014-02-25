Clean up field class structure:
----

- `ElementsField`: holds the raw list returned by `xpath()`; IF for all elements `el == str(el)` (i.e. selector ends in `text()`) then maps `clean_ascii` to them
    - `ElementField`: holds the first element of the super's list value IF it `hasattr('xpath')`, ELSE holds the entire list
        - `StringsField`: extracts the cleaned `text()`s of super's value IF it `hasattr('xpath')`, ELSE holds super's value IF it's a list and for all its elements `el == str(el)`, ELSE `ParseError`
            - `TextField`: `separator.join(super.value)`
                - `IntField`: `int(super.value)` with default
                - `FloatField`: `float(super.value)` with default
                - `DateField`: converts super's value to `date`
                - `DateTimeField`: convers super's value to `datetime`
                - `StructuredTextField`: noop, declares intention to write apply a custom parser that extracts info from the element's text
        - `URLField`: extracts super's href if `hasattr('attrib')` ELSE considers href was already selected by the xpath and holds it as a string, optionally urljoins it to some basepath
        - `DictField`: given a structure template as a dict `{key: <subclass of ElementsField>}`, recursively extracts each value field from the super's element
    - `ListField`: given a value param of type `<subclass of ElementsField>`, it extracts this field from each element in super's value
        - `StructuredListField`: convenience class for creating a list of `DictField('.', structure=...)`


#### Examples:

Extract urls from the texts of list items:

    ListField('//li', value=URLField('./text()'))

Extract structured info from list items:

    ListField(
        '//li',
        item=DictField(
            '.',
            {
                'name': TextField('.//span'),
                'url': URLField('.//a'),
                'homepage': TextField('.//a'),
            },
            key_name='name'
        )
    )

Apply custom parser to a sub-field. Due to limitations in Python decorator expressions we need to use the old fashioned way of applying decorators:

    # consider a document with attribute 'info' set to the ListField above

    def _uppercase_homepage(value, *args, **kwargs):
        return value.upper()
    uppercase_homepage = info.item.structure['homepage'].parser()(_uppercase_homepage)
