cmake_minimum_required(VERSION 3.18...3.18 FATAL_ERROR)

project(CppUnit VERSION 1.15.1)

# Generates include/cppunit/config-auto.h
# This is originally done by autoconf

include(CheckIncludeFile)
include(CheckIncludeFileCXX)
include(CheckCXXSourceCompiles)
include(CheckCSourceCompiles)
include(CheckLibraryExists)
include(CheckFunctionExists)

#Not used == not seen in any *.h *.cpp files
#Not used FUNC_STRING_COMPARE_STRING_FIRST

check_include_file_cxx(sstream CPPUNIT_HAVE_SSTREAM)
check_include_file_cxx(strstream CPPUNIT_HAVE_STRSTREAM)
set (CMAKE_REQUIRED_DEFINITIONS -DHAVE_STRSTREAM=CPPUNIT_HAVE_STRSTREAM)
check_cxx_source_compiles(
"#ifdef HAVE_STRSTREAM
#include <strstream>
#else
#include <strstream.h>
#endif
int main() {
  std::ostrstream message;
  message << \"Hello\";
  return 0;
}" CPPUNIT_HAVE_CLASS_STRSTREAM)

check_include_file_cxx(cmath CPPUNIT_HAVE_CMATH)
#Not used, dld library is obsolete anyway HAVE_DLD
#Not used HAVE_DLERROR
check_include_file(dlfcn.h CPPUNIT_HAVE_DLFCN_H)

check_c_source_compiles(
"#include <math.h>
int main() {
  return finite(3);
}" CPPUNIT_HAVE_FINITE)

check_c_source_compiles(
"#include <math.h>
int main() {
  return _finite(3);
}" CPPUNIT_HAVE__FINITE)

check_include_file_cxx(cxxabi.h CPPUNIT_HAVE_GCC_ABI_DEMANGLE)
#Not used HAVE_INTTYPES_H

check_c_source_compiles(
"#include <math.h>
int main() {
  return isfinite(3);
}" CPPUNIT_HAVE_ISFINITE)

check_library_exists(dl dlopen "" CPPUNIT_HAVE_LIBDL)
#Not used HAVE_MEMORY_H

check_cxx_source_compiles(
"namespace Outer {
  namespace Inner {
    int i = 0;
  }
}
using namespace Outer::Inner;
int main() {
  return i;
}" CPPUNIT_HAVE_NAMESPACES)

check_cxx_source_compiles(
"#include <typeinfo>
class Base {
public:
  Base() {}
  virtual int f() { return 0; }
};
class Derived : public Base {
public:
  Derived() {}
  virtual int f() { return 1; }
};
int main() {
  Derived d;
  Base * ptr = &d;
  return typeid(*ptr) == typeid(Derived);
}" CPPUNIT_HAVE_RTTI)

check_library_exists(dl shl_load "" CPPUNIT_HAVE_SHL_LOAD)

#Not used HAVE_STDINT_H
#Not used HAVE_STDLIB_H
#Not used HAVE_STRINGS_H
#Not used HAVE_STRING_H
#Not used HAVE_SYS_STAT_H
#Not used HAVE_SYS_TYPES_H
#Not used HAVE_UNISTD_H
#Not used PACKAGE
#Not used PACKAGE_BUGREPORT
#Not used PACKAGE_NAME
#Not used PACKAGE_STRING
#Not used PACKAGE_TARNAME
#Not used PACKAGE_VERSION
#Not used STDC_HEADERS
check_include_file_cxx(typeinfo CPPUNIT_USE_TYPEINFO_NAME)
#CPPUNIT_VERSION ok

configure_file(config/config.h.cmake ${CMAKE_CURRENT_SOURCE_DIR}/include/cppunit/config-auto.h)
##


set(cppunit_SRCS
  src/cppunit/AdditionalMessage.cpp
  src/cppunit/Asserter.cpp
  src/cppunit/BriefTestProgressListener.cpp
  src/cppunit/CompilerOutputter.cpp
  src/cppunit/DefaultProtector.h
  src/cppunit/DefaultProtector.cpp
  src/cppunit/DynamicLibraryManager.cpp
  src/cppunit/DynamicLibraryManagerException.cpp
  src/cppunit/Exception.cpp
  src/cppunit/Message.cpp
  src/cppunit/PlugInManager.cpp
  src/cppunit/PlugInParameters.cpp
  src/cppunit/Protector.cpp
  src/cppunit/ProtectorChain.h
  src/cppunit/ProtectorContext.h
  src/cppunit/ProtectorChain.cpp
  src/cppunit/RepeatedTest.cpp
  src/cppunit/ShlDynamicLibraryManager.cpp
  src/cppunit/SourceLine.cpp
  src/cppunit/StringTools.cpp
  src/cppunit/SynchronizedObject.cpp
  src/cppunit/Test.cpp
  src/cppunit/TestAssert.cpp
  src/cppunit/TestCase.cpp
  src/cppunit/TestCaseDecorator.cpp
  src/cppunit/TestComposite.cpp
  src/cppunit/TestDecorator.cpp
  src/cppunit/TestFactoryRegistry.cpp
  src/cppunit/TestFailure.cpp
  src/cppunit/TestLeaf.cpp
  src/cppunit/TestNamer.cpp
  src/cppunit/TestPath.cpp
  src/cppunit/TestPlugInDefaultImpl.cpp
  src/cppunit/TestResult.cpp
  src/cppunit/TestResultCollector.cpp
  src/cppunit/TestRunner.cpp
  src/cppunit/TestSetUp.cpp
  src/cppunit/TestSuccessListener.cpp
  src/cppunit/TestSuite.cpp
  src/cppunit/TestSuiteBuilderContext.cpp
  src/cppunit/TextOutputter.cpp
  src/cppunit/TextTestProgressListener.cpp
  src/cppunit/TextTestResult.cpp
  src/cppunit/TextTestRunner.cpp
  src/cppunit/TypeInfoHelper.cpp
  src/cppunit/UnixDynamicLibraryManager.cpp
  src/cppunit/Win32DynamicLibraryManager.cpp
  src/cppunit/XmlDocument.cpp
  src/cppunit/XmlElement.cpp
  src/cppunit/XmlOutputter.cpp
  src/cppunit/XmlOutputterHook.cpp
)

add_library(cppunit SHARED ${cppunit_SRCS})

target_include_directories(cppunit
  PUBLIC "$<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include>"
         "$<INSTALL_INTERFACE:include>"
)

target_compile_definitions(cppunit PRIVATE CPPUNIT_BUILD_DLL)

set_target_properties(cppunit PROPERTIES
  DEBUG_POSTFIX d
  SOVERSION ${PROJECT_VERSION_MAJOR}
  VERSION ${PROJECT_VERSION}
)

set(${PROJECT_NAME}_LIBRARIES cppunit)

install(TARGETS ${${PROJECT_NAME}_LIBRARIES} EXPORT ${PROJECT_NAME}_TARGETS
  LIBRARY DESTINATION lib
  ARCHIVE DESTINATION lib
  RUNTIME DESTINATION bin
)

install(DIRECTORY include/cppunit
  DESTINATION include
)

configure_file(
  ${PROJECT_NAME}Config.cmake.in
  ${CMAKE_CURRENT_BINARY_DIR}/${PROJECT_NAME}Config.cmake
  @ONLY
)
configure_file(
  ${CMAKE_CURRENT_SOURCE_DIR}/${PROJECT_NAME}ConfigVersion.cmake.in
  ${CMAKE_CURRENT_BINARY_DIR}/${PROJECT_NAME}ConfigVersion.cmake
  @ONLY
)

export(EXPORT ${PROJECT_NAME}_TARGETS
  FILE ${CMAKE_CURRENT_BINARY_DIR}/${PROJECT_NAME}Targets.cmake
)

set(config_package_location lib/cmake/${PROJECT_NAME})

install(EXPORT ${PROJECT_NAME}_TARGETS
  FILE ${PROJECT_NAME}Targets.cmake
  DESTINATION ${config_package_location}
)

install(FILES
    "${CMAKE_CURRENT_BINARY_DIR}/${PROJECT_NAME}Config.cmake"
    "${CMAKE_CURRENT_BINARY_DIR}/${PROJECT_NAME}ConfigVersion.cmake"
  DESTINATION ${config_package_location}
)
