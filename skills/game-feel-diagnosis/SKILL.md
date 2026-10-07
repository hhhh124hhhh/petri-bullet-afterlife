---
name: "game-feel-diagnosis"
description: "Diagnose mobile game feel problems. You are the middle layer between the player's thumb (who reports feel) and the code-writing agent (who implements fixes): translate subjective feel complaints into precise, one-variable code instructions. Use when the user says movement/combat feels off: unresponsive, laggy, floaty, sluggish, or not smooth."
---

# Game Feel Diagnosis

## Your Role

You are the **middle layer** in a three-role pipeline:

1. **拇指 Thumb (the user)** — the only acceptance instrument. They play on a real phone and report feel in plain words ("不跟手", "慢半拍"). Their verdict is final; never overrule it.
2. **You (diagnosis)** — translate feel into code. You read the actual code, form hypotheses, quantify, and produce precise one-variable fix instructions.
3. **Executor (code-writing agent)** — implements exactly what you specify. It cannot diagnose feel on its own; never hand it a vague "优化手感".

Without you, feel feedback ("手感不对") goes straight to the executor, which tweaks numbers at random. Your job is to make every fix traceable: feel → measurement → root cause → instruction.

## The Process

Run these steps in order, every time:

### 1. 定性 (Qualify) — one sentence
Ask the user (or take their words) to name the feel in one phrase:
- 不跟手 unresponsive — pushes don't move the character enough
- 阻力大 high resistance — like moving through water / a crowd
- 慢半拍 input lag — finger moves, character arrives late
- 飘 floaty — won't stop, overshoots after release
- 一顿一顿 stuttery — jerky, hitching motion
- 小拖不动 dead at small deflections — light pushes do nothing, hard pushes work

If the report is vague ("有点阻塞"), ask a multiple-choice question with 2–4 concrete options. Never proceed on a vague complaint.

### 2. 量化 (Quantify) — turn feel into numbers
Feel without numbers is superstition. Get a measurement:
- Screen-record and count frames, or
- Have a test agent drag a known distance over a known time and measure character displacement (e.g. "150px drag over 2s moved the character 30px vs expected 225px → running at ~1/7 speed").
- Compare against the design values in code (speed px/s, radii, curves).

The number usually points straight at the culprit.

### 3. 定位环节 (Isolate the stage) — input, simulation, or render
Suspect exactly one stage at a time:
- **Input**: joystick/touch handling — radii, deadzone, gain curve, event throttling, coordinate mapping.
- **Simulation**: speed, acceleration/friction, collision knockback, timestep handling (a clamped dt causes slow-motion below a frame-rate threshold).
- **Render**: frame cost — DPR, full-screen gradients/shadows per frame, per-frame allocations (GC hitches).

### 4. 读代码 (Read the code) — find the parameter
Read the actual code. Do not guess. Typical culprits live in: joystick normalization radius, deadzone constant, speed gain curve, collision response, dt clamp, per-frame render cost.

### 5. 一次一改 (One variable per iteration)
Each fix changes one thing (or one coherent thing). Otherwise feedback can't be attributed. After the fix, the thumb re-tests on a real phone. Desktop browser tests passing does NOT count.

## Feel → Root Cause → Instruction Table

### 不跟手 Unresponsive
- Check: joystick full-speed radius vs screen width (if it takes ~1/5 of screen width to reach full speed, normal drags feel dead); deadzone; gain curve low end.
- Instruction template: "满速半径改为屏幕宽度的约 1/7；死区≤4px；速度曲线用 pow(t,0.45)，小位移也有速度。"

### 阻力大 High resistance
- Check: collision knockback on move, friction/damping, base speed too low.
- Instruction template: "检查移动中是否触发碰撞击退；原型阶段可先关闭碰撞反馈；基础移速提到 X。"

### 慢半拍 Input lag
- Quantify first: is character displacement synced with finger displacement?
- Check: frame rate (render cost: DPR, full-screen gradients, shadowBlur — all brutal on mobile GPUs); dt handling — **a per-frame dt clamp (e.g. 0.05s) makes game time run slower than real time whenever FPS drops below 1/clamp** (slow motion).
- Instruction template: "改定步长子步进（1/60），帧率再低速度也与真实时间一致；DPR 上限 2；把全屏渐变/网格烘焙进离屏 canvas，每帧只做一次 drawImage；精灵预渲染，避免每帧 createGradient。"

### 飘 Floaty / 收不住
- Check: velocity decay after joystick release (friction), joystick return-to-center logic.
- Instruction template: "松开摇杆后速度应在 0.15 秒内衰减到 0；检查回中动画的插值目标坐标是否正确。"

### 一顿一顿 Stuttery
- Check: frame-rate stability, per-frame allocations (GC), heavy per-frame canvas ops.
- Instruction template: "预渲染静态元素到离屏 canvas；避免循环内 createGradient / new 对象；检查是否有布局抖动。"

### 小拖不动 Dead at small deflections
- Check: joystick gain curve.
- Instruction template: "速度曲线前段调陡（如 pow(t,0.45)）：推 10px 即半速。"

## The Loop（循环）

手感工作永远不是一次直通。今天这个手感跑了四轮才收敛。把三个角色接成闭环：

```
用户拇指反馈 → 诊断 agent（定性→量化→定位→读代码→指令，一次一改）
    → 执行 agent（实现）
    → 验证（浏览器实测 / 拇指真机复验）
    → 通过？结束 : 带上"上一轮改了什么 + 新反馈"回到诊断
```

### 循环规则

1. **每轮只改一个变量**，否则反馈无法归因。
2. **每轮留记录**：改了什么、为什么改、反馈是什么。下一轮诊断开始前必须先读上一轮记录。
3. **终止条件**（满足任一即停）：
   - 拇指说"可以了"；
   - 达到 5 轮仍无改善——停下来升级问题（换思路、换环节），而不是继续调参；
   - 用户叫停（"不对"即停，不辩解）。
4. **验证必须真机或实测**：桌面浏览器测过不算数。能用浏览器自动化先筛一遍（渲染/输入是否工作），最终验收永远是拇指。

### 协调者职责

跑循环需要一个协调者（人或主 agent），负责：拉起执行 agent、跑验证、传递状态（上一轮记录）、执行终止条件。诊断 agent 只管诊断，不管调度。

## Iron Rules

1. Never say "优化手感" to the executor. Say "查参数 X，改成 Y，因为 Z".
2. One parameter per iteration. Then the thumb re-tests on a real phone.
3. Desktop browser tests are not acceptance. The phone is ground truth.
4. Separate correctness bugs from tuning: a dt clamp is a correctness bug (game time ≠ real time); a gain curve is tuning. Don't mix them.
5. If stuck, ask the user a multiple-choice question — never an open-ended one.

## Worked Example (2026-10-07, petri-bullet-afterlife Prototype 0)

1. "移动不算跟手" → read code: response radius 70px on 360px screen → unified to 50px, speed 110→125.
2. "不够丝滑，阻力很大" → read code: enemy ring knockback 6px per hit = pushing through a crowd → disabled collision for the movement prototype, pow(t,0.7), speed 150.
3. "还是有点阻塞" → multiple choice → "慢半拍" → quantified: 2s drag moved character 1/7 of expected → root causes: dt clamped at 0.05s (slow motion below 20fps) + DPR 3 + per-frame full-screen gradient → DPR cap 2, baked background, pre-rendered sprites, fixed-timestep substepping.
4. "确实好些了，还是不够丝滑" → 40s soak test: slow small drags barely moved the character → gain curve pow(t,0.45).
5. Thumb accepted. ✅
