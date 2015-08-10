# C Language Family Front-end

Welcome to Clang.  This is a compiler front-end for the C family of languages
(C, C++, Objective-C, and Objective-C++) which is built as part of the LLVM
compiler infrastructure project.

Unlike many other compiler frontends, Clang is useful for a number of things
beyond just compiling code: we intend for Clang to be host to a number of
different source-level tools.  One example of this is the Clang Static Analyzer.

If you're interested in more (including how to build Clang) it is best to read
the relevant web sites.  Here are some pointers:

Information on Clang:              http://clang.llvm.org/
Building and using Clang:          http://clang.llvm.org/get_started.html
Clang Static Analyzer:             http://clang-analyzer.llvm.org/
Information on the LLVM project:   http://llvm.org/

If you have questions or comments about Clang, a great place to discuss them is
on the Clang development mailing list:
  http://lists.cs.uiuc.edu/mailman/listinfo/cfe-dev

If you find a bug in Clang, please file it in the LLVM bug tracker:
  http://llvm.org/bugs/

### Compiling libclang with Xcode

(see <http://www.stuartcarnie.com/2012/06/llvm-clang-hacking-part-1.html> and <http://clang.llvm.org/get_started.html>)

1) git pull the latest versions of:

- llvm ($LLVM/llvm, cloned from http://llvm.org/git/llvm.git ), 
- compiler-rt ($LLVM/llvm/projects/compiler-rt, cloned from http://llvm.org/git/compiler-rt.git )
- clang ($LLVM/llvm/tools/clang, cloned from **this repo** or from http://llvm.org/git/clang.git )
	
2) Build LLVM and Clang

	> cd $LLVM/build
	> cmake -G "Unix Makefiles" ../llvm
	> make -j 8
	
Notes:

- `-j 8`: parallel compilation with 8 jobs simultaneously (adapt it for your machine CPU)
- If you get an error "fatal error: 'libxml/parser.h' file not found", make sure that libxml2 is available in /usr/include:

		> sudo ln -s /Applications/Xcode.app/Contents/Developer/Platforms/MacOSX.platform/Developer/SDKs/MacOSX10.9.sdk/usr/include/libxml2 /usr/include
		> make

3) Create an Xcode project for llvm

	> mkdir xcode-project
	> cd $LLVM/xcode-project
	> cmake -G Xcode ../llvm

4) Fix the Clang Xcode project

- set the base SDK to 'Latest OS X'
- add the libclang exported header files (the ones in $LLVM/llvm/tools/clang/include/clang-c) to the Clang Xcode Project
- in target libclang:
	- set dynamic Library install name to `@rpath/$(EXECUTABLE_PATH`
	- add a Copy Files build phase for the exported headers:  
	Destination: *Absolute Path*;  
	Subpath: `$(BUILT_PRODUCTS_DIR)/libclang/include/clang-c`;  
	Files: *CXString.h, Platform.h, Index.h, BuildSystem.h, CXCompilationDatabase.h, CXErrorCode.h, Documentation.h* 
	- add a Run Script build phase for copying the dynamic library: `cp -p ${SCRIPT_INPUT_FILE_0} ${SCRIPT_OUTPUT_FILE_0}`  
	  Input Files: `$(BUILT_PRODUCTS_DIR)/$(EXECUTABLE_PATH)`  
	  Output Files: `$(BUILT_PRODUCTS_DIR)/libclang/libclang.dylib`

5) Recompile libclang from Xcode

6) You will find libclang.dylib and the associated include files in `$LLVM/xcode-project/Release/libclang` (or in `$LLVM/xcode-project/Debug/libclang` if you have compiled in Debug mode)
