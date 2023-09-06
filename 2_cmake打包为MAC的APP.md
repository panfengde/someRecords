## cmake打包为MAC的APP

### 注意camke设置
```sh
#重要
set(CMAKE_OSX_DEPLOYMENT_TARGET "11.5")

add_executable(GLFW_TEST
        MACOSX_BUNDLE
        main.cpp
        )

set_target_properties(GLFW_TEST PROPERTIES
          BUNDLE True
          MACOSX_BUNDLE_BUNDLE_NAME GLFW_TEST
          MACOSX_BUNDLE_ICON_FILE "0.1"
          MACOSX_BUNDLE_BUNDLE_VERSION "0.1"
          MACOSX_BUNDLE_SHORT_VERSION_STRING "0.1"
          MACOSX_BUNDLE_INFO_PLIST /Users/......../src/MacOSXBundleInfo.plist.in
          )

include(CPack)
```

### 注意camke设置
打包后的.app为生成的MAC App包
```sh
#查看程序的信息，其中注意minos
atool -lv LUI

#查看程序依赖的动态库
atool -L LUI

#修改依赖的动态包路径
sudo install_name_tool -change @rpath/libglfw3.3.dylib  @executable_path/../Frameworks/libglfw3.3.dylib LUI

#设置动态包的name id
sudo install_name_tool -id @executable_path/../Frameworks/libLAB.dylib  libLAB.dylib

#允许MAC运行APPTest.app
sudo xattr -rd com.apple.quarantine /Applications/APPTest.app
```


## 查询APP Resources包
将依赖文件放在Resources中，在通过系统AI获取到路径
```c++
#include <iostream>
#include <CoreFoundation/CoreFoundation.h>
std::string getLUIResourcesPath() {
    static std::string result;
    if (result.empty()) {
        CFBundleRef mainBundle = CFBundleGetMainBundle();
        CFURLRef resourcesURL = CFBundleCopyResourcesDirectoryURL(mainBundle);
        char resourcesPath[PATH_MAX];
        if (!CFURLGetFileSystemRepresentation(resourcesURL, TRUE, (UInt8 *) resourcesPath, PATH_MAX)) {
            // error!
            std::cout << "error Path: " << resourcesPath << std::endl;
        }

        CFRelease(resourcesURL);
        chdir(resourcesPath);
        result = resourcesPath;
    }else{
      chdir(resourcesPath);
    }
    return result;
};


int main()[
   auto resourcesPath = getLUIResourcesPath();
   fs::path entryPath(string(resourcesPath) + "/index.lab");
]
```

注意CmakeList设置Foundation动态库
```sh
target_link_libraries(LUI
        "-framework Foundation"
        glfw
        )
```

## 其他
MAC虚拟机中，不包含OPENGL环境，通过AMC虚拟机不能验证程序是否能正常运行