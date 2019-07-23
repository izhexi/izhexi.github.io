---
title: Xctool打包脚本
date: 2016-07-23 17:28:06
tags: [iOS, xctool]
categories: iOS
---
``` sh

# 获取UUID命令 
openssl smime -in /Users/证书.mobileprovision -inform der -verify 

```


``` sh
#!/bin/sh 

set -e 

#cd `dirname $0` 

SCHEME_NAME='name' 

WORKSPACE_NAME='name.xcworkspace' 

#打包到路径 

BUILD_Path=/Users/{path}/build 

PROVISIONING_NAME='ca9102e0-83b1-4658-befa-e88ecxxxxxxxx' 

CONFIGURATION='Debug' 

xctool -workspace ${WORKSPACE_NAME} -scheme ${SCHEME_NAME} clean build -reporter pretty -configuration ${CONFIGURATION} BUILD_DIR=${BUILD_Path} BUILD_ROOT=${BUILD_Path} CONFIGURATION_BUILD_DIR=${BUILD_Path} PROVISIONING_PROFILE=${PROVISIONING_NAME} 'CODE_SIGN_IDENTITY=iPhone Developer: xxxx' 

cd ${BUILD_Path} 

echo '*********************新建文件夹***************************' 

mkdir -p ${SCHEME_NAME} 

#chmod 777 ${SCHEME_NAME} 

echo '********************新建文件夹完成*************************' 

echo '****************复制文件************************' 

mv ${SCHEME_NAME}.app ${SCHEME_NAME} 

echo '******************复制成功*******************' 

echo '*******************正在压缩*******************' 

zip -r ${SCHEME_NAME}.zip ${SCHEME_NAME}/* 

echo '*******************压缩成功******************' 

echo '**********************正在打包******************' 

mv ${SCHEME_NAME}.zip ${SCHEME_NAME}.ipa 

echo '*********************打包成功*********************' 

```
