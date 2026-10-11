---
name: smartcar-vision-simulator
description: Import a track and control the browser based smart car vision simulator with an agent supplied steering algorithm.
---

# Smart Car Vision Simulator

Use this Skill when an agent needs to load a track, read simulated camera frames, run a steering controller, or inspect a trial in the Smart Car Vision Simulator.

## Connect to the simulator

Open the simulator page and use the browser's page JavaScript context. The simulator exposes `window.smartcarSim`.

Available methods:

- `setTrack({ width, height, mask, start })`
- `setAlgorithm(fn)` or `setAlgorithm(null)`
- `getCameraFrame()`
- `getState()`
- `start()`, `stop()`, `step()`, `reset()`

## Import a track

Convert the supplied track image or track description into a flat mask array. Use `1` for drivable road and `0` for off-road. Keep the source dimensions in `width` and `height`; the simulator scales the mask to its map canvas.

Provide a start point in the same pixel coordinates as the source mask. `heading` is optional and uses radians, with zero pointing to the right.

```js
window.smartcarSim.setTrack({
  width,
  height,
  mask,
  start: { x, y, heading }
});
```

Confirm the returned normalized start and inspect the overhead map before running.

## Install a steering algorithm

Install an algorithm callback. It receives the current binary camera frame and must return a finite steering value from `-1` to `1`.

```js
window.smartcarSim.setAlgorithm((frame) => {
  const { width, height, pixels, time } = frame;
  // Derive steering from this frame or from agent-maintained state.
  return { steer: 0 };
});
```

`pixels[y * width + x]` is `1` for a bright pixel and `0` for a dark pixel. Positive steering turns right. The simulator clamps values to the supported range; invalid results pause the run.

## Run and observe

1. Import the track and install the controller.
2. Call `reset()` and inspect `getCameraFrame()` before the trial.
3. Call `start()` to run continuously, or `step()` for a single simulation step.
4. Read `getState()` and camera frames to evaluate progress and adjust the algorithm.
5. Call `stop()` when the trial is complete. Use `setAlgorithm(null)` to return control to the manual steering slider.

Keep controller state inside the agent callback's closure when temporal filtering or stage tracking is needed. Do not assume that a successful API call means the vehicle completed the track; inspect the returned vehicle state and the camera frames during the run.

