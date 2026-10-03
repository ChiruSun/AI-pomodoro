<template>
  <main class="pomodoro">
    <!-- 標題 -->
    <h1>🍅 番茄鐘</h1>

    <!-- 模式 -->
    <div class="mode-buttons">
      <button :class="{ active: mode === 'work' }" :disabled="isRunning" @click="changeMode('work')">工作</button>

      <button :class="{ active: mode === 'shortBreak' }" :disabled="isRunning" @click="changeMode('shortBreak')">
        短休息
      </button>

      <button :class="{ active: mode === 'longBreak' }" :disabled="isRunning" @click="changeMode('longBreak')">
        長休息
      </button>
    </div>

    <!-- 計時器 -->
    <section class="timer">
      <div class="mode-name">
        {{ modeName }}
      </div>

      <div class="time">
        {{ displayTime }}
      </div>

      <!-- 進度條 -->
      <div class="progress">
        <div class="progress-bar" :style="{ width: `${progress}%` }"></div>
      </div>

      <!-- 控制按鈕 -->
      <div class="controls">
        <button v-if="!isRunning" class="start" @click="startTimer">▶ 開始</button>

        <button v-else class="pause" @click="pauseTimer">⏸ 暫停</button>

        <button class="reset" @click="resetTimer">🔄 重置</button>
      </div>
    </section>

    <!-- 完成輪數 -->
    <div class="rounds">
      完成工作輪數：
      <strong>{{ completedRounds }}</strong>
    </div>

    <!-- 設定 -->
    <section class="settings">
      <h2>⚙️ 時間設定</h2>

      <div class="setting-item">
        <label> 工作時間 </label>

        <input v-model.number="settings.work" type="number" min="1" max="120" :disabled="isRunning" />

        <span>分鐘</span>
      </div>

      <div class="setting-item">
        <label> 短休息 </label>

        <input v-model.number="settings.shortBreak" type="number" min="1" max="60" :disabled="isRunning" />

        <span>分鐘</span>
      </div>

      <div class="setting-item">
        <label> 長休息 </label>

        <input v-model.number="settings.longBreak" type="number" min="1" max="120" :disabled="isRunning" />

        <span>分鐘</span>
      </div>

      <button class="apply" :disabled="isRunning" @click="applySettings">套用設定</button>
    </section>
  </main>
</template>

<script setup>
import { ref, computed, onUnmounted } from "vue";

// ====================
// 計時設定
// ====================

const settings = ref({
  work: Number(localStorage.getItem("pomodoro-work")) || 25,
  shortBreak: Number(localStorage.getItem("pomodoro-short")) || 5,
  longBreak: Number(localStorage.getItem("pomodoro-long")) || 15,
});

// ====================
// 狀態
// ====================

const mode = ref("work");
const remainingSeconds = ref(settings.value.work * 60);

const isRunning = ref(false);
const completedRounds = ref(0);

let timer = null;

// ====================
// 計算目前模式名稱
// ====================

const modeName = computed(() => {
  if (mode.value === "work") {
    return "工作時間";
  }

  if (mode.value === "shortBreak") {
    return "短休息";
  }

  return "長休息";
});

// ====================
// 顯示 MM:SS
// ====================

const displayTime = computed(() => {
  const minutes = Math.floor(remainingSeconds.value / 60);
  const seconds = remainingSeconds.value % 60;

  return `${String(minutes).padStart(2, "0")}:${String(seconds).padStart(2, "0")}`;
});

// ====================
// 目前模式的總時間
// ====================

const totalSeconds = computed(() => {
  if (mode.value === "work") {
    return settings.value.work * 60;
  }

  if (mode.value === "shortBreak") {
    return settings.value.shortBreak * 60;
  }

  return settings.value.longBreak * 60;
});

// ====================
// 進度百分比
// ====================

const progress = computed(() => {
  if (totalSeconds.value === 0) {
    return 0;
  }

  return ((totalSeconds.value - remainingSeconds.value) / totalSeconds.value) * 100;
});

// ====================
// 開始計時
// ====================

function startTimer() {
  if (isRunning.value) {
    return;
  }

  isRunning.value = true;

  timer = setInterval(() => {
    if (remainingSeconds.value > 0) {
      remainingSeconds.value--;
    } else {
      finishTimer();
    }
  }, 1000);
}

// ====================
// 暫停
// ====================

function pauseTimer() {
  isRunning.value = false;

  clearInterval(timer);
  timer = null;
}

// ====================
// 重置
// ====================

function resetTimer() {
  pauseTimer();

  remainingSeconds.value = totalSeconds.value;
}

// ====================
// 時間結束
// ====================

function finishTimer() {
  pauseTimer();

  // 瀏覽器提示
  alert(`${modeName.value}結束！`);

  if (mode.value === "work") {
    completedRounds.value++;

    // 每 4 個工作循環後進入長休息
    if (completedRounds.value % 4 === 0) {
      changeMode("longBreak");
    } else {
      changeMode("shortBreak");
    }
  } else {
    // 休息結束後回到工作
    changeMode("work");
  }
}

// ====================
// 切換模式
// ====================

function changeMode(newMode) {
  pauseTimer();

  mode.value = newMode;

  if (newMode === "work") {
    remainingSeconds.value = settings.value.work * 60;
  }

  if (newMode === "shortBreak") {
    remainingSeconds.value = settings.value.shortBreak * 60;
  }

  if (newMode === "longBreak") {
    remainingSeconds.value = settings.value.longBreak * 60;
  }
}

// ====================
// 套用設定
// ====================

function applySettings() {
  // 確保最少 1 分鐘
  settings.value.work = Math.max(1, Number(settings.value.work));
  settings.value.shortBreak = Math.max(1, Number(settings.value.shortBreak));
  settings.value.longBreak = Math.max(1, Number(settings.value.longBreak));

  // 儲存到 localStorage
  localStorage.setItem("pomodoro-work", settings.value.work);
  localStorage.setItem("pomodoro-short", settings.value.shortBreak);
  localStorage.setItem("pomodoro-long", settings.value.longBreak);

  resetTimer();
}

// ====================
// 離開頁面前清除 timer
// ====================

onUnmounted(() => {
  clearInterval(timer);
});
</script>

<style scoped>
* {
  box-sizing: border-box;
}

.pomodoro {
  width: min(500px, 90%);
  margin: 50px auto;
  padding: 30px;

  text-align: center;

  background: #ffffff;
  border-radius: 20px;

  box-shadow: 0 10px 30px rgba(0, 0, 0, 0.1);
}

h1 {
  margin-bottom: 25px;
}

/* 模式按鈕 */

.mode-buttons {
  display: flex;
  gap: 8px;
  justify-content: center;
}

.mode-buttons button {
  padding: 8px 15px;

  border: none;
  border-radius: 8px;

  background: #eeeeee;
  cursor: pointer;
}

.mode-buttons button.active {
  background: #ff6347;
  color: white;
}

.mode-buttons button:disabled {
  cursor: not-allowed;
  opacity: 0.6;
}

/* Timer */

.timer {
  margin-top: 30px;
}

.mode-name {
  font-size: 20px;
  margin-bottom: 10px;
}

.time {
  font-size: 80px;
  font-weight: bold;
  letter-spacing: 3px;
  margin: 40px 0;
}

/* 進度條 */

.progress {
  height: 8px;
  margin-top: 20px;

  background: #eeeeee;
  border-radius: 10px;

  overflow: hidden;
}

.progress-bar {
  height: 100%;

  background: #ff6347;

  transition: width 0.3s;
}

/* 控制 */

.controls {
  display: flex;
  justify-content: center;
  gap: 10px;

  margin-top: 25px;
}

.controls button {
  padding: 10px 20px;

  border: none;
  border-radius: 8px;

  cursor: pointer;
}

.start {
  background: #4caf50;
  color: white;
}

.pause {
  background: #ff9800;
  color: white;
}

.reset {
  background: #eeeeee;
}

/* 輪數 */

.rounds {
  margin-top: 25px;
  font-size: 18px;
}

/* 設定 */

.settings {
  margin-top: 30px;
  padding-top: 25px;

  border-top: 1px solid #eeeeee;
}

.settings h2 {
  margin-bottom: 20px;
}

.setting-item {
  display: flex;
  align-items: center;

  margin-bottom: 12px;
}

.setting-item label {
  width: 100px;
  text-align: left;
}

.setting-item input {
  width: 80px;
  padding: 7px;

  text-align: center;

  border: 1px solid #cccccc;
  border-radius: 6px;
}

.setting-item span {
  margin-left: 8px;
}

.apply {
  margin-top: 10px;

  padding: 10px 20px;

  border: none;
  border-radius: 8px;

  background: #333333;
  color: white;

  cursor: pointer;
}

.apply:disabled {
  opacity: 0.5;
  cursor: not-allowed;
}
</style>
