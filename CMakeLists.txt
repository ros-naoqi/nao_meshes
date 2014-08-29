cmake_minimum_required(VERSION 2.8)
project(nao_meshes)

find_package(catkin)
catkin_package()

# When getting a new binay, the MD5 might change, and the folder hierarchy too
# read comments below to adapt

set(TMP_DIR ${CATKIN_DEVEL_PREFIX}/tmp)

set(INSTALLER_TMP_PATH ${TMP_DIR}/tmp/installer.run)

if (${CMAKE_SYSTEM_PROCESSOR} STREQUAL "x86_64")
file(DOWNLOAD "https://github.com/ros-nao/nao_meshes_installer/raw/master/naomeshes-0.6.4-linux-x64-installer.run"
  ${INSTALLER_TMP_PATH} SHOW_PROGRESS
  INACTIVITY_TIMEOUT 60
  EXPECTED_MD5 6513ce0ca155ba03876ca2924e556203
)
else()
file(DOWNLOAD "https://github.com/ros-nao/nao_meshes_installer/raw/master/naomeshes-0.6.4-linux-installer.run"
  ${INSTALLER_TMP_PATH} SHOW_PROGRESS
  INACTIVITY_TIMEOUT 60
  EXPECTED_MD5 1804d6dbccf9cf3d5367ed44a5d3bb75
)
endif()

# specify the interesting folder in the archive. You might want to change
# that for a new binary where the structure is different
set(TMP_MESH_PATH ${TMP_DIR}/nao_meshes/meshes)

# final path in devel/install space
set(FINAL_MESH_PATH ${CATKIN_GLOBAL_SHARE_DESTINATION}/nao_description/meshes)

# make the file executable to install
file(COPY ${INSTALLER_TMP_PATH}
     FILE_PERMISSIONS OWNER_READ OWNER_WRITE OWNER_EXECUTE GROUP_READ GROUP_EXECUTE WORLD_READ WORLD_EXECUTE
     DESTINATION ${TMP_DIR}
)

set(INSTALLER_PATH ${TMP_DIR}/installer.run)

# add a target for devel space
add_custom_target(${PROJECT_NAME}_meshes
  COMMAND ${INSTALLER_PATH} --mode text --prefix ${TMP_DIR}
  COMMAND mkdir -p ${CATKIN_DEVEL_PREFIX}/${FINAL_MESH_PATH}
  COMMAND cp -fr ${TMP_MESH_PATH}/* ${CATKIN_DEVEL_PREFIX}/${FINAL_MESH_PATH}/
  COMMAND rm -fr ${TMP_DIR}/rollback*
)

# uncompress the meshes during install
install(CODE
"
# uncompress the archive
message(STATUS \"using ${INSTALLER_PATH} to decompress to ${TMP_DIR}\")
execute_process(COMMAND ${INSTALLER_PATH} --mode unattended --prefix ${TMP_DIR}
                OUTPUT_VARIABLE OUT
                ERROR_VARIABLE ERROR
                RESULT_VARIABLE RESULT
)
message(STATUS \"intall out: ${OUT}\")
message(STATUS \"install error: ${ERROR}\")
message(STATUS \"install result: ${RESULT}\")
"
)
# install the archive
install(DIRECTORY ${TMP_MESH_PATH}/
        DESTINATION ${FINAL_MESH_PATH}/
)
