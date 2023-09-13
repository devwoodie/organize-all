## React Native Android apk파일 추출 (release 모드)
> - 안드로이드 스튜디오에서 진행

1. keystore 만들기
```bash
 keytool -genkey -v -keystore skyksit.keystore -alias skyksit -keyalg RSA -keysize 2048 -validity 10000
```
- keystore : 이름
- alias : 별칭
- validity : 키 유효기간

이름, 조직단위명, 조직이름, 지역(시), 지역(도), 나라(ko) 입력 후 `y` 누르면 keystore 파일 생성

2. 생성된 keystore 파일 루트/app 폴더 내부로 이동
3. bundle 파일 만들기
```bash
## assets 폴더 없으면 만들기
mkdir android/app/src/main/assets
```
```bash
## assets 폴더 내에 index.android.bundle 파일 생성
npx react-native bundle --platform android --dev false --entry-file index.js --bundle-output android/app/src/main/assets/index.android.bundle --assets-dest android/app/src/main/res
```
4. app/gradle.properties에 keystore 설정
```dotenv
MYAPP_RELEASE_STORE_FILE=app-release-key.keystore
MYAPP_RELEASE_KEY_ALIAS=app-release
MYAPP_RELEASE_STORE_PASSWORD=dbehddn
MYAPP_RELEASE_KEY_PASSWORD=dbehddn 
```
5. android/app/build.gradle 에 release 설정 추가
```
...
android {
    ...
    defaultConfig { ... }
    signingConfigs {
        debug {
            storeFile file('debug.keystore')
            storePassword 'android'
            keyAlias 'androiddebugkey'
            keyPassword 'android'
        }
        release {
            storeFile file('app-release-key.keystore')
            storePassword 'dbehddn'
            keyAlias 'app-release'
            keyPassword 'dbehddn'
        }
    }
    buildTypes {
        release {
            ...
            signingConfig signingConfigs.release
        }
    }
}
...
```
6. 앱 빌드
```bash
cd android
./gradlew build
## /android/app/build/outputs/apk/release 에 apk 파일 생성
```
