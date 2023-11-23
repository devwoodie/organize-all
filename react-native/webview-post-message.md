## Webview PostMessage (데이터 주고 받기)

### react-native webview 옵션 세팅
```js
const webview = useRef();
<WebView
    style={{backgroundColor: "#000"}}
    overScrollMode="never"
    ref={webview}
    mixedContentMode={'always'}
    mediaPlaybackRequiresUserAction={false}
    javaScriptEnabled={true}
    domStorageEnabled={true}
    startInLoadingState={true}
    originWhitelist={['*']}
    allowsInlineMediaPlayback={true}
    injectedJavaScript={debugging}
    onNavigationStateChange={onNavigationStateChange}
    // onMessage setting
    onMessage={(e) => {
        web_to_native(e)
    }}
    source={{
        uri: uri,
    }}
/>
```

### react-native -> react
- react-native (보내기)
```js
// type, data 전송 (react에서 받은 데이터를 type으로 구분하기 위함)
await webview.current.postMessage(JSON.stringify({
    type: 'type',
    data:{
        key1: "postData1",
        key2: "postData2"
    }
}));
```
- react (받기)
```js
const listenerData = () => {
    const {key1, key2} = data;
}

const receiveMessage = (event) => {
    // 받은 데이터(type, data)
    const {type, data} = JSON.parse(event.data);

    if(type ==='type'){
        // 만들어 놓은 함수에 data 전송
        listenerData(data)
    }
}
    
useEffect(() => {
    // ios
    window.addEventListener('message', receiveMessage);
    // aos
    document.addEventListener('message', receiveMessage);

    return() => {
        // ios
        window.removeEventListener('message', receiveMessage);
        // aos
        document.removeEventListener('message', receiveMessage);
    }
}, []);
```

### react -> react-native
- react (보내기)
```js
export const postNativeMessage = async(message, callback) => {
    return new Promise((resolve, reject) => {
        if (window.ReactNativeWebView) {
            const listener = async(event) => {
                // setIsChecked(data)

                clearTimeout(timer);
                /** android */
                document.removeEventListener('message', listener);
                /** ios */
                window.removeEventListener('message', listener);
                resolve(JSON.parse(event.data).result);
            };

            window.ReactNativeWebView.postMessage(message);
            const TIMEOUT = 10000;
            const timer = setTimeout(() => {
                /** android */
                document.removeEventListener('message', listener);
                /** ios */
                window.removeEventListener('message', listener);
                reject('TIMEOUT');
            }, TIMEOUT);

            if(callback){
                /** android */
                document.addEventListener('message', listener);
                /** ios */
                window.addEventListener('message', listener);
            }else {
                clearTimeout(timer);
                return
            }
        } else {
            return
        }
    })
}

// 사용
await postNativeMessage(JSON.stringify({type: "type1", data: ''}), true);
```
- react-native (받기)
```js
const web_to_native = async (e) => {
    const {type, data} = JSON.parse(e.nativeEvent.data);

    if (type === "type1") {
        // 실행 로직
        console.log(data);
    }
}
```
