---
layout: page
title: 채용 트렌드
permalink: /stats/
---

<style>
  .stats-container { max-width: 820px; margin: 0 auto; }
  .chart-section { margin-bottom: 52px; }
  .chart-section h2 { font-size: 1.15rem; margin-bottom: 16px; border-bottom: 2px solid #e0e0e0; padding-bottom: 8px; }
  .bar-wrap { display: flex; align-items: center; margin-bottom: 10px; }
  .bar-label { width: 180px; font-size: 0.88rem; text-align: right; padding-right: 12px; flex-shrink: 0; color: #333; }
  .bar-track { flex: 1; background: #f0f0f0; border-radius: 4px; height: 22px; }
  .bar-fill { height: 100%; border-radius: 4px; transition: width 0.4s; }
  .bar-fill.blue  { background: #4a90e2; }
  .bar-fill.teal  { background: #36b0a0; }
  .bar-fill.purple{ background: #8e6bbf; }
  .bar-count { margin-left: 10px; font-size: 0.85rem; color: #555; width: 30px; }
  .role-tabs { display: flex; flex-wrap: wrap; gap: 8px; margin-bottom: 20px; }
  .role-tab { padding: 6px 14px; border: 1px solid #ccc; border-radius: 20px; cursor: pointer; font-size: 0.85rem; background: #fff; }
  .role-tab.active { background: #4a90e2; color: #fff; border-color: #4a90e2; }
  .no-data { color: #999; font-size: 0.9rem; }

  /* 경력 도넛 차트 */
  .donut-wrap { display: flex; align-items: center; gap: 40px; flex-wrap: wrap; }
  .donut-legend { display: flex; flex-direction: column; gap: 10px; }
  .legend-item { display: flex; align-items: center; gap: 8px; font-size: 0.9rem; }
  .legend-dot { width: 14px; height: 14px; border-radius: 50%; flex-shrink: 0; }
</style>

<div class="stats-container">

  <div class="chart-section">
    <h2>🔧 자주 요구되는 기술 스택 TOP 10</h2>
    <div id="skill-chart"><p class="no-data">데이터를 불러오는 중...</p></div>
  </div>

  <div class="chart-section">
    <h2>💼 직무별 자주 나오는 키워드</h2>
    <div class="role-tabs" id="role-tabs"></div>
    <div id="role-chart"><p class="no-data">직무를 선택해주세요.</p></div>
  </div>

  <div class="chart-section">
    <h2>🔗 자주 함께 요구되는 스킬 조합 TOP 10</h2>
    <div id="pair-chart"><p class="no-data">데이터를 불러오는 중...</p></div>
  </div>

  <div class="chart-section">
    <h2>🎯 경력 수준 분포</h2>
    <div class="donut-wrap">
      <canvas id="level-donut" width="200" height="200"></canvas>
      <div class="donut-legend" id="level-legend"></div>
    </div>
  </div>

</div>

<script>
const COLORS = ['#4a90e2','#36b0a0','#f5a623','#8e6bbf','#e25c5c','#50c878'];

fetch('/assets/stats.json')
  .then(res => res.json())
  .then(data => {
    renderSkillChart(data.skillCount || {});
    renderRoleTabs(data.roleSkillMap || {});
    renderPairChart(data.skillPairs || {});
    renderLevelDonut(data.levelCount || {});
  })
  .catch(() => {
    ['skill-chart','pair-chart'].forEach(id => {
      document.getElementById(id).innerHTML = '<p class="no-data">아직 데이터가 없어요. 공고를 먼저 추가해주세요!</p>';
    });
  });

function makeBars(skillMap, containerId, colorClass, topN) {
  const container = document.getElementById(containerId);
  const sorted = Object.entries(skillMap).sort((a, b) => b[1] - a[1]).slice(0, topN);
  if (sorted.length === 0) {
    container.innerHTML = '<p class="no-data">데이터가 없어요.</p>';
    return;
  }
  const max = sorted[0][1];
  container.innerHTML = sorted.map(([label, count]) => `
    <div class="bar-wrap">
      <span class="bar-label">${label}</span>
      <div class="bar-track">
        <div class="bar-fill ${colorClass}" style="width:${(count/max*100).toFixed(1)}%"></div>
      </div>
      <span class="bar-count">${count}</span>
    </div>
  `).join('');
}

function renderSkillChart(skillCount) {
  makeBars(skillCount, 'skill-chart', 'blue', 10);
}

function renderPairChart(skillPairs) {
  makeBars(skillPairs, 'pair-chart', 'teal', 10);
}

function renderRoleTabs(roleSkillMap) {
  const tabs = document.getElementById('role-tabs');
  const roles = Object.keys(roleSkillMap);
  if (roles.length === 0) {
    tabs.innerHTML = '<p class="no-data">아직 직무 데이터가 없어요.</p>';
    return;
  }
  tabs.innerHTML = roles.map((role, i) => `
    <button class="role-tab ${i === 0 ? 'active' : ''}" onclick="selectRole(this, '${role}')">${role}</button>
  `).join('');
  window._roleSkillMap = roleSkillMap;
  makeBars(roleSkillMap[roles[0]], 'role-chart', 'purple', 10);
}

function selectRole(btn, role) {
  document.querySelectorAll('.role-tab').forEach(b => b.classList.remove('active'));
  btn.classList.add('active');
  makeBars(window._roleSkillMap[role], 'role-chart', 'purple', 10);
}

function renderLevelDonut(levelCount) {
  const canvas = document.getElementById('level-donut');
  const legend = document.getElementById('level-legend');
  const entries = Object.entries(levelCount).sort((a, b) => b[1] - a[1]);

  if (entries.length === 0) {
    canvas.style.display = 'none';
    legend.innerHTML = '<p class="no-data">데이터가 없어요.</p>';
    return;
  }

  const total = entries.reduce((s, [, v]) => s + v, 0);
  const ctx = canvas.getContext('2d');
  const cx = 100, cy = 100, r = 80, inner = 48;
  let angle = -Math.PI / 2;

  entries.forEach(([label, count], i) => {
    const slice = (count / total) * 2 * Math.PI;
    ctx.beginPath();
    ctx.moveTo(cx, cy);
    ctx.arc(cx, cy, r, angle, angle + slice);
    ctx.closePath();
    ctx.fillStyle = COLORS[i % COLORS.length];
    ctx.fill();
    angle += slice;
  });

  // 가운데 구멍 (도넛 효과)
  ctx.beginPath();
  ctx.arc(cx, cy, inner, 0, 2 * Math.PI);
  ctx.fillStyle = '#fff';
  ctx.fill();

  // 가운데 총 공고수 텍스트
  ctx.fillStyle = '#333';
  ctx.font = 'bold 22px sans-serif';
  ctx.textAlign = 'center';
  ctx.textBaseline = 'middle';
  ctx.fillText(total, cx, cy - 8);
  ctx.font = '12px sans-serif';
  ctx.fillStyle = '#888';
  ctx.fillText('공고', cx, cy + 12);

  // 범례
  legend.innerHTML = entries.map(([label, count], i) => `
    <div class="legend-item">
      <div class="legend-dot" style="background:${COLORS[i % COLORS.length]}"></div>
      <span>${label} <strong>${count}개</strong> (${Math.round(count/total*100)}%)</span>
    </div>
  `).join('');
}
</script>