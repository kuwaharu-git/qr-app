FROM node:24.12.0-slim AS base

WORKDIR /app

# 依存関係のみコピー・インストール
COPY ./app/package.json ./app/package-lock.json ./
RUN npm ci


FROM base AS dev
# 0.0.0.0:5173 で起動
CMD ["npm", "run", "dev", "--", "--host", "0.0.0.0", "--port", "5173"]


FROM base AS build
COPY app/ ./
RUN npm run build


FROM nginx:1.27-alpine AS prod
# ビルド成果物を Nginx の公開ディレクトリへ
COPY --from=build /app/dist /usr/share/nginx/html
# SPA 用のルーティング設定
COPY ./nginx/default.conf /etc/nginx/conf.d/default.conf
EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]

