# WRF documentation for NoICE project

A guide for running the Weather Research and Forecasting (WRF) Model within the scope of the NoICE project. 

# Prerequisites 

`NoICE-Simulations` need various previous steps to be done before you can do the runs.

1. You need access to the HPC2N cluster. Therefore, go to HPC2N website (http://www.hpc2n.umu.se/) and follow the instructions in "Systems and Support" -> "Access/Accounts" -> "User Accounts" to register and have access to the cluster. You can find information on how to login in "Systems and Support" -> "Guides" -> "Short Quick-Start Guide", it will be `$ ssh username@abisko.hpc2n.umu.se`.

2. To run any simulations at Kebnekaise cluster you need to be part of a accepted project. So, once you created the account, ask javier (or me if I still have access to my LTU account) to add you to the project. 

# Introduction to WRF Model

WRF is a fully compressible, non-hydrostatic numerical model (with a hydrostatic option) that uses a terrain-following hydrostatic pressure coordinate in the vertical and the Arakawa-C grid staggering for horizontal discretization. WRF is a community model used in a wide variety of applications including couple-model applications (e.g. Samala et al., 2013), idealised simulations (e.g. Steele et al., 2013), hurricane research (e.g. Davis et al., 2007) and regional climate research (e.g. Chotamonsak et al., 2011).

Different versions of the model exist such as PolarWRF (a modified version of the WRF model for polar regions; please note that recent versions of the "traditional" WRF model already include most of the new features available in PolarWRF), ClimWRF (modified version for climate studies) and PlanetWRF (modified version for some Solar System planets including Mars and Titan). You can also couple the atmospheric model with a chemistry module (WRF-CHEM) (Javier wanted to use it in Aberdeen so it's good to be familiar with it) or an ocean model such as Regional Ocean Modelling System (ROMS) even though recent versions of WRF already come with the option of coupling the atmospheric model with a 3D or slab ocean model.

```
PolarWRF:
http://www.polarmet.osu.edu/PWRF/

ClimWRF:
http://cwrf.umd.edu/

WRF-CHEM:
http://ruc.noaa.gov/wrf/wrf-chem/

ROMS:
https://www.myroms.org/

PlanetWRF:
http://planetwrf.com/
```

I am now going to go over the main aspects related to downloading, compiling and running the model. For real-case simulations you have to run the WRF Pre-Processor (WPS) first in which you read in the coarser resolution data (e.g. re-analysis or GCM data) which you use to generate the initial and boundary conditions for the WRF runs. After downloading and compiling the model I strongly encourage you to go over the tutorials, you can perform them relatively quickly and they will give you an idea of how the model works and how to properly setup and configure WRF for different case studies. If you just want to 

```
WRF Online Tutorial:
http://www2.mmm.ucar.edu/wrf/OnLineTutorial/

WRF User's Guide:
http://www2.mmm.ucar.edu/wrf/users/docs/user_guide_V3.9/ARWUsersGuideV3.9.pdf

WRF Technical Description:
http://www2.mmm.ucar.edu/wrf/users/docs/arw_v3.pdf
```

# General info about WRF simulation

As for the WRF pre-processor (WPS), you can run it in our cluster or on the HPC2N Kebnekaise cluster, it is up to you. In this document I will give the steps of how I do it based on my experience and in Ricardo's initial advice. We will run  the WPS on Lancelot, then transfer the WRF input files to the HPC2N Kebnekaise cluster, run WRF there and then transfer the output files back to our cluster for post-processing. I put the WRF output files in "/data/no_backup/roberto/output_WRF", you can create a new folder under your name and put your output there.

Once the model is compiled you can run it. Our group has 1 project with 40,000 CPUh per month and you can use them for your simulations.

Your home folder in the HPC2N Abisko cluster will have 2 GB (backed up) and under /pfs/nobackup/home/j/juajua/ you have 2TB (not backed up!) which should be enough for all input files. Now let's go into the how to run model.

# Run the model

Before you start to configure, compile and run you need to ensure that your computer environment is set up correctly. Therefore, you have to export several variables pointing to the libraries correctly. If you ever need to reinstall them, do it as it is said in https://www2.mmm.ucar.edu/wrf/OnLineTutorial/compilation_tutorial.php#STEP2 and then export the paths to the directory where you installed it.
The paths you have to export are:
```
$ export DIR=/data/roberto/Icing_Noice_Umea_15_16/Build_WRF/LIBRARIES
$ export JASPERLIB=$DIR/grib2/lib
$ export JASPERINC=$DIR/grib2/include
$ export LDFLAGS=-L$DIR/grib2/lib
$ export CPPFLAGS=-I$DIR/grib2/include
$ export PATH=$DIR/netcdf/bin:$PATH
$ export NETCDF=$DIR/netcdf
$ export PATH=$DIR/mpich/bin:$PATH
```
———————LANCELOT CLUSTER———————

Go to your WRF directory (you can copy an existing one already compiled in Lancelot from "/data/roberto/Icing_Noice_Umea_15_16/". Copy the WPS and WRF directories to your working directory. Once you compile, you don't have to do it again, but I will tell from the first step in case you have to re-do it anytime.

1. Inside your WRF directory and type `$ ./clean` and `$ ./clean -a`

2. Then, run `$ ./configure` and select option 32 (serial with  GNU gfortran/gcc compiler). You will see various options. Choose the option that lists the compiler you are using and the way you wish to build WRF (i.e., serially or in parallel). In Lancelot we use serial, and later on you will see that in HPC2N we choose parallel.

3. Once your configuration is complete, you should have a configure.wrf file, and you are ready to compile. To compile WRF, you will need to decide which type of case you wish to compile. In our case we use "em_real" option. Now, run `$ ./compile em_real >& log.compile` to compile (this will take around 40 min). Always check the log file to make sure that everything is working well.

Once the compilation is completed, to check whether it was successful or not, you need to look for executables in the WRF/main directory: `$ ls -ls main/*.exe`. You should see: wrf.exe (model executable) real.exe (real data initialization) ndown.exe (one-way nesting) tc.exe (for tc bogusing--serial only).

IMPORTANT: Remember that the WRF you copied is already compiled in Lancelot so you don't have to compile it. This is just in case you change the fortran base code (which I guess you won't be doing). 


1. Go to your WPS directory (`$ ../WPS`)  and type `$ ./clean` `$ ./clean -a`

2. Run `./configure` and choose 1 (gnufortran/serial). The metgrid.exe and geogrid.exe programs rely on the WRF model's I/O libraries. There is a line in the configure.wps file that directs the WPS build system to the location of the I/O libraries from the WRF model, so make sure that the WRF directory is reachable from the WPS folder with `$ ../WRF`. If not then go to the configure.wps file and change the following variable to the corresponding path pointing the WRF folder: `WRF_DIR = ../WRF`. My recommendation is to call the leave it by default and just change the folder name to WRF.


3. Now is time to compile WPS with `$ ./compile >& log.compile`. This should be few minutes. Make sure that the log file looks okay at the end. If the compilation is successful, there should be 3 executables in the WPS top-level directory, that are linked to their corresponding src/ directories: geogrid.exe -> geogrid/src/geogrid.exe | ungrib.exe -> ungrib/src/ungrib.exe | metgrid.exe -> metgrid/src/metgrid.exe. IMPORTANT: Remember that the WRF you copied is already compiled in Lancelot so you dont have to compile it. This is just in case you change the fortran base code (which I guess you won't be doing). 


4. Now is time to edit namelist.wps. The "geog_data_path" variable should be '/data/no_backup/roberto/WPS_GEOG'. Here you edit your domain, the dates to run the simulation and so on. Please, see pages #66-81 of the User Guide for information about the namelist parameters.


5. Now, run `$ ./geogrid.exe >& log.geogrid`. What GEOGRID does is basically to define your model domain and static fields like orography, vegetation fraction, albedo, etc.. For each grid you have a file "geo_em.d0X.nc" where "X" is your grid number which starts from 1 and goes to your innermost grid; Now, you are ready to prepare to run ungrib. 

6. We need to link the input NCEP CFSR data. As we lost all the data if you want to do a new run you have to download the new CFSR files for the period you are interested in. I will explain how to do it. Go to `$ cd /data/no_backup/WRF_WPS_NCEP_CFSR_FILES` and download NCEP CFSR data (from http://rda.ucar.edu/; register first and then click on "NCEP CFSR" and download the 6-hourly pressure level and surface data for the period you are interested in). Go back again to the WPS main folder. And let's link the recent downloaded data. I will explain it with an example for December 2015. You have to link the surface and pressure files, hence, run the command `$ ./link_grib.csh /data/no_backup/WRF_WPS_NCEP_CFSR_FILES/*201512*sflux* /data/no_backup/WRF_WPS_NCEP_CFSR_FILES/*201512*pgrbh*`. Change "201512" for the year and month you want. The bottom line is first do for the surface files and then for the pressure level files.

7. Put the "Vtable" file in this repository in `$ ungrib/Variable_Tables/` and then inside the main WPS folder type `$ ln -sf ungrib/Variable_Tables/Vtable.CFSR Vtable`. If you use as input data from another dataset (e.g. ERA-Interim) you should use the correspondent Vtable, for our simulations we use NCEP CFSR data; 

8. You are ready to sun ungrib. Then run the ungrib executable typing `$ ./ungrib.exe >& log.ungrib`. What UNGRIB does is to read your input data and put it into an intermediate file format; (in a 3 month run this will take around 6 hours). Open log and see that it was succesfull. You should now have files with the prefix "FILE".

9. After it finishes type `$ ./metgrid.exe >& log.metgrid` and you are done with WPS! What METGRID does is to interpolate horizontally the input data into your model domain. The vertical interpolation is done in ./real.exe. In a 3 month run this will take around 3 hours. Open log and see that it was succesfull. Also see that the met_em.d0* files are created for you whole timeline.

Go back to the WRF directory `$ cd ../WRF/test/em_real` (we are doing real cases and not ideal cases) and is time to do the final steps in Lancelot

1. Before running the "real" program, you need to make all necessary changes to reflect your particular case to the namelist.input file. Edit namelist.input with your requirements. Once that is complete, you need to link your met_em* files into the working directory.

2. Link the recent created files with metgrid with the following command: `$ ln -sf ../../WPS/met_em*`.

3. You can now run the "real" program. The command for running this may vary depending on your system and the number of processors you have available, but in Lancelot it works well with: `$ mpirun -np 1 ./real.exe` (in a 3 month run this will take around 45 minutes). If you see a "SUCCESS" in there, and you see a wrfbdy_d01 file, wrffdda_d01 file, wrflowinp_d0* files and wrfinput_d0* files for each of your domains, then the run was successful.

4. Let's change to the Kebnekaise cluster.

———————KEBNEKAISE CLUSTER———————

1. Scp the WRF_Kebnekaise folder copied for you already compiled in Lancelot in /data/WRF_NoICE to your /pfs/ home path. Rename the folder from "WRF_Kebnekaise" to "WRF". This folder is already compiled so is important you take this one. 

2. Go to the WRF directory and then to `$ cd test/em_real`

3. Scp the `$ wrfbdy_d01  wrffdda_d01  wrfinput_d0*  wrflowinp_d0*` files from the Lancelots WRF/test/em_real to you current directory.

4. To run with these projects use the files "mpi_wrf_planetwrf.pbs" in WRF/test/em_real. You have to edit them (you can leave them by default as I have made it for the project you are involved in now) and change the number of processors (each node has 48 CPUs so use a multiple of it, so far I have used from 48 to 144, 1 to 3 nodes) and the walltime (maximum 7 days or 168h) and then submit them doing `$ sbatch mpi_wrf_planetwrf.pbs `. You can check how many CPU hours we have used in each project by typing `$ projinfo` (the maximum is flexible, once I have used twice the limit in a month but once you cross the limit you have to wait longer for the job to go through) in a terminal and the status of your jobs by typing `$ squeue -u username`. You can find more information about this and the usage of the cluster on the HPC2N's website.


# Hands on with WRF:

In order to run WRF for real data cases please follow the steps on the link below. I suggest you to do all the tests they have in the tutorial to learn how to use the model. The tests are fairly easy to perform and include different configurations in which you can run the model.
http://www2.mmm.ucar.edu/wrf/OnLineTutorial/CASES/index.html

As before the meaning (and options) of each of the parameters in the namelist can be found in the guide (link below, pages #166-226). The different options for the physics schemes are available in pages #143-165.
http://www2.mmm.ucar.edu/wrf/users/docs/user_guide_V3.9/ARWUsersGuideV3.9.pdf
