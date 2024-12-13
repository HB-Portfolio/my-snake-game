<template>
  <div>
    <div
      v-if="!assetsLoaded"
      class="fixed inset-0 z-50 flex flex-col items-center justify-center bg-gray-100"
    >
      <div class="w-64 bg-white rounded-lg shadow-lg p-6">
        <div class="text-center mb-4">
          <h2 class="text-xl font-bold text-gray-800">Loading Game Assets</h2>
          <p class="text-gray-600">{{ loadingProgress }}%</p>
        </div>
        <div class="w-full bg-gray-200 rounded-full h-2.5">
          <div
            class="bg-orange-600 h-2.5 rounded-full transition-all duration-300"
            :style="{ width: `${loadingProgress}%` }"
          />
        </div>
      </div>
    </div>

    <section
      v-show="assetsLoaded"
      ref="containerRef"
      class="flex flex-col items-center mt-4 px-4 md:px-8 w-full h-full mx-auto flex-grow pb-4"
    >
      <div class="flex flex-col items-center justify-center">
        <div class="mb-2 flex gap-2">
          <div
            class="p-2 w-28 md:w-40 h-20 rounded bg-[url('/public/sprites/score.svg')] bg-contain bg-no-repeat bg-center"
          ></div>
          <div
            class="flex flex-col w-28 md:w-40 rounded bg-[url('/public/sprites/blank.svg')] bg-contain bg-no-repeat bg-center items-center justify-center"
          >
            <p
              class="text-5xl font-bold text-center text-white [text-shadow:_2px_2px_0_rgb(120_38_35),_-2px_-2px_0_rgb(120_38_35),_2px_-2px_0_rgb(120_38_35),_-2px_2px_0_rgb(120_38_35)]"
            >
              {{ score }}
            </p>
          </div>
        </div>

        <div
          class="relative bg-gray-400 bg-opacity-50 bg-[length:30px_30px] bg-[linear-gradient(to_right,#f0f0f0_2px,transparent_2px),linear-gradient(to_bottom,#f0f0f0_2px,transparent_2px)] border-r-2 border-b-2 border-white"
          :style="boardStyle"
        >
          <div
            v-if="isGameRunning"
            v-for="(segment, index) in snake"
            :key="segment.id"
            v-memo="[segment.x, segment.y]"
            :class="[
              'absolute transform-gpu',
              addedSegmentIds.has(segment.id) ? 'animate-ping' : '',
              removedSegmentIds.has(segment.id)
                ? 'animate-pulse opacity-0'
                : '',
              getCornerClass(segment, index),
              getTailClass(segment, index),
              getHeadClass(segment, index),
              getStraightClass(segment, index),
            ]"
            :style="getSegmentStyle(segment)"
          >
            <img
              :src="getSegmentSprite(segment, index)"
              :alt="getSegmentType(segment, index)"
              
              style="width: 100%; height: 100%"
            />
          </div>

          <div
            v-if="isGameRunning"
            class="absolute"
            :style="`left: ${food.x * cellSize}px; top: ${
              food.y * cellSize
            }px; width: ${cellSize}px; height: ${cellSize}px;`"
          >
            <img
              src="public/sprites/apple.svg"
              alt="apple"
              
              style="width: 100%; height: 100%"
            />
          </div>

          <div
            v-if="isGameRunning && goldenApple"
            class="absolute"
            :style="`left: ${goldenApple.x * cellSize}px; top: ${
              goldenApple.y * cellSize
            }px; width: ${cellSize}px; height: ${cellSize}px;`"
          >
            <img
              src="/public/sprites/golden_apple.svg"
              alt="golden apple"
              
              style="width: 100%; height: 100%"
            />
          </div>

          <div
            v-if="isGameRunning && orangeTriangle"
            class="absolute"
            :style="`left: ${orangeTriangle.x * cellSize}px; top: ${
              orangeTriangle.y * cellSize
            }px; width: ${cellSize}px; height: ${cellSize}px;`"
          >
            <img
              src="/public/sprites/agent_orange.svg"
              alt="obstacle"
              
              style="width: 100%; height: 100%"
            />
          </div>

          <div
            v-if="isPaused && isGameRunning"
            class="absolute inset-0 flex items-center justify-center bg-white bg-opacity-75"
          >
            <h2 class="text-2xl font-bold text-gray-700">Paused</h2>
          </div>

          <div
            v-if="!isGameRunning && !showGameOver"
            class="absolute inset-0 flex items-center justify-center bg-white bg-opacity-75"
          >
            <button
              @click="startGame"
              class="md:w-40 w-32 md:hover:scale-105 transition-transform duration-200 ease-in-out"
            >
              <img
                src="/public/sprites/play.svg"
                alt="restart"
                class="w-full h-full"
              />
            </button>
          </div>

          <div
            v-if="showGameOver"
            class="absolute inset-0 flex items-center justify-center bg-black bg-opacity-50 border-t-2 border-l-2 border-white"
          >
            <div class="py-2 px-6 bg-white rounded">
              <h2 class="mb-2 text-2xl font-bold text-center text-red-600">
                Game Over!
              </h2>
              <p class="mb-4 font-medium text-green-500 text-center">
                Your score: {{ score }}
              </p>
              <button
                @click="startGame"
                class="md:w-40 w-32 md:hover:scale-105 transition-transform duration-200 ease-in-out"
              >
                <img
                  src="/public/sprites/play.svg"
                  alt="restart"
                  class="w-full h-full"
                />
              </button>
            </div>
          </div>
        </div>
      </div>

      <div class="flex gap-1 md:gap-2 justify-center mt-4 flex-grow">
        <button
          v-if="isGameRunning"
          @click="restartGame"
          class="md:w-40 w-24 md:hover:scale-105 transition-transform duration-200 ease-in-out"
        >
          <img
            src="/public/sprites/reload.svg"
            alt="restart"
            class="w-full h-full"
          />
        </button>
        <button
          v-if="isGameRunning"
          @click="toggleEndGame"
          class="md:w-40 w-24 md:hover:scale-105 transition-transform duration-200 ease-in-out"
        >
          <img
            src="/public/sprites/end.svg"
            alt="restart"
            class="w-full h-full"
          />
        </button>
        <button
          v-if="isGameRunning"
          @click="togglePause"
          class="md:w-40 w-24 md:hover:scale-105 transition-transform duration-200 ease-in-out"
        >
          <img
            :src="isPaused ? '/sprites/play.svg' : '/sprites/pause.svg'"
            alt="pause toggle"
            class="w-full h-full"
          />
        </button>
      </div>
      <div class="flex flex-col items-center mt-4 md:hidden">
        <div>
          <button
            @click="changeDirectionMobile('up')"
            class="px-6 py-4 font-bold text-white bg-orange-500 rounded md:hover::bg-orange-700 m-1"
          >
            &uarr;
          </button>
        </div>
        <div class="flex">
          <button
            @click="changeDirectionMobile('left')"
            class="px-6 py-4 font-bold text-white bg-orange-500 rounded md:hover::bg-orange-700 m-1"
          >
            &larr;
          </button>
          <button
            @click="changeDirectionMobile('down')"
            class="px-6 py-4 font-bold text-white bg-orange-500 rounded md:hover::bg-orange-700 m-1"
          >
            &darr;
          </button>
          <button
            @click="changeDirectionMobile('right')"
            class="px-6 py-4 font-bold text-white bg-orange-500 rounded md:hover::bg-orange-700 m-1"
          >
            &rarr;
          </button>
        </div>
      </div>
    </section>
  </div>
</template>

<script setup>
import { ref, onMounted, onBeforeUnmount, computed, nextTick } from "vue";

const containerRef = ref(null);

const cellSize = 30;
const boardCols = ref(30);
const boardRows = ref(30);

const snake = ref([]);
const direction = ref({ x: 0, y: 0 });
const food = ref({ x: 0, y: 0 });
const orangeTriangle = ref(null);
const goldenApple = ref(null);
const goldenAppleChance = 1 / 30;
let gameInterval = null;
const isGameRunning = ref(false);
const showGameOver = ref(false);
const isPaused = ref(false);
const score = ref(0);
let growthCounter = 0;
const speed = ref(100);
let lastUpdateTime = 0;
const baseSpeed = 150;
const minSpeed = 70;

const assetsLoaded = ref(false);
const loadingProgress = ref(0);

const addedSegmentIds = ref(new Set());
const removedSegmentIds = ref(new Set());
const flashTimeouts = ref(new Map());

let segmentIdCounter = 0;

const spritePaths = [
  "/sprites/head_up.svg",
  "/sprites/head_down.svg",
  "/sprites/head_left.svg",
  "/sprites/head_right.svg",
  "/sprites/tail_up.svg",
  "/sprites/tail_down.svg",
  "/sprites/tail_left.svg",
  "/sprites/tail_right.svg",
  "/sprites/body_horizontal.svg",
  "/sprites/body_vertical.svg",
  "/sprites/top_right.svg",
  "/sprites/top_left.svg",
  "/sprites/bottom_right.svg",
  "/sprites/bottom_left.svg",
  "/sprites/apple.svg",
  "/sprites/golden_apple.svg",
  "/sprites/agent_orange.svg",
];

const addFlashEffect = (segmentId, type) => {
  if (flashTimeouts.value.has(segmentId)) {
    clearTimeout(flashTimeouts.value.get(segmentId));
    flashTimeouts.value.delete(segmentId);
  }

  if (type === "added") {
    addedSegmentIds.value.add(segmentId);
    const timeout = setTimeout(() => {
      addedSegmentIds.value.delete(segmentId);
      flashTimeouts.value.delete(segmentId);
    }, 150);
    flashTimeouts.value.set(segmentId, timeout);
  } else if (type === "removed") {
    removedSegmentIds.value.add(segmentId);
    const timeout = setTimeout(() => {
      removedSegmentIds.value.delete(segmentId);
      flashTimeouts.value.delete(segmentId);
    }, 150);
    flashTimeouts.value.set(segmentId, timeout);
  }
};

const preloadAssets = async () => {
  let loadedCount = 0;
  const totalImages = spritePaths.length;

  const loadPromises = spritePaths.map((src) => {
    return new Promise((resolve, reject) => {
      const img = new Image();
      img.src = src;
      img.onload = () => {
        loadedCount++;
        loadingProgress.value = Math.floor((loadedCount / totalImages) * 100);
        resolve();
      };
      img.onerror = reject;
    });
  });

  try {
    await Promise.all(loadPromises);
    assetsLoaded.value = true;
  } catch (error) {
    console.error("Failed to load some sprites:", error);
    assetsLoaded.value = true;
  }
};

const createSegment = (x, y) => {
  return { x, y, id: segmentIdCounter++ };
};

const initializeGame = () => {
  segmentIdCounter = 0;
  const startX = Math.floor(boardCols.value / 2);
  const startY = Math.floor(boardRows.value / 2);
  snake.value = [
    createSegment(startX, startY),
    createSegment(startX - 1, startY),
  ];
  direction.value = { x: 1, y: 0 };
  placeFood();
  score.value = 0;
  growthCounter = 0;
  speed.value = baseSpeed;
  lastUpdateTime = 0;
  addedSegmentIds.value.clear();
  removedSegmentIds.value.clear();
  flashTimeouts.value.forEach((timeout) => clearTimeout(timeout));
  flashTimeouts.value.clear();
};

const getSegmentType = (segment, index) => {
  if (index === 0) return "head";
  if (index === snake.value.length - 1) return "tail";
  return "body";
};

const getSegmentDirection = (current, next) => {
  let dx = next.x - current.x;
  let dy = next.y - current.y;

  if (Math.abs(dx) > 1) {
    dx = dx > 0 ? -1 : 1;
  }
  if (Math.abs(dy) > 1) {
    dy = dy > 0 ? -1 : 1;
  }

  if (dx === 1) return "left";
  if (dx === -1) return "right";
  if (dy === 1) return "up";
  if (dy === -1) return "down";
  return "right";
};

const getSegmentSprite = (segment, index) => {
  if (snake.value.length === 1) {
    return `/sprites/head_${
      direction.value.x > 0
        ? "right"
        : direction.value.x < 0
        ? "left"
        : direction.value.y > 0
        ? "down"
        : "up"
    }.svg`;
  }

  const type = getSegmentType(segment, index);

  if (type === "head") {
    if (!snake.value[1]) {
      return `/sprites/head_${
        direction.value.x > 0
          ? "right"
          : direction.value.x < 0
          ? "left"
          : direction.value.y > 0
          ? "down"
          : "up"
      }.svg`;
    }
    const dir = getSegmentDirection(segment, snake.value[1]);
    return `/sprites/head_${dir}.svg`;
  }

  if (type === "tail") {
    if (!snake.value[snake.value.length - 2]) {
      return "/sprites/tail_right.svg";
    }
    const dir = getSegmentDirection(
      snake.value[snake.value.length - 2],
      segment
    );
    return `/sprites/tail_${dir}.svg`;
  }

  const prev = snake.value[index - 1];
  const next = snake.value[index + 1];

  if (!prev || !next) {
    return "/sprites/body_horizontal.svg";
  }

  const prevDir = getSegmentDirection(segment, prev);
  const nextDir = getSegmentDirection(segment, next);

  if (
    (prevDir === "up" && nextDir === "down") ||
    (prevDir === "down" && nextDir === "up") ||
    (prevDir === "left" && nextDir === "right") ||
    (prevDir === "right" && nextDir === "left")
  ) {
    return prevDir === "up" || prevDir === "down"
      ? "/sprites/body_vertical.svg"
      : "/sprites/body_horizontal.svg";
  }

  if (
    (prevDir === "right" && nextDir === "up") ||
    (prevDir === "up" && nextDir === "right")
  ) {
    return "/sprites/top_right.svg";
  }
  if (
    (prevDir === "left" && nextDir === "up") ||
    (prevDir === "up" && nextDir === "left")
  ) {
    return "/sprites/top_left.svg";
  }
  if (
    (prevDir === "right" && nextDir === "down") ||
    (prevDir === "down" && nextDir === "right")
  ) {
    return "/sprites/bottom_right.svg";
  }
  if (
    (prevDir === "left" && nextDir === "down") ||
    (prevDir === "down" && nextDir === "left")
  ) {
    return "/sprites/bottom_left.svg";
  }

  return "/sprites/body_horizontal.svg";
};

const getHeadClass = (segment, index) => {
  const sprite = getSegmentSprite(segment, index);
  if (index === 0) {
    if (sprite.includes("head_up")) return "head-up";
    if (sprite.includes("head_down")) return "head-down";
    if (sprite.includes("head_left")) return "head-left";
    if (sprite.includes("head_right")) return "head-right";
  }
  return "";
};

const getCornerClass = (segment, index) => {
  const sprite = getSegmentSprite(segment, index);
  if (sprite.includes("top_right")) return "corner-top-right";
  if (sprite.includes("top_left")) return "corner-top-left";
  if (sprite.includes("bottom_right")) return "corner-bottom-right";
  if (sprite.includes("bottom_left")) return "corner-bottom-left";
  return "";
};

const getTailClass = (segment, index) => {
  const sprite = getSegmentSprite(segment, index);
  if (index === snake.value.length - 1) {
    if (sprite.includes("tail_up")) return "tail-up";
    if (sprite.includes("tail_down")) return "tail-down";
    if (sprite.includes("tail_left")) return "tail-left";
    if (sprite.includes("tail_right")) return "tail-right";
  }
  return "";
};

const getStraightClass = (segment, index) => {
  const sprite = getSegmentSprite(segment, index);
  if (index > 0 && index < snake.value.length - 1) {
    if (sprite.includes("body_vertical")) return "straight-vertical";
    if (sprite.includes("body_horizontal")) return "straight-horizontal";
  }
  return "";
};

const placeFood = () => {
  let newFoodPosition;
  do {
    newFoodPosition = {
      x: Math.floor(Math.random() * boardCols.value),
      y: Math.floor(Math.random() * boardRows.value),
    };
  } while (
    snake.value.some(
      (segment) =>
        segment.x === newFoodPosition.x && segment.y === newFoodPosition.y
    )
  );
  food.value = newFoodPosition;

  if (snake.value.length > 1) {
    let newOrangePosition;
    do {
      newOrangePosition = {
        x: Math.floor(Math.random() * boardCols.value),
        y: Math.floor(Math.random() * boardRows.value),
      };
    } while (
      (newOrangePosition.x === food.value.x &&
        newOrangePosition.y === food.value.y) ||
      snake.value.some(
        (segment) =>
          segment.x === newOrangePosition.x && segment.y === newOrangePosition.y
      )
    );
    orangeTriangle.value = newOrangePosition;
  } else {
    orangeTriangle.value = null;
  }

  if (Math.random() < goldenAppleChance) {
    let newGoldenApplePosition;
    do {
      newGoldenApplePosition = {
        x: Math.floor(Math.random() * boardCols.value),
        y: Math.floor(Math.random() * boardRows.value),
      };
    } while (
      (newGoldenApplePosition.x === food.value.x &&
        newGoldenApplePosition.y === food.value.y) ||
      (orangeTriangle.value &&
        newGoldenApplePosition.x === orangeTriangle.value.x &&
        newGoldenApplePosition.y === orangeTriangle.value.y) ||
      snake.value.some(
        (segment) =>
          segment.x === newGoldenApplePosition.x &&
          segment.y === newGoldenApplePosition.y
      )
    );
    goldenApple.value = newGoldenApplePosition;
  } else {
    goldenApple.value = null;
  }
};

const startGame = () => {
  initializeGame();
  isGameRunning.value = true;
  showGameOver.value = false;
  isPaused.value = false;
  if (gameInterval) cancelAnimationFrame(gameInterval);
  gameLoop();
};

const restartGame = () => {
  initializeGame();
  isGameRunning.value = true;
  showGameOver.value = false;
  isPaused.value = false;
  if (gameInterval) cancelAnimationFrame(gameInterval);
  gameLoop();
};

const endGame = () => {
  if (gameInterval) cancelAnimationFrame(gameInterval);
  isGameRunning.value = false;
  showGameOver.value = false;
  isPaused.value = false;
};

const toggleEndGame = () => {
  if (gameInterval) cancelAnimationFrame(gameInterval);
  isGameRunning.value = false;
  showGameOver.value = false;
  isPaused.value = false;
  score.value = 0;
};

const togglePause = () => {
  if (isGameRunning.value) {
    isPaused.value = !isPaused.value;
    if (!isPaused.value) {
      gameLoop();
    }
  }
};

const changeDirection = (e) => {
  if (!isGameRunning.value || isPaused.value) return;

  switch (e.key) {
    case "ArrowUp":
      if (direction.value.y !== 1) direction.value = { x: 0, y: -1 };
      break;
    case "ArrowDown":
      if (direction.value.y !== -1) direction.value = { x: 0, y: 1 };
      break;
    case "ArrowLeft":
      if (direction.value.x !== 1) direction.value = { x: -1, y: 0 };
      break;
    case "ArrowRight":
      if (direction.value.x !== -1) direction.value = { x: 1, y: 0 };
      break;
  }
};

const changeDirectionMobile = (dir) => {
  if (!isGameRunning.value || isPaused.value) return;

  if (dir === "up" && direction.value.y !== 1)
    direction.value = { x: 0, y: -1 };
  if (dir === "down" && direction.value.y !== -1)
    direction.value = { x: 0, y: 1 };
  if (dir === "left" && direction.value.x !== 1)
    direction.value = { x: -1, y: 0 };
  if (dir === "right" && direction.value.x !== -1)
    direction.value = { x: 1, y: 0 };
};

const gameLoop = (timestamp) => {
  if (!isPaused.value && isGameRunning.value) {
    if (!lastUpdateTime || timestamp - lastUpdateTime >= speed.value) {
      moveSnake();
      adjustSpeed();
      lastUpdateTime = timestamp;
    }
    gameInterval = requestAnimationFrame(gameLoop);
  }
};

const moveSnake = () => {
  let newHeadX = snake.value[0].x + direction.value.x;
  let newHeadY = snake.value[0].y + direction.value.y;

  if (newHeadX >= boardCols.value) {
    newHeadX = 0;
  } else if (newHeadX < 0) {
    newHeadX = boardCols.value - 1;
  }

  if (newHeadY >= boardRows.value) {
    newHeadY = 0;
  } else if (newHeadY < 0) {
    newHeadY = boardRows.value - 1;
  }

  if (
    snake.value.some(
      (segment, index) =>
        index > 0 && segment.x === newHeadX && segment.y === newHeadY
    )
  ) {
    endGame();
    showGameOver.value = true;
    return;
  }

  const newHead = createSegment(newHeadX, newHeadY);
  snake.value.unshift(newHead);

  if (newHeadX === food.value.x && newHeadY === food.value.y) {
    addFlashEffect(newHead.id, "added");
    score.value += 10;
    growthCounter += 1;
    placeFood();
  } else if (
    goldenApple.value &&
    newHeadX === goldenApple.value.x &&
    newHeadY === goldenApple.value.y
  ) {
    addFlashEffect(newHead.id, "added");
    score.value += 30;
    growthCounter += 3;
    goldenApple.value = null;
  } else if (
    orangeTriangle.value &&
    newHeadX === orangeTriangle.value.x &&
    newHeadY === orangeTriangle.value.y
  ) {
    score.value = Math.max(0, score.value - 5);
    if (snake.value.length > 1) {
      const removedSegment = snake.value.pop();
      if (removedSegment) {
        addFlashEffect(removedSegment.id, "removed");
      }
    }
    orangeTriangle.value = null;
  } else if (growthCounter <= 0) {
    snake.value.pop();
  } else {
    growthCounter--;
  }
};

const adjustSpeed = () => {
  const newSpeed = Math.max(
    minSpeed,
    baseSpeed - Math.floor(score.value / 50) * 10
  );
  speed.value = newSpeed;
};

const repositionItemsIfOutOfBounds = () => {
  const outOfBounds = (item) =>
    item && (item.x >= boardCols.value || item.y >= boardRows.value);

  if (
    outOfBounds(food.value) ||
    outOfBounds(orangeTriangle.value) ||
    outOfBounds(goldenApple.value)
  ) {
    placeFood();
  }
};

const updateBoardSize = () => {
  const container = containerRef.value;
  if (!container) return;

  const viewportHeight = window.innerHeight;

  const titleHeight = document.querySelector("h1")?.offsetHeight || 0;
  const scoreHeight = container.querySelector(".mb-2")?.offsetHeight || 0;
  const controlsHeight =
    container.querySelector(".flex.gap-2")?.offsetHeight || 0;
  const mobileControlsHeight =
    window.innerWidth < 768
      ? container.querySelector(".flex.flex-col.items-center.mt-4")
          ?.offsetHeight || 0
      : 0;

  const availableHeight =
    viewportHeight -
    titleHeight -
    scoreHeight -
    controlsHeight -
    mobileControlsHeight -
    80;

  const availableWidth = container.offsetWidth;

  const cellsWide = Math.floor(availableWidth / cellSize);
  const cellsHigh = Math.floor(availableHeight / cellSize);

  boardCols.value = Math.min(30, Math.max(4, cellsWide));
  boardRows.value = Math.min(30, Math.max(4, cellsHigh));

  repositionItemsIfOutOfBounds();
};

const boardStyle = computed(() => {
  return {
    width: `${boardCols.value * cellSize}px`,
    height: `${boardRows.value * cellSize}px`,
  };
});

const getSegmentStyle = (segment) => {
  return `left: ${segment.x * cellSize}px; top: ${
    segment.y * cellSize
  }px; width: ${cellSize}px; height: ${cellSize}px;`;
};

onMounted(async () => {
  await preloadAssets();
  window.addEventListener("keydown", changeDirection);
  window.addEventListener("resize", updateBoardSize);

  const section = containerRef.value?.closest("section");
  if (section) {
    const resizeObserver = new ResizeObserver(() => {
      requestAnimationFrame(() => {
        updateBoardSize();
      });
    });
    resizeObserver.observe(section);
  }

  await nextTick();
  updateBoardSize();

  setTimeout(() => {
    updateBoardSize();
  }, 100);
});

onBeforeUnmount(() => {
  window.removeEventListener("keydown", changeDirection);
  window.removeEventListener("resize", updateBoardSize);
  if (gameInterval) cancelAnimationFrame(gameInterval);
  flashTimeouts.value.forEach((timeout) => clearTimeout(timeout));
  flashTimeouts.value.clear();

  const section = containerRef.value?.closest("section");
  if (section) {
    resizeObserver.disconnect();
  }
});
</script>

<style scoped>
body {
  margin: 0;
}

.corner-top-right img {
  width: 23px !important;
  height: 23px !important;
  position: relative;
  top: 10px;
  right: 2px;
}

.corner-top-left img {
  width: 23px !important;
  height: 23px !important;
  position: relative;
  top: 9px;
  left: 9px;
}

.corner-bottom-right img {
  width: 23px !important;
  height: 23px !important;
  position: relative;
  bottom: 1.6px;
  right: 1.6px;
}

.corner-bottom-left img {
  width: 23px !important;
  height: 23px !important;
  position: relative;
  bottom: 2px;
  left: 9px;
}

.tail-up img {
  width: 24px !important;
  height: 24px !important;
  position: relative;
  left: 3px;
}

.tail-down img {
  width: 24px !important;
  height: 24px !important;
  position: relative;
  left: 3px;
  top: 6px;
}

.tail-left img {
  width: 24px !important;
  height: 24px !important;
  position: relative;
  top: 3px;
}

.tail-right img {
  width: 24px !important;
  height: 24px !important;
  position: relative;
  top: 3px;
  left: 6px;
}

.head-up img {
  width: 30px !important;
  height: 30px !important;
  position: relative;
}

.head-down img {
  width: 30px !important;
  height: 30px !important;
  position: relative;
}

.head-left img {
  width: 30px !important;
  height: 30px !important;
  position: relative;
  bottom: 1px;
}

.head-right img {
  width: 30px !important;
  height: 30px !important;
  position: relative;
  bottom: 1px;
}

.straight-vertical img {
  width: 30px !important;
  height: 30px !important;
  position: relative;
}

.straight-horizontal img {
  width: 30px !important;
  height: 30px !important;
  position: relative;
}
</style>
