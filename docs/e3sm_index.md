# E3SM

**Main E3SM content:**

- main repository is [https://github.com/E3SM-Project/E3SM](https://github.com/E3SM-Project/E3SM).
-  E3SM documentation [https://docs.e3sm.org/](https://docs.e3sm.org/)
- [YouTube channel](https://www.youtube.com/@e3sm-project); amongst other subjects, the lectures from a recent tutorial are available.
- 
E3SM's case control system documentation, [CIME docs](https://esmci.github.io/cime/versions/master/html/what_cime/index.html).

**For our project's work** however, we will **use the fork [https://github.com/PCLAeroParams/E3SM](https://github.com/PCLAeroParams/E3SM) for all work related to PCLAP.**

**Computing resources:** E3SM is not intended for public use; it is meant to work well on huge computers.  We have access to Sandia's _Flight_ and _Solo_ supercomputers.

## Flight

To get started:

- To use this machine you will need to have an SRN Capacity Cluster account enabled for your username and a valid workcenter id (wcid).  To check, after logging in with `ssh flight` from a Sandia machine, the command `mywcid` will list the wcids that are avialable to you.
- Set up git with the usual `git config --global` commands for your username, email, editor, and diff tool.  E3SM relies on several git submodules, and cloning them requires forwarding Sandia's proxy information to git.   

```{.bash}
git config --global http.proxy http://proxy.sandia.gov:80
git config --global http.sslVerify False
git config --global core.gitproxy proxy.sandia.gov:80
git config --global core.noproxy 127.0.0.1,localhost,.sandia.gov,::1,10.,172.16.,172.17.,192.168.,.local,169.254/16
```
    
- Clone E3SM from our fork and update submodules: 

```{.bash}
git clone https://github.com/PCLAeroParams/E3SM pclap-e3sm
cd pclap-e3sm
git submodule update --init --recursive
```

- Set up your environment for E3SM.  Most of the work is done by CIME's machine files; these machine-specific data files configure the machine-specific compiler and job submission options that E3SM will use.  The PCLAP fork of E3SM has new machine files for Flight; however, there are still some steps that the user must do manually.  You can place the following lines in your `.bashrc` file.   

```{.bash}
  hname=$(hostname)
  FLIGHT=flight
  if [[ $hname =~ $FLIGHT ]]; then
    export SEMS_PLATFORM_OVERRIDE=boca
    source /projects/sems/modulefiles/utils/sems-archive-modules-init.sh
    module purge
    module load sems-archive-env
    module load sems-archive-intel/21.3.0
    module load sems-archive-openmpi/4.1.4
    module load sems-archive-netcdf/4.4.1
    export NETCDF_PATH=$SEMS_NETCDF_ROOT
    export NETCDFROOT=$SEMS_NETCDF_ROOT
    export PNETCDFROOT=$SEMS_NETCDF_ROOT
  fi
```

   These guards check to make sure the host is actually Flight, since it uses a shared file system that could be accessed from other machines, then loads the third-party library modules required by E3SM.  The `SEMS_PLATFORM_OVERRIDE` line is required to allow both Flight and Boca share the same module files.  
    
E3SM's case control system will attempt to download any input data that it needs and can't find already on Flight.  You will need to set up your environment to handle the Sandia proxy server, as well.  At a minimum, you'll need to define the proxies in your `.bashrc` 
    
```{.sh}
export ALL_PROXY=http://proxy.sandia.gov:80
export HTTP_PROXY=$ALL_PROXY
export HTTPS_PROXY=$ALL_PROXY
export FTP_PROXY=$ALL_PROXY
export RSYNC_PROXY=$ALL_PROXY
export SOCKS_PROXY=$ALL_PROXY
export NO_PROXY=127.0.0.1,localhost,*.sandia.gov,.sandia.gov,sandia.gov,::1,10.,172.16.,172.17.,192.168.,*.local,.local,169.254/16
export http_proxy=$HTTP_PROXY
export https_proxy=$HTTPS_PROXY
export ftp_proxy=$FTP_PROXY
export rsync_proxy=$RSYNC_PROXY
export no_proxy=$NO_PROXY
```
    
and `.wgetrc` files
    
```{.bash}
https_proxy = http://proxy.sandia.gov:80/
http_proxy = http://proxy.sandia.gov:80/
ftp_proxy = http://proxy.sandia.gov:80/
check_certificate = off
```

You can find additional help for proxy issues on [Sandia Confluence](https://wiki.sandia.gov/pages/viewpage.action?pageId=227381234#SandiaProxyConfiguration,Troubleshooting&HTTPS/SSLinterception-Environmentconfiguration(System-wide)).

### An example job script

```{.bash}

# path to your E3SM repository clone
e3sm_src=$HOME/pclap-e3sm

# case id (use a unique id for each new simulation)
case_id=test-pclap-e3sm-fork

# pick a compset 
# to list all available, run $e3sm_src/cime/scripts/query_config --compsets all
compset=F1850 #F2010, etc. 

# choose a resolution
resolution=ne4pg2_oQU480 # note: this string must match a pre-existing CIME configuration

# machine-specific stuff
wcid=FY210162
mach=flight

# CIME case setup
case_name=$case_id.$compset.$resolution
# remove pre-existing directories, if necessary
rm -rf $case_name

# set up your case, download inputdata if necessary, etc.
$e3sm_src/cime/scripts/create_newcase -case $case_name -res $resolution -compset $compset -v -project $wcid --handle-preexisting-dirs r --machine $mach --compiler intel

cd $case_name

###########################
# Parallel job parameters #
#-------------------------#
./xmlchange USER_REQUESTED_WALLTIME="00:15:00"
nthr=1 # number of openmp threads
ntask=48 # number of mpi ranks
./xmlchange MAX_MPITASKS_PER_NODE=$ntask
./xmlchange MAX_TASKS_PER_NODE=$(( $ntask * $nthr ))
./xmlchange NTASKS=$ntask
./xmlchange NTHRDS=$nthr
###########################

./case.setup
./case.build
./case.submit

```
    
### Troubleshooting

- Build fails to configure due to missing ELM submodule.  Go back to the E3SM root directory and force the submodules to update again:

```{.sh}
cd pclap-e3sm
git submodule update --init --force --recursive
```

- CIME (the case control system) doesn't populate inputdata filenames correctly; for example, instead of looking for `'/projects/ccsm/inputdata/ice/mpas-seaice/oQU480/partitions/mpas-seaice.graph.info.230422.part.48`, it looks for (and can't find) `/projects/ccsm/inputdata/ice/mpas-seaice/oQU240/.part.48` instead.   This usually means that CIME doesn't understand your grid/resolution combination.       