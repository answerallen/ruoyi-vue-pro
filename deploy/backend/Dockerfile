# ============================================================
# ruoyi-vue-pro (master-jdk25) 后端 Docker 镜像
# monorepo 构建上下文 = 仓库根；后端源码在仓库根(pom 在根)
# CI: docker build -f deploy/backend/Dockerfile   (context=仓库根)
# 多阶段: maven reactor install -> JRE 运行
#
# 关键点：
#   - 脚本依赖: 启用模块 = yudao-dependencies/framework/server/system/infra
#   - 用 -pl yudao-server -am 一次 reactor install，自动带起依赖模块(install 到 .m2)
# ============================================================
FROM maven:3.9-eclipse-temurin-25 AS builder
WORKDIR /build
# 复制启用模块的全部源码(reactor 需要相互 resolve)
COPY pom.xml ./
# 必须含 lombok.config（lombok.accessors.chain=true 依赖它）否则链式 setter 编译失败
COPY lombok.config ./
COPY yudao-dependencies ./yudao-dependencies
COPY yudao-framework ./yudao-framework
COPY yudao-module-system ./yudao-module-system
COPY yudao-module-infra ./yudao-module-infra
COPY yudao-module-bpm ./yudao-module-bpm
COPY yudao-module-ai ./yudao-module-ai
COPY yudao-server ./yudao-server
# 显式指定要 install 的模块(含 bpm/ai), -am 带起其依赖(framework/system/infra/dependencies)
RUN mvn -q clean install \
    -DskipTests -Dmaven.test.skip=true \
    -Dspotless.check.skip=true -Dmaven.javadoc.skip=true \
    -pl yudao-module-bpm,yudao-module-ai,yudao-server -am && \
    echo "=== built jar ===" && ls -la yudao-server/target/*.jar && \
    # 固定 jar 名：真实产物名是 yudao-server.jar(finalName 无版本号, 不是 -*.jar)
    cp yudao-server/target/yudao-server.jar /yudao-server.jar

FROM eclipse-temurin:25-jre
WORKDIR /app
COPY --from=builder /yudao-server.jar /app/yudao-server.jar
ENV JAVA_OPTS="-Xms256m -Xmx768m -XX:+UseSerialGC -Dfile.encoding=UTF-8"
ENV SPRING_PROFILES_ACTIVE=dev
EXPOSE 48080
ENTRYPOINT ["sh","-c","java $JAVA_OPTS -jar /app/yudao-server.jar --spring.profiles.active=$SPRING_PROFILES_ACTIVE"]