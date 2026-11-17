# ============================================================
# AgentOne 后端 - 多阶段构建
# Stage 1: Maven 构建（pom 先行，利用依赖缓存层加速重建）
# Stage 2: JRE 运行
# ============================================================

FROM maven:3.9-eclipse-temurin-17 AS build
WORKDIR /build

# 使用阿里云镜像加速依赖下载（国内直连 Maven Central 极慢）
COPY maven-settings.xml /root/.m2/settings.xml

# 先拷贝所有 pom，单独拉依赖（源码变更不触发重新下载依赖）
COPY pom.xml .
COPY agentone-common/pom.xml    agentone-common/
COPY agentone-auth/pom.xml      agentone-auth/
COPY agentone-workspace/pom.xml agentone-workspace/
COPY agentone-agent/pom.xml     agentone-agent/
COPY agentone-knowledge/pom.xml agentone-knowledge/
COPY agentone-skill/pom.xml     agentone-skill/
COPY agentone-apikey/pom.xml    agentone-apikey/
COPY agentone-api/pom.xml       agentone-api/
RUN mvn -B -q dependency:go-offline || true

# 拷贝源码并打包（只构建可运行模块 agentone-api 及其依赖模块）
COPY . .
RUN mvn -B -pl agentone-api -am package -DskipTests

# ------------------------------------------------------------
FROM eclipse-temurin:17-jre
# 容器健康检查（/v1/health）依赖 curl：基础镜像已内置 curl（无需 apt 安装，
# 也避免构建时依赖 Ubuntu 软件源可用性）。

WORKDIR /app
COPY --from=build /build/agentone-api/target/agentone-api-*.jar app.jar

EXPOSE 8080
ENTRYPOINT ["java", "-XX:MaxRAMPercentage=75.0", "-jar", "app.jar"]
