# ShrewMacOS
 Shrew for MacOS 10.13.6

 Guide to build ShrewSoftVPN for MacOS working and tested under 10.13.6 
 
 This guide is based of a guide from http://kb.amft-it.de/doku.php?id=kb-macos:shrewsoftvpn
 I only edited it to work in 2020 with latest releases of everything that brew ships 
 
 # Future Goals
It would be nice to make it work under Mojave or Catalina with the latest qt5 release but it is a lot of work i guess


# Guide
### First disable Gatekeeper
```
sudo spctl –-master-disable 
```
### Then install Xcode cmdline-tools if not already installed
```
xcode-select –-install
```
### intstall brew if not already 
```
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install.sh)"
```
### install required Tun/Tap driver -- there are know problems with the names of th deamons under Catalina
```
brew install Caskroom/cask/tuntap
brew install openssl
```
### if there is an installed version of openssl and u get an error like i did: Error: openssl@1.0 is already installed from homebrew/core! then just run -> 
``` 
brew install https://raw.githubusercontent.com/Homebrew/homebrew-core/64555220bfbf4a25598523c2e4d3a232560eaad7/Formula/openssl.rb -f
```
### Next install Qt4. Its very very important to use v4
```
brew tap cartr/qt4
brew install qt@4
brew uninstall qt //only if qt5 or other version installed 
```
# get sources from https://www.shrew.net/download/ike/ike-2.2.1-release.tgz

### for Chrium users 
```
cd ~/Downloads
tar xzvf ike-2.2.1-
release.tgz
cd ike
```
### for Safari users 
```
cd ~/Downloads
tar xvf ike-2.2.1-release.tar
cd ike
```

#### pre-compilation and -installation cleanup
```
sudo rm -rf /Applications/Shrew*
sudo rm -rf /usr/local/opt/shrew*
sudo rm -rf /usr/local/opt/shrew*
sudo rm -rf /usr/local/opt/shrew*
```
#### go to /Library/LaunchDaemons and check for files with shrew in the name in .plist format 
```
sudo launchctl <filename_with_*shrew*>.plist
sudo rm <filename_with_*shrew*>.plist
```
#### because i installed openssl earlier i had to change the path to openssl in the CMakeList.txt, but first we have to change directories
```
cd ~/Downloads/ike
open CMakeLists.txt 
```

## MakeLists.txt 
Add lines under #marked line 
```
project( IKE )

cmake_minimum_required( VERSION 2.4 )

#add next line - for set prefix path to /usr/local
set(CMAKE_INSTALL_PREFIX "/usr/local") // added this line 
#add next 2 lines - additional dirs for compiler and linker - it depends where your OpenSSL is installed - check this
include_directories(/usr/local/Cellar/openssl@1.0/1.0.2t/include) // your openssl/include installpath  
link_directories(/usr/local/Cellar/openssl/1.0.2s/lib) // your openssl/lib installpath  

if( COMMAND cmake_policy )

cmake_policy( SET CMP0003 NEW )

endif( COMMAND cmake_policy )
```

## A little bit further down you have to add the paths too

```
set(
SEARCH_INC
#add next line - it depends where your OpenSSL is installed - check this
/usr/local/Cellar/openssl@1.0/1.0.2t/include //<- thats your path that was added
/usr/local/include
/usr/include )

set(
SEARCH_LIB
#add next line - it depends where your OpenSSL is installed - check this
//usr/local/Cellar/openssl@1.0/1.0.2t/lib //<- thats your path that was added
/usr/local/lib
/usr/lib )

set(
SEARCH_BIN
#add next line - it depends where your OpenSSL is installed - check this
/usr/local/Cellar/openssl@1.0/1.0.2t/bin //<- thats your path that was added
/usr/local/bin
/usr/pkg/bin
/usr/bin )

set(
SEARCH_SYS
#add next line - it depends where your OpenSSL is installed - check this
/usr/local/Cellar/openssl@1.0/1.0.2t/share //<- thats your path that was added
/usr/local
/usr/share
/usr )

```
## Save and override MakeLists.txt
## Now it is time to create Cmake-files to compile afterwards we do so like:

```
cmake -DQTGUI=YES -DNATT=YES -DCMAKE_INSTALL_PREFIX=/usr/local -DQT_QMAKE_EXECUTABLE=/usr/local/Cellar/qt@4/4.8.7_6/bin/qmake
``` 
### here it is important that alsp the path to your installed qt4 ist correct 

### then 
` make `
### and 
` sudo make install` needs sudo rights to write something in /Library/Frameworks

## there are still things to do 
``` 
cd script/macosx/
``` 
### change part of net.shrew.iked.plist in /ike/scripts/macosx from 
``` 
<array>
<string>/usr/sbin/iked</string>
<string>-F</string>
</array>
``` 
to 

``` 
<array>
<string>/usr/local/sbin/iked</string>
<string>-F</string>
</array>
``` 

## Last steps
``` 
udo cp net.shrew.iked.plist /Library/LaunchDaemons
sudo cp /usr/local/etc/iked.conf.sample /usr/local/etc/iked.conf
cd /Library/LaunchDaemons
sudo launchctl load net.shrew.iked.plist
``` 
## check if iked is running
``` 
sudo ps x | grep iked
``` 

## Start Application from /Applications folder also it generated an ~/.ike folder where the .vpn configs are in ~/.ike/ so whenever you want to manually 
``` 
cp <filename>.vpn ~/.ike/sites
``` 
it is important to have a file in .vpn format otherwise it wont be recognnized by shrew 

## also there is a commandline-tool, ikec. it is used for starting a vpn-connection
``` 
ikec -r "<filename>.vpn" -a
``` 

