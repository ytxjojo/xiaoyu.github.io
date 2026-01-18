---
layout: default
title: 狗戏团
permalink: /
---

<!-- 核心HTML结构 -->
<div class="game-container">
  <!-- 昵称输入弹窗 -->
  <div id="nickname-modal" class="modal">
    <div class="modal-content">
      <h3>请输入你的昵称</h3>
      <input type="text" id="nickname-input" placeholder="例如：张三" required>
      <button id="confirm-nickname">确认</button>
    </div>
  </div>

  <!-- 游戏配置区（管理员视角，可隐藏/显示） -->
  <div class="game-config">
    <h4>游戏配置</h4>
    <div class="config-item">
      <label>游戏名称：</label>
      <input type="text" id="game-name" placeholder="输入游戏名">
      <label>上限人数：</label>
      <input type="number" id="game-limit" min="1" value="5">
      <button id="add-game">添加游戏</button>
    </div>
    <div id="game-list" class="game-list"></div>
  </div>

  <!-- 转盘区域 -->
  <div class="wheel-container">
    <div id="wheel" class="wheel"></div>
    <button id="spin-wheel" disabled>开始转盘</button>
  </div>

  <!-- 分组结果区 -->
  <div class="group-result">
    <h4>当前分组</h4>
    <div id="current-group" class="current-group"></div>
    <button id="leave-group" disabled>下车（退出当前组）</button>
    <button id="send-group" disabled>发车（清空当前组）</button>
  </div>

  <!-- 所有游戏分组列表 -->
  <div class="all-groups">
    <h4>所有游戏分组</h4>
    <div id="all-groups-list"></div>
  </div>
</div>

<!-- 引入JS和CSS -->
<style>
/* 核心样式（也可抽离到assets/css/style.scss） */
.modal {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background: rgba(0,0,0,0.7);
  display: flex;
  justify-content: center;
  align-items: center;
  z-index: 1000;
}
.modal-content {
  background: white;
  padding: 2rem;
  border-radius: 8px;
  text-align: center;
}
#nickname-input {
  padding: 0.5rem;
  margin: 1rem 0;
  width: 200px;
}
.game-container {
  max-width: 1200px;
  margin: 2rem auto;
  padding: 0 1rem;
}
.game-config {
  margin-bottom: 2rem;
  padding: 1rem;
  border: 1px solid #eee;
  border-radius: 8px;
}
.config-item {
  margin: 1rem 0;
  display: flex;
  align-items: center;
  gap: 0.5rem;
}
.game-list {
  margin-top: 1rem;
  padding: 0.5rem;
  border: 1px solid #ddd;
  border-radius: 4px;
}
.wheel-container {
  margin: 2rem 0;
  text-align: center;
}
.wheel {
  width: 300px;
  height: 300px;
  border-radius: 50%;
  border: 5px solid #333;
  margin: 0 auto 1rem;
  position: relative;
  overflow: hidden;
  transition: transform 3s cubic-bezier(0.2, 0.8, 0.2, 1);
}
.wheel-segment {
  position: absolute;
  width: 50%;
  height: 50%;
  transform-origin: bottom right;
  display: flex;
  justify-content: center;
  align-items: center;
  color: white;
  font-weight: bold;
}
#spin-wheel {
  padding: 0.5rem 2rem;
  font-size: 1rem;
  background: #4CAF50;
  color: white;
  border: none;
  border-radius: 4px;
  cursor: pointer;
}
#spin-wheel:disabled {
  background: #ccc;
  cursor: not-allowed;
}
.group-result, .all-groups {
  margin: 2rem 0;
  padding: 1rem;
  border: 1px solid #eee;
  border-radius: 8px;
}
.current-group {
  margin: 1rem 0;
  padding: 0.5rem;
  border: 1px solid #ddd;
  border-radius: 4px;
}
#leave-group, #send-group {
  margin: 0 0.5rem;
  padding: 0.5rem 1rem;
  border: none;
  border-radius: 4px;
  cursor: pointer;
}
#leave-group {
  background: #ff9800;
  color: white;
}
#send-group {
  background: #f44336;
  color: white;
}
#leave-group:disabled, #send-group:disabled {
  background: #ccc;
  cursor: not-allowed;
}
</style>
