<script setup>
import { ref, reactive, onMounted } from "vue";

// 棋盘大小
const BOARD_SIZE = 15;

// 游戏状态
const gameState = reactive({
  board: Array(BOARD_SIZE)
    .fill()
    .map(() => Array(BOARD_SIZE).fill(0)),
  currentPlayer: 1, // 1代表黑棋，2代表白棋
  gameOver: false,
  winner: 0, // 0无胜者，1黑棋胜，2白棋胜
  lastMove: { x: -1, y: -1 },
});

// 棋盘尺寸
const boardSize = ref(0);
const cellSize = ref(0);

// 初始化棋盘尺寸
onMounted(() => {
  const systemInfo = uni.getSystemInfoSync();
  boardSize.value = Math.min(
    systemInfo.windowWidth * 0.9,
    systemInfo.windowHeight * 0.7
  );
  cellSize.value = boardSize.value / BOARD_SIZE;
});

// 落子
const placeStone = (row, col) => {
  // 如果游戏已结束或该位置已有棋子，则不允许落子
  if (gameState.gameOver || gameState.board[row][col] !== 0) {
    return;
  }

  // 落子
  gameState.board[row][col] = gameState.currentPlayer;
  gameState.lastMove = { x: row, y: col };

  // 检查是否获胜
  if (checkWin(row, col)) {
    gameState.gameOver = true;
    gameState.winner = gameState.currentPlayer;
    uni.showToast({
      title: gameState.currentPlayer === 1 ? "黑棋获胜！" : "白棋获胜！",
      icon: "success",
    });
    return;
  }

  // 检查是否平局
  if (checkDraw()) {
    gameState.gameOver = true;
    uni.showToast({
      title: "平局！",
      icon: "none",
    });
    return;
  }

  // 切换玩家
  gameState.currentPlayer = gameState.currentPlayer === 1 ? 2 : 1;
};

// 检查是否获胜
const checkWin = (row, col) => {
  const directions = [
    [1, 0], // 水平
    [0, 1], // 垂直
    [1, 1], // 右下对角线
    [1, -1], // 右上对角线
  ];

  const player = gameState.board[row][col];

  for (const [dx, dy] of directions) {
    let count = 1; // 当前位置已经有一个棋子

    // 正向检查
    for (let i = 1; i < 5; i++) {
      const newRow = row + i * dx;
      const newCol = col + i * dy;

      if (
        newRow < 0 ||
        newRow >= BOARD_SIZE ||
        newCol < 0 ||
        newCol >= BOARD_SIZE ||
        gameState.board[newRow][newCol] !== player
      ) {
        break;
      }

      count++;
    }

    // 反向检查
    for (let i = 1; i < 5; i++) {
      const newRow = row - i * dx;
      const newCol = col - i * dy;

      if (
        newRow < 0 ||
        newRow >= BOARD_SIZE ||
        newCol < 0 ||
        newCol >= BOARD_SIZE ||
        gameState.board[newRow][newCol] !== player
      ) {
        break;
      }

      count++;
    }

    if (count >= 5) {
      return true;
    }
  }

  return false;
};

// 检查是否平局
const checkDraw = () => {
  for (let i = 0; i < BOARD_SIZE; i++) {
    for (let j = 0; j < BOARD_SIZE; j++) {
      if (gameState.board[i][j] === 0) {
        return false; // 还有空位，不是平局
      }
    }
  }
  return true; // 棋盘已满，平局
};

// 重新开始游戏
const restartGame = () => {
  gameState.board = Array(BOARD_SIZE)
    .fill()
    .map(() => Array(BOARD_SIZE).fill(0));
  gameState.currentPlayer = 1;
  gameState.gameOver = false;
  gameState.winner = 0;
  gameState.lastMove = { x: -1, y: -1 };
};
</script>

<template>
  <view class="gobang-container">
    <view class="game-info">
      <text class="game-status">{{
        gameState.gameOver
          ? gameState.winner === 0
            ? "平局！"
            : gameState.winner === 1
            ? "黑棋获胜！"
            : "白棋获胜！"
          : gameState.currentPlayer === 1
          ? "黑棋回合"
          : "白棋回合"
      }}</text>
    </view>

    <view
      class="board"
      :style="{ width: boardSize + 'px', height: boardSize + 'px' }"
    >
      <!-- 棋盘背景 -->
      <view class="board-grid">
        <view
          v-for="i in BOARD_SIZE - 1"
          :key="'h' + i"
          class="grid-line horizontal"
          :style="{ top: i * cellSize + 'px', width: boardSize + 'px' }"
        ></view>
        <view
          v-for="i in BOARD_SIZE - 1"
          :key="'v' + i"
          class="grid-line vertical"
          :style="{ left: i * cellSize + 'px', height: boardSize + 'px' }"
        ></view>
      </view>

      <!-- 棋盘上的标记点 -->
      <view
        v-for="(point, index) in [
          { x: 3, y: 3 },
          { x: 3, y: 11 },
          { x: 7, y: 7 },
          { x: 11, y: 3 },
          { x: 11, y: 11 },
        ]"
        :key="index"
        class="mark-point"
        :style="{
          left: point.y * cellSize + 'px',
          top: point.x * cellSize + 'px',
        }"
      ></view>

      <!-- 棋子 -->
      <view v-for="row in BOARD_SIZE" :key="'row' + row" class="board-row">
        <view
          v-for="col in BOARD_SIZE"
          :key="'cell' + row + '-' + col"
          class="board-cell"
          :style="{ width: cellSize + 'px', height: cellSize + 'px' }"
          @tap="placeStone(row - 1, col - 1)"
        >
          <view
            v-if="gameState.board[row - 1][col - 1] !== 0"
            class="stone"
            :class="{
              black: gameState.board[row - 1][col - 1] === 1,
              white: gameState.board[row - 1][col - 1] === 2,
              'last-move':
                gameState.lastMove.x === row - 1 &&
                gameState.lastMove.y === col - 1,
            }"
          ></view>
        </view>
      </view>
    </view>

    <view class="controls">
      <button class="restart-btn" @tap="restartGame">重新开始</button>
    </view>
  </view>
</template>

<style>
.gobang-container {
  display: flex;
  flex-direction: column;
  align-items: center;
  padding: 20px;
}

.game-info {
  margin-bottom: 20px;
}

.game-status {
  font-size: 18px;
  font-weight: bold;
}

.board {
  position: relative;
  background-color: #e6c88c; /* 棋盘背景色 */
  border: 1px solid #333;
}

.board-grid {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
}

.grid-line {
  position: absolute;
  background-color: #333;
}

.horizontal {
  height: 1px;
}

.vertical {
  width: 1px;
}

.mark-point {
  position: absolute;
  width: 6px;
  height: 6px;
  border-radius: 50%;
  background-color: #333;
  transform: translate(-3px, -3px);
}

.board-row {
  display: flex;
}

.board-cell {
  position: relative;
  display: flex;
  justify-content: center;
  align-items: center;
}

.stone {
  width: 80%;
  height: 80%;
  border-radius: 50%;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.3);
}

.black {
  background-color: #000;
}

.white {
  background-color: #fff;
}

.last-move {
  box-shadow: 0 0 5px 2px rgba(255, 0, 0, 0.7);
}

.controls {
  margin-top: 20px;
}

.restart-btn {
  background-color: #4caf50;
  color: white;
  padding: 10px 20px;
  border-radius: 5px;
  font-size: 16px;
}
</style>
