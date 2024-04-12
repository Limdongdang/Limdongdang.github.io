---
layout: single
title: Firebase Web Push Notification
toc: true
toc_label: 'Firebase Web Push Notification'
toc_sticky: true
categories:
    - push_notification
---

웹 Firebase push notification 사용 테스트

## 참고

> [Github 소스 코드](https://github.com/Limdongdang/Fcm_front_test)
> 사용 환경 : Vite + React + TypeScript

## Firebase SDK 등록

firebase console로 가서 firebase 프로젝트 등록 후 sdk등록을 진행합니다

```js
import { initializeApp } from 'firebase/app';

// Firebase 프로젝트 설정
// Firebase Console에서 프로젝트 설정을 확인하고 아래에 복사해 넣으세요
// Firebase 설정 값은 클라이언트에서 노출이 되도 상관은 없다고 하는데 다른 복잡한 방법을 사용해서 숨기기도 가능하다고 합니다
// https://stackoverflow.com/questions/37482366/is-it-safe-to-expose-firebase-apikey-to-the-public
const firebaseConfig = {
    apiKey: import.meta.env.VITE_APIKEY,
    authDomain: import.meta.env.VITE_AUTHDOMAIN,
    projectId: import.meta.env.VITE_PROJECTID,
    storageBucket: import.meta.env.VITE_STORAGEBUCKET,
    messagingSenderId: import.meta.env.VITE_MESSAGINGSENDERID,
    appId: import.meta.env.VITE_APPID,
    measurementId: import.meta.env.VITE_MEASUREMENTID,
};
// Initialize Firebase
const app = initializeApp(firebaseConfig);
```

## VAPID KEY를 사용해 토큰 발급

```js
const fcmToken = await getToken(messaging, {
    // vapidKey는 Firebase Console에서 프로젝트 설정 > 클라우드 메시징 > 웹 구성 메뉴에서 확인 가능
    // vapidkey를 사용해 fcm push 서버에 클라이언트를 등록하고 토큰을 발급받아 서버로 전송해서 사용함
    vapidKey: import.meta.env.VAPIDKEY,
});
```

## 등록 토큰 액세스

**알림 권한 요청**

`Notification.requestPermission()` 함수를 사용하여 사용자에게 알림 권한을 요청합니다

```js
const permission = await Notification.requestPermission();
// Notification.requestPermission()은 Promise를 반환함 granted / denied / default 중 하나
// granted: 사용자가 알림을 허용함
// denied: 사용자가 알림을 거부함
// default: 사용자가 알림 허용을 거부하거나 브라우저가 알림 허용을 지원하지 않음
if (permission === 'granted') {
    console.log('Notification permission granted.');
} else {
    console.log('Unable to get permission to notify.');
}
```

## 정상적인 메시지 수신을 위한 서비스 워커 등록

    백그라운드 메시지와 포그라운드 메시지를 정상적으로 받기 위해
    서비스 워커 등록을 합니다
    서비스워커를 작성해 백그라운드 메시지를 받을 수 있다는 데 저는 이상하게도
    서비스 워커를 작성하지 않아도 백그라운드 메시지를 받을 순 있었습니다
    다만 메시지 내용이 이상하고 메시지를 받는 이벤트를 통해
    메시지 처리 로직을 짜려면 서비스 워커를 등록해야 합니다

```js
importScripts('https://www.gstatic.com/firebasejs/8.10.1/firebase-app.js');
importScripts('https://www.gstatic.com/firebasejs/8.10.1/firebase-messaging.js');

firebase.initializeApp({
    apiKey: '',
    authDomain: '',
    projectId: '',
    storageBucket: '',
    messagingSenderId: '',
    appId: '',
    measurementId: '',
});

const messaging = firebase.messaging();
```

프로젝트 루트 public 폴더에 firebase-messaging-sw.js 이름으로 파일 생성 및 sdk 등록

## 메시지 수신

**포그라운드 메시지 수신**

사용자가 현재 웹 페이지를 보고 있는 상태에서 메시지 수신

```js
// 포그라운드 알림 수신
onMessage(messaging, (payload) => {
    // payload는 알림 메시지의 데이터를 가지고 있음
    console.log('Foreground Message received. ', payload);

    // 알림을 표시하는 코드
    new Notification(payload.notification?.title ?? 'Default Title', {
        body: payload.notification?.body,
    });
});
```

**백그라운드 메시지 수신**

웹이 꺼져 있거나 다른 창을 보고 있을 시 메시지 수신
서비스 워커를 통해서 접근 가능

```js
// firebase-messaging-sw.js ...

// 백그라운드 상태에서 메시지 수신
messaging.onBackgroundMessage(function (payload) {
    // 여기에 알림 메시지 처리 코드를 추가
    console.log('[firebase-messaging-sw.js] Received background message ', payload);
});
```

## 메시지 수신 테스트

firebase console >> FCM프로젝트 >> Messaging >> 첫 번째 캠페인 만들기

제목, 본문 내용 + 발급받은 토큰으로 포그라운드, 백그라운드 메시지 수신 테스트 가능
