# Copyright (C) 2008-2010 Trinity <http://www.trinitycore.org/>
#
# Copyright (C) 2009-2010 Oregon <http://www.oregoncore.org/>
#
# This file is free software; as a special exception the author gives
# unlimited permission to copy and/or distribute it, with or without
# modifications, as long as this notice is preserved.
#
# This program is distributed in the hope that it will be useful, but
# WITHOUT ANY WARRANTY, to the extent permitted by law; without even the
# implied warranty of MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.

file(GLOB sources_localdir *.cpp *.h)

set(oregon-auth_SRCS
  ${oregon-auth_SRCS}
  ${sources_localdir}
)

if( WIN32 )
  set(oregon-auth_SRCS
    ${oregon-auth_SRCS}
    ${sources_Debugging}
    OregonAuth.rc
  )
endif()

include_directories(
  ${CMAKE_BINARY_DIR}
  ${CMAKE_SOURCE_DIR}/src/shared
  ${CMAKE_SOURCE_DIR}/src/shared/Database
  ${CMAKE_SOURCE_DIR}/src/framework
  ${ACE_INCLUDE_DIR}
  ${MYSQL_INCLUDE_DIR}
  ${OPENSSL_INCLUDE_DIR}
)

set(oregon-auth_LINK_FLAGS "")

add_executable(oregon-auth
  ${oregon-auth_SRCS}
)

add_dependencies(oregon-auth revision.h)

if( NOT WIN32 )
  add_definitions(-D_oregon-auth_CONFIG="${CONF_DIR}/oregonauth.conf")
endif()

if( UNIX )
  set(oregon-auth_LINK_FLAGS "-pthread ${oregon-auth_LINK_FLAGS}")
endif()

if( CMAKE_SYSTEM_NAME MATCHES "Darwin" )
  set(oregon-auth_LINK_FLAGS "-framework Carbon ${oregon-auth_LINK_FLAGS}")
endif()

set_target_properties(oregon-auth PROPERTIES LINK_FLAGS "${oregon-auth_LINK_FLAGS}")

target_link_libraries(oregon-auth
  shared
  oregonframework
  ${MYSQL_LIBRARY}
  ${OPENSSL_LIBRARIES}
  ${OPENSSL_EXTRA_LIBRARIES}
  ${OSX_LIBS}
)

if( WIN32 )
  add_custom_command(TARGET oregon-auth
    POST_BUILD
    COMMAND ${CMAKE_COMMAND} -E copy ${CMAKE_CURRENT_SOURCE_DIR}/oregonauth.conf.dist ${CMAKE_BINARY_DIR}/bin/$(ConfigurationName)/
  )
endif()

if( UNIX )
  install(TARGETS oregon-auth DESTINATION bin)
  install(FILES oregonauth.conf.dist DESTINATION etc)
elseif( WIN32 )
  install(TARGETS oregon-auth DESTINATION "${CMAKE_INSTALL_PREFIX}")
  install(FILES oregonauth.conf.dist DESTINATION "${CMAKE_INSTALL_PREFIX}")
endif()
