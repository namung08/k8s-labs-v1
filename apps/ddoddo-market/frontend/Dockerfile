# 1단계: 빌드
FROM node:20 AS builder

WORKDIR /app

ARG NEXT_PUBLIC_SUPABASE_URL
ENV NEXT_PUBLIC_SUPABASE_URL=$NEXT_PUBLIC_SUPABASE_URL
ARG NEXT_PUBLIC_SUPABASE_ANON_KEY
ENV NEXT_PUBLIC_SUPABASE_ANON_KEY=$NEXT_PUBLIC_SUPABASE_ANON_KEY


COPY package*.json ./
RUN npm install
COPY . .
RUN npm run build

# 2단계: 실행
FROM node:20-alpine

WORKDIR /app
COPY --from=builder /app ./
EXPOSE 3000

ENV NODE_ENV=production

# 여기에서도 NEXT_PUBLIC_ 관련 ENV 라인을 모두 제거합니다.
# 이 변수들은 Kubernetes Deployment에서 런타임에 주입될 것입니다.

CMD ["npm", "start"]
