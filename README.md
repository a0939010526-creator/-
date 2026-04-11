<!DOCTYPE html>
<html lang="zh-TW">
<head>
<meta charset="UTF-8">
<title>MPU 零件庫存管理</title>
<script>
let cart = [];
let orders = [];

const parts = [
  { id: "001", name: "零件A" },
  { id: "002", name: "零件B" },
  { id: "003", name: "零件C" }
];

function renderParts() {
  const table = document.getElementById("partsTable");
  table.innerHTML = "";

  parts.forEach(p => {
    table.innerHTML += `
      <tr>
        <td>${p.id}</td>
        <td>${p.name}</td>
        <td>
          <input type="number" min="1" value="1" id="qty-${p.id}" style="width:60px">
          <button onclick="addToCart('${p.id}','${p.name}')">預留 / 預訂</button>
        </td>
      </tr>
    `;
  });
}

function addToCart(id, name) {
  const qty = parseInt(document.getElementById(`qty-${id}`).value);

  const existing = cart.find(i => i.id === id);

  if (existing) {
    existing.qty += qty;
  } else {
    cart.push({ id, name, qty, price: 5 });
  }

  alert("已加入購物車");
  renderCart();
}

function renderCart() {
  const list = document.getElementById("cartList");
  list.innerHTML = "";

  let total = 0;

  cart.forEach(i => {
    const subtotal = i.qty * i.price;
    total += subtotal;

    list.innerHTML += `<li>${i.name} x ${i.qty} = $${subtotal}</li>`;
  });

  document.getElementById("total").innerText = total;
}

function submitOrder() {
  const name = document.getElementById("name").value;
  const phone = document.getElementById("phone").value;
  const handler = document.getElementById("handler").value;

  if (!name || !phone) {
    alert("姓名與電話必填！");
    return;
  }

  const total = cart.reduce((sum, i) => sum + i.qty * i.price, 0);

  const order = {
    id: Date.now(),
    name,
    phone,
    handler,
    items: [...cart],
    total,
    notifiedDate: ""
  };

  orders.push(order);
  cart = [];

  renderCart();
  renderOrders();

  alert("訂單已送出");
}

function renderOrders() {
  const table = document.getElementById("ordersTable");
  table.innerHTML = "";

  orders.forEach(o => {
    const items = o.items.map(i => `${i.name} x ${i.qty}`).join("<br>");

    table.innerHTML += `
      <tr>
        <td>${o.id}</td>
        <td>${o.name}</td>
        <td>${o.phone}</td>
        <td>${items}</td>
        <td>
          <input type="number" value="${o.total}" onchange="updateTotal(${o.id}, this.value)">
        </td>
        <td>
          <input type="date" onchange="updateDate(${o.id}, this.value)">
        </td>
        <td>${o.handler || ""}</td>
      </tr>
    `;
  });
}

function updateTotal(id, value) {
  const order = orders.find(o => o.id === id);
  if (order) order.total = parseInt(value);
}

function updateDate(id, date) {
  const order = orders.find(o => o.id === id);
  if (order) order.notifiedDate = date;
}

window.onload = () => {
  renderParts();
};
</script>
</head>
<body>

<h2>零件列表</h2>
<table border="1">
<thead>
<tr>
  <th>編號</th>
  <th>名稱</th>
  <th></th>
</tr>
</thead>
<tbody id="partsTable"></tbody>
</table>

<h2>購物車</h2>
<ul id="cartList"></ul>
總金額：$<span id="total">0</span>

<h3>填寫訂購單</h3>
姓名：<input id="name" required><br>
電話：<input id="phone" required><br>
客服處理人：<input id="handler"><br>
<button onclick="submitOrder()">送出訂單</button>

<h2>後台訂單</h2>
<table border="1">
<thead>
<tr>
  <th>訂單ID</th>
  <th>姓名</th>
  <th>電話</th>
  <th>商品</th>
  <th>金額</th>
  <th>已通知日期</th>
  <th>客服</th>
</tr>
</thead>
<tbody id="ordersTable"></tbody>
</table>

</body>
</html>
