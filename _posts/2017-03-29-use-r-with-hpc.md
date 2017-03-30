---
layout: post
title: Use R and Rcpp on a HPC cluster
tag: [R, HPC, Rcpp]
---

Here I want to show how I use `Rcpp` with `R` on a HPC (high
performance computing) cluster.  There are two parts related to this
topic,

1. create a package that uses `Rcpp`,
2. setup parallel computing environment.

## Create a package with Rcpp

During the code development phase, `Rcpp` provides a very convenient
function `sourceCpp()` which compiles the C++ code and makes it ready
in the console.  Options such as `verbose` is also provided for
examing the compiling options and errors.  I personally prefer to use
`sourceCpp()` before trying to create a package for further
development.

To create a package with C code that uses `Rcpp`, the following are
the minimal code that is needed.  Here we create a package directory
called `PKG_NAME`, where you can replace `PKG_NAME` with your own
package name.

```{r}
library(Rcpp)
Rcpp.package.skeleton("PKG_NAME")
```

The structure of this folder is shown below

	[-]PKG_NAME
         DESCRIPTION
         NAMESPACE
      [-]R
            RcppExports.R
         Read-and-delete-me
      [-]man
            rcpp_hello_world.Rd
            testHello-package.Rd
      [-]src
            RcppExports.cpp
            rcpp_hello_world.cpp

The `rcpp_hello_world.cpp` can be replaced by your own `.cpp` file.
After all the desired `.cpp` files are added, we need to run 

```{r}
compileAttributes("PKG_NAME")
```

If you used packages such as `RcppArmadillo` and their namespace, the
namespace needs to be added to the file `RcppExports.cpp`, and these
packages should also be added to the `DESCRIPTION` file in the
`Imports` or `LinkingTo` fields.

## Setup Rmpi

`Rmpi` and `snow` are two packages on CRAN most often used for
distributed memory parallel computing (i.e. on parallel on different
nodes). `Rmpi` works well with `openMPI` in my case (I didn't have the
luck to get MPICH2 working).  If `openMPI` is the default one, then we
simply run `install.package("Rmpi")` to install it.  Otherwise we need
to run the following,

```{r}
install.package("Rmpi", configure.args="--with-mpi=path_to_openmpi")
```

where `path_to_openmpi` is obviously the path to the installed
directory of openMPI.  Since `Rmpi` provides the interface to the MPI
API which other parallel computing packages rely on, we can now
install other packages like `snow`, `foreach`, etc.

## Submit PBS Jobs

Usually, I start a interactive session to test if my code works.
After running

```{bash}
qsub -l "nodes=2:ppn=4" -I
```
I can start a `R` session with

```{bash}
/path_to_openmpi/mpirun -np 1 -hostfile $PBS_NODEFILE R --vanilla
```

In `R`, initialize the cluster, load the package needed, export the
variables needed, and perform the parallel computing.

```{r}
library(Rmpi)
library(snow)

cl <- makeCluster(mpi.universe.size() - 1, type="MPI")

clusterEvalQ(cl, {
    library(PKG_NAME)
    load(...)
})

x <- matrix(1:9, nrow=3)

clusterExport(cl, c('x'))

test <- function(i) sum(x[,i]) ## define the function as you like

parLapply(cl, 1:3, test)
```

Here you can call functions defined the your package `PKG_NAME`, which
all the compute nodes have access to.
