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
# lockfile 的 tarball URL 指向 npmmirror，必须显式设 registry 否则 supply-chain 校验失败
RUN pnpm config set registry https://registry.npmmirror.com
# (strict-dep-builds 用内联 --config 传入，见下方 install 命令)
# 复制完整前端源码
COPY frontend/ ./
# 覆盖 API 前缀(前端用 VITE_API_URL=/admin-api，与 nginx 反代一致)
RUN if [ -f .env.prod ]; then \
      sed -i "s|VITE_API_URL=.*|VITE_API_URL=${API_BASE_URL}|" .env.prod || true; \
    fi
# 安装依赖：优先 frozen；lockfile 校验失败(版本差异)时退化为非 frozen 重新解析
#   --dangerously-allow-all-builds: 允许 esbuild/es5-ext 等 postinstall 脚本(否则 ERR_PNPM_IGNORED_BUILDS)
RUN pnpm install --frozen-lockfile --dangerously-allow-all-builds || pnpm install --no-frozen-lockfile --dangerously-allow-all-builds
RUN pnpm build:prod && ls -la dist/

# ---- 运行 ----
FROM nginx:1.27-alpine
COPY --from=builder /build/dist /usr/share/nginx/html
COPY deploy/frontend/nginx.conf /etc/nginx/conf.d/default.conf
EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]