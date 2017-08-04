# Generates a fat16 file system

While this is intended for use in embedded devices, the test code provided will generate a static image.   In the dynamic
case, the file system consists of a constant table driven file system with a few built-in functions to generate
content on demand.  This content includes the fat16 boot record and fat tables as well as directories and static files.
The code also provides a hook for dynamic file content.

## How this works

The main module, gensys, takes as input a root directory (containing static files and directories) and a list
of "virtual file" definitions (where they go in the file hierarchy, their size, a generator function name,
and a (void *) operand).    The code below (in gensys.py) generates an example file system definition on *stdout*.

````c
    volume_name = 'Test Volume'   # fat16 volume name
    table_name  = 'filesys'       # C name of file system table
    fdout = sys.stdout            # output file
    rootdir = 'testdir'           # path to root file system

    vfiles = [{'path' : os.path.join(rootdir,'a virtual file.txt'), 
               'size' : 1024, 
               'func' : 'vfile_1', 
               'operand' : '(void *) 0'
               }
              ]

    fs = vFileSystem(rootdir,vfiles)
    fs.gensys(fdout,table_name,volume_name)
````

It doesn't appear (easily) feasible to generate a file system that the host can modify; thankfully that wasn't necessary for
our application --  an embedded data logger with a large flash (2Gbit) chip.  We wanted an easy way to 
access that data and perhaps a bit of status information (through a virtual file).

To test this, you don't need an embedded device.  The Makefile will generate an image  --
execute  **make** to generate test image _test.img_ that can be mounted.  This image
contains the files and directories in _testdir_ along with one virtual file _a virtual file.txt_. The contents of this
file are generated by the procedure *vfile_1* in *testmain.c*

## Implementation Notes

The other major module is *fatname.py*.  This is where all the conversion from host file system names to Microsoft long and
short file names occurs.  It's somewhat hard to be sure that all possible cases are accounted for (the available fat file
system documentation leaves a bit too much as an exercise to the reader).   If there's a case that doesn't work, perhaps the
best thing is to rename your static files; alternatively, suggest a patch.  

The code also doesn't check for "long name" collisions -- cases where two
long names are rewritten (to meet windows requirements) to the same string.  

This was implemented using the gcc compiler; _fat.h_ uses attributes to define packed data structures for key components of the 
fat16 file system.  This is an area of risk if you move to a different compiler.

I did look at emfat (see below) but decided that I wanted to pursue a different direction.  In particular, fat16gen uses a python script to generate information that is similar to what emfat requires as input.  Another essential difference is that the fat16gen generates static directory "files".   This greatly simplified the necessary runtime and made it easier to support the rather complex issues related to long file names.

link to fat documentation: http://download.microsoft.com/download/1/6/1/161ba512-40e2-4cc9-843a-923143f3456c/fatgen103.doc
a related project for fat32: https://github.com/fetisov/emfat
