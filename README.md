# BBC-BCPLStandAlone01
A first look at the stand alone generator.

Having finally got access to the Stand Alone Generator (SAG) guide (http://www.stardot.org.uk/forums/viewtopic.php?f=2&t=10380) I have built an extremely simple Hello World application and compiled it using the SAG.

It is actually surprisingly simple (for a small program anyway). This project shows how it is done.

I have included the following files: - 

$.COMPILE - the file used to build and deploy the runtime file

$.TESTRT - the program to be deployed

BCPLS.ssd the disc image of the BCPL source files

BCPLSAG - the disc image of the runtime files and deployed files (I added EX to this to avoid messing around with *LIB)


$.TESTRT - this file needs a bit of futher explanation, as below.

      //Trivial program to test the Stand alone generator
      //
      //Filename TESTRT
      //
      //29/05/2016
      //
      //v1.0
      
      NEEDS "VDU"
      NEEDS "INTERP"
      NEEDS "MODE"
      NEEDS "RUNPROG"
      
      GET "LIBHDR"
      
      LET START() BE
      $(
         MODE(0)
         VDU("19,1,2")
         VDU("17,1")
         WRITES("*N")
         WRITES("Hello world from BCPL Runtime!")
         WRITES("*N")
         RUNPROG("**BASIC")
         STOP(0)
      $)

The key thing to note with this file is the additional NEEDS sections (see points 3, 4 and 7 below) at the start and the RUNPROG (see point 6 below) at the end.
The NEEDS listing tells the compiler which sections of the runtime library to include when it is being built. The RUNPROG although not strictly necessary gracefully drops the user back into BASIC after running the program.


Summarised stand along generation process from the guide: - 

1) Create a very simple BCPL program and test it

2) Make any required changes for the SAG. This is basically removing any function calls that are not supported

3) List all of the library routines used in your program and note the section they belong to (see chapter 5 for a list) and add each one as a NEEDS "..."

4) Add NEEDS "INTERP" at the end of the list

5) Look at the IO options required (none for my simple program)

6) Decide on termination options. I went for a very simple return to basic by calling *BASIC at the end. It may well be fine to do nothing and force the user to do a CTRL+Break to exit, i.e. do nothing

7) Add all of the NEEDS either to main program or one of the files in it, or ideally a separate file (I added them to the main program file but will do it properly later on)

8) Compile the BCPL programs as normal but with the switch NONAMES as this reduces file size slightly, in my case "BCPL TESTRT RT NONAMES". Note that you do not do the normal NEEDCIN at this stage, unless you are including a library you have written yourself. All standard library stuff, e.g. VDU will be picked up from the run time library later on

9) Decide on whether to use SYSLIB1 or 2. This mainly comes down to file IO I believe. We are not using any so will stick with SYSLIB1

10) Time for NEEDCIN! Ideally use EX to run a commmand file (I have used this) to do all the heavy lifting of files

11) Use PACKCIN to remove unwanted stuff and reduce the file size a bit

12) Ignore the ROM stuff for now

13) Decide on the location of the global vector (effectively the load location of the program which needs to be at PAGE or above). I have stuck with &1900 assuming a DFS of some sort

14) Use FIXCIN to create the program (ignore ROMs for now)

15) Use FILETRN to copy the file to keep its execution address (I think I used *COPY which seems to work as well)

All of stages 9 through to 14 are managed by a single command file used in EX. I took mine directly from the manual and it works well in this simple example: - 

      .KEY INFILE/A,OUTFILE/A
      NEEDCIN <INFILE> SYSLIB1 $TEMP1
      PACKCIN FROM $TEMP1 TO $TEMP2
      DELETE $TEMP1
      FIXCIN FROM $TEMP2 TO <OUTFILE> GV=1900
      DELETE $TEMP2

  The .KEY line handles the file names.
  The NEEDCIN line handles points 9 and 10.
  The PACKCIN line handles point 11.
  The DELETE lines tidies up an unwanted temp file.
  The FIXCIN line handles points 13 and 14.

So assuming that our original BCPL compiled file is called RT (and the EX command file is called COMPILE) I would call - 

      EX COMPILE RT HELLO

The complete sequence to build and deploy this file is therefore: - 

      Load BCPLS into drive 0
      Load BCPLSAG into drive 1 (we don't need BCPLT for this project)
      On drive 0 type "BCPL TESTRT RT NONAMES"
      Switch to drive 1 and "SAVE RT"
      On drive 1 type "EX COMPILE RT HELLO"

After this has finished running you end up with a file called HELLO that can be run outside of BCPL by typing *HELLO

In terms of file sizes the initial RT (compiled file) is &90 (144) bytes long and after being converted to runtime is &F82 (3970) bytes long. This is therefore &EF2 (3826) bytes of interpreter and function code. While this is quite a jump in size it is impressive given that it includes the BCPL interpreter! Don't be expecting to do anything useful in MODE 0 on a non-expanded machine though...


When run the HELLO program should change the screen to MODE 0, turn the text green and say hello. It then calls RUNPROG("**BASIC") to terminate and return control to BASIC.

There are a whole raft of options and further settings but this is good enough for now for a very simple demo.
