GetOpt.btm User Manual 
Document Version 2.0


INTRODUCTION
------------
Getopt is a very useful UNIX tool that will process command line options.  I wanted to build a similar capability for my TakeCmd batch files.

Detailed instructions reside below, but at a high level, this batch file is called from a source file and is sent the command line entered.  It will process the options and parameters and set variables that can be referenced when control is passed back to the calling file.

The best way to learn this tool is probably to just run GetOpt.btm and give it parameters.  If you turn DEBUG mode on you can see everything that is being done very clearly (if I do say so myself.)  To turn on Debug Mode just execute: 


```
#!batch

set DEBUG=1 
```

in your environment before you run GetOpt.btm.  This also works if you run your batchfile that calls GetOpt.btm.

The latest version of the software can always be found at:  

**## https://bitbucket.org/frossm/getopt.btm ##**


DISCLAIMER
----------
In no way does this tool attempt to be directly compatible with UNIX GetOpt.  It does a similar thing, but customized to work in a batch environment.

Secondly, this is free software and there is no warrantee at all, implied or otherwise.  Use at your own risk.


CALLING GETOPT.BTM
------------------
In order for GetOpt to do its job, it has to be called from your source batch file.  This is done near the top of the source file as follows:


```
#!batch

call C:\Where\Ever\You\Put\It\GetOpt.btm %*
```


If GetOpt.btm is in your path then you shouldn't need the full path.  The %* sends the command line your batch file received when started to GetOpt.btm for processing.


COMMAND LINE ARGUMENTS
----------------------
I have divided the command line arguments into three groups.

   **SWITCHES:**  These are On/Off switches and do not pass a value.  They can be a single character or they can be longer words.


```
#!batch

Example:  Batchfile.btm /Verbose /D /Foo /Bar
```

   **OPTIONS:**  These are the same as switches, but they can pass a value.  Like other switches, they can be a single character or a longer name
	

```
#!batch

Example:  Batchfile.btm /N:7 /Name=FooBar /B="Arg With Spaces"
```

   **PARAMATERS:**  These are not switches or options, they are just bare parameters passed to the batch file  They are usually required as switches are often optional.


```
#!python

Example:  Batchfile.btm Filename1 Filename2
```

After GetOpt.btm is processed and control returns back to the source batch file, a set of environment variables will be set according to what was processed.  These can be checked from within source program and appropriate actions taken.  For example, if a /v switch is given, the source batch file could turn on Verbose Messages and display additional information to the user.

Please note, it is highly encouraged that the source program make use of the SetLocal / EndLocal TCMD commands.  If used properly, this will ensure that all of the GetOpt environment variables will not exist when the source batch file ends.  If not they will hang out in your environment.  Probably not a huge deal as they will be cleaned up before GetOpt.btm processes another program, but it's sloppy.  Your mom wouldn't want you to be messy would she?  Thought not. :)



SWITCHES & OPTIONS
------------------
For any switches entered, an environment variable will be created and hold the value of 1.  The 1 is really not important as you should check for the existence of the environment variable, not really the value (although you could.)

As for options, the value of the environment variable will be the value entered with the option.

Only the "/" or the "-" can be used to start switches & options.  Anything else will not be recognized as a switch character and will be considered a parameter.

For example, if you specify the switch:


```
#!batch

GetOpt.btm /a /foo:bar /LongOption=Snowman
```

the following environment variables will be set:


```
#!python

%Option_a=1
%Option_foo=bar
%OPTION_LongOption=Snowman

```

To check for an option in your source batch file use:


```
#!batch

if defined OPTION_arg
```


```
#!batch

Example: GetOpt.btm -a -foo:bar -Chicago=Bears

Should be checked in the source batch file as:

   if defined OPTION_a echo User used a switch called 'a'
   if defined OPTION_foo echo User used an option called foo and set it's value to %OPTION_foo
   if defined OPTION_Chicago echo User's favorite football team is obviously the %OPTION_Chicago

```


PARAMETERS
----------
Getopt also sets a parameter variable for each paramater entered: PARAM_1 to PARAM_n.  PARAM_0 is special and holds the value that contains the number of PARAMs.  This is useful for looping through all of them.  For example, you can check this in the source batch file by using:


```
#!batch

do i = 1 to %PARAM_0 by 1 ...

Example:   GetOpt.btm /a /foo:bar filename1 filename2

%OPTION_a will be set to 1
%OPTION_foo will be set to bar
%PARAM_1 will be set to filename1
%PARAM_2 wil lbe set to filename2
%PARAM_0 will be set to 2
```

   
   
LARGE EXAMPLE
-------------


```
#!batch

BatchFile.btm /v /port=171 Filename1.csv -LongOpt OutputFileName /x:"Hello There!"
      
%OPTION_v will equal 1
%OPTION_port will equal 171
%PARAM_1 will equal Filename1.csv
%OPTION_LongOpt will equal 1
%PARAM_2 will equal OutputFileName
%OPTION_x will equal "Hello There!"
%PARAM_0 will be set to the number of parameters, so 2 in this case

```

	  
Remember, if this is all a bit confusing, there is a robust DEBUG mode.  Set the environment variable DEBUG=1 prior to running your batch file to see a nice display of what's happening.  You can even run GetOpt.btm directly with your command line parameters and DEBUG on to see what's going on.  

Here is the above example with the debug toggle before and after:


```
#!batch

set DEBUG=1 %+ GetOpt.btm /v /port=171 Filename1.csv -LongOpt OutputFileName /x:"Hello There!" %+ unset DEBUG

```

   
MORE HELP PLEASE
----------------

If you have questions or need a hand, just let me know.  This is a fun batch file to write and if you are stuck or have suggestions to improve it please let me know.  Also, if there are areas in this document that need further clarifications or additions, just drop me a note.

michael@fross.org