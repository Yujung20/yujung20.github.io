---
layout: page
title: 스킬 통계
permalink: /stats/
---

<style>
  .stats-container { max-width: 800px; margin: 0 auto; }
  .chart-section { margin-bottom: 48px; }
  .chart-section h2 { font-size: 1.2rem; margin-bottom: 16px; border-bottom: 2px solid #e0e0e0; padding-bottom: 8px; }
  .bar-wrap { display: flex; align-items: center; margin-bottom: 10px; }
  .bar-label { width: 160px; font-size: 0.9rem; text-align: right; padding-right: 12px; flex-shrink: 0; }
  .bar-track { flex: 1; background: #f0f0f0; border-radius: 4px; height: 22px; }
  .bar-fill { height: 100%; border-radius: 4px; background: #4a90e2; transition: width 0.4s; }
  .bar-count { margin-left: 10px; font-size: 0.85rem; color: #555; width: 30px; }
  .role-tabs { display: flex; flex-wrap: wrap; gap: 8px; margin-bottom: 20px; }
  .role-tab { padding: 6px 14px; border: 1px solid #ccc; border-radius: 20px; cursor: pointer; font-size: 0.85rem; background: #fff; }
  .role-tab.active { background: #4a90e2; color: #fff; border-color: #4a90e2; }
  .no-data { color: #999; font-size: 0.9rem; }
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

</div>

<script>
fetch('/assets/stats.json')
  .then(res => res.json())
  .then(data => {
    renderSkillChart(data.skillCount || {});
    renderRoleTabs(data.roleSkillMap || {});
  })
  .catch(() => {
    document.getElementById('skill-chart').innerHTML = '<p class="no-data">아직 데이터가 없어요. 공고를 먼저 추가해주세요!</p>';
  });

function makeBars(skillMap, containerId, topN = 10) {
  const container = document.getElementById(containerId);
  const sorted = Object.entries(skillMap).sort((a, b) => b[1] - a[1]).slice(0, topN);
  if (sorted.length === 0) {
    container.innerHTML = '<p class="no-data">해당 직무의 스킬 데이터가 없어요.</p>';
    return;
  }
  const max = sorted[0][1];
  container.innerHTML = sorted.map(([skill, count]) => `
    <div class="bar-wrap">
      <span class="bar-label">${skill}</span>
      <div class="bar-track">
        <div class="bar-fill" style="width:${(count/max*100).toFixed(1)}%"></div>
      </div>
      <span class="bar-count">${count}</span>
    </div>
  `).join('');
}

function renderSkillChart(skillCount) {
  makeBars(skillCount, 'skill-chart', 10);
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
  makeBars(roleSkillMap[roles[0]], 'role-chart', 10);
}

function selectRole(btn, role) {
  document.querySelectorAll('.role-tab').forEach(b => b.classList.remove('active'));
  btn.classList.add('active');
  makeBars(window._roleSkillMap[role], 'role-chart', 10);
}
</script>
