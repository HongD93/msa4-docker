# msa4-docker

Dockerfile로 Nginx 이미지를 만들고, Docker Compose로 MySQL과 Java 애플리케이션을 연결하는 학습 저장소입니다. Nginx 이미지 예제와 Compose 예제는 서로 독립적으로 사용할 수 있습니다.

## 예제 구성

| 예제 | 구성과 관계 |
| --- | --- |
| Nginx 이미지 | Dockerfile이 Nginx와 편집 도구를 설치하고 `nginx.conf`를 적용합니다. 컨테이너의 `/usr/share/nginx/html`에서 정적 파일을 제공합니다. |
| Java·MySQL | `web1`이 JAR 애플리케이션을 실행하고 `db1`이 MySQL을 제공합니다. 두 서비스는 같은 Compose 네트워크를 사용합니다. |

Compose의 `db1`은 MySQL 8.4, `web1`은 Eclipse Temurin 17 JRE를 사용합니다. 애플리케이션은 포함된 `app.jar`로 실행하며, 소스와 JAR 빌드 설정은 이 저장소에 없습니다.

## 파일 안내

| 파일 | 알 수 있는 내용 |
| --- | --- |
| [dockerfile/Dockerfile](dockerfile/Dockerfile) | Nginx 이미지의 기반과 패키지 설치 과정 |
| [dockerfile/nginx.conf](dockerfile/nginx.conf) | 정적 파일 제공과 SPA 경로 처리 설정 |
| [compose/docker-compose.yaml](compose/docker-compose.yaml) | DB·애플리케이션 연결 설정, 포트, 볼륨과 네트워크 |
| [compose/app.jar](compose/app.jar) | Compose가 마운트해 실행하는 애플리케이션 |

## Compose로 시작하기

Docker Engine과 Docker Compose 플러그인을 준비한 뒤 저장소 루트에서 작업합니다.

### 1. 실행 설정 확인

[Compose 설정](compose/docker-compose.yaml)은 값을 직접 지정합니다. DB 접속 설정, `JWT_SECRET`, `SERVER_URI`, `CORS_ALLOWEDORIGINS`를 자신의 실습 환경에 맞게 변경합니다. `.env`의 값으로 이 항목들을 대체하는 참조식은 정의되어 있지 않습니다.

DB 비밀번호는 `db1`의 `MYSQL_ROOT_PASSWORD`와 `web1`의 `DB_PASSWORD` 등 실제 연결 설정 사이에서 일치해야 합니다. 고정된 컨테이너 이름과 아래 호스트 포트가 기존 컨테이너와 충돌하지 않는지도 확인합니다.

| 서비스 | 호스트 포트 | 컨테이너 포트 | 볼륨 역할 |
| --- | --- | --- | --- |
| `db1` | `33306` | `3306` | 명명된 볼륨으로 `/var/lib/mysql` 데이터 유지 |
| `web1` | `38080` | `8080` | `compose/app.jar`를 `/app/app.jar`에 연결 |

### 2. DB부터 실행

다음 명령은 MySQL 컨테이너와 필요한 Compose 네트워크·볼륨을 생성합니다. DB 로그에서 초기화와 연결 준비 상태를 확인한 뒤 애플리케이션을 실행합니다.

```sh
docker compose -f compose/docker-compose.yaml up -d db1
docker compose -f compose/docker-compose.yaml logs db1
```

### 3. 애플리케이션 실행

```sh
docker compose -f compose/docker-compose.yaml up -d web1
docker compose -f compose/docker-compose.yaml ps
docker compose -f compose/docker-compose.yaml logs web1
```

설정에는 DB 준비를 기다리는 `healthcheck`나 `depends_on`이 없습니다. 초기 DB 연결 문제가 생기면 두 서비스의 로그와 접속 설정을 함께 확인합니다.

### 4. 중지

```sh
docker compose -f compose/docker-compose.yaml down
```

이 명령은 컨테이너와 Compose 네트워크를 정리하며 명명된 MySQL 볼륨은 유지합니다.

## Nginx 이미지 실습

저장소 루트에서 빌드합니다. `nginx.conf`가 포함된 `dockerfile` 디렉터리가 빌드 컨텍스트입니다.

```sh
docker build -t msa4-nginx-demo -f dockerfile/Dockerfile dockerfile
docker run --rm -p 8080:80 msa4-nginx-demo
```

실행 후 브라우저에서 `http://localhost:8080`으로 접근합니다. 이 Dockerfile에는 별도의 웹 애플리케이션 빌드 파일을 복사하는 단계가 없습니다. 이미지 빌드가 실패하면 패키지 설치 명령과 Dockerfile 마지막 줄의 줄 연결 문자도 확인합니다.
