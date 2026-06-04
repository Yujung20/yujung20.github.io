---
layout: page
title: 채용 트렌드
permalink: /stats/
---

<link href="https://fonts.googleapis.com/css2?family=Noto+Sans+KR:wght@400;500;700&family=Nanum+Gothic:wght@400;700&display=swap" rel="stylesheet">

<style>
  .stats-container { max-width: 780px; margin: 2.5rem auto; padding: 0 1.5rem; font-family: 'Nanum Gothic', sans-serif; }

  /* 페이지 대제목 — post.html 회사명(1.8rem, Noto Sans KR)에 맞춤 */
  .post-content h1, h1.post-title, .page-title, h1 {
    font-family: 'Noto Sans KR', sans-serif;
    font-size: 1.8rem;
    font-weight: 700;
    color: #1a1a1a;
  }
  .chart-section { margin-bottom: 52px; }

  /* 섹션 제목 — post.html의 h2와 동일 */
  .chart-section h2 {
    font-family: 'Nanum Gothic', sans-serif;
    font-size: 14px;
    font-weight: 700;
    color: #1a1a1a;
    border-left: 3px solid #0A66C2;
    border-bottom: none;
    padding-left: 10px;
    padding-bottom: 0;
    margin: 2rem 0 1.2rem;
    line-height: 1.4;
  }

  .bar-wrap { display: flex; align-items: center; margin-bottom: 10px; }
  .bar-label { width: 180px; font-size: 14px; font-family: 'Nanum Gothic', sans-serif; text-align: right; padding-right: 12px; flex-shrink: 0; color: #444; }
  .bar-track { flex: 1; background: #f0f0f0; border-radius: 4px; height: 22px; }
  .bar-fill { height: 100%; border-radius: 4px; transition: width 0.4s; }
  .bar-fill.blue   { background: #4A7A9B; }
  .bar-fill.teal   { background: #3D7A6A; }
  .bar-fill.purple { background: #5A5A8F; }
  .bar-count { margin-left: 10px; font-size: 13px; font-family: 'Nanum Gothic', sans-serif; color: #555; width: 30px; }

  .role-tabs { display: flex; flex-wrap: wrap; gap: 8px; margin-bottom: 20px; }
  .role-tab {
    padding: 6px 14px;
    border: 1px solid #ccc;
    border-radius: 20px;
    cursor: pointer;
    font-size: 13px;
    font-family: 'Nanum Gothic', sans-serif;
    background: #fff;
    color: #444;
  }
  .role-tab.active { background: #4A7A9B; color: #fff; border-color: #4A7A9B; }

  .no-data { color: #999; font-size: 14px; font-family: 'Nanum Gothic', sans-serif; }

  /* 경력 도넛 차트 */
  .donut-wrap { display: flex; align-items: center; gap: 40px; flex-wrap: wrap; }
  .donut-legend { display: flex; flex-direction: column; gap: 10px; }
  .legend-item { display: flex; align-items: center; gap: 8px; font-size: 14px; font-family: 'Nanum Gothic', sans-serif; color: #444; }
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
const COLORS = ['#4A7A9B','#3D7A6A','#5A5A8F','#7A6A4A','#7A4A4A','#6A7A4A'];

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
  ctx.font = "bold 22px 'Nanum Gothic', sans-serif";
  ctx.textAlign = 'center';
  ctx.textBaseline = 'middle';
  ctx.fillText(total, cx, cy - 8);
  ctx.font = "12px 'Nanum Gothic', sans-serif";
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