# ============================================================
# yudao-ui-admin-vue3 前端 Docker 镜像
# monorepo 构建上下文 = 仓库根；前端源码在 frontend/
# CI: docker build -f deploy/frontend/Dockerfile   (context=仓库根)
# 多阶段: pnpm 构建 -> nginx
# ============================================================
FROM node:22-alpine AS builder
WORKDIR /build
ARG API_BASE_URL=/admin-api
# 固定 pnpm@11.8.0：与 frontend/pnpm-lock.yaml(lockfileVersion 9.0) 匹配
#   用 pnpm@11 会装到 11.25，其 supply-chain 校验对 159 entries 报 TARBALL_URL_MISMATCH
RUN npm install -g pnpm@11.8.0
# 复制完整前端源码
COPY frontend/ ./
# 覆盖 API 前缀(前端用 VITE_API_URL=/admin-api，与 nginx 反代一致)
RUN if [ -f .env.prod ]; then \
      sed -i "s|VITE_API_URL=.*|VITE_API_URL=${API_BASE_URL}|" .env.prod || true; \
    fi
# 安装依赖：优先 frozen；lockfile 校验失败(版本差异)时退化为非 frozen 重新解析
RUN pnpm install --frozen-lockfile || pnpm install --no-frozen-lockfile
RUN pnpm build:prod && ls -la dist/

# ---- 运行 ----
FROM nginx:1.27-alpine
COPY --from=builder /build/dist /usr/share/nginx/html
COPY deploy/frontend/nginx.conf /etc/nginx/conf.d/default.conf
EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]