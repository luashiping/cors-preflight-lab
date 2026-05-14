# CORS Preflight Lab

这个实验环境用来复现一个很容易误判的现象：

- 浏览器调试信息里先看到 CORS 报错
- 但真正的根因可能是 `OPTIONS` 预检请求返回了 `400`、`403`、`502` 等非 2xx 状态码
- 也可能是预检虽然返回了 `204`，但缺少关键的 CORS 允许头，导致协商失败
- 即使某些 `403` / `502` 响应里带上了 `Access-Control-Allow-*` 头，浏览器依然不会放行后续真实请求

## 目录说明

- `frontend/`：静态测试页，运行在 `http://localhost:8080`
- `api/`：Nginx API，运行在 `http://localhost:8081`

## 启动

```bash
cd "cors-preflight-lab"
docker compose up -d
```

打开 `http://localhost:8080` 后：

- 点击“触发 400 预检失败”：
  - 前端会发起一个携带 `Authorization` 头的跨域 `GET`
  - 浏览器会先发送 `OPTIONS /bad-request`
  - API 对该预检返回 `400`
  - 浏览器不会继续发送真正的 `GET /bad-request`
  - 前端最终表现为 CORS 错误

- 点击“触发 403 有 CORS 头”：
  - 前端会发起一个携带 `Authorization` 头的跨域 `GET`
  - 浏览器会先发送 `OPTIONS /forbidden-with-cors`
  - API 对该预检返回 `403`
  - 响应里仍然带有 `Access-Control-Allow-*` 头
  - 浏览器不会继续发送真正的 `GET /forbidden-with-cors`
  - 前端最终表现为 CORS 错误

- 点击“触发 403 无 CORS 头”：
  - 前端会发起一个携带 `Authorization` 头的跨域 `GET`
  - 浏览器会先发送 `OPTIONS /forbidden-no-cors`
  - API 对该预检返回 `403`
  - 响应里不返回 `Access-Control-Allow-*` 头
  - 浏览器不会继续发送真正的 `GET /forbidden-no-cors`
  - 前端最终表现为 CORS 错误

- 点击“触发 502 有 CORS 头”：
  - 前端会发起一个携带 `Authorization` 头的跨域 `GET`
  - 浏览器会先发送 `OPTIONS /bad-gateway-with-cors`
  - API 对该预检返回 `502`
  - 响应里仍然带有 `Access-Control-Allow-*` 头
  - 浏览器不会继续发送真正的 `GET /bad-gateway-with-cors`
  - 前端最终表现为 CORS 错误

- 点击“触发 502 无 CORS 头”：
  - 前端会发起一个携带 `Authorization` 头的跨域 `GET`
  - 浏览器会先发送 `OPTIONS /bad-gateway-no-cors`
  - API 对该预检返回 `502`
  - 响应里不返回 `Access-Control-Allow-*` 头
  - 浏览器不会继续发送真正的 `GET /bad-gateway-no-cors`
  - 前端最终表现为 CORS 错误

- 点击“触发协商失败”：
  - 前端会发起一个携带 `Authorization` 头的跨域 `GET`
  - 浏览器会先发送 `OPTIONS /missing-allow-headers`
  - API 返回 `204`
  - 但响应里故意不返回 `Access-Control-Allow-Headers: Authorization`
  - 浏览器依然不会继续发送真正的 `GET /missing-allow-headers`
  - 前端最终表现为 CORS 错误

- 点击“触发成功请求”：
  - 浏览器会先发送 `OPTIONS /ok`
  - API 返回 `204` 且带上允许的 CORS 响应头
  - 浏览器继续发送真正的 `GET /ok`
  - 页面看到成功响应

## 观察命令

```bash
docker compose logs -f api
```

你会看到：

- 失败场景里只有 `OPTIONS /bad-request`
- 失败场景里只有 `OPTIONS /forbidden-with-cors`
- 失败场景里只有 `OPTIONS /forbidden-no-cors`
- 失败场景里只有 `OPTIONS /bad-gateway-with-cors`
- 失败场景里只有 `OPTIONS /bad-gateway-no-cors`
- 协商失败场景里只有 `OPTIONS /missing-allow-headers`
- 成功场景里先有 `OPTIONS /ok`，再有 `GET /ok`

## 停止

```bash
docker compose down
```
