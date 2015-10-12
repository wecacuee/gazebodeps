# gazebodeps
Install gazebo dependencies from source (say if you dont have root privilleges on a cluster)

I wrote this script to install gazebo dependencies on bluewaters cluster from source. I am sharing this in hope that
someone may find it useful as I found the deps script written for OGRE3D. 

There are some patches in the repository to fix some of the problems I faced while installation. 
I had to disable the GUI part of gazebo because of proper QT support, you can enable it by changing the flag
NO_BUILD_GUI. Disabling GUI will disable scripts gazebo and tool gz as well.
