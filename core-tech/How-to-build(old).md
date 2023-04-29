# Windows
1. Install Visual Studio >= 2017 with CMake support.
1. Download and install Boost prebuilt binaries https://sourceforge.net/projects/boost/files/boost-binaries/1.68.0/boost_1_68_0-msvc-14.1-64.exe, also add `BOOST_ROOT` to the _Environment Variables_.
1. Download and install OpenSSL prebuilt binaries https://slproweb.com/products/Win32OpenSSL.html (`Win64 OpenSSL v1.1.0h` for example) and add `OPENSSL_ROOT_DIR` to the _Environment Variables_.
1. Download and install QT 5.11 http://master.qt.io/new_archive/qt/5.11/5.11.0/qt-opensource-windows-x86-5.11.0.exe and add `QT5_ROOT_DIR` to the _Environment Variables_ (usually it looks like `.../5.11.0/msvc2017_64`), also add `QML_IMPORT_PATH` (it should look like `%QT5_ROOT_DIR%\qml`). BTW disabling system antivirus on Windows makes QT installing process much faster.
1. Add `.../qt511/5.11.1/msvc2017_64/bin` and `.../boost_1_68_0/lib64-msvc-14.1` to the _System Path_.
1. Open project folder in Visual Studio, select your target (`Release-x64` for example, if you downloaded 64bit Boost and OpenSSL) and select `CMake -> Build All`.
1. Go to `CMake -> Cache -> Open Cache Folder -> beam` (you'll find binaries in `beam/...`, `wallet/...`, `ui/...`, `explorer/...` subfolders).

# Linux 

**Please read carefully the points below**

**1.** After finishing your build you'll find the the `beam-node` binary (node) in the `beam/beam` folder, `beam-wallet` (cli wallet) in the `beam/wallet` folder and `BeamWallet` (GUI wallet) in the `beam/ui` folder.

**2.** You need to clone beam sources before building
```
git clone https://github.com/BeamMW/beam.git
```
The command above checks out the latest master branch where all the development happens. Usually it is not what you want to build. Choose an appropriate branch [here](https://github.com/BeamMW/beam/branches) and check out it after you have the main repository cloned. For example the command below checks out the Double Doppler 4.0 release branch
```
cd beam
git checkout double_doppler_4.0RC
```

**3.** By default instructions produce masternet builds (development version, development blockchain). To be able to build testnet (testing) or mainnet (real-world blockchain) you need to edit the `beam/CMakeLists.txt` file lines 151-157. Uncomment (remove the '#' sign) respective definitions.
```
# uncomment next line for testnet
#set(BEAM_TESTNET TRUE)
# uncomment next line for mainnet
#set(BEAM_MAINNET TRUE)

# uncomment next line for swap mainnet
#set(SWAP_MAINNET TRUE)
```

**4.** Server instructions result only in a valid build and enable you running all the CLI tools. They are not intended to ensure that you would be able to run the UI. Desktop build instructions include additional steps to ensure that the UI can be launched.

**5.** At the moment building the UI can take a lot of time (a few hours) due to the bug in QT: [QTBUG-27936](https://bugreports.qt.io/browse/QTBUG-27936). To skip building the UI pass `-DBEAM_NO_QT_UI_WALLET=On` to cmake. You can also omit all Qt5 dependencies in this case.

## Ubuntu 18.04 Desktop

Instructions below are valid for a clean default Ubuntu 18.04.03 Desktop LTS install. If your're building on an aged system please check that steps 1, 2 & 4 do not overwrite/conflict with more recent versions of cmake and boost that might have been installed before. You can omit cmake/boost installation if you already have more recent versions.

1. Install dependencies
    ```
    sudo add-apt-repository ppa:beineri/opt-qt-5.11.0-bionic
    sudo add-apt-repository ppa:mhier/libboost-latest
    sudo apt-get update && sudo apt-get upgrade
    sudo apt-get install g++-8 libssl-dev curl wget git make 
    sudo apt-get install libgl1-mesa-dev zlib1g-dev libboost1.67-dev
    sudo apt-get install qt511base qt511declarative qt511svg qt511tools 
    ```

1. Install cmake
    ```
    wget https://cmake.org/files/v3.13/cmake-3.13.0-Linux-x86_64.sh
    sudo sh ./cmake-3.13.0-Linux-x86_64.sh --skip-license --prefix=/usr
    ```

1. Go to the Beam project folder and start the release build
    ```
    export PATH=/opt/qt511/bin:$PATH && export CC=gcc-8 && export CXX=g++-8
    cmake -DCMAKE_BUILD_TYPE=Release && make -j4
    ```

1. To be able to run the UI (BeamWallet) install additional dependencies
    ```
    sudo apt-get install qt511quick* qt511graphicaleffects
    ```

## Ubuntu 18.04 Server

Instructions below are valid for a clean default Ubuntu 18.04.03 Server LTS with all the updates applied during install. If your're building on an aged system please check that steps 1 & 2 do not overwrite/conflict with more recent versions of cmake and boost that might have been installed before. You can omit cmake/boost installation if you already have more recent versions.

1. Install dependencies
    ```
    sudo add-apt-repository ppa:beineri/opt-qt-5.11.0-bionic
    sudo add-apt-repository ppa:mhier/libboost-latest
    sudo apt-get update && sudo apt-get upgrade
    sudo apt-get install g++-8 libssl-dev curl wget git make 
    sudo apt-get install libgl1-mesa-dev zlib1g-dev libboost1.67-dev
    sudo apt-get install qt511base qt511declarative qt511svg qt511tools 
    ```

1. Install cmake
    ```
    wget https://cmake.org/files/v3.13/cmake-3.13.0-Linux-x86_64.sh
    sudo sh ./cmake-3.13.0-Linux-x86_64.sh --skip-license --prefix=/usr
    ```

1. Go to the Beam project folder and start the release build
    ```
    export PATH=/opt/qt511/bin:$PATH && export CC=gcc-8 && export CXX=g++-8
    cmake -DCMAKE_BUILD_TYPE=Release && make -j4
    ```

## Ubuntu 19.10 Server

Instructions below are valid for a default Ubuntu 19.10 Server install with updates applied during install.

1. Install dependencies
    ```
    sudo apt-get update && sudo apt-get upgrade
    sudo apt-get install g++ git make cmake libboost-all-dev zlib1g-dev
    sudo apt-get install libssl-dev qtbase5-dev qtdeclarative5-dev 
    sudo apt-get install libqt5svg5-dev qttools5-dev qt5-default
    ```

1. Go to the Beam project folder and start the release build
    ```
    cmake -DCMAKE_BUILD_TYPE=Release . && make -j4
    ```

## Ubuntu 19.10 Desktop

Instructions below are valid for a minimal Ubuntu 19.10 Desktop install with 'update during install' option enabled.

1. Install dependencies
    ```
    sudo apt-get install g++ git make cmake libboost-all-dev zlib1g-dev
    sudo apt-get install libssl-dev qtbase5-dev qtdeclarative5-dev 
    sudo apt-get install libqt5svg5-dev qttools5-dev qt5-default
    ```

1. Go to the Beam project folder and start the release build
    ```
    cmake -DCMAKE_BUILD_TYPE=Release . && make -j4
    ```

1. To be able not only to build but also to run the UI (BeamWallet) you need to install additional dependencies

    ```
    sudo apt-get install qml-module-qt*
    ```

## Centos 7.7

Instructions below are valid for a clean CentOS 7.0-1908 minimal install. If your're building on an aged system please check that steps 2 & 3 do not overwrite more recent versions of cmake and boost that might have been installed before.

N.B. These build instruction provide only commandline tools including node but no GUI wallet. Use newer operating system with more up to date Qt libraries (5.11+) to effortlessly build the UI.

1. Install dependencies 
    ```
    sudo yum update && shutdown -r now
    sudo yum install centos-release-scl yum-utils
    sudo yum-config-manager --enable rhel-server-rhscl-8-rpms
    sudo yum install devtoolset-8-gcc* 
    scl enable devtoolset-8 bash
    sudo yum install git make wget openssl-devel
    ```

1. Build & install boost. This step could take considerable amount of time
    ```
    wget https://dl.bintray.com/boostorg/release/1.66.0/source/boost_1_66_0.tar.gz
    tar xzvf boost_1_66_0.tar.gz
    cd boost_1_66_0 && ./bootstrap.sh && sudo ./b2 install && cd ~
    ```

1. Install cmake
    ```
    wget https://cmake.org/files/v3.13/cmake-3.13.0-Linux-x86_64.sh
    sudo sh ./cmake-3.13.0-Linux-x86_64.sh --skip-license --prefix=/usr
    ```

1. Go to the beam sources folder and start the release build 
    ```
    cmake -DCMAKE_BUILD_TYPE=Release -DBEAM_NO_QT_UI_WALLET=On . && make -j4
    ```

## CentOS 8

Instructions below are valid for a clean CentOS 8.0-1905 minimal install. If your're building on an aged install please check that step 2 does not overwrite more recent version of cmake that might have been installed before.

1. Install dependencies
    ```
    sudo dnf update
    sudo dnf config-manager --set-enabled PowerTools
    sudo dnf install git make cmake gcc-c++ libstdc++-static boost-devel 
    sudo dnf install openssl-devel tar wget qt5-qtbase-devel qt5-linguist
    sudo dnf install qt5-qtsvg-devel qt5-qtdeclarative-devel 
    ```

1. Install cmake
    ```
    wget https://cmake.org/files/v3.13/cmake-3.13.0-Linux-x86_64.sh
    sudo sh ./cmake-3.13.0-Linux-x86_64.sh --skip-license --prefix=/usr
    ```

1. Go to the beam sources folder and start the release build
    ```
    export PATH=${PATH}:/usr/lib64/qt5/bin
    cmake -DCMAKE_BUILD_TYPE=Release . && make -j4
    ```

## Fedora 31 Workstation

Instructions below are valid for a default Fedora 31-1.9 Workstation install.

1. Install dependencies
    ```
    sudo dnf update && shutdown -r now
    sudo dnf install git make cmake gcc-c++ libstdc++-static boost-devel 
    sudo dnf install zlib-devel openssl-devel tar qt5-qtbase-devel qt5-linguist
    sudo dnf install qt5-qtsvg-devel qt5-qtdeclarative-devel 
    ```

1. Go to the Beam project folder and start the release build
    ```
    cmake -DCMAKE_BUILD_TYPE=Release . && make -j4
    ```

1. To be able to run the UI (BeamWallet) install additional dependencies
    ```
    sudo dnf install qt5-qtquickcontrols qt5-qtquickcontrols2
    ```

## Fedora 31 Server

Instructions below are valid for a default Fedora 31-1.9 Headless Server install.

1. Install dependencies
    ```
    sudo dnf update && shutdown -r now
    sudo dnf install git make cmake gcc-c++ libstdc++-static boost-devel 
    sudo dnf install zlib-devel openssl-devel tar qt5-qtbase-devel qt5-linguist
    sudo dnf install qt5-qtsvg-devel qt5-qtdeclarative-devel 
    ```

1. Go to the Beam project folder and start the release build
    ```
    cmake -DCMAKE_BUILD_TYPE=Release . && make -j4
    ```

# Mac
1. Download the source code by using: 
    ```
    git clone https://github.com/BeamMW/beam.git
    ```
    this will give you a master branch which is developer's version of Beam. To get _mainnet_ or _testnet_ use 
    ```
    git clone --branch mainnet https://github.com/BeamMW/beam.git
    ```
    or 
    ```
    git clone --branch testnet https://github.com/BeamMW/beam.git
    ```

1. Install Brew Package Manager.
1. Install necessary packages using:  
    ```
    brew install openssl cmake
    ```
1. Remove any existing brew installations of qt and boost:  
    ```
    brew uninstall --ignore-dependencies qt boost
    ```
1. Download and install a compatible version of boost:  
    ```
    curl -O https://raw.githubusercontent.com/Homebrew/homebrew-core/5da5895add2f6b9320d654dd844d4827f6876c8b/Formula/boost.rb
    brew install ./boost.rb
    ```
1. Download and install QT5.11 from the official website http://master.qt.io/new_archive/qt/5.11/5.11.0/qt-opensource-mac-x64-5.11.0.dmg
1. Set your Environment Variables by using the following:  
    ```
    - export OPENSSL_ROOT_DIR="/usr/local/opt/openssl@1.1"
    - export PATH=/Users/<username>/Qt5.11.0/5.11.0/clang_64/bin:$PATH
    - export QT5_ROOT_DIR=/Users/<username>/Qt5.11.0/5.11.0/
    - export QML_IMPORT_PATH="/Users/<username>/Qt5.11.0/5.11.0/qml"
    ```
1. Go to Beam project folder and call  
    ```
    cmake -DCMAKE_BUILD_TYPE=Release . && make -j4
    ```
1. Use `cpack` to get .dmg file
1. You'll find binaries in `beam/wallet`, `beam/ui`, and `beam/explorer` subfolders.

If you don't want to build UI don't install QT5 and add `-DBEAM_NO_QT_UI_WALLET=On` command line parameter when you are calling `cmake`.
