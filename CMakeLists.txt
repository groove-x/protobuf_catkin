cmake_minimum_required(VERSION 2.8.3)
project(protobuf_catkin)

find_package(catkin_simple REQUIRED)
catkin_simple()

include(ExternalProject)

file(MAKE_DIRECTORY ${CATKIN_DEVEL_PREFIX}/include)

ExternalProject_Add(protobuf_src
  URL https://github.com/protocolbuffers/protobuf/releases/download/v3.15.8/protobuf-all-3.15.8.tar.gz
  UPDATE_COMMAND ""
  CONFIGURE_COMMAND cd ../protobuf_src && ./autogen.sh && ./configure --with-pic --disable-static --prefix=${CATKIN_DEVEL_PREFIX}
  BUILD_COMMAND cd ../protobuf_src && make -j8 && cd python && python3 setup.py build --cpp_implementation --build-purelib build
  INSTALL_COMMAND cd ../protobuf_src && make install -j8 &&
                  cd python && python3 setup.py install --cpp_implementation --root ${CATKIN_DEVEL_PREFIX} --install-lib ${CATKIN_GLOBAL_PYTHON_DESTINATION}
)

install(DIRECTORY ${CATKIN_DEVEL_PREFIX}/include/google/protobuf
        DESTINATION ${CATKIN_GLOBAL_INCLUDE_DESTINATION}/google
        FILES_MATCHING PATTERN "*.h")
install(DIRECTORY ${CATKIN_DEVEL_PREFIX}/lib/
        DESTINATION ${CATKIN_PACKAGE_LIB_DESTINATION}
        FILES_MATCHING PATTERN "libproto*")
install(FILES ${CATKIN_DEVEL_PREFIX}/bin/protoc
        PERMISSIONS OWNER_READ OWNER_WRITE OWNER_EXECUTE GROUP_READ GROUP_EXECUTE WORLD_READ WORLD_EXECUTE
        DESTINATION ${CATKIN_GLOBAL_BIN_DESTINATION})
cs_export(INCLUDE_DIRS ${CATKIN_DEVEL_PREFIX}/include
          LIBRARIES protobuf protobuf-lite protoc
          CFG_EXTRAS protobuf-generate-cpp.cmake)
