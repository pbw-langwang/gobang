<script setup>
import GobangGame from "@/components/GobangGame.vue";
</script>

<template>
  <view class="root-container">
    <view class="title">
      <text>五子棋游戏</text>
    </view>
    <GobangGame />
  </view>
</template>

<style scoped>
.root-container {
  padding: 2rem 1rem;
  text-align: center;
  display: flex;
  flex-direction: column;
  align-items: center;
}

.title {
  font-size: 24px;
  font-weight: bold;
  margin-bottom: 20px;
}
</style>
