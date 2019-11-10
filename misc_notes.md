## Installing R under macOS

Some packages will need compilation from source code. To do so, you need to install Xcode command-line tools. 

It is imperative that you re-run the following command _everytime_ Xcode updates.

``` sh
xcode-select --install
```

**For R 3.4.0 to 3.6.0.** To use `data.table`’s parallel processing capability, you will need a version of OpenMP to be installed on your machine. Consider to follow the instructions on [https://github.com/Rdatatable/data.table/](https://github.com/Rdatatable/data.table/wiki/Installation#openmp-enabled-compiler-for-mac). This is optional.

**For R 3.6.1.** This release uses Clang 7.0.0 and GNU Fortran 6.1, neither of which is supplied by Apple. Make sure you install these [tools](https://cran.r-project.org/bin/macosx/tools/) first. This affects _all packages_ and is mandatory.

<!--- https://thecoatlessprofessor.com/programming/cpp/r-compiler-tools-for-rcpp-on-macos/ --->
