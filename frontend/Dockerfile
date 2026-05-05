# ===============
# Build Stage
# Vue 프로젝트를 빌드해서 dist 폴더를 만드는 단계
# ===============
FROM node:22-alpine AS build

# 컨테이너 내부 작업 디렉터리
WORKDIR /app

# package 파일을 먼저 복사해서 npm install 캐시를 활용
COPY package*.json ./

# package-lock.json 기준으로 정확한 의존성 설치
RUN npm ci

# 나머지 프론트 소스 복사
COPY . .

# Vite 빌드 실행 -> dist 폴더 생성
RUN npm run build

# ===============
# Run Stage
# 빌드된 정적 파일을 Nginx로 서빙하는 단계
# ===============
FROM nginx:alpine

#  기본 Nginx 정적 파일 위치에 Vue 빌드 결과 복사
COPY --from=build /app/dist /usr/share/nginx/html

# Nginx 기본 포트
EXPOSE 80

# Nginx를 foreground로 실행
CMD [ "nginx", "-g", "daemon off;" ]