# 培养皿：弹幕余生 PETRI: BULLET AFTERLIFE（网页重制版）

竖屏 Survivors-like 网页小游戏。HTML5 Canvas + 纯 JavaScript，无后端，手机浏览器即开即玩。目标发布：itch.io。

这是 Godot 桌面版《培养皿：弹幕余生》（[hhhh124hhhh/bullet-heaven](https://github.com/hhhh124hhhh/bullet-heaven)）的重制版：保留世界观（废弃实验室培养皿 / PETRI-07 / 霓虹像素风）与机制 Boss 理念，玩法改为竖屏轻量形态（Survivor.io-like：单手摇杆、技能拾取与合成、7–9 分钟一局）。

完整设计文档见 [docs/DESIGN.md](docs/DESIGN.md)。

## 开发顺序（按设计稿 Prototype 0–4）

- **Prototype 0**：移动手感 —— PETRI-07 + 虚拟摇杆 + 20 个追踪敌人，手机单手移动无摩擦
- **Prototype 1**：战斗爽感 —— 基础技能 + 命中特效 + 死亡反馈
- **Prototype 2**：决策验证 —— SAMPLE 资源 + 三选一 + 技能分支（最重要的 Gate：玩家是否愿意换着花样玩）
- **Prototype 3**：机制 Boss PETRI-06
- **Prototype 4**：完整 8 分钟局 + 美术包装 + 音乐，itch.io 发布

## 本地运行

用任意静态服务器打开项目根目录，手机浏览器访问即可（竖屏 360×640 逻辑分辨率）：

```bash
python3 -m http.server 8000
```

## 管线

需求 → GPT 出设计 → 免费代码模型（OpenCode）写代码 → Lin 逐行审查 + 浏览器实点验证 → GitHub → itch.io。UI 类改动必须截图验收。
