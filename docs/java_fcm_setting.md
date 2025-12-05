## 🚀 Java Spring + Gradle 기반 FCM 연동 서버 셋업 매뉴얼

본 문서는 발급된 **Firebase Service Account Key**를 사용하여 Java Spring Boot 애플리케이션에 FCM(Firebase Cloud Messaging) 메시지 전송 기능을 설정하는 방법을 안내합니다.

-----

## 1\. 전제 조건 및 서비스 키 위치

### 1.1. Service Account Key 파일 준비

  * **파일 이름:** `serviceAccountKey.json` (혹은 프로젝트 정책에 따른 이름)
  * **파일 위치:** 서버 개발의 표준적인 관행에 따라, 클래스패스 내의 별도 폴더에 저장하여 관리합니다.

$$\text{src/main/resources/firebase/serviceAccountKey.json}$$

### 1.2. 키 파일 로드 원리

파일이 `src/main/resources/firebase`에 위치하므로, Spring에서 해당 파일을 로드할 때는 `classpath:firebase/serviceAccountKey.json` 경로를 사용합니다. `application.yml`에 별도의 설정은 필요 없습니다.

-----

## 2\. Step 1: Gradle 의존성 추가 (Firebase Admin SDK)

Gradle 빌드 파일인 \*\*`build.gradle`\*\*에 Firebase Admin SDK 의존성을 추가합니다.

안정성과 HTTP/2 기반의 성능 개선(다량 메시지 전송 효율 증가)을 위해 **버전 9.4.3**을 사용합니다.

### `build.gradle` 파일 수정

```groovy
dependencies {
    // Spring Boot Starter (기존 의존성)
    // implementation 'org.springframework.boot:spring-boot-starter-web'
    
    // ⭐ Firebase Admin SDK 추가 (권장 안정화 버전 9.4.3)
    implementation 'com.google.firebase:firebase-admin:9.4.3' 
}
```

의존성을 추가한 후, Gradle Refresh 또는 재빌드를 수행합니다.

-----

## 3\. Step 2: Firebase Admin SDK 초기화 (Spring Configuration)

애플리케이션이 시작될 때 서비스 키를 사용하여 Firebase Admin SDK를 한 번만 초기화하는 설정 클래스를 작성합니다.

### `FirebaseConfig.java` 작성

```java
package com.yourproject.config;

import com.google.auth.oauth2.GoogleCredentials;
import com.google.firebase.FirebaseApp;
import com.google.firebase.FirebaseOptions;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.context.annotation.Configuration;
import org.springframework.core.io.Resource;

import javax.annotation.PostConstruct;
import java.io.IOException;

@Configuration
public class FirebaseConfig {

    // 1. serviceAccountKey.json 파일 경로를 클래스패스에서 로드
    // 
    @Value("classpath:firebase/serviceAccountKey.json")
    private Resource serviceAccount; 

    @PostConstruct
    public void initialize() {
        try {
            // 중복 초기화 방지
            if (FirebaseApp.getApps().isEmpty()) {
                
                // 2. Resource 객체에서 InputStream을 얻어 Credentials 생성
                FirebaseOptions options = FirebaseOptions.builder()
                        .setCredentials(GoogleCredentials.fromStream(serviceAccount.getInputStream()))
                        // (선택 사항) 필요 시 Database URL 등 추가 설정 가능
                        .build();

                // 3. Firebase SDK 초기화
                FirebaseApp.initializeApp(options);
                System.out.println("Firebase Admin SDK가 성공적으로 초기화되었습니다.");
            }
        } catch (IOException e) {
            System.err.println("Firebase Admin SDK 초기화 실패: " + e.getMessage());
            // 초기화 실패는 심각한 오류이므로, 적절한 로깅 및 처리 필요
        }
    }
}
```

-----

## 4\. Step 3: FCM 메시지 전송 서비스 구현

초기화가 완료되면, `FirebaseMessaging.getInstance()`를 통해 메시지 전송 객체를 얻어 FCM 메시지 전송 로직을 구현합니다.

### `FcmService.java` 작성 (예시)

```java
package com.yourproject.fcm;

import com.google.firebase.messaging.*;
import org.springframework.stereotype.Service;

import java.util.concurrent.ExecutionException;

@Service
public class FcmService {

    /**
     * 특정 기기 토큰으로 알림 메시지를 전송합니다.
     * @param token 대상 기기의 FCM 토큰
     * @param title 알림 제목
     * @param body 알림 내용
     * @return FCM 서버 응답 ID
     */
    public String sendNotification(String token, String title, String body) throws InterruptedException, ExecutionException {
        
        // 1. Notification (알림창에 표시될 데이터) 구성
        Notification notification = Notification.builder()
                .setTitle(title)
                .setBody(body)
                .build();
        
        // 2. Message (FCM 전송 메시지) 구성
        // 
        Message message = Message.builder()
                .setToken(token) // 특정 토큰 대상 전송
                .setNotification(notification)
                // .putData("key", "value") // Data Payload를 추가하여 커스텀 데이터 전송 가능
                .build();
        
        // 3. 메시지 전송
        try {
            String response = FirebaseMessaging.getInstance().send(message);
            System.out.println("FCM 메시지 전송 성공. Response: " + response);
            return response;
        } catch (FirebaseMessagingException e) {
            System.err.println("FCM 메시지 전송 실패. Error: " + e.getMessage());
            // 토큰 만료, 잘못된 토큰 등 오류 처리 로직 추가 필요
            throw new RuntimeException("FCM 전송 실패", e);
        }
    }
}
```

-----

## 5\. 최종 점검 및 다음 단계

1.  **키 위치 확인:** `src/main/resources/firebase/serviceAccountKey.json`
2.  **의존성 확인:** `build.gradle`에 `com.google.firebase:firebase-admin:9.4.3` 추가
3.  **초기화 확인:** `FirebaseConfig.java`에서 `FirebaseApp.initializeApp()` 호출 확인
4.  **테스트:** `FcmService`를 Controller 등에서 주입받아 실제 클라이언트(앱) 토큰으로 메시지 전송을 테스트합니다.

**다음 주요 과제:** 실제 운영 환경에서 **FCM Device Registration Token**을 서버 데이터베이스에 저장하고, 사용자 이벤트 발생 시 이 토큰을 조회하여 `FcmService`를 통해 메시지를 전송하는 로직을 구현해야 합니다.