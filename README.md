# srm-t-shirt-store-
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <title>SRM T-Shirt Store</title>
  <meta name="viewport" content="width=device-width, initial-scale=1" />
  <style>
    body { font-family: Arial; background: #f9f9f9; margin: 0; }
    header { background: #000; color: #fff; padding: 15px; text-align: center; font-size: 22px; }
    nav { background: #333; padding: 10px; display: flex; justify-content: center; gap: 10px; flex-wrap: wrap; }
    nav button { background: #555; color: #fff; border: none; padding: 10px 15px; border-radius: 5px; cursor: pointer; }
    nav button:hover { background: #777; }
    #logoutBtn { display: none; background: crimson; }

    .page { display: none; padding: 20px; max-width: 900px; margin: auto; background: #fff; border-radius: 8px; }
    .active { display: block; }

    .products { display: flex; flex-wrap: wrap; gap: 15px; justify-content: center; }
    .product { width: 220px; border: 1px solid #ccc; padding: 10px; border-radius: 8px; background: #fff; text-align: center; }
    .product img { width: 100%; height: 140px; object-fit: cover; border-radius: 5px; }
    .btn { margin-top: 10px; padding: 8px; width: 100%; background: #28a745; color: #fff; border: none; border-radius: 5px; cursor: pointer; }

    h2, h3 { text-align: center; }
    .center { text-align: center; margin: 20px 0; }
    #statusMessage { font-weight: bold; text-align: center; margin-top: 10px; color: green; }
    .pay-options { display: flex; flex-direction: column; gap: 10px; }
    .order-box { border: 1px solid #ccc; padding: 10px; margin: 10px 0; border-radius: 6px; background: #f0f0f0; }
  </style>
</head>
<body>

<header>🛍️ SRM T-Shirt Store</header>
<nav>
  <button onclick="showPage('home')">Home</button>
  <button onclick="showPage('cart')">Cart (<span id="cart-count">0</span>)</button>
  <button onclick="showPage('adminLogin')">Admin Login</button>
  <button onclick="showPage('adminOrders')" id="adminOrdersBtn" style="display:none">Orders</button>
  <button id="logoutBtn" onclick="logout()">Logout</button>
</nav>

<div id="home" class="page active">
  <h2>Available T-Shirts</h2>
  <div class="products" id="product-list"></div>
</div>

<div id="cart" class="page">
  <h2>Your Cart</h2>
  <div id="cart-items"></div>
  <p style="text-align:center;">Total: ₹<span id="cart-total">0</span></p>
  <div style="max-width: 400px; margin: auto;">
    <input type="text" id="name" placeholder="Your Name" required style="width:100%; padding:10px; margin:5px;" />
    <input type="tel" id="mobile" placeholder="Mobile Number" required style="width:100%; padding:10px; margin:5px;" />
    <textarea id="address" placeholder="Address" rows="3" required style="width:100%; padding:10px; margin:5px;"></textarea>

    <div class="pay-options">
      <button class="btn" onclick="payWithUPI()">Pay via UPI App</button>
      <button class="btn" onclick="payWithQR()">Pay with QR Code</button>
    </div>

    <div class="center">
      <img src="" alt="UPI QR Code" id="qrCode" style="display:none">
      <p><a href="#" id="upiLink" target="_blank" style="display:none">Click here to open in UPI App</a></p>
      <p id="countdown" style="font-weight:bold;"></p>
      <div id="statusMessage"></div>
    </div>
  </div>
</div>

<div id="adminLogin" class="page">
  <h2>Admin Login</h2>
  <div style="max-width: 300px; margin: auto;">
    <input type="text" id="adminUser" placeholder="Admin ID" style="width:100%; padding:10px; margin:5px;" />
    <input type="password" id="adminPass" placeholder="Password" style="width:100%; padding:10px; margin:5px;" />
    <button class="btn" onclick="adminLogin()">Login</button>
  </div>
</div>

<div id="adminOrders" class="page">
  <h2>Order History</h2>
  <div id="orderList"></div>
</div>

<script>
let session = '';
let products = [
  { name: "Casual White T-Shirt", price: 199, desc: "Plain white cotton T-shirt", img: "https://via.placeholder.com/220x140.png?text=White+T-Shirt" },
  { name: "Black Graphic Tee", price: 299, desc: "Stylish black T-shirt with front graphic print", img: "https://via.placeholder.com/220x140.png?text=Black+Graphic+Tee" },
  { name: "Navy Blue Polo", price: 249, desc: "Solid navy polo shirt for men", img: "https://via.placeholder.com/220x140.png?text=Navy+Polo" },
  { name: "Black Hoodie T-Shirt", price: 1, desc: "Black half sleeve hoodie with mountain print", img: "https://i.ibb.co/XyY3yFf/printed-tshirt.jpg" }
];

let cart = [];

function showPage(id) {
  document.querySelectorAll('.page').forEach(p => p.classList.remove('active'));
  document.getElementById(id).classList.add('active');
  if (id === 'adminOrders') loadOrders();
}

function logout() {
  session = '';
  cart = [];
  document.getElementById('logoutBtn').style.display = "none";
  document.getElementById('adminOrdersBtn').style.display = "none";
  alert("Logged out.");
  updateCart();
  showPage('home');
}

function displayProducts() {
  const list = document.getElementById('product-list');
  list.innerHTML = '';
  products.forEach((p, i) => {
    list.innerHTML += `
      <div class="product">
        <img src="${p.img}" alt="${p.name}" />
        <h4>${p.name}</h4>
        <p>₹${p.price}</p>
        <small>${p.desc}</small>
        <button class="btn" onclick="addToCart(${i})">Add to Cart</button>
      </div>`;
  });
}

function addToCart(i) {
  cart.push(products[i]);
  updateCart();
  alert("Added to cart");
}

function updateCart() {
  document.getElementById('cart-count').textContent = cart.length;
  const items = document.getElementById('cart-items');
  items.innerHTML = '';
  let total = 0;
  cart.forEach((item, i) => {
    total += item.price;
    items.innerHTML += `<p>🛒 ${item.name} - ₹${item.price}</p>`;
  });
  document.getElementById('cart-total').textContent = total;
}

function getUserDetails() {
  const name = document.getElementById('name').value.trim();
  const mobile = document.getElementById('mobile').value.trim();
  const address = document.getElementById('address').value.trim();
  if (!name || !mobile || !address) {
    alert("Please fill all details");
    return null;
  }
  return { name, mobile, address };
}

function saveOrder(user, total, cart) {
  let orders = JSON.parse(localStorage.getItem("orders") || "[]");
  orders.push({ ...user, total, items: cart, date: new Date().toLocaleString() });
  localStorage.setItem("orders", JSON.stringify(orders));
}

function payWithUPI() {
  const user = getUserDetails();
  if (!user) return;
  const total = document.getElementById('cart-total').textContent;
  const upiLink = `upi://pay?pa=9334734456@ybl&pn=SRM%20PAYMENT&am=${total}`;
  document.getElementById('upiLink').href = upiLink;
  document.getElementById('upiLink').style.display = 'block';
  document.getElementById('qrCode').style.display = 'none';
  document.getElementById('statusMessage').textContent = "Redirecting to UPI App...";
  window.location.href = upiLink;
  setTimeout(() => {
    document.getElementById('statusMessage').textContent = "✅ Payment Success! Order Placed.";
    saveOrder(user, total, cart);
    cart = [];
    updateCart();
    showPage('home');
  }, 10000);
}

function payWithQR() {
  const user = getUserDetails();
  if (!user) return;
  const total = document.getElementById('cart-total').textContent;
  const upiLink = `upi://pay?pa=9334734456@ybl&pn=SRM%20PAYMENT&am=${total}`;
  const qrUrl = `https://api.qrserver.com/v1/create-qr-code/?size=200x200&data=${encodeURIComponent(upiLink)}`;
  document.getElementById('qrCode').src = qrUrl;
  document.getElementById('qrCode').style.display = 'block';
  document.getElementById('upiLink').style.display = 'none';
  document.getElementById('statusMessage').textContent = "Please scan and pay within 30 seconds...";
  let timeLeft = 30;
  const countdown = document.getElementById('countdown');
  countdown.textContent = `⏳ ${timeLeft}s left to complete payment...`;
  const timer = setInterval(() => {
    timeLeft--;
    countdown.textContent = `⏳ ${timeLeft}s left to complete payment...`;
    if (timeLeft <= 0) {
      clearInterval(timer);
      document.getElementById('statusMessage').textContent = "✅ Payment Success! Order Placed.";
      saveOrder(user, total, cart);
      cart = [];
      updateCart();
      countdown.textContent = "";
      showPage('home');
    }
  }, 1000);
}

function loadOrders() {
  const orderList = document.getElementById("orderList");
  const orders = JSON.parse(localStorage.getItem("orders") || "[]");
  orderList.innerHTML = orders.map(order => `
    <div class='order-box'>
      <b>Name:</b> ${order.name}<br>
      <b>Mobile:</b> ${order.mobile}<br>
      <b>Address:</b> ${order.address}<br>
      <b>Total:</b> ₹${order.total}<br>
      <b>Items:</b> ${order.items.map(i => i.name).join(", ")}<br>
      <b>Date:</b> ${order.date}
    </div>
  `).join('');
}

function adminLogin() {
  const user = document.getElementById('adminUser').value.trim();
  const pass = document.getElementById('adminPass').value.trim();
  if (user === "Admin" && pass === "123") {
    session = 'admin';
    document.getElementById('logoutBtn').style.display = "inline-block";
    document.getElementById('adminOrdersBtn').style.display = "inline-block";
    alert("Admin login successful!");
    showPage('home');
  } else {
    alert("Invalid credentials!");
  }
}

displayProducts();
</script>

</body>
</html>
