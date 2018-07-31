# How to compile telldus core on Ubuntu 18.04.

## Problem

Moving my tellstick to Ubuntu 18.04 LTS (Bionic Beaver) gave us a few hours of headache as the deb package from Telldus couldn't be installed as it requires libconfuse0 which isn't installable in 18.04. It seems that Telldus hasn't released any new package to fix this either so the only option left is to download the source and compile it. Well, compilation wasn't as straight forward as we thought since we got compilation errors. Not knowing the source code and not wanting to dive into it we thought that at one point in time it was possible to compile it so perhaps we could turn back time to when the source code was released (circa 2012)? We managed to tell make/cmake to use an older compilation standard (c++98) instead of the latest and greatest (which just threw errors).

The compilation went without errors or warnings and after importing the old tellstick.conf file we could turn devices on and off and we also saw sensor events. 

Below is the steps we followed to make it work. Good luck!

*This [link](https://forum.telldus.com/viewtopic.php?f=15&t=6201&sid=2958e789288f7d971a7515a474875c7f) was the closest to a solution we came wihout actually getting it to work so we had to be a bit creative.*

## Ubuntu 18.04 LTS (Bionic Beaver)

### Step 1 - Install prereqs
    apt-get install build-essential libftdi1 libftdi-dev libconfuse-common libconfuse-dev libconfuse2 cmake pkg-config

Note: On Debian 9 (latest as of 2018-07-31), change libconfuse2 to libfconfuse1.

### Step 2 - Download latest telldus core from Telldus

    mkdir /tmp/telldus
    cd /tmp/telldus
    wget http://download.telldus.com/TellStick/Software/telldus-core/telldus-core-2.1.2.tar.gz
    tar zxvf telldus-core-2.1.2.tar.gz
    cd telldus-core-2.1.2

### Step 3 - Remove the Doxygen part from the CMAKE file

Edit `CMakeLists.txt` and remove the following lines (at the end of the file)

    FIND_PACKAGE(Doxygen)
    IF(DOXYGEN_FOUND)
        SET(DOXY_CONFIG ${CMAKE_CURRENT_BINARY_DIR}/Doxyfile)
        CONFIGURE_FILE(
              "${CMAKE_CURRENT_SOURCE_DIR}/Doxyfile.in"
              ${DOXY_CONFIG} @ONLY
        )
        ADD_CUSTOM_TARGET(docs
                ${DOXYGEN_EXECUTABLE} ${DOXY_CONFIG}
                DEPENDS ${DOXY_CONFIG}
                WORKING_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}
                COMMENT "Generating doxygen documentation" VERBATIM
        )
    ENDIF()

### Step 4 - Add a compilation flag

Edit `CMakeLists.txt` and add (at the top of the file):

    set(CMAKE_CXX_FLAGS "-std=c++98")

### Step 5 - Run cmake

    cmake .

### Step 6 - Add -pthread for tdtool

Edit `tdtool/CMakeFiles/tdtool.dir/link.txt` and at the end of the line, add 

    -pthread

### Step 7 - Add -pthread for tdadmin

Edit `tdadmin/CMakeFiles/tdadmin.dir/link.txt` and att the end of the line, add

    -pthread

### Step 8 - Run make

    make

### Step 9 - Install

    make install
