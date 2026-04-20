# TG Sticker Downloader

一个纯前端的 Telegram 贴纸包在线预览与批量下载工具。无需后端，打开即用。


## 功能

- **在线预览** — 支持静态 WebP、动画 TGS（Lottie）、视频 WebM 三种贴纸格式的实时预览
- **格式转换** — WebP/TGS/WebM → PNG / GIF，也可保留原始格式直接下载
- **批量下载** — 一键打包为 ZIP，支持全选、多选、单张下载
- **链接校验** — 本地结构校验 + 远程 API 交叉验证，自动补全标准链接
- **纯前端** — 所有逻辑在浏览器内完成，数据不经服务器存储
- **自适应** — 桌面端与移动端均可正常使用

## 快速开始

直接用浏览器打开 `index.html` 即可使用。

默认已内置 CORS 代理地址和 Bot Token，开箱即用。如果默认代理不可用，可以在页面的「高级配置」中替换为自己的代理。

## 使用方法

1. 在 Telegram 中打开任意贴纸 → 进入贴纸包详情 → 复制分享链接
2. 将链接粘贴到输入框中，点击「获取」
   - 支持的格式：`https://t.me/addstickers/包名`、`https://t.me/addemoji/包名`、`tg://resolve?domain=addstickers&set=包名`，或直接输入包名
3. 预览贴纸，选择需要的格式（PNG / GIF / 原始格式）
4. 点击「全部下载」或勾选后点击「下载选中」

## 自建 CORS 代理（可选）

如果默认代理失效，可以通过 Cloudflare Workers 免费部署自己的代理：

1. 登录 [Cloudflare](https://dash.cloudflare.com/) → Workers & Pages → 创建 Worker
2. 粘贴以下代码并部署：

```js
export default {
  async fetch(req) {
    const cors = {
      "Access-Control-Allow-Origin": "*",
      "Access-Control-Allow-Methods": "GET,POST,OPTIONS",
      "Access-Control-Allow-Headers": "*"
    };
    if (req.method === "OPTIONS") {
      return new Response(null, { headers: cors });
    }
    const url = new URL(req.url);
    if (url.pathname === "/health") {
      return Response.json({ status: "ok" }, { headers: cors });
    }
    const tg = "https://api.telegram.org" + url.pathname + url.search;
    const resp = await fetch(tg, {
      method: req.method,
      headers: req.headers,
      body: ["GET", "HEAD"].includes(req.method) ? undefined : req.body
    });
    const headers = new Headers(resp.headers);
    Object.entries(cors).forEach(([k, v]) => headers.set(k, v));
    return new Response(resp.body, { status: resp.status, headers });
  }
};
```

3. 将得到的 `https://你的名字.workers.dev` 填入页面的「高级配置 → CORS 代理 URL」
4. 点击「测试连接」验证

## 依赖

通过 CDN 加载，无需本地安装：

- [JSZip](https://stuk.github.io/jszip/) — ZIP 打包
- [FileSaver.js](https://github.com/nicolo-ribaudo/FileSaver.js) — 浏览器端文件保存
- [gif.js](https://jnordberg.github.io/gif.js/) — GIF 编码
- [lottie-web](https://github.com/airbnb/lottie-web) — TGS (Lottie JSON) 渲染

## License

MIT
