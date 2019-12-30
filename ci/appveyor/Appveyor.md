# Appveyor
作者：康林
---------

### 基本操作
- 在 appveyor.yml 中多行书写脚本，使用“ |- ”

          - |-
            if [ "$BUILD_TARGERT"="unix" -a -z "$RABBIT_ARCH" ]; then
                source /opt/qt511/bin/qt511-env.sh
            fi

### [分发](https://docs.travis-ci.com/user/deployment/)
#### [分发到 github](https://docs.travis-ci.com/user/deployment/releases/)

        deploy:
          - provider: GitHub
           #release: RabbitThirdLibrary_$(BUILD_VERSION)
           description: 'Release Tasks $(APPVEYOR_REPO_TAG_NAME) on windows'
           #token : https://github.com/settings/tokens
           #password encrypt: https://ci.appveyor.com/tools/encrypt
           auth_token:
             secure: 
           #draft: true
           #prerelease: true
        on:
           appveyor_repo_tag: true        # deploy on tag push only


参数说明：
- provider: github releases 是 GitHub
- auth_token: 登录github的api key。
  + 生成：https://github.com/settings/tokens
  + 加密：https://ci.appveyor.com/tools/encrypt
- on: 条件分发。环境变量可做为条件
