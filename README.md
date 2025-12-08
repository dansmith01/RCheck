
## CRAN 

### Special Checks

* https://cran.r-project.org/web/checks/check_issue_kinds.html

### CRAN-Hosted Builders

* https://mac.r-project.org/macbuilder/submit.html
* https://win-builder.r-project.org/upload.aspx


## GitHub Actions

### `r-lib` Builders

```r
usethis::use_github_action_check_standard()
```


### `rhub` Builders

```r
rhub::rhub_setup()

# Add "rchk" if your package compiles a shared library.
rhub::rhub_check(platforms = c(
  "clang-asan", "clang-ubsan", "gcc-asan", "valgrind" ,
  "gcc13", "gcc14", "gcc15", "intel", "c23",
  "clang16", "clang17", "clang18", "clang19", "clang20",
  "linux", "windows", "macos", "m1-san", "macos-arm64",
  "ubuntu-clang", "ubuntu-gcc12", "ubuntu-next", "ubuntu-release",
  "lto", "atlas", "mkl", "nold", "noremap", "donttest" ))
```


## Local Docker

### Valgrind
```sh
docker pull rocker/r-base
docker run -it rocker/r-base bash
export GITHUB_REPO=cmmr/hdf5lib
export R_PACKAGE=$(basename "$GITHUB_REPO")
export VALGRIND_ARGS="--tool=memcheck --leak-check=full --track-origins=yes"
apt update
apt install -y git libcurl4-openssl-dev valgrind
git clone "https://github.com/${GITHUB_REPO}.git"
R -q -e "install.packages('pak', repos = 'https://r-lib.github.io/p/pak/stable/source/linux-gnu/x86_64')"
R -q -e "pak::local_install_dev_deps('$R_PACKAGE')"
cd "$R_PACKAGE"
R -d "valgrind $VALGRIND_ARGS" --vanilla -e 'testthat::test_local()' > valgrind_log.txt 2>&1
```

### Alpine Linux (`musl`)
```sh
docker pull rhub/r-minimal
docker run -it rhub/r-minimal bash
export GITHUB_REPO=cmmr/hdf5lib
export R_PACKAGE=$(basename "$GITHUB_REPO")
export RCMDCHECK_ERROR_ON=warning
export NOT_CRAN=true
export _R_CHECK_CRAN_INCOMING_=false
apk add --no-cache gcc g++ gfortran musl-dev R-dev linux-headers curl-dev libxml2-dev git checkbashisms
git clone "https://github.com/${GITHUB_REPO}.git"
R -q -e "install.packages('pak', repos='https://r-lib.github.io/p/pak/stable/source/linux-musl/x86_64')"
R -q -e "pak::local_install_dev_deps('$R_PACKAGE')"
R -q -e "pak::pak('rcmdcheck')"
R -q -e "rcmdcheck::rcmdcheck('$R_PACKAGE', args = c('--no-manual', '--as-cran'))"
find . -name '00install.out' -exec cat {} +
```

### Static Code Analysis (`rchk`)

*From https://github.com/kalibera/cran-checks/tree/master/rchk#readme and https://github.com/kalibera/rchk*

```sh
docker pull kalibera/rchk
mkdir rchk
R CMD build ecodive
cp ecodive_2.2.1.tar.gz ecodive/rchk
# On Unix (not git bash for windows)
docker run -v `pwd`/ecodive/rchk:/rchk/packages kalibera/rchk /rchk/packages/ecodive_2.2.1.tar.gz 
# On Windows
docker run -v %cd%/ecodive/rchk:/rchk/packages kalibera/rchk /rchk/packages/ecodive_2.2.1.tar.gz
```
