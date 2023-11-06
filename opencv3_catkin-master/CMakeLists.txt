cmake_minimum_required(VERSION 2.8.3)
project(opencv3_catkin)

find_package(catkin_simple REQUIRED)

catkin_simple()

include(ExternalProject)

file(MAKE_DIRECTORY ${CATKIN_DEVEL_PREFIX}/include)

SET(CONTRIB_NAME "opencv3_contrib")
ExternalProject_Add(${CONTRIB_NAME}
  GIT_REPOSITORY https://github.com/Ws1234567879789/opencv_contrib-3.4.2.git
  SOURCE_DIR ${CONTRIB_NAME}_src
  UPDATE_COMMAND ""
  CONFIGURE_COMMAND ""
  BUILD_COMMAND ""
  INSTALL_COMMAND ""
)

SET(OPENCV_SRC_PATH "opencv3_src")
SET(OPENCV_BUILD_PATH "opencv3_build")
ExternalProject_Add(opencv3_src
  DEPENDS ${CONTRIB_NAME}
  GIT_REPOSITORY https://github.com/Ws1234567879789/opencv-3.4.2.git
  SOURCE_DIR ${OPENCV_SRC_PATH}
  BINARY_DIR ${OPENCV_BUILD_PATH}
  PATCH_COMMAND patch -p0 < ${CMAKE_SOURCE_DIR}/fix_python_discovery.patch &&
                patch -p1 < ${CMAKE_CURRENT_SOURCE_DIR}/rename_unordered_map_definition.patch
  CONFIGURE_COMMAND cd ../${OPENCV_BUILD_PATH} && cmake
    -DOPENCV_EXTRA_MODULES_PATH=../${CONTRIB_NAME}_src/modules .
    -DWITH_TBB=ON
    -DBUILD_opencv_python2=ON
    -DBUILD_opencv_python3=ON
    -DWITH_GTK_2_X=ON
    -DWITH_PROTOBUF=OFF
    # Rest of the configurations...
    # (Add or modify any configurations as required)
    # ...
  BUILD_COMMAND cd ../${OPENCV_BUILD_PATH} && make -j8
  INSTALL_COMMAND cd ../${OPENCV_BUILD_PATH} && make install -j8
)
install(DIRECTORY ${CATKIN_DEVEL_PREFIX}/include/opencv
        DESTINATION ${CATKIN_GLOBAL_INCLUDE_DESTINATION}
        FILES_MATCHING PATTERN "*.h"
                       PATTERN "*.hpp")
install(DIRECTORY ${CATKIN_DEVEL_PREFIX}/include/opencv2
        DESTINATION ${CATKIN_GLOBAL_INCLUDE_DESTINATION}
        FILES_MATCHING PATTERN "*.h"
                       PATTERN "*.hpp")
install(DIRECTORY ${CATKIN_DEVEL_PREFIX}/lib/
        DESTINATION ${CATKIN_PACKAGE_LIB_DESTINATION}
        FILES_MATCHING PATTERN "libopencv*")
install(DIRECTORY ${CATKIN_DEVEL_PREFIX}/share/OpenCV
        DESTINATION ${CATKIN_GLOBAL_SHARE_DESTINATION})
set(OPENCV_INSTALLED_EXECUTABLES
    opencv_annotation
    opencv_createsamples
    opencv_traincascade
    opencv_version
    opencv_visualisation
)

foreach(installed_file ${OPENCV_INSTALLED_EXECUTABLES})
  # Need to rename binaries as they will conflict with the binaries from
  # the ROS OpenCV package otherwise.
  string(REPLACE opencv opencv3_catkin new_file_name ${installed_file} )
  install(FILES ${CATKIN_DEVEL_PREFIX}/bin/${installed_file}
          RENAME ${new_file_name}
          PERMISSIONS OWNER_READ OWNER_WRITE OWNER_EXECUTE GROUP_READ GROUP_EXECUTE WORLD_READ WORLD_EXECUTE
          DESTINATION ${CATKIN_GLOBAL_BIN_DESTINATION})
endforeach()

cs_export(INCLUDE_DIRS ${CATKIN_DEVEL_PREFIX}/include
          LIBRARIES opencv_aruco
                    opencv_calib3d
                    opencv_core
                    opencv_features2d
                    opencv_flann
                    opencv_highgui
                    opencv_imgcodecs
                    opencv_imgproc
                    opencv_line_descriptor
                    opencv_ml
                    opencv_objdetect
                    opencv_phase_unwrapping
                    opencv_photo
                    opencv_rgbd
                    opencv_saliency
                    opencv_shape
                    opencv_stitching
                    opencv_superres
                    opencv_video
                    opencv_videoio
                    opencv_videostab
                    opencv_viz
                    opencv_xfeatures2d
                    opencv_ximgproc
)
