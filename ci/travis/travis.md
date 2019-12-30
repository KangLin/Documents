# Travis
---------
作者：康林(kl222@126.com)

### 基本操作
- 在 .travis 中多行书写脚本，使用“ | ”

          - |
            if [ "$BUILD_TARGERT"="unix" -a -z "$RABBIT_ARCH" ]; then
                source /opt/qt511/bin/qt511-env.sh
            fi

### [分发](https://docs.travis-ci.com/user/deployment/)
#### [分发到 github](https://docs.travis-ci.com/user/deployment/releases/)

        deploy:
          provider: releases
          api_key: ${GITHUB_TOKEN}
          file:
            - ./build_android/${BUILD_TARGERT}_*.tar.gz
            - file1
            - directroy/*
          skip_cleanup: true
          on:
            condition: $TRAVIS_OS_NAME = android
            tags: true

参数说明：
- provider: github releases 是 releases
- api_key: 登录github的api key。
  + 生成：https://github.com/settings/tokens
  + 使用：
    + 用环境变量。**注意**：Display value in build log 必须关闭
    + 直接使用加密变量，用 travis client 加密。参见：https://github.com/travis-ci/travis.rb#encrypt

          $ travis encrypt GITHUB_TOKEN=bar
          Please add the following to your .travis.yml file:

          secure: "gSly+Kvzd5uSul15CVaEV91ALwsGSU7yJLHSK0vk+oqjmLm0jp05iiKfs08j\n/Wo0DG8l4O9WT0mCEnMoMBwX4GiK4mUmGdKt0R2/2IAea+M44kBoKsiRM7R3\n+62xEl0q9Wzt8Aw3GCDY4XnoCyirO49DpCH6a9JEAfILY/n6qF8="

          Pro Tip™: You can add it automatically by running with --add.

- skip_cleanup: 如果为 true，分发前跳过清除。
- condition: 在单个bash条件评估为true时部署。 这必须是一个字符串值，并且等同于if [[<condition>]]; then <deploy>; fi。 例如，$ CC = gcc。 详见：https://docs.travis-ci.com/user/deployment#conditional-releases-with-on

