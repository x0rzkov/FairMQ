################################################################################
# Copyright (C) 2012-2018 GSI Helmholtzzentrum fuer Schwerionenforschung GmbH  #
#                                                                              #
#              This software is distributed under the terms of the             #
#              GNU Lesser General Public Licence (LGPL) version 3,             #
#                  copied verbatim in the file "LICENSE"                       #
################################################################################

if(BUILD_FAIRMQ OR BUILD_SDK)
  ###########
  # Version #
  ###########
  configure_file(Version.h.in
    ${CMAKE_BINARY_DIR}/${PROJECT_NAME_LOWER}/Version.h
    @ONLY
  )
  install(FILES ${CMAKE_BINARY_DIR}/${PROJECT_NAME_LOWER}/Version.h
    DESTINATION ${PROJECT_INSTALL_INCDIR}
  )

  #########
  # Tools #
  #########
  set(target Tools)

  set(TOOLS_PUBLIC_HEADER_FILES
    tools/CppSTL.h
    tools/InstanceLimit.h
    tools/Network.h
    tools/Process.h
    tools/RateLimit.h
    tools/Semaphore.h
    tools/Strings.h
    tools/Unique.h
    tools/Version.h
    Tools.h
  )

  set(TOOLS_SOURCE_FILES
    tools/Network.cxx
    tools/Process.cxx
    tools/Semaphore.cxx
    tools/Unique.cxx
  )

  add_library(${target}
    ${TOOLS_SOURCE_FILES}
    ${TOOLS_PUBLIC_HEADER_FILES}
  )
  target_compile_definitions(${target} PUBLIC BOOST_ERROR_CODE_HEADER_ONLY)
  # workaround https://github.com/boostorg/asio/commit/43874d5497414c67655d901e48c939ef01337edb
  if(    Boost_VERSION VERSION_LESS 1.69
     AND CMAKE_CXX_COMPILER_ID STREQUAL AppleClang
     AND CMAKE_CXX_COMPILER_VERSION VERSION_GREATER_EQUAL 10.0.1)
    target_compile_definitions(${target} PUBLIC BOOST_ASIO_HAS_STD_STRING_VIEW)
  endif()
  target_include_directories(${target}
    PUBLIC
    $<BUILD_INTERFACE:${CMAKE_SOURCE_DIR}>
    $<INSTALL_INTERFACE:include>
  )
  target_link_libraries(${target}
    PRIVATE
    FairLogger::FairLogger
    PUBLIC
    Boost::boost
  )
  set_target_properties(${target} PROPERTIES
    VERSION ${PROJECT_VERSION}
    SOVERSION "${PROJECT_VERSION_MAJOR}.${PROJECT_VERSION_MINOR}"
    OUTPUT_NAME FairMQ${target}
  )
  install(
    TARGETS ${target}
    EXPORT ${PROJECT_EXPORT_SET}
    RUNTIME DESTINATION ${PROJECT_INSTALL_BINDIR}
    LIBRARY DESTINATION ${PROJECT_INSTALL_LIBDIR}
    ARCHIVE DESTINATION ${PROJECT_INSTALL_LIBDIR}
  )
  foreach(HEADER ${TOOLS_PUBLIC_HEADER_FILES})
    get_filename_component(_path ${HEADER} DIRECTORY)
    file(TO_CMAKE_PATH ${PROJECT_INSTALL_INCDIR}/${_path} _destination)
    install(FILES ${HEADER}
      DESTINATION ${_destination}
    )
  endforeach()

  #################
  # State Machine #
  #################
  set(target StateMachine)

  set(FSM_PUBLIC_HEADER_FILES
    StateMachine.h
    States.h
    StateQueue.h
  )

  set(FSM_SOURCE_FILES
    StateMachine.cxx
    States.cxx
  )

  add_library(${target}
    ${FSM_SOURCE_FILES}
    ${FSM_PUBLIC_HEADER_FILES}
  )
  target_compile_definitions(${target} PUBLIC BOOST_ERROR_CODE_HEADER_ONLY)
  target_include_directories(${target}
    PUBLIC
    $<BUILD_INTERFACE:${CMAKE_SOURCE_DIR}>
    $<INSTALL_INTERFACE:include>
  )
  target_link_libraries(${target}
    PUBLIC
    FairLogger::FairLogger

    PRIVATE
    Boost::boost
    Tools
  )
  set_target_properties(${target} PROPERTIES
    VERSION ${PROJECT_VERSION}
    SOVERSION "${PROJECT_VERSION_MAJOR}.${PROJECT_VERSION_MINOR}"
    OUTPUT_NAME FairMQ${target}
  )
  install(
    TARGETS ${target}
    EXPORT ${PROJECT_EXPORT_SET}
    RUNTIME DESTINATION ${PROJECT_INSTALL_BINDIR}
    LIBRARY DESTINATION ${PROJECT_INSTALL_LIBDIR}
    ARCHIVE DESTINATION ${PROJECT_INSTALL_LIBDIR}
  )
  foreach(HEADER ${FSM_PUBLIC_HEADER_FILES})
    get_filename_component(_path ${HEADER} DIRECTORY)
    file(TO_CMAKE_PATH ${PROJECT_INSTALL_INCDIR}/${_path} _destination)
    install(FILES ${HEADER}
      DESTINATION ${_destination}
    )
  endforeach()
endif()

if(BUILD_FAIRMQ)
  ##########################
  # libFairMQ header files #
  ##########################
  set(FAIRMQ_PUBLIC_HEADER_FILES
    DeviceRunner.h
    EventManager.h
    FairMQChannel.h
    FairMQDevice.h
    FairMQLogger.h
    FairMQMessage.h
    FairMQParts.h
    FairMQPoller.h
    FairMQUnmanagedRegion.h
    FairMQSocket.h
    FairMQTransportFactory.h
    MemoryResources.h
    MemoryResourceTools.h
    Transports.h
    options/FairMQProgOptions.h
    JSONParser.h
    ProgOptionsFwd.h
    ProgOptions.h
    Properties.h
    PropertyOutput.h
    SuboptParser.h
    Plugin.h
    PluginManager.h
    PluginServices.h
    runFairMQDevice.h
    shmem/Monitor.h
  )

  set(FAIRMQ_PRIVATE_HEADER_FILES
    plugins/Builtin.h
    plugins/config/Config.h
    plugins/Control.h
    shmem/Message.h
    shmem/Poller.h
    shmem/UnmanagedRegion.h
    shmem/Socket.h
    shmem/TransportFactory.h
    shmem/Common.h
    shmem/Manager.h
    shmem/Region.h
    zeromq/Context.h
    zeromq/Message.h
    zeromq/Poller.h
    zeromq/UnmanagedRegion.h
    zeromq/Socket.h
    zeromq/TransportFactory.h
  )

  if(BUILD_OFI_TRANSPORT)
    set(FAIRMQ_PRIVATE_HEADER_FILES ${FAIRMQ_PRIVATE_HEADER_FILES}
      ofi/Context.h
      ofi/Message.h
      ofi/Poller.h
      ofi/Socket.h
      ofi/TransportFactory.h
    )
  endif()

  ##########################
  # libFairMQ source files #
  ##########################
  set(FAIRMQ_SOURCE_FILES
    DeviceRunner.cxx
    FairMQChannel.cxx
    FairMQDevice.cxx
    FairMQLogger.cxx
    FairMQMessage.cxx
    FairMQPoller.cxx
    FairMQSocket.cxx
    FairMQTransportFactory.cxx
    Plugin.cxx
    PluginManager.cxx
    PluginServices.cxx
    ProgOptions.cxx
    JSONParser.cxx
    Properties.cxx
    SuboptParser.cxx
    plugins/config/Config.cxx
    plugins/Control.cxx
    MemoryResources.cxx
    shmem/Monitor.cxx
  )

  if(BUILD_OFI_TRANSPORT)
    set(FAIRMQ_SOURCE_FILES ${FAIRMQ_SOURCE_FILES}
      ofi/Context.cxx
      ofi/Message.cxx
      ofi/Poller.cxx
      ofi/Socket.cxx
      ofi/TransportFactory.cxx
    )
  endif()


  ###################
  # configure files #
  ###################
  set(FAIRMQ_BIN_DIR ${CMAKE_BINARY_DIR}/fairmq)
  configure_file(${CMAKE_CURRENT_SOURCE_DIR}/run/startMQBenchmark.sh.in ${CMAKE_CURRENT_BINARY_DIR}/startMQBenchmark.sh)

  #################################
  # define libFairMQ build target #
  #################################
  if(FAST_BUILD)
    set(_target FairMQ_)
  else()
    set(_target FairMQ)
  endif()
  add_library(${_target}
    ${FAIRMQ_SOURCE_FILES}
    ${FAIRMQ_PUBLIC_HEADER_FILES} # for IDE integration
    ${FAIRMQ_PRIVATE_HEADER_FILES} # for IDE integration
  )
  set_target_properties(${_target} PROPERTIES LABELS coverage)
  if(FAST_BUILD)
    set_target_properties(${_target} PROPERTIES OUTPUT_NAME FairMQ)
  endif()


  ############################
  # preprocessor definitions #
  ############################
  target_compile_definitions(${_target} PUBLIC BOOST_ERROR_CODE_HEADER_ONLY)
  if(FAIRMQ_DEBUG_MODE)
    target_compile_definitions(${_target} PUBLIC FAIRMQ_DEBUG_MODE)
  endif()
  if(BUILD_OFI_TRANSPORT)
    target_compile_definitions(${_target} PRIVATE BUILD_OFI_TRANSPORT)
  endif()


  #######################
  # include directories #
  #######################
  target_include_directories(${_target}
    PUBLIC # consumers inherit public include directories
    $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}>
    $<BUILD_INTERFACE:${CMAKE_SOURCE_DIR}>
    $<BUILD_INTERFACE:${CMAKE_BINARY_DIR}>
    $<INSTALL_INTERFACE:include/fairmq>
    $<INSTALL_INTERFACE:include>
  )

  ##################
  # link libraries #
  ##################
  if(BUILD_OFI_TRANSPORT)
    set(OFI_DEPS
      asiofi::asiofi
      Boost::container
    )
  endif()
  set(optional_deps ${OFI_DEPS})
  if(optional_deps)
    list(REMOVE_DUPLICATES optional_deps)
  endif()

  target_link_libraries(${_target}
    INTERFACE # only consumers link against interface dependencies
    Boost::container

    PUBLIC # libFairMQ AND consumers of libFairMQ link aginst public dependencies
    Threads::Threads
    dl
    $<$<PLATFORM_ID:Linux>:rt>
    Boost::boost
    Boost::program_options
    Boost::filesystem
    Boost::regex
    FairLogger::FairLogger
    Tools
    StateMachine

    PRIVATE # only libFairMQ links against private dependencies
    libzmq
    PicoSHA2
    ${OFI_DEPS}
  )
  set_target_properties(${_target} PROPERTIES
    VERSION ${PROJECT_VERSION}
    SOVERSION "${PROJECT_VERSION_MAJOR}.${PROJECT_VERSION_MINOR}"
  )


  ##############
  # fast build #
  ##############
  if(FAST_BUILD)
    set_target_properties(${_target} PROPERTIES
      COTIRE_UNITY_TARGET_NAME "FairMQ"
      # COTIRE_ENABLE_PRECOMPILED_HEADER FALSE
      EXCLUDE_FROM_ALL TRUE
    )
    cotire(${_target})
    set_target_properties(FairMQ PROPERTIES EXCLUDE_FROM_ALL FALSE)
    set_target_properties(FairMQ PROPERTIES LABELS coverage)
  endif()


  ###############
  # executables #
  ###############
  add_executable(fairmq-bsampler run/runBenchmarkSampler.cxx)
  target_link_libraries(fairmq-bsampler FairMQ)

  add_executable(fairmq-merger run/runMerger.cxx)
  target_link_libraries(fairmq-merger FairMQ)

  add_executable(fairmq-multiplier run/runMultiplier.cxx)
  target_link_libraries(fairmq-multiplier FairMQ)

  add_executable(fairmq-proxy run/runProxy.cxx)
  target_link_libraries(fairmq-proxy FairMQ)

  add_executable(fairmq-sink run/runSink.cxx)
  target_link_libraries(fairmq-sink FairMQ)

  add_executable(fairmq-splitter run/runSplitter.cxx)
  target_link_libraries(fairmq-splitter FairMQ)

  add_executable(fairmq-shmmonitor shmem/Monitor.cxx shmem/Monitor.h shmem/runMonitor.cxx)
  target_compile_definitions(fairmq-shmmonitor PUBLIC BOOST_ERROR_CODE_HEADER_ONLY)
  if(FAIRMQ_DEBUG_MODE)
    target_compile_definitions(fairmq-shmmonitor PUBLIC FAIRMQ_DEBUG_MODE)
  endif()
  target_link_libraries(fairmq-shmmonitor PUBLIC
    Threads::Threads
    $<$<PLATFORM_ID:Linux>:rt>
    Boost::boost
    Boost::date_time
    Boost::program_options
    FairLogger::FairLogger
    PicoSHA2
  )
  target_include_directories(fairmq-shmmonitor PUBLIC
    $<BUILD_INTERFACE:${CMAKE_SOURCE_DIR}>
  )

  add_executable(fairmq-uuid-gen run/runUuidGenerator.cxx)
  target_link_libraries(fairmq-uuid-gen FairMQ)


  ###########
  # install #
  ###########
  install(
    TARGETS
    FairMQ
    fairmq-bsampler
    fairmq-merger
    fairmq-multiplier
    fairmq-proxy
    fairmq-sink
    fairmq-splitter
    fairmq-shmmonitor
    fairmq-uuid-gen

    EXPORT ${PROJECT_EXPORT_SET}
    RUNTIME DESTINATION ${PROJECT_INSTALL_BINDIR}
    LIBRARY DESTINATION ${PROJECT_INSTALL_LIBDIR}
    ARCHIVE DESTINATION ${PROJECT_INSTALL_LIBDIR}
  )

  # preserve relative path and prepend fairmq
  foreach(HEADER ${FAIRMQ_PUBLIC_HEADER_FILES})
    get_filename_component(_path ${HEADER} DIRECTORY)
    file(TO_CMAKE_PATH ${PROJECT_INSTALL_INCDIR}/${_path} _destination)
    install(FILES ${HEADER}
      DESTINATION ${_destination}
    )
  endforeach()
endif()

if(BUILD_SDK_COMMANDS)
  add_subdirectory(sdk/commands)
endif()

if(BUILD_SDK)
  add_subdirectory(sdk)
endif()

####################
# external plugins #
####################
if(BUILD_DDS_PLUGIN)
  add_subdirectory(plugins/DDS)
endif()
if(BUILD_PMIX_PLUGIN)
  add_subdirectory(plugins/PMIx)
endif()
