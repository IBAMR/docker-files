FROM fedora:33
MAINTAINER David Wells <drwells@email.unc.edu>

##
## I. Build and install dependencies
##   A. Some from RPMs:
RUN dnf update -y && \
  dnf install -y gcc g++ gfortran && \
  dnf install -y diffutils m4 gawk sed grep python3 patch which && \
  dnf install -y sudo cmake wget && \
  dnf install -y zlib-devel mpich-devel blas-devel && \
  dnf install -y lapack-devel hdf5-mpich-devel libtirpc-devel && \
  dnf clean all

##   B. Some non-RPM binaries from elsewhere:

##     i. sccache binaries for use when building IBAMR:

RUN \
  cd /tmp && \
  echo "e735afaac1ac3655a77104296535476a9485f571c67d22a1381a3f1b32c73172 sccache" > sccache.sum && \
  wget -q -O sccache "https://gitlab.kitware.com/api/v4/projects/6955/packages/generic/sccache/v0.2.15-background-init-20210523.0/sccache-v0.2.15-background-init-x86_64-unknown-linux-musl" && \
  sha256sum -c sccache.sum && \
  chmod +x sccache && \
  mv sccache /usr/bin

##   C. Specific versions of things from source:
##
##      i. petsc
##         + Note that we monkey-patch the configure script to work with
##           Python >= 3.9 as threading.Thread.isAlive has been renamed.
##         + We also manually download silo; the original download was removed
##           due to a bug, causing the petsc build to fail.

RUN \
  unset PETSC_DIR && \
  export PETSC_ARCH=x86_64 && \
  mkdir -p /build/petsc && \
  cd /build/petsc && \
  wget -q https://wci.llnl.gov/sites/wci/files/2021-01/silo-4.10.2-bsd-smalltest.tgz && \
  wget -q http://ftp.mcs.anl.gov/pub/petsc/release-snapshots/petsc-3.13.4.tar.gz && \
  tar xzf petsc-3.13.4.tar.gz && \
  cd petsc-3.13.4 && \
  mv config/BuildSystem/script.py /tmp/script.py && \
  sed -e 's/isAlive/is_alive/g' /tmp/script.py > config/BuildSystem/script.py && \
  rm -f /tmp/script.py && \
  /usr/bin/python3.9 configure \
    --prefix=/petsc/ \
    --CC=/usr/lib64/mpich/bin/mpicc \
    --CXX=/usr/lib64/mpich/bin/mpic++ \
    --FC=/usr/lib64/mpich/bin/mpifort \
    --COPTFLAGS="-O3" \
    --CXXOPTFLAGS="-O3" \
    --FOPTFLAGS="-O3" \
    --with-mpi=1 \
    --with-shared-libraries=1 \
    --with-64-bit-indices=0 \
    --with-debugging=0 \
    --download-hypre=1 \
    --download-metis=1 \
    --download-parmetis=1 \
    --with-fortran-bindings=0 \
    --download-silo=/build/petsc/silo-4.10.2-bsd-smalltest.tgz \
    --with-x=0 && \
  make PETSC_DIR=$(pwd) PETSC_ARCH=${PETSC_ARCH} all && \
  make install && \
  cd ~/ && \
  rm -rf /build

##      ii. libmesh

RUN \
  mkdir -p /build/libmesh/bld && \
  mkdir -p /libmesh && \
  cd /build/libmesh && \
  wget -q https://github.com/libMesh/libmesh/releases/download/v1.6.2/libmesh-1.6.2.tar.gz && \
  tar xzf libmesh-1.6.2.tar.gz && \
  mv libmesh-1.6.2 1.6.2 && \
  cd bld && \
  export PETSC_DIR=/petsc/ && \
  export PETSC_ARCH= && \
    ../1.6.2/configure \
    CC=/usr/lib64/mpich/bin/mpicc \
    CXX=/usr/lib64/mpich/bin/mpic++ \
    FC=/usr/lib64/mpich/bin/mpifort \
    --prefix=/libmesh \
    --with-methods=opt \
    --with-mpi \
    --disable-boost \
    --disable-capnproto \
    --disable-cppunit \
    --disable-eigen \
    --disable-examples \
    --disable-fparser \
    --disable-hdf5 \
    --disable-laspack \
    --disable-metaphysicl \
    --disable-nlopt \
    --disable-openmp \
    --disable-perflog \
    --disable-pthreads \
    --disable-reference-counting \
    --disable-vtk \
    --enable-curl=no \
    --enable-exodus=yes \
    --enable-glpk=no \
    --enable-nanoflann=no \
    --enable-qhull=no \
    --enable-tecplot=no \
    --enable-timestamps=no \
    --enable-petsc \
    --enable-petsc-required \
    --enable-triangle \
    --enable-unique-id \
    --with-thread-model=none \
    --with-metis=PETSc \
    --with-parmetis=PETSc \
    --disable-strict-lgpl \
    --disable-glibcxx-debugging \
    PETSC_DIR=${PETSC_DIR} \
    PETSC_ARCH=${PETSC_ARCH} \
    && \
  make -j6 install && \
  cd ~/ && \
  rm -rf /build

##      ii. numdiff

RUN \
  mkdir -p /build && \
  mkdir -p /numdiff && \
  mkdir -p /build/numdiff/bld && \
  cd /build/numdiff && \
  wget -q "http://download-mirror.savannah.gnu.org/releases/numdiff/numdiff-5.9.0.tar.gz" && \
  tar xzf numdiff-5.9.0.tar.gz && \
  mv numdiff-5.9.0 5.9.0 && \
  cd 5.9.0 && \
  ./configure --prefix=/numdiff && \
  make -j6 install && \
  cd ~/ && \
  rm -rf /build

##      iii. SAMRAI

# This is tricky since SAMRAI explicitly checks for include/hdf5.h which does 
# not exist on Fedora (here it is /usr/include/mpich-x86_64/hdf5.h). Get around
# this by first creating a shim.
RUN \
  mkdir -p /build && \
  mkdir -p /build/hdf5-shim && \
  ln -s /usr/include/mpich-x86_64 /build/hdf5-shim/include && \
  ln -s /usr/lib64/mpich/lib /build/hdf5-shim/lib && \
  mkdir -p /samrai && \
  mkdir -p /build/samrai/bld && \
  cd /build/samrai && \
  wget -q https://github.com/LLNL/SAMRAI/archive/refs/tags/2.4.4.tar.gz && \
  wget -q https://github.com/IBAMR/IBAMR/releases/download/v0.10.1/samrai-2.4.4-patch-ibamr-0.10.patch && \
  tar xzf 2.4.4.tar.gz && \
  mv SAMRAI-2.4.4 2.4.4 && \
  cd 2.4.4 && \
  patch -p1 < ../samrai-2.4.4-patch-ibamr-0.10.patch && \
  cd ../bld && \
  ../2.4.4/configure \
    CFLAGS="-fPIC -O3" \
    CXXFLAGS="-fPIC -O3" \
    FFLAGS="-fPIC -O3" \
    --prefix=/samrai \
    CC=/usr/lib64/mpich/bin/mpicc \
    CXX=/usr/lib64/mpich/bin/mpic++ \
    FC=/usr/lib64/mpich/bin/mpifort \
    --with-F77=/usr/lib64/mpich/bin/mpifort \
    --with-hdf5=/build/hdf5-shim \
    --without-hypre \
    --with-silo=/petsc/ \
    --without-blaslapack \
    --without-cubes \
    --without-eleven \
    --without-kinsol \
    --without-petsc \
    --without-sundials \
    --without-x \
    --enable-implicit-template-instantiation \
    --disable-deprecated \
    && \
  make -j6 && \
  make -j6 install && \
  cd ~/ && \
  rm -rf /build/
