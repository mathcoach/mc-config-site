**********************************
How Configuration Files Are Parsed
**********************************


The class ``ClasspathBasedConfiguration`` accepts a properties-file as configuration.
It utilizes ``java.util.Properties.load(InputStream)`` for parsing. 
Consequently, any syntactically valid properties file will also be a valid configuration file for this class.


Whitespaces handling
====================

Leading and trailing whitespaces in configuration values are automatically removed. 
This behavior aligns with our requirements by preventing ambiguous values and helping users quickly identify errors in their configuration files.
In this example

.. code-block:: properties

    weird-config = \t

the value of ``weird-config`` is the empty string.


If you explicitly need a configuration value consisting solely of whitespaces, you must specify it precisely. 
For example, your app must parse a CSV file, 
where column separators can be a space, a tab, or another visible character (e.g. ``,`` or ``;``) for column separator, 
you can structure your configuration as follows:

.. code-block:: properties

    # If set to true, use space for the column separator 
    # and ignore other CSV separator configurations.
    space_as_separator = false

    # if space_as_separator not set|false and this set to true , use tab for column separator,
    # ignore other configuration for CSV column separator
    tab_as_separator = false

    # if both space_as_separator and tab_as_separator are not set or are parsed to false,
    # use the value of this configuration for CSV column separator
    separator_char = ;

Naturally, you will need to implement code to evaluate these configuration parameters. 
The ``EnvConfiguration`` class provides all configured parameters,
which you should check in your desired order:


.. code-block:: java


    public static final String SPACE_SEPARATOR_CONFIG = "space_as_separator";
    public static final String TAB_SEPARATOR_CONFIG = "tab_as_separator";
    public static final String SEPARATOR_CHAR_CONFIG = "separator_char";

    public static final char DEFAULT_SEPARATOR = ';';

    char getCsvColumnSeparator(EnvConfiguration config) {

        BiFunction<String,String,Boolean> parseBooleanConfig = (param, value) ->  Boolean.parseBoolean(value);

        if( config.getConfigValue(SPACE_SEPARATOR_CONFIG, parseBooleanConfig ) ) {
            return ' ';
        }
        if( config.getConfigValue(TAB_SEPARATOR_CONFIG, parseBooleanConfig ) ) {
            return '\t';
        }
        return config.getConfigValue(SEPARATOR_CHAR_CONFIG, (param, value) -> {
            try {
                return value.charAt(0);
            } catch(StringIndexOutOfBoundsException ex) {
                // in the case, that the configuration is an empty string
                return DEFAULT_SEPARATOR;
            }
        });
    }



Substitution
============

The two annotations ``${PWD}`` / ``$PWD`` and ``${HOME}`` / ``$HOME`` are resolved to the Working directory of the Java Process and the Home directory of the Java Process Owner respectively. 
Therefore, you shoud avoid using `PWD` and `HOME` as your own configuration parameter keys. 
The mechanism for resolving these annotations relies on Java's system environment properties ``user.home`` and ``user.dir``. 
Thus, it should function on platforms supported by Java.
*Please note: This library has only been tested with Linux.*


A parameter can reference another parameter within its value, 
and the embedded parameter will also be resolved to its corresponding value.
For example:

.. code-block:: properties

    myapp.base = $HOME/myapp
    myapp.data = ${myapp.base}/data
    myapp.conf = ${myapp.base}/conf

Circular references, whether direct or indirect, are not permitted and will result in an exception being thrown.