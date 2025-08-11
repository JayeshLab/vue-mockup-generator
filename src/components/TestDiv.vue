<template>
  <div id="container" @mousemove="onMouseMove" @mouseup="onMouseUp">
    <!-- Rectangle -->
    <div class="rect" :style="rectStyle" @mousedown="onRectMouseDown">
      <img src="../assets/flower.png" alt="Flower" style="width: 100%; height: 100%; object-fit: cover" />
    </div>

    <!-- 4 draggable vertex handles -->
    <div v-for="(corner, index) in corners" :key="index" class="vertex" :style="{ left: corner.x + 'px', top: corner.y + 'px' }" @mousedown.stop="onVertexMouseDown(index, $event)"></div>
  </div>
</template>

<script>
export default {
  name: "RealRectangleWithVertices",
  data() {
    return {
      rect: {
        top: 100,
        left: 100,
        width: 200,
        height: 200,
      },
      draggingCorner: null,
      draggingRect: false,
      lastMouse: { x: 0, y: 0 },
    };
  },
  computed: {
    rectStyle() {
      return {
        top: this.rect.top + "px",
        left: this.rect.left + "px",
        width: this.rect.width + "px",
        height: this.rect.height + "px",
      };
    },
    corners() {
      const { top, left, width, height } = this.rect;
      return [
        { x: left, y: top }, // Top-left
        { x: left + width, y: top }, // Top-right
        { x: left + width, y: top + height }, // Bottom-right
        { x: left, y: top + height }, // Bottom-left
      ];
    },
  },
  methods: {
    onVertexMouseDown(index, event) {
      this.draggingCorner = index;
      this.lastMouse = { x: event.clientX, y: event.clientY };
      event.preventDefault();
    },
    onRectMouseDown(event) {
      this.draggingRect = true;
      this.lastMouse = { x: event.clientX, y: event.clientY };
      event.preventDefault();
    },
    onMouseMove(event) {
      const dx = event.clientX - this.lastMouse.x;
      const dy = event.clientY - this.lastMouse.y;

      if (this.draggingCorner !== null) {
        this.resizeFromCorner(this.draggingCorner, dx, dy);
      } else if (this.draggingRect) {
        this.rect.left += dx;
        this.rect.top += dy;
      }

      this.lastMouse = { x: event.clientX, y: event.clientY };
    },
    onMouseUp() {
      this.draggingCorner = null;
      this.draggingRect = false;
    },
    resizeFromCorner(index, dx, dy) {
      const r = this.rect;
      switch (index) {
        case 0: // Top-left
          r.left += dx;
          r.top += dy;
          r.width -= dx;
          r.height -= dy;
          break;
        case 1: // Top-right
          r.top += dy;
          r.width += dx;
          r.height -= dy;
          break;
        case 2: // Bottom-right
          r.width += dx;
          r.height += dy;
          break;
        case 3: // Bottom-left
          r.left += dx;
          r.width -= dx;
          r.height += dy;
          break;
      }

      // Prevent negative sizes
      r.width = Math.max(r.width, 20);
      r.height = Math.max(r.height, 20);
    },
  },
};
</script>

<style scoped>
#container {
  width: 100vw;
  height: 100vh;
  position: relative;
  background: #f9f9f9;
}

.rect {
  position: absolute;
  background: rgba(0, 123, 255, 0.2);
  border: 2px solid #007bff;
  cursor: move;
}

.vertex {
  position: absolute;
  width: 12px;
  height: 12px;
  background: #007bff;
  border-radius: 50%;
  cursor: pointer;
  margin-left: -6px;
  margin-top: -6px;
}
</style>
