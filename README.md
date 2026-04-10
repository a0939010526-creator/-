<!DOCTYPE html>
<html lang="zh-TW">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>MPU 零件庫存管理</title>
<style>
*{box-sizing:border-box;margin:0;padding:0;}
body{font-family:-apple-system,BlinkMacSystemFont,"Segoe UI",Arial,sans-serif;background:#f5f5f3;color:#1a1a1a;font-size:15px;}
nav{background:#1a1a1a;padding:0 24px;display:flex;align-items:center;justify-content:space-between;height:52px;position:sticky;top:0;z-index:50;}
nav .logo{color:#fff;font-size:16px;font-weight:600;}
nav .nav-links{display:flex;gap:4px;}
nav button{background:transparent;border:none;color:#aaa;padding:8px 14px;border-radius:8px;cursor:pointer;font-size:13px;transition:all 0.15s;}
nav button:hover{background:#333;color:#fff;}
nav button.active{background:#333;color:#fff;}
.container{max-width:1200px;margin:0 auto;padding:24px 20px;}
.stats{display:grid;grid-template-columns:repeat(auto-fit,minmax(120px,1fr));gap:12px;margin-bottom:20px;}
.stat{background:#fff;border-radius:10px;border:1px solid #e8e8e8;padding:14px 16px;}
.stat-label{font-size:12px;color:#888;margin-bottom:6px;}
.stat-val{font-size:26px;font-weight:600;}
.toolbar{display:flex;gap:10px;flex-wrap:wrap;margin-bottom:14px;align-items:center;}
.toolbar input,.toolbar select{border:1px solid #ddd;background:#fff;color:#1a1a1a;border-radius:8px;padding:8px 12px;font-size:14px;outline:none;transition:border-color 0.15s;}
.toolbar input{flex:1;min-width:160px;}
.toolbar input:focus,.toolbar select:focus{border-color:#888;}
.tab-bar{display:flex;gap:0;border-bottom:2px solid #e8e8e8;margin-bottom:18px;}
.tab-btn{background:none;border:none;padding:10px 20px;font-size:14px;cursor:pointer;color:#888;border-bottom:2px solid transparent;margin-bottom:-2px;transition:all 0.15s;}
.tab-btn.active{color:#1a1a1a;font-weight:600;border-bottom:2px solid #1a1a1a;}
.tab-btn .badge-count{display:inline-block;background:#FCEBEB;color:#A32D2D;font-size:10px;font-weight:600;padding:1px 6px;border-radius:20px;margin-left:5px;}
.table-wrap{background:#fff;border-radius:12px;border:1px solid #e8e8e8;overflow:hidden;}
table{width:100%;border-collapse:collapse;font-size:13px;}
th{text-align:left;padding:10px 14px;color:#888;font-weight:500;border-bottom:1px solid #f0f0f0;font-size:12px;background:#fafafa;white-space:nowrap;}
td{padding:9px 14px;border-bottom:1px solid #f5f5f5;vertical-align:middle;}
tr:last-child td{border-bottom:none;}
tr:hover td{background:#fafafa;}
tr.row-expired td{background:#fff5f5!important;}
tr.row-expired:hover td{background:#ffe8e8!important;}
tr.row-notified td{background:#fffbf0!important;}
.mono{font-family:"SF Mono","Consolas",monospace;font-size:13px;font-weight:500;}
.tag{display:inline-block;padding:2px 8px;border-radius:20px;font-size:11px;font-weight:500;white-space:nowrap;}
.tag-mpu{background:#EEEDFE;color:#3C3489;}
.tag-asis{background:#E1F5EE;color:#0F6E56;}
.tag-asisb{background:#FAEEDA;color:#854F0B;}
.tag-series{background:#f0f0f0;color:#555;}
.locations{display:flex;flex-wrap:wrap;gap:4px;}
.badge{display:inline-block;padding:2px 10px;border-radius:20px;font-size:11px;font-weight:500;}
.lv-high{background:#EAF3DE;color:#3B6D11;}
.lv-low{background:#FAECE7;color:#993C1D;}
.lv-out{background:#FCEBEB;color:#A32D2D;}
.lv-na{background:#f0e8f8;color:#6b3fa0;}
.lv-select{border:1px solid #ddd;background:#fff;color:#1a1a1a;border-radius:8px;padding:3px 8px;font-size:12px;cursor:pointer;outline:none;}
.note-cell{max-width:110px;overflow:hidden;text-overflow:ellipsis;white-space:nowrap;color:#888;font-size:12px;}
.actions-cell{display:flex;gap:4px;flex-wrap:nowrap;align-items:center;}
.pagination{display:flex;gap:8px;align-items:center;justify-content:flex-end;margin-top:14px;font-size:13px;color:#888;}
.pagination button{background:#fff;border:1px solid #ddd;border-radius:8px;padding:5px 14px;cursor:pointer;font-size:13px;color:#333;}
.pagination button:hover:not(:disabled){background:#f5f5f3;}
.pagination button:disabled{opacity:0.4;cursor:default;}
.no-results{text-align:center;padding:40px;color:#aaa;font-size:14px;}
.no-results-order{margin-top:14px;}
.btn{border:1px solid #ddd;background:#fff;border-radius:8px;padding:7px 16px;font-size:13px;cursor:pointer;color:#333;transition:background 0.1s;}
.btn:hover{background:#f5f5f3;}
.btn-primary{background:#1a1a1a;color:#fff;border-color:#1a1a1a;}
.btn-primary:hover{background:#333;}
.btn-danger{color:#A32D2D;border-color:#f5c1c1;}
.btn-danger:hover{background:#FCEBEB;}
.btn-order{background:#FCEBEB;color:#A32D2D;border-color:#f5c1c1;font-size:11px;padding:3px 8px;border-radius:6px;cursor:pointer;border:1px solid;}
.btn-order:hover{background:#f7c1c1;}
.btn-reserve{background:#EEEDFE;color:#3C3489;border-color:#ceccf6;font-size:11px;padding:3px 8px;border-radius:6px;cursor:pointer;border:1px solid;}
.btn-reserve:hover{background:#d8d6f6;}
.btn-sm{padding:4px 10px;font-size:12px;}
.status-select{border:1px solid #ddd;background:#fff;color:#1a1a1a;border-radius:8px;padding:3px 8px;font-size:12px;cursor:pointer;outline:none;min-width:110px;}
/* status badge colors */
.st-ordered{background:#E1F5EE;color:#0F6E56;}
.st-arrived{background:#EAF3DE;color:#3B6D11;}
.st-reserved{background:#EEEDFE;color:#3C3489;}
.st-notified{background:#FAEEDA;color:#854F0B;}
.st-expired{background:#FCEBEB;color:#A32D2D;}
.st-unavail{background:#f0e8f8;color:#6b3fa0;}
.expired-tag{display:inline-flex;align-items:center;gap:4px;background:#FCEBEB;color:#A32D2D;font-size:11px;padding:2px 8px;border-radius:20px;font-weight:500;}
.days-left{font-size:11px;color:#854F0B;margin-left:4px;}
.overlay{display:none;position:fixed;inset:0;background:rgba(0,0,0,0.4);z-index:100;align-items:center;justify-content:center;}
.overlay.show{display:flex;}
.modal{background:#fff;border-radius:14px;padding:28px;width:100%;max-width:500px;box-shadow:0 8px 32px rgba(0,0,0,0.12);max-height:90vh;overflow-y:auto;}
.modal h2{font-size:17px;font-weight:600;margin-bottom:20px;}
.form-row{margin-bottom:14px;}
.form-row label{display:block;font-size:12px;color:#888;margin-bottom:5px;font-weight:500;}
.form-row input,.form-row select,.form-row textarea{width:100%;border:1px solid #ddd;border-radius:8px;padding:9px 12px;font-size:14px;outline:none;background:#fff;color:#1a1a1a;font-family:inherit;}
.form-row input:focus,.form-row select:focus,.form-row textarea:focus{border-color:#888;}
.form-row input:disabled{background:#f5f5f3;color:#888;}
.form-row textarea{resize:vertical;min-height:60px;}
.form-row-2{display:grid;grid-template-columns:1fr 1fr;gap:12px;margin-bottom:14px;}
.loc-block{border:1px solid #eee;border-radius:10px;padding:14px;margin-bottom:10px;background:#fafafa;}
.loc-check{display:flex;align-items:center;gap:8px;margin-bottom:10px;}
.loc-check input[type=checkbox]{width:16px;height:16px;cursor:pointer;accent-color:#1a1a1a;}
.loc-check label{font-size:13px;font-weight:500;color:#1a1a1a;cursor:pointer;}
.loc-row{display:flex;gap:8px;align-items:center;margin-bottom:8px;}
.loc-row label{font-size:12px;color:#888;width:40px;flex-shrink:0;}
.loc-row input,.loc-row select{flex:1;border:1px solid #ddd;border-radius:8px;padding:7px 10px;font-size:13px;outline:none;background:#fff;color:#1a1a1a;}
.modal-footer{display:flex;gap:10px;justify-content:flex-end;margin-top:20px;}
.section-divider{font-size:12px;font-weight:600;color:#888;margin:16px 0 8px;text-transform:uppercase;letter-spacing:0.5px;}
.form-modal-header{display:flex;align-items:center;gap:10px;margin-bottom:20px;}
.form-modal-header .part-id-badge{background:#1a1a1a;color:#fff;font-family:"SF Mono","Consolas",monospace;padding:4px 10px;border-radius:6px;font-size:14px;}
.login-wrap{display:flex;align-items:center;justify-content:center;min-height:calc(100vh - 52px);}
.login-card{background:#fff;border-radius:14px;border:1px solid #e8e8e8;padding:32px;width:100%;max-width:340px;text-align:center;}
.login-card h2{font-size:18px;font-weight:600;margin-bottom:8px;}
.login-card p{font-size:13px;color:#888;margin-bottom:24px;}
.login-card input{width:100%;border:1px solid #ddd;border-radius:8px;padding:10px 14px;font-size:15px;outline:none;text-align:center;letter-spacing:4px;margin-bottom:14px;}
.login-err{color:#A32D2D;font-size:13px;margin-top:8px;min-height:20px;}
.toast{position:fixed;bottom:24px;right:24px;background:#1a1a1a;color:#fff;padding:10px 18px;border-radius:10px;font-size:13px;opacity:0;transition:opacity 0.3s;pointer-events:none;z-index:999;}
.toast.show{opacity:1;}
.page{display:none;}
.page.active{display:block;}
.record-type-order{border-left:3px solid #A32D2D;}
.record-type-reserve{border-left:3px solid #3C3489;}
</style>
</head>
<body>

<nav>
  <span class="logo">MPU 零件庫存</span>
  <div class="nav-links">
    <button id="navQuery" class="active" onclick="showPage('query')">查詢</button>
    <button id="navAdmin" onclick="goAdmin()">管理</button>
  </div>
</nav>

<!-- 查詢頁 -->
<div id="pageQuery" class="page active">
  <div class="container">
    <div class="stats" id="statsQ"></div>
    <div class="toolbar">
      <input type="text" id="searchQ" placeholder="搜尋零件編號..." oninput="renderQuery()">
      <select id="seriesQ" onchange="renderQuery()"><option value="">全部系列</option></select>
      <select id="wallQ" onchange="renderQuery()">
        <option value="">全部位置</option>
        <option value="MPU">MPU</option>
        <option value="ASIS">ASIS</option>
        <option value="ASIS背面">ASIS背面</option>
      </select>
      <select id="levelQ" onchange="renderQuery()">
        <option value="">全部水位</option>
        <option value="high">充足</option>
        <option value="low">偏低</option>
        <option value="out">缺貨</option>
        <option value="na">缺貨無法訂購</option>
      </select>
    </div>
    <div class="table-wrap">
      <table>
        <thead><tr>
          <th>零件編號</th><th>系列</th><th>架位</th><th>數量</th><th>庫存水位</th><th>備註</th><th>動作</th>
        </tr></thead>
        <tbody id="tbodyQ"></tbody>
      </table>
      <div id="noResultsQ" class="no-results" style="display:none">
        查無符合的零件
        <div class="no-results-order">
          <button class="btn btn-order" id="noResultsOrderBtn" style="font-size:13px;padding:7px 16px;" onclick="openOrderModalUnknown()">+ 訂購此零件</button>
        </div>
      </div>
    </div>
    <div class="pagination">
      <span id="pageInfoQ"></span>
      <button id="prevQ" onclick="changePageQ(-1)">← 上一頁</button>
      <button id="nextQ" onclick="changePageQ(1)">下一頁 →</button>
    </div>
  </div>
</div>

<!-- 登入 -->
<div id="pageLogin" class="page">
  <div class="login-wrap">
    <div class="login-card">
      <h2>管理者登入</h2>
      <p>請輸入管理密碼</p>
      <input type="password" id="pwInput" placeholder="••••" onkeydown="if(event.key==='Enter')doLogin()">
      <button class="btn btn-primary" style="width:100%" onclick="doLogin()">進入管理頁面</button>
      <div class="login-err" id="loginErr"></div>
    </div>
  </div>
</div>

<!-- 管理頁 -->
<div id="pageAdmin" class="page">
  <div class="container">
    <div class="tab-bar">
      <button class="tab-btn active" id="tabInventory" onclick="switchAdminTab('inventory')">庫存管理</button>
      <button class="tab-btn" id="tabRecords" onclick="switchAdminTab('records')">訂購/預留紀錄 <span class="badge-count" id="expiredBadge" style="display:none">!</span></button>
    </div>

    <!-- 庫存管理 tab -->
    <div id="adminInventory">
      <div class="stats" id="statsA"></div>
      <div class="toolbar">
        <input type="text" id="searchA" placeholder="搜尋零件編號..." oninput="renderAdmin()">
        <select id="seriesA" onchange="renderAdmin()"><option value="">全部系列</option></select>
        <select id="wallA" onchange="renderAdmin()">
          <option value="">全部位置</option>
          <option value="MPU">MPU</option>
          <option value="ASIS">ASIS</option>
          <option value="ASIS背面">ASIS背面</option>
        </select>
        <select id="levelA" onchange="renderAdmin()">
          <option value="">全部水位</option>
          <option value="high">充足</option>
          <option value="low">偏低</option>
          <option value="out">缺貨</option>
          <option value="na">缺貨無法訂購</option>
        </select>
        <button class="btn btn-primary" onclick="openAddModal()">＋ 新增零件</button>
        <button class="btn" onclick="exportCSV()">匯出 CSV</button>
        <button class="btn" onclick="logout()" style="margin-left:auto">登出</button>
      </div>
      <div class="table-wrap">
        <table>
          <thead><tr>
            <th>零件編號</th><th>系列</th><th>架位</th><th>數量</th><th>庫存水位</th><th>備註</th><th>操作</th>
          </tr></thead>
          <tbody id="tbodyA"></tbody>
        </table>
        <div id="noResultsA" class="no-results" style="display:none">查無符合的零件</div>
      </div>
      <div class="pagination">
        <span id="pageInfoA"></span>
        <button id="prevA" onclick="changePageA(-1)">← 上一頁</button>
        <button id="nextA" onclick="changePageA(1)">下一頁 →</button>
      </div>
    </div>

    <!-- 紀錄 tab -->
    <div id="adminRecords" style="display:none">
      <div class="toolbar">
        <input type="text" id="searchRec" placeholder="搜尋零件編號或電話..." oninput="renderRecords()">
        <select id="recTypeFilter" onchange="renderRecords()">
          <option value="">全部類型</option>
          <option value="訂購">訂購單</option>
          <option value="預留">預留單</option>
        </select>
        <select id="recStatusFilter" onchange="renderRecords()">
          <option value="">全部狀態</option>
          <option value="未處理">未處理</option>
          <option value="已訂購">已訂購</option>
          <option value="已到貨">已到貨</option>
          <option value="已預留">已預留</option>
          <option value="已通知">已通知</option>
          <option value="逾期未領">逾期未領</option>
          <option value="缺貨無法訂購">缺貨無法訂購</option>
        </select>
        <button class="btn" onclick="exportRecordsCSV()">匯出紀錄 CSV</button>
      </div>
      <div class="table-wrap">
        <table>
          <thead><tr>
            <th>類型</th><th>零件編號</th><th>系列</th><th>數量</th><th>聯絡人</th><th>電話</th><th>CSC處理人</th><th>日期</th><th>狀態</th><th>到期日</th><th>備註</th><th>操作</th>
          </tr></thead>
          <tbody id="tbodyRec"></tbody>
        </table>
        <div id="noResultsRec" class="no-results" style="display:none">目前沒有紀錄</div>
      </div>
      <div class="pagination">
        <span id="pageInfoRec"></span>
        <button id="prevRec" onclick="changePageRec(-1)">← 上一頁</button>
        <button id="nextRec" onclick="changePageRec(1)">下一頁 →</button>
      </div>
    </div>
  </div>
</div>

<!-- 新增/編輯零件 Modal -->
<div class="overlay" id="partModal">
  <div class="modal">
    <h2 id="modalTitle">新增零件</h2>
    <div class="form-row-2">
      <div class="form-row" style="margin-bottom:0">
        <label>零件編號 *</label>
        <input type="text" id="fId" placeholder="例：110019">
      </div>
      <div class="form-row" style="margin-bottom:0">
        <label>系列（手動輸入）</label>
        <input type="text" id="fSeries" placeholder="例：沙發系列">
      </div>
    </div>
    <div class="section-divider" style="margin-top:16px">架位設定</div>
    <div class="loc-block">
      <div class="loc-check">
        <input type="checkbox" id="chkMPU" onchange="toggleLoc('MPU')">
        <label for="chkMPU">MPU 零件牆</label>
      </div>
      <div id="locMPU" style="display:none">
        <div class="loc-row">
          <label>區域</label>
          <select id="fZoneMPUarea">
            <option value="">請選擇</option>
            <option value="A區">A區</option><option value="B區">B區</option>
            <option value="C區">C區</option><option value="D區">D區</option>
            <option value="E區">E區</option><option value="F區">F區</option>
            <option value="G區">G區</option><option value="H區">H區</option>
            <option value="小房間">小房間</option>
          </select>
        </div>
        <div class="loc-row">
          <label>第幾排</label>
          <input type="number" id="fZoneMPUrow" placeholder="例：3" min="1" style="max-width:120px">
          <span style="font-size:12px;color:#888;margin-left:4px">排（選填）</span>
        </div>
      </div>
    </div>
    <div class="loc-block">
      <div class="loc-check">
        <input type="checkbox" id="chkASIS" onchange="toggleLoc('ASIS')">
        <label for="chkASIS">ASIS 零件牆</label>
      </div>
      <div id="locASIS" style="display:none">
        <div class="loc-row">
          <label>排</label>
          <select id="fASISrow">
            <option value="">請選擇</option>
            <option value="1">1排</option><option value="2">2排</option><option value="3">3排</option>
            <option value="4">4排</option><option value="5">5排</option><option value="6">6排</option>
            <option value="7">7排</option><option value="8">8排</option><option value="9">9排</option>
            <option value="10">10排</option><option value="11">11排</option><option value="12">12排</option>
          </select>
        </div>
        <div class="loc-row">
          <label>格</label>
          <select id="fASIScol">
            <option value="">請選擇</option>
            <option value="1">1</option><option value="2">2</option><option value="3">3</option>
            <option value="4">4</option><option value="5">5</option><option value="6">6</option>
            <option value="7">7</option><option value="8">8</option><option value="9">9</option>
            <option value="10">10</option><option value="11">11</option><option value="12">12</option>
            <option value="13">13</option><option value="14">14</option><option value="15">15</option>
            <option value="16">16</option>
          </select>
        </div>
      </div>
    </div>
    <div class="loc-block">
      <div class="loc-check">
        <input type="checkbox" id="chkASISB" onchange="toggleLoc('ASISB')">
        <label for="chkASISB">ASIS 背面</label>
      </div>
    </div>
    <div class="form-row-2" style="margin-top:4px">
      <div class="form-row" style="margin-bottom:0">
        <label>數量</label>
        <input type="number" id="fQty" placeholder="0" min="0">
      </div>
      <div class="form-row" style="margin-bottom:0">
        <label>庫存水位</label>
        <select id="fLevel">
          <option value="high">充足</option>
          <option value="low">偏低</option>
          <option value="out">缺貨</option>
          <option value="na">缺貨無法訂購</option>
        </select>
      </div>
    </div>
    <div class="form-row" style="margin-top:14px">
      <label>備註</label>
      <textarea id="fNote" placeholder="選填"></textarea>
    </div>
    <div id="modalErr" style="color:#A32D2D;font-size:13px;min-height:18px;margin-top:4px;"></div>
    <div class="modal-footer">
      <button class="btn" onclick="closeModal()">取消</button>
      <button class="btn btn-primary" onclick="savePart()">✓ 儲存</button>
    </div>
  </div>
</div>

<!-- 刪除確認 -->
<div class="overlay" id="delModal">
  <div class="modal" style="max-width:360px;text-align:center;">
    <h2 style="margin-bottom:10px">確認刪除？</h2>
    <p style="color:#888;font-size:14px;margin-bottom:24px">零件 <strong id="delPartId"></strong> 刪除後無法復原</p>
    <div style="display:flex;gap:10px;justify-content:center;">
      <button class="btn" onclick="closeDelModal()">取消</button>
      <button class="btn btn-danger" onclick="confirmDelete()">確認刪除</button>
    </div>
  </div>
</div>

<!-- 訂購表單 -->
<div class="overlay" id="orderModal">
  <div class="modal" style="max-width:440px;">
    <div class="form-modal-header">
      <h2 style="margin-bottom:0">訂購單</h2>
      <span class="part-id-badge" id="orderPartId"></span>
    </div>
    <div class="form-row"><label>零件編號</label><input type="text" id="oPartId" disabled></div>
    <div class="form-row"><label>系列</label><input type="text" id="oSeries" disabled></div>
    <div class="form-row-2">
      <div class="form-row" style="margin-bottom:0"><label>訂購數量 *</label><input type="number" id="oQty" placeholder="0" min="1"></div>
      <div class="form-row" style="margin-bottom:0"><label>日期</label><input type="text" id="oDate" disabled></div>
    </div>
    <div class="form-row"><label>聯絡人姓名 *</label><input type="text" id="oName" placeholder="請輸入姓名"></div>
    <div class="form-row"><label>聯絡電話</label><input type="tel" id="oPhone" placeholder="請輸入電話"></div>
    <div class="form-row"><label>備註</label><textarea id="oNote" placeholder="選填"></textarea></div>
    <div id="orderErr" style="color:#A32D2D;font-size:13px;min-height:18px;"></div>
    <div class="modal-footer">
      <button class="btn" onclick="closeOrderModal()">取消</button>
      <button class="btn" onclick="printForm('order')">列印</button>
      <button class="btn btn-primary" onclick="submitOrder()">✓ 送出訂購</button>
    </div>
  </div>
</div>

<!-- 預留表單 -->
<div class="overlay" id="reserveModal">
  <div class="modal" style="max-width:440px;">
    <div class="form-modal-header">
      <h2 style="margin-bottom:0">預留單</h2>
      <span class="part-id-badge" id="reservePartId"></span>
    </div>
    <div class="form-row"><label>零件編號</label><input type="text" id="rPartId" disabled></div>
    <div class="form-row"><label>系列</label><input type="text" id="rSeries" disabled></div>
    <div class="form-row-2">
      <div class="form-row" style="margin-bottom:0"><label>預留數量 *</label><input type="number" id="rQty" placeholder="0" min="1"></div>
      <div class="form-row" style="margin-bottom:0"><label>日期</label><input type="text" id="rDate" disabled></div>
    </div>
    <div class="form-row"><label>聯絡人姓名 *</label><input type="text" id="rName" placeholder="請輸入姓名"></div>
    <div class="form-row"><label>聯絡電話</label><input type="tel" id="rPhone" placeholder="請輸入電話"></div>
    <div class="form-row"><label>CSC 處理人</label><input type="text" id="rCSC" placeholder="請輸入處理人姓名"></div>
    <div class="form-row"><label>備註</label><textarea id="rNote" placeholder="選填"></textarea></div>
    <div id="reserveErr" style="color:#A32D2D;font-size:13px;min-height:18px;"></div>
    <div class="modal-footer">
      <button class="btn" onclick="closeReserveModal()">取消</button>
      <button class="btn" onclick="printForm('reserve')">列印</button>
      <button class="btn btn-primary" onclick="submitReserve()">✓ 送出預留</button>
    </div>
  </div>
</div>

<!-- 儲存成功 -->
<div class="overlay" id="savedModal">
  <div class="modal" style="max-width:320px;text-align:center;">
    <div style="font-size:32px;margin-bottom:12px">✓</div>
    <h2 style="margin-bottom:8px" id="savedTitle">儲存成功</h2>
    <p style="color:#888;font-size:14px;margin-bottom:24px" id="savedMsg"></p>
    <button class="btn btn-primary" style="width:100%" onclick="closeSavedModal()">確定</button>
  </div>
</div>

<!-- 紀錄備註編輯 -->
<div class="overlay" id="recNoteModal">
  <div class="modal" style="max-width:380px;">
    <h2>編輯備註</h2>
    <div class="form-row" style="margin-top:14px">
      <label>備註</label>
      <textarea id="recNoteInput" style="min-height:80px" placeholder="請輸入備註"></textarea>
    </div>
    <div class="modal-footer">
      <button class="btn" onclick="closeRecNoteModal()">取消</button>
      <button class="btn btn-primary" onclick="saveRecNote()">✓ 儲存</button>
    </div>
  </div>
</div>

<div class="toast" id="toast"></div>

<script>
const ADMIN_PW='5237';
const STORAGE_KEY='mpu_parts_v6';
const ORDERS_KEY='mpu_orders_v6';
const PAGE_SIZE=15;

const DEFAULT_PARTS=[
  {id:"110019",series:"沙發系列",locations:[{wall:"MPU",zone:"A區"}],qty:"",level:"high",note:""},
  {id:"117175",series:"沙發系列",locations:[{wall:"MPU",zone:"A區"}],qty:"",level:"high",note:""},
  {id:"100712",series:"沙發系列",locations:[{wall:"MPU",zone:"B區"}],qty:"",level:"high",note:""},
  {id:"100854",series:"沙發系列",locations:[{wall:"MPU",zone:"B區"}],qty:"",level:"high",note:""},
  {id:"110439",series:"沙發系列",locations:[{wall:"MPU",zone:""}],qty:"",level:"high",note:""},
  {id:"114509",series:"沙發系列",locations:[{wall:"MPU",zone:"B區"}],qty:"",level:"high",note:""},
  {id:"106629",series:"沙發系列",locations:[{wall:"MPU",zone:""}],qty:"",level:"high",note:""},
  {id:"130914",series:"沙發系列",locations:[{wall:"MPU",zone:""}],qty:"",level:"high",note:""},
  {id:"109142",series:"沙發系列",locations:[{wall:"MPU",zone:"B區"}],qty:"",level:"high",note:""},
  {id:"130646",series:"沙發系列",locations:[{wall:"MPU",zone:"C區"}],qty:"",level:"high",note:""},
  {id:"105107",series:"沙發系列",locations:[{wall:"MPU",zone:"C區"}],qty:"",level:"high",note:""},
  {id:"154571",series:"沙發系列",locations:[{wall:"MPU",zone:"D區"}],qty:"",level:"high",note:""},
  {id:"130618",series:"電視櫃系列",locations:[{wall:"MPU",zone:""}],qty:"",level:"high",note:""},
  {id:"118137",series:"電視櫃系列",locations:[{wall:"MPU",zone:""}],qty:"",level:"high",note:""},
  {id:"122971",series:"電視櫃系列",locations:[{wall:"MPU",zone:""}],qty:"",level:"high",note:""},
  {id:"119253",series:"電視櫃系列",locations:[{wall:"MPU",zone:""}],qty:"",level:"high",note:""},
  {id:"109049",series:"電視櫃系列",locations:[{wall:"MPU",zone:""}],qty:"",level:"high",note:""},
  {id:"113287",series:"電視櫃系列",locations:[{wall:"MPU",zone:""}],qty:"",level:"high",note:""},
  {id:"116637",series:"電視櫃系列",locations:[{wall:"MPU",zone:""}],qty:"",level:"high",note:""},
  {id:"113301",series:"電視櫃系列",locations:[{wall:"MPU",zone:""}],qty:"",level:"high",note:""},
  {id:"100218",series:"電視櫃系列",locations:[{wall:"MPU",zone:""}],qty:"",level:"high",note:""},
  {id:"118331",series:"電視櫃系列",locations:[{wall:"MPU",zone:"B區"}],qty:"",level:"high",note:""},
  {id:"110630",series:"電視櫃系列",locations:[{wall:"MPU",zone:""}],qty:"",level:"high",note:""},
  {id:"100365",series:"電視櫃系列",locations:[{wall:"MPU",zone:""}],qty:"",level:"high",note:""},
  {id:"151705",series:"電視櫃系列",locations:[{wall:"MPU",zone:""}],qty:"",level:"high",note:""},
  {id:"103430",series:"電視櫃系列",locations:[{wall:"MPU",zone:""}],qty:"",level:"high",note:""},
  {id:"119252",series:"電視櫃系列",locations:[{wall:"MPU",zone:""}],qty:"",level:"high",note:""},
  {id:"105248",series:"電視櫃系列",locations:[{wall:"MPU",zone:""}],qty:"",level:"high",note:""},
  {id:"101350",series:"電視櫃系列",locations:[{wall:"MPU",zone:""}],qty:"",level:"high",note:""},
  {id:"144574",series:"電視櫃系列",locations:[{wall:"MPU",zone:""}],qty:"",level:"high",note:""},
  {id:"106720",series:"電視櫃系列",locations:[{wall:"MPU",zone:""}],qty:"",level:"high",note:""},
  {id:"101345",series:"電視櫃系列",locations:[{wall:"MPU",zone:""}],qty:"",level:"high",note:""},
  {id:"110519",series:"電視櫃系列",locations:[{wall:"MPU",zone:""}],qty:"",level:"high",note:""},
  {id:"100391",series:"床區系列",locations:[{wall:"MPU",zone:""}],qty:"",level:"high",note:""},
  {id:"100325",series:"床區系列",locations:[{wall:"MPU",zone:""}],qty:"",level:"high",note:""},
  {id:"105307",series:"床區系列",locations:[{wall:"MPU",zone:""}],qty:"",level:"high",note:""},
  {id:"105330",series:"床區系列",locations:[{wall:"MPU",zone:""}],qty:"",level:"high",note:""},
  {id:"104875",series:"床區系列",locations:[{wall:"MPU",zone:""}],qty:"",level:"high",note:""},
  {id:"113434",series:"床區系列",locations:[{wall:"MPU",zone:""}],qty:"",level:"high",note:""},
  {id:"100349",series:"床區系列",locations:[{wall:"MPU",zone:""}],qty:"",level:"high",note:""},
  {id:"100229",series:"床區系列",locations:[{wall:"MPU",zone:""}],qty:"",level:"high",note:""},
  {id:"114671",series:"床區系列",locations:[{wall:"MPU",zone:""}],qty:"",level:"high",note:""},
  {id:"118381",series:"床區系列",locations:[{wall:"MPU",zone:""}],qty:"",level:"high",note:""},
  {id:"113301",series:"層板豆系列",locations:[{wall:"MPU",zone:""}],qty:"",level:"high",note:""},
  {id:"106414",series:"層板豆系列",locations:[{wall:"MPU",zone:""}],qty:"",level:"high",note:""},
  {id:"104171",series:"層板豆系列",locations:[{wall:"MPU",zone:""}],qty:"",level:"high",note:""},
  {id:"128643",series:"層板豆系列",locations:[{wall:"MPU",zone:""}],qty:"",level:"high",note:""},
  {id:"114929",series:"層板豆系列",locations:[{wall:"MPU",zone:""}],qty:"",level:"high",note:""},
  {id:"110646",series:"",locations:[{wall:"ASIS背面",zone:""}],qty:"",level:"high",note:""},
  {id:"110364",series:"",locations:[{wall:"ASIS背面",zone:""}],qty:"",level:"high",note:""},
  {id:"110328",series:"",locations:[{wall:"ASIS背面",zone:""}],qty:"",level:"high",note:""},
  {id:"116713",series:"",locations:[{wall:"ASIS背面",zone:""}],qty:"",level:"high",note:""},
  {id:"124328",series:"",locations:[{wall:"ASIS背面",zone:""}],qty:"",level:"high",note:""},
  {id:"141413",series:"",locations:[{wall:"ASIS背面",zone:""}],qty:"",level:"high",note:""},
  {id:"103693",series:"",locations:[{wall:"ASIS背面",zone:""}],qty:"",level:"high",note:""},
  {id:"106989",series:"",locations:[{wall:"ASIS背面",zone:""}],qty:"",level:"high",note:""},
  {id:"141414",series:"",locations:[{wall:"ASIS背面",zone:""}],qty:"",level:"high",note:""},
  {id:"10107965",series:"",locations:[{wall:"ASIS背面",zone:""}],qty:"",level:"high",note:""},
  {id:"109338",series:"",locations:[{wall:"ASIS背面",zone:""}],qty:"",level:"high",note:""},
  {id:"114947",series:"",locations:[{wall:"ASIS背面",zone:""}],qty:"",level:"high",note:""},
  {id:"10107957",series:"",locations:[{wall:"ASIS背面",zone:""}],qty:"",level:"high",note:""},
  {id:"109221",series:"",locations:[{wall:"ASIS背面",zone:""}],qty:"",level:"high",note:""},
  {id:"10050684",series:"",locations:[{wall:"ASIS背面",zone:""}],qty:"",level:"high",note:""},
  {id:"126916",series:"",locations:[{wall:"ASIS背面",zone:""}],qty:"",level:"high",note:""},
  {id:"117687",series:"",locations:[{wall:"ASIS背面",zone:""}],qty:"",level:"high",note:""},
  {id:"133194",series:"",locations:[{wall:"ASIS背面",zone:""}],qty:"",level:"high",note:""},
  {id:"114931",series:"",locations:[{wall:"ASIS背面",zone:""}],qty:"",level:"high",note:""},
  {id:"152650",series:"",locations:[{wall:"ASIS背面",zone:""}],qty:"",level:"high",note:""},
  {id:"130531",series:"",locations:[{wall:"ASIS背面",zone:""}],qty:"",level:"high",note:""},
  {id:"139285",series:"",locations:[{wall:"ASIS背面",zone:""}],qty:"",level:"high",note:""},
  {id:"139294",series:"",locations:[{wall:"ASIS背面",zone:""}],qty:"",level:"high",note:""},
  {id:"10051085",series:"",locations:[{wall:"ASIS背面",zone:""}],qty:"",level:"high",note:""},
  {id:"10056864",series:"",locations:[{wall:"ASIS背面",zone:""}],qty:"",level:"high",note:""},
  {id:"10056885",series:"",locations:[{wall:"ASIS背面",zone:""}],qty:"",level:"high",note:""},
  {id:"10058633",series:"",locations:[{wall:"ASIS背面",zone:""}],qty:"",level:"high",note:""},
  {id:"142400",series:"",locations:[{wall:"ASIS背面",zone:""}],qty:"",level:"high",note:""},
  {id:"10050403",series:"",locations:[{wall:"ASIS背面",zone:""}],qty:"",level:"high",note:""},
  {id:"10045938",series:"",locations:[{wall:"ASIS背面",zone:""}],qty:"",level:"high",note:""},
  {id:"115443",series:"",locations:[{wall:"ASIS背面",zone:""}],qty:"",level:"high",note:""},
  {id:"100707",series:"",locations:[{wall:"ASIS背面",zone:""}],qty:"",level:"high",note:""},
  {id:"192158",series:"",locations:[{wall:"ASIS背面",zone:""}],qty:"",level:"high",note:""},
  {id:"10045937",series:"",locations:[{wall:"ASIS背面",zone:""}],qty:"",level:"high",note:""},
  {id:"10045911",series:"",locations:[{wall:"ASIS背面",zone:""}],qty:"",level:"high",note:""},
  {id:"115444",series:"",locations:[{wall:"ASIS背面",zone:""}],qty:"",level:"high",note:""},
  {id:"144261",series:"",locations:[{wall:"ASIS背面",zone:""}],qty:"",level:"high",note:""},
  {id:"10061614",series:"",locations:[{wall:"ASIS背面",zone:""}],qty:"",level:"high",note:""},
  {id:"10087158",series:"",locations:[{wall:"ASIS背面",zone:""}],qty:"",level:"high",note:""},
  {id:"101324",series:"",locations:[{wall:"ASIS背面",zone:""}],qty:"",level:"high",note:""},
  {id:"157409",series:"",locations:[{wall:"ASIS背面",zone:""}],qty:"",level:"high",note:""},
  {id:"107091",series:"",locations:[{wall:"ASIS背面",zone:""}],qty:"",level:"high",note:""},
  {id:"102384",series:"",locations:[{wall:"ASIS背面",zone:""}],qty:"",level:"high",note:""},
  {id:"10082805",series:"",locations:[{wall:"ASIS背面",zone:""}],qty:"",level:"high",note:""},
  {id:"152059",series:"",locations:[{wall:"ASIS背面",zone:""}],qty:"",level:"high",note:""},
  {id:"153550",series:"",locations:[{wall:"ASIS背面",zone:""}],qty:"",level:"high",note:""},
  {id:"104663",series:"",locations:[{wall:"ASIS背面",zone:""}],qty:"",level:"high",note:""},
  {id:"115753",series:"",locations:[{wall:"ASIS背面",zone:""}],qty:"",level:"high",note:""},
  {id:"149842",series:"",locations:[{wall:"ASIS背面",zone:""}],qty:"",level:"high",note:""},
  {id:"107103",series:"",locations:[{wall:"ASIS背面",zone:""}],qty:"",level:"high",note:""},
  {id:"104895",series:"",locations:[{wall:"ASIS背面",zone:""}],qty:"",level:"high",note:""},
  {id:"107616",series:"",locations:[{wall:"ASIS背面",zone:""}],qty:"",level:"high",note:""},
  {id:"115983",series:"",locations:[{wall:"ASIS背面",zone:""}],qty:"",level:"high",note:""},
  {id:"190949",series:"",locations:[{wall:"ASIS背面",zone:""}],qty:"",level:"high",note:""},
  {id:"190950",series:"",locations:[{wall:"ASIS背面",zone:""}],qty:"",level:"high",note:""},
  {id:"190948",series:"",locations:[{wall:"ASIS背面",zone:""}],qty:"",level:"high",note:""},
  {id:"190905",series:"",locations:[{wall:"ASIS背面",zone:""}],qty:"",level:"high",note:""},
  {id:"128864",series:"",locations:[{wall:"ASIS背面",zone:""}],qty:"",level:"high",note:""},
  {id:"139212",series:"",locations:[{wall:"ASIS背面",zone:""}],qty:"",level:"high",note:""},
  {id:"102335",series:"",locations:[{wall:"ASIS背面",zone:""}],qty:"",level:"high",note:""},
  {id:"101313",series:"",locations:[{wall:"ASIS背面",zone:""}],qty:"",level:"high",note:""},
  {id:"153748",series:"",locations:[{wall:"ASIS背面",zone:""}],qty:"",level:"high",note:""},
  {id:"130485",series:"",locations:[{wall:"ASIS背面",zone:""}],qty:"",level:"high",note:""},
  {id:"10081882",series:"",locations:[{wall:"ASIS背面",zone:""}],qty:"",level:"high",note:""},
  {id:"110389",series:"",locations:[{wall:"ASIS背面",zone:""}],qty:"",level:"high",note:""},
  {id:"10082205",series:"",locations:[{wall:"ASIS背面",zone:""}],qty:"",level:"high",note:""},
  {id:"190169",series:"",locations:[{wall:"ASIS背面",zone:""}],qty:"",level:"high",note:""},
  {id:"10082290",series:"",locations:[{wall:"ASIS背面",zone:""}],qty:"",level:"high",note:""},
  {id:"10082310",series:"",locations:[{wall:"ASIS背面",zone:""}],qty:"",level:"high",note:""},
  {id:"10064763",series:"",locations:[{wall:"ASIS背面",zone:""}],qty:"",level:"high",note:""},
  {id:"10062668",series:"",locations:[{wall:"ASIS背面",zone:""}],qty:"",level:"high",note:""},
  {id:"190170",series:"",locations:[{wall:"ASIS背面",zone:""}],qty:"",level:"high",note:""},
  {id:"146114",series:"",locations:[{wall:"ASIS背面",zone:""}],qty:"",level:"high",note:""},
  {id:"152527",series:"",locations:[{wall:"ASIS背面",zone:""}],qty:"",level:"high",note:""},
  {id:"123742",series:"",locations:[{wall:"ASIS背面",zone:""}],qty:"",level:"high",note:""},
  {id:"158457",series:"",locations:[{wall:"ASIS背面",zone:""}],qty:"",level:"high",note:""},
  {id:"10047244",series:"",locations:[{wall:"ASIS背面",zone:""}],qty:"",level:"high",note:""},
  {id:"10094827",series:"",locations:[{wall:"ASIS背面",zone:""}],qty:"",level:"high",note:""},
  {id:"152526",series:"",locations:[{wall:"ASIS背面",zone:""}],qty:"",level:"high",note:""},
  {id:"152497",series:"",locations:[{wall:"ASIS背面",zone:""}],qty:"",level:"high",note:""},
  {id:"194377",series:"",locations:[{wall:"ASIS背面",zone:""}],qty:"",level:"high",note:""},
  {id:"101513",series:"",locations:[{wall:"ASIS背面",zone:""}],qty:"",level:"high",note:""},
  {id:"139477",series:"",locations:[{wall:"ASIS背面",zone:""}],qty:"",level:"high",note:""},
  {id:"133127",series:"",locations:[{wall:"ASIS背面",zone:""}],qty:"",level:"high",note:""},
  {id:"10036248",series:"",locations:[{wall:"ASIS背面",zone:""}],qty:"",level:"high",note:""},
  {id:"139467",series:"",locations:[{wall:"ASIS背面",zone:""}],qty:"",level:"high",note:""},
  {id:"139468",series:"",locations:[{wall:"ASIS背面",zone:""}],qty:"",level:"high",note:""},
  {id:"155200",series:"",locations:[{wall:"ASIS背面",zone:""}],qty:"",level:"high",note:""},
  {id:"155201",series:"",locations:[{wall:"ASIS背面",zone:""}],qty:"",level:"high",note:""},
  {id:"10036250",series:"",locations:[{wall:"ASIS背面",zone:""}],qty:"",level:"high",note:""},
  {id:"155197",series:"",locations:[{wall:"ASIS背面",zone:""}],qty:"",level:"high",note:""},
  {id:"155198",series:"",locations:[{wall:"ASIS背面",zone:""}],qty:"",level:"high",note:""},
  {id:"10003221",series:"",locations:[{wall:"ASIS背面",zone:""}],qty:"",level:"high",note:""},
  {id:"133148",series:"",locations:[{wall:"ASIS背面",zone:""}],qty:"",level:"high",note:""},
  {id:"10003241",series:"",locations:[{wall:"ASIS背面",zone:""}],qty:"",level:"high",note:""},
  {id:"105160",series:"",locations:[{wall:"ASIS背面",zone:""}],qty:"",level:"high",note:""},
  {id:"10050797",series:"",locations:[{wall:"ASIS背面",zone:""}],qty:"",level:"high",note:""},
  {id:"128857",series:"",locations:[{wall:"ASIS背面",zone:""}],qty:"",level:"high",note:""},
  {id:"194077",series:"",locations:[{wall:"ASIS背面",zone:""}],qty:"",level:"high",note:""},
  {id:"104850",series:"",locations:[{wall:"ASIS背面",zone:""}],qty:"",level:"high",note:""},
  {id:"148624",series:"",locations:[{wall:"ASIS背面",zone:""}],qty:"",level:"high",note:""},
  {id:"117363",series:"",locations:[{wall:"ASIS背面",zone:""}],qty:"",level:"high",note:""},
  {id:"104864",series:"",locations:[{wall:"ASIS背面",zone:""}],qty:"",level:"high",note:""},
  {id:"108116",series:"",locations:[{wall:"ASIS背面",zone:""}],qty:"",level:"high",note:""},
];

let parts=[], orders=[];
let loggedIn=false, editIdx=null, deleteIdx=null;
let pageQ=1, pageA=1, pageRec=1;
let filteredQ=[], filteredA=[], filteredRec=[];
let savedPageA=1;
let currentAdminTab='inventory';
let editRecNoteIdx=null;

function load(){
  try{const s=localStorage.getItem(STORAGE_KEY);parts=s?JSON.parse(s):JSON.parse(JSON.stringify(DEFAULT_PARTS));}
  catch(e){parts=JSON.parse(JSON.stringify(DEFAULT_PARTS));}
  try{const o=localStorage.getItem(ORDERS_KEY);orders=o?JSON.parse(o):[];}
  catch(e){orders=[];}
  checkExpiry();
}
function save(){localStorage.setItem(STORAGE_KEY,JSON.stringify(parts));}
function saveOrders(){localStorage.setItem(ORDERS_KEY,JSON.stringify(orders));}

function checkExpiry(){
  const now=new Date();
  let changed=false;
  orders.forEach(o=>{
    if (o.status === '已通知' && o.notifiedDate) {
  const notified = new Date(o.notifiedDate);
  const expireDate = new Date(notified);
  expireDate.setDate(expireDate.getDate() + 30);
  if (now >= expireDate) {o.status = '逾期未領';changed = true;
  }
}

  });
  if(changed)saveOrders();
}

function daysLeft(notifiedDate){
  if(!notifiedDate)return null;
  const notified=new Date(notifiedDate);
  const now=new Date();
  const diff=30-Math.floor((now-notified)/(1000*60*60*24));
  return diff;
}

function lvClass(lv){return{high:'lv-high',low:'lv-low',out:'lv-out',na:'lv-na'}[lv]||'lv-high';}
function lvLabel(lv){return{high:'充足',low:'偏低',out:'缺貨',na:'缺貨無法訂購'}[lv]||'充足';}
function wallTagClass(w){if(w==='MPU')return'tag-mpu';if(w==='ASIS')return'tag-asis';return'tag-asisb';}

function statusClass(s){
  return{
    '未處理': 'st-pending','已訂購':'st-ordered','已到貨':'st-arrived','已預留':'st-reserved',
    '已通知':'st-notified','逾期未領':'st-expired','缺貨無法訂購':'st-unavail'
  }[s]||'st-pending';
}

function locHTML(locs){
  if(!locs||!locs.length)return'<span style="color:#bbb">—</span>';
  return'<div class="locations">'+locs.map(l=>{
    const label=l.zone?`${l.wall}-${l.zone}`:l.wall;
    return`<span class="tag ${wallTagClass(l.wall)}">${label}</span>`;
  }).join('')+'</div>';
}

function todayStr(){return new Date().toLocaleDateString('zh-TW',{year:'numeric',month:'2-digit',day:'2-digit'});}
function todayISO(){return new Date().toISOString().split('T')[0];}

function showToast(msg){
  const t=document.getElementById('toast');
  t.textContent=msg;t.classList.add('show');
  setTimeout(()=>t.classList.remove('show'),2500);
}

function getSeriesList(){
  return Array.from(new Set(parts.map(p=>p.series).filter(Boolean))).sort();
}
function updateSeriesDropdowns(){
  const list=getSeriesList();
  ['seriesQ','seriesA'].forEach(id=>{
    const sel=document.getElementById(id);
    if(!sel)return;
    const cur=sel.value;
    sel.innerHTML='<option value="">全部系列</option>'+list.map(s=>`<option value="${s}">${s}</option>`).join('');
    sel.value=cur;
  });
}

function updateExpiredBadge(){
  checkExpiry();
  const cnt=orders.filter(o=>o.status==='逾期未領').length;
  const badge=document.getElementById('expiredBadge');
  if(cnt>0){badge.style.display='';badge.textContent=cnt;}
  else badge.style.display='none';
}

/* ---- PAGE NAV ---- */
function showPage(p){
  ['Query','Login','Admin'].forEach(x=>document.getElementById('page'+x).classList.remove('active'));
  document.getElementById('navQuery').classList.remove('active');
  document.getElementById('navAdmin').classList.remove('active');
  document.getElementById('page'+p.charAt(0).toUpperCase()+p.slice(1)).classList.add('active');
  if(p==='query'){document.getElementById('navQuery').classList.add('active');updateSeriesDropdowns();renderQuery();renderStatsQ();}
  if(p==='admin'){document.getElementById('navAdmin').classList.add('active');updateSeriesDropdowns();updateExpiredBadge();
    if(currentAdminTab==='inventory'){renderAdmin();renderStatsA();}else{renderRecords();}}
}
function goAdmin(){
  if(loggedIn){showPage('admin');}
  else{showPage('login');document.getElementById('pwInput').value='';document.getElementById('loginErr').textContent='';}
}
function doLogin(){
  if(document.getElementById('pwInput').value===ADMIN_PW){loggedIn=true;showPage('admin');}
  else{document.getElementById('loginErr').textContent='密碼錯誤，請重試';}
}
function logout(){loggedIn=false;showPage('query');showToast('已登出');}

function switchAdminTab(tab){
  currentAdminTab=tab;
  document.getElementById('tabInventory').classList.toggle('active',tab==='inventory');
  document.getElementById('tabRecords').classList.toggle('active',tab==='records');
  document.getElementById('adminInventory').style.display=tab==='inventory'?'':'none';
  document.getElementById('adminRecords').style.display=tab==='records'?'':'none';
  if(tab==='records'){checkExpiry();renderRecords();}
  else{renderAdmin();renderStatsA();}
}

/* ---- STATS ---- */
function statHTML(t,h,l,o,na){
  return`<div class="stat"><div class="stat-label">總零件數</div><div class="stat-val">${t}</div></div>
<div class="stat"><div class="stat-label">庫存充足</div><div class="stat-val" style="color:#3B6D11">${h}</div></div>
<div class="stat"><div class="stat-label">偏低</div><div class="stat-val" style="color:#993C1D">${l}</div></div>
<div class="stat"><div class="stat-label">缺貨</div><div class="stat-val" style="color:#A32D2D">${o}</div></div>
<div class="stat"><div class="stat-label">無法訂購</div><div class="stat-val" style="color:#6b3fa0">${na}</div></div>`;
}
function renderStatsQ(){document.getElementById('statsQ').innerHTML=statHTML(parts.length,parts.filter(p=>p.level==='high').length,parts.filter(p=>p.level==='low').length,parts.filter(p=>p.level==='out').length,parts.filter(p=>p.level==='na').length);}
function renderStatsA(){document.getElementById('statsA').innerHTML=statHTML(parts.length,parts.filter(p=>p.level==='high').length,parts.filter(p=>p.level==='low').length,parts.filter(p=>p.level==='out').length,parts.filter(p=>p.level==='na').length);}

function matchWall(p,wall){if(!wall)return true;return p.locations&&p.locations.some(l=>l.wall===wall);}

/* ---- QUERY ---- */
function filterQ(){
  const q=document.getElementById('searchQ').value.trim().toLowerCase();
  const ser=document.getElementById('seriesQ').value;
  const wall=document.getElementById('wallQ').value;
  const lf=document.getElementById('levelQ').value;
  filteredQ=parts.filter(p=>{
    if(q&&!p.id.includes(q))return false;
    if(ser&&p.series!==ser)return false;
    if(!matchWall(p,wall))return false;
    if(lf&&p.level!==lf)return false;
    return true;
  });
}
function renderQuery(){pageQ=1;filterQ();drawQ();renderStatsQ();}
function changePageQ(d){pageQ+=d;drawQ();}
function drawQ(){
  const tbody=document.getElementById('tbodyQ');
  const noR=document.getElementById('noResultsQ');
  const searchVal=document.getElementById('searchQ').value.trim();
  if(!filteredQ.length){
    tbody.innerHTML='';noR.style.display='';
    document.getElementById('noResultsOrderBtn').dataset.searchId=searchVal;
    document.getElementById('pageInfoQ').textContent='';
    document.getElementById('prevQ').disabled=true;document.getElementById('nextQ').disabled=true;return;
  }
  noR.style.display='none';
  const tp=Math.ceil(filteredQ.length/PAGE_SIZE);
  if(pageQ>tp)pageQ=tp;
  const slice=filteredQ.slice((pageQ-1)*PAGE_SIZE,pageQ*PAGE_SIZE);
  tbody.innerHTML=slice.map(p=>{
    const canOrder=p.level==='out';
    const canReserve=p.level==='high'||p.level==='low';
    return`<tr>
      <td class="mono">${p.id}</td>
      <td>${p.series?`<span class="tag tag-series">${p.series}</span>`:'<span style="color:#bbb">—</span>'}</td>
      <td>${locHTML(p.locations)}</td>
      <td style="color:#555">${p.qty!==''?p.qty:'—'}</td>
      <td><span class="badge ${lvClass(p.level)}">${lvLabel(p.level)}</span></td>
      <td class="note-cell" title="${p.note||''}">${p.note||'—'}</td>
      <td><div class="actions-cell">
        ${canOrder?`<button class="btn-order" onclick="openOrderModal('${p.id}')">訂購</button>`:''}
        ${canReserve?`<button class="btn-reserve" onclick="openReserveModal('${p.id}')">預留</button>`:''}
      </div></td>
    </tr>`;
  }).join('');
  document.getElementById('pageInfoQ').textContent=`第 ${pageQ} / ${tp} 頁，共 ${filteredQ.length} 筆`;
  document.getElementById('prevQ').disabled=pageQ<=1;
  document.getElementById('nextQ').disabled=pageQ>=tp;
}

/* ---- ADMIN INVENTORY ---- */
function filterA(){
  const q=document.getElementById('searchA').value.trim().toLowerCase();
  const ser=document.getElementById('seriesA').value;
  const wall=document.getElementById('wallA').value;
  const lf=document.getElementById('levelA').value;
  filteredA=parts.map((p,i)=>({...p,_i:i})).filter(p=>{
    if(q&&!p.id.includes(q))return false;
    if(ser&&p.series!==ser)return false;
    if(!matchWall(p,wall))return false;
    if(lf&&p.level!==lf)return false;
    return true;
  });
}
function renderAdmin(){pageA=1;filterA();drawA();renderStatsA();}
function refreshAdminPage(){filterA();const tp=Math.ceil(filteredA.length/PAGE_SIZE);if(pageA>tp&&tp>0)pageA=tp;drawA();renderStatsA();}
function changePageA(d){pageA+=d;drawA();}
function drawA(){
  const tbody=document.getElementById('tbodyA');
  const noR=document.getElementById('noResultsA');
  if(!filteredA.length){tbody.innerHTML='';noR.style.display='';
    document.getElementById('pageInfoA').textContent='';
    document.getElementById('prevA').disabled=true;document.getElementById('nextA').disabled=true;return;}
  noR.style.display='none';
  const tp=Math.ceil(filteredA.length/PAGE_SIZE);
  if(pageA>tp)pageA=tp;
  const slice=filteredA.slice((pageA-1)*PAGE_SIZE,pageA*PAGE_SIZE);
  tbody.innerHTML=slice.map(p=>`<tr>
    <td class="mono">${p.id}</td>
    <td>${p.series?`<span class="tag tag-series">${p.series}</span>`:'<span style="color:#bbb">—</span>'}</td>
    <td>${locHTML(p.locations)}</td>
    <td style="color:#555">${p.qty!==''?p.qty:'—'}</td>
    <td>
      <select class="lv-select" onchange="quickLevel(${p._i},this.value)">
        <option value="high" ${p.level==='high'?'selected':''}>充足</option>
        <option value="low"  ${p.level==='low' ?'selected':''}>偏低</option>
        <option value="out"  ${p.level==='out' ?'selected':''}>缺貨</option>
        <option value="na"   ${p.level==='na'  ?'selected':''}>缺貨無法訂購</option>
      </select>
      <span class="badge ${lvClass(p.level)}" style="margin-left:5px">${lvLabel(p.level)}</span>
    </td>
    <td class="note-cell" title="${p.note||''}">${p.note||'—'}</td>
    <td><div class="actions-cell">
      <button class="btn btn-sm" onclick="openEditModal(${p._i})">編輯</button>
      <button class="btn btn-sm btn-danger" onclick="openDelModal(${p._i})">刪除</button>
    </div></td>
  </tr>`).join('');
  document.getElementById('pageInfoA').textContent=`第 ${pageA} / ${tp} 頁，共 ${filteredA.length} 筆`;
  document.getElementById('prevA').disabled=pageA<=1;
  document.getElementById('nextA').disabled=pageA>=tp;
}

function quickLevel(i,val){
  parts[i].level=val;save();
  filterA();const tp=Math.ceil(filteredA.length/PAGE_SIZE);if(pageA>tp&&tp>0)pageA=tp;
  drawA();renderStatsA();
}

/* ---- RECORDS TAB ---- */
function filterRec(){
  const q=document.getElementById('searchRec').value.trim().toLowerCase();
  const type=document.getElementById('recTypeFilter').value;
  const status=document.getElementById('recStatusFilter').value;
  filteredRec=orders.map((o,i)=>({...o,_i:i})).filter(o=>{
    if(q&&!o.partId.includes(q)&&!(o.name||'').toLowerCase().includes(q))return false;
    if(type&&o.type!==type)return false;
    if(status&&o.status!==status)return false;
    return true;
  }).reverse();
}
function renderRecords(){pageRec=1;filterRec();drawRec();updateExpiredBadge();}
function changePageRec(d){pageRec+=d;drawRec();}
function drawRec(){
  const tbody=document.getElementById('tbodyRec');
  const noR=document.getElementById('noResultsRec');
  if(!filteredRec.length){tbody.innerHTML='';noR.style.display='';
    document.getElementById('pageInfoRec').textContent='';
    document.getElementById('prevRec').disabled=true;document.getElementById('nextRec').disabled=true;return;}
  noR.style.display='none';
  const tp=Math.ceil(filteredRec.length/PAGE_SIZE);
  if(pageRec>tp)pageRec=tp;
  const slice=filteredRec.slice((pageRec-1)*PAGE_SIZE,pageRec*PAGE_SIZE);
  tbody.innerHTML=slice.map(o=>{
    const isExpired=o.status==='逾期未領';
    const isNotified=o.status==='已通知';
    const dl=isNotified&&o.notifiedDate?daysLeft(o.notifiedDate):null;
    const rowClass=isExpired?'row-expired':isNotified?'row-notified':'';
    const typeColor=o.type==='訂購'?'#A32D2D':'#3C3489';
    const expiryCell=isNotified&&dl!==null?`<span class="days-left">剩 ${dl} 天</span>`:
      isExpired&&o.notifiedDate?`<span style="color:#A32D2D;font-size:11px">已逾期</span>`:'—';
    const notifiedDateDisplay=o.notifiedDate||'—';
    return`<tr class="${rowClass}">
      <td><span style="color:${typeColor};font-weight:600;font-size:12px">${o.type}</span></td>
      <td class="mono">${o.partId}</td>
      <td>${o.series||'—'}</td>
      <td>${o.qty}</td>
      <td>${o.name||'—'}</td>
      <td>${o.phone||'—'}</td>
      <td>${o.csc||'—'}</td>
      <td style="white-space:nowrap;font-size:12px">${o.date||'—'}</td>
      <td>
        <select class="status-select" onchange="updateRecStatus(${o._i},this.value)">
          <option value="已訂購" ${o.status==='已訂購'?'selected':''}>已訂購</option>
          <option value="已到貨" ${o.status==='已到貨'?'selected':''}>已到貨</option>
          <option value="已預留" ${o.status==='已預留'?'selected':''}>已預留</option>
          <option value="已通知" ${o.status==='已通知'?'selected':''}>已通知</option>
          <option value="逾期未領" ${o.status==='逾期未領'?'selected':''}>逾期未領</option>
          <option value="缺貨無法訂購" ${o.status==='缺貨無法訂購'?'selected':''}>缺貨無法訂購</option>
        </select>
        <span class="badge ${statusClass(o.status)}" style="margin-left:4px;font-size:10px">${o.status}</span>
      </td>
      <td style="font-size:12px;white-space:nowrap">${expiryCell}</td>
      <td class="note-cell" title="${o.note||''}">${o.note||'—'}</td>
      <td><div class="actions-cell">
        <button class="btn btn-sm" onclick="openRecNoteModal(${o._i})">備註</button>
        <button class="btn btn-sm btn-danger" onclick="deleteRec(${o._i})">刪除</button>
      </div></td>
    </tr>`;
  }).join('');
  document.getElementById('pageInfoRec').textContent=`第 ${pageRec} / ${tp} 頁，共 ${filteredRec.length} 筆`;
  document.getElementById('prevRec').disabled=pageRec<=1;
  document.getElementById('nextRec').disabled=pageRec>=tp;
}

function updateRecStatus(i,val){
  orders[i].status=val;
  if(val==='已通知'&&!orders[i].notifiedDate){orders[i].notifiedDate=todayISO();}
  if(val!=='已通知'&&val!=='逾期未領'){orders[i].notifiedDate=null;}
  saveOrders();filterRec();const tp=Math.ceil(filteredRec.length/PAGE_SIZE);if(pageRec>tp&&tp>0)pageRec=tp;
  drawRec();updateExpiredBadge();
}

function deleteRec(i){
  orders.splice(i,1);saveOrders();renderRecords();showToast('已刪除紀錄');
}

function openRecNoteModal(i){
  editRecNoteIdx=i;
  document.getElementById('recNoteInput').value=orders[i].note||'';
  document.getElementById('recNoteModal').classList.add('show');
}
function closeRecNoteModal(){document.getElementById('recNoteModal').classList.remove('show');editRecNoteIdx=null;}
function saveRecNote(){
  if(editRecNoteIdx===null)return;
  orders[editRecNoteIdx].note=document.getElementById('recNoteInput').value.trim();
  saveOrders();closeRecNoteModal();filterRec();drawRec();showToast('備註已更新');
}

/* ---- PART MODAL ---- */
function toggleLoc(type){
  const chk=document.getElementById('chk'+type);
  const box=document.getElementById('loc'+type);
  if(box)box.style.display=chk.checked?'':'none';
}
function resetModal(){
  ['MPU','ASIS','ASISB'].forEach(t=>{
    document.getElementById('chk'+t).checked=false;
    const b=document.getElementById('loc'+t);
    if(b)b.style.display='none';
  });
  document.getElementById('fZoneMPUarea').value='';
  document.getElementById('fZoneMPUrow').value='';
  document.getElementById('fASISrow').value='';
  document.getElementById('fASIScol').value='';
}
function openAddModal(){
  editIdx=null;
  document.getElementById('modalTitle').textContent='新增零件';
  document.getElementById('fId').value='';document.getElementById('fId').disabled=false;
  document.getElementById('fSeries').value='';
  resetModal();
  document.getElementById('fQty').value='';
  document.getElementById('fLevel').value='high';
  document.getElementById('fNote').value='';
  document.getElementById('modalErr').textContent='';
  document.getElementById('partModal').classList.add('show');
}
function openEditModal(i){
  editIdx=i;savedPageA=pageA;
  const p=parts[i];
  document.getElementById('modalTitle').textContent='編輯零件 — '+p.id;
  document.getElementById('fId').value=p.id;document.getElementById('fId').disabled=true;
  document.getElementById('fSeries').value=p.series||'';
  resetModal();
  (p.locations||[]).forEach(l=>{
    if(l.wall==='MPU'){
      document.getElementById('chkMPU').checked=true;document.getElementById('locMPU').style.display='';
      const m=(l.zone||'').match(/^(.+?)第(\d+)排$/);
      if(m){document.getElementById('fZoneMPUarea').value=m[1];document.getElementById('fZoneMPUrow').value=m[2];}
      else{document.getElementById('fZoneMPUarea').value=l.zone||'';}
    }
    if(l.wall==='ASIS'){
      document.getElementById('chkASIS').checked=true;document.getElementById('locASIS').style.display='';
      const m=(l.zone||'').match(/^(\d+)排(\d+)$/);
      if(m){document.getElementById('fASISrow').value=m[1];document.getElementById('fASIScol').value=m[2];}
    }
    if(l.wall==='ASIS背面'){document.getElementById('chkASISB').checked=true;}
  });
  document.getElementById('fQty').value=p.qty||'';
  document.getElementById('fLevel').value=p.level||'high';
  document.getElementById('fNote').value=p.note||'';
  document.getElementById('modalErr').textContent='';
  document.getElementById('partModal').classList.add('show');
}
function closeModal(){document.getElementById('partModal').classList.remove('show');}

function buildMPUzone(){
  const area=document.getElementById('fZoneMPUarea').value;
  const row=document.getElementById('fZoneMPUrow').value.trim();
  if(!area)return'';
  return row?`${area}第${row}排`:area;
}

function savePart(){
  const id=document.getElementById('fId').value.trim();
  if(!id){document.getElementById('modalErr').textContent='零件編號為必填';return;}
  if(editIdx===null&&parts.find(p=>p.id===id)){document.getElementById('modalErr').textContent='此零件編號已存在';return;}
  const locs=[];
  if(document.getElementById('chkMPU').checked)locs.push({wall:'MPU',zone:buildMPUzone()});
  if(document.getElementById('chkASIS').checked){
    const r=document.getElementById('fASISrow').value,c=document.getElementById('fASIScol').value;
    locs.push({wall:'ASIS',zone:(r&&c)?`${r}排${c}`:''});
  }
  if(document.getElementById('chkASISB').checked)locs.push({wall:'ASIS背面',zone:''});
  const obj={id,series:document.getElementById('fSeries').value.trim(),locations:locs,
    qty:document.getElementById('fQty').value,level:document.getElementById('fLevel').value,
    note:document.getElementById('fNote').value.trim()};
  const isNew=editIdx===null;
  if(isNew)parts.push(obj);else parts[editIdx]=obj;
  save();updateSeriesDropdowns();closeModal();
  filterA();
  if(isNew){pageA=Math.ceil(filteredA.length/PAGE_SIZE)||1;}
  else{const tp=Math.ceil(filteredA.length/PAGE_SIZE);pageA=Math.min(savedPageA,tp||1);}
  drawA();renderStatsA();
  document.getElementById('savedTitle').textContent=isNew?'新增成功':'儲存成功';
  document.getElementById('savedMsg').textContent=`零件 ${id} 已${isNew?'新增':'更新'}`;
  document.getElementById('savedModal').classList.add('show');
}
function closeSavedModal(){document.getElementById('savedModal').classList.remove('show');}

/* ---- DELETE ---- */
function openDelModal(i){deleteIdx=i;document.getElementById('delPartId').textContent=parts[i].id;document.getElementById('delModal').classList.add('show');}
function closeDelModal(){document.getElementById('delModal').classList.remove('show');deleteIdx=null;}
function confirmDelete(){
  if(deleteIdx===null)return;
  const id=parts[deleteIdx].id,cur=pageA;
  parts.splice(deleteIdx,1);save();closeDelModal();updateSeriesDropdowns();
  filterA();const tp=Math.ceil(filteredA.length/PAGE_SIZE);pageA=Math.min(cur,tp||1);drawA();renderStatsA();showToast('已刪除 '+id);
}

/* ---- ORDER / RESERVE ---- */
function openOrderModal(id){
  const p=parts.find(x=>x.id===id);
  document.getElementById('orderPartId').textContent=id;
  document.getElementById('oPartId').value=id;
  document.getElementById('oSeries').value=p?p.series||'—':'—';
  document.getElementById('oDate').value=todayStr();
  document.getElementById('oQty').value='';document.getElementById('oName').value='';
  document.getElementById('oPhone').value='';document.getElementById('oNote').value='';
  document.getElementById('orderErr').textContent='';
  document.getElementById('orderModal').classList.add('show');
}
function openOrderModalUnknown(){
  const searchId=document.getElementById('noResultsOrderBtn').dataset.searchId||'';
  openOrderModal(searchId);
  document.getElementById('oPartId').disabled=false;
}
function closeOrderModal(){document.getElementById('orderModal').classList.remove('show');}

function openReserveModal(id){
  const p=parts.find(x=>x.id===id);
  document.getElementById('reservePartId').textContent=id;
  document.getElementById('rPartId').value=id;
  document.getElementById('rSeries').value=p?p.series||'—':'—';
  document.getElementById('rDate').value=todayStr();
  document.getElementById('rQty').value='';document.getElementById('rName').value='';
  document.getElementById('rPhone').value='';document.getElementById('rCSC').value='';document.getElementById('rNote').value='';
  document.getElementById('reserveErr').textContent='';
  document.getElementById('reserveModal').classList.add('show');
}
function closeReserveModal(){document.getElementById('reserveModal').classList.remove('show');}

function submitOrder(){
  const qty=document.getElementById('oQty').value,name=document.getElementById('oName').value.trim();
  if(!qty||parseInt(qty)<1){document.getElementById('orderErr').textContent='請輸入訂購數量';return;}
  if(!name){document.getElementById('orderErr').textContent='請輸入聯絡人姓名';return;}
  const partId=document.getElementById('oPartId').value.trim();
  if(!partId){document.getElementById('orderErr').textContent='請輸入零件編號';return;}
  orders.push({type:'訂購',partId,series:document.getElementById('oSeries').value,
    qty,date:todayStr(),name,phone:document.getElementById('oPhone').value,
    note:document.getElementById('oNote').value,status:'未處理',notifiedDate:null,time:new Date().toISOString()});
  saveOrders();closeOrderModal();updateExpiredBadge();
  document.getElementById('savedTitle').textContent='訂購單已送出';
  document.getElementById('savedMsg').textContent=`零件 ${partId}，數量 ${qty}，聯絡人 ${name}`;
  document.getElementById('savedModal').classList.add('show');
}

function submitReserve(){
  const qty=document.getElementById('rQty').value,name=document.getElementById('rName').value.trim();
  if(!qty||parseInt(qty)<1){document.getElementById('reserveErr').textContent='請輸入預留數量';return;}
  if(!name){document.getElementById('reserveErr').textContent='請輸入聯絡人姓名';return;}
  orders.push({type:'預留',partId:document.getElementById('rPartId').value,series:document.getElementById('rSeries').value,
    qty,date:todayStr(),name,phone:document.getElementById('rPhone').value,
    csc:document.getElementById('rCSC').value,note:document.getElementById('rNote').value,
    status:'已預留',notifiedDate:null,time:new Date().toISOString()});
  saveOrders();closeReserveModal();updateExpiredBadge();
  document.getElementById('savedTitle').textContent='預留單已送出';
  document.getElementById('savedMsg').textContent=`零件 ${document.getElementById('rPartId').value}，數量 ${qty}，聯絡人 ${name}`;
  document.getElementById('savedModal').classList.add('show');
}

function printForm(type){
  const isO=type==='order';
  const id=isO?document.getElementById('oPartId').value:document.getElementById('rPartId').value;
  const series=isO?document.getElementById('oSeries').value:document.getElementById('rSeries').value;
  const qty=isO?document.getElementById('oQty').value:document.getElementById('rQty').value;
  const date=isO?document.getElementById('oDate').value:document.getElementById('rDate').value;
  const name=isO?document.getElementById('oName').value:document.getElementById('rName').value;
  const phone=isO?document.getElementById('oPhone').value:document.getElementById('rPhone').value;
  const note=isO?document.getElementById('oNote').value:document.getElementById('rNote').value;
  const csc=isO?'':document.getElementById('rCSC').value;
  const title=isO?'零件訂購單':'零件預留單';
  const w=window.open('','_blank');
  w.document.write(`<!DOCTYPE html><html><head><meta charset="UTF-8"><title>${title}</title>
  <style>body{font-family:Arial,sans-serif;padding:40px;max-width:600px;margin:0 auto;}
  h1{font-size:20px;border-bottom:2px solid #000;padding-bottom:10px;margin-bottom:24px;}
  .row{display:flex;margin-bottom:12px;font-size:14px;}
  .label{width:100px;font-weight:bold;color:#555;flex-shrink:0;}
  .val{flex:1;border-bottom:1px solid #ccc;padding-bottom:4px;min-height:22px;}
  .footer{margin-top:40px;font-size:12px;color:#888;}
  @media print{body{padding:20px;}}</style></head>
  <body><h1>${title}</h1>
  <div class="row"><span class="label">零件編號</span><span class="val">${id}</span></div>
  <div class="row"><span class="label">系列</span><span class="val">${series}</span></div>
  <div class="row"><span class="label">數量</span><span class="val">${qty}</span></div>
  <div class="row"><span class="label">日期</span><span class="val">${date}</span></div>
  <div class="row"><span class="label">聯絡人</span><span class="val">${name}</span></div>
  <div class="row"><span class="label">電話</span><span class="val">${phone}</span></div>
  ${!isO?`<div class="row"><span class="label">CSC處理人</span><span class="val">${csc}</span></div>`:''}
  <div class="row"><span class="label">備註</span><span class="val">${note}</span></div>
  <div class="footer">列印時間：${new Date().toLocaleString('zh-TW')}</div>
  <script>window.onload=function(){window.print();}<\/script></body></html>`);
  w.document.close();
}

/* ---- EXPORT ---- */
function exportCSV(){
  const rows=[['零件編號','系列','架位','數量','庫存水位','備註']];
  parts.forEach(p=>{
    const locs=(p.locations||[]).map(l=>l.zone?`${l.wall}-${l.zone}`:l.wall).join(' / ');
    rows.push([p.id,p.series,locs,p.qty,lvLabel(p.level),p.note]);
  });
  const csv='\uFEFF'+rows.map(r=>r.map(c=>`"${c||''}"`).join(',')).join('\n');
  const a=document.createElement('a');
  a.href='data:text/csv;charset=utf-8,'+encodeURIComponent(csv);
  a.download='MPU零件庫存_'+new Date().toLocaleDateString('zh-TW').replace(/\//g,'-')+'.csv';
  a.click();showToast('已匯出 CSV ✓');
}
function exportRecordsCSV(){
  const rows=[['類型','零件編號','系列','數量','聯絡人','電話','CSC處理人','日期','狀態','通知日期','備註']];
  orders.forEach(o=>rows.push([o.type,o.partId,o.series||'',o.qty,o.name||'',o.phone||'',o.csc||'',o.date||'',o.status,o.notifiedDate||'',o.note||'']));
  const csv='\uFEFF'+rows.map(r=>r.map(c=>`"${c||''}"`).join(',')).join('\n');
  const a=document.createElement('a');
  a.href='data:text/csv;charset=utf-8,'+encodeURIComponent(csv);
  a.download='MPU訂購預留紀錄_'+new Date().toLocaleDateString('zh-TW').replace(/\//g,'-')+'.csv';
  a.click();showToast('已匯出紀錄 CSV ✓');
}

/* ---- INIT ---- */
load();
updateSeriesDropdowns();
filteredQ=[...parts];drawQ();renderStatsQ();
</script>
</body>
</html>
