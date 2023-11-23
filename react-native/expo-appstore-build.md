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

---

```bash
## duid 추가 명령어
eas device:create

## 버전
aos -> app.json versionCode + 1.
ios -> package.json version
```

```
eas build --profile production 빌드 후에 아래 페이지 접속 후 각각 앱 정보 입력
(ios는 eas submit --platform ios 명령어로 apple connect에 올라감)
aos -> google play console
ios -> apple store connect

- aos
    - 앱 만들기
    - 앱 관련 정보, 사진 등록
    - aab 파일 업로드(eas build --profile production 빌드 후 다운 받을 수 있음)
    
- ios
    - eas submit --platform ios 명령어 입력 후 생성된 apple connect app에서 정보 입력
    - 빌드된 파일 선택 후 심사 등록
``` 
