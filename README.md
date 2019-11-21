# LG_v30-LineageOS_16.0_Manifest
Manifest needed to help build LineageOS 16.0 for LG v30

# Files
* README.md -- The self explanatory file you read now.
* lg_v30-h932-Stable.xml -- Manifest for building from the last known working build of LineageOS 16.0 FOR T-MOBILE H932 VARIANT ONLY. Code is provided from their respective GitHub sources listed in the file.
* lg_v30-h930-Stable.xml -- Manifest for building from the last known working build of LineageOS 16.0 UNIVERSAL BUILD EXECPT FOR T-MOBILE H932. Code is provided from their respective GitHub sources listed in the file.
* lg_v30-h932.xml -- Manifest for building from my source edits UNIVERSAL BUILD EXCEPT FOR T-MOBILE H932.
* lg_v30-h932.xml -- Manifest for building from my source edits FOR T-MOBILE VARIANT H932 ONLY.


# Notes for me to remember, should provide clues to how to build LOS.

### Dependencies
Ubuntu 18.04

sudo apt-get update && sudo apt-get upgrade && sudo apt-get dist-upgrade && sudo apt-get install openjdk-11-jdk

#### Ensure the correct Java is used
sudo update-alternatives --config java

sudo apt-get install bc bison build-essential ccache curl flex g++-multilib gcc-multilib git gnupg gperf imagemagick lib32ncurses5-dev lib32readline-dev lib32z1-dev liblz4-tool libncurses5-dev libsdl1.2-dev libssl-dev libwxgtk3.0-dev libxml2 libxml2-utils lzop pngcrush rsync schedtool squashfs-tools xsltproc zip zlib1g-dev python

### This sets up correct udev rules in Ubuntu so the system can connect to Android devices via USB, ADB, Fastboot.  Replace "username" with system username (Optional for build)
```
$ wget -S -O - http://source.android.com/source/51-android.rules | sed "s/<username>/$USER/" | sudo tee >/dev/null /etc/udev/rules.d/51-android.rules; sudo udevadm control --reload-rules
```
### Setup the "repo" binary. This helps download all the different repositories.
$ mkdir -p ~/bin  
$ curl https://storage.googleapis.com/git-repo-downloads/repo > ~/bin/repo  
$ chmod a+x ~/bin/repo

### Add the following to ~/.profile to ensure the above "repo" binary will always be accessible.
```
# set PATH so it includes user's private bin if it exists
if [ -d "$HOME/bin" ] ; then
    PATH="$HOME/bin:$PATH"
fi
```

### Create build directory and fetch the required LineageOS 16.0 source files. Make sure you replace the directory to the one you want to use for the source directory
$ mkdir -p ~/android/lineage 

$ cd ~/android/lineage  
$ repo init -u https://github.com/LineageOS/android.git -b lineage-16.0

### Now add the LG v30 manifest. Pick whether or not to use the stable manifest or not. MAKE SURE TO USE THE CORRECT MANIFEST! H932 is a T-Mobile specific build. H930 is universal and will work for other LG v30. 
### Change directory to point to .repo   folder of the source directory.
#### Make sure to point to the .repo folder inside of the build root location you choose earlier!
$ mkdir -p ~/android/lineage/.repo/local_manifests  
$ wget -S https://raw.githubusercontent.com/ShapeShifter499/LG_v30-LineageOS_Manifest/lineage-16.0/lg_v30-h930-Stable.xml -O ~/android/lineage/.repo/local_manifests/lg_v30-h930-Stable.xml

### Now grab all the source files! Run at the root of the source directory.
$ repo sync

### Change some git variables so the build correctly identifies who you are, run the following at the root of source. (Optional) 
$ git config --global user.name "Your Name"  
$ git config --global user.email "Your Email"

### Make sure the environment is setup
$ source build/envsetup.sh  

### Setup CCACHE to speed up things. Run from root of source.
$ export USE_CCACHE=1  
$ mkdir .ccache  
$ export CCACHE_DIR=.ccache  
$ ccache -M 50G

#### You can also enable the optional ccache compression. While this may involve a slight performance slowdown, it increases the number of files that fit in the cache. To enable also add this line (Optional)
$ export CCACHE_COMPRESS=1

#### Make sure CCACHE will always be used. Put the following in your .bashrc (or equivalent)
export USE_CCACHE=1

#### Optional compression also add to .bashrc 
export CCACHE_COMPRESS=1

#### Monitor CCACHE being used, run from root of source.
$ watch -n1 -d ccache -s

### SuperSU Root
Home builders that want to bake su back into the ROM can use the command ‘export WITH_SU=true’ prior to building.

### Make sure things are clean before build
$ mka clobber

### Now build! Run commands at root of source.
#### Swap the next line for lineage_h932-userdebug if compiling for T-Mobile H932, otherwise all other devices use H930
$ lunch lineage_h930-userdebug
$ mka bacon

#### To build just the boot.img
$ mka bootimage
