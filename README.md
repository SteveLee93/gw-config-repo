`config-server`의 `application.yml`과 Git 저장소에 있는 설정 파일(`application.yml`)은 서로 다른 역할을 수행합니다. **Config Server의 `application.yml`**은 Config Server 자체의 설정을 정의하고, **Git 저장소의 설정 파일**은 Config Server가 제공하는 클라이언트 애플리케이션에 전달될 설정을 정의합니다.

아래에서 두 설정 파일의 차이점과 설정 방법을 단계별로 설명하겠습니다.

---

## **1. Config Server의 `application.yml`**

Config Server의 `application.yml`은 Config Server 자체를 설정하는 파일입니다. 이 설정은 Config Server가 Git 저장소와 통신하고 설정 파일을 클라이언트에게 제공하는 역할을 합니다.

### **Config Server의 `application.yml` 예제**
```yaml
server:
  port: 8888 # Config Server가 사용할 포트 번호

spring:
  application:
    name: config-server # Config Server의 애플리케이션 이름
  cloud:
    config:
      server:
        git:
          uri: https://github.com/SteveLee93/gw-config-repo # Git 저장소 URL
          default-label: main # 기본 브랜치 (main)
          username: your-username # (필요한 경우) Git 저장소 인증 정보
          password: your-password # (필요한 경우) Git 저장소 인증 정보

logging:
  level:
    org.springframework: INFO
```

#### **포인트**
1. **Git 저장소 URL**:
   - `uri` 필드는 Config Server가 연결할 Git 저장소의 URL을 설정합니다.
   - 이 저장소에서 설정 파일을 읽습니다.

2. **Git 인증 정보**:
   - 공개 저장소라면 `username`과 `password`를 생략할 수 있습니다.
   - 비공개 저장소인 경우, Git 인증 정보가 필요합니다.

3. **기본 브랜치**:
   - `default-label`은 Config Server가 기본적으로 읽을 브랜치를 설정합니다.

---

## **2. Git 저장소의 설정 파일 (클라이언트용)**

Git 저장소에는 Config Server가 읽어서 클라이언트 애플리케이션에 제공할 설정 파일이 저장됩니다. 파일 이름은 클라이언트 애플리케이션 이름에 따라 결정됩니다.

### **Git 저장소 구조**
Git 저장소의 구조는 일반적으로 다음과 같습니다:
```
gw-config-repo/
│
├── application.yml
├── auth-service.yml
├── user-service.yml
└── gateway-service.yml
```

- **`application.yml`**: 모든 애플리케이션에 공통으로 적용되는 설정.
- **`auth-service.yml`**: `auth-service` 클라이언트 전용 설정.
- **`user-service.yml`**: `user-service` 클라이언트 전용 설정.

---

### **Git 저장소의 설정 파일 예제**

#### **1. `application.yml` (공통 설정)**
```yaml
logging:
  level:
    root: INFO
    org.springframework.web: DEBUG

spring:
  datasource:
    driver-class-name: org.h2.Driver
    url: jdbc:h2:mem:testdb
    username: sa
    password: password
```

- 모든 애플리케이션에 공통으로 적용되는 설정입니다.

#### **2. `auth-service.yml` (Auth Service 전용 설정)**
```yaml
server:
  port: 8081

spring:
  application:
    name: auth-service # 애플리케이션 이름
  datasource:
    url: jdbc:mysql://localhost:3306/authdb
    username: authuser
    password: authpassword
```

- `auth-service`에만 적용되는 데이터베이스 설정과 포트 정보를 포함합니다.

#### **3. `user-service.yml` (User Service 전용 설정)**
```yaml
server:
  port: 8082

spring:
  application:
    name: user-service # 애플리케이션 이름
  datasource:
    url: jdbc:mysql://localhost:3306/userdb
    username: useruser
    password: userpassword
```

- `user-service`에만 적용되는 설정입니다.

---

## **3. Config Server와 클라이언트의 연동**

### **클라이언트 애플리케이션의 `application.yml`**
Config Server에서 설정을 가져오는 클라이언트 애플리케이션은 다음과 같이 설정해야 합니다:

```yaml
spring:
  application:
    name: auth-service # 애플리케이션 이름 (Config Server에서 검색)
  cloud:
    config:
      uri: http://localhost:8888 # Config Server의 주소
      fail-fast: true # Config Server와 연결 실패 시 애플리케이션 실행 중단
```

#### **작동 방식**
1. 클라이언트 애플리케이션(`auth-service`)이 Config Server에 요청을 보냅니다.
2. Config Server는 Git 저장소에서 `auth-service.yml`과 `application.yml`을 읽어 클라이언트에 제공합니다.
3. 클라이언트 애플리케이션은 이 설정 값을 기반으로 실행됩니다.

---

## **4. 요청 흐름**

1. **Config Server**:
   - Config Server는 Git 저장소에서 설정 파일을 읽어옵니다.
   - 예: `application.yml`과 `auth-service.yml`을 조합하여 클라이언트에 제공.

2. **클라이언트 애플리케이션**:
   - 클라이언트 애플리케이션(`auth-service`)은 Config Server에서 설정 값을 가져옵니다.
   - Config Server가 제공한 설정 값을 기반으로 초기화됩니다.

---

## **5. 실행 순서**
1. **Config Server 실행**:
   - `config-server` 애플리케이션을 먼저 실행합니다.
   - Git 저장소와 연결되는지 확인합니다.

2. **클라이언트 애플리케이션 실행**:
   - `auth-service`, `user-service`와 같은 클라이언트를 실행합니다.
   - 실행 시 Config Server와 통신하여 설정 값을 가져옵니다.

3. **Config Server 동작 확인**:
   - 브라우저에서 [http://localhost:8888/auth-service/default](http://localhost:8888/auth-service/default)를 열어 `auth-service`의 설정 값을 확인합니다.

---

## **결론**
1. **Config Server의 `application.yml`**:
   - Config Server의 Git 저장소 설정, 포트 등을 정의합니다.
2. **Git 저장소의 설정 파일**:
   - 클라이언트 애플리케이션별로 필요한 설정을 저장합니다.
3. **클라이언트의 `application.yml`**:
   - Config Server와 통신하여 설정을 가져옵니다.

이 구조를 설정하면, 설정 값을 중앙에서 관리하고 각 애플리케이션에서 동적으로 사용할 수 있습니다. 추가 질문이 있거나 문제가 발생하면 언제든지 알려주세요! 😊
