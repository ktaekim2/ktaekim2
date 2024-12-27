# 스프링 시큐리티
스프링 시큐리티 + JWT를 사용한 회원 기능 개발

## 개요
기본적으로 JWT는 엑세스 토큰과 리프레시 토큰으로 기존 세션 기능을 대체한다.  
클라이언트는 로그인 시, 리프레시 토큰을 발급받아 db에 저장한다. 리프레시 토큰은 보통 유효기간 7일로 설정한다.
리프레시 토큰과 함께 엑세스 토큰을 발급하여 클라이언트가 가지고 있으며, 통신 시 엑세스 토큰을 통해 회원 유효성을 검사한다.
다중로그인 관리를 위해 각 클라이언트별(PC별) 개별 리프레시 토큰 발급을 위해 다중 리프레시 토큰을 관리할 필요가 있다.
만약, 리프레시 토큰이 멤버와 1:1관계를 맺는다면, PC-1에서 로그인 한 유저가, PC-2에서 동일 계정 로그인 할 경우, 기존 리프레시 토큰과 엑세스 토큰을 재발급 받을 것이고, PC-1이 가지고 있는 엑세스 토큰은 더이상 유효하지 않을 것이다.

그렇다고 리프레시 토큰을 초기화, 삭제하지 않고 한 번 발급받은 리프레시 토큰을 유효기간이 지날때 까지 계속 사용한다면 다음 문제가 발생한다.
- 다중 로그인 불가
- 보안 취약점
- 세션 관리 문제

## 엑세스 토큰
엑세스 토큰은 기본 stateless 토큰이며 db에 저장하지 않는것이 표준이다. 서버는 엑세스 토큰에 포함된 정보만으로 인증과 권한을 확인한다.  
보통 15분 정도의 짧은 만료 시간을 갖고, 재사용성이 적다.

## 세션방식과 비교하면...
### 세션
- 보안적으로 안전한편.
- 세션 정보를 서버 메모리에 저장되므로 서비스가 커질수록 메모리 부담이 증가함.

### 토큰
- 토큰의 경우 stateless(상태 비저장), 모든 인증 정보는 토큰에 포함.
- 서버에서 상태를 유지하지 않으므로, 분산 서버에 유리.(확장성에 유리)
- 토큰은 서명되어있어 변조 방지.



## 로그인 유저 정보를 앞단에 어떻게 넘길 것인가

### 헤더에 담기
가장 간단한 방법이지만, 보안에 취약함. 넘겨야 하는 정보가 많아질 경우 확정성이 부족하고 네트워크 성능에 악영향. 그리고 유지보수성이 떨어짐.

## 테이블 설계

### nt_member
멤버 테이블

```sql
CREATE TABLE nt_member (
    member_seq BIGINT PRIMARY KEY,
    username VARCHAR(255) UNIQUE NOT NULL,
    password VARCHAR(255) NOT NULL,
    nickname VARCHAR(255),
    phone VARCHAR(255),
    address VARCHAR(255),
    profile_img VARCHAR(255),
    cretr_id             varchar(20),
    cret_dt              timestamp(6)                                          NOT NULL,
    amdr_id              varchar(20),
    amd_dt               timestamp(6),
    del_yn               varchar(1) DEFAULT 'N'::character varying             NOT NULL,
)
```

### nt_roles
역할 테이블, 개발자가 관리

```sql
CREATE TABLE nt_roles (
    role_seq SERIAL PRIMARY KEY,
    role_name VARCHAR(255) UNIQUE NOT NULL, -- ROLE_USER, ROLE_ADMIN 등
    cretr_id             varchar(20),
    cret_dt              timestamp(6)                                          NOT NULL,
    amdr_id              varchar(20),
    amd_dt               timestamp(6),
    del_yn               varchar(1) DEFAULT 'N'::character varying             NOT NULL,
)

INSERT INTO nt_roles (role_name, cretr_id, cret_dt, del_yn) VALUES
('ROLE_USER', 'system', NOW(), 'N'),
('ROLE_ADMIN', 'system', NOW(), 'N');

```

### nt_member_roles
멤버, 역할 매핑 테이블
member는 여러개의 role을 가질 수 있음.

```sql
CREATE TABLE nt_member_roles (
    member_seq BIGINT NOT NULL REFERENCES nt_member(member_seq)
    role_seq INT NOT NULL REFERENCES nt_roles(role_seq),
    cretr_id             varchar(20),
    cret_dt              timestamp(6)                                          NOT NULL,
    amdr_id              varchar(20),
    amd_dt               timestamp(6),
    del_yn               varchar(1) DEFAULT 'N'::character varying             NOT NULL
    PRIMARY KEY (member_seq, role_seq)
)
```

### nt_permissions
권한 테이블, 개발자가 관리

```sql
CREATE TABLE nt_permissions (
    permission_seq SERIAL PRIMARY KEY,
    permission_name VARCHAR(255) UNIQUE NOT NULL, -- CREATE_POST, DELETE_POST 등
    cretr_id             varchar(20),
    cret_dt              timestamp(6)                                          NOT NULL,
    amdr_id              varchar(20),
    amd_dt               timestamp(6),
    del_yn               varchar(1) DEFAULT 'N'::character varying             NOT NULL,
)

INSERT INTO nt_permissions (permission_name, cretr_id, cret_dt, del_yn) VALUES
('VIEW', 'system', NOW(), 'N'),
('CREATE', 'system', NOW(), 'N'),
('DELETE', 'system', NOW(), 'N'),
('MANAGE', 'system', NOW(), 'N');

```

### nt_role_permissions
역할, 권한 매핑 테이블
role은 여러개의 permission을 가질 수 있음.

```sql
CREATE TABLE nt_role_permissions (
    role_seq INT NOT NULL REFERENCES nt_roles(role_seq),
    permission_seq INT NOT NULL REFERENCES nt_permissions(permission_seq),
        cretr_id             varchar(20),
    cret_dt              timestamp(6)                                          NOT NULL,
    amdr_id              varchar(20),
    amd_dt               timestamp(6),
    del_yn               varchar(1) DEFAULT 'N'::character varying             NOT NULL
    PRIMARY KEY (role_seq, permission_seq)
)

-- ROLE_ADMIN: 모든 권한
INSERT INTO nt_role_permissions (role_seq, permission_seq, cretr_id, cret_dt, del_yn) VALUES
(2, 1, 'system', NOW(), 'N'), -- VIEW
(2, 2, 'system', NOW(), 'N'), -- CREATE
(2, 3, 'system', NOW(), 'N'), -- DELETE
(2, 4, 'system', NOW(), 'N'); -- MANAGE

-- ROLE_USER: VIEW 권한만
INSERT INTO nt_role_permissions (role_seq, permission_seq, cretr_id, cret_dt, del_yn) VALUES
(1, 1, 'system', NOW(), 'N'); -- VIEW

```

### nt_refresh_tokens
리프레시 토큰 관리 테이블.  
기기별로 리프레시 토큰을 고유하게 유지하면, 특정 기기의 세션이 탈취되거나 종료되더라도 다른 기기의 세션에 영향을 미치지 않음.
하나의 기기에서 로그아웃 하더라도, 모든 기기에서 로그아웃되지 않음.(중복 로그인 가능)

```sql
CREATE TABLE nt_refresh_tokens (
    token_seq SERIAL PRIMARY KEY,
    member_seq BIGINT NOT NULL REFERENCES nt_member(member_seq),
    refresh_token VARCHAR(255) NOT NULL,
    device_info VARCHAR(255), -- 기기 정보 (옵션)
    issued_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
    expires_at TIMESTAMP NOT NULL,
    del_yn               varchar(1) DEFAULT 'N'::character varying             NOT NULL,
)

CREATE INDEX idx_refresh_tokens_expires_at ON nt_refresh_tokens(expires_at);
CREATE INDEX idx_refresh_tokens_del_yn ON nt_refresh_tokens(del_yn);
CREATE INDEX idx_refresh_tokens_member_seq_device_info
    ON nt_refresh_tokens (member_seq, device_info);

```

## 만료된 토큰 자동 삭제

```java
@Scheduled(cron = "0 0 * * * ?") // 매 시간 실행
public void cleanExpiredRefreshTokens() {
    refreshTokenRepository.deleteByExpiresAtBefore(LocalDateTime.now());
}
```

## 로그인
로그인 과정에서 SecurityContextHolder에 로그인 정보를 저장한다.  

```java
Authentication authentication = authenticationManager.authenticate(
        new UsernamePasswordAuthenticationToken(loginRequest.getUsername(), loginRequest.getPassword()));
```

AuthenticationManager의 authenticate()를 통해 UserDetailsService 구현체의 loadUserByUsername()를 자동호출하여, authentication 객체를 반환한다.  
유저 정보를 가지고 있는 authentication 객체를 SecurityContextHolder에 저장하면 로그인이 끝난다.  
이후 토큰과 별개로 SecurityContextHolder객체의 getPrincipal()를 통해 UserDetails 객체를 생성할 수 있다.  























## JWT