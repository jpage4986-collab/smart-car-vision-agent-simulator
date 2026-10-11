# Smart Car Vision Agent Simulator

一个用于智能车视觉算法接入与试跑的浏览器仿真框架。它提供车辆运动、车载前视相机、灰度与二值图像、赛道显示和算法控制接口。智能体负责导入赛道与控制算法，也可以通过配套 Skill 读取相机帧并操控车辆。

## 使用

用浏览器打开 `index.html`。无需安装依赖或启动服务。

页面自带赛道和控制器作为接口示例。接入智能体时，可替换赛道掩码并安装自己的控制算法。

## 智能体接口

### 导入赛道

赛道以扁平数组表示，`1` 表示可行驶路面，`0` 表示场外。仿真器会将掩码映射到俯视图画布。

```js
window.smartcarSim.setTrack({
  width: 590,
  height: 341,
  mask: roadMask, // width * height 个 0/1 值
  start: { x: 90, y: 270, heading: 0 } // 坐标以输入掩码像素为单位，角度用弧度
});
```

### 接入算法

算法每帧收到前视相机的二值图像，并返回 `-1` 到 `1` 的转向命令。正值表示向右转。

```js
window.smartcarSim.setAlgorithm((frame) => {
  // frame.width, frame.height, frame.pixels, frame.time
  // pixels[y * frame.width + x]：1 为亮像素，0 为暗像素
  return { steer: 0 };
});
```

算法异常或返回无效舵量时，仿真会暂停并在运行记录中显示错误。`setAlgorithm(null)` 切回手动舵机。

### 仿真控制

```js
window.smartcarSim.getCameraFrame(); // 读取当前相机帧
window.smartcarSim.getState();       // 读取车辆状态和模拟时间
window.smartcarSim.start();
window.smartcarSim.stop();
window.smartcarSim.step();
window.smartcarSim.reset();
```

## 智能体 Skill

`skills/smartcar-vision-simulator/SKILL.md` 描述智能体如何导入赛道、连接控制算法、读取运行状态并调整策略。将该 Skill 提供给智能体后，智能体可以围绕仿真器接口完成试跑与控制。

## 许可

MIT License，见 [LICENSE](LICENSE)。

