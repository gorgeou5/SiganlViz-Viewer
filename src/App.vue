<template>
  <div class="wrapper">
    <div class="inner">
      <div class="content">
        <div class="title">
          <h1>교차로 현시 구성</h1>
        </div>
        <div class="card-grid Alink-card">
          <span>A링</span>
          <!-- A링 inputs -->
          <div v-for="(val, idx) in Ainputs" :key="'A' + idx" class="card">
            <input v-model="Ainputs[idx]" placeholder="L010270 or S010270" maxlength="7" class="input-box" />
            <svg v-if="Aarrows[idx]" width="200" height="200" viewBox="0 0 400 400">
              <circle cx="200" cy="200" r="180" fill="none" stroke="#646464ff" stroke-width="2" />
              <component :is="Aarrows[idx].type === 'S' ? 'line' : 'polyline'" v-bind="arrowProps(Aarrows[idx], 0)" />
              <defs>
                <marker id="arrowhead" markerWidth="10" markerHeight="7" refX="8" refY="3.5" orient="auto">
                  <polygon points="0 0, 10 3.5, 0 7" fill="black" />
                </marker>
              </defs>
            </svg>
          </div>
          <button v-if="Ainputs.length < 6" @click="addInput('A')" class="add-button">+</button>
        </div>

        <div class="card-grid Blink-card">
          <span>B링</span>
          <!-- B링 inputs -->
          <div v-for="(val, idx) in Binputs" :key="'B' + idx" class="card">
            <input v-model="Binputs[idx]" placeholder="L010270 or S010270" maxlength="7" class="input-box" />
            <svg v-if="Barrows[idx]" width="200" height="200" viewBox="0 0 400 400">
              <circle cx="200" cy="200" r="180" fill="none" stroke="#646464ff" stroke-width="2" />
              <component :is="Barrows[idx].type === 'S' ? 'line' : 'polyline'" v-bind="arrowProps(Barrows[idx], 1)" />
              <defs>
                <marker id="arrowhead" markerWidth="10" markerHeight="7" refX="8" refY="3.5" orient="auto">
                  <polygon points="0 0, 10 3.5, 0 7" fill="black" />
                </marker>
              </defs>
            </svg>
          </div>
          <button v-if="Binputs.length < 6" @click="addInput('B')" class="add-button">+</button>
        </div>

        <button @click="removeAll" class="remove-button">삭제</button>

        <div class="card-grid" style="margin-top: 30px">
          <div v-for="(pair, idx) in mergedArrows" :key="'merged' + idx" class="card">
            <svg width="200" height="200" viewBox="0 0 400 400">
              <circle cx="200" cy="200" r="180" fill="none" stroke="#646464ff" stroke-width="2" />
              <defs>
                <marker id="arrowhead" markerWidth="10" markerHeight="7" refX="8" refY="3.5" orient="auto">
                  <polygon points="0 0, 10 3.5, 0 7" fill="black" />
                </marker>
              </defs>
              <component v-for="(arrow, aIdx) in pair" :key="aIdx" :is="arrow.type === 'S' ? 'line' : 'polyline'" v-bind="arrowProps(arrow, aIdx)" />
            </svg>
          </div>
        </div>
      </div>
      <footer>
        <small>Copyright &copy;Sieun CHOI with Seonil YOON</small>
      </footer>
    </div>
  </div>
</template>

<script setup>
import { ref, watch } from "vue";

const Ainputs = ref([""]); // 시작할 때 1개의 input만 있음
const Binputs = ref([""]); // 시작할 때 1개의 input만 있음
const Aarrows = ref([null]); // 초기에는 null
const Barrows = ref([null]); // 초기에는 null
const mergedArrows = ref([]);

const toRadians = (deg) => (deg * Math.PI) / 180;

function point(angle) {
  const rad = toRadians(angle);
  return {
    x: 200 + 180 * Math.sin(rad),
    y: 200 - 180 * Math.cos(rad),
  };
}

function shortenedPoint(angle, shrink = 10) {
  const rad = toRadians(angle);
  const fullX = 200 + 180 * Math.sin(rad);
  const fullY = 200 - 180 * Math.cos(rad);
  const dx = fullX - 200;
  const dy = fullY - 200;
  const length = Math.sqrt(dx * dx + dy * dy);
  if (length === 0) return { x: fullX, y: fullY };
  const ratio = (length - shrink) / length;
  return {
    x: 200 + dx * ratio,
    y: 200 + dy * ratio,
  };
}

function getMiddleOfLine(startAngle, endAngle) {
  const p1 = point(startAngle);
  const p2 = shortenedPoint(endAngle);
  return {
    x: (p1.x + p2.x) / 2,
    y: (p1.y + p2.y) / 2,
  };
}

function arrowProps(arrow, idx = 0, shrink = 10) {
  const colors = ["red", "blue"];
  if (arrow.type === "S") {
    return {
      x1: point(arrow.startAngle).x,
      y1: point(arrow.startAngle).y,
      x2: shortenedPoint(arrow.endAngle, shrink).x,
      y2: shortenedPoint(arrow.endAngle, shrink).y,
      stroke: colors[idx],
      "stroke-width": 5,
      "marker-end": "url(#arrowhead)",
    };
  } else {
    const middle = arrow.middle || { x: 200, y: 200 };
    return {
      points: `${point(arrow.startAngle).x},${point(arrow.startAngle).y} ${middle.x},${middle.y} ${shortenedPoint(arrow.endAngle, shrink).x},${shortenedPoint(arrow.endAngle, shrink).y}`,
      fill: "none",
      stroke: colors[idx],
      "stroke-width": 5,
      "marker-end": "url(#arrowhead)",
    };
  }
}

function parseInput(text) {
  if (text.length === 7) {
    const type = text[0].toUpperCase();
    const start = parseInt(text.slice(1, 4));
    const end = parseInt(text.slice(4, 7));
    if ((type === "L" || type === "S") && !isNaN(start) && !isNaN(end) && start >= 0 && start <= 360 && end >= 0 && end <= 360) {
      return { type, startAngle: start, endAngle: end };
    }
  }
  return null;
}

function generateArrows() {
  Aarrows.value = Ainputs.value.map(parseInput);
  Barrows.value = Binputs.value.map(parseInput);

  mergedArrows.value = [];

  for (let i = 0; i < 6; i++) {
    const A = Aarrows.value[i];
    const B = Barrows.value[i];

    if (A && B) {
      // A가 S, B가 L
      if (A.type === "S" && B.type === "L") {
        const mid = getMiddleOfLine(A.startAngle, A.endAngle);
        B.middle = mid;
      }

      // A가 L, B가 S
      if (A.type === "L" && B.type === "S") {
        const mid = getMiddleOfLine(B.startAngle, B.endAngle);
        A.middle = mid;
      }

      mergedArrows.value.push([A, B]);
    }
  }
}

function removeAll() {
  Ainputs.value = [""];
  Binputs.value = [""];
  Aarrows.value = [null];
  Barrows.value = [null];
  mergedArrows.value = [];
}

function addInput(type) {
  if (type === "A" && Ainputs.value.length < 6) {
    Ainputs.value.push("");
  } else if (type === "B" && Binputs.value.length < 6) {
    Binputs.value.push("");
  }
}

watch(
  [Ainputs, Binputs],
  () => {
    generateArrows();
  },
  { deep: true }
);
</script>

<style scoped>
.inner {
  display: flex;
  /* flex: 1; */
  height: 100vh;
  flex-direction: column;
  justify-content: space-between;
}

.title {
  margin: 20px auto 100px;
  text-align: center;
}

.content {
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  font-size: 20px;
}

.card-grid {
  max-width: 1490px;
  display: flex;
  flex-wrap: wrap;
  justify-content: center;
  gap: 20px;
  margin-bottom: 20px;
}
.card {
  width: 200px;
  padding: 10px;
  border: solid 1.5px #666;
}

.input-box {
  width: 95%;
  display: inline-block;
  font-size: 16px;
}

.add-button {
  background-color: rgb(207, 207, 207);
  width: 40px;
  height: 40px;
  border-radius: 20%;
}

.remove-button {
  width: 200px;
  height: 32px;
  font-size: 16px;
  color: #ffffff;
  background-color: rgb(127, 127, 127);
}

.footer {
  width: 100%;
  text-align: right;
  background-color: rgb(207, 207, 207);
  position: absolute;
  bottom: 0;
  right: 0;
}

footer {
  height: 60px; /*footer의 높이*/
  position: relative;
  transform: translateY(0%);
  display: flex;
  align-items: center;
  justify-content: center;
  background-color: rgb(207, 207, 207);
}
</style>
