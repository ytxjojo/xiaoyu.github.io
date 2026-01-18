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

<!-- 所有样式内嵌 -->
<style>
/* 基础样式 */
body {
  font-family: "Microsoft YaHei", sans-serif;
  margin: 0;
  padding: 0;
  background-color: #f5f5f5;
}

h1, h2, h3, h4, h5 {
  color: #333;
}

button {
  transition: background-color 0.3s;
}

button:hover:not(:disabled) {
  opacity: 0.9;
}

input {
  padding: 0.5rem;
  border: 1px solid #ddd;
  border-radius: 4px;
}

/* 弹窗样式 */
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

/* 游戏容器 */
.game-container {
  max-width: 1200px;
  margin: 2rem auto;
  padding: 0 1rem;
}

/* 游戏配置区 */
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

.game-list-item {
  margin: 0.5rem 0;
  padding: 0.5rem;
  background: #f0f0f0;
  border-radius: 4px;
  display: flex;
  justify-content: space-between;
  align-items: center;
}

/* 转盘区域 */
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

/* 分组结果区 */
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

/* 所有分组列表 */
.group-item {
  margin: 0.5rem 0;
  padding: 0.5rem;
  background: #f8f8f8;
  border-radius: 4px;
}

/* 移动端适配 */
@media (max-width: 768px) {
  .wheel {
    width: 200px;
    height: 200px;
  }
  .config-item {
    flex-direction: column;
    align-items: flex-start;
  }
}
</style>

<!-- 所有JS逻辑内嵌 -->
<script>
// 全局变量
let userNickname = "";
let gameList = []; // 游戏列表：[{name: '游戏1', limit: 5, members: []}]
let currentUserGroup = null; // 当前用户所在分组

// DOM加载完成后执行
document.addEventListener('DOMContentLoaded', function() {
  // 获取DOM元素
  const nicknameModal = document.getElementById('nickname-modal');
  const nicknameInput = document.getElementById('nickname-input');
  const confirmNicknameBtn = document.getElementById('confirm-nickname');
  const spinWheelBtn = document.getElementById('spin-wheel');
  const addGameBtn = document.getElementById('add-game');
  const leaveGroupBtn = document.getElementById('leave-group');
  const sendGroupBtn = document.getElementById('send-group');
  const gameNameInput = document.getElementById('game-name');
  const gameLimitInput = document.getElementById('game-limit');
  const gameListDom = document.getElementById('game-list');
  const currentGroupDom = document.getElementById('current-group');
  const allGroupsListDom = document.getElementById('all-groups-list');

  // 强制显示昵称弹窗
  nicknameModal.style.display = 'flex';

  // 昵称确认逻辑（增强关闭可靠性）
  function confirmNickname() {
    const nickname = nicknameInput.value.trim();
    if (!nickname) {
      alert('请输入有效的昵称！');
      return;
    }
    // 存储昵称
    userNickname = nickname;
    // 强制关闭弹窗（多重兜底）
    nicknameModal.style.display = 'none';
    nicknameModal.style.visibility = 'hidden';
    nicknameModal.style.opacity = '0';
    // 提示
    alert(`欢迎你，${nickname}！`);
    // 若有游戏则启用转盘
    if (gameList.length > 0) {
      spinWheelBtn.disabled = false;
    }
    updateAllGroups();
  }

  // 点击确认按钮
  confirmNicknameBtn.addEventListener('click', confirmNickname);
  // 按回车确认昵称
  nicknameInput.addEventListener('keydown', function(e) {
    if (e.key === 'Enter') {
      confirmNickname();
    }
  });

  // 添加游戏
  addGameBtn.addEventListener('click', function() {
    const gameName = gameNameInput.value.trim();
    const gameLimit = parseInt(gameLimitInput.value);
    if (!gameName || isNaN(gameLimit) || gameLimit < 1) {
      alert('请输入有效的游戏名称和人数上限！');
      return;
    }
    // 检查游戏名是否重复
    const isDuplicate = gameList.some(item => item.name === gameName);
    if (isDuplicate) {
      alert('该游戏名称已存在！');
      return;
    }
    // 添加游戏到列表
    gameList.push({
      name: gameName,
      limit: gameLimit,
      members: []
    });
    // 清空输入框
    gameNameInput.value = '';
    gameLimitInput.value = 5;
    // 更新视图
    renderGameList();
    renderWheel();
    // 启用转盘
    spinWheelBtn.disabled = false;
    updateAllGroups();
  });

  // 渲染游戏列表（配置区）
  function renderGameList() {
    gameListDom.innerHTML = '';
    gameList.forEach((game, index) => {
      const item = document.createElement('div');
      item.className = 'game-list-item';
      item.innerHTML = `
        <span>${game.name}（上限：${game.limit}人，当前：${game.members.length}人）</span>
        <button class="delete-game" data-index="${index}">删除</button>
      `;
      gameListDom.appendChild(item);
    });
    // 删除游戏事件
    document.querySelectorAll('.delete-game').forEach(btn => {
      btn.addEventListener('click', function() {
        const index = parseInt(this.dataset.index);
        // 若用户在被删除的组，清空当前组
        if (currentUserGroup === gameList[index]?.name) {
          leaveGroup();
        }
        // 删除游戏
        gameList.splice(index, 1);
        // 更新视图
        renderGameList();
        renderWheel();
        // 若游戏列表为空则禁用转盘
        if (gameList.length === 0) {
          spinWheelBtn.disabled = true;
        }
        updateAllGroups();
      });
    });
  }

  // 渲染转盘
  function renderWheel() {
    const wheel = document.getElementById('wheel');
    wheel.innerHTML = '';
    if (gameList.length === 0) {
      wheel.innerHTML = '<div style="line-height: 300px; color: #333;">暂无游戏</div>';
      return;
    }
    // 计算每个扇区的角度
    const anglePerSegment = 360 / gameList.length;
    const colors = ['#e74c3c', '#3498db', '#2ecc71', '#f1c40f', '#9b59b6', '#e67e22', '#1abc9c'];
    gameList.forEach((game, index) => {
      const segment = document.createElement('div');
      segment.className = 'wheel-segment';
      segment.style.transform = `rotate(${index * anglePerSegment}deg)`;
      segment.style.background = colors[index % colors.length];
      // 调整文字角度
      segment.style.transform += ` translate(-50%, -50%) rotate(${-index * anglePerSegment}deg)`;
      segment.innerHTML = game.name;
      wheel.appendChild(segment);
    });
  }

  // 转盘转动逻辑
  spinWheelBtn.addEventListener('click', function() {
    if (gameList.length === 0 || !userNickname) return;
    // 禁用转盘按钮
    spinWheelBtn.disabled = true;
    // 随机生成旋转角度（360*N + 随机扇区角度）
    const randomRotate = 360 * 5 + Math.random() * 360;
    const wheel = document.getElementById('wheel');
    wheel.style.transform = `rotate(${randomRotate}deg)`;
    // 旋转结束后计算结果
    setTimeout(() => {
      // 计算最终指向的扇区
      const anglePerSegment = 360 / gameList.length;
      const finalAngle = randomRotate % 360;
      const selectedIndex = Math.floor((360 - finalAngle) / anglePerSegment) % gameList.length;
      const selectedGame = gameList[selectedIndex];
      
      // 检查人数上限
      if (selectedGame.members.length >= selectedGame.limit) {
        alert(`【${selectedGame.name}】人数已达上限（${selectedGame.limit}人），无法加入！`);
        spinWheelBtn.disabled = false;
        return;
      }

      // 加入分组
      currentUserGroup = selectedGame.name;
      selectedGame.members.push(userNickname);
      // 更新当前分组显示
      currentGroupDom.innerHTML = `
        <p>游戏：${selectedGame.name}</p>
        <p>成员：${selectedGame.members.join('、')}</p>
        <p>剩余名额：${selectedGame.limit - selectedGame.members.length}</p>
      `;
      // 启用下车/发车按钮
      leaveGroupBtn.disabled = false;
      sendGroupBtn.disabled = false;
      spinWheelBtn.disabled = false;
      // 更新所有分组
      updateAllGroups();
      alert(`恭喜你抽到【${selectedGame.name}】！`);
    }, 3000); // 对应转盘过渡时间3s
  });

  // 下车（退出当前组）
  function leaveGroup() {
    if (!currentUserGroup) return;
    const game = gameList.find(item => item.name === currentUserGroup);
    if (!game) return;
    // 移除当前用户
    game.members = game.members.filter(member => member !== userNickname);
    // 清空当前组状态
    currentUserGroup = null;
    currentGroupDom.innerHTML = '<p>你尚未加入任何分组</p>';
    leaveGroupBtn.disabled = true;
    sendGroupBtn.disabled = true;
    // 更新所有分组
    updateAllGroups();
    alert('已退出当前分组！');
  }

  leaveGroupBtn.addEventListener('click', leaveGroup);

  // 发车（清空当前组）
  sendGroupBtn.addEventListener('click', function() {
    if (!currentUserGroup) return;
    if (!confirm(`确认发车？将清空【${currentUserGroup}】的所有成员！`)) return;
    const game = gameList.find(item => item.name === currentUserGroup);
    if (!game) return;
    // 清空成员
    game.members = [];
    // 清空当前组状态
    currentUserGroup = null;
    currentGroupDom.innerHTML = '<p>你尚未加入任何分组</p>';
    leaveGroupBtn.disabled = true;
    sendGroupBtn.disabled = true;
    // 更新所有分组
    updateAllGroups();
    alert(`【${currentUserGroup}】已发车，所有成员已清空！`);
  });

  // 更新所有分组列表
  function updateAllGroups() {
    allGroupsListDom.innerHTML = '';
    if (gameList.length === 0) {
      allGroupsListDom.innerHTML = '<p>暂无配置游戏</p>';
      return;
    }
    gameList.forEach(game => {
      const groupItem = document.createElement('div');
      groupItem.className = 'group-item';
      groupItem.innerHTML = `
        <h5>${game.name}</h5>
        <p>人数上限：${game.limit}</p>
        <p>当前成员：${game.members.length > 0 ? game.members.join('、') : '暂无'}</p>
        <p>剩余名额：${game.limit - game.members.length}</p>
      `;
      allGroupsListDom.appendChild(groupItem);
    });
  }

  // 初始化视图
  renderGameList();
  renderWheel();
  updateAllGroups();
  // 初始化当前分组显示
  currentGroupDom.innerHTML = '<p>你尚未加入任何分组</p>';
});
</script>
