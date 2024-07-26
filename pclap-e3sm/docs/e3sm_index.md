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

1. To use this machine you will need to have an SRN Capacity Cluster account enabled for your username and a valid workcenter id (wcid).  To check, after logging in with `ssh flight` from a Sandia machine, the command `mywcid` will list the wcids that are avialable to you.
2. Set up git with the usual `git config --global` commands for your username, email, editor, and diff tool.  E3SM relies on several git submodules, and cloning them requires forwarding Sandia's proxy information to git.   

    ```
      git config --global http.proxy http://proxy.sandia.gov:80
      git config --global http.sslVerify False
      git config --global core.gitproxy proxy.sandia.gov:80
      git config --global core.noproxy 127.0.0.1,localhost,.sandia.gov,::1,10.,172.16.,172.17.,192.168.,.local,169.254/16
    ```
    
3. Clone E3SM from our fork and update submodules: 

    ```
    git clone https://github.com/PCLAeroParams/E3SM pclap-e3sm
    cd pclap-e3sm
    git submodule update --init --recursive
    ```

4. Set up your environment for E3SM.  Most of the work is done by CIME's machine files; these machine-specific data files configure the machine-specific compiler and job submission options that E3SM will use.  The PCLAP fork of E3SM has new machine files for Flight; however, there are still some steps that the user must do manually.  You can place the following lines in your `.bashrc` file.   

    ```
      hname=$(hostname)
      FLIGHT=flight
      if [[ $hname =~ $FLIGHT ]]; then
        export SEMS_PLATFORM_OVERRIDE=boca
        source /projects/sems/modulefiles/utils/sems-archive-modules-init.sh
      fi
    ```

    These guards check to make sure the host is actually Flight, since it uses a shared file system that could be accessed from other machines, then loads the third-party library modules required by E3SM.  The `SEMS_PLATFORM_OVERRIDE` line is required to allow both Flight and Boca share the same module files.  
    
    
    
    
### Troubleshooting

1. Build fails to configure due to missing ELM submodule.  Go back to the E3SM root directory and force the submodules to update again:

    ```
    cd pclap-e3sm
    git submodule update --init --force --recursive
    ```
    
2. Input data fail to download due to proxy issues with `wget` and `svn`.  Asked for help from the E3SM/CIME team for this one.  Answer TBD.