# 개념 정리 — Redis / Kafka / OAuth / DB 라우팅

- 작성일: 2026-07-31 (DB 라우팅 추가)
- 목적: 인프라 성격의 기술들의 **개념**을 잡고, **lgoneid 프로젝트에서 실제로 어디에 쓰이는지** 연결해서 이해하기.
- 깊은 이론보다 "왜 쓰는가 / 우리 코드 어디에 있는가 / 뭘 조심해야 하는가" 중심.
- 인증 관련(쿠키·세션·토큰)은 별도 문서: `개념정리_쿠키_세션_토큰_인증.md`

---

# 1. Redis

## 한 줄 요약
**메모리에 데이터를 두는 초고속 key-value 저장소.** DB(디스크)보다 훨씬 빠르지만, 메모리라 용량이 작고 영속성이 약하다. → **"자주 읽지만 자주 바뀌지 않는 것"** 을 얹어두는 데 쓴다.

## 왜 쓰나
| 용도 | 설명 |
|--|--|
| **캐시** | DB 조회 결과를 담아두고 다음부터는 Redis에서 바로 읽음 (DB 부하↓, 응답속도↑) |
| **세션 저장** | 서버가 여러 대일 때 세션을 공유. (서버 메모리에 두면 다른 서버로 요청이 가면 로그인이 풀림) |
| **카운터 / 임시값** | 인증 시도 횟수, 만료 시간 있는 토큰 등. **TTL(만료시간)** 을 걸 수 있는 게 큰 장점 |

## 자료구조 (알아두면 좋은 것만)
- **String**: 가장 단순한 `key = value`
- **Hash**: `key` 안에 `field = value` 여러 개. **우리 공통코드 캐시가 이 방식**
  (예: key=`공통코드`, field=`CI_ENC:SWITCH`, value=`ON`)
- 그 외 List / Set / Sorted Set 등이 있지만 지금은 몰라도 됨

## lgoneid 에서
- 서버: `ec-redis-lgoneid-dev...` (AWS ElastiCache), 포트 6379
- **공통코드 캐시** — `admin-service/.../common/config/redis/RedisCacheHandler.java`
  `putOpsForHash` / `getOpsForHash` / `deleteAllOpsForHash` → **Hash 구조**로 공통코드를 얹어둠
- admin-web-service 는 **세션 저장**에도 사용 (`common/config/SessionConfig.java`)
- 인증 재시도 횟수 같은 **임시 카운터** 로도 사용 (`MemberAccHandlerImpl`)

## ⚠️ 우리가 실제로 당한 함정
**DB 공통코드를 UPDATE 했는데 앱 동작이 안 바뀜** — 앱은 Redis 캐시를 읽기 때문이다.

캐시의 본질적 트레이드오프다. **빠른 대신 "원본과 잠깐 다를 수 있다"(stale)**. 그래서 "DB는 바꿨는데 앱은 옛 값" 증상이면 **캐시를 1순위로 의심**해야 한다. (해결은 아래 "캐시 갱신 호출 URL")

### 자동으로 갱신되나? → 아니다 (코드 확인 결과)
| 확인 항목 | 결과 |
|--|--|
| **TTL(만료시간)** | **없음** — `RedisCacheHandler`/`RedisCacheConfig` 에 `expire`·`entryTtl` 설정 전무 → 삭제·덮어쓰기 전까지 **영구 유지** |
| 캐시 미스 시 | **DB 폴백은 있음** (`CommonCodeServiceImpl` ~50, ~65) → 서비스는 정상 동작 |
| 폴백으로 읽은 값을 캐시에 다시 채우나 | **안 채움** (재적재 코드 없음) |
| 갱신 주체 | `admin-service/.../commoncode/controller/CommonCodeRedisCacheController` 의 엔드포인트 3개 |

동작 정리:
```
캐시에 값 있음 + DB만 변경   → 계속 옛 값 (시간이 지나도 안 바뀜)   ← 우리가 당한 케이스
캐시가 비어 있음             → DB 폴백으로 최신값 (단 캐시에 채우진 않음)
refresh 호출                → DB 기준으로 재적재 → 반영
```

### 어드민 화면에서 수정하면 캐시도 갱신되나? → 코드상 아니다
- 공통코드 관리 화면(`admin-web-service/.../code/controller/CmnCodeMngJspController.java`)에 **캐시 갱신 호출이 없다.**
- `admin-service` 의 공통코드 저장 로직도 **읽기에만** 캐시를 쓰고 저장 시 갱신하지 않는다.
- 정작 `refreshCommonCodeRedisCache` 를 호출하는 곳은 **예산 화면**(`admin-web-service/.../reward/controller/BudgetJspController.java:447`) 이다.

→ 즉 **DB를 직접 바꿨든 화면에서 바꿨든, refresh 를 따로 호출해야 한다.** (설계 의도인지 누락인지는 미확인 — 원 담당 확인 필요)

### 캐시 갱신 호출 URL

**(A) 배치 잡 (우리가 쓰던 방법)** — `adminbatch` 서비스 (이 저장소에 없는 별도 프로젝트)

| 환경 | URL |
|--|--|
| **dev (게이트웨이 경유)** | `http://dev.mylgid.com:1080/adminbatch/job/refreshCommoncodeRedisCacheJob` |
| 로컬에서 직접 띄운 경우 | `http://localhost:8330/adminbatch/job/refreshCommoncodeRedisCacheJob` |
| 컨테이너 내부 | `http://adminbatch:8330/adminbatch/job/refreshCommoncodeRedisCacheJob` |

- **GET** 방식이라 브라우저 주소창에 붙여도 된다.
- 파라미터 예(코드 주석에 있는 형태): `?createDate=20231213`
- 출처: `admin-web-service/.../reward/service/AdmBatchIFService.java:18-19` (`@FeignClient(url = "${connect-info.api.admin-batch-url}")`), 환경값은 `admin-web-service/src/main/resources/application-local.yml:50,60` / `application.yml:56`

**(B) admin-service REST 엔드포인트 (배치 없이 직접)** — `admin-service/.../commoncode/controller/CommonCodeRedisCacheController.java`, 공통 경로 `@RequestMapping("/api/v1/admin/common-codes")`

| 메서드 | 경로 | 동작 |
|--|--|--|
| **PUT** | `/api/v1/admin/common-codes` | **refresh** (캐시 삭제 후 재적재) ← 보통 이것 |
| POST | `/api/v1/admin/common-codes` | 캐시 적재(전체 조회 후 put) |
| DELETE | `/api/v1/admin/common-codes` | 캐시 삭제만 |

호출 예 (dev 게이트웨이 경유):
```bash
curl -X PUT http://dev.mylgid.com:1080/admin/api/v1/admin/common-codes
# 로컬에서 admin-service 를 직접 띄운 경우 (port 7320)
curl -X PUT http://localhost:7320/admin/api/v1/admin/common-codes
```
- base 경로는 각 서비스의 `admin-url` 설정과 동일한 규칙이다(`http://dev.mylgid.com:1080/admin`).
- **PUT/POST/DELETE 라 브라우저 주소창으로는 안 되고** curl·Postman 등이 필요하다. (그래서 GET 인 배치 잡이 손에 익기 쉽다)

> 참고: `refresh` 는 **비우고 다시 채우는** 순서라 그 사이 짧은 캐시 미스 구간이 생기지만, 위 DB 폴백 덕분에 서비스는 끊기지 않는다.

---

# 2. Kafka

## 한 줄 요약
**메시지를 줄 세워 흘려보내는 파이프.** 보내는 쪽(Producer)과 받는 쪽(Consumer)이 **직접 호출하지 않고 분리**된다. 받는 쪽이 잠깐 죽어도 메시지는 남아 있다가 처리된다.

## 왜 쓰나 — 동기 호출과 비교
```
[동기 호출]  A ──직접 호출──> B
   B가 느리면 A도 느려지고, B가 죽으면 A도 실패한다.

[Kafka]      A ──발행──> [토픽(큐)] ──구독──> B
   A는 던지고 바로 끝(빠름). B는 자기 속도로 처리. B가 죽어도 메시지는 토픽에 남아있다.
```
→ **로그 적재, 이력 저장, 알림 발송** 처럼 "즉시 결과가 필요 없는 부수 작업"에 잘 맞는다.

## 용어 (딱 4개만)
| 용어 | 뜻 |
|--|--|
| **Producer** | 메시지를 보내는 쪽 |
| **Topic** | 메시지가 쌓이는 이름 붙은 통로 (예: `dev.myid.ci-coll-log`) |
| **Consumer** | 토픽을 구독해서 꺼내 처리하는 쪽 |
| **Consumer Group** | 같은 그룹끼리는 메시지를 나눠 처리(분산). 그룹이 다르면 각자 전부 받음 |

## lgoneid 에서
- AWS **MSK**(관리형 Kafka) 사용. 인증은 `SASL_SSL` + `AWS_MSK_IAM`
- 토픽 이름에 환경이 붙는다: `local.myid.ci-coll-log` / `dev.myid.ci-coll-log`
- **공통 모듈**: `common-kafka` — `CustomKafkaProducer`(발행), `KafkaConsumerConfig`(구독 설정)
- **소비 전용 서비스**: `login-kafka-service`, `affiliates-kafka-service`
  - `CiCollLogConsumer` — CI 수집로그를 받아 DB에 insert/update
  - `ApiCallInfoConsumer`, `CpsTrsRcvLogConsumer` 등 로그성 컨슈머들

## 우리 작업과 직접 연결된 예 (CI 수집로그)
`CiCollLogUtil` 안에 **스위치로 두 경로**가 있다:
```java
if (IS_USE_CI_COLL_LOG_KAFKA) {
    userwebKafkaCiCollLogService.sendCiCollLogMessage(dto, "create");  // Kafka 경로
} else {
    loginService.createCiCollLog(dto);                                 // DB 직접 호출
}
```
- **Kafka 경로**: user-web → 토픽 발행 → `login-kafka-service`의 `CiCollLogConsumer` → DB insert
- **직접 경로**: user-web → login-service(Feign) → DB insert ← *우리가 SNS CI 수집로그 테스트할 때 이 경로였다*

### 환경별 스위치 값 (`user-web.kafka.topics.ci-coll-log.is-use`)
| 환경 | 값 | 실제 경로 |
|--|--|--|
| local | `false` | login-service Feign → DB 직접 |
| dev / stg / prod | `true` | **Kafka** → `login-kafka-service` → DB |

- **운영·개발서버는 이미 Kafka**, **로컬만 직접 호출**이다. (수신측 `login-kafka-service` 는 local 도 `is-use: true` 라 컨슈머는 로컬에서도 대기 중)
- 토픽 이름에 환경 접두어: `local.myid.ci-coll-log` / `dev.myid.ci-coll-log` / `prod.myid.ci-coll-log`
- **중요**: 우리가 추가한 SNS CI 수집로그는 `ciCollLogUtil.createCiCollLog(...)` 호출 한 줄이므로 **이 스위치를 그대로 따른다**(소셜만 따로 처리하지 않음). 게다가 호출을 try/catch 로 감싸 **발행/적재 실패가 로그인 흐름을 깨지 않는다**.
- 로컬에서 Kafka 경로를 검증하려면 user-web 의 `is-use` 만 `true` 로 바꾸고 `login-kafka-service` 를 띄우면 된다(로컬 설정이라 커밋 제외).

## 알아두면 좋은 특성
- **재시도 설정**이 있다 — `retry-max-count`, `retry-sleep`, `nack-sleep`(실패 시 잠시 뒤 재처리)
- **비동기의 대가**: 발행 성공 ≠ 처리 완료. 그래서 **바로 DB를 조회하면 아직 없을 수 있다**(eventual consistency)
- 컨슈머가 꺼져 있으면 메시지가 **쌓여 있다가** 뜰 때 몰려서 처리된다

---

# 3. OAuth 2.0

## 한 줄 요약
**"비밀번호를 주지 않고, 다른 서비스의 내 정보 일부를 쓰게 허락하는 표준."**
네이버/카카오 로그인이 바로 이것 — 우리는 사용자의 네이버 비밀번호를 절대 모른다. 대신 네이버가 **"이 사람 맞다"** 고 보증해 준다.

## 핵심 흐름 (Authorization Code 방식 — 우리가 쓰는 것)
```
(1) 우리 서비스 → 네이버 로그인 화면으로 보냄
      /oauth2.0/authorize?client_id=...&redirect_uri=...&state=...&response_type=code
(2) 사용자가 네이버에서 로그인 + 동의
(3) 네이버 → 우리 콜백 주소로 되돌려줌 (?code=xxx&state=yyy)
      ※ code = "인증했다는 단기 교환권" (그 자체로는 정보 조회 불가)
(4) 우리 서버 → 네이버 토큰 API 에 code 를 보내 access_token 을 받음  ← 서버끼리(뒤에서) 통신
(5) 우리 서버 → access_token 으로 프로필 API 호출 → 이름/이메일/CI 등 수신
```
**왜 code를 한 번 더 교환하나?** (3)은 브라우저를 통해 오므로 노출 위험이 있다. 실제 열쇠인 `access_token` 은 **서버끼리** 주고받아 브라우저에 노출되지 않게 한다.

### 프론트 채널 vs 백 채널 (code 의 존재 이유)
| 구간 | 통로 | 위험 | 대응 |
|--|--|--|--|
| code 전달 | **브라우저 리다이렉트**(URL에 노출) | 로그·히스토리·Referer 에 남을 수 있음 | **1회용 + 짧은 수명**, 그 자체로는 정보 조회 불가 |
| 토큰 발급·갱신 | **서버 ↔ 제공자 직접**(백 채널) | 브라우저를 안 거침 | `client_secret` 으로 검증 |

- 그래서 `client_secret` 은 **절대 브라우저로 나가지 않는다** (우리도 `secret-local.yml` 에만 둠).
- `state` 가 필요한 이유도 같다 — 브라우저 구간이라 **다른 사이트가 위조 요청을 끼워넣을 수 있어서**, 세션에 저장한 값과 대조한다.
- 한 줄로: **code 는 신뢰할 수 없는 통로(브라우저)를 건너기 위한 징검다리**다. 건너간 뒤엔 서버-서버만 쓰니 다시 필요 없다.

### access_token 과 refresh_token 은 같은 code 로 함께 받는다
`code` **한 번의 교환으로 두 토큰이 같은 응답에 함께** 내려온다. 우리 코드도 동일 응답에서 둘 다 꺼낸다:
```java
// SnsNaverController ~261 (카카오도 SnsKakaoController ~173 동일)
snsLoginDto.setAccessToken(String.valueOf(authResultMap.get("access_token")));
snsLoginDto.setRefreshToken(String.valueOf(authResultMap.get("refresh_token")));
```
이후 **갱신에는 code 가 필요 없다** — 네이버는 `grant_type` 으로 용도를 구분하고, 우리 코드에 3가지가 이미 구현되어 있다(`SnsNaverController` ~273~298):

| grant_type | 용도 | 보내는 값 |
|--|--|--|
| `authorization_code` | **발급** | client_id, client_secret, **code**, state |
| `refresh_token` | **갱신** | client_id, client_secret, **refresh_token** |
| `delete` | 연동 해제 | client_id, client_secret, **access_token** |

- 토큰은 `TB_MEM_SNS` 에 저장하고 `updateSnsTokenInfo` 로 갱신한다.
- 우리 소셜 로그인은 매번 새로 로그인(code 교환)하므로 refresh 를 자주 쓰지 않는다. refresh 는 **사용자 없이 백그라운드에서 제공자 API 를 호출**할 때 필요하다(연동 상태 확인·해제 등).

## 용어
| 용어 | 뜻 | lgoneid |
|--|--|--|
| **client_id** | 우리 서비스 식별자(공개) | 네이버 `naver.client.client-id` / 카카오 `kakao.rest-api-key` |
| **client_secret** | 우리 서비스 비밀키(절대 노출 금지) | 네이버만 사용 (`secret-local.yml`), 카카오는 REST API 키로 대체 |
| **redirect_uri** | code 를 돌려받을 우리 주소 | `/sns/naverCallbackApi`, `/sns/kakaoCallback` (제공자 콘솔에 등록된 값과 **정확히 일치**해야 함) |
| **state** | 위조(CSRF) 방지용 임의값 | 세션에 저장 후 콜백에서 **일치 검사** (`KAKAO_LOGIN_STATE`) |
| **access_token** | 프로필 조회용 열쇠(만료 있음) | `SnsLoginDto.accessToken` |
| **refresh_token** | access_token 재발급용 | `SnsLoginDto.refreshToken` |
| **scope / 동의항목** | 어떤 정보까지 받을지 | 이름·이메일·생년·성별·CI 등 (동의 안 하면 값이 안 옴) |

## lgoneid 에서 (소셜 로그인 = OAuth)
- 시작: `GET /sns/naverLogin` / `/sns/kakaoLogin` → 제공자 authorize 로 리다이렉트 (`state` 생성·세션 저장)
- 콜백: `SnsNaverController` / `SnsKakaoController`
  → `state` 위조 검사 → 토큰 발급 → 프로필 조회 → **`SnsLoginDto` 구성** → 세션 `SNS_LOGIN_INFO_DTO`
- 이후는 우리 로직(OAuth 아님): `snsChk` → 기존회원 후보 → 선택/연동 or 신규가입

## ⚠️ 주의점
- **제공자마다 주는 정보가 다르다** — 네이버는 CI·전화번호를 주고, 카카오는 CI를 주지만 전화번호는 안 준다(현재 구현 기준)
- **동의항목에 의존** — 사용자가 동의 안 하면 필드가 비어 온다 → 매칭·가입 로직이 영향받음
- **client_secret 은 코드/깃에 올리지 않는다** — 우리도 `secret-local.yml`(gitignore)에 둔다
- **OAuth 는 "인증"이 아니라 원래 "권한 위임"** 이다. 로그인 용도로 쓰는 표준은 엄밀히는 **OpenID Connect(OIDC)** 지만, 실무에서는 소셜 로그인을 통틀어 OAuth 로 부른다

---

# 4. DB 읽기/쓰기 라우팅 (writer / RO 복제본)

## 한 줄 요약
**읽기 전용 트랜잭션은 복제본(read only) DB로, 나머지는 원본(writer) DB로 자동 분배하는 구조.** 조회가 훨씬 많으니 읽기를 복제본으로 분산해 writer 부하를 줄인다.

## Aurora 는 DB 가 두 주소로 열려 있다
| 구분 | 엔드포인트 | 역할 |
|--|--|--|
| **writer**(원본) | `db-clust-lgoneid-dev.cluster-...` | INSERT/UPDATE/DELETE 가능 |
| **복제본**(read only) | `db-clust-lgoneid-dev.cluster-ro-...` | SELECT 만. writer 변경을 복제받음 |

→ 주소 차이는 **`cluster-`(writer) vs `cluster-ro-`(복제본)** 한 군데다.

## 우리 코드의 라우팅 (공통 모듈 `common-db`)
**(1) 판정** — `common-db/.../data/ReplicationRoutingDataSource.java`
```java
protected Object determineCurrentLookupKey() {
    return TransactionSynchronizationManager.isCurrentTransactionReadOnly() ? "read" : "write";
}
```
→ **현재 트랜잭션이 readOnly 인지**만 보고 결정한다.

**(2) 매핑** — `common-db/.../config/DataSourceConfig.java`
```java
dataSourceMap.put("write", masterDataSource);   // writer
dataSourceMap.put("read",  slaveDataSource);    // 복제본(cluster-ro)
routingDataSource.setDefaultTargetDataSource(masterDataSource);   // 판단 불가 시 writer
```

**(3) 지연 연결(필수 장치)** — 같은 파일
```java
return new LazyConnectionDataSourceProxy(routingDataSource);
```
트랜잭션 시작 시점에 커넥션을 바로 잡으면 **아직 readOnly 여부를 모르므로** 라우팅이 틀린다. Lazy 프록시는 **실제 쿼리를 던지는 순간까지 커넥션 획득을 미뤄** 올바른 DB를 고르게 한다.

## 어떻게 갈리나
```
@Transactional(readOnly = true)  → "read"  → 복제본(cluster-ro)
@Transactional                   → "write" → writer
트랜잭션 없음                     → writer (기본값도 writer)
```

## ⚠️ 알아둘 함정
1. **복제 지연(replica lag)** — 보통 수 ms 지만, **쓰고 바로 읽으면**(read-after-write) 아직 안 보일 수 있다. 저장 직후 readOnly 조회는 주의.
2. **`@Transactional(readOnly = true)` 는 단순 힌트가 아니라 라우팅 스위치**다 — 이 프로젝트에선 **어느 DB로 갈지**를 결정한다. 무심코 붙이거나 빼면 대상 DB가 바뀐다.
3. **쓰기 작업에 readOnly 를 붙이면** 복제본으로 가서 실패한다(복제본은 쓰기 불가).
4. DB 툴로 접속할 때도 어느 엔드포인트인지 확인: `SELECT @@innodb_read_only;` → `1` 복제본 / `0` writer.

> 우리가 `CI_ENC/SWITCH` 문제를 추적할 때 이 구조가 **유력 후보**였다(앱은 복제본에서 읽고, DBeaver 는 writer 에 썼으니). 실제 원인은 Redis 캐시였지만, **같은 증상을 만들 수 있는 구조**이므로 함께 기억해둘 것.

---

# 5. 네 가지를 한 그림으로 (우리 소셜 로그인 예)

```
[사용자] ──OAuth──> [네이버/카카오] ──code/token──> [user-web]
                                                      |
                                    (1) 공통코드 스위치 읽기 ─────> [Redis 캐시] (admin)
                                                      |
                                    (2) 회원 조회/연동 ──Feign──> [member-service]
                                                                     |
                                                       readOnly? ──┬──> [복제본 cluster-ro]  (조회)
                                                                   └──> [writer cluster]     (등록·수정)
                                                      |
                                    (3) CI 수집로그 ──Kafka 발행──> [토픽] ──> [login-kafka-service] ──> [DB]
                                                        (또는 Feign 으로 login-service 직접 호출)
```
- **OAuth** = 외부 제공자에게 "이 사람 맞다"를 확인받는 구간
- **Redis** = 자주 읽는 설정/세션을 빠르게 읽는 구간 (단, 갱신 타이밍 주의)
- **Kafka** = 지금 당장 결과가 필요 없는 기록성 작업을 뒤로 넘기는 구간
- **DB 라우팅** = 같은 DB 논리인데 **읽기는 복제본 / 쓰기는 writer** 로 갈리는 구간

---

# 6. 한 줄 비교

| | 목적 | 데이터 성격 | 실패했을 때 |
|--|--|--|--|
| **Redis** | 빠른 읽기(캐시·세션·카운터) | 원본은 다른 곳(DB)에 있음 | 원본에서 다시 읽으면 됨 (단 stale 주의) |
| **Kafka** | 비동기 전달(로그·이력·알림) | 흘러가는 메시지 | 토픽에 남아 재처리 |
| **OAuth** | 외부 신원 확인/권한 위임 | 토큰(만료 있음) | 다시 로그인 요청 |
| **DB 라우팅** | 읽기 부하 분산 | 원본(writer)과 복제본이 거의 같음 | 복제 지연 시 **옛 값**을 읽을 수 있음 |

---

## 부록 — "DB는 바꿨는데 앱은 옛 값" 증상의 후보 정리

우리가 실제로 겪은 순서대로. **위에서부터 확인**하면 빠르다.

| 순위 | 후보 | 확인 방법 |
|--|--|--|
| 1 | **Redis 캐시** (공통코드 등) | 분기 지점에 값 찍는 임시 로그 → 다르면 캐시. **캐시 갱신 호출** — URL 은 1장 "캐시 갱신 호출 URL" 참고 |
| 2 | **읽은 곳이 복제본**(복제 지연) | RO 엔드포인트에 직접 접속해 같은 SELECT 비교 |
| 3 | 앱이 **다른 서비스 인스턴스**를 호출 | yml 의 `*-url` 확인 (로컬 → dev 를 호출하는 경우 많음) |
| 4 | 트랜잭션 미커밋 | DB 툴의 autocommit 상태 확인 |

※ 이 프로젝트는 **로컬 DB 가 없고 모두 DEV DB** 를 보므로, "로컬 DB vs DEV DB" 는 후보가 아니다.

### 자주 쓰는 명령 모음 (복사용)

**공통코드 캐시 갱신** (1순위 대응)
```bash
# (A) 배치 잡 — GET, 브라우저 주소창에 붙여도 됨
http://dev.mylgid.com:1080/adminbatch/job/refreshCommoncodeRedisCacheJob

# (B) admin-service REST — refresh(PUT)
curl -X PUT http://dev.mylgid.com:1080/admin/api/v1/admin/common-codes
```

**지금 접속한 DB 가 writer 인지 복제본인지** (2순위 확인)
```sql
SELECT @@innodb_read_only;   -- 0=writer, 1=복제본(read only)
SHOW GLOBAL STATUS LIKE 'Aurora_replica_lag%';   -- 복제본에서만 값이 나옴
```

**공통코드 스위치 값 확인/변경** (예: `CI_ENC/SWITCH`)
```sql
SELECT CODE_ID, CODE, CODE_NM FROM admin.TB_ADM_CODE_DT
 WHERE CODE_ID='CI_ENC' AND CODE='SWITCH';

UPDATE admin.TB_ADM_CODE_DT SET CODE_NM='OFF'   -- 바꾸는 건 CODE 가 아니라 CODE_NM
 WHERE CODE_ID='CI_ENC' AND CODE='SWITCH';
-- 변경 후 반드시 위 캐시 갱신 호출
```
⚠️ 공통코드·DB 는 **DEV 공용**이다. 스위치를 바꾸면 다른 사람 작업에도 영향을 주니, 테스트 후 원복하거나 사전에 공유할 것.
