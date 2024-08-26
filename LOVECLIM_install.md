# 1. Download LOVECLIM
You can download LOVECLIM from the below link.      
<https://www.elic.ucl.ac.be/modx/index.php?id=289>     
~~~bash
tar xvfz LOVELIM1.4.tar.gz
~~~
<br>

# 2. Install Libraries
## 2.1. Create a Folder for Library Source Files
~~~bash
mkdir -p /home/LOVECLIM/LOVECLIM/LIB_SRC  # userid : LOVECLIM
~~~
## 2.2. Download Library Source Files
The README file in LOVECLIM suggests using udunits version 1.12.11 and ANTLR version 2.7.7.    
For other libraries, the latest versions are acceptable.    
However, depending on your server and compiler settings, there may be **version conflicts**.    
If you encounter issues, please **adjust the versions** and try again.    
(For compatibility, I used the existing versions of NetCDF and CDO that were already installed on the server.)

<br>

~~~
You need the following library source files.
You can download them from the web and extract them using 'tar xvfz'.

- antlr-2.7.7.tar.gz 
- udunits-1.12.11.tar.gz
- udunits-2.1.24.tar.gz  
- hdf5-1.14.4-3.tar.gz
- netcdf-4.1.3
- lapack-3.5.0.tar
- nco-4.6.5.tar.gz
- cdo-1.9.3
~~~

## 2.3. Install libraries
If you are not using libraries that are already installed on your server, follow these three steps:
1. Use the '**env**' command for each library.
2. Run '**make**' followed by '**make install**'.
3. Verify that a folder named '**/home/[userid]/LOVECLIM/[library_name]**' is created.    

To intsall lapack, these steps are required unlike the others:
1. Navigate to the directory '/home/[userid]/LOVECLIM/LIB_SRC/lapack-3.5.0/'.
2. Run the command '**cp INSTALL/make.inc.ifort ./make.inc**'.
3. Run '**make**' in the '/BLAS/SRC' directory.
4. Verify that the file '**librefblas.a**' is created in the '/lapack-3.5.0/' directory, then run '**cp librefblas.a libblas.a**'.
5. Run '**make**' again in the '/lapack-3.5.0/' directory.
6. Verify that the file '**liblapack.a**' is created.
7. Copy the files '**libblas.a**' and '**liblapack.a**' to the directory '**/home/[userid]/LOVECLIM/LIB/lapack/lib/**'.

Belows are the commands I used, but you may need to adjust them.
Be sure to watch out for any extra spaces, as they can affect the execution.  
~~~shell
env CC=icc CFLAGS='-Df2cFortran -fPIC' PERL=perl ./configure --prefix=/home/LOVECLIM/LOVECLIM_1.4/LIB/antlr

env CC=icc CFLAGS='-Df2cFortran -fPIC' PERL= ./configure --prefix=/home/LOVECLIM/LOVECLIM_1.4/LIB/udunits

env CC=icc CFLAGS='-Df2cFortran -fPIC' PERL= ./configure --prefix=/home/LOVECLIM/LOVECLIM_1.4/LIB/udunits2

env FC=ifort F77=ifort CXX=icpc CC=icc CPP='icpc -E' CXXCPP='icpc -E' CFLAGS='-Df2cFortran -fPIC' ./configure --prefix=/home/LOVECLIM/LOVECLIM_1.4/LIB/hdf5 --enable-fortran --enable-cxx

env FC=ifort F77=ifort CXX=icpc CC=icc CPP='icpc -E' CXXCPP='icpc -E' CFLAGS='-Df2cFortran -fPIC' CPPFLAGS="-I/home/LOVECLIM/LOVECLIM_1.4/LIB/hdf5/include -I/home/LOVECLIM/LOVECLIM_1.4/LIB/udunits2/include -I /usr/local/netcdf/4.1.3_intel21/include -I/home/LOVECLIM/LOVECLIM_1.4/LIB/antlr/include" LIBS="-L/home/LOVECLIM/LOVECLIM_1.4/LIB/hdf5/lib -L/home/LOVECLIM/LOVECLIM_1.4/LIB/udunits2/lib -L/usr/local/netcdf/4.1.3_intel21/lib -L/home/LOVECLIM/LOVECLIM_1.4/LIB/antlr/lib" NETCDF4_ROOT=/usr/local/netcdf/4.1.3_intel21 UDUNITS2_PATH=/home/LOVECLIM/LOVECLIM_1.4/LIB/udunits2 ANTLR_ROOT=/home/LOVECLIM/LOVECLIM_1.4/LIB/antlr ./configure --prefix=/home/LOVECLIM/LOVECLIM_1.4/LIB/nco
~~~

## 2.4. Modifying the '.cshrc' File
You need to update the '.cshrc' file to use libraries with the C shell.     
I used the Intel v21 compiler, and you can refer to the following section of my file. 
~~~shell
source /usr/local/intel/oneapi/compiler/latest/env/vars.csh
setenv NETCDF /usr/local/netcdf/4.1.3_intel21  
setenv LD_LIBRARY_PATH ${LD_LIBRARY_PATH}:${NETCDF}/lib
setenv NCO /home/LOVECLIM/LOVECLIM_1.4/LIB/nco
setenv CDO /usr/local/cdo/1.9.3_gcc485

# Please remember to run 'source ~/.cshrc' after making changes.
~~~

</br>

# 3. Set up the model 
## 3.1. Modifying the 'make.macros' File
> **From now on, please make a copy of the original file before making changes, using a command like 'cp [file] [file.original]'.**

You can refer to the contents of the file '/home/[userid]/LOVECLIM/CONFIG/make.macros' and add your own entries.
Here's what I have:
~~~
# Compiler names
export COMPILFORTAN  = ifort     
export COMPILCPP     = icc       

# Library paths
export NETCDFPATH    = /usr/local/netcdf/413_intel21
export HDF5PATH      = /home/LOVECLIM/LOVECLIM_1.4/LIB/hdf5
export LAPACKPATH    = /home/LOVECLIM/LOVECLIM_1.4/LIB/lapack
export BLASPATH      = /home/LOVECLIM/LOVECLIM_1.4/LIB/lapack

# Options when compiling
export OPTIFLAGSFORTRAN   = -O2 -W1 #-warn all
export OPTIFLAGSCPP  = -O2
export FORTANFLAGS   =  -cpp -convert big_endian -assume byterecl -align dcommon -Dlinux86 $(OPTIFLAGSFORTRAN)
export CPPFLAGS =  $(OPTIFLAGSCPP)
export INCS          = -Isrc -I. -I$(NETCDFPATH)/include
export LIBSFORTRAN   = -L$(NETCDFPATH)/lib -lnetcdf -lnetcdff -L$(LAPACKPATH)/lib -llapack -L$(BLASPATH)/lib -lblas
export LIBSCPP       = -L$(NETCDFPATH)/lib -lnetcdf_c++ -lnetcdff -lnetcdf
~~~

## 3.2. Fixing bugs
 ※ If you cannot fine the 'atlas' folder, you can obtain it from LOVECLIM v1.3. 

### 3.2.1. In lines of 80, 105, 135, and 174 of the file '/home/[userid]/LOVECLIM/RUN/tools/atlas/src/post/input.f', delete the second argument of the 'error' subroutine as shown below.

~~~Fortran
# Before 
call error('insufficient data on input line ',34)    
# After  
call error('insufficient data on input line ')
~~~
### 3.2.2. In lines of 8 and 12 of the file '/home/[userid]/LOVECLIM/RUN/tools/EvoluCat2NetCDF.cpp, modify the code as follows.

~~~cpp
// Before
#include <netcdf>
// After    
#include <netcdf.h>

//Before 
using namespace netCDF    
// After 
//using namespace netCDF
~~~

### 3.2.3. Apply the same modifications as described in section 3.2.2. to the following files:
**'/home/[userid]/LOVECLIM/RUN/tools/EvoluCat2NetCDFan.cpp'**   
**'/home/[userid]/LOVECLIM/TOOLS/EvoluCat2NetCDF.cpp'**   
**'/home/[userid]/LOVECLIM/TOOLS/EvoluCat2NetCDFan.cpp'**   
**'/home/[userid]/LOVECLIM/TOOLS/VolcCat2NetCDF.cpp'**   
**'/home/[userid]/LOVECLIM/TOOLS/VolcCat2NetCDF_oldFormat.cpp'**   

## 3.3. Compiling and Setting up the environment
### 3.3.1. Compile in the '/home/[userid]/LOVECLIM/CONFIG' directory by running the 'make' command. If you have already run 'make', first run 'make clean' and then execute 'make' again.
Please chech if the executable files in the following folers are created:   
/home/[userid]/LOVECLIM/RUN/tools/atlas/src/post   
/home/[userid]/LOVECLIM/RUN/tools/atlas/src/ocean   
/home/[userid]/LOVECLIM/RUN/tools/atlas/veget   
/home/[userid]/LOVECLIM/RUN/   
/home/[userid]/LOVECLIM/RUN/figlet   
/home/[userid]/LOVECLIM/TOOLS/ 

### 3.3.2. In lines of 31, 32, 37, and 38 of the file '/home/[userid]/LOVECLIM/RUN/expdir/ref/make.macros', adjust the code to match your environment. 
~~~macros
NETCDFINCLUDE   = -I/usr/local/netcdf/413_intel21/include
NETCDFLIB       = -L/usr/local/netcdf/413_intel21/lib -lnetcdf -lnetcdff
UDUNITSINCLUDE  = -I/home/LOVECLIM/LOVECLIM_1.4/LIB/udunits/include
UDUNITSLIB      = -L/home/LOVECLIM/LOVECLIM_1.4/LIB/udunits/lib -ludunits
~~~

### 3.3.3 In lines of 40, 83, 96, and 100 of the file '/home/[userid]/LOVECLIM/RUN/expdir/newexp', modify the code as follows.
~~~
# Before 
if [ -z ${LOVEOUTTMP} ]; then
# After 
if [ -z #{Scratchdir} ]; then

# Before
scratchdir="${LOVEOUTTMP}/LOVECLIM$loveclimver"
# After
scratchdir="${Scratchdir}/LOVECLIM$loveclimver"

# Before
NbRun=`python3 -c "from math import ceil; printint(ceil(${lengthInDays}/(360.*100.)))"`
# After
NbRun=`python3 -c "from math import ceil; print(int(ceil(${lengthInDays}/(360.*100.))))"`

# Before
SecondTotal=`python3 -c "from math import ceil; printint(ceil((${lengthInDays}/360.)*4*36/${NbRun}.))"`
# After
SecondTotal=`python3 -c "from math import ceil; print(int(ceil((${lengthInDays}/360.)*4*36/${NbRun}.)))"`
~~~
※ You may have to comment out the parts in the 'newexp' file that causes errors.

# 4. A quick classical run test
1. Navigate to the 'RUN/expdir' directory.
2. Edit the example.param file, paying attention to the '**Scratchdir**' and '**SavingPath**' settings.
3. Run '**./newexp example.param**'.
4. Change to the 'example' directory.
5. Run '**./launch_r1 &**' and then '**tail -f ./log/subrepo.log**' to monitor the log.