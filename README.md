![GetOpt Logo](https://github.com/frossm/getopt.btm/blob/master/images/GetOpt.btm-1280x320.jpg)
# GetOpt.btm User Manual #
*Document Version 2.0*


## INTRODUCTION  ##
GetOpt is a very useful UNIX tool that will process command line options and allow the parsing and heavy lifting to be done in the background so they can be easily processed by the calling program.  I wanted to build a similar capability for my TakeCmd batch files.  Take Command is a comprehensive interactive GUI and command line environment that makes using the Windows command prompt and creating batch files easy, faster and far more powerful.  For more information, please take a look at the [JPSoftware](http://jpsoft.com).

Detailed instructions reside below, but at a high level, this batch file is called from a source batch file and is sent the command line entered (%*).  It will process the switches, options and parameters and set environment variables that can be referenced when control is passed back to the calling script.

The best way to learn this tool is probably to just turn on DEBUG mode and run GetOpt.btm and give it parameters.  You'll see everything that is being done very clearly (if I do say so myself.)  To turn on Debug Mode just execute the following in your tcc window before you run GetOpt.btm.  This also works if you run your batch file that calls GetOpt.btm.


```
set DEBUG=1 

GetOpt.btm /p1 /p2:2 file1 -P3="hello there" file2 -LongName=34
```

The version at the time of this writing produces the following debug output:
```
--- GetOpt Debug Mode Activated ------------------------------------------------

Working with command line:
  /p1 /p2:2 file1 -P3="hello there" file2 -longname=34

GetOpt is processing 6 arguments:

Scan #1: /p1
 - Item is a switch or option
 - No colon or equal sign found in "/p1"
 - ParmName = "p1"
 - Setting Variable OPTION_p1=1

Scan #2: /p2:2
 - Item is a switch or option
 - Found colon at index position: 3
 - ParmName  = "p2"
 - Parmvalue = "2"
 - Setting Variable OPTION_p2=2

Scan #3: file1
 - "file1" is a parameter, not a switch or option
 - Updating Number of Parms.  PARAM_0=1
 - Setting Variable PARAM_1="file1"

Scan #4: -P3="hello there"
 - Item is a switch or option
 - Found equal sign at index position: 3
 - ParmName  = "P3"
 - Parmvalue = ""hello there""
 - Setting Variable OPTION_P3="hello there"

Scan #5: file2
 - "file2" is a parameter, not a switch or option
 - Updating Number of Parms.  PARAM_0=2
 - Setting Variable PARAM_2="file2"

Scan #6: -longname=34
 - Item is a switch or option
 - Found equal sign at index position: 9
 - ParmName  = "longname"
 - Parmvalue = "34"
 - Setting Variable OPTION_longname=34

There were 2 parameters found.  Setting PARAM_0=2

GetOpt has completed processing 6 arguments.  Ending Execution.

--- End GetOpt Debug Messages -------------------------------------------------
```
The latest version of the software and this document can always be found at:  

**https://github.com/frossm/getopt.btm**


## DISCLAIMER ##
In no way does this tool attempt to be directly compatible with UNIX GetOpt.  It does a similar thing, but customized to work in a batch environment.

This program is licensed per the LICENSE section below.


## CALLING GETOPT.BTM ##
In order for GetOpt to do its job, it has to be called from your source batch file.  This is done near the top of the file as follows:

```
call C:\Where\Ever\You\Put\It\GetOpt.btm %*
```

If GetOpt.btm is in your path then you shouldn't need the full path.  The %* sends the command line your batch file received when started to GetOpt.btm for it to process.


## COMMAND LINE ARGUMENTS ##
I have divided the command line arguments into three groups.

   **SWITCHES:** These are On/Off switches and do not pass a value.  They can be a single character or they can be longer words.


```
Example:
Batchfile.btm /Verbose /D /Foo /Bar
```

   **OPTIONS:** These are the same as switches, but they can pass a value.  Like other switches, they can be a single character or a longer name.  The seperator character between the option name and the value can be either a colon (:) or an equal sign (=).  Please remember to quote values that contain spaces.

Also note there can be no spaces around the colon or equal sign.
	

```
Example:
Batchfile.btm /N:42 /Name=FooBar /B="Value With Spaces"
```

   **PARAMETERS:** These are not switches or options, they are just bare parameters passed to the batch file.  They are usually required as switches are often optional.


```
Example:
Batchfile.btm Filename1 Filename2
```

After GetOpt.btm is processed and control returns back to the source batch file, a set of environment variables will be set according to what was processed.  These can be checked from within source program and appropriate actions taken.  For example, if a /v switch is given, the source batch file could turn on Verbose Messages and display additional information to the user.

Please note, it is highly encouraged that the source program make use of the SetLocal / EndLocal TCMD commands.  If used properly, this will ensure that all of the GetOpt environment variables will not exist when the source batch file ends.  If not they will hang out in your environment.  Probably not a huge deal as they will be cleaned up before GetOpt.btm processes another program, but it's sloppy and your mom wouldn't want you to be messy would she?  Thought not. :-)


## PROCESSING SWITCHES & OPTIONS ##
After processing, GetOpt.btm will set an environment variable for each option and switch.  The name of the variable will begin with OPTION_ and end with the name of your option.    Please see the examples below where it will be more clear.

For any switches entered, an environment variable will be created and hold the value of 1.  The 1 is really not important as you should check for the existence of the environment variable, not really the value (although you could.)

As for options, the value of the environment variable will be the value entered with the option.

Only the "/" or the "-" can be used to start switches & options.  Anything else will not be recognized as a switch character and will be considered a parameter.

For example, if you specify the switch:

```
GetOpt.btm /a /foo:bar /LongOption=Zaphod
```

the following environment variables will be set:


```
%Option_a=1
%Option_foo=bar
%OPTION_LongOption=Zapod

```
In the calling program to check for the existance of a switch or option, I used if defined.  Here is an example:

```
Example: GetOpt.btm /Guest=Trillian

You would check for this in the calling program as:

iff defined OPTION_Guest then
  echo Hello there %OPTION_Guest
endiff

```
Here is a longer example
```
Example: GetOpt.btm -a -foo:bar -Chicago=Bears

Should be checked in the source batch file as:

   if defined OPTION_a echo User used a switch called 'a'
   if defined OPTION_foo echo User used an option called foo and set it's value to %OPTION_foo
   if defined OPTION_Chicago echo User's favorite football team is obviously the %OPTION_Chicago

```


## PARAMETERS ##
Getopt also sets a parameter variable for each paramater entered: PARAM_1 to PARAM_n.  The order of the parameters will be order on the command line.

PARAM_0 is special and contains the number of parameters entered.  This is useful for looping through all of them.  For example, you can check this in the source batch file by using:

```
do i = 1 to %PARAM_0 by 1 ...

Example:   GetOpt.btm /a /foo:bar filename1 filename2

%OPTION_a will be set to 1
%OPTION_foo will be set to bar
%PARAM_1 will be set to filename1
%PARAM_2 will be set to filename2
%PARAM_0 will be set to 2
```

## LARGE EXAMPLE ##

```
BatchFile.btm /v /port=171 Filename1.csv -LongOpt OutputFileName /x:"Hello There!"
      
%OPTION_v will equal 1
%OPTION_port will equal 171
%PARAM_1 will equal Filename1.csv
%OPTION_LongOpt will equal 1
%PARAM_2 will equal OutputFileName
%OPTION_x will equal "Hello There!"
%PARAM_0 will be set to the number of parameters, so 2 in this case

```
	  
Remember, if this is all a bit confusing, there is a robust DEBUG mode.  Set the environment variable DEBUG=1 prior to running your batch file to see a nice display of what's happening.  You can even run GetOpt.btm directly with your command line parameters and DEBUG on to see what's going on.  Please see the example at the top of this readme....going through this carefull should make this fairly self explanatory.


## CONCLUSION ##
If you have questions or need a hand, just let me know.  This is a fun batch file to write and if you are stuck or have suggestions to improve it please let me know.  Also, if there are areas in this document that need further clarifications or additions, just drop me a note.

If you find a bug please submit it to the bug tracking site at Github (where you downloaded the software) or just drop me a note.

Lastly, if you have ideas on enhancements I'd love for this to continue to evolve.  Just drop me a note.

michael @ fross.org


## LICENSE ##
[The MIT License](https://opensource.org/licenses/MIT)

Copyright 2015 - 2025 by Michael Fross

Permission is hereby granted, free of charge, to any person obtaining a copy of this software and associated documentation files (the "Software"), to deal in the Software without restriction, including without limitation the rights to use, copy, modify, merge, publish, distribute, sublicense, and/or sell copies of the Software, and to permit persons to whom the Software is furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.
