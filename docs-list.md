<div id="docs-list-app"></div>

<style>
.dl-wrap {
  max-width: 1000px;
  margin: 0 auto;
  padding: 0 0 60px;
  font-family: Arial, "Noto Sans KR", sans-serif;
}

/* 헤더 */
.dl-header {
  margin-bottom: 32px;
  padding-bottom: 16px;
  border-bottom: 2px solid #eef4ff;
}
.dl-header h2 {
  font-size: 26px;
  font-weight: 700;
  color: #0f172a;
  margin: 0 0 8px;
}
.dl-header p {
  color: #64748b;
  font-size: 14px;
  margin: 0;
}

/* 필터 */
.dl-filter {
  display: flex;
  gap: 8px;
  flex-wrap: wrap;
  margin-bottom: 28px;
}
.dl-filter-btn {
  padding: 7px 18px;
  border: 1.5px solid #1A6EBD;
  border-radius: 20px;
  background: #fff;
  color: #1A6EBD;
  font-size: 13px;
  cursor: pointer;
  transition: all 0.2s;
}
.dl-filter-btn.active,
.dl-filter-btn:hover {
  background: #1A6EBD;
  color: #fff;
}

/* 카드 그리드 */
.dl-grid {
  display: grid;
  grid-template-columns: repeat(2, 1fr);
  gap: 20px;
}

.dl-card {
  display: flex;
  border: 1px solid #e2e8f0;
  border-radius: 12px;
  overflow: hidden;
  text-decoration: none;
  color: inherit;
  background: #fff;
  transition: all 0.25s;
}
.dl-card:hover {
  box-shadow: 0 6px 24px rgba(0,0,0,0.09);
  transform: translateY(-2px);
  border-color: #1A6EBD;
  text-decoration: none;
  color: inherit;
}

/* 썸네일 */
.dl-thumb {
  width: 120px;
  min-width: 120px;
  background: linear-gradient(135deg, #eef4ff, #dce8ff);
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 32px;
}
.dl-thumb img {
  width: 120px;
  height: 100%;
  object-fit: cover;
  display: block;
}

/* 카드 본문 */
.dl-body {
  padding: 16px 18px;
  display: flex;
  flex-direction: column;
  justify-content: space-between;
  flex: 1;
  min-width: 0;
}
.dl-category {
  font-size: 10px;
  font-weight: 700;
  letter-spacing: 1.5px;
  text-transform: uppercase;
  color: #1A6EBD;
  margin-bottom: 6px;
}
.dl-title {
  font-size: 14px;
  font-weight: 700;
  color: #0f172a;
  line-height: 1.45;
  margin-bottom: 8px;
  display: -webkit-box;
  -webkit-line-clamp: 2;
  -webkit-box-orient: vertical;
  overflow: hidden;
}
.dl-summary {
  font-size: 12px;
  color: #64748b;
  line-height: 1.6;
  display: -webkit-box;
  -webkit-line-clamp: 2;
  -webkit-box-orient: vertical;
  overflow: hidden;
  margin-bottom: 10px;
}
.dl-footer {
  display: flex;
  justify-content: space-between;
  align-items: center;
  font-size: 11px;
  color: #94a3b8;
}
.dl-more {
  color: #1A6EBD;
  font-weight: 700;
  font-size: 11px;
}

/* 총 문서 수 */
.dl-count {
  font-size: 13px;
  color: #94a3b8;
  margin-bottom: 16px;
}
.dl-count span {
  font-weight: 700;
  color: #1A6EBD;
}

/* 로딩 / 에러 */
.dl-loading {
  text-align: center;
  padding: 60px;
  color: #aaa;
}

@media (max-width: 640px) {
  .dl-grid {
    grid-template-columns: 1fr;
  }
  .dl-thumb {
    width: 90px;
    min-width: 90px;
  }
}
</style>

<script>
(function() {
  const app = document.getElementById('docs-list-app');

  app.innerHTML = `
    <div class="dl-wrap">
      <div class="dl-header">
        <h2>📋 전체 문서 목록</h2>
        <p>senskeylab 기술 문서 및 인사이트 전체 목록입니다.</p>
      </div>
      <div class="dl-filter" id="dl-filter"></div>
      <div class="dl-count" id="dl-count"></div>
      <div class="dl-grid" id="dl-grid">
        <div class="dl-loading">문서를 불러오는 중...</div>
      </div>
    </div>
  `;

  let allDocs = [];

  fetch('https://docs.senskeylab.com/index.json')
    .then(r => r.json())
    .then(data => {
      allDocs = data;
      buildFilter(data);
      renderCards(data);
    })
    .catch(() => {
      document.getElementById('dl-grid').innerHTML =
        '<div class="dl-loading">문서를 불러올 수 없습니다.</div>';
    });

  function buildFilter(data) {
    const categories = ['전체', ...new Set(data.map(d => d.category))];
    const filterEl = document.getElementById('dl-filter');
    filterEl.innerHTML = categories.map(cat =>
      `<button class="dl-filter-btn ${cat === '전체' ? 'active' : ''}"
        onclick="dlFilter('${cat}', this)">${cat}</button>`
    ).join('');
  }

  window.dlFilter = function(category, btn) {
    document.querySelectorAll('.dl-filter-btn').forEach(b => b.classList.remove('active'));
    btn.classList.add('active');
    const filtered = category === '전체'
      ? allDocs
      : allDocs.filter(d => d.category === category);
    renderCards(filtered);
  };

  function renderCards(data) {
    const countEl = document.getElementById('dl-count');
    const gridEl  = document.getElementById('dl-grid');

    countEl.innerHTML = `총 <span>${data.length}</span>개의 문서`;

    if (data.length === 0) {
      gridEl.innerHTML = '<div class="dl-loading">등록된 문서가 없습니다.</div>';
      return;
    }

    gridEl.innerHTML = data.map(item => `
      <a href="${item.url}" target="_blank" class="dl-card">
        <div class="dl-thumb">
          ${item.thumbnail
            ? `<img src="${item.thumbnail}" alt="${item.title}" loading="lazy">`
            : (item.icon || '📄')
          }
        </div>
        <div class="dl-body">
          <div>
            <div class="dl-category">${item.category}</div>
            <div class="dl-title">${item.title}</div>
            <div class="dl-summary">${item.summary || item.description || ''}</div>
          </div>
          <div class="dl-footer">
            <span>${item.date || ''}</span>
            <span class="dl-more">보기 →</span>
          </div>
        </div>
      </a>
    `).join('');
  }
})();
</script>
