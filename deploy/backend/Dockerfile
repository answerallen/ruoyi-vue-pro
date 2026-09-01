# ============================================================
# ruoyi-vue-pro (master-jdk25) 后端 Docker 镜像
# monorepo 构建上下文 = 仓库根(/) ；后端源码在 ./ 根(ruoyi-pom 在仓库根)
# CI: docker build -f deploy/backend/Dockerfile   (context=仓库根)
# 多阶段: maven 构建 fat jar -> JRE 运行
# ============================================================
FROM maven:3.9-eclipse-temurin-25 AS builder
WORKDIR /build
# 整个仓库作为 context；后端使用根 pom (ruoyi 是单体, pom 在仓库根)
COPY pom.xml ./
# 需要 yudao-dependencies 先 install
COPY yudao-dependencies ./yudao-dependencies
COPY yudao-framework ./yudao-framework
COPY yudao-module-system ./yudao-module-system
COPY yudao-module-infra ./yudao-module-infra
COPY yudao-server ./yudao-server
# 先安装 framework/dependencies 到本地仓库，再构建 server
RUN mvn -q -N install -DskipTests
RUN mvn -q -DskipTests -Dmaven.test.skip=true \
    -Dspotless.check.skip=true -Dmaven.javadoc.skip=true \
    -pl yudao-server -am package && \
    ls -la yudao-server/target/*.jar

FROM eclipse-temurin:25-jre
WORKDIR /app
COPY --from=builder /build/yudao-server/target/yudao-server-*.jar /app/yudao-server.jar
ENV JAVA_OPTS="-Xms256m -Xmx768m -XX:+UseSerialGC -Dfile.encoding=UTF-8"
ENV SPRING_PROFILES_ACTIVE=dev
EXPOSE 48080
ENTRYPOINT ["sh","-c","java $JAVA_OPTS -jar /app/yudao-server.jar --spring.profiles.active=$SPRING_PROFILES_ACTIVE"]