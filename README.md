<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Doru Shops Admin Demo</title>
<style>
body { font-family: Arial, sans-serif; margin:0; padding:0; background:#e6f4ea; }
header { background:#28a745; color:white; padding:15px; display:flex; justify-content:space-between; align-items:center; position:sticky; top:0; }
header h1 { margin:0; font-size:1.5rem; }

.section { padding:15px; }
.section h2 { margin-bottom:10px; color:#155724; }
.shop-list { display:grid; grid-template-columns:repeat(auto-fill,minmax(150px,1fr)); gap:10px; }
.shop-card { background:#fff; border-radius:5px; text-align:center; box-shadow:0 2px 5px rgba(0,0,0,0.1); display:flex; flex-direction:column; align-items:center; justify-content:center; width:150px; height:150px; flex-shrink:0; }
.shop-card img { width:80px; height:80px; object-fit:cover; border-radius:50%; margin-bottom:5px; }

.filters { display:flex; gap:10px; padding:10px 0; overflow-x:auto; justify-content:center; }
.filters button { padding:8px 12px; border:none; border-radius:5px; background:#28a745; color:white; cursor:pointer; flex-shrink:0; }

.modal { display:none; position:fixed; top:0; left:0; width:100%; height:100%; background:rgba(0,0,0,0.5); justify-content:center; align-items:center; }
.modal-content { background:white; padding:20px; border-radius:8px; width:90%; max-width:500px; max-height:80%; overflow-y:auto; }
.modal-header { display:flex; justify-content:space-between; align-items:center; }
.modal-header h3 { margin:0; color:#155724; }
.close { cursor:pointer; font-weight:bold; font-size:20px; }

.product, .checkout-field, .admin-field { display:flex; justify-content:space-between; margin:10px 0; }
.product button, .checkout-field button, .admin-field button { background:#28a745; color:white; border:none; padding:5px 10px; border-radius:5px; cursor:pointer; }
.checkout-field input, .checkout-field textarea, .admin-field input { flex:1; margin-left:10px; padding:5px; border:1px solid #ccc; border-radius:5px; }

.admin-panel { display:none; background:#d4edda; padding:15px; border:2px solid #28a745; border-radius:5px; margin-bottom:15px; }
.admin-panel h3 { margin-top:0; color:#155724; }
.admin-shop { display:flex; justify-content:space-between; align-items:center; margin:5px 0; gap:5px; }
.admin-shop input[type="text"], .admin-shop input[type="file"] { flex:1; }
</style>
</head>
<body>

<header>
<h1>Doru Shops</h1>
<span class="admin-toggle" onclick="toggleAdmin()">Admin Panel</span>
</header>

<!-- Admin Login Modal -->
<div class="modal" id="loginModal">
<div class="modal-content">
<div class="modal-header">
<h3>Admin Login</h3>
<span class="close" onclick="closeLogin()">&times;</span>
</div>
<div class="admin-field">
<label>Username:</label><input type="text" id="adminUser">
</div>
<div class="admin-field">
<label>Password:</label><input type="password" id="adminPass">
</div>
<button onclick="adminLogin()">Login</button>
</div>
</div>

<!-- Admin Panel -->
<div class="admin-panel" id="adminPanel">
<h3>Admin Panel - Manage Shops & Products</h3>
<div id="adminShopList"></div>
<div class="admin-field">
<input type="text" id="newShopName" placeholder="Add new shop">
<input type="file" id="newShopImage">
<button onclick="addNewShop()">Add Shop</button>
</div>
</div>

<div class="section filters">
<button onclick="filterShops('near')">Near & Fast</button>
<button onclick="filterShops('budget')">Under Budget</button>
<button onclick="filterShops('explore')">Explore More</button>
</div>

<div class="section">
<h2>Shops</h2>
<div class="shop-list" id="shopList"></div>
</div>

<!-- Order Modal -->
<div class="modal" id="orderModal">
<div class="modal-content">
<div class="modal-header">
<h3 id="modalShopName">Shop Name</h3>
<span class="close" onclick="closeModal()">&times;</span>
</div>
<div id="productList"></div>
</div>
</div>

<!-- Checkout Modal -->
<div class="modal" id="checkoutModal">
<div class="modal-content">
<div class="modal-header">
<h3>Checkout</h3>
<span class="close" onclick="closeCheckout()">&times;</span>
</div>
<div id="checkoutForm">
<div class="checkout-field"><label>Email:</label><input type="email" id="userEmail" required></div>
<div class="checkout-field"><label>Phone:</label><input type="text" id="userPhone" required></div>
<div class="checkout-field"><label>Address:</label><textarea id="userAddress" rows="3" required></textarea></div>
<button id="placeOrder">Place Order</button>
</div>
</div>
</div>

<script>
// Demo admin credentials
const ADMIN_USER = "admin";
const ADMIN_PASS = "1234";

// Shop Data
let shopData = [
{name:'Shree Ganesh Store', img:'https://via.placeholder.com/80?text=G', products:[{name:'Chai Patti', img:'https://via.placeholder.com/50?text=Chai'}]},
{name:'Maa Vaishno Traders', img:'https://via.placeholder.com/80?text=M', products:[{name:'Demo 1', img:'https://via.placeholder.com/50?text=1'}]},
{name:'Om Sai Kirana', img:'https://via.placeholder.com/80?text=O', products:[{name:'Demo 2', img:'https://via.placeholder.com/50?text=2'}]}
];

// Render Shops
function renderShops(shops){
  const container = document.getElementById('shopList');
  container.innerHTML='';
  shops.forEach((shop,i)=>{
    const div = document.createElement('div');
    div.className='shop-card';
    div.innerHTML=`<img src="${shop.img}" alt="shop"><p>${shop.name}</p><button onclick="openModal(${i})">Order Now</button>`;
    container.appendChild(div);
  });
  renderAdminPanel();
}

// Filters
const filterCategories = {near:[0,2], budget:[1,2], explore:[0,1,2]};
function filterShops(type){
  const ids = filterCategories[type];
  const filtered = ids.map(i=>shopData[i]).filter(Boolean);
  renderShops(filtered);
}

// Modal logic
let currentShop='';
function openModal(shopId){
  const shop = shopData[shopId];
  currentShop = shop.name;
  document.getElementById('modalShopName').innerText = shop.name;
  const productList = document.getElementById('productList');
  productList.innerHTML='';
  shop.products.forEach((p,pi)=>{
    const div = document.createElement('div');
    div.className='product';
    div.innerHTML=`<img src="${p.img}" width="40" height="40" style="margin-right:5px;"> <span>${p.name}</span> <button onclick="openCheckout('${p.name}')">Add to Cart</button>`;
    productList.appendChild(div);
  });
  document.getElementById('orderModal').style.display='flex';
}
function closeModal(){document.getElementById('orderModal').style.display='none';}

// Checkout
function openCheckout(product){closeModal(); document.getElementById('checkoutModal').style.display='flex';}
function closeCheckout(){document.getElementById('checkoutModal').style.display='none';}
document.getElementById('placeOrder').onclick=function(){
  const email=document.getElementById('userEmail').value;
  const phone=document.getElementById('userPhone').value;
  const address=document.getElementById('userAddress').value;
  if(!email||!phone||!address){alert('Fill all fields'); return;}
  alert(`Order placed for ${currentShop}\nEmail: ${email}\nPhone: ${phone}\nAddress: ${address}`);
  closeCheckout();
}

// Admin Panel
function toggleAdmin(){ document.getElementById('loginModal').style.display='flex'; }
function closeLogin(){document.getElementById('loginModal').style.display='none';}
function adminLogin(){
  const u = document.getElementById('adminUser').value;
  const p = document.getElementById('adminPass').value;
  if(u===ADMIN_USER && p===ADMIN_PASS){
    closeLogin();
    document.getElementById('adminPanel').style.display='flex';
  } else alert('Invalid credentials');
}

function renderAdminPanel(){
  const container = document.getElementById('adminShopList');
  container.innerHTML='';
  shopData.forEach((shop,i)=>{
    const div = document.createElement('div');
    div.className='admin-shop';
    div.innerHTML=`<span>${shop.name}</span>
      <input type="file" onchange="updateShopImage(event,${i})">
      <input type="text" id="newProd${i}" placeholder="New Product Name">
      <input type="file" id="newProdImg${i}">
      <button onclick="addProduct(${i})">Add Product</button>
      <button onclick="deleteShop(${i})">Delete Shop</button>`;
    container.appendChild(div);
  });
}

// Admin actions
function updateShopImage(e,index){
  const file = e.target.files[0];
  if(file){ shopData[index].img = URL.createObjectURL(file); renderShops(shopData);}
}
function addProduct(shopIndex){
  const nameInput = document.getElementById('newProd'+shopIndex);
  const imgInput = document.getElementById('newProdImg'+shopIndex);
  const name = nameInput.value.trim();
  if(!name) return alert('Enter product name');
  let img = 'https://via.placeholder.com/50?text=P';
  if(imgInput.files[0]) img = URL.createObjectURL(imgInput.files[0]);
  shopData[shopIndex].products.push({name,img});
  nameInput.value=''; imgInput.value='';
  renderShops(shopData);
}
function deleteShop(index){ if(confirm('Delete shop?')){ shopData.splice(index,1); renderShops(shopData); } }
function addNewShop(){
  const nameInput = document.getElementById('newShopName');
  const imgInput = document.getElementById('newShopImage');
  const name = nameInput.value.trim();
  if(!name) return alert('Enter shop name');
  let img = 'https://via.placeholder.com/80?text=Shop';
  if(imgInput.files[0]) img = URL.createObjectURL(imgInput.files[0]);
  shopData.push({name,img,products:[]});
  nameInput.value=''; imgInput.value='';
  renderShops(shopData);
}

// Initial render
renderShops(shopData);
</script>

</body>
</html>