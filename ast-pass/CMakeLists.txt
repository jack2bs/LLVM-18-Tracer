# CMakeList for ClangTool obtained from:
# http://stackoverflow.com/questions/25782537/cmake-and-clang-tooling-linking-error-outside-of-source-tree

set(CMAKE_C_COMPILER gcc)
set(CMAKE_CXX_COMPILER g++)

link_directories($ENV{LLVM_HOME}/lib)
include_directories($ENV{LLVM_HOME}/include)

add_definitions(
  -D__STDC_LIMIT_MACROS
  -D__STDC_CONSTANT_MACROS
)

set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -fno-rtti -std=c++17")

set(SOURCE_FILES GetLabeledStmts.cpp)
add_executable(get-labeled-stmts ${SOURCE_FILES})
install(TARGETS get-labeled-stmts RUNTIME DESTINATION bin)

include(FindCurses)

# These libraries have complex dependencies, so easier to put in a rescan group
set (_CLANG_LIBS_TO_LINK_ 
  "$<LINK_GROUP:RESCAN,
  clangFrontend,
  clangSerialization,
  clangDriver,
  clangParse,
  clangSema,
  clangAnalysis,
  clangAST,
  clangBasic,
  clangSupport,
  clangEdit,
  clangLex,
  clangTooling,
  clangASTMatchers,
  clangAPINotes>"
)
string(REGEX REPLACE "[ \t\r\n]" "" _CLANG_LIBS_TO_LINK_ ${_CLANG_LIBS_TO_LINK_}) #Can't have whitespace


target_link_libraries(get-labeled-stmts

  PUBLIC

  ${_CLANG_LIBS_TO_LINK_}

  LLVMX86AsmParser # MC, MCParser, Support, X86Desc, X86Info
  LLVMX86Desc # MC, Support, X86AsmPrinter, X86Info
  #LLVMX86AsmPrinter # MC, Support, X86Utils
  LLVMX86Info # MC, Support, Target
  #LLVMX86Utils # Core, Support
  LLVMipo
  LLVMScalarOpts
  LLVMInstCombine
  LLVMTransformUtils
  LLVMAnalysis
  LLVMTarget
  LLVMOption # Support
  LLVMMCParser # MC, Support
  LLVMMC # Object, Support
  LLVMObject # BitReader, Core, Support
  LLVMBitReader # Core, Support
  LLVMCore # Support
  LLVMSupport
  LLVM-18

  pthread
  z
  dl
  ${CURSES_LIBRARIES}
  
)

target_link_options(get-labeled-stmts PRIVATE "LINKER:--version-script,/home/jack2bs/Documents/LLVM-Tracer/ast-pass/linker_script.txt")