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
.stats{display:grid;grid-template-columns:repeat(auto-fit,minmax(110px,1fr));gap:12px;margin-bottom:20px;}
.stat{background:#fff;border-radius:10px;border:1px solid #e8e8e8;padding:14px 16px;}
.stat-label{font-size:12px;color:#888;margin-bottom:6px;}
.stat-val{font-size:24px;font-weight:600;}
.toolbar{display:flex;gap:10px;flex-wrap:wrap;margin-bottom:14px;align-items:center;}
.toolbar input,.toolbar select{border:1px solid #ddd;background:#fff;color:#1a1a1a;border-radius:8px;padding:8px 12px;font-size:14px;outline:none;}
.toolbar input{flex:1;min-width:160px;}
.toolbar input:focus,.toolbar select:focus{border-color:#888;}
.tab-bar{display:flex;gap:0;border-bottom:2px solid #e8e8e8;margin-bottom:18px;}
.tab-btn{background:none;border:none;padding:10px 20px;font-size:14px;cursor:pointer;color:#888;border-bottom:2px solid transparent;margin-bottom:-2px;transition:all 0.15s;}
.tab-btn.active{color:#1a1a1a;font-weight:600;border-bottom:2px solid #1a1a1a;}
.badge-count{display:inline-block;background:#FCEBEB;color:#A32D2D;font-size:10px;font-weight:600;padding:1px 6px;border-radius:20px;margin-left:5px;}
.table-wrap{background:#fff;border-radius:12px;border:1px solid #e8e8e8;overflow:hidden;}
table{width:100%;border-collapse:collapse;font-size:13px;}
th{text-align:left;padding:10px 14px;color:#888;font-weight:500;border-bottom:1px solid #f0f0f0;font-size:12px;background:#fafafa;white-space:nowrap;}
td{padding:9px 14px;border-bottom:1px solid #f5f5f5;vertical-align:middle;}
tr:last-child td{border-bottom:none;}
tr:hover td{background:#fafafa;}
tr.row-expired td{background:#fff5f5!important;}
tr.row-notified td{background:#fffbf0!important;}
tr.order-row{background:#fafafa;}
tr.order-row td{padding:12px 14px;}
tr.item-row td{padding:7px 14px 7px 28px;font-size:12px;background:#fff;}
tr.item-row:hover td{background:#f9f9f9;}
.mono{font-family:"SF Mono","Consolas",monospace;font-size:13px;font-weight:500;}
.tag{display:inline-block;padding:2px 8px;border-radius:20px;font-size:11px;font-weight:500;white-space:nowrap;}
.tag-mpu{background:#EEEDFE;color:#3C3489;}
.tag-asis{background:#E1F5EE;color:#0F6E56;}
.tag-asisb{background:#FAEEDA;color:#854F0B;}
.tag-series{background:#f0f0f0;color:#555;}
.tag-order{background:#FCEBEB;color:#A32D2D;}
.tag-reserve{background:#EEEDFE;color:#3C3489;}
.locations{display:flex;flex-wrap:wrap;gap:4px;}
.badge{display:inline-block;padding:2px 10px;border-radius:20px;font-size:11px;font-weight:500;}
.lv-high{background:#EAF3DE;color:#3B6D11;}
.lv-low{background:#FAECE7;color:#993C1D;}
.lv-out{background:#FCEBEB;color:#A32D2D;}
.lv-na{background:#f0e8f8;color:#6b3fa0;}
.lv-select{border:1px solid #ddd;background:#fff;color:#1a1a1a;border-radius:8px;padding:3px 8px;font-size:12px;cursor:pointer;outline:none;}
.note-cell{max-width:100px;overflow:hidden;text-overflow:ellipsis;white-space:nowrap;color:#888;font-size:12px;}
.part-actions{display:flex;align-items:center;gap:6px;flex-wrap:wrap;}
.qty-input{width:52px;border:1px solid #ddd;border-radius:6px;padding:3px 6px;font-size:12px;text-align:center;}
.btn-add-cart{background:#1a1a1a;color:#fff;border:none;border-radius:6px;padding:4px 10px;font-size:11px;cursor:pointer;white-space:nowrap;}
.btn-add-cart:hover{background:#333;}
.btn-reserve-add{background:#EEEDFE;color:#3C3489;border:1px solid #ceccf6;border-radius:6px;padding:4px 10px;font-size:11px;cursor:pointer;white-space:nowrap;}
.btn-reserve-add:hover{background:#d8d6f6;}
.out-hint{font-size:11px;color:#A32D2D;background:#fff5f5;border:1px solid #f5c1c1;border-radius:6px;padding:3px 8px;white-space:nowrap;}
.pagination{display:flex;gap:8px;align-items:center;justify-content:flex-end;margin-top:14px;font-size:13px;color:#888;}
.pagination button{background:#fff;border:1px solid #ddd;border-radius:8px;padding:5px 14px;cursor:pointer;font-size:13px;color:#333;}
.pagination button:hover:not(:disabled){background:#f5f5f3;}
.pagination button:disabled{opacity:0.4;cursor:default;}
.no-results{text-align:center;padding:40px;color:#aaa;font-size:14px;}
.no-results-hint{font-size:13px;color:#888;margin:8px 0 12px;line-height:1.6;}
.btn{border:1px solid #ddd;background:#fff;border-radius:8px;padding:7px 16px;font-size:13px;cursor:pointer;color:#333;transition:background 0.1s;}
.btn:hover{background:#f5f5f3;}
.btn-primary{background:#1a1a1a;color:#fff;border-color:#1a1a1a;}
.btn-primary:hover{background:#333;}
.btn-danger{color:#A32D2D;border-color:#f5c1c1;}
.btn-danger:hover{background:#FCEBEB;}
.btn-sm{padding:4px 10px;font-size:12px;}
.cart-bar{background:#1a1a1a;color:#fff;padding:10px 20px;display:flex;align-items:center;justify-content:space-between;position:sticky;bottom:0;z-index:40;}
.cart-bar-left{font-size:13px;}
.cart-bar span{font-weight:600;margin:0 4px;}
.cart-btn{background:#fff;color:#1a1a1a;border:none;border-radius:8px;padding:7px 16px;font-size:13px;cursor:pointer;font-weight:600;}
.cart-btn:hover{background:#f0f0f0;}
.status-select{border:1px solid #ddd;background:#fff;color:#1a1a1a;border-radius:8px;padding:3px 6px;font-size:11px;cursor:pointer;outline:none;min-width:90px;}
.st-pending{background:#f0f0f0;color:#555;}
.st-ordered{background:#E1F5EE;color:#0F6E56;}
.st-arrived{background:#EAF3DE;color:#3B6D11;}
.st-reserved{background:#EEEDFE;color:#3C3489;}
.st-notified{background:#FAEEDA;color:#854F0B;}
.st-expired{background:#FCEBEB;color:#A32D2D;}
.st-unavail{background:#f0e8f8;color:#6b3fa0;}
.overlay{display:none;position:fixed;inset:0;background:rgba(0,0,0,0.4);z-index:100;align-items:center;justify-content:center;}
.overlay.show{display:flex;}
.modal{background:#fff;border-radius:14px;padding:28px;width:100%;max-width:500px;box-shadow:0 8px 32px rgba(0,0,0,0.12);max-height:90vh;overflow-y:auto;}
.modal h2{font-size:17px;font-weight:600;margin-bottom:20px;}
.form-row{margin-bottom:14px;}
.form-row label{display:block;font-size:12px;color:#888;margin-bottom:5px;font-weight:500;}
.form-row input,.form-row select,.form-row textarea{width:100%;border:1px solid #ddd;border-radius:8px;padding:9px 12px;font-size:14px;outline:none;background:#fff;color:#1a1a1a;font-family:inherit;}
.form-row input:disabled{background:#f5f5f3;color:#888;}
.form-row textarea{resize:vertical;min-height:55px;}
.form-row-2{display:grid;grid-template-columns:1fr 1fr;gap:12px;margin-bottom:14px;}
.loc-block{border:1px solid #eee;border-radius:10px;padding:14px;margin-bottom:10px;background:#fafafa;}
.loc-check{display:flex;align-items:center;gap:8px;margin-bottom:10px;}
.loc-check input[type=checkbox]{width:16px;height:16px;cursor:pointer;accent-color:#1a1a1a;}
.loc-check label{font-size:13px;font-weight:500;cursor:pointer;}
.loc-row{display:flex;gap:8px;align-items:center;margin-bottom:8px;}
.loc-row label{font-size:12px;color:#888;width:40px;flex-shrink:0;}
.loc-row input,.loc-row select{flex:1;border:1px solid #ddd;border-radius:8px;padding:7px 10px;font-size:13px;outline:none;background:#fff;color:#1a1a1a;}
.modal-footer{display:flex;gap:10px;justify-content:flex-end;margin-top:20px;}
.section-divider{font-size:12px;font-weight:600;color:#888;margin:16px 0 8px;text-transform:uppercase;letter-spacing:0.5px;}
.login-wrap{display:flex;align-items:center;justify-content:center;min-height:calc(100vh - 52px);}
.login-card{background:#fff;border-radius:14px;border:1px solid #e8e8e8;padding:32px;width:100%;max-width:340px;text-align:center;}
.login-card h2{font-size:18px;font-weight:600;margin-bottom:8px;}
.login-card p{font-size:13px;color:#888;margin-bottom:24px;}
.login-card input{width:100%;border:1px solid #ddd;border-radius:8px;padding:10px 14px;font-size:15px;outline:none;text-align:center;letter-spacing:4px;margin-bottom:14px;}
.login-err{color:#A32D2D;font-size:13px;margin-top:8px;min-height:20px;}
.toast{position:fixed;bottom:80px;right:24px;background:#1a1a1a;color:#fff;padding:10px 18px;border-radius:10px;font-size:13px;opacity:0;transition:opacity 0.3s;pointer-events:none;z-index:999;}
.toast.show{opacity:1;}
.page{display:none;}
.page.active{display:block;}
.cart-item-row{display:flex;align-items:center;gap:8px;padding:9px 0;border-bottom:1px solid #f0f0f0;}
.cart-item-row:last-child{border-bottom:none;}
.cart-item-id{font-family:"SF Mono","Consolas",monospace;font-size:13px;font-weight:500;min-width:90px;}
.cart-item-type{font-size:11px;padding:2px 7px;border-radius:20px;font-weight:500;}
.cart-item-qty-input{width:52px;border:1px solid #ddd;border-radius:6px;padding:4px 6px;font-size:13px;text-align:center;}
.cart-item-price{font-size:12px;color:#888;min-width:50px;text-align:right;}
.cart-item-remove{background:none;border:none;color:#A32D2D;cursor:pointer;font-size:16px;padding:0 4px;margin-left:auto;}
.cart-total{display:flex;justify-content:space-between;align-items:center;padding:12px 0 0;border-top:1px solid #e8e8e8;margin-top:8px;font-weight:600;font-size:15px;}
.order-expand-btn{background:none;border:none;cursor:pointer;font-size:12px;color:#3C3489;padding:0;text-decoration:underline;}
.price-input{width:70px;border:1px solid #ddd;border-radius:6px;padding:3px 6px;font-size:12px;text-align:center;}
.order-total-cell{font-weight:600;color:#1a1a1a;}
.notified-date-input{border:1px solid #ddd;border-radius:6px;padding:3px 6px;font-size:12px;width:120px;}
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
          <th>零件編號</th><th>系列</th><th>架位</th><th>數量</th><th>庫存水位</th><th>備註</th><th></th>
        </tr></thead>
        <tbody id="tbodyQ"></tbody>
      </table>
      <div id="noResultsQ" class="no-results" style="display:none">
        <p>查無符合的零件</p>
        <div class="no-results-hint">此零件目前不在系統庫存內，您仍可填寫訂購單，<br>工作人員將盡快為您確認並回覆。</div>
        <div style="display:flex;align-items:center;gap:8px;justify-content:center;flex-wrap:wrap">
          <input type="number" id="noResultsQty" min="1" value="1" class="qty-input" style="width:60px">
          <button class="btn-add-cart" onclick="addUnknownToCart()">+ 填寫訂購單加入購物車</button>
        </div>
      </div>
    </div>
    <div class="pagination">
      <span id="pageInfoQ"></span>
      <button id="prevQ" onclick="changePageQ(-1)">← 上一頁</button>
      <button id="nextQ" onclick="changePageQ(1)">下一頁 →</button>
    </div>
  </div>
  <div class="cart-bar" id="cartBar" style="display:none">
    <div class="cart-bar-left">購物車 <span id="cartCount">0</span> 項・共 $<span id="cartTotal">0</span></div>
    <button class="cart-btn" onclick="openCartModal()">查看購物車 →</button>
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

    <!-- 庫存管理 -->
    <div id="adminInventory">
      <div class="stats" id="statsA"></div>
      <div class="toolbar">
        <input type="text" id="searchA" placeholder="搜尋零件編號..." oninput="renderAdmin()">
        <select id="seriesA" onchange="renderAdmin()"><option value="">全部系列</option></select>
        <select id="wallA" onchange="renderAdmin()">
          <option value="">全部位置</option>
          <option value="MPU">MPU</option><option value="ASIS">ASIS</option><option value="ASIS背面">ASIS背面</option>
        </select>
        <select id="levelA" onchange="renderAdmin()">
          <option value="">全部水位</option>
          <option value="high">充足</option><option value="low">偏低</option>
          <option value="out">缺貨</option><option value="na">缺貨無法訂購</option>
        </select>
        <button class="btn btn-primary" onclick="openAddModal()">＋ 新增零件</button>
        <button class="btn" onclick="exportCSV()">匯出 CSV</button>
        <button class="btn" onclick="logout()" style="margin-left:auto">登出</button>
      </div>
      <div class="table-wrap">
        <table>
          <thead><tr>
            <th>零件編號</th><th>系列</th><th>架位</th><th>數量</th><th>庫存水位</th><th>單價($)</th><th>備註</th><th></th>
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

    <!-- 訂購紀錄 -->
    <div id="adminRecords" style="display:none">
      <div class="toolbar">
        <input type="text" id="searchRec" placeholder="搜尋零件編號或電話..." oninput="renderRecords()">
        <select id="recTypeFilter" onchange="renderRecords()">
          <option value="">全部類型</option>
          <option value="訂購">訂購單</option><option value="預留">預留單</option>
        </select>
        <select id="recMonthFilter" onchange="renderRecords()"><option value="">全部月份</option></select>
        <select id="recStatusFilter" onchange="renderRecords()">
          <option value="">全部狀態</option>
          <option value="未處理">未處理</option><option value="已訂購">已訂購</option>
          <option value="已到貨">已到貨</option><option value="已預留">已預留</option>
          <option value="已通知">已通知</option><option value="逾期未領">逾期未領</option>
          <option value="缺貨無法訂購">缺貨無法訂購</option>
        </select>
        <button class="btn" onclick="exportRecordsCSV()">匯出紀錄 CSV</button>
      </div>
      <div class="table-wrap">
        <table id="recTable">
          <thead><tr>
            <th></th><th>類型</th><th>聯絡人</th><th>電話</th><th>客服處理人</th><th>日期</th><th>狀態</th><th>通知日</th><th>到期日</th><th>總金額</th><th>備註</th><th></th>
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
      <div class="form-row" style="margin-bottom:0"><label>零件編號 *</label><input type="text" id="fId" placeholder="例：110019"></div>
      <div class="form-row" style="margin-bottom:0"><label>系列</label><input type="text" id="fSeries" placeholder="例：沙發系列"></div>
    </div>
    <div class="section-divider" style="margin-top:16px">架位設定</div>
    <div class="loc-block">
      <div class="loc-check"><input type="checkbox" id="chkMPU" onchange="toggleLoc('MPU')"><label for="chkMPU">MPU 零件牆</label></div>
      <div id="locMPU" style="display:none">
        <div class="loc-row"><label>區域</label>
          <select id="fZoneMPUarea">
            <option value="">請選擇</option>
            <option value="A區">A區</option><option value="B區">B區</option><option value="C區">C區</option>
            <option value="D區">D區</option><option value="E區">E區</option><option value="F區">F區</option>
            <option value="G區">G區</option><option value="H區">H區</option><option value="小房間">小房間</option>
          </select>
        </div>
        <div class="loc-row"><label>第幾排</label><input type="number" id="fZoneMPUrow" placeholder="例：3" min="1" style="max-width:120px"><span style="font-size:12px;color:#888;margin-left:4px">排（選填）</span></div>
      </div>
    </div>
    <div class="loc-block">
      <div class="loc-check"><input type="checkbox" id="chkASIS" onchange="toggleLoc('ASIS')"><label for="chkASIS">ASIS 零件牆</label></div>
      <div id="locASIS" style="display:none">
        <div class="loc-row"><label>排</label>
          <select id="fASISrow"><option value="">請選擇</option>
            <option value="1">1排</option><option value="2">2排</option><option value="3">3排</option><option value="4">4排</option>
            <option value="5">5排</option><option value="6">6排</option><option value="7">7排</option><option value="8">8排</option>
            <option value="9">9排</option><option value="10">10排</option><option value="11">11排</option><option value="12">12排</option>
          </select>
        </div>
        <div class="loc-row"><label>格</label>
          <select id="fASIScol"><option value="">請選擇</option>
            <option value="1">1</option><option value="2">2</option><option value="3">3</option><option value="4">4</option>
            <option value="5">5</option><option value="6">6</option><option value="7">7</option><option value="8">8</option>
            <option value="9">9</option><option value="10">10</option><option value="11">11</option><option value="12">12</option>
            <option value="13">13</option><option value="14">14</option><option value="15">15</option><option value="16">16</option>
          </select>
        </div>
      </div>
    </div>
    <div class="loc-block">
      <div class="loc-check"><input type="checkbox" id="chkASISB" onchange="toggleLoc('ASISB')"><label for="chkASISB">ASIS 背面</label></div>
    </div>
    <div class="form-row-2" style="margin-top:4px">
      <div class="form-row" style="margin-bottom:0"><label>數量</label><input type="number" id="fQty" placeholder="0" min="0"></div>
      <div class="form-row" style="margin-bottom:0"><label>庫存水位</label>
        <select id="fLevel">
          <option value="high">充足</option><option value="low">偏低</option>
          <option value="out">缺貨</option><option value="na">缺貨無法訂購</option>
        </select>
      </div>
    </div>
    <div class="form-row" style="margin-top:14px"><label>備註</label><textarea id="fNote" placeholder="選填"></textarea></div>
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

<!-- 購物車 Modal -->
<div class="overlay" id="cartModal">
  <div class="modal" style="max-width:500px;">
    <h2 style="margin-bottom:16px">購物車</h2>
    <div id="cartItemsWrap" style="max-height:260px;overflow-y:auto;margin-bottom:12px;"></div>
    <div id="cartEmpty" style="text-align:center;color:#aaa;padding:20px;display:none">購物車是空的</div>
    <div id="cartTotalRow" class="cart-total" style="display:none">
      <span>合計</span><span id="cartTotalDisplay">$0</span>
    </div>
    <div style="margin-top:14px">
      <div class="form-row"><label>聯絡人姓名 *</label><input type="text" id="cartName" placeholder="請輸入姓名"></div>
      <div class="form-row"><label>聯絡電話 *</label><input type="tel" id="cartPhone" placeholder="請輸入電話"></div>
      <div class="form-row"><label>客服處理人</label><input type="text" id="cartCSC" placeholder="請輸入處理人姓名"></div>
      <div class="form-row"><label>備註</label><textarea id="cartNote" placeholder="選填" style="min-height:50px"></textarea></div>
    </div>
    <div id="cartErr" style="color:#A32D2D;font-size:13px;min-height:18px;"></div>
    <div class="modal-footer">
      <button class="btn" onclick="closeCartModal()">取消</button>
      <button class="btn" onclick="printCartForm()">列印</button>
      <button class="btn btn-primary" onclick="submitCart()">✓ 送出</button>
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

<!-- 紀錄備註 -->
<div class="overlay" id="recNoteModal">
  <div class="modal" style="max-width:380px;">
    <h2>編輯備註</h2>
    <div class="form-row" style="margin-top:14px"><label>備註</label><textarea id="recNoteInput" style="min-height:80px"></textarea></div>
    <div class="modal-footer">
      <button class="btn" onclick="closeRecNoteModal()">取消</button>
      <button class="btn btn-primary" onclick="saveRecNote()">✓ 儲存</button>
    </div>
  </div>
</div>

<div class="toast" id="toast"></div>

<script>
const ADMIN_PW='5237';
const STORAGE_KEY='mpu_inventory';
const ORDERS_KEY='mpu_orders';
const PRICES_KEY='mpu_prices';
const PAGE_SIZE=15;
const DEFAULT_PRICE=5;
const DEFAULT_PARTS=[{"id": "10005644", "series": "", "locations": [{"wall": "MPU", "zone": "A區"}], "qty": "", "level": "high", "note": ""}, {"id": "102553", "series": "", "locations": [{"wall": "MPU", "zone": "A區"}], "qty": "", "level": "high", "note": ""}, {"id": "110525", "series": "", "locations": [{"wall": "MPU", "zone": "A區"}, {"wall": "ASIS", "zone": "8排10"}], "qty": "", "level": "high", "note": ""}, {"id": "112614", "series": "", "locations": [{"wall": "MPU", "zone": "A區"}], "qty": "", "level": "high", "note": ""}, {"id": "123482", "series": "", "locations": [{"wall": "MPU", "zone": "A區"}], "qty": "", "level": "high", "note": ""}, {"id": "102319", "series": "", "locations": [{"wall": "MPU", "zone": "A區"}, {"wall": "ASIS", "zone": "14排3"}], "qty": "", "level": "high", "note": ""}, {"id": "114258", "series": "", "locations": [{"wall": "MPU", "zone": "A區"}], "qty": "", "level": "high", "note": ""}, {"id": "114334", "series": "", "locations": [{"wall": "MPU", "zone": "A區"}, {"wall": "ASIS", "zone": "9排12"}], "qty": "", "level": "high", "note": ""}, {"id": "110617", "series": "", "locations": [{"wall": "MPU", "zone": "A區"}, {"wall": "ASIS", "zone": "2排11"}], "qty": "", "level": "high", "note": ""}, {"id": "119363", "series": "", "locations": [{"wall": "MPU", "zone": "A區"}, {"wall": "ASIS", "zone": "11排3"}], "qty": "", "level": "high", "note": ""}, {"id": "123786", "series": "", "locations": [{"wall": "MPU", "zone": "A區"}], "qty": "", "level": "high", "note": ""}, {"id": "114613", "series": "", "locations": [{"wall": "MPU", "zone": "A區"}], "qty": "", "level": "high", "note": ""}, {"id": "139299", "series": "", "locations": [{"wall": "MPU", "zone": "A區"}], "qty": "", "level": "high", "note": ""}, {"id": "190015", "series": "", "locations": [{"wall": "MPU", "zone": "A區"}, {"wall": "ASIS", "zone": "10排3"}], "qty": "", "level": "high", "note": ""}, {"id": "128643", "series": "", "locations": [{"wall": "MPU", "zone": "A區"}], "qty": "", "level": "high", "note": ""}, {"id": "114671", "series": "", "locations": [{"wall": "MPU", "zone": "A區"}, {"wall": "ASIS", "zone": "10排12"}], "qty": "", "level": "high", "note": ""}, {"id": "120292", "series": "", "locations": [{"wall": "MPU", "zone": "A區"}], "qty": "", "level": "high", "note": ""}, {"id": "106414", "series": "", "locations": [{"wall": "MPU", "zone": "A區"}, {"wall": "ASIS", "zone": "10排9"}], "qty": "", "level": "high", "note": ""}, {"id": "139240", "series": "", "locations": [{"wall": "MPU", "zone": "A區"}, {"wall": "ASIS", "zone": "11排8"}], "qty": "", "level": "high", "note": ""}, {"id": "123026", "series": "", "locations": [{"wall": "MPU", "zone": "A區"}], "qty": "", "level": "high", "note": ""}, {"id": "110364", "series": "", "locations": [{"wall": "MPU", "zone": "A區"}, {"wall": "ASIS背面", "zone": ""}], "qty": "", "level": "high", "note": ""}, {"id": "114204", "series": "", "locations": [{"wall": "MPU", "zone": "A區"}], "qty": "", "level": "high", "note": ""}, {"id": "100498", "series": "", "locations": [{"wall": "MPU", "zone": "A區"}], "qty": "", "level": "high", "note": ""}, {"id": "123257", "series": "", "locations": [{"wall": "MPU", "zone": "A區"}], "qty": "", "level": "high", "note": ""}, {"id": "110646", "series": "", "locations": [{"wall": "MPU", "zone": "A區"}, {"wall": "ASIS背面", "zone": ""}], "qty": "", "level": "high", "note": ""}, {"id": "114996", "series": "", "locations": [{"wall": "MPU", "zone": "A區"}, {"wall": "ASIS", "zone": "12排8"}], "qty": "", "level": "high", "note": ""}, {"id": "103437", "series": "", "locations": [{"wall": "MPU", "zone": "A區"}], "qty": "", "level": "high", "note": ""}, {"id": "123491", "series": "", "locations": [{"wall": "MPU", "zone": "A區"}, {"wall": "ASIS", "zone": "7排12"}], "qty": "", "level": "high", "note": ""}, {"id": "10083714", "series": "", "locations": [{"wall": "MPU", "zone": "A區"}], "qty": "", "level": "high", "note": ""}, {"id": "101324", "series": "", "locations": [{"wall": "MPU", "zone": "A區"}, {"wall": "ASIS背面", "zone": ""}], "qty": "", "level": "high", "note": ""}, {"id": "105346", "series": "", "locations": [{"wall": "MPU", "zone": "A區"}], "qty": "", "level": "high", "note": ""}, {"id": "123492", "series": "", "locations": [{"wall": "MPU", "zone": "A區"}, {"wall": "ASIS", "zone": "1排6"}], "qty": "", "level": "high", "note": ""}, {"id": "124439", "series": "", "locations": [{"wall": "MPU", "zone": "A區"}], "qty": "", "level": "high", "note": ""}, {"id": "101514", "series": "", "locations": [{"wall": "MPU", "zone": "A區"}, {"wall": "ASIS", "zone": "11排7"}], "qty": "", "level": "high", "note": ""}, {"id": "106940", "series": "", "locations": [{"wall": "MPU", "zone": "A區"}, {"wall": "ASIS", "zone": "10排11"}], "qty": "", "level": "high", "note": ""}, {"id": "130449", "series": "", "locations": [{"wall": "MPU", "zone": "A區"}, {"wall": "ASIS", "zone": "3排4"}], "qty": "", "level": "high", "note": ""}, {"id": "124461", "series": "", "locations": [{"wall": "MPU", "zone": "A區"}], "qty": "", "level": "high", "note": ""}, {"id": "10083715", "series": "", "locations": [{"wall": "MPU", "zone": "A區"}], "qty": "", "level": "high", "note": ""}, {"id": "101532", "series": "", "locations": [{"wall": "MPU", "zone": "A區"}, {"wall": "ASIS", "zone": "12排9"}], "qty": "", "level": "high", "note": ""}, {"id": "108001", "series": "", "locations": [{"wall": "MPU", "zone": "A區"}, {"wall": "ASIS", "zone": "2排6"}], "qty": "", "level": "high", "note": ""}, {"id": "130618", "series": "", "locations": [{"wall": "MPU", "zone": "A區"}, {"wall": "ASIS", "zone": "1排5"}], "qty": "", "level": "high", "note": ""}, {"id": "157306", "series": "", "locations": [{"wall": "MPU", "zone": "A區"}, {"wall": "ASIS", "zone": "10排10"}], "qty": "", "level": "high", "note": ""}, {"id": "101577", "series": "", "locations": [{"wall": "MPU", "zone": "A區"}, {"wall": "ASIS", "zone": "11排9"}], "qty": "", "level": "high", "note": ""}, {"id": "110115", "series": "", "locations": [{"wall": "MPU", "zone": "A區"}], "qty": "", "level": "high", "note": ""}, {"id": "114929", "series": "", "locations": [{"wall": "MPU", "zone": "A區"}, {"wall": "ASIS", "zone": "10排8"}], "qty": "", "level": "high", "note": ""}, {"id": "152533", "series": "", "locations": [{"wall": "MPU", "zone": "A區"}], "qty": "", "level": "high", "note": ""}, {"id": "104171", "series": "", "locations": [{"wall": "MPU", "zone": "A區"}], "qty": "", "level": "high", "note": ""}, {"id": "110618", "series": "", "locations": [{"wall": "MPU", "zone": "A區"}, {"wall": "ASIS", "zone": "2排11"}], "qty": "", "level": "high", "note": ""}, {"id": "120298", "series": "", "locations": [{"wall": "MPU", "zone": "A區"}, {"wall": "ASIS", "zone": "9排10"}], "qty": "", "level": "high", "note": ""}, {"id": "105494", "series": "", "locations": [{"wall": "MPU", "zone": "B區"}], "qty": "", "level": "high", "note": ""}, {"id": "115443", "series": "", "locations": [{"wall": "MPU", "zone": "B區"}, {"wall": "ASIS背面", "zone": ""}], "qty": "", "level": "high", "note": ""}, {"id": "116713", "series": "", "locations": [{"wall": "MPU", "zone": "B區"}, {"wall": "ASIS背面", "zone": ""}], "qty": "", "level": "high", "note": ""}, {"id": "113950", "series": "", "locations": [{"wall": "MPU", "zone": "B區"}], "qty": "", "level": "high", "note": ""}, {"id": "102335", "series": "", "locations": [{"wall": "MPU", "zone": "B區"}, {"wall": "ASIS背面", "zone": ""}], "qty": "", "level": "high", "note": ""}, {"id": "115444", "series": "", "locations": [{"wall": "MPU", "zone": "B區"}, {"wall": "ASIS背面", "zone": ""}], "qty": "", "level": "high", "note": ""}, {"id": "124217", "series": "", "locations": [{"wall": "MPU", "zone": "B區"}], "qty": "", "level": "high", "note": ""}, {"id": "118224", "series": "", "locations": [{"wall": "MPU", "zone": "B區"}], "qty": "", "level": "high", "note": ""}, {"id": "124476", "series": "", "locations": [{"wall": "MPU", "zone": "B區"}], "qty": "", "level": "high", "note": ""}, {"id": "117685", "series": "", "locations": [{"wall": "MPU", "zone": "B區"}], "qty": "", "level": "high", "note": ""}, {"id": "128998", "series": "", "locations": [{"wall": "MPU", "zone": "B區"}, {"wall": "ASIS", "zone": "11排10"}], "qty": "", "level": "high", "note": ""}, {"id": "106916", "series": "", "locations": [{"wall": "MPU", "zone": "B區"}], "qty": "", "level": "high", "note": ""}, {"id": "124477", "series": "", "locations": [{"wall": "MPU", "zone": "B區"}], "qty": "", "level": "high", "note": ""}, {"id": "147191", "series": "", "locations": [{"wall": "MPU", "zone": "B區"}, {"wall": "ASIS", "zone": "12排10"}], "qty": "", "level": "high", "note": ""}, {"id": "103693", "series": "", "locations": [{"wall": "MPU", "zone": "B區"}, {"wall": "ASIS背面", "zone": ""}], "qty": "", "level": "high", "note": ""}, {"id": "114931", "series": "", "locations": [{"wall": "MPU", "zone": "B區"}, {"wall": "ASIS背面", "zone": ""}], "qty": "", "level": "high", "note": ""}, {"id": "131120", "series": "", "locations": [{"wall": "MPU", "zone": "B區"}, {"wall": "ASIS", "zone": "7排11"}], "qty": "", "level": "high", "note": ""}, {"id": "141413", "series": "", "locations": [{"wall": "MPU", "zone": "B區"}, {"wall": "ASIS背面", "zone": ""}], "qty": "", "level": "high", "note": ""}, {"id": "118331", "series": "", "locations": [{"wall": "MPU", "zone": "B區"}, {"wall": "ASIS", "zone": "1排4"}], "qty": "", "level": "high", "note": ""}, {"id": "102267", "series": "", "locations": [{"wall": "MPU", "zone": "B區"}, {"wall": "ASIS", "zone": "2排12"}], "qty": "", "level": "high", "note": ""}, {"id": "131121", "series": "", "locations": [{"wall": "MPU", "zone": "B區"}, {"wall": "ASIS", "zone": "7排11"}], "qty": "", "level": "high", "note": ""}, {"id": "107271", "series": "", "locations": [{"wall": "MPU", "zone": "B區"}, {"wall": "ASIS", "zone": "16排11"}], "qty": "", "level": "high", "note": ""}, {"id": "110670", "series": "", "locations": [{"wall": "MPU", "zone": "B區"}, {"wall": "ASIS", "zone": "4排11"}], "qty": "", "level": "high", "note": ""}, {"id": "124526", "series": "", "locations": [{"wall": "MPU", "zone": "B區"}], "qty": "", "level": "high", "note": ""}, {"id": "133194", "series": "", "locations": [{"wall": "MPU", "zone": "B區"}, {"wall": "ASIS背面", "zone": ""}], "qty": "", "level": "high", "note": ""}, {"id": "122998", "series": "", "locations": [{"wall": "MPU", "zone": "B區"}, {"wall": "ASIS", "zone": "9排11"}], "qty": "", "level": "high", "note": ""}, {"id": "106989", "series": "", "locations": [{"wall": "MPU", "zone": "B區"}, {"wall": "ASIS背面", "zone": ""}], "qty": "", "level": "high", "note": ""}, {"id": "114667", "series": "", "locations": [{"wall": "MPU", "zone": "B區"}], "qty": "", "level": "high", "note": ""}, {"id": "133191", "series": "", "locations": [{"wall": "MPU", "zone": "B區"}], "qty": "", "level": "high", "note": ""}, {"id": "131372", "series": "", "locations": [{"wall": "MPU", "zone": "B區"}, {"wall": "ASIS", "zone": "9排9"}], "qty": "", "level": "high", "note": ""}, {"id": "10050797", "series": "", "locations": [{"wall": "MPU", "zone": "B區"}, {"wall": "ASIS背面", "zone": ""}], "qty": "", "level": "high", "note": ""}, {"id": "192284", "series": "", "locations": [{"wall": "MPU", "zone": "B區"}], "qty": "", "level": "high", "note": ""}, {"id": "122305", "series": "", "locations": [{"wall": "MPU", "zone": "B區"}], "qty": "", "level": "high", "note": ""}, {"id": "120933", "series": "", "locations": [{"wall": "MPU", "zone": "B區"}], "qty": "", "level": "high", "note": ""}, {"id": "144717", "series": "", "locations": [{"wall": "MPU", "zone": "B區"}], "qty": "", "level": "high", "note": ""}, {"id": "114594", "series": "", "locations": [{"wall": "MPU", "zone": "B區"}, {"wall": "ASIS", "zone": "14排9"}], "qty": "", "level": "high", "note": ""}, {"id": "100001", "series": "", "locations": [{"wall": "MPU", "zone": "B區"}], "qty": "", "level": "high", "note": ""}, {"id": "100712", "series": "", "locations": [{"wall": "MPU", "zone": "C區"}, {"wall": "ASIS", "zone": "15排2"}], "qty": "", "level": "high", "note": ""}, {"id": "108817", "series": "", "locations": [{"wall": "MPU", "zone": "C區"}], "qty": "", "level": "high", "note": ""}, {"id": "103430", "series": "", "locations": [{"wall": "MPU", "zone": "C區"}], "qty": "", "level": "high", "note": ""}, {"id": "100716", "series": "", "locations": [{"wall": "MPU", "zone": "C區"}, {"wall": "ASIS", "zone": "15排3"}], "qty": "", "level": "high", "note": ""}, {"id": "110573", "series": "", "locations": [{"wall": "MPU", "zone": "C區"}], "qty": "", "level": "high", "note": ""}, {"id": "119030", "series": "", "locations": [{"wall": "MPU", "zone": "C區"}, {"wall": "ASIS", "zone": "11排5"}], "qty": "", "level": "high", "note": ""}, {"id": "100843", "series": "", "locations": [{"wall": "MPU", "zone": "C區"}, {"wall": "ASIS", "zone": "16排4"}], "qty": "", "level": "high", "note": ""}, {"id": "112891", "series": "", "locations": [{"wall": "MPU", "zone": "C區"}], "qty": "", "level": "high", "note": ""}, {"id": "119081", "series": "", "locations": [{"wall": "MPU", "zone": "C區"}, {"wall": "ASIS", "zone": "10排5"}], "qty": "", "level": "high", "note": ""}, {"id": "104875", "series": "", "locations": [{"wall": "MPU", "zone": "C區"}, {"wall": "ASIS", "zone": "14排1"}], "qty": "", "level": "high", "note": ""}, {"id": "114254", "series": "", "locations": [{"wall": "MPU", "zone": "C區"}, {"wall": "ASIS", "zone": "15排1"}], "qty": "", "level": "high", "note": ""}, {"id": "122332", "series": "", "locations": [{"wall": "MPU", "zone": "C區"}], "qty": "", "level": "high", "note": ""}, {"id": "117434", "series": "", "locations": [{"wall": "MPU", "zone": "C區"}], "qty": "", "level": "high", "note": ""}, {"id": "110630", "series": "", "locations": [{"wall": "MPU", "zone": "C區"}, {"wall": "ASIS", "zone": "12排4"}], "qty": "", "level": "high", "note": ""}, {"id": "130619", "series": "", "locations": [{"wall": "MPU", "zone": "C區"}], "qty": "", "level": "high", "note": ""}, {"id": "110126", "series": "", "locations": [{"wall": "MPU", "zone": "C區"}, {"wall": "ASIS", "zone": "13排5"}], "qty": "", "level": "high", "note": ""}, {"id": "102509", "series": "", "locations": [{"wall": "MPU", "zone": "C區"}, {"wall": "ASIS", "zone": "12排5"}], "qty": "", "level": "high", "note": ""}, {"id": "118137", "series": "", "locations": [{"wall": "MPU", "zone": "C區"}], "qty": "", "level": "high", "note": ""}, {"id": "120076", "series": "", "locations": [{"wall": "MPU", "zone": "C區"}, {"wall": "ASIS", "zone": "11排4"}], "qty": "", "level": "high", "note": ""}, {"id": "114927", "series": "", "locations": [{"wall": "MPU", "zone": "C區"}], "qty": "", "level": "high", "note": ""}, {"id": "103114", "series": "", "locations": [{"wall": "MPU", "zone": "C區"}, {"wall": "ASIS", "zone": "11排6"}], "qty": "", "level": "high", "note": ""}, {"id": "119253", "series": "", "locations": [{"wall": "MPU", "zone": "C區"}], "qty": "", "level": "high", "note": ""}, {"id": "122971", "series": "", "locations": [{"wall": "MPU", "zone": "C區"}, {"wall": "ASIS", "zone": "10排4"}], "qty": "", "level": "high", "note": ""}, {"id": "106720", "series": "", "locations": [{"wall": "MPU", "zone": "C區"}], "qty": "", "level": "high", "note": ""}, {"id": "101339", "series": "", "locations": [{"wall": "MPU", "zone": "C區"}, {"wall": "ASIS", "zone": "1排7"}], "qty": "", "level": "high", "note": ""}, {"id": "101350", "series": "", "locations": [{"wall": "MPU", "zone": "C區"}, {"wall": "ASIS", "zone": "2排9"}], "qty": "", "level": "high", "note": ""}, {"id": "101356", "series": "", "locations": [{"wall": "MPU", "zone": "C區"}], "qty": "", "level": "high", "note": ""}, {"id": "101372", "series": "", "locations": [{"wall": "MPU", "zone": "C區"}, {"wall": "ASIS", "zone": "4排7"}], "qty": "", "level": "high", "note": ""}, {"id": "101341", "series": "", "locations": [{"wall": "MPU", "zone": "C區"}], "qty": "", "level": "high", "note": ""}, {"id": "101351", "series": "", "locations": [{"wall": "MPU", "zone": "C區"}, {"wall": "ASIS", "zone": "3排7"}], "qty": "", "level": "high", "note": ""}, {"id": "101357", "series": "", "locations": [{"wall": "MPU", "zone": "C區"}], "qty": "", "level": "high", "note": ""}, {"id": "101375", "series": "", "locations": [{"wall": "MPU", "zone": "C區"}, {"wall": "ASIS", "zone": "3排9"}], "qty": "", "level": "high", "note": ""}, {"id": "101343", "series": "", "locations": [{"wall": "MPU", "zone": "C區"}], "qty": "", "level": "high", "note": ""}, {"id": "101352", "series": "", "locations": [{"wall": "MPU", "zone": "C區"}, {"wall": "ASIS", "zone": "2排7"}], "qty": "", "level": "high", "note": ""}, {"id": "101359", "series": "", "locations": [{"wall": "MPU", "zone": "C區"}, {"wall": "ASIS", "zone": "2排8"}], "qty": "", "level": "high", "note": ""}, {"id": "118740", "series": "", "locations": [{"wall": "MPU", "zone": "C區"}, {"wall": "ASIS", "zone": "1排9"}], "qty": "", "level": "high", "note": ""}, {"id": "101345", "series": "", "locations": [{"wall": "MPU", "zone": "C區"}, {"wall": "ASIS", "zone": "1排8"}], "qty": "", "level": "high", "note": ""}, {"id": "101354", "series": "", "locations": [{"wall": "MPU", "zone": "C區"}], "qty": "", "level": "high", "note": ""}, {"id": "101360", "series": "", "locations": [{"wall": "MPU", "zone": "C區"}], "qty": "", "level": "high", "note": ""}, {"id": "120173", "series": "", "locations": [{"wall": "MPU", "zone": "C區"}], "qty": "", "level": "high", "note": ""}, {"id": "106422", "series": "", "locations": [{"wall": "MPU", "zone": "D區"}], "qty": "", "level": "high", "note": ""}, {"id": "122568", "series": "", "locations": [{"wall": "MPU", "zone": "D區"}], "qty": "", "level": "high", "note": ""}, {"id": "121003", "series": "", "locations": [{"wall": "MPU", "zone": "D區"}], "qty": "", "level": "high", "note": ""}, {"id": "100440", "series": "", "locations": [{"wall": "MPU", "zone": "D區"}], "qty": "", "level": "high", "note": ""}, {"id": "106698", "series": "", "locations": [{"wall": "MPU", "zone": "D區"}], "qty": "", "level": "high", "note": ""}, {"id": "148903", "series": "", "locations": [{"wall": "MPU", "zone": "D區"}], "qty": "", "level": "high", "note": ""}, {"id": "10084881", "series": "", "locations": [{"wall": "MPU", "zone": "D區"}], "qty": "", "level": "high", "note": ""}, {"id": "100600", "series": "", "locations": [{"wall": "MPU", "zone": "D區"}], "qty": "", "level": "high", "note": ""}, {"id": "115344", "series": "", "locations": [{"wall": "MPU", "zone": "D區"}, {"wall": "ASIS", "zone": "9排8"}], "qty": "", "level": "high", "note": ""}, {"id": "151705", "series": "", "locations": [{"wall": "MPU", "zone": "D區"}, {"wall": "ASIS", "zone": "15排5"}], "qty": "", "level": "high", "note": ""}, {"id": "100602", "series": "", "locations": [{"wall": "MPU", "zone": "D區"}, {"wall": "ASIS", "zone": "7排3"}], "qty": "", "level": "high", "note": ""}, {"id": "139199", "series": "", "locations": [{"wall": "MPU", "zone": "D區"}], "qty": "", "level": "high", "note": ""}, {"id": "151706", "series": "", "locations": [{"wall": "MPU", "zone": "D區"}], "qty": "", "level": "high", "note": ""}, {"id": "100646", "series": "", "locations": [{"wall": "MPU", "zone": "D區"}, {"wall": "ASIS", "zone": "3排1"}], "qty": "", "level": "high", "note": ""}, {"id": "153311", "series": "", "locations": [{"wall": "MPU", "zone": "D區"}, {"wall": "ASIS", "zone": "14排11"}], "qty": "", "level": "high", "note": ""}, {"id": "102372", "series": "", "locations": [{"wall": "MPU", "zone": "D區"}, {"wall": "ASIS", "zone": "16排10"}], "qty": "", "level": "high", "note": ""}, {"id": "121121", "series": "", "locations": [{"wall": "MPU", "zone": "D區"}], "qty": "", "level": "high", "note": ""}, {"id": "100673", "series": "", "locations": [{"wall": "MPU", "zone": "D區"}], "qty": "", "level": "high", "note": ""}, {"id": "153312", "series": "", "locations": [{"wall": "MPU", "zone": "D區"}, {"wall": "ASIS", "zone": "14排11"}], "qty": "", "level": "high", "note": ""}, {"id": "107849", "series": "", "locations": [{"wall": "MPU", "zone": "D區"}], "qty": "", "level": "high", "note": ""}, {"id": "100751", "series": "", "locations": [{"wall": "MPU", "zone": "D區"}, {"wall": "ASIS", "zone": "3排11"}], "qty": "", "level": "high", "note": ""}, {"id": "152741", "series": "", "locations": [{"wall": "MPU", "zone": "D區"}, {"wall": "ASIS", "zone": "13排11"}], "qty": "", "level": "high", "note": ""}, {"id": "128546", "series": "", "locations": [{"wall": "MPU", "zone": "D區"}], "qty": "", "level": "high", "note": ""}, {"id": "110301", "series": "", "locations": [{"wall": "MPU", "zone": "D區"}], "qty": "", "level": "high", "note": ""}, {"id": "153307", "series": "", "locations": [{"wall": "MPU", "zone": "D區"}, {"wall": "ASIS", "zone": "13排11"}], "qty": "", "level": "high", "note": ""}, {"id": "100719", "series": "", "locations": [{"wall": "MPU", "zone": "D區"}], "qty": "", "level": "high", "note": ""}, {"id": "148624", "series": "", "locations": [{"wall": "MPU", "zone": "D區"}, {"wall": "ASIS背面", "zone": ""}], "qty": "", "level": "high", "note": ""}, {"id": "130884", "series": "", "locations": [{"wall": "MPU", "zone": "D區"}, {"wall": "ASIS", "zone": "11排1"}], "qty": "", "level": "high", "note": ""}, {"id": "102728", "series": "", "locations": [{"wall": "MPU", "zone": "D區"}], "qty": "", "level": "high", "note": ""}, {"id": "130885", "series": "", "locations": [{"wall": "MPU", "zone": "D區"}, {"wall": "ASIS", "zone": "10排1"}], "qty": "", "level": "high", "note": ""}, {"id": "122158", "series": "", "locations": [{"wall": "MPU", "zone": "D區"}], "qty": "", "level": "high", "note": ""}, {"id": "130886", "series": "", "locations": [{"wall": "MPU", "zone": "D區"}, {"wall": "ASIS", "zone": "11排2"}], "qty": "", "level": "high", "note": ""}, {"id": "121762", "series": "", "locations": [{"wall": "MPU", "zone": "D區"}, {"wall": "ASIS", "zone": "12排7"}], "qty": "", "level": "high", "note": ""}, {"id": "130887", "series": "", "locations": [{"wall": "MPU", "zone": "D區"}, {"wall": "ASIS", "zone": "10排2"}], "qty": "", "level": "high", "note": ""}, {"id": "123660", "series": "", "locations": [{"wall": "MPU", "zone": "D區"}], "qty": "", "level": "high", "note": ""}, {"id": "100325", "series": "", "locations": [{"wall": "MPU", "zone": "E區"}], "qty": "", "level": "high", "note": ""}, {"id": "105248", "series": "", "locations": [{"wall": "MPU", "zone": "E區"}, {"wall": "ASIS", "zone": "5排4"}], "qty": "", "level": "high", "note": ""}, {"id": "110365", "series": "", "locations": [{"wall": "MPU", "zone": "E區"}], "qty": "", "level": "high", "note": ""}, {"id": "151743", "series": "", "locations": [{"wall": "MPU", "zone": "E區"}], "qty": "", "level": "high", "note": ""}, {"id": "100347", "series": "", "locations": [{"wall": "MPU", "zone": "E區"}, {"wall": "ASIS", "zone": "8排4"}], "qty": "", "level": "high", "note": ""}, {"id": "105307", "series": "", "locations": [{"wall": "MPU", "zone": "E區"}, {"wall": "ASIS", "zone": "7排4"}], "qty": "", "level": "high", "note": ""}, {"id": "112548", "series": "", "locations": [{"wall": "MPU", "zone": "E區"}], "qty": "", "level": "high", "note": ""}, {"id": "193783", "series": "", "locations": [{"wall": "MPU", "zone": "E區"}], "qty": "", "level": "high", "note": ""}, {"id": "100349", "series": "", "locations": [{"wall": "MPU", "zone": "E區"}, {"wall": "ASIS", "zone": "8排5"}], "qty": "", "level": "high", "note": ""}, {"id": "105344", "series": "", "locations": [{"wall": "MPU", "zone": "E區"}, {"wall": "ASIS", "zone": "6排8"}], "qty": "", "level": "high", "note": ""}, {"id": "113286", "series": "", "locations": [{"wall": "MPU", "zone": "E區"}], "qty": "", "level": "high", "note": ""}, {"id": "100167", "series": "", "locations": [{"wall": "MPU", "zone": "E區"}], "qty": "", "level": "high", "note": ""}, {"id": "100359", "series": "", "locations": [{"wall": "MPU", "zone": "E區"}], "qty": "", "level": "high", "note": ""}, {"id": "105892", "series": "", "locations": [{"wall": "MPU", "zone": "E區"}], "qty": "", "level": "high", "note": ""}, {"id": "116342", "series": "", "locations": [{"wall": "MPU", "zone": "E區"}], "qty": "", "level": "high", "note": ""}, {"id": "100365", "series": "", "locations": [{"wall": "MPU", "zone": "E區"}, {"wall": "ASIS", "zone": "9排4"}], "qty": "", "level": "high", "note": ""}, {"id": "106632", "series": "", "locations": [{"wall": "MPU", "zone": "E區"}], "qty": "", "level": "high", "note": ""}, {"id": "116894", "series": "", "locations": [{"wall": "MPU", "zone": "E區"}], "qty": "", "level": "high", "note": ""}, {"id": "100400", "series": "", "locations": [{"wall": "MPU", "zone": "E區"}, {"wall": "ASIS", "zone": "1排2"}], "qty": "", "level": "high", "note": ""}, {"id": "109049", "series": "", "locations": [{"wall": "MPU", "zone": "E區"}], "qty": "", "level": "high", "note": ""}, {"id": "120923", "series": "", "locations": [{"wall": "MPU", "zone": "E區"}], "qty": "", "level": "high", "note": ""}, {"id": "100402", "series": "", "locations": [{"wall": "MPU", "zone": "E區"}, {"wall": "ASIS", "zone": "2排1"}], "qty": "", "level": "high", "note": ""}, {"id": "109534", "series": "", "locations": [{"wall": "MPU", "zone": "E區"}], "qty": "", "level": "high", "note": ""}, {"id": "122006", "series": "", "locations": [{"wall": "MPU", "zone": "E區"}], "qty": "", "level": "high", "note": ""}, {"id": "100408", "series": "", "locations": [{"wall": "MPU", "zone": "E區"}, {"wall": "ASIS", "zone": "1排3"}], "qty": "", "level": "high", "note": ""}, {"id": "109557", "series": "", "locations": [{"wall": "MPU", "zone": "E區"}], "qty": "", "level": "high", "note": ""}, {"id": "123757", "series": "", "locations": [{"wall": "MPU", "zone": "E區"}], "qty": "", "level": "high", "note": ""}, {"id": "100412", "series": "", "locations": [{"wall": "MPU", "zone": "E區"}], "qty": "", "level": "high", "note": ""}, {"id": "109560", "series": "", "locations": [{"wall": "MPU", "zone": "E區"}], "qty": "", "level": "high", "note": ""}, {"id": "126887", "series": "", "locations": [{"wall": "MPU", "zone": "E區"}], "qty": "", "level": "high", "note": ""}, {"id": "100413", "series": "", "locations": [{"wall": "MPU", "zone": "E區"}], "qty": "", "level": "high", "note": ""}, {"id": "109566", "series": "", "locations": [{"wall": "MPU", "zone": "E區"}], "qty": "", "level": "high", "note": ""}, {"id": "128814", "series": "", "locations": [{"wall": "MPU", "zone": "E區"}], "qty": "", "level": "high", "note": ""}, {"id": "101065", "series": "", "locations": [{"wall": "MPU", "zone": "E區"}, {"wall": "ASIS", "zone": "8排7"}], "qty": "", "level": "high", "note": ""}, {"id": "109598", "series": "", "locations": [{"wall": "MPU", "zone": "E區"}], "qty": "", "level": "high", "note": ""}, {"id": "130421", "series": "", "locations": [{"wall": "MPU", "zone": "E區"}], "qty": "", "level": "high", "note": ""}, {"id": "102138", "series": "", "locations": [{"wall": "MPU", "zone": "E區"}, {"wall": "ASIS", "zone": "7排6"}], "qty": "", "level": "high", "note": ""}, {"id": "110327", "series": "", "locations": [{"wall": "MPU", "zone": "E區"}], "qty": "", "level": "high", "note": ""}, {"id": "194732", "series": "", "locations": [{"wall": "MPU", "zone": "E區"}], "qty": "", "level": "high", "note": ""}, {"id": "105811", "series": "", "locations": [{"wall": "MPU", "zone": "E區"}], "qty": "", "level": "high", "note": ""}, {"id": "10003444", "series": "", "locations": [{"wall": "MPU", "zone": "F區"}], "qty": "", "level": "high", "note": ""}, {"id": "105111", "series": "", "locations": [{"wall": "MPU", "zone": "F區"}, {"wall": "ASIS", "zone": "6排2"}], "qty": "", "level": "high", "note": ""}, {"id": "105907", "series": "", "locations": [{"wall": "MPU", "zone": "F區"}], "qty": "", "level": "high", "note": ""}, {"id": "147168", "series": "", "locations": [{"wall": "MPU", "zone": "F區"}], "qty": "", "level": "high", "note": ""}, {"id": "100108", "series": "", "locations": [{"wall": "MPU", "zone": "F區"}], "qty": "", "level": "high", "note": ""}, {"id": "105236", "series": "", "locations": [{"wall": "MPU", "zone": "F區"}], "qty": "", "level": "high", "note": ""}, {"id": "105330", "series": "", "locations": [{"wall": "MPU", "zone": "F區"}, {"wall": "ASIS", "zone": "8排3"}], "qty": "", "level": "high", "note": ""}, {"id": "139210", "series": "", "locations": [{"wall": "MPU", "zone": "F區"}], "qty": "", "level": "high", "note": ""}, {"id": "100165", "series": "", "locations": [{"wall": "MPU", "zone": "F區"}], "qty": "", "level": "high", "note": ""}, {"id": "105237", "series": "", "locations": [{"wall": "MPU", "zone": "F區"}], "qty": "", "level": "high", "note": ""}, {"id": "100214", "series": "", "locations": [{"wall": "MPU", "zone": "F區"}, {"wall": "ASIS", "zone": "4排5"}], "qty": "", "level": "high", "note": ""}, {"id": "122899", "series": "", "locations": [{"wall": "MPU", "zone": "F區"}], "qty": "", "level": "high", "note": ""}, {"id": "100178", "series": "", "locations": [{"wall": "MPU", "zone": "F區"}], "qty": "", "level": "high", "note": ""}, {"id": "106877", "series": "", "locations": [{"wall": "MPU", "zone": "F區"}], "qty": "", "level": "high", "note": ""}, {"id": "100218", "series": "", "locations": [{"wall": "MPU", "zone": "F區"}], "qty": "", "level": "high", "note": ""}, {"id": "111000", "series": "", "locations": [{"wall": "MPU", "zone": "F區"}], "qty": "", "level": "high", "note": ""}, {"id": "100194", "series": "", "locations": [{"wall": "MPU", "zone": "F區"}], "qty": "", "level": "high", "note": ""}, {"id": "112522", "series": "", "locations": [{"wall": "MPU", "zone": "F區"}, {"wall": "ASIS", "zone": "6排3"}], "qty": "", "level": "high", "note": ""}, {"id": "100224", "series": "", "locations": [{"wall": "MPU", "zone": "F區"}], "qty": "", "level": "high", "note": ""}, {"id": "124435", "series": "", "locations": [{"wall": "MPU", "zone": "F區"}], "qty": "", "level": "high", "note": ""}, {"id": "100227", "series": "", "locations": [{"wall": "MPU", "zone": "F區"}], "qty": "", "level": "high", "note": ""}, {"id": "116599", "series": "", "locations": [{"wall": "MPU", "zone": "F區"}], "qty": "", "level": "high", "note": ""}, {"id": "100298", "series": "", "locations": [{"wall": "MPU", "zone": "F區"}], "qty": "", "level": "high", "note": ""}, {"id": "100179", "series": "", "locations": [{"wall": "MPU", "zone": "F區"}], "qty": "", "level": "high", "note": ""}, {"id": "100260", "series": "", "locations": [{"wall": "MPU", "zone": "F區"}], "qty": "", "level": "high", "note": ""}, {"id": "117794", "series": "", "locations": [{"wall": "MPU", "zone": "F區"}], "qty": "", "level": "high", "note": ""}, {"id": "109060", "series": "", "locations": [{"wall": "MPU", "zone": "F區"}, {"wall": "ASIS", "zone": "5排5"}], "qty": "", "level": "high", "note": ""}, {"id": "104322", "series": "", "locations": [{"wall": "MPU", "zone": "F區"}], "qty": "", "level": "high", "note": ""}, {"id": "100263", "series": "", "locations": [{"wall": "MPU", "zone": "F區"}], "qty": "", "level": "high", "note": ""}, {"id": "123756", "series": "", "locations": [{"wall": "MPU", "zone": "F區"}], "qty": "", "level": "high", "note": ""}, {"id": "110907", "series": "", "locations": [{"wall": "MPU", "zone": "F區"}, {"wall": "ASIS", "zone": "5排6"}], "qty": "", "level": "high", "note": ""}, {"id": "122900", "series": "", "locations": [{"wall": "MPU", "zone": "F區"}], "qty": "", "level": "high", "note": ""}, {"id": "100295", "series": "", "locations": [{"wall": "MPU", "zone": "F區"}], "qty": "", "level": "high", "note": ""}, {"id": "124584", "series": "", "locations": [{"wall": "MPU", "zone": "F區"}], "qty": "", "level": "high", "note": ""}, {"id": "115980", "series": "", "locations": [{"wall": "MPU", "zone": "F區"}], "qty": "", "level": "high", "note": ""}, {"id": "101385", "series": "", "locations": [{"wall": "MPU", "zone": "F區"}], "qty": "", "level": "high", "note": ""}, {"id": "100391", "series": "", "locations": [{"wall": "MPU", "zone": "F區"}], "qty": "", "level": "high", "note": ""}, {"id": "146654", "series": "", "locations": [{"wall": "MPU", "zone": "F區"}], "qty": "", "level": "high", "note": ""}, {"id": "121817", "series": "", "locations": [{"wall": "MPU", "zone": "F區"}], "qty": "", "level": "high", "note": ""}, {"id": "139214", "series": "", "locations": [{"wall": "MPU", "zone": "F區"}], "qty": "", "level": "high", "note": ""}, {"id": "105102", "series": "", "locations": [{"wall": "MPU", "zone": "F區"}], "qty": "", "level": "high", "note": ""}, {"id": "154571", "series": "", "locations": [{"wall": "MPU", "zone": "F區"}], "qty": "", "level": "high", "note": ""}, {"id": "128745", "series": "", "locations": [{"wall": "MPU", "zone": "F區"}], "qty": "", "level": "high", "note": ""}, {"id": "100136", "series": "", "locations": [{"wall": "MPU", "zone": "F區"}], "qty": "", "level": "high", "note": ""}, {"id": "105107", "series": "", "locations": [{"wall": "MPU", "zone": "F區"}], "qty": "", "level": "high", "note": ""}, {"id": "110019", "series": "", "locations": [{"wall": "MPU", "zone": "F區"}], "qty": "", "level": "high", "note": ""}, {"id": "111034", "series": "", "locations": [{"wall": "MPU", "zone": "F區"}], "qty": "", "level": "high", "note": ""}, {"id": "122604", "series": "", "locations": [{"wall": "MPU", "zone": "F區"}], "qty": "", "level": "high", "note": ""}, {"id": "114235", "series": "", "locations": [{"wall": "MPU", "zone": "G區"}], "qty": "", "level": "high", "note": ""}, {"id": "109842", "series": "", "locations": [{"wall": "MPU", "zone": "G區"}], "qty": "", "level": "high", "note": ""}, {"id": "115447", "series": "", "locations": [{"wall": "MPU", "zone": "G區"}], "qty": "", "level": "high", "note": ""}, {"id": "190447", "series": "", "locations": [{"wall": "MPU", "zone": "G區"}], "qty": "", "level": "high", "note": ""}, {"id": "140956", "series": "", "locations": [{"wall": "MPU", "zone": "G區"}], "qty": "", "level": "high", "note": ""}, {"id": "151059", "series": "", "locations": [{"wall": "MPU", "zone": "G區"}], "qty": "", "level": "high", "note": ""}, {"id": "108444", "series": "", "locations": [{"wall": "MPU", "zone": "G區"}], "qty": "", "level": "high", "note": ""}, {"id": "151060", "series": "", "locations": [{"wall": "MPU", "zone": "G區"}], "qty": "", "level": "high", "note": ""}, {"id": "100266", "series": "", "locations": [{"wall": "MPU", "zone": "G區"}], "qty": "", "level": "high", "note": ""}, {"id": "151061", "series": "", "locations": [{"wall": "MPU", "zone": "G區"}], "qty": "", "level": "high", "note": ""}, {"id": "157682", "series": "", "locations": [{"wall": "MPU", "zone": "G區"}], "qty": "", "level": "high", "note": ""}, {"id": "130856", "series": "", "locations": [{"wall": "MPU", "zone": "G區"}], "qty": "", "level": "high", "note": ""}, {"id": "10087158", "series": "", "locations": [{"wall": "MPU", "zone": "G區"}, {"wall": "ASIS背面", "zone": ""}], "qty": "", "level": "high", "note": ""}, {"id": "130914", "series": "", "locations": [{"wall": "MPU", "zone": "G區"}], "qty": "", "level": "high", "note": ""}, {"id": "190226", "series": "", "locations": [{"wall": "MPU", "zone": "G區"}], "qty": "", "level": "high", "note": ""}, {"id": "121214", "series": "", "locations": [{"wall": "MPU", "zone": "G區"}], "qty": "", "level": "high", "note": ""}, {"id": "10067913", "series": "", "locations": [{"wall": "MPU", "zone": "G區"}], "qty": "", "level": "high", "note": ""}, {"id": "130417", "series": "", "locations": [{"wall": "MPU", "zone": "G區"}], "qty": "", "level": "high", "note": ""}, {"id": "141288", "series": "", "locations": [{"wall": "MPU", "zone": "G區"}], "qty": "", "level": "high", "note": ""}, {"id": "107967", "series": "", "locations": [{"wall": "MPU", "zone": "G區"}], "qty": "", "level": "high", "note": ""}, {"id": "100823", "series": "", "locations": [{"wall": "MPU", "zone": "G區"}, {"wall": "ASIS", "zone": "13排3"}], "qty": "", "level": "high", "note": ""}, {"id": "131337", "series": "", "locations": [{"wall": "MPU", "zone": "G區"}], "qty": "", "level": "high", "note": ""}, {"id": "158208", "series": "", "locations": [{"wall": "MPU", "zone": "G區"}], "qty": "", "level": "high", "note": ""}, {"id": "10049259", "series": "", "locations": [{"wall": "MPU", "zone": "G區"}], "qty": "", "level": "high", "note": ""}, {"id": "10056050", "series": "", "locations": [{"wall": "MPU", "zone": "G區"}], "qty": "", "level": "high", "note": ""}, {"id": "124607", "series": "", "locations": [{"wall": "MPU", "zone": "G區"}], "qty": "", "level": "high", "note": ""}, {"id": "130490", "series": "", "locations": [{"wall": "MPU", "zone": "G區"}], "qty": "", "level": "high", "note": ""}, {"id": "158747", "series": "", "locations": [{"wall": "MPU", "zone": "G區"}], "qty": "", "level": "high", "note": ""}, {"id": "10044713", "series": "", "locations": [{"wall": "MPU", "zone": "G區"}], "qty": "", "level": "high", "note": ""}, {"id": "122145", "series": "", "locations": [{"wall": "MPU", "zone": "G區"}], "qty": "", "level": "high", "note": ""}, {"id": "102384", "series": "", "locations": [{"wall": "MPU", "zone": "G區"}, {"wall": "ASIS背面", "zone": ""}], "qty": "", "level": "high", "note": ""}, {"id": "195564", "series": "", "locations": [{"wall": "MPU", "zone": "G區"}], "qty": "", "level": "high", "note": ""}, {"id": "102720", "series": "", "locations": [{"wall": "MPU", "zone": "G區"}], "qty": "", "level": "high", "note": ""}, {"id": "114361", "series": "", "locations": [{"wall": "MPU", "zone": "G區"}], "qty": "", "level": "high", "note": ""}, {"id": "104663", "series": "", "locations": [{"wall": "MPU", "zone": "G區"}, {"wall": "ASIS背面", "zone": ""}], "qty": "", "level": "high", "note": ""}, {"id": "111402", "series": "", "locations": [{"wall": "MPU", "zone": "G區"}], "qty": "", "level": "high", "note": ""}, {"id": "117559", "series": "", "locations": [{"wall": "MPU", "zone": "G區"}], "qty": "", "level": "high", "note": ""}, {"id": "124216", "series": "", "locations": [{"wall": "MPU", "zone": "G區"}], "qty": "", "level": "high", "note": ""}, {"id": "109142", "series": "", "locations": [{"wall": "MPU", "zone": "G區"}], "qty": "", "level": "high", "note": ""}, {"id": "111401", "series": "", "locations": [{"wall": "MPU", "zone": "G區"}], "qty": "", "level": "high", "note": ""}, {"id": "124479", "series": "", "locations": [{"wall": "MPU", "zone": "G區"}], "qty": "", "level": "high", "note": ""}, {"id": "122556", "series": "", "locations": [{"wall": "MPU", "zone": "G區"}], "qty": "", "level": "high", "note": ""}, {"id": "117138", "series": "", "locations": [{"wall": "MPU", "zone": "H區"}], "qty": "", "level": "high", "note": ""}, {"id": "113333", "series": "", "locations": [{"wall": "MPU", "zone": "H區"}], "qty": "", "level": "high", "note": ""}, {"id": "159426", "series": "", "locations": [{"wall": "MPU", "zone": "H區"}, {"wall": "ASIS", "zone": "15排7"}], "qty": "", "level": "high", "note": ""}, {"id": "108442", "series": "", "locations": [{"wall": "MPU", "zone": "H區"}], "qty": "", "level": "high", "note": ""}, {"id": "150450", "series": "", "locations": [{"wall": "MPU", "zone": "H區"}], "qty": "", "level": "high", "note": ""}, {"id": "116645", "series": "", "locations": [{"wall": "MPU", "zone": "H區"}], "qty": "", "level": "high", "note": ""}, {"id": "100648", "series": "", "locations": [{"wall": "MPU", "zone": "H區"}], "qty": "", "level": "high", "note": ""}, {"id": "106299", "series": "", "locations": [{"wall": "MPU", "zone": "H區"}], "qty": "", "level": "high", "note": ""}, {"id": "106917", "series": "", "locations": [{"wall": "MPU", "zone": "H區"}], "qty": "", "level": "high", "note": ""}, {"id": "122925", "series": "", "locations": [{"wall": "MPU", "zone": "H區"}], "qty": "", "level": "high", "note": ""}, {"id": "106498", "series": "", "locations": [{"wall": "MPU", "zone": "H區"}, {"wall": "ASIS", "zone": "4排2"}], "qty": "", "level": "high", "note": ""}, {"id": "111451", "series": "", "locations": [{"wall": "MPU", "zone": "H區"}], "qty": "", "level": "high", "note": ""}, {"id": "100343", "series": "", "locations": [{"wall": "MPU", "zone": "H區"}], "qty": "", "level": "high", "note": ""}, {"id": "106986", "series": "", "locations": [{"wall": "MPU", "zone": "H區"}, {"wall": "ASIS", "zone": "4排3"}], "qty": "", "level": "high", "note": ""}, {"id": "114182", "series": "", "locations": [{"wall": "MPU", "zone": "H區"}], "qty": "", "level": "high", "note": ""}, {"id": "100240", "series": "", "locations": [{"wall": "MPU", "zone": "H區"}], "qty": "", "level": "high", "note": ""}, {"id": "110328", "series": "", "locations": [{"wall": "MPU", "zone": "H區"}, {"wall": "ASIS背面", "zone": ""}], "qty": "", "level": "high", "note": ""}, {"id": "118326", "series": "", "locations": [{"wall": "MPU", "zone": "H區"}], "qty": "", "level": "high", "note": ""}, {"id": "112666", "series": "", "locations": [{"wall": "MPU", "zone": "H區"}], "qty": "", "level": "high", "note": ""}, {"id": "128973", "series": "", "locations": [{"wall": "MPU", "zone": "H區"}], "qty": "", "level": "high", "note": ""}, {"id": "114584", "series": "", "locations": [{"wall": "MPU", "zone": "H區"}], "qty": "", "level": "high", "note": ""}, {"id": "114509", "series": "", "locations": [{"wall": "MPU", "zone": "H區"}], "qty": "", "level": "high", "note": ""}, {"id": "119036", "series": "", "locations": [{"wall": "MPU", "zone": "H區"}], "qty": "", "level": "high", "note": ""}, {"id": "117175", "series": "", "locations": [{"wall": "MPU", "zone": "H區"}], "qty": "", "level": "high", "note": ""}, {"id": "130646", "series": "", "locations": [{"wall": "MPU", "zone": "H區"}], "qty": "", "level": "high", "note": ""}, {"id": "157314", "series": "", "locations": [{"wall": "MPU", "zone": "小房間"}], "qty": "", "level": "high", "note": ""}, {"id": "10068193", "series": "", "locations": [{"wall": "MPU", "zone": "小房間"}], "qty": "", "level": "high", "note": ""}, {"id": "111033", "series": "", "locations": [{"wall": "MPU", "zone": "小房間"}], "qty": "", "level": "high", "note": ""}, {"id": "112578", "series": "", "locations": [{"wall": "MPU", "zone": "小房間"}], "qty": "", "level": "high", "note": ""}, {"id": "10003305", "series": "", "locations": [{"wall": "MPU", "zone": "小房間"}], "qty": "", "level": "high", "note": ""}, {"id": "146701", "series": "", "locations": [{"wall": "MPU", "zone": "小房間"}], "qty": "", "level": "high", "note": ""}, {"id": "120748", "series": "", "locations": [{"wall": "MPU", "zone": "小房間"}, {"wall": "ASIS", "zone": "16排7"}], "qty": "", "level": "high", "note": ""}, {"id": "151636", "series": "", "locations": [{"wall": "MPU", "zone": "小房間"}], "qty": "", "level": "high", "note": ""}, {"id": "10043142", "series": "", "locations": [{"wall": "MPU", "zone": "小房間"}], "qty": "", "level": "high", "note": ""}, {"id": "139301", "series": "", "locations": [{"wall": "MPU", "zone": "小房間"}], "qty": "", "level": "high", "note": ""}, {"id": "108904", "series": "", "locations": [{"wall": "MPU", "zone": "小房間"}], "qty": "", "level": "high", "note": ""}, {"id": "122628", "series": "", "locations": [{"wall": "MPU", "zone": "小房間"}], "qty": "", "level": "high", "note": ""}, {"id": "139298", "series": "", "locations": [{"wall": "MPU", "zone": "小房間"}], "qty": "", "level": "high", "note": ""}, {"id": "10049775", "series": "", "locations": [{"wall": "MPU", "zone": "小房間"}], "qty": "", "level": "high", "note": ""}, {"id": "108116", "series": "", "locations": [{"wall": "MPU", "zone": "小房間"}, {"wall": "ASIS背面", "zone": ""}], "qty": "", "level": "high", "note": ""}, {"id": "141414", "series": "", "locations": [{"wall": "MPU", "zone": "小房間"}, {"wall": "ASIS背面", "zone": ""}], "qty": "", "level": "high", "note": ""}, {"id": "102370", "series": "", "locations": [{"wall": "MPU", "zone": "小房間"}], "qty": "", "level": "high", "note": ""}, {"id": "130451", "series": "", "locations": [{"wall": "MPU", "zone": "小房間"}], "qty": "", "level": "high", "note": ""}, {"id": "10050782", "series": "", "locations": [{"wall": "MPU", "zone": "小房間"}], "qty": "", "level": "high", "note": ""}, {"id": "10050806", "series": "", "locations": [{"wall": "MPU", "zone": "小房間"}], "qty": "", "level": "high", "note": ""}, {"id": "116546", "series": "", "locations": [{"wall": "MPU", "zone": "小房間"}], "qty": "", "level": "high", "note": ""}, {"id": "130650", "series": "", "locations": [{"wall": "MPU", "zone": "小房間"}], "qty": "", "level": "high", "note": ""}, {"id": "124639", "series": "", "locations": [{"wall": "MPU", "zone": "小房間"}], "qty": "", "level": "high", "note": ""}, {"id": "109337", "series": "", "locations": [{"wall": "MPU", "zone": "小房間"}], "qty": "", "level": "high", "note": ""}, {"id": "190096", "series": "", "locations": [{"wall": "MPU", "zone": "小房間"}, {"wall": "ASIS", "zone": "14排7"}], "qty": "", "level": "high", "note": ""}, {"id": "124341", "series": "", "locations": [{"wall": "MPU", "zone": "小房間"}], "qty": "", "level": "high", "note": ""}, {"id": "130452", "series": "", "locations": [{"wall": "MPU", "zone": "小房間"}], "qty": "", "level": "high", "note": ""}, {"id": "131341", "series": "", "locations": [{"wall": "MPU", "zone": "小房間"}], "qty": "", "level": "high", "note": ""}, {"id": "111032", "series": "", "locations": [{"wall": "MPU", "zone": "小房間"}], "qty": "", "level": "high", "note": ""}, {"id": "102366", "series": "", "locations": [{"wall": "MPU", "zone": "小房間"}], "qty": "", "level": "high", "note": ""}, {"id": "131342", "series": "", "locations": [{"wall": "MPU", "zone": "小房間"}], "qty": "", "level": "high", "note": ""}, {"id": "114947", "series": "", "locations": [{"wall": "MPU", "zone": "小房間"}, {"wall": "ASIS背面", "zone": ""}], "qty": "", "level": "high", "note": ""}, {"id": "105021", "series": "", "locations": [{"wall": "ASIS", "zone": "1排1"}], "qty": "", "level": "high", "note": ""}, {"id": "124154", "series": "", "locations": [{"wall": "ASIS", "zone": "1排10"}], "qty": "", "level": "high", "note": ""}, {"id": "124982", "series": "", "locations": [{"wall": "ASIS", "zone": "1排11"}], "qty": "", "level": "high", "note": ""}, {"id": "105163", "series": "", "locations": [{"wall": "ASIS", "zone": "1排12"}], "qty": "", "level": "high", "note": ""}, {"id": "119976", "series": "", "locations": [{"wall": "ASIS", "zone": "2排2"}], "qty": "", "level": "high", "note": ""}, {"id": "119046", "series": "", "locations": [{"wall": "ASIS", "zone": "2排3"}], "qty": "", "level": "high", "note": ""}, {"id": "115348", "series": "", "locations": [{"wall": "ASIS", "zone": "2排5"}], "qty": "", "level": "high", "note": ""}, {"id": "10119612", "series": "", "locations": [{"wall": "ASIS", "zone": "2排10"}], "qty": "", "level": "high", "note": ""}, {"id": "100644", "series": "", "locations": [{"wall": "ASIS", "zone": "3排2"}], "qty": "", "level": "high", "note": ""}, {"id": "112571", "series": "", "locations": [{"wall": "ASIS", "zone": "3排3"}], "qty": "", "level": "high", "note": ""}, {"id": "104323", "series": "", "locations": [{"wall": "ASIS", "zone": "3排5"}], "qty": "", "level": "high", "note": ""}, {"id": "104321", "series": "", "locations": [{"wall": "ASIS", "zone": "3排6"}], "qty": "", "level": "high", "note": ""}, {"id": "101367", "series": "", "locations": [{"wall": "ASIS", "zone": "3排8"}], "qty": "", "level": "high", "note": ""}, {"id": "121194", "series": "", "locations": [{"wall": "ASIS", "zone": "3排10"}], "qty": "", "level": "high", "note": ""}, {"id": "191315", "series": "", "locations": [{"wall": "ASIS", "zone": "3排12"}], "qty": "", "level": "high", "note": ""}, {"id": "120925", "series": "", "locations": [{"wall": "ASIS", "zone": "4排1"}], "qty": "", "level": "high", "note": ""}, {"id": "100229", "series": "", "locations": [{"wall": "ASIS", "zone": "4排4"}], "qty": "", "level": "high", "note": ""}, {"id": "100215", "series": "", "locations": [{"wall": "ASIS", "zone": "4排6"}], "qty": "", "level": "high", "note": ""}, {"id": "101344", "series": "", "locations": [{"wall": "ASIS", "zone": "4排9"}], "qty": "", "level": "high", "note": ""}, {"id": "112581", "series": "", "locations": [{"wall": "ASIS", "zone": "4排10"}], "qty": "", "level": "high", "note": ""}, {"id": "117327", "series": "", "locations": [{"wall": "ASIS", "zone": "4排12"}], "qty": "", "level": "high", "note": ""}, {"id": "105100", "series": "", "locations": [{"wall": "ASIS", "zone": "5排1"}], "qty": "", "level": "high", "note": ""}, {"id": "118325", "series": "", "locations": [{"wall": "ASIS", "zone": "5排2"}], "qty": "", "level": "high", "note": ""}, {"id": "100014", "series": "", "locations": [{"wall": "ASIS", "zone": "5排3"}], "qty": "", "level": "high", "note": ""}, {"id": "100211", "series": "", "locations": [{"wall": "ASIS", "zone": "5排6"}], "qty": "", "level": "high", "note": ""}, {"id": "107489", "series": "", "locations": [{"wall": "ASIS", "zone": "5排7"}], "qty": "", "level": "high", "note": ""}, {"id": "190018", "series": "", "locations": [{"wall": "ASIS", "zone": "5排8"}], "qty": "", "level": "high", "note": ""}, {"id": "109567", "series": "", "locations": [{"wall": "ASIS", "zone": "5排9"}], "qty": "", "level": "high", "note": ""}, {"id": "110519", "series": "", "locations": [{"wall": "ASIS", "zone": "5排10"}], "qty": "", "level": "high", "note": ""}, {"id": "195225", "series": "", "locations": [{"wall": "ASIS", "zone": "5排11"}], "qty": "", "level": "high", "note": ""}, {"id": "115349", "series": "", "locations": [{"wall": "ASIS", "zone": "5排12"}], "qty": "", "level": "high", "note": ""}, {"id": "190419", "series": "", "locations": [{"wall": "ASIS", "zone": "6排1"}], "qty": "", "level": "high", "note": ""}, {"id": "109067", "series": "", "locations": [{"wall": "ASIS", "zone": "6排4"}], "qty": "", "level": "high", "note": ""}, {"id": "100344", "series": "", "locations": [{"wall": "ASIS", "zone": "6排5"}], "qty": "", "level": "high", "note": ""}, {"id": "110789", "series": "", "locations": [{"wall": "ASIS", "zone": "6排6"}], "qty": "", "level": "high", "note": ""}, {"id": "116637", "series": "", "locations": [{"wall": "ASIS", "zone": "6排7"}], "qty": "", "level": "high", "note": ""}, {"id": "108443", "series": "", "locations": [{"wall": "ASIS", "zone": "6排9"}], "qty": "", "level": "high", "note": ""}, {"id": "115339", "series": "", "locations": [{"wall": "ASIS", "zone": "6排10"}], "qty": "", "level": "high", "note": ""}, {"id": "131122", "series": "", "locations": [{"wall": "ASIS", "zone": "6排11"}], "qty": "", "level": "high", "note": ""}, {"id": "131123", "series": "", "locations": [{"wall": "ASIS", "zone": "6排11"}], "qty": "", "level": "high", "note": ""}, {"id": "114670", "series": "", "locations": [{"wall": "ASIS", "zone": "6排12"}], "qty": "", "level": "high", "note": ""}, {"id": "124539", "series": "", "locations": [{"wall": "ASIS", "zone": "7排1"}], "qty": "", "level": "high", "note": ""}, {"id": "117001", "series": "", "locations": [{"wall": "ASIS", "zone": "7排2"}], "qty": "", "level": "high", "note": ""}, {"id": "142777", "series": "", "locations": [{"wall": "ASIS", "zone": "7排5"}], "qty": "", "level": "high", "note": ""}, {"id": "109535", "series": "", "locations": [{"wall": "ASIS", "zone": "7排7"}], "qty": "", "level": "high", "note": ""}, {"id": "110438", "series": "", "locations": [{"wall": "ASIS", "zone": "7排8"}], "qty": "", "level": "high", "note": ""}, {"id": "108530", "series": "", "locations": [{"wall": "ASIS", "zone": "7排9"}], "qty": "", "level": "high", "note": ""}, {"id": "102323", "series": "", "locations": [{"wall": "ASIS", "zone": "7排10"}], "qty": "", "level": "high", "note": ""}, {"id": "106569", "series": "", "locations": [{"wall": "ASIS", "zone": "8排1"}], "qty": "", "level": "high", "note": ""}, {"id": "122606", "series": "", "locations": [{"wall": "ASIS", "zone": "8排2"}], "qty": "", "level": "high", "note": ""}, {"id": "100372", "series": "", "locations": [{"wall": "ASIS", "zone": "8排6"}], "qty": "", "level": "high", "note": ""}, {"id": "124386", "series": "", "locations": [{"wall": "ASIS", "zone": "8排8"}], "qty": "", "level": "high", "note": ""}, {"id": "109041", "series": "", "locations": [{"wall": "ASIS", "zone": "8排9"}], "qty": "", "level": "high", "note": ""}, {"id": "130527", "series": "", "locations": [{"wall": "ASIS", "zone": "8排11"}], "qty": "", "level": "high", "note": ""}, {"id": "139430", "series": "", "locations": [{"wall": "ASIS", "zone": "8排12"}], "qty": "", "level": "high", "note": ""}, {"id": "131187", "series": "", "locations": [{"wall": "ASIS", "zone": "9排2"}], "qty": "", "level": "high", "note": ""}, {"id": "10004072", "series": "", "locations": [{"wall": "ASIS", "zone": "9排3"}], "qty": "", "level": "high", "note": ""}, {"id": "113928", "series": "", "locations": [{"wall": "ASIS", "zone": "9排5"}], "qty": "", "level": "high", "note": ""}, {"id": "113287", "series": "", "locations": [{"wall": "ASIS", "zone": "9排6"}], "qty": "", "level": "high", "note": ""}, {"id": "107832", "series": "", "locations": [{"wall": "ASIS", "zone": "9排7"}], "qty": "", "level": "high", "note": ""}, {"id": "124593", "series": "", "locations": [{"wall": "ASIS", "zone": "10排6"}], "qty": "", "level": "high", "note": ""}, {"id": "113301", "series": "", "locations": [{"wall": "ASIS", "zone": "10排7"}], "qty": "", "level": "high", "note": ""}, {"id": "10042365", "series": "", "locations": [{"wall": "ASIS", "zone": "10排10"}], "qty": "", "level": "high", "note": ""}, {"id": "190473", "series": "", "locations": [{"wall": "ASIS", "zone": "11排11"}], "qty": "", "level": "high", "note": ""}, {"id": "116791", "series": "", "locations": [{"wall": "ASIS", "zone": "11排12"}], "qty": "", "level": "high", "note": ""}, {"id": "128671", "series": "", "locations": [{"wall": "ASIS", "zone": "12排1"}], "qty": "", "level": "high", "note": ""}, {"id": "122620", "series": "", "locations": [{"wall": "ASIS", "zone": "12排2"}], "qty": "", "level": "high", "note": ""}, {"id": "128654", "series": "", "locations": [{"wall": "ASIS", "zone": "12排3"}], "qty": "", "level": "high", "note": ""}, {"id": "113434", "series": "", "locations": [{"wall": "ASIS", "zone": "12排6"}], "qty": "", "level": "high", "note": ""}, {"id": "190471", "series": "", "locations": [{"wall": "ASIS", "zone": "12排11"}], "qty": "", "level": "high", "note": ""}, {"id": "124402", "series": "", "locations": [{"wall": "ASIS", "zone": "12排12"}], "qty": "", "level": "high", "note": ""}, {"id": "100854", "series": "", "locations": [{"wall": "ASIS", "zone": "13排1"}], "qty": "", "level": "high", "note": ""}, {"id": "100829", "series": "", "locations": [{"wall": "ASIS", "zone": "13排2"}], "qty": "", "level": "high", "note": ""}, {"id": "130722", "series": "", "locations": [{"wall": "ASIS", "zone": "13排4"}], "qty": "", "level": "high", "note": ""}, {"id": "145718", "series": "", "locations": [{"wall": "ASIS", "zone": "13排6"}], "qty": "", "level": "high", "note": ""}, {"id": "10051660", "series": "", "locations": [{"wall": "ASIS", "zone": "13排7"}], "qty": "", "level": "high", "note": ""}, {"id": "159945", "series": "", "locations": [{"wall": "ASIS", "zone": "13排8"}], "qty": "", "level": "high", "note": ""}, {"id": "152529", "series": "", "locations": [{"wall": "ASIS", "zone": "13排9"}], "qty": "", "level": "high", "note": ""}, {"id": "10040039", "series": "", "locations": [{"wall": "ASIS", "zone": "13排10"}], "qty": "", "level": "high", "note": ""}, {"id": "103095", "series": "", "locations": [{"wall": "ASIS", "zone": "13排12"}], "qty": "", "level": "high", "note": ""}, {"id": "100710", "series": "", "locations": [{"wall": "ASIS", "zone": "14排2"}], "qty": "", "level": "high", "note": ""}, {"id": "144575", "series": "", "locations": [{"wall": "ASIS", "zone": "14排4"}], "qty": "", "level": "high", "note": ""}, {"id": "151708", "series": "", "locations": [{"wall": "ASIS", "zone": "14排5"}], "qty": "", "level": "high", "note": ""}, {"id": "10093081", "series": "", "locations": [{"wall": "ASIS", "zone": "14排6"}], "qty": "", "level": "high", "note": ""}, {"id": "139434", "series": "", "locations": [{"wall": "ASIS", "zone": "14排6"}], "qty": "", "level": "high", "note": ""}, {"id": "191407", "series": "", "locations": [{"wall": "ASIS", "zone": "14排8"}], "qty": "", "level": "high", "note": ""}, {"id": "153548", "series": "", "locations": [{"wall": "ASIS", "zone": "14排10"}], "qty": "", "level": "high", "note": ""}, {"id": "10003068", "series": "", "locations": [{"wall": "ASIS", "zone": "14排12"}], "qty": "", "level": "high", "note": ""}, {"id": "144574", "series": "", "locations": [{"wall": "ASIS", "zone": "15排4"}], "qty": "", "level": "high", "note": ""}, {"id": "10093082", "series": "", "locations": [{"wall": "ASIS", "zone": "15排6"}], "qty": "", "level": "high", "note": ""}, {"id": "139435", "series": "", "locations": [{"wall": "ASIS", "zone": "15排6"}], "qty": "", "level": "high", "note": ""}, {"id": "103518", "series": "", "locations": [{"wall": "ASIS", "zone": "15排8"}], "qty": "", "level": "high", "note": ""}, {"id": "153549", "series": "", "locations": [{"wall": "ASIS", "zone": "15排10"}], "qty": "", "level": "high", "note": ""}, {"id": "153552", "series": "", "locations": [{"wall": "ASIS", "zone": "15排11"}], "qty": "", "level": "high", "note": ""}, {"id": "153807", "series": "", "locations": [{"wall": "ASIS", "zone": "15排12"}], "qty": "", "level": "high", "note": ""}, {"id": "110912", "series": "", "locations": [{"wall": "ASIS", "zone": "16排1"}], "qty": "", "level": "high", "note": ""}, {"id": "100505", "series": "", "locations": [{"wall": "ASIS", "zone": "16排2"}], "qty": "", "level": "high", "note": ""}, {"id": "120189", "series": "", "locations": [{"wall": "ASIS", "zone": "16排3"}], "qty": "", "level": "high", "note": ""}, {"id": "100514", "series": "", "locations": [{"wall": "ASIS", "zone": "16排3"}], "qty": "", "level": "high", "note": ""}, {"id": "153761", "series": "", "locations": [{"wall": "ASIS", "zone": "16排5"}], "qty": "", "level": "high", "note": ""}, {"id": "158461", "series": "", "locations": [{"wall": "ASIS", "zone": "16排9"}], "qty": "", "level": "high", "note": ""}, {"id": "150491", "series": "", "locations": [{"wall": "ASIS", "zone": "16排12"}], "qty": "", "level": "high", "note": ""}, {"id": "124328", "series": "", "locations": [{"wall": "ASIS背面", "zone": ""}], "qty": "", "level": "high", "note": ""}, {"id": "10107965", "series": "", "locations": [{"wall": "ASIS背面", "zone": ""}], "qty": "", "level": "high", "note": ""}, {"id": "109338", "series": "", "locations": [{"wall": "ASIS背面", "zone": ""}], "qty": "", "level": "high", "note": ""}, {"id": "10107957", "series": "", "locations": [{"wall": "ASIS背面", "zone": ""}], "qty": "", "level": "high", "note": ""}, {"id": "109221", "series": "", "locations": [{"wall": "ASIS背面", "zone": ""}], "qty": "", "level": "high", "note": ""}, {"id": "10050684", "series": "", "locations": [{"wall": "ASIS背面", "zone": ""}], "qty": "", "level": "high", "note": ""}, {"id": "126916", "series": "", "locations": [{"wall": "ASIS背面", "zone": ""}], "qty": "", "level": "high", "note": ""}, {"id": "117687", "series": "", "locations": [{"wall": "ASIS背面", "zone": ""}], "qty": "", "level": "high", "note": ""}, {"id": "152650", "series": "", "locations": [{"wall": "ASIS背面", "zone": ""}], "qty": "", "level": "high", "note": ""}, {"id": "130531", "series": "", "locations": [{"wall": "ASIS背面", "zone": ""}], "qty": "", "level": "high", "note": ""}, {"id": "139285", "series": "", "locations": [{"wall": "ASIS背面", "zone": ""}], "qty": "", "level": "high", "note": ""}, {"id": "139294", "series": "", "locations": [{"wall": "ASIS背面", "zone": ""}], "qty": "", "level": "high", "note": ""}, {"id": "10051085", "series": "", "locations": [{"wall": "ASIS背面", "zone": ""}], "qty": "", "level": "high", "note": ""}, {"id": "10056864", "series": "", "locations": [{"wall": "ASIS背面", "zone": ""}], "qty": "", "level": "high", "note": ""}, {"id": "10056885", "series": "", "locations": [{"wall": "ASIS背面", "zone": ""}], "qty": "", "level": "high", "note": ""}, {"id": "10058633", "series": "", "locations": [{"wall": "ASIS背面", "zone": ""}], "qty": "", "level": "high", "note": ""}, {"id": "142400", "series": "", "locations": [{"wall": "ASIS背面", "zone": ""}], "qty": "", "level": "high", "note": ""}, {"id": "10050403", "series": "", "locations": [{"wall": "ASIS背面", "zone": ""}], "qty": "", "level": "high", "note": ""}, {"id": "10045938", "series": "", "locations": [{"wall": "ASIS背面", "zone": ""}], "qty": "", "level": "high", "note": ""}, {"id": "100707", "series": "", "locations": [{"wall": "ASIS背面", "zone": ""}], "qty": "", "level": "high", "note": ""}, {"id": "192158", "series": "", "locations": [{"wall": "ASIS背面", "zone": ""}], "qty": "", "level": "high", "note": ""}, {"id": "10045937", "series": "", "locations": [{"wall": "ASIS背面", "zone": ""}], "qty": "", "level": "high", "note": ""}, {"id": "10045911", "series": "", "locations": [{"wall": "ASIS背面", "zone": ""}], "qty": "", "level": "high", "note": ""}, {"id": "144261", "series": "", "locations": [{"wall": "ASIS背面", "zone": ""}], "qty": "", "level": "high", "note": ""}, {"id": "10061614", "series": "", "locations": [{"wall": "ASIS背面", "zone": ""}], "qty": "", "level": "high", "note": ""}, {"id": "157409", "series": "", "locations": [{"wall": "ASIS背面", "zone": ""}], "qty": "", "level": "high", "note": ""}, {"id": "107091", "series": "", "locations": [{"wall": "ASIS背面", "zone": ""}], "qty": "", "level": "high", "note": ""}, {"id": "10082805", "series": "", "locations": [{"wall": "ASIS背面", "zone": ""}], "qty": "", "level": "high", "note": ""}, {"id": "152059", "series": "", "locations": [{"wall": "ASIS背面", "zone": ""}], "qty": "", "level": "high", "note": ""}, {"id": "153550", "series": "", "locations": [{"wall": "ASIS背面", "zone": ""}], "qty": "", "level": "high", "note": ""}, {"id": "115753", "series": "", "locations": [{"wall": "ASIS背面", "zone": ""}], "qty": "", "level": "high", "note": ""}, {"id": "149842", "series": "", "locations": [{"wall": "ASIS背面", "zone": ""}], "qty": "", "level": "high", "note": ""}, {"id": "107103", "series": "", "locations": [{"wall": "ASIS背面", "zone": ""}], "qty": "", "level": "high", "note": ""}, {"id": "104895", "series": "", "locations": [{"wall": "ASIS背面", "zone": ""}], "qty": "", "level": "high", "note": ""}, {"id": "107616", "series": "", "locations": [{"wall": "ASIS背面", "zone": ""}], "qty": "", "level": "high", "note": ""}, {"id": "115983", "series": "", "locations": [{"wall": "ASIS背面", "zone": ""}], "qty": "", "level": "high", "note": ""}, {"id": "190949", "series": "", "locations": [{"wall": "ASIS背面", "zone": ""}], "qty": "", "level": "high", "note": ""}, {"id": "190950", "series": "", "locations": [{"wall": "ASIS背面", "zone": ""}], "qty": "", "level": "high", "note": ""}, {"id": "190948", "series": "", "locations": [{"wall": "ASIS背面", "zone": ""}], "qty": "", "level": "high", "note": ""}, {"id": "190905", "series": "", "locations": [{"wall": "ASIS背面", "zone": ""}], "qty": "", "level": "high", "note": ""}, {"id": "128864", "series": "", "locations": [{"wall": "ASIS背面", "zone": ""}], "qty": "", "level": "high", "note": ""}, {"id": "139212", "series": "", "locations": [{"wall": "ASIS背面", "zone": ""}], "qty": "", "level": "high", "note": ""}, {"id": "101313", "series": "", "locations": [{"wall": "ASIS背面", "zone": ""}], "qty": "", "level": "high", "note": ""}, {"id": "153748", "series": "", "locations": [{"wall": "ASIS背面", "zone": ""}], "qty": "", "level": "high", "note": ""}, {"id": "130485", "series": "", "locations": [{"wall": "ASIS背面", "zone": ""}], "qty": "", "level": "high", "note": ""}, {"id": "10081882", "series": "", "locations": [{"wall": "ASIS背面", "zone": ""}], "qty": "", "level": "high", "note": ""}, {"id": "110389", "series": "", "locations": [{"wall": "ASIS背面", "zone": ""}], "qty": "", "level": "high", "note": ""}, {"id": "10082205", "series": "", "locations": [{"wall": "ASIS背面", "zone": ""}], "qty": "", "level": "high", "note": ""}, {"id": "190169", "series": "", "locations": [{"wall": "ASIS背面", "zone": ""}], "qty": "", "level": "high", "note": ""}, {"id": "10082290", "series": "", "locations": [{"wall": "ASIS背面", "zone": ""}], "qty": "", "level": "high", "note": ""}, {"id": "10082310", "series": "", "locations": [{"wall": "ASIS背面", "zone": ""}], "qty": "", "level": "high", "note": ""}, {"id": "10064763", "series": "", "locations": [{"wall": "ASIS背面", "zone": ""}], "qty": "", "level": "high", "note": ""}, {"id": "10062668", "series": "", "locations": [{"wall": "ASIS背面", "zone": ""}], "qty": "", "level": "high", "note": ""}, {"id": "190170", "series": "", "locations": [{"wall": "ASIS背面", "zone": ""}], "qty": "", "level": "high", "note": ""}, {"id": "146114", "series": "", "locations": [{"wall": "ASIS背面", "zone": ""}], "qty": "", "level": "high", "note": ""}, {"id": "152527", "series": "", "locations": [{"wall": "ASIS背面", "zone": ""}], "qty": "", "level": "high", "note": ""}, {"id": "123742", "series": "", "locations": [{"wall": "ASIS背面", "zone": ""}], "qty": "", "level": "high", "note": ""}, {"id": "158457", "series": "", "locations": [{"wall": "ASIS背面", "zone": ""}], "qty": "", "level": "high", "note": ""}, {"id": "10047244", "series": "", "locations": [{"wall": "ASIS背面", "zone": ""}], "qty": "", "level": "high", "note": ""}, {"id": "10094827", "series": "", "locations": [{"wall": "ASIS背面", "zone": ""}], "qty": "", "level": "high", "note": ""}, {"id": "152526", "series": "", "locations": [{"wall": "ASIS背面", "zone": ""}], "qty": "", "level": "high", "note": ""}, {"id": "152497", "series": "", "locations": [{"wall": "ASIS背面", "zone": ""}], "qty": "", "level": "high", "note": ""}, {"id": "194377", "series": "", "locations": [{"wall": "ASIS背面", "zone": ""}], "qty": "", "level": "high", "note": ""}, {"id": "101513", "series": "", "locations": [{"wall": "ASIS背面", "zone": ""}], "qty": "", "level": "high", "note": ""}, {"id": "139477", "series": "", "locations": [{"wall": "ASIS背面", "zone": ""}], "qty": "", "level": "high", "note": ""}, {"id": "133127", "series": "", "locations": [{"wall": "ASIS背面", "zone": ""}], "qty": "", "level": "high", "note": ""}, {"id": "10036248", "series": "", "locations": [{"wall": "ASIS背面", "zone": ""}], "qty": "", "level": "high", "note": ""}, {"id": "139467", "series": "", "locations": [{"wall": "ASIS背面", "zone": ""}], "qty": "", "level": "high", "note": ""}, {"id": "139468", "series": "", "locations": [{"wall": "ASIS背面", "zone": ""}], "qty": "", "level": "high", "note": ""}, {"id": "155200", "series": "", "locations": [{"wall": "ASIS背面", "zone": ""}], "qty": "", "level": "high", "note": ""}, {"id": "155201", "series": "", "locations": [{"wall": "ASIS背面", "zone": ""}], "qty": "", "level": "high", "note": ""}, {"id": "10036250", "series": "", "locations": [{"wall": "ASIS背面", "zone": ""}], "qty": "", "level": "high", "note": ""}, {"id": "155197", "series": "", "locations": [{"wall": "ASIS背面", "zone": ""}], "qty": "", "level": "high", "note": ""}, {"id": "155198", "series": "", "locations": [{"wall": "ASIS背面", "zone": ""}], "qty": "", "level": "high", "note": ""}, {"id": "10003221", "series": "", "locations": [{"wall": "ASIS背面", "zone": ""}], "qty": "", "level": "high", "note": ""}, {"id": "133148", "series": "", "locations": [{"wall": "ASIS背面", "zone": ""}], "qty": "", "level": "high", "note": ""}, {"id": "10003241", "series": "", "locations": [{"wall": "ASIS背面", "zone": ""}], "qty": "", "level": "high", "note": ""}, {"id": "105160", "series": "", "locations": [{"wall": "ASIS背面", "zone": ""}], "qty": "", "level": "high", "note": ""}, {"id": "128857", "series": "", "locations": [{"wall": "ASIS背面", "zone": ""}], "qty": "", "level": "high", "note": ""}, {"id": "194077", "series": "", "locations": [{"wall": "ASIS背面", "zone": ""}], "qty": "", "level": "high", "note": ""}, {"id": "104850", "series": "", "locations": [{"wall": "ASIS背面", "zone": ""}], "qty": "", "level": "high", "note": ""}, {"id": "117363", "series": "", "locations": [{"wall": "ASIS背面", "zone": ""}], "qty": "", "level": "high", "note": ""}, {"id": "104864", "series": "", "locations": [{"wall": "ASIS背面", "zone": ""}], "qty": "", "level": "high", "note": ""}];

let parts=[],orders=[],prices={};
let loggedIn=false,editIdx=null,deleteIdx=null;
let pageQ=1,pageA=1,pageRec=1;
let filteredQ=[],filteredA=[],filteredRec=[];
let savedPageA=1,currentAdminTab='inventory',editRecNoteIdx=null;
let cart={};
let expandedOrders=new Set();

function load(){
  try{const s=localStorage.getItem(STORAGE_KEY);parts=s?JSON.parse(s):JSON.parse(JSON.stringify(DEFAULT_PARTS));}catch(e){parts=JSON.parse(JSON.stringify(DEFAULT_PARTS));}
  try{const o=localStorage.getItem(ORDERS_KEY);orders=o?JSON.parse(o):[];}catch(e){orders=[];}
  try{const p=localStorage.getItem(PRICES_KEY);prices=p?JSON.parse(p):{};}catch(e){prices={};}
  checkExpiry();
}
function save(){localStorage.setItem(STORAGE_KEY,JSON.stringify(parts));}
function saveOrders(){localStorage.setItem(ORDERS_KEY,JSON.stringify(orders));}
function savePrices(){localStorage.setItem(PRICES_KEY,JSON.stringify(prices));}

function getPrice(id){return prices[id]!==undefined?prices[id]:DEFAULT_PRICE;}
function setPrice(id,val){prices[id]=parseFloat(val)||DEFAULT_PRICE;savePrices();drawA();}

function checkExpiry(){
  const now=new Date();let changed=false;
  orders.forEach(o=>{
    if(o.status==='已通知'&&o.notifiedDate){
      const diff=(now-new Date(o.notifiedDate))/(1000*60*60*24);
      if(diff>=30){o.status='逾期未領';changed=true;}
    }
  });
  if(changed)saveOrders();
}

function expiryDateStr(notifiedDate){
  if(!notifiedDate)return'—';
  const d=new Date(notifiedDate);d.setDate(d.getDate()+30);
  return d.toLocaleDateString('zh-TW',{year:'numeric',month:'2-digit',day:'2-digit'});
}

function lvClass(lv){return{high:'lv-high',low:'lv-low',out:'lv-out',na:'lv-na'}[lv]||'lv-high';}
function lvLabel(lv){return{high:'充足',low:'偏低',out:'缺貨',na:'缺貨無法訂購'}[lv]||'充足';}
function wallTagClass(w){if(w==='MPU')return'tag-mpu';if(w==='ASIS')return'tag-asis';return'tag-asisb';}
function statusClass(s){return{'未處理':'st-pending','已訂購':'st-ordered','已到貨':'st-arrived','已預留':'st-reserved','已通知':'st-notified','逾期未領':'st-expired','缺貨無法訂購':'st-unavail'}[s]||'st-pending';}

function locHTML(locs){
  if(!locs||!locs.length)return'<span style="color:#bbb">—</span>';
  return'<div class="locations">'+locs.map(l=>{const label=l.zone?`${l.wall}-${l.zone}`:l.wall;return`<span class="tag ${wallTagClass(l.wall)}">${label}</span>`;}).join('')+'</div>';
}

function todayStr(){return new Date().toLocaleDateString('zh-TW',{year:'numeric',month:'2-digit',day:'2-digit'});}
function todayISO(){return new Date().toISOString().split('T')[0];}
function monthStr(iso){if(!iso)return'';return iso.substring(0,7);}

function showToast(msg){const t=document.getElementById('toast');t.textContent=msg;t.classList.add('show');setTimeout(()=>t.classList.remove('show'),2500);}

function getSeriesList(){return Array.from(new Set(parts.map(p=>p.series).filter(Boolean))).sort();}
function updateSeriesDropdowns(){
  const list=getSeriesList();
  ['seriesQ','seriesA'].forEach(id=>{
    const sel=document.getElementById(id);if(!sel)return;
    const cur=sel.value;
    sel.innerHTML='<option value="">全部系列</option>'+list.map(s=>`<option value="${s}">${s}</option>`).join('');
    sel.value=cur;
  });
}
function updateMonthDropdown(){
  const months=Array.from(new Set(orders.map(o=>monthStr(o.isoDate)).filter(Boolean))).sort().reverse();
  const sel=document.getElementById('recMonthFilter');if(!sel)return;
  const cur=sel.value;
  sel.innerHTML='<option value="">全部月份</option>'+months.map(m=>`<option value="${m}">${m.replace('-','年')}月</option>`).join('');
  if(months.includes(cur))sel.value=cur;
}
function updateExpiredBadge(){
  checkExpiry();
  const cnt=orders.filter(o=>o.status==='逾期未領').length;
  const badge=document.getElementById('expiredBadge');
  if(cnt>0){badge.style.display='';badge.textContent=cnt;}else badge.style.display='none';
}

/* CART */
function cartItemCount(){return Object.values(cart).reduce((a,c)=>a+(c.qty||1),0);}
function cartTotalAmt(){return Object.values(cart).reduce((a,c)=>a+getPrice(c.id)*(c.qty||1),0);}
function updateCartBar(){
  const cnt=Object.keys(cart).length;
  document.getElementById('cartBar').style.display=cnt>0?'flex':'none';
  document.getElementById('cartCount').textContent=cnt;
  document.getElementById('cartTotal').textContent=cartTotalAmt().toFixed(0);
}

function addToCart(id,type){
  const qEl=document.getElementById('qty_'+id);
  const qty=qEl?Math.max(1,parseInt(qEl.value)||1):1;
  if(cart[id]){cart[id].qty+=qty;cart[id].type=type;}
  else cart[id]={id,type,qty};
  updateCartBar();showToast(id+' 已加入購物車');
  drawQ();
}
function addUnknownToCart(){
  const searchId=document.getElementById('searchQ').value.trim();
  const qty=Math.max(1,parseInt(document.getElementById('noResultsQty').value)||1);
  if(!searchId){showToast('請先輸入零件編號');return;}
  if(cart[searchId]){cart[searchId].qty+=qty;}
  else cart[searchId]={id:searchId,type:'訂購',qty};
  updateCartBar();showToast(searchId+' 已加入購物車');
}

function openCartModal(){
  const items=Object.values(cart);
  const wrap=document.getElementById('cartItemsWrap');
  const empty=document.getElementById('cartEmpty');
  const totalRow=document.getElementById('cartTotalRow');
  if(!items.length){wrap.innerHTML='';empty.style.display='';totalRow.style.display='none';}
  else{
    empty.style.display='none';totalRow.style.display='flex';
    wrap.innerHTML=items.map(c=>{
      const p=parts.find(x=>x.id===c.id);
      const price=getPrice(c.id);
      const typeLabel=c.type==='預留'?'預留':'訂購';
      const typeClass=c.type==='預留'?'tag-reserve':'tag-order';
      return`<div class="cart-item-row">
        <span class="cart-item-id">${c.id}</span>
        <span class="cart-item-type ${typeClass}">${typeLabel}</span>
        <span style="flex:1;font-size:12px;color:#888">${p?p.series||'':''}</span>
        <span style="font-size:12px;color:#888;margin-right:4px">$${price}×</span>
        <input class="cart-item-qty-input" type="number" min="1" value="${c.qty}" onchange="updateCartQty('${c.id}',this.value)">
        <span class="cart-item-price">$${(price*c.qty).toFixed(0)}</span>
        <button class="cart-item-remove" onclick="removeFromCart('${c.id}')">×</button>
      </div>`;
    }).join('');
    document.getElementById('cartTotalDisplay').textContent='$'+cartTotalAmt().toFixed(0);
  }
  document.getElementById('cartName').value='';
  document.getElementById('cartPhone').value='';
  document.getElementById('cartCSC').value='';
  document.getElementById('cartNote').value='';
  document.getElementById('cartErr').textContent='';
  document.getElementById('cartModal').classList.add('show');
}
function updateCartQty(id,val){
  if(cart[id])cart[id].qty=Math.max(1,parseInt(val)||1);
  document.getElementById('cartTotalDisplay').textContent='$'+cartTotalAmt().toFixed(0);
  document.getElementById('cartTotal').textContent=cartTotalAmt().toFixed(0);
}
function removeFromCart(id){delete cart[id];updateCartBar();openCartModal();}
function closeCartModal(){document.getElementById('cartModal').classList.remove('show');}

function submitCart(){
  const name=document.getElementById('cartName').value.trim();
  const phone=document.getElementById('cartPhone').value.trim();
  if(!name){document.getElementById('cartErr').textContent='請輸入聯絡人姓名';return;}
  if(!phone){document.getElementById('cartErr').textContent='請輸入聯絡電話';return;}
  const items=Object.values(cart);
  if(!items.length){document.getElementById('cartErr').textContent='購物車是空的';return;}
  const orderId='ORD'+Date.now();
  const totalAmt=cartTotalAmt();
  const orderItems=items.map(c=>({id:c.id,type:c.type,qty:c.qty,price:getPrice(c.id),subtotal:getPrice(c.id)*c.qty,series:(parts.find(p=>p.id===c.id)||{}).series||''}));
  orders.push({
    orderId,type:'購物車',items:orderItems,
    totalAmt,name,phone,
    csc:document.getElementById('cartCSC').value,
    note:document.getElementById('cartNote').value,
    date:todayStr(),isoDate:todayISO(),
    status:'未處理',notifiedDate:null,notifiedInputDate:null,time:new Date().toISOString()
  });
  saveOrders();closeCartModal();cart={};updateCartBar();updateExpiredBadge();drawQ();
  document.getElementById('savedTitle').textContent='訂購單已送出';
  document.getElementById('savedMsg').textContent=`共 ${orderItems.length} 項零件，合計 $${totalAmt.toFixed(0)}，聯絡人 ${name}`;
  document.getElementById('savedModal').classList.add('show');
}

function printCartForm(){
  const name=document.getElementById('cartName').value||'—';
  const phone=document.getElementById('cartPhone').value||'—';
  const csc=document.getElementById('cartCSC').value||'—';
  const note=document.getElementById('cartNote').value||'—';
  const items=Object.values(cart);
  const rows=items.map(c=>{const price=getPrice(c.id);return`<tr><td>${c.id}</td><td>${(parts.find(p=>p.id===c.id)||{}).series||''}</td><td>${c.type}</td><td>${c.qty}</td><td>$${price}</td><td>$${(price*c.qty).toFixed(0)}</td></tr>`;}).join('');
  const total=cartTotalAmt().toFixed(0);
  const w=window.open('','_blank');
  w.document.write(`<!DOCTYPE html><html><head><meta charset="UTF-8"><title>零件訂購單</title>
  <style>body{font-family:Arial,sans-serif;padding:40px;max-width:700px;margin:0 auto;}
  h1{font-size:20px;border-bottom:2px solid #000;padding-bottom:10px;margin-bottom:20px;}
  .row{display:flex;margin-bottom:10px;font-size:14px;}.label{width:100px;font-weight:bold;color:#555;flex-shrink:0;}
  .val{flex:1;border-bottom:1px solid #ccc;padding-bottom:4px;min-height:22px;}
  table{width:100%;border-collapse:collapse;margin-top:20px;}th,td{border:1px solid #ddd;padding:8px;font-size:13px;}th{background:#f5f5f5;}
  .total-row td{font-weight:bold;background:#f9f9f9;}
  .footer{margin-top:40px;font-size:12px;color:#888;}@media print{body{padding:20px;}}</style></head>
  <body><h1>零件訂購單</h1>
  <div class="row"><span class="label">聯絡人</span><span class="val">${name}</span></div>
  <div class="row"><span class="label">電話</span><span class="val">${phone}</span></div>
  <div class="row"><span class="label">客服處理人</span><span class="val">${csc}</span></div>
  <div class="row"><span class="label">日期</span><span class="val">${todayStr()}</span></div>
  <div class="row"><span class="label">備註</span><span class="val">${note}</span></div>
  <table><thead><tr><th>零件編號</th><th>系列</th><th>類型</th><th>數量</th><th>單價</th><th>小計</th></tr></thead>
  <tbody>${rows}<tr class="total-row"><td colspan="5" style="text-align:right">合計</td><td>$${total}</td></tr></tbody></table>
  <div class="footer">列印時間：${new Date().toLocaleString('zh-TW')}</div>
  <script>window.onload=function(){window.print();}<\/script></body></html>`);
  w.document.close();
}

/* PAGE NAV */
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
  if(tab==='records'){checkExpiry();updateMonthDropdown();renderRecords();}
  else{renderAdmin();renderStatsA();}
}

/* STATS */
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

/* QUERY */
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
  if(!filteredQ.length){
    tbody.innerHTML='';noR.style.display='';
    document.getElementById('pageInfoQ').textContent='';
    document.getElementById('prevQ').disabled=true;document.getElementById('nextQ').disabled=true;return;
  }
  noR.style.display='none';
  const tp=Math.ceil(filteredQ.length/PAGE_SIZE);if(pageQ>tp)pageQ=tp;
  const slice=filteredQ.slice((pageQ-1)*PAGE_SIZE,pageQ*PAGE_SIZE);
  tbody.innerHTML=slice.map(p=>{
    const inCart=!!cart[p.id];
    const isOut=p.level==='out';
    const isNA=p.level==='na';
    const canOrder=isOut;
    const canReserve=p.level==='high'||p.level==='low';
    let actionHTML='';
    if(isNA){
      actionHTML='<span style="font-size:11px;color:#888">無法訂購</span>';
    } else if(isOut){
      actionHTML=`<div class="part-actions">
        <span class="out-hint">無庫存・預訂需 1-2 個月</span>
        <input type="number" id="qty_${p.id}" min="1" value="1" class="qty-input">
        <button class="btn-add-cart" onclick="addToCart('${p.id}','加入購物車')">+ 加入購物車</button>
      </div>`;
    } else {
      const inCartBadge=inCart?`<span style="font-size:11px;color:#3B6D11;margin-left:4px">✓ 已入車</span>`:'';
      actionHTML=`<div class="part-actions">
        <input type="number" id="qty_${p.id}" min="1" value="1" class="qty-input">
        <button class="btn-reserve-add" onclick="addToCart('${p.id}','加入購物車')">加入購物車</button>
        ${inCartBadge}
      </div>`;
    }
    return`<tr>
      <td class="mono">${p.id}</td>
      <td>${p.series?`<span class="tag tag-series">${p.series}</span>`:'<span style="color:#bbb">—</span>'}</td>
      <td>${locHTML(p.locations)}</td>
      <td style="color:#555">${p.qty!==''?p.qty:'—'}</td>
      <td><span class="badge ${lvClass(p.level)}">${lvLabel(p.level)}</span></td>
      <td class="note-cell" title="${p.note||''}">${p.note||'—'}</td>
      <td>${actionHTML}</td>
    </tr>`;
  }).join('');
  document.getElementById('pageInfoQ').textContent=`第 ${pageQ} / ${tp} 頁，共 ${filteredQ.length} 筆`;
  document.getElementById('prevQ').disabled=pageQ<=1;
  document.getElementById('nextQ').disabled=pageQ>=tp;
}

/* ADMIN INVENTORY */
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
function changePageA(d){pageA+=d;drawA();}
function drawA(){
  const tbody=document.getElementById('tbodyA');
  const noR=document.getElementById('noResultsA');
  if(!filteredA.length){tbody.innerHTML='';noR.style.display='';
    document.getElementById('pageInfoA').textContent='';
    document.getElementById('prevA').disabled=true;document.getElementById('nextA').disabled=true;return;}
  noR.style.display='none';
  const tp=Math.ceil(filteredA.length/PAGE_SIZE);if(pageA>tp)pageA=tp;
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
    <td><input class="price-input" type="number" min="0" step="0.5" value="${getPrice(p.id)}" onchange="setPrice('${p.id}',this.value)"> <span style="font-size:11px;color:#888">元</span></td>
    <td class="note-cell" title="${p.note||''}">${p.note||'—'}</td>
    <td><div style="display:flex;gap:4px">
      <button class="btn btn-sm" onclick="openEditModal(${p._i})">編輯</button>
      <button class="btn btn-sm btn-danger" onclick="openDelModal(${p._i})">刪除</button>
    </div></td>
  </tr>`).join('');
  document.getElementById('pageInfoA').textContent=`第 ${pageA} / ${tp} 頁，共 ${filteredA.length} 筆`;
  document.getElementById('prevA').disabled=pageA<=1;
  document.getElementById('nextA').disabled=pageA>=tp;
}
function quickLevel(i,val){
  parts[i].level=val;save();filterA();
  const tp=Math.ceil(filteredA.length/PAGE_SIZE);if(pageA>tp&&tp>0)pageA=tp;drawA();renderStatsA();
}

/* RECORDS */
function filterRec(){
  const q=document.getElementById('searchRec').value.trim().toLowerCase();
  const type=document.getElementById('recTypeFilter').value;
  const month=document.getElementById('recMonthFilter').value;
  const status=document.getElementById('recStatusFilter').value;
  filteredRec=orders.map((o,i)=>({...o,_i:i})).filter(o=>{
    const searchText=(o.partId||'')+(o.phone||'')+(o.items||[]).map(it=>it.id).join('');
    if(q&&!searchText.toLowerCase().includes(q)&&!(o.name||'').toLowerCase().includes(q))return false;
    if(type){
      if(o.type==='購物車'){if(type!=='訂購'&&!o.items.some(it=>it.type===type))return false;}
      else if(o.type!==type)return false;
    }
    if(month&&monthStr(o.isoDate)!==month)return false;
    if(status&&o.status!==status)return false;
    return true;
  }).reverse();
}
function renderRecords(){pageRec=1;filterRec();drawRec();updateExpiredBadge();}
function changePageRec(d){pageRec+=d;drawRec();}
function toggleOrder(orderId){
  if(expandedOrders.has(orderId))expandedOrders.delete(orderId);
  else expandedOrders.add(orderId);
  drawRec();
}
function drawRec(){
  const tbody=document.getElementById('tbodyRec');
  const noR=document.getElementById('noResultsRec');
  if(!filteredRec.length){tbody.innerHTML='';noR.style.display='';
    document.getElementById('pageInfoRec').textContent='';
    document.getElementById('prevRec').disabled=true;document.getElementById('nextRec').disabled=true;return;}
  noR.style.display='none';
  const tp=Math.ceil(filteredRec.length/PAGE_SIZE);if(pageRec>tp)pageRec=tp;
  const slice=filteredRec.slice((pageRec-1)*PAGE_SIZE,pageRec*PAGE_SIZE);
  let html='';
  slice.forEach(o=>{
    const isExpired=o.status==='逾期未領';
    const isNotified=o.status==='已通知';
    const rowClass=isExpired?'row-expired':isNotified?'row-notified':'';
    const isCart=o.type==='購物車';
    const expanded=expandedOrders.has(o.orderId||o._i);
    const typeLabel=isCart?'購物車':'<span style="color:'+(o.type==='訂購'?'#A32D2D':'#3C3489')+'">'+o.type+'</span>';
    const totalDisplay=o.totalAmt!==undefined?`$${parseFloat(o.totalAmt).toFixed(0)}`:'—';
    const expiryDisplay=isNotified&&o.notifiedDate?expiryDateStr(o.notifiedDate):isExpired&&o.notifiedDate?`<span style="color:#A32D2D;font-size:11px">${expiryDateStr(o.notifiedDate)}</span>`:'—';
    const notifiedInputVal=o.notifiedInputDate||o.notifiedDate||'';
    const itemSummary=isCart&&o.items?`<span style="font-size:11px;color:#888">${o.items.length}項零件</span> <button class="order-expand-btn" onclick="toggleOrder('${o.orderId||o._i}')">${expanded?'收起▲':'展開▼'}</button>`:'';
    const partDisplay=isCart?itemSummary:`<span class="mono">${o.partId||'—'}</span>`;

    html+=`<tr class="order-row ${rowClass}">
      <td>${partDisplay}</td>
      <td>${typeLabel}</td>
      <td>${o.name||'—'}</td>
      <td>${o.phone||'—'}</td>
      <td>${o.csc||'—'}</td>
      <td style="white-space:nowrap;font-size:12px">${o.date||'—'}</td>
      <td>
        <select class="status-select" onchange="updateRecStatus(${o._i},this.value)">
          <option value="未處理" ${o.status==='未處理'?'selected':''}>未處理</option>
          <option value="已訂購" ${o.status==='已訂購'?'selected':''}>已訂購</option>
          <option value="已到貨" ${o.status==='已到貨'?'selected':''}>已到貨</option>
          <option value="已預留" ${o.status==='已預留'?'selected':''}>已預留</option>
          <option value="已通知" ${o.status==='已通知'?'selected':''}>已通知</option>
          <option value="逾期未領" ${o.status==='逾期未領'?'selected':''}>逾期未領</option>
          <option value="缺貨無法訂購" ${o.status==='缺貨無法訂購'?'selected':''}>缺貨無法訂購</option>
        </select>
        <span class="badge ${statusClass(o.status)}" style="margin-left:3px;font-size:10px">${o.status}</span>
      </td>
      <td><input type="date" class="notified-date-input" value="${notifiedInputVal}" onchange="updateNotifiedDate(${o._i},this.value)"></td>
      <td style="font-size:12px">${expiryDisplay}</td>
      <td class="order-total-cell">${totalDisplay}</td>
      <td class="note-cell" title="${o.note||''}">${o.note||'—'}</td>
      <td><div style="display:flex;gap:4px">
        <button class="btn btn-sm" onclick="openRecNoteModal(${o._i})">備註</button>
        <button class="btn btn-sm btn-danger" onclick="deleteRec(${o._i})">刪除</button>
      </div></td>
    </tr>`;
    if(isCart&&expanded&&o.items){
      o.items.forEach(it=>{
        html+=`<tr class="item-row ${rowClass}">
          <td colspan="2" style="padding-left:28px"><span class="mono" style="font-size:12px">${it.id}</span> <span style="color:#888;font-size:11px">${it.series||''}</span> <span class="tag ${it.type==='預留'?'tag-reserve':'tag-order'}" style="font-size:10px">${it.type}</span></td>
          <td colspan="2" style="font-size:12px;color:#888">數量：${it.qty}</td>
          <td colspan="2" style="font-size:12px;color:#888">單價：$${it.price}</td>
          <td colspan="4" style="font-size:12px;font-weight:500">小計：$${it.subtotal}</td>
          <td colspan="2"></td>
        </tr>`;
      });
    }
  });
  tbody.innerHTML=html;
  document.getElementById('pageInfoRec').textContent=`第 ${pageRec} / ${tp} 頁，共 ${filteredRec.length} 筆`;
  document.getElementById('prevRec').disabled=pageRec<=1;
  document.getElementById('nextRec').disabled=pageRec>=tp;
}

function updateRecStatus(i,val){
  orders[i].status=val;
  if(val==='已通知'&&!orders[i].notifiedDate){orders[i].notifiedDate=todayISO();orders[i].notifiedInputDate=todayISO();}
  if(val!=='已通知'&&val!=='逾期未領')orders[i].notifiedDate=null;
  saveOrders();filterRec();const tp=Math.ceil(filteredRec.length/PAGE_SIZE);if(pageRec>tp&&tp>0)pageRec=tp;drawRec();updateExpiredBadge();
}
function updateNotifiedDate(i,val){
  orders[i].notifiedInputDate=val;orders[i].notifiedDate=val;
  if(val)orders[i].status='已通知';
  saveOrders();filterRec();drawRec();updateExpiredBadge();
}
function deleteRec(i){orders.splice(i,1);saveOrders();renderRecords();showToast('已刪除紀錄');}
function openRecNoteModal(i){editRecNoteIdx=i;document.getElementById('recNoteInput').value=orders[i].note||'';document.getElementById('recNoteModal').classList.add('show');}
function closeRecNoteModal(){document.getElementById('recNoteModal').classList.remove('show');editRecNoteIdx=null;}
function saveRecNote(){
  if(editRecNoteIdx===null)return;
  orders[editRecNoteIdx].note=document.getElementById('recNoteInput').value.trim();
  saveOrders();closeRecNoteModal();filterRec();drawRec();showToast('備註已更新');
}

/* PART MODAL */
function toggleLoc(type){const chk=document.getElementById('chk'+type);const box=document.getElementById('loc'+type);if(box)box.style.display=chk.checked?'':'none';}
function resetModal(){
  ['MPU','ASIS','ASISB'].forEach(t=>{document.getElementById('chk'+t).checked=false;const b=document.getElementById('loc'+t);if(b)b.style.display='none';});
  document.getElementById('fZoneMPUarea').value='';document.getElementById('fZoneMPUrow').value='';
  document.getElementById('fASISrow').value='';document.getElementById('fASIScol').value='';
}
function openAddModal(){
  editIdx=null;
  document.getElementById('modalTitle').textContent='新增零件';
  document.getElementById('fId').value='';document.getElementById('fId').disabled=false;
  document.getElementById('fSeries').value='';resetModal();
  document.getElementById('fQty').value='';document.getElementById('fLevel').value='high';
  document.getElementById('fNote').value='';document.getElementById('modalErr').textContent='';
  document.getElementById('partModal').classList.add('show');
}
function openEditModal(i){
  editIdx=i;savedPageA=pageA;const p=parts[i];
  document.getElementById('modalTitle').textContent='編輯零件 — '+p.id;
  document.getElementById('fId').value=p.id;document.getElementById('fId').disabled=true;
  document.getElementById('fSeries').value=p.series||'';resetModal();
  (p.locations||[]).forEach(l=>{
    if(l.wall==='MPU'){document.getElementById('chkMPU').checked=true;document.getElementById('locMPU').style.display='';
      const m=(l.zone||'').match(/^(.+?)第(\d+)排$/);
      if(m){document.getElementById('fZoneMPUarea').value=m[1];document.getElementById('fZoneMPUrow').value=m[2];}
      else document.getElementById('fZoneMPUarea').value=l.zone||'';}
    if(l.wall==='ASIS'){document.getElementById('chkASIS').checked=true;document.getElementById('locASIS').style.display='';
      const m=(l.zone||'').match(/^(\d+)排(\d+)$/);
      if(m){document.getElementById('fASISrow').value=m[1];document.getElementById('fASIScol').value=m[2];}}
    if(l.wall==='ASIS背面')document.getElementById('chkASISB').checked=true;
  });
  document.getElementById('fQty').value=p.qty||'';document.getElementById('fLevel').value=p.level||'high';
  document.getElementById('fNote').value=p.note||'';document.getElementById('modalErr').textContent='';
  document.getElementById('partModal').classList.add('show');
}
function closeModal(){document.getElementById('partModal').classList.remove('show');}
function buildMPUzone(){
  const area=document.getElementById('fZoneMPUarea').value;
  const row=document.getElementById('fZoneMPUrow').value.trim();
  if(!area)return'';return row?`${area}第${row}排`:area;
}
function savePart(){
  const id=document.getElementById('fId').value.trim();
  if(!id){document.getElementById('modalErr').textContent='零件編號為必填';return;}
  if(editIdx===null&&parts.find(p=>p.id===id)){document.getElementById('modalErr').textContent='此零件編號已存在';return;}
  const locs=[];
  if(document.getElementById('chkMPU').checked)locs.push({wall:'MPU',zone:buildMPUzone()});
  if(document.getElementById('chkASIS').checked){const r=document.getElementById('fASISrow').value,c=document.getElementById('fASIScol').value;locs.push({wall:'ASIS',zone:(r&&c)?`${r}排${c}`:''});}
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

/* DELETE */
function openDelModal(i){deleteIdx=i;document.getElementById('delPartId').textContent=parts[i].id;document.getElementById('delModal').classList.add('show');}
function closeDelModal(){document.getElementById('delModal').classList.remove('show');deleteIdx=null;}
function confirmDelete(){
  if(deleteIdx===null)return;const id=parts[deleteIdx].id,cur=pageA;
  parts.splice(deleteIdx,1);save();closeDelModal();updateSeriesDropdowns();
  filterA();const tp=Math.ceil(filteredA.length/PAGE_SIZE);pageA=Math.min(cur,tp||1);drawA();renderStatsA();showToast('已刪除 '+id);
}

/* EXPORT */
function exportCSV(){
  const rows=[['零件編號','系列','架位','數量','庫存水位','單價','備註']];
  parts.forEach(p=>{
    const locs=(p.locations||[]).map(l=>l.zone?`${l.wall}-${l.zone}`:l.wall).join(' / ');
    rows.push([p.id,p.series,locs,p.qty,lvLabel(p.level),getPrice(p.id),p.note]);
  });
  const csv='\uFEFF'+rows.map(r=>r.map(c=>`"${c||''}"`).join(',')).join('\n');
  const a=document.createElement('a');a.href='data:text/csv;charset=utf-8,'+encodeURIComponent(csv);
  a.download='MPU零件庫存_'+new Date().toLocaleDateString('zh-TW').replace(/\//g,'-')+'.csv';
  a.click();showToast('已匯出 CSV ✓');
}
function exportRecordsCSV(){
  const rows=[['類型','零件編號','系列','數量','聯絡人','電話','客服處理人','日期','狀態','通知日','到期日','總金額','備註']];
  orders.forEach(o=>{
    if(o.items){
      o.items.forEach(it=>{rows.push([o.type,it.id,it.series||'',it.qty,o.name||'',o.phone||'',o.csc||'',o.date||'',o.status,o.notifiedDate||'',o.notifiedDate?expiryDateStr(o.notifiedDate):'',`$${it.subtotal}`,o.note||'']);});
    } else {
      rows.push([o.type,o.partId||'',o.series||'',o.qty||'',o.name||'',o.phone||'',o.csc||'',o.date||'',o.status,o.notifiedDate||'',o.notifiedDate?expiryDateStr(o.notifiedDate):'',o.totalAmt?`$${o.totalAmt}`:'',o.note||'']);
    }
  });
  const csv='\uFEFF'+rows.map(r=>r.map(c=>`"${c||''}"`).join(',')).join('\n');
  const a=document.createElement('a');a.href='data:text/csv;charset=utf-8,'+encodeURIComponent(csv);
  a.download='MPU訂購預留紀錄_'+new Date().toLocaleDateString('zh-TW').replace(/\//g,'-')+'.csv';
  a.click();showToast('已匯出紀錄 CSV ✓');
}

/* INIT */
load();updateSeriesDropdowns();filteredQ=[...parts];drawQ();renderStatsQ();
</script>
</body>
</html>
