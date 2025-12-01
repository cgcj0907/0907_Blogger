---
title: "QR Scanner 使用指南"
date: 2024-08-16
draft: false               
tags: ["前端", "二维码", "Toolkits"]
ShowToc: true             
---

# 简介

`qr-scanner` 是一款纯前端、零依赖的 JavaScript 二维码扫描库，完全在浏览器端运行，无需后端配合。它同时支持：

- 摄像头实时扫码
- 静态图片/文件离线扫码（支持 `<img>`、`<canvas>`、`<video>`、File、Blob 等多种来源）

底层使用 WebAssembly 加速解码，识别速度快、准确率高，在手机端表现尤为优秀，是目前最推荐的前端二维码扫描方案

本文以 `React + Next.js` 框架为例

---

## 1. 安装

```bash
npm install qr-scanner
```

或使用 yarn：

```bash
yarn add qr-scanner
```

> 注意：已不再推荐直接用 `<script>` 标签引入，现代项目请通过包管理器安装

---

## 2. 基本引入方式

```ts
import QrScanner from 'qr-scanner';
```

---

## 3. 核心类：QrScanner

### 3.1 摄像头实时扫码

```ts
const videoElement = document.getElementById('video') as HTMLVideoElement;

const scanner = new QrScanner(
  videoElement,
  (result) => {
    console.log('扫码成功:', result.data);
    // result.cornerPoints 可用来做边框动画
  },
  {
    returnDetailedScanResult: true, // 返回完整对象而非仅字符串
    highlightScanRegion: true,      // 显示扫描区域遮罩
    highlightCodeOutline: true,     // 高亮二维码轮廓
  }
);

// 启动摄像头
await scanner.start();

// 停止扫描
scanner.stop();
```

#### 常用配置项

| 参数                         | 说明                                      |
|----------------------------|-----------------------------------------|
| returnDetailedScanResult   | true 时返回 { data, cornerPoints } 对象   |
| highlightScanRegion        | 显示半透明扫描区域框                          |
| highlightCodeOutline       | 实时绘制二维码四角轮廓                        |

---

### 3.2 离线扫描图片文件

```ts
const file: File = 文件输入框.files[0];
const result = await QrScanner.scanImage(file, {
  returnDetailedScanResult: true
});

console.log('二维码内容:', result.data);
```

支持所有图片源：File、Blob、HTMLImageElement、HTMLCanvasElement、URL 等

---

## 4. 常用场景示例

### 4.1 摄像头扫码自动填充地址

```ts
const scanner = new QrScanner(videoRef.current!, (result) => {
  const match = result.data.match(/0x[a-fA-F0-9]{40}/i);
  if (match) {
    setAddress(match[0] as `0x${string}`);
    scanner.stop();           // 扫到即停止
    onClose();                // 关闭弹窗
  }
});

await scanner.start();
```

### 4.2 上传图片扫码

```ts
const handleUpload = async (file: File) => {
  try {
    const result = await QrScanner.scanImage(file);
    setAddress(result.data);
    message.success('扫码成功');
  } catch (e) {
    message.error('未识别到二维码');
  }
};
```

### 4.3 正确销毁实例（很重要！）

```ts
// React useEffect 清理示例
useEffect(() => {
  return () => {
    scanner?.stop();
    scanner?.destroy();
    scanner = null;
  };
}, []);
```

不销毁会导致摄像头一直被占用、内存泄漏

---

## 5. React 项目注意事项

1. video 元素必须通过 useRef 绑定，不能直接用 document.getElementById
2. 建议在弹窗/抽屉打开时才 start()，关闭时立即 stop + destroy
3. start() 是异步的，建议加 await 并 try/catch 处理权限问题
4. 识别以太坊地址推荐正则：`/0x[a-fA-F0-9]{40}/i`
5. 提供「上传图片扫码」作为摄像头失败的兜底方案

---

## 6. 常见问题与解决方案

| 问题                  | 解决方案                                                    |
|---------------------|-----------------------------------------------------------|
| 摄像头黑屏/无法开启      | 必须 HTTPS 或 localhost；检查是否已授权摄像头                           |
| start() 报错          | 确保 videoRef.current 已有值，且上一个 scanner 已 destroy                |
| 扫不到码              | 光线要充足、二维码要完整；可提示用户「支持上传图片扫码」                      |
| 切换页面后摄像头仍占用   | 组件卸载必须执行 stop() + destroy()                                 |
| 手机端识别慢           | qr-scanner 已使用 WASM，属于目前最快方案；仍建议保持二维码占屏 1/3 以上大小 |

---

## 7. 总结

- 实时摄像头扫码 + 离线图片扫码一把抓
- 识别速度快、准确率高、手机端表现优秀
- 提供扫描区域高亮、二维码轮廓高亮、角点数据等高级功能
- React 使用时配合 useRef + useEffect 生命周期管理即可
- 强烈建议搭配「上传图片扫码」作为备用方案，用户体验更佳


