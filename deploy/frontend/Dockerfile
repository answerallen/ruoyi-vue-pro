# ============================================================
# yudao-ui-admin-vue3 前端 Docker 镜像
# monorepo 构建上下文 = 仓库根用于 frontend/ 子目录
# CI: docker build -f deploy/frontend/Dockerfile   (context=仓库根)
# 多阶段: pnpm 构建(前端源码在 frontend/) -> nginx
# ============================================================
FROM node:22-alpine AS builder
WORKDIR /build
ARG API_BASE_URL=/admin-api
RUN corepack enable
# monorepo: 前端源码在 frontend/
COPY frontend/package.json frontend/pnpm-lock.yaml ./
RUN pnpm install --frozen-lockfile
# 复制其余源码(frontend/pnpm-lock 已在)
COPY frontend/ ./
# 覆盖 API 前缀
RUN if [ -f .env.production ]; then \
      sed -i "s|VITE_APP_BASE_API=.*|VITE_APP_BASE_API=${API_BASE_URL}|" .env.production || true; \
    fi
RUN pnpm build:prod && ls -la dist/

# ---- 运行 ----
FROM nginx:1.27-alpine
COPY --from=builder /build/dist /usr/share/nginx/html
COPY deploy/frontend/nginx.conf /etc/nginx/conf.d/default.conf
EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]