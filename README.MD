# robotframework-datadriver

 DataDriver is a Data-Driven Testing library for Robot Framework.  
 This document explains how to use the DataDriver library listener. For information about installation, support, and more, please visit the [project pages](https://github.com/Snooz82/robotframework-datadriver). 
 For more information about Robot Framework, see http://robotframework.org.  
 
 DataDriver is used/imported as Library but does not provide keywords which can be used in a test. DataDriver uses the Listener Interface Version 3 to manipulate the test cases and creates new test cases based on a Data-File that contains the data for Data-Driven Testing. 
 These data file may be .csv , .xls or .xlsx files.
 
 Data Driver is also able to cooperate with Microsoft PICT. An Open Source Windows tool for data combination testing.
 Pict is able to generate data combinations based on textual model definitions.
 https://github.com/Microsoft/pict
 

## Installation

If you already have Python >= 3.6 with pip installed, you can simply run:

`pip install --upgrade robotframework-datadriver`

or if you have Python 2 and 3 installed in parallel you may use 

`pip3 install --upgrade robotframework-datadriver`



## Table of contents

 - [What DataDriver does](#WhatDataDriverdoes)
 - [How DataDriver works](#HowDataDriverworks)
 - [Usage](#Usage)
 - [Structure of test suite](#Structureoftestsuite)
 - [Structure of data file](#Structureofdatafile)
 - [Data Sources](#DataSources)
 - [Encoding and CSV Dialect](#EncodingandCSVDialect)
  

## What DataDriver does
<a name="WhatDataDriverdoes"></a>

DataDriver is an alternative approach to create Data-Driven Tests with Robot Framework. DataDriver creates multiple test cases based on a test template and data content of a csv or Excel file. All created tests share the same test sequence (keywords) and differ in the test data. Because these tests are created on runtime only the template has to be specified within the robot test specification and the used data are specified in an external data file.  

DataDriver gives an alternative to the build in data driven approach like:

    *** Settings ***
    Resource    login_resources.robot
    
    Suite Setup    Open my Browser
    Suite Teardown    Close Browsers    
    Test Setup      Open Login Page
    Test Template    Invalid login
    
    
    *** Test Cases ***       User        Passwort
    Right user empty pass    demo        ${EMPTY}
    Right user wrong pass    demo        FooBar
        
    Empty user right pass    ${EMPTY}    mode
    Empty user empty pass    ${EMPTY}    ${EMPTY}
    Empty user wrong pass    ${EMPTY}    FooBar
        
    Wrong user right pass    FooBar      mode
    Wrong user empty pass    FooBar      ${EMPTY}
    Wrong user wrong pass    FooBar      FooBar
            
    *** Keywords ***
    Invalid login
        [Arguments]    ${username}    ${password}
        Input username    ${username}
        Input pwd    ${password}
        click login button
        Error page should be visible

This inbuild approach is fine for a hand full of data and a hand full of test cases.
If you have generated or calculated data and specially if you have a variable amount of test case / combinations these robot files becom quite a pain. 
With datadriver you may write the same test case syntax but only once and deliver the data from en external data file.

One of the rare reasons when Microsoft&reg; Excel or  LibreOffice Calc may be used in testing... ;-)

[See example test suite](#example-suite)

[See example csv table](#example-csv)

## How DataDriver works
<a name="HowDataDriverworks"></a>

 When the DataDriver is used in a test suite it will be activated before the test suite starts. It uses the Listener Interface Version 3 of Robot Framework to read and modify the test specification objects. After activation it searches for the `Test Template` -Keyword to analyze the `[Arguments]` it has. As a second step, it loads the data from the specified CSV file. Based on the `Test Template` -Keyword, DataDriver creates as much test cases as lines are in the CSV file. As values for the arguments of the `Test Template` -Keyword it reads values from the column of the CSV file with the matching name of the `[Arguments]`. For each line of the CSV data table, one test case will be created. It is also possible to specify test case names, tags and documentation for each test case in the specific test suite related CSV file.  
 

## Usage
<a name="Usage"></a>

 Data Driver is a "Listener" but should not be set as a global listener as command line option of robot. Because Data Driver is a listener and a library at the same time it sets itself as a listener when this library is imported into a test suite.
 
 To use it, just use it as Library in your suite.
 You may use the first argument (option) which may set the file name or path to the data file.
 
     Library    DatadDriver    my_data_file.csv

### Limitation

#### Eclipse plug-in RED
There are known issues if the Eclipse plug-in RED is used. Because the debugging Listener of this tool pre-calculates the number of test cases before the creation of test cases by the Data Driver. This leads to the situation that the RED listener throws exceptions because it is called for each test step but the RED GUI already stopped debugging so that the listener cannot send Information to the GUI. This does not influence the execution in any way but produces a lot of unwanted exceptions in the Log.  

#### Variable types 
In this early Version of DataDriver, only scalar variables are supported. Lists and dictionaries may be added in the next releases.  

#### MS Excel and typed cells
Microsoft Excel xls or xlsx file have the possibility to type thair data cells. Numbers are typically of the type float. If these data are not explicitly defined as text in Excel, pandas will read it as the type that is has in excel. Because we have to work with strings in Robot Framework these data are converted to string. This leads to the situation that a European time value like "04.02.2019" (4th January 2019) is handed over to Robot Framework in Iso time "2019-01-04 00:00:00".
This may cause unwanted behavior. To mitigate this risk you should define Excel based files explicitly as text within Excel.

### How to activate the Data Driver
To activate the DataDriver for a test suite (one specific *.robot file) just import it as a library.
You may also specify some options if the default parameters do not fit your needs.  

**Example**:

    *** Settings *** 
    Library          DataDriver
    Test Template    Invalid Logins 

  
## Structure of test suite
<a name="Structureoftestsuite"></a>
### Requirements
In the Moment there are some requirements how a test suite must be structured so that the DataDriver can get all the information it needs.  
 - only the first test case will be used as a template. All other test cases will be deleted. 
 - Test cases have to be defined with a `Test Template`. Reason for this is, that the DataDriver needs to know the names of the test case arguments. Test cases do not have named arguments. Keywords do. 
 - The keyword which is used as `Test Template` must be defined within the test suite (in the same \*.robot file). If the keyword which is used as `Test Template` is defined in a `Resource` the DataDriver has no access to its arguments names.  

### Example Test Suite
<a name="example-suite"></a>

    ***Settings*** 
    Library              DataDriver 
    Resource          login_resources.robot 
    Suite Setup       Open my Browser 
    Suite Teardown    Close Browsers 
    Test Setup        Open Login Page 
    Test Template     Invalid Login 
    
    *** Test Case *** 
    Login with user ${username} and password ${password}    Default    UserData 
    
    ***** *Keywords* ***** 
    Invalid login 
        [Arguments]    ${username}    ${password}
        Input username    ${username}
        Input pwd    ${password}
        click login button 
        Error page should be visible  

 In this example, the DataDriver is activated by using it as a Library. It is used with default settings.  
 As `Test Template` the keyword `Invalid Login` is used. This keyword has two arguments. Argument names are `${username}` and `${password}`. These names have to be in the CSV file as column header. The test case has two variable names included in its name, which does not have any functionality in Robot Framework. However, the Data Driver will use the test case name as a template name and replaces the variables with the specific value of the single generated test case.  
 This template test will only be used as a template. The specified data `Default` and `UserData` would only be used if no CSV file has been found.  
 
## Structure of data file
<a name="Structureofdatafile"></a>
### min. required columns
 
- `*** Test Cases ***` column has to be the first one. 
- *Argument columns:* For each argument of the `Test Template` keyword one column must be existing in the data file as data source. The name of this column must match the variable name and syntax.  

### optional columns
- *[Tags]* column may be used to add specific tags to a test case. Tags may be comma separated. 
- *[Documentation]* column may be used to add specific test case documentation.  

### Example Data file
<a name="example-csv"></a>

|*** Test Cases ***|${username}|${password}|[Tags]|[Documentation]|
|--|--|--|--|--|
| Right user empty pass      | demo            | ${EMPTY}        | 1          | This is a test case documentation of the first one.         |
| Right user wrong pass      | demo            | FooBar          | 2          |                                                             |
| empty user mode pass       | ${EMPTY}        | mode            | 1,2,3,4    | This test case has the Tags 1,2,3 and 4 assigned.           |
|                            | ${EMPTY}        | ${EMPTY}        |            | This test case has a generated name based on template name. |
|                            | ${EMPTY}        | FooBar          |            | This test case has a generated name based on template name. |
|                            | FooBar          | mode            |            | This test case has a generated name based on template name. |
|                            | FooBar          | ${EMPTY}        |            | This test case has a generated name based on template name. |
|                            | FooBar          | FooBar          |            | This test case has a generated name based on template name. | 

 In this data file, eight test cases are defined. Each line specifies one test case. The first two test cases have specific names. The other six test cases will generate names based on template test cases name with the replacement of variables in this name. The order of columns is irrelevant except the first column, `*** Test Cases ***`  
 
## Data Sources
<a name="DataSources"></a>

### CSV / TSV (Character-separated values)
By default DataDriver reads csv files. With the [Encoding and CSV Dialect](#EncodingandCSVDialect) settings you may configure which structure your data source has.

### XLS / XLSX Files
If you want to use Excel based data sources, you may just set the file to the extention or you may point to the correct file. If the extention is ".xls" or ".xlsx" DataDriver will interpret it as Excel file.
XLS interpreter will ignore all other options like encoding, delimiters etc.

    Library    DataDriver    .xlsx

    Library    DataDriver    file=my_data_source.xlsx
    
### PICT (Pairwise Independent Combinatorial Testing)
Pict is able to generate data files based on a model file.
https://github.com/Microsoft/pict

Documentation:
https://github.com/Microsoft/pict/blob/master/doc/pict.md

#### Requirements
- Path to pict.exe must be set in the %PATH% environment variable.
- Data model file has the file extention ".pict"
- Pict model file must be encoded in UTF-8

#### How it works
If the file option is set to a file with the extention pict, DataDriver will hand over this file to pict.exe and let it automatically generates a file with the extention ".pictout". This file will the be used as data source for the test generation. (It is tab seperated and UTF-8 encoded)
Except the file option all other options of the library will be ignored.

    Library    DataDriver    my_model_file.pict

## CSV Encoding and CSV Dialect
<a name="EncodingandCSVDialect"></a>
CSV is far away from well designed and has absolutely no "common" format. Therefore it is possible to define your own dialect or use predefined. The default is Excel-EU which is a semicolon separated file.  
 These Settings are changeable as options of the Data Driver Library.  
 
#### file=
`Library         DataDriver    file=../data/my_data_source.csv`

- None(default): Data Driver will search in the test suites folder if a *.csv file with the same name than the test suite *.robot file exists 
- only file extention: if you just set a file extentions like ".xls" or ".xlsx" DataDriver will search 
- absolute path: If an absolute path to a file is set, DataDriver tries to find and open the given data file. 
- relative path: If the option does not point to a data file as an absolute path, Data Driver tries to find a data file relative to the folder where the test suite is located.  

#### encoding=
may set the encoding of the CSV file. 
i.e. `cp1252, ascii, iso-8859-1, latin-1, utf_8, utf_16, utf_16_be, utf_16_le`, etc... 
https://docs.python.org/3.7/library/codecs.html#standard-encodings  

#### dialect=
You may change the CSV Dialect here. If the Dialect is set to 'UserDefined' the following options are used. Otherwise, they are ignored.  
 supported Dialects are:  
 

    "excel"
        delimiter = ','
        quotechar = '"'
        doublequote = True
        skipinitialspace = False
        lineterminator = '\r\n'
        quoting = QUOTE_MINIMAL
    
    "excel-tab"
        delimiter = '\t'
    
    "unix"
        delimiter = ','
        quotechar = '"'
        doublequote = True
        skipinitialspace = False
        lineterminator = '\n'
        quoting = QUOTE_ALL

#### Defaults:
        file=None,
        encoding='cp1252',
        dialect='Excel-EU',
        delimiter=';',
        quotechar='"',
        escapechar='\',
        doublequote=True,
        skipinitialspace=False,
        lineterminator='\r\n'
