<template>
  <div ref="container" class="container" @mouseup.stop="onMouseUp" @mousemove="onMouseMove" @dragstart.prevent>
    <div ref="box" class="box" :style="boxStyle" @mousedown.stop="onBoxMouseDown">
      <img src="../assets/flower.png" alt="Flower" class="image" />
    </div>
    <template v-for="({ corner }, index) in corners" :key="index">
      <div v-if="index % 2 === 0" class="corner" :style="{ left: corners[index] + 'px', top: corners[index + 1] + 'px' }" @mousedown.stop="onMouseDown(index)"></div>
    </template>
    <div class="rotator" :style="rotatorStyle" @mousedown.stop="onRotateMouseDown"></div>
  </div>
</template>
<script setup>
import { ref, reactive, useTemplateRef, computed } from "vue";
import PerspT from "../perspective-transform.js";
const corners = reactive([100, 100, 300, 100, 100, 300, 300, 300]);
const isBoxSelected = ref(false);
const selectedCorner = ref(-1);
const isRotating = ref(false);
const initialCorners = reactive([]);
const boxCenter = reactive({ x: 0, y: 0 });
const box = useTemplateRef("box");
const ip = ref({ x: 0, y: 0 });

const throttle = (func, limit) => {
  let lastFunc;
  let lastRan;
  return function (...args) {
    const context = this;
    if (!lastRan) {
      func.apply(context, args);
      lastRan = Date.now();
    } else {
      clearTimeout(lastFunc);
      lastFunc = setTimeout(() => {
        if (Date.now() - lastRan >= limit) {
          func.apply(context, args);
          lastRan = Date.now();
        }
      }, limit - (Date.now() - lastRan));
    }
  };
};

const transform2d = (elt, x1, y1, x2, y2, x3, y3, x4, y4) => {
  if (!elt) {
    return "matrix(1, 0, 0, 1, 0, 0)";
  }
  var w = elt ? elt.offsetWidth : 152,
    h = elt ? elt.offsetHeight : 122;
  var transform = new PerspT([0, 0, w, 0, 0, h, w, h], [x1, y1, x2, y2, x3, y3, x4, y4]);
  var t = transform.coeffs;
  t = [t[0], t[3], 0, t[6], t[1], t[4], 0, t[7], 0, 0, 1, 0, t[2], t[5], 0, t[8]];
  return "matrix3d(" + t.join(", ") + ")";
};

const boxStyle = computed(() => {
  return {
    transform: transform2d(box.value, corners[0], corners[1], corners[2], corners[3], corners[4], corners[5], corners[6], corners[7]),
  };
});

// Mouse down event for starting the drag
const onBoxMouseDown = (event) => {
  isBoxSelected.value = true;
  ip.value.x = event.clientX;
  ip.value.y = event.clientY;
  console.log("mouseboxdown", ip.value.x, ip.value.y);
};

const onRotateMouseDown = (event) => {
  isRotating.value = true;
  ip.value.x = event.clientX;
  ip.value.y = event.clientY;
  Object.assign(initialCorners, corners);
  boxCenter.x = (corners[0] + corners[2] + corners[4] + corners[6]) / 4;
  boxCenter.y = (corners[1] + corners[3] + corners[5] + corners[7]) / 4;
};
const rotatorStyle = computed(() => {
  const x = (corners[0] + corners[2]) / 2;
  const y = (corners[1] + corners[3]) / 2;
  const edgeVecX = corners[2] - corners[0];
  const edgeVecY = corners[3] - corners[1];
  const edgeLength = Math.sqrt(edgeVecX * edgeVecX + edgeVecY * edgeVecY);
  const perpVecX = -edgeVecY / edgeLength;
  const perpVecY = edgeVecX / edgeLength;
  const handleDistance = 30;
  return {
    left: x + perpVecX * handleDistance + "px",
    top: y + perpVecY * handleDistance + "px",
  };
});
const onMouseDown = (index) => {
  selectedCorner.value = index;
  console.log("mousedown");
};
const onMouseUp = () => {
  selectedCorner.value = -1;
  isBoxSelected.value = false;
  isRotating.value = false;
  console.log("mouseup");
};

const onMouseMove = throttle((event) => {
  if (isRotating.value) {
    const { clientX, clientY } = event;
    const initialAngle = Math.atan2(ip.value.y - boxCenter.y, ip.value.x - boxCenter.x);
    const currentAngle = Math.atan2(clientY - boxCenter.y, clientX - boxCenter.x);
    const angle = currentAngle - initialAngle;
    for (let i = 0; i < initialCorners.length; i += 2) {
      const x = initialCorners[i];
      const y = initialCorners[i + 1];
      const rotatedX = boxCenter.x + (x - boxCenter.x) * Math.cos(angle) - (y - boxCenter.y) * Math.sin(angle);
      const rotatedY = boxCenter.y + (x - boxCenter.x) * Math.sin(angle) + (y - boxCenter.y) * Math.cos(angle);
      corners[i] = rotatedX;
      corners[i + 1] = rotatedY;
    }
    return;
  }
  if (isBoxSelected.value) {
    const { clientX, clientY } = event;
    console.log("mousemove", clientX, clientY);
    const dx = clientX - ip.value.x;
    const dy = clientY - ip.value.y;

    // Update all corners to move the box
    for (let i = 0; i < corners.length; i += 2) {
      corners[i] += dx; // Update x-coordinate
      corners[i + 1] += dy; // Update y-coordinate
    }
    // Update the initial position
    ip.value.x = clientX;
    ip.value.y = clientY;
  }
  if (selectedCorner.value !== -1) {
    const { clientX, clientY } = event;
    console.log("mousemove", selectedCorner.value), clientX, clientY;
    corners[selectedCorner.value] = clientX;
    corners[selectedCorner.value + 1] = clientY;
  }
}, 16);
</script>
<style>
.container {
  width: 100vw;
  height: 100vh;
  position: relative;
  background: #f9f9f9;
}
.box {
  position: absolute;
  top: 0px;
  left: 0px;
  width: 150px;
  height: 120px;
  border: 1px solid red;
  transform-origin: 0 0;
  -webkit-transform-origin: 0 0;
  -moz-transform-origin: 0 0;
  -o-transform-origin: 0 0;
  cursor: move;
}
.box img {
  width: 150px;
  height: 120px;
}

.corner {
  position: absolute;
  width: 12px;
  height: 12px;
  background: #007bff;
  border-radius: 50%;
  cursor: pointer;
  margin-left: -6px;
  margin-top: -6px;
}
.rotator {
  position: absolute;
  width: 16px;
  height: 16px;
  background: #ffc107;
  border: 1px solid #d39e00;
  border-radius: 50%;
  cursor: grab;
  margin-left: -8px;
  margin-top: -8px;
}
</style>
