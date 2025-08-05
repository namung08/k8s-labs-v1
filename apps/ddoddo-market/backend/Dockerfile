FROM gradle:8.3-jdk17 AS builder

WORKDIR /app

# 종속성 캐싱 단계: Gradle 설정 파일만 복사 후 빌드 실행
COPY --chown=gradle:gradle build.gradle settings.gradle /app/
RUN gradle build --no-daemon || true

# 소스 파일을 컨테이너에 복사
COPY --chown=gradle:gradle . /app

RUN gradle clean bootjar

FROM openjdk:17-jdk-slim

COPY --from=builder /app/build/libs/*.jar app.jar

EXPOSE 8080

ENTRYPOINT ["java", "-jar", "/app.jar"]
