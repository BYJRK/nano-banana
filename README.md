# Nanobanana - 基于 OpenRouter 的图片生成 Web UI

本代码仓库 Fork 自[这个项目](https://github.com/hkfires/nano-banana/)。详细的 README 内容请查看原始项目。

## 核心变化

为提升使用体验，本项目在原有基础上做了以下改进（按照功能实现的顺序排列）：

1. 当图片生成完毕（成功或失败）时，会弹出提示，并显示耗时
2. 如果 API Key 已经配置过，则会自动折叠
3. 添加取消生成的功能
4. 支持最新的 Nano Banana 2 模型（512 分辨率因为 OpenRouter 的 API 问题暂时屏蔽）
5. 支持选择保存图片的格式（PNG / JPEG / WebP）
6. 支持深色主题
7. 图文生图的图片上传支持拖拽调整图片的顺序
8. 支持点击全屏预览生成的图片

## 本地部署

请参考原始项目的 README 文件中的部署说明。推荐使用 [bun](https://bun.sh/) 来安装依赖和运行项目。

```bash
# 安装依赖
bun install

# 运行项目
bun run dev
```

然后就可以在 <http://localhost:3000/> 上访问 Web UI 了。

如果安装了 [TaskFile](https://taskfile.dev/)，也可以使用以下命令：

```bash
task init
task run
```
