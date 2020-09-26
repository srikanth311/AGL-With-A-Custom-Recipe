## Yocto-Project-1: Creating a sample Yocto project recipe and adding it to Automotive Grade Linux Image

In this article, we will see how to build an AGL image along with a custom meta layer and a sample recipe. These steps are basic understanding steps that we need to understand to build our own custom recipes.

#### Steps:
1. Prepare the build host.
2. Create an AGL image using Yocto project
3. Create a custom meta layer with name : meta-srikanth
4. Create a sample bitbake recipe with name `helloworld` inside the "meta-srikanth" layer.
5. Attach the new layer and recipe to the base image
6. Build the image
7. Run the emulator using `runqemu` and execute the sample recipe program that was creaated above.

#### Build the host
```
sudo apt-get update
sudo apt-get install curl gawk wget git diffstat unzip texinfo gcc-multilib build-essential chrpath socat cpio python python3 python3-pip python3-pexpect python-dev xz-utils debianutils iputils-ping cpu-checker default-jre parted
```
#### Create AGL Base Image.
```
mkdir agl-yocto
cd agl-yocto/
```

```
mkdir -p ~/bin
export PATH=~/bin:$PATH
curl https://storage.googleapis.com/git-repo-downloads/repo > ~/bin/repo
chmod a+x ~/bin/repo
repo init -u https://gerrit.automotivelinux.org/gerrit/AGL/AGL-repo
repo sync
```

```
source meta-agl/scripts/aglsetup.sh
=> This creates a build directory and it will take you to that directory.
=> pwd
/home/ubuntu/agl-yocto/build

=> Now bitbake with minimal image. This process will take 30 mins to 1 hr depending on your machine number of cores.
```
Depending on your host CPU and memory, running the below step takes anywhere from 30 mins to 1 hr for the 1st time.
```
=> bitbake agl-image-minimal
```

#### To check layers:
```
cd agl-yocto
source meta-agl/scripts/aglsetup.sh

> bitbake-layers show-layers
NOTE: Starting bitbake server...
layer                 path                                      priority
==========================================================================
meta-oe               /home/ubuntu/agl-yocto/external/meta-openembedded/meta-oe  6
meta-multimedia       /home/ubuntu/agl-yocto/external/meta-openembedded/meta-multimedia  6
meta-networking       /home/ubuntu/agl-yocto/external/meta-openembedded/meta-networking  5
meta-python           /home/ubuntu/agl-yocto/external/meta-openembedded/meta-python  7
meta-filesystems      /home/ubuntu/agl-yocto/external/meta-openembedded/meta-filesystems  6
meta-agl-profile-core  /home/ubuntu/agl-yocto/meta-agl/meta-agl-profile-core  80
meta-agl-distro       /home/ubuntu/agl-yocto/meta-agl/meta-agl-distro  70
meta-agl-bsp          /home/ubuntu/agl-yocto/meta-agl/meta-agl-bsp  60
meta-security         /home/ubuntu/agl-yocto/external/meta-security  8
meta-perl             /home/ubuntu/agl-yocto/external/meta-openembedded/meta-perl  6
meta-security         /home/ubuntu/agl-yocto/meta-agl/meta-security  60
meta-app-framework    /home/ubuntu/agl-yocto/meta-agl/meta-app-framework  70
meta                  /home/ubuntu/agl-yocto/external/poky/meta  5
meta-poky             /home/ubuntu/agl-yocto/external/poky/meta-poky  5

```
#### Creating a custom layer
```
> cd agl-yocto
> $ bitbake-layers create-layer meta-srikanth
NOTE: Starting bitbake server...
Add your new layer with 'bitbake-layers add-layer meta-srikanth'

> tree -L 2 meta-srikanth/
meta-srikanth/
├── conf
│   └── layer.conf
├── COPYING.MIT
├── README
└── recipes-example
    └── example

3 directories, 3 files
```

#### Adding the Layer and Checking If the layer was created
```
> cd build/
> bitbake-layers add-layer ../meta-srikanth/
> $ bitbake-layers show-layers
NOTE: Starting bitbake server...
layer                 path                                      priority
==========================================================================
meta-oe               /home/ubuntu/agl-yocto/external/meta-openembedded/meta-oe  6
meta-multimedia       /home/ubuntu/agl-yocto/external/meta-openembedded/meta-multimedia  6
meta-networking       /home/ubuntu/agl-yocto/external/meta-openembedded/meta-networking  5
meta-python           /home/ubuntu/agl-yocto/external/meta-openembedded/meta-python  7
meta-filesystems      /home/ubuntu/agl-yocto/external/meta-openembedded/meta-filesystems  6
meta-agl-profile-core  /home/ubuntu/agl-yocto/meta-agl/meta-agl-profile-core  80
meta-agl-distro       /home/ubuntu/agl-yocto/meta-agl/meta-agl-distro  70
meta-agl-bsp          /home/ubuntu/agl-yocto/meta-agl/meta-agl-bsp  60
meta-security         /home/ubuntu/agl-yocto/external/meta-security  8
meta-perl             /home/ubuntu/agl-yocto/external/meta-openembedded/meta-perl  6
meta-security         /home/ubuntu/agl-yocto/meta-agl/meta-security  60
meta-app-framework    /home/ubuntu/agl-yocto/meta-agl/meta-app-framework  70
meta                  /home/ubuntu/agl-yocto/external/poky/meta  5
meta-poky             /home/ubuntu/agl-yocto/external/poky/meta-poky  5
meta-srikanth         /home/ubuntu/agl-yocto/meta-srikanth  6
srikanth@ubuntu:~/agl-yocto/build$


================================================================
```

#### Creating a recipe

```
cd meta-srikanth/
mkdir recipes-helloworld
cd recipes-helloworld/
mkdir helloworld
cd helloworld
touch helloworld_0.1.bb
```
Add the below contents in this `helloworld_0.1.bb` file.
```
SUMMARY = "Simple helloworld application"
SECTION = "examples"
LICENSE = "MIT"
LIC_FILES_CHKSUM = "file://${COMMON_LICENSE_DIR}/MIT;md5=0835ade698e0bcf8506ecda2f7b4f302"

SRC_URI = "file://helloworld.c"

S = "${WORKDIR}"
INSANE_SKIP_${PN} = "ldflags"

do_compile() {
         ${CC} helloworld.c -o helloworld
}

do_install() {
         install -d ${D}${bindir}
         install -m 0755 helloworld ${D}${bindir}
}
```

```
mkdir files
cd files
touch helloworld.c
```
Add the below contents to the `helloworld.c` file.

```
#include <stdio.h>
int main() {
   printf("Hello, World!");
   return 0;
}
```

#### Bit bake the helloworld recipe

```
cd meta-srikanth/recipes-helloworld/helloworld/files
bitbake helloworld
```
Output will be something like below.
```
Loading cache: 100% |############################################################################################################################################################################################################################################| Time: 0:00:00
Loaded 3572 entries from dependency cache.
Parsing recipes: 100% |##########################################################################################################################################################################################################################################| Time: 0:00:00
Parsing of 2402 .bb files complete (2401 cached, 1 parsed). 3573 targets, 291 skipped, 1 masked, 0 errors.
WARNING: No bb files matched BBFILE_PATTERN_agl-distro '^/home/ubuntu/agl-yocto/meta-agl/meta-agl-distro/'
NOTE: Resolving any missing task queue dependencies

Build Configuration:
BB_VERSION           = "1.46.0"
BUILD_SYS            = "x86_64-linux"
NATIVELSBSTRING      = "universal"
TARGET_SYS           = "x86_64-agl-linux"
MACHINE              = "qemux86-64"
DISTRO               = "poky-agl"
DISTRO_VERSION       = "9.99.3"
TUNE_FEATURES        = "m64 corei7"
TARGET_FPU           = ""
meta-oe              
meta-multimedia      
meta-networking      
meta-python          
meta-filesystems     = "HEAD:cc6fc6b1641ab23089c1e3bba11e0c6394f0867c"
meta-agl-profile-core 
meta-agl-distro      
meta-agl-bsp         = "HEAD:cb90a88d382971a146c4b7413bb6df6563cef7d1"
meta-security        = "HEAD:982a29bbb7ef32475aea7c4bb56c620065a50927"
meta-perl            = "HEAD:cc6fc6b1641ab23089c1e3bba11e0c6394f0867c"
meta-security        
meta-app-framework   = "HEAD:cb90a88d382971a146c4b7413bb6df6563cef7d1"
meta                 
meta-poky            = "HEAD:4e931b1d05018923dc145cd97f6f965f5cb6e1a5"
meta-srikanth        = "<unknown>:<unknown>"

Initialising tasks: 100% |#######################################################################################################################################################################################################################################| Time: 0:00:01
Sstate summary: Wanted 7 Found 0 Missed 7 Current 130 (0% match, 94% complete)
NOTE: Executing Tasks
NOTE: Tasks Summary: Attempted 551 tasks of which 536 didn't need to be rerun and all succeeded.

Summary: There was 1 WARNING message shown.
```

#### Building an image

```
bitbake-layers show-recipes "*-image-*"
```

```
cd agl-yocto
cd meta-srikanth
mkdir recipes-core
cd recipes-core
mkdir images
cd images
touch core-image-srikanth_0.1.bb
```
Add the below contents to the above `core-image-srikanth_0.1.bb` file.
```
cat core-image-srikanth_0.1.bb 
SUMMARY = "A simple image just capable of allowing a device to boot with helloworld package added."

IMAGE_INSTALL = "packagegroup-core-boot ${CORE_IMAGE_EXTRA_INSTALL}"

IMAGE_LINGUAS = " "

LICENSE = "MIT"

inherit core-image

CORE_IMAGE_EXTRA_INSTALL += "helloworld"
```

##### Now bitbake the image.
```
bitbake core-image-srikanth
```

You will see the below output.

```
Loading cache: 100% |############################################################################################################################################################################################################################################| Time: 0:00:00
Loaded 3573 entries from dependency cache.
WARNING: No bb files matched BBFILE_PATTERN_agl-distro '^/home/ubuntu/agl-yocto/meta-agl/meta-agl-distro/'
NOTE: Resolving any missing task queue dependencies

Build Configuration:
BB_VERSION           = "1.46.0"
BUILD_SYS            = "x86_64-linux"
NATIVELSBSTRING      = "universal"
TARGET_SYS           = "x86_64-agl-linux"
MACHINE              = "qemux86-64"
DISTRO               = "poky-agl"
DISTRO_VERSION       = "9.99.3"
TUNE_FEATURES        = "m64 corei7"
TARGET_FPU           = ""
meta-oe              
meta-multimedia      
meta-networking      
meta-python          
meta-filesystems     = "HEAD:cc6fc6b1641ab23089c1e3bba11e0c6394f0867c"
meta-agl-profile-core 
meta-agl-distro      
meta-agl-bsp         = "HEAD:cb90a88d382971a146c4b7413bb6df6563cef7d1"
meta-security        = "HEAD:982a29bbb7ef32475aea7c4bb56c620065a50927"
meta-perl            = "HEAD:cc6fc6b1641ab23089c1e3bba11e0c6394f0867c"
meta-security        
meta-app-framework   = "HEAD:cb90a88d382971a146c4b7413bb6df6563cef7d1"
meta                 
meta-poky            = "HEAD:4e931b1d05018923dc145cd97f6f965f5cb6e1a5"
meta-srikanth        = "<unknown>:<unknown>"

Initialising tasks: 100% |#######################################################################################################################################################################################################################################| Time: 0:00:06
Sstate summary: Wanted 3 Found 0 Missed 3 Current 1267 (0% match, 99% complete)
NOTE: Executing Tasks
NOTE: Tasks Summary: Attempted 3390 tasks of which 3375 didn't need to be rerun and all succeeded.

Summary: There was 1 WARNING message shown.
```

#### Now check the images that are created.

```
cd  /home/agl-yocto/build/tmp/deploy/images/qemux86-64

You will see the images that are created as below.

ls core-image-srikanth*
core-image-srikanth.env                                      core-image-srikanth-qemux86-64-20200901184604.rootfs.manifest  core-image-srikanth-qemux86-64-20200901184604.rootfs.wic.xz  core-image-srikanth-qemux86-64.manifest       core-image-srikanth-qemux86-64.wic.bmap
core-image-srikanth-qemux86-64-20200901184604.qemuboot.conf  core-image-srikanth-qemux86-64-20200901184604.rootfs.wic.bmap  core-image-srikanth-qemux86-64-20200901184604.testdata.json  core-image-srikanth-qemux86-64.qemuboot.conf  core-image-srikanth-qemux86-64.wic.vmdk
core-image-srikanth-qemux86-64-20200901184604.rootfs.ext4    core-image-srikanth-qemux86-64-20200901184604.rootfs.wic.vmdk  core-image-srikanth-qemux86-64.ext4                          core-image-srikanth-qemux86-64.testdata.json  core-image-srikanth-qemux86-64.wic.xz
```

#### Listing Tasks

```
bitbake -c listtasks helloworld
Loading cache: 100% |############################################################################################################################################################################################################################################| Time: 0:00:00
Loaded 3573 entries from dependency cache.
WARNING: No bb files matched BBFILE_PATTERN_agl-distro '^/home/ubuntu/agl-yocto/meta-agl/meta-agl-distro/'
NOTE: Resolving any missing task queue dependencies

Build Configuration:
BB_VERSION           = "1.46.0"
BUILD_SYS            = "x86_64-linux"
NATIVELSBSTRING      = "universal"
TARGET_SYS           = "x86_64-agl-linux"
MACHINE              = "qemux86-64"
DISTRO               = "poky-agl"
DISTRO_VERSION       = "9.99.3"
TUNE_FEATURES        = "m64 corei7"
TARGET_FPU           = ""
meta-oe              
meta-multimedia      
meta-networking      
meta-python          
meta-filesystems     = "HEAD:cc6fc6b1641ab23089c1e3bba11e0c6394f0867c"
meta-agl-profile-core 
meta-agl-distro      
meta-agl-bsp         = "HEAD:cb90a88d382971a146c4b7413bb6df6563cef7d1"
meta-security        = "HEAD:982a29bbb7ef32475aea7c4bb56c620065a50927"
meta-perl            = "HEAD:cc6fc6b1641ab23089c1e3bba11e0c6394f0867c"
meta-security        
meta-app-framework   = "HEAD:cb90a88d382971a146c4b7413bb6df6563cef7d1"
meta                 
meta-poky            = "HEAD:4e931b1d05018923dc145cd97f6f965f5cb6e1a5"
meta-srikanth        = "<unknown>:<unknown>"

Initialising tasks: 100% |#######################################################################################################################################################################################################################################| Time: 0:00:00
Sstate summary: Wanted 0 Found 0 Missed 0 Current 0 (0% match, 0% complete)
NOTE: No setscene tasks
NOTE: Executing Tasks
do_build                              Default task for a recipe - depends on all other normal tasks required to 'build' a recipe
do_checkuri                           Validates the SRC_URI value
do_clean                              Removes all output files for a target
do_cleanall                           Removes all output files, shared state cache, and downloaded source files for a target
do_cleansstate                        Removes all output files and shared state cache for a target
do_compile                            Compiles the source in the compilation directory
do_configure                          Configures the source by enabling and disabling any build-time and configuration options for the software being built
do_deploy_source_date_epoch           
do_deploy_source_date_epoch_setscene   (setscene version)
do_devpyshell                         Starts an interactive Python shell for development/debugging
do_devshell                           Starts a shell with the environment set up for development/debugging
do_fetch                              Fetches the source code
do_install                            Copies files from the compilation directory to a holding area
do_listtasks                          Lists all defined tasks for a target
do_package                            Analyzes the content of the holding area and splits it into subsets based on available packages and files
do_package_qa                         Runs QA checks on packaged files
do_package_qa_setscene                Runs QA checks on packaged files (setscene version)
do_package_setscene                   Analyzes the content of the holding area and splits it into subsets based on available packages and files (setscene version)
do_package_write_rpm                  Creates the actual RPM packages and places them in the Package Feed area
do_package_write_rpm_setscene         Creates the actual RPM packages and places them in the Package Feed area (setscene version)
do_packagedata                        Creates package metadata used by the build system to generate the final packages
do_packagedata_setscene               Creates package metadata used by the build system to generate the final packages (setscene version)
do_patch                              Locates patch files and applies them to the source code
do_populate_lic                       Writes license information for the recipe that is collected later when the image is constructed
do_populate_lic_setscene              Writes license information for the recipe that is collected later when the image is constructed (setscene version)
do_populate_sysroot                   Copies a subset of files installed by do_install into the sysroot in order to make them available to other recipes
do_populate_sysroot_setscene          Copies a subset of files installed by do_install into the sysroot in order to make them available to other recipes (setscene version)
do_prepare_recipe_sysroot             
do_unpack                             Unpacks the source code into a working directory
NOTE: Tasks Summary: Attempted 1 tasks of which 0 didn't need to be rerun and all succeeded.

Summary: There was 1 WARNING message shown.
```

#### Listing tasks and its dependencies

```
bitbake -g helloworld

It creates task-dependendents.dot
```

#### Add the package to minimal image

```
Edit build/conf/local.conf file

-> Go to the end of this file and add the below line and save the file.
-> Make sure you have **space** before **helloworld** as shown below.

IMAGE_INSTALL_append = " helloworld"

```

#### Build the image now.
```
cd build
bitbake agl-image-minimal
```

```
cd tmp/deploy/images/qemux86-64/
runqemu bzImage agl-image-minimal-qemux86-64.ext4
```

```
It will open a new window and will ask for password: Enter "root"
```
```
now run the below command at the emulator's command prompt.
> helloworld
```
```
It will execute the helloworld "c" program and will print out the output.
```
