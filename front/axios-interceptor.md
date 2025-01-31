## Axios Interceptors 세팅

1. 기본 interceptor 정의

```ts
// axiosInstance.ts
import axios, { AxiosInstance, AxiosResponse, AxiosError } from 'axios';
import toast from 'react-hot-toast';

import { refreshTokenApi } from '@/apis/authApi';
import { signOutFunc } from '@/utils/auth';

export const axiosInstance: AxiosInstance = axios.create({
  baseURL: process.env.NEXT_PUBLIC_BACKEND_URL,
  timeout: 10000,
  headers: {
    'Content-Type': 'application/json',
  },
})

const saveTokens = (accessToken: string, refreshToken?: string) => {
  localStorage.setItem('uvs-access-token', accessToken);
  if (refreshToken) {
    localStorage.setItem('uvs-refresh-token', refreshToken);
  }
};

axiosInstance.interceptors.request.use(
  async (config) => {
    const accessToken = localStorage.getItem('uvs-access-token');
    if (accessToken) {
      config.headers.Authorization = `Bearer ${accessToken}`;
    }

    return config;
  },
  (error) => {
    return Promise.reject(error);
  }
);

axiosInstance.interceptors.response.use(
  (response: AxiosResponse) => response,
  async (error: AxiosError) => {
    const originalRequest = error.config;
    if (originalRequest && error.response?.status === 401) {
      const refreshToken = localStorage.getItem('uvs-refresh-token');
      if (refreshToken) {
        try {
          const res = await refreshTokenApi(refreshToken);
          saveTokens(res?.token, refreshToken);
          axiosInstance.defaults.headers.common.Authorization = `Bearer ${res?.token}`;
          originalRequest.headers.Authorization = `Bearer ${res?.token}`;

          return axiosInstance(originalRequest);
        } catch (refreshError) {
          toast.error('Refresh token expired or invalid.');
          await signOutFunc();

          return Promise.reject(refreshError);
        }
      }else{
        toast.error('Refresh token expired or invalid.');
        await signOutFunc();
      }
    }

    return Promise.reject(error);
  }
);
```

<br/>

2. custom api 세팅

```ts
import { axiosInstance } from '@/apis/axiosInstance';
import { TGetProps, TPostProps } from '@/types/axiosType';
import { AxiosError } from 'axios';

export const fetchData = async ({
  url,
  params,
  config
}: TGetProps) => {
  try{
    const response = await axiosInstance.get(`${url}`, {
      params: params,
      ...config
    });

    return { response: response?.data, status: response?.status };
  }catch (error: unknown) {
    if (error instanceof AxiosError) {
      return { response: error?.response?.data, status: error?.status };
    }

    return Promise.reject(error);
  }
}

export const postData = async ({
  url,
  requestMethod,
  body
}: TPostProps) => {
  try{
    const response = await axiosInstance({
      url: `${url}`,
      method: requestMethod,
      data: body,
    });

    return { response: response?.data?.data, status: response?.status };
  }catch (error) {
    if (error instanceof AxiosError) {
      return { response: error?.response?.data, status: error?.status };
    }

    return Promise.reject(error);
  }
}
```

<br/>

3. access token 만료시 refresh 세팅
```ts
import axios from 'axios';

export const refreshTokenApi = async (refreshToken: string) => {

    const response = await axios.get(`${process.env.NEXT_PUBLIC_BACKEND_URL}/auth/refresh`, {
        headers: {
            Authorization: `Bearer ${refreshToken}`
        }
    });

    return response.data;
};
```