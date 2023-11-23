## Expo eas app store build(aos/ios)

1. eas setting
```bash
npm install -g eas-cli
```

2. eas login (expo login - email,pw)
```bash
eas login 
```
---
- preview build (app test용 빌드) - aos/ios
```bash
eas build --profile preview 
```

- production build (app 배포용 빌드) - aos/ios
```bash
eas build --profile production 
```

- ios appstore 제출하기
```bash
eas submit --platform ios 
```

```bash
## duid 추가 명령어
eas device:create

## 버전
aos -> app.json versionCode + 1.
ios -> package.json version
```
