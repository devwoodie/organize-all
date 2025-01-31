## 비밀번호 해시화하기

```bash
"crypto-js": "^4.2.0",
```

```ts
import CryptoJS from 'crypto-js';

// 랜덤 salt 생성
export const generateSalt = (length: number) => {
  return CryptoJS.lib.WordArray.random(length).toString();
};

// password + salt 생성
export const hashPassword = (password: string, salt: string) => {
  const hash = CryptoJS.SHA256(password + salt).toString(CryptoJS.enc.Hex);
  return { salt, hash };
};
```