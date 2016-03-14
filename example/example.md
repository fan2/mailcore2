
## mailcore2.xcodeproj
### Run Script
Shell ： `/bin/sh`

```
"$SRCROOT/../scripts/get-ios.sh"
```

`/scripts/get-ios.sh` :

deps="ctemplate-ios libetpan-ios tidy-html5-ios"

- build-ctemplate-ios.sh
- build-libetpan-ios.sh
- build-tidy-ios.sh

### Externals
Header Search Paths：IOS_HEADERS_SEARCH_PATHS

libetpan-ios

- include/libetpan
- lib/libetpan-ios.a

## ios UI Test
iOS UI 测试工程： `example/ios/iOS UI Test`

### Build Phases|Target Dependencies
`iOS UI Test.xcodeproj` 依赖：static mailcore2 ios(mailcore2)。
包含子工程 `mailcore2.xcodeproj`。

### Link Binary with Libraries
libMailCore-ios.a
