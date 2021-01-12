# Setup on Mac
## Get the source code. At present, this is from the default branch 'develop'.
### Open a new Terminal window and navigate to the folder of your choice to store the source code. Then run:
```sh
git clone https://github.com/pgmodeler/pgmodeler.git  
cd pgmodeler
```

## Install the dependencies. Doesn't matter what directory you're in, but does require you have HomeBrew installed. By extension, this means you'll also need XCode installed, since it's a dependency for HomeBrew.
```sh
brew install qt
brew link qt5 --force
brew install postgresql
brew install libxml2
```  

## Edit file path variables PGSQL_LIB, PGSQL_INC, XML_INC, XML_LIB in pgmodeler/pgmodeler.pri. At time of writing, these are lines 160-166:
```
macx {
  !defined(PGSQL_LIB, var): PGSQL_LIB = /usr/local/opt/postgresql/lib/libpq.dylib
  !defined(PGSQL_INC, var): PGSQL_INC = /usr/local/opt/postgresql/include
  !defined(XML_INC, var): XML_INC = /Applications/Xcode.app/Contents/Developer/Platforms/MacOSX.platform/Developer/SDKs/MacOSX.sdk/usr/include/libxml2
  !defined(XML_LIB, var): XML_LIB = /usr/lib/libxml2.dylib
  INCLUDEPATH += $$PGSQL_INC $$XML_INC
}
```  
#### Note: confirm that these file paths are correct and actually contain the files/directories outlined above.  
  
## Set environment variables for ease of installation. Confirm the paths here before moving to the next step.
```sh
export QT_ROOT=/usr/local/opt/qt
export INSTALLATION_ROOT=/Applications/pgmodeler.app
export PGSQL_ROOT=/usr/local/opt/postgres
export OPENSSL_ROOT=/usr/local/opt/openssl
```  
  
## Build Commands (note: Differing from the instructions on the pgmodeler installation guide, the libssl.1. and libcrypto.1. are found in the openssl install, not in the pg install). Run each command one at a time, except for commands/new lines ending with a \, since that negates the new line. In those instances, copy-paste the full command as is.
```sh
$QT_ROOT/bin/qmake -r CONFIG+=release pgmodeler.pro
make && make install
$QT_ROOT/bin/macdeployqt $INSTALLATION_ROOT \
                         $INSTALLATION_ROOT/Contents/MacOS/pgmodeler-ch \
                         $INSTALLATION_ROOT/Contents/MacOS/pgmodeler-cli
```  

## Resolve Dependencies
```sh
cp $PGSQL_ROOT/lib/libpq.5.dylib $OPENSSL_ROOT/lib/libssl.1.* \
   $OPENSSL_ROOT/lib/libcrypto.1.* $INSTALLATION_ROOT/Contents/Frameworks
```  

```sh
install_name_tool -change "@loader_path/../lib/libcrypto.1.1.dylib" "@loader_path/../Frameworks/libcrypto.1.1.dylib" $INSTALLATION_ROOT/Contents/Frameworks/libssl.1.1.dylib
install_name_tool -change "@loader_path/../lib/libcrypto.1.1.dylib" "@loader_path/../Frameworks/libcrypto.1.1.dylib" $INSTALLATION_ROOT/Contents/Frameworks/libpq.5.dylib
install_name_tool -change "@loader_path/../lib/libssl.1.1.dylib" "@loader_path/../Frameworks/libssl.1.1.dylib" $INSTALLATION_ROOT/Contents/Frameworks/libpq.5.dylib
install_name_tool -change libpq.5.dylib "@loader_path/../Frameworks/libpq.5.dylib" $INSTALLATION_ROOT/Contents/Frameworks/libpgconnector.dylib
```  
  
## This process builds pgmodeler.app, visually just 'pgmodeler', moves it to the Applications folder, and moves appropriate loading dependencies into the Application files. That's it! You should be able to open pgmodeler from your Applications folder now.