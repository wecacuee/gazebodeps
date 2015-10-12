cmake_minimum_required(VERSION 2.8)
project(gazebodepsinstall)
include(ExternalProject)

set(INSTALL_PREFIX $ENV{HOME})
set(GAZEBO_VERSION gazebo4_4.1.3)
set(FREEIMAGE_VERSION 3.17.0)
set(THREAD_BUILDING_BLOCKS_VERSION 44_20150728)
set(OGRE3D_VERSION 1.8.1)
set(SDFORMAT_VERSION sdformat2_2.3.2)
set(IGNITION_MATH_VERSION ignition-math2_2.2.3)

execute_process(COMMAND which ruby OUTPUT_VARIABLE RUBYPATH)
string(STRIP ${RUBYPATH} RUBYPATH)


ExternalProject_Add(gazebosrc
      PREFIX gazebo
      CMAKE_COMMAND CC=gcc CXX=g++ cmake -DCMAKE_INSTALL_PREFIX:string=${INSTALL_PREFIX} -DNO_BUILD_GUI:bool=True
      BUILD_COMMAND CPATH=$ENV{BOOST_ROOT}/include:$ENV{CPATH} make
      PATCH_COMMAND patch -p1 < ${CMAKE_CURRENT_LIST_DIR}/gazebo.src.4.1.3.patch
      HG_REPOSITORY https://bitbucket.org/osrf/gazebo
      HG_TAG ${GAZEBO_VERSION}
    )


string(REPLACE "." "" FREEIMAGE_SHORT_VERSION ${FREEIMAGE_VERSION})
ExternalProject_Add(freeimage
      PREFIX freeimage
      URL http://iweb.dl.sourceforge.net/project/freeimage/Source%20Distribution/${FREEIMAGE_VERSION}/FreeImage${FREEIMAGE_SHORT_VERSION}.zip
      BUILD_IN_SOURCE 1
      PATCH_COMMAND patch -u Makefile.gnu ${CMAKE_CURRENT_LIST_DIR}/freeimage.Makefile.gnu.patch
      CONFIGURE_COMMAND echo configured
      BUILD_COMMAND make INSTALLDIR=${INSTALL_PREFIX}/lib INCDIR=${INSTALL_PREFIX}/include
      INSTALL_COMMAND make install INSTALLDIR=${INSTALL_PREFIX}/lib INCDIR=${INSTALL_PREFIX}/include
    )


ExternalProject_Add(tinyxml
      PREFIX tinyxml
      GIT_REPOSITORY git://git.code.sf.net/p/tinyxml/git 
      BUILD_IN_SOURCE 1
      PATCH_COMMAND patch -u tinyxml.h ${CMAKE_CURRENT_LIST_DIR}/tinyxmldefaultstlpatch.patch
      CONFIGURE_COMMAND python scons.py sharedlibrary=True
      INSTALL_COMMAND cp tinystr.h tinyxml.h ${INSTALL_PREFIX}/include/ && cp libtinyxml.so ${INSTALL_PREFIX}/lib/
    )

ExternalProject_Add_Step(tinyxml sconslocal
    COMMAND wget http://iweb.dl.sourceforge.net/project/scons/scons-local/2.4.0/scons-local-2.4.0.tar.gz
    COMMAND tar xzf scons-local-2.4.0.tar.gz
    COMMENT "Downloading scons ..."
    DEPENDEES patch
    DEPENDERS configure
    WORKING_DIRECTORY <SOURCE_DIR>
    LOG 1
    )

ExternalProject_Add(protobuf
      PREFIX proto
      GIT_REPOSITORY https://github.com/google/protobuf.git 
      BUILD_IN_SOURCE 1
      CONFIGURE_COMMAND ./autogen.sh && ./configure --prefix=${INSTALL_PREFIX}
    )

ExternalProject_Add(libtar
      PREFIX libtar
      GIT_REPOSITORY http://repo.or.cz/libtar.git
      BUILD_IN_SOURCE 1
      CONFIGURE_COMMAND autoreconf --force --install && ./configure --prefix=${INSTALL_PREFIX}
    )

ExternalProject_Add(tbb
      PREFIX tbb
      DOWNLOAD_COMMAND wget https://www.threadingbuildingblocks.org/sites/default/files/software_releases/linux/tbb${THREAD_BUILDING_BLOCKS_VERSION}oss_lin_0.tgz -O tbb.tgz && tar xzf tbb.tgz && rsync -rv tbb${THREAD_BUILDING_BLOCKS_VERSION}oss/ tbb/
      BUILD_IN_SOURCE 1
      CONFIGURE_COMMAND echo configured
      BUILD_COMMAND echo build
      INSTALL_COMMAND rsync -vr bin/ ${INSTALL_PREFIX}/bin/ && rsync -vr include/ ${INSTALL_PREFIX}/include/ && rsync -vr lib/intel64/gcc4.4 ${INSTALL_PREFIX}/lib/ && rsync -vr lib/mic ${INSTALL_PREFIX}/lib/
    )

ExternalProject_Add(ogredeps
    PREFIX ogredeps
    HG_REPOSITORY https://bitbucket.org/cabalistic/ogredeps
    CMAKE_CACHE_ARGS -DCMAKE_INSTALL_PREFIX=${INSTALL_PREFIX}
    )

execute_process(COMMAND which g++ OUTPUT_VARIABLE ENVGCXX)
string(STRIP ${ENVGCXX} ENVGCXX)
execute_process(COMMAND which gcc OUTPUT_VARIABLE ENVGCC)
string(STRIP ${ENVGCC} ENVGCC)
string(REPLACE "." "-" OGRE3D_HYPHEN_VERSION ${OGRE3D_VERSION})
string(REGEX_REPLACE "\.\d$" "" OGRE3D_MAJOR_VERSION ${OGRE3D_VERSION})
ExternalProject_Add(ogre3d
    PREFIX ogre3d
    CMAKE_COMMAND CXX=${ENVGCXX} CC=${ENVGCC} cmake -DCMAKE_INSTALL_PREFIX=${INSTALL_PREFIX}
    URL http://superb-dca2.dl.sourceforge.net/project/ogre/ogre/${OGRE3D_MAJOR_VERSION}/${OGRE3D_VERSION}/ogre_src_v${OGRE3D_HYPHEN_VERSION}.tar.bz2
   ) 

add_dependencies(ogre3d ogredeps)

ExternalProject_Add(ignitionmath
    PREFIX ignmath
    CMAKE_COMMAND CC=gcc CXX=g++ cmake -DCMAKE_INSTALL_PREFIX:string=${INSTALL_PREFIX}
    HG_REPOSITORY https://bitbucket.org/ignitionrobotics/ign-math
    HG_TAG ${IGNITION_MATH_VERSION}
    )
ExternalProject_Add(sdfformat
    PREFIX sdfformat
    PATCH_COMMAND patch -u cmake/SearchForStuff.cmake ${CMAKE_CURRENT_LIST_DIR}/sdfformat.cmake.SearchForStuff.cmake.patch
    CMAKE_COMMAND CC=gcc CXX=g++ cmake -DCMAKE_INSTALL_PREFIX:string=${INSTALL_PREFIX} -DRUBY:filepath=${RUBYPATH}
    HG_REPOSITORY https://bitbucket.org/osrf/sdformat
    HG_TAG ${SDFORMAT_VERSION}
    )

add_dependencies(sdfformat ignitionmath tinyxml)

add_dependencies(gazebosrc freeimage protobuf libtar tinyxml tbb ogre3d
   sdformat)
