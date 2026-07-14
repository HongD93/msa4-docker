# 베이스 이미지 설정
FROM nginx

COPY nginx.conf /etc/nginx/conf.d/default.conf

# 이미지를 생성하는 과정에서 실행시킬 명령어를 작성
# \ : 개행
# && : 명령어를 같이 실행시키고 싶을 때
# -y : 설치과정에서 일어나는 명령을 승인할 때
RUN apt update \
    && apt install -y vim \
    && apt install -y nano \
    && apt purge -y --auto-remove -o APT::AutoRemove::RecommendsImportant=false \
    && rm -rf /var/lib/apt/lists/* \
