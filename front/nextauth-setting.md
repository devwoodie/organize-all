## NextAuth 기본 세팅

```bash
"next-auth": "^4.24.10",
```

1. src/app/api/auth/[...nextauth]/route.ts 경로에 파일 생성

```ts
// route.ts

import CredentialsProvider from "next-auth/providers/credentials"
import NextAuth from 'next-auth';

const handler = NextAuth({
  secret: process.env.NEXTAUTH_SECRET,
  pages: {
    signIn: '/sign-in',
  },
  providers: [
    CredentialsProvider({
      name: "Credentials",
      credentials: {
        email: { },
        password: { }
      },
      async authorize(credentials) {
        // 백엔드 로그인 api 호출
        const res = await fetch(`${process.env.NEXT_PUBLIC_BACKEND_URL}/auth/login`,{
          method: 'POST',
          headers: {
            'Content-Type': 'application/json',
          },
          body: JSON.stringify({
            email: credentials?.email,
            password: credentials?.password,
          }),
        });

        const user = await res.json();
        if (user?.message === 'success') {
          return user?.data
        } else {
          throw new Error(String(res?.status));
        }
      }
    }),
  ],
  callbacks:{
    async jwt({ token, user }) {
      return { ...token, ...user };
    },
    async session({ session, token }) {
      session.user = token as never;
      return session;
    },
  }
})

export { handler as GET, handler as POST }
```

<br/>

2. src 폴더 내부에 middleware.ts 파일 생성

```ts
// middleware.ts
import { NextRequest, NextResponse } from "next/server";
import { getToken } from "next-auth/jwt";

export async function middleware(request: NextRequest) {

  const pathUrl = request.nextUrl.pathname;
  const authToken = request.cookies.get("next-auth.auth-level");

  try {
    const decodedToken = await getToken({
      req: request,
      secret: process.env.NEXTAUTH_SECRET,
    });
    if (!decodedToken) {
      const response = NextResponse.redirect(new URL("/sign-in", request.url));
      response.headers.set(
        'Cache-Control',
        'no-store, no-cache, must-revalidate, proxy-revalidate',
      );
      return response;
    }

  } catch (error) {
    console.log(error)
    return NextResponse.redirect(new URL("/sign-in", request.url));
  }

  if (authToken?.value === 'USER' && pathUrl.startsWith('/admin')) {
    const response = NextResponse.redirect(new URL("/sign-in", request.url));
    response.headers.set(
      'Cache-Control',
      'no-store, no-cache, must-revalidate, proxy-revalidate',
    );
    return response;
  }

  return NextResponse.next();
}

// 경로 핸들링
export const config = {
  matcher: [
    "/cloud/:path*",
    "/sdk/:path*",
    "/prod/:path*",
    "/admin/:path*",
    "/home",
    "/activity-log"
  ],
};
```

<br/>

3. root 경로에 next-auth.ts 파일 생성(session 타입 정의)

```ts
import NextAuth from "next-auth";

declare module "next-auth" {
  interface Session {
    user: {
      org:{
        id: string;
        email: string;
        feature: {
          id: string;
          sub: [];
          name: string;
          path: string
        }
        locale: string;
        role: "USER" | "ADMIN";
        name: string;
        status: string;
        manager_name: string;
      }
      token: string;
      refreshToken: string;
    };
  }
}
```

<br/>

4. env 정의

```bash
NEXTAUTH_URL=http://192.168.0.66:3001
NEXTAUTH_SECRET=secretkey
```

<br/>

5. 로그인 

```ts
const res = await signIn('credentials', {
    email: email || '',
    password: hash || '',
    redirect: false,
    callbackUrl: '/',
});
```