# thomas12
name: arm64 build checks

on: [pull_request]

jobs:
  build:

    runs-on: ubuntu-18.04

    steps:
    - uses: actions/checkout@v2
    - name: Install dependency packages
      run: |
        sudo sed -i -E 's|^deb ([^ ]+) (.*)$|deb [arch=amd64] \1 \2\ndeb [arch=arm64] http://ports.ubuntu.com/ubuntu-ports/ \2|' /etc/apt/sources.list
        sudo dpkg --add-architecture arm64
        sudo apt-get update
        sudo apt-get install -y --no-install-recommends \
                crossbuild-essential-arm64 \
                git \
                cmake \
                libpython-dev:arm64 \
                libpython3-dev:arm64 \
                python-numpy \
                python3-numpy
    - name: Fetch opencv_contrib
      run: |
        git clone --depth 1 https://github.com/opencv/opencv_contrib.git ../opencv_contrib
    - name: Configure
      run: |
        mkdir build
        cd build
        cmake -DPYTHON2_INCLUDE_PATH=/usr/include/python2.7/ \
              -DPYTHON2_LIBRARIES=/usr/lib/aarch64-linux-gnu/libpython2.7.so \
              -DPYTHON2_NUMPY_INCLUDE_DIRS=/usr/lib/python2.7/dist-packages/numpy/core/include \
              -DPYTHON3_INCLUDE_PATH=/usr/include/python3.6m/ \
              -DPYTHON3_LIBRARIES=/usr/lib/aarch64-linux-gnu/libpython3.6m.so \
              -DPYTHON3_NUMPY_INCLUDE_DIRS=/usr/lib/python3/dist-packages/numpy/core/include \
              -DCMAKE_TOOLCHAIN_FILE=../platforms/linux/aarch64-gnu.toolchain.cmake \
              -DOPENCV_EXTRA_MODULES_PATH=../../opencv_contrib/modules \
              ../
    - name: Build
      run: |
        cd build
        make -j$(nproc --all)
© 2020 GitHub, Inc.
