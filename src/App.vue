<script setup>
import { ref, computed } from 'vue'

// 购物车商品数据
const cartItems = ref([
  {
    id: 1,
    name: 'iPhone 15 Pro Max 256GB 钛金属原色',
    price: 9999,
    quantity: 1,
    image: 'https://via.placeholder.com/100x100?text=iPhone',
    checked: true
  },
  {
    id: 2,
    name: '小米13 Ultra 12GB+256GB 黑色',
    price: 5999,
    quantity: 1,
    image: 'https://via.placeholder.com/100x100?text=Mi',
    checked: true
  },
  {
    id: 3,
    name: 'MacBook Pro 14英寸 M3 Pro芯片',
    price: 14999,
    quantity: 2,
    image: 'https://via.placeholder.com/100x100?text=MacBook',
    checked: false
  },
  {
    id: 4,
    name: 'AirPods Pro 第二代',
    price: 1899,
    quantity: 1,
    image: 'https://via.placeholder.com/100x100?text=AirPods',
    checked: false
  },
  {
    id: 5,
    name: '华为Mate 60 Pro 12GB+512GB 雅川青',
    price: 6999,
    quantity: 1,
    image: 'https://via.placeholder.com/100x100?text=Huawei',
    checked: false
  }
])

// 全选状态
const selectAll = ref(true)

// 修改商品数量
const updateQuantity = (item, delta) => {
  const newQuantity = item.quantity + delta
  if (newQuantity >= 1) {
    item.quantity = newQuantity
  }
}

// 切换单个商品选中状态
const toggleItemCheck = (item) => {
  item.checked = !item.checked
  // 更新全选状态
  selectAll.value = cartItems.value.every(item => item.checked)
}

// 切换全选状态
const toggleSelectAll = () => {
  selectAll.value = !selectAll.value
  cartItems.value.forEach(item => {
    item.checked = selectAll.value
  })
}

// 删除商品
const deleteItem = (id) => {
  const index = cartItems.value.findIndex(item => item.id === id)
  if (index > -1) {
    cartItems.value.splice(index, 1)
    // 更新全选状态
    if (cartItems.value.length === 0) {
      selectAll.value = false
    } else {
      selectAll.value = cartItems.value.every(item => item.checked)
    }
  }
}

// 清空购物车
const clearCart = () => {
  if (confirm('确定要清空购物车吗？')) {
    cartItems.value = []
    selectAll.value = false
  }
}

// 选中的商品数量
const selectedCount = computed(() => {
  return cartItems.value.filter(item => item.checked).length
})

// 选中的商品总数量
const selectedTotalCount = computed(() => {
  return cartItems.value
    .filter(item => item.checked)
    .reduce((total, item) => total + item.quantity, 0)
})

// 选中的商品总价
const totalPrice = computed(() => {
  return cartItems.value
    .filter(item => item.checked)
    .reduce((total, item) => total + item.price * item.quantity, 0)
})

// 结算
const checkout = () => {
  if (selectedCount.value === 0) {
    alert('请先选择要结算的商品')
    return
  }
  alert(`结算成功！共 ${selectedTotalCount.value} 件商品，总价 ¥${totalPrice.value.toLocaleString()}`)
}
</script>

<template>
  <div class="cart-container">
    <!-- 头部 -->
    <header class="header">
      <h1>🛒 购物车</h1>
      <p class="cart-info">共 {{ cartItems.length }} 件商品，已选 {{ selectedCount }} 件</p>
    </header>

    <!-- 购物车商品列表 -->
    <div class="cart-content">
      <div v-if="cartItems.length === 0" class="empty-cart">
        <div class="empty-icon">🛒</div>
        <p class="empty-text">购物车是空的</p>
        <button class="shop-btn" @click="() => cartItems.length = 0">去逛逛</button>
      </div>

      <div v-else class="cart-list">
        <!-- 列表头部 -->
        <div class="list-header">
          <label class="checkbox-label">
            <input type="checkbox" v-model="selectAll" @change="toggleSelectAll" />
            <span>全选</span>
          </label>
          <span class="col-name">商品信息</span>
          <span class="col-price">单价</span>
          <span class="col-quantity">数量</span>
          <span class="col-total">小计</span>
          <span class="col-action">操作</span>
        </div>

        <!-- 商品列表 -->
        <div class="list-body">
          <div v-for="item in cartItems" :key="item.id" class="cart-item">
            <label class="checkbox-label">
              <input type="checkbox" v-model="item.checked" @change="toggleItemCheck(item)" />
            </label>
            <div class="item-info">
              <img :src="item.image" :alt="item.name" class="item-image" />
              <div class="item-details">
                <h3 class="item-name">{{ item.name }}</h3>
                <span class="item-sku">规格：默认</span>
              </div>
            </div>
            <span class="item-price">¥{{ item.price.toLocaleString() }}</span>
            <div class="quantity-control">
              <button class="qty-btn" @click="updateQuantity(item, -1)" :disabled="item.quantity <= 1">-</button>
              <span class="qty-number">{{ item.quantity }}</span>
              <button class="qty-btn" @click="updateQuantity(item, 1)">+</button>
            </div>
            <span class="item-total">¥{{ (item.price * item.quantity).toLocaleString() }}</span>
            <button class="delete-btn" @click="deleteItem(item.id)">删除</button>
          </div>
        </div>
      </div>
    </div>

    <!-- 底部结算栏 -->
    <footer class="cart-footer" v-if="cartItems.length > 0">
      <div class="footer-left">
        <label class="checkbox-label">
          <input type="checkbox" v-model="selectAll" @change="toggleSelectAll" />
          <span>全选</span>
        </label>
        <button class="clear-btn" @click="clearCart">清空购物车</button>
      </div>
      <div class="footer-right">
        <div class="total-info">
          <span class="total-text">已选商品 <strong>{{ selectedCount }}</strong> 件</span>
          <span class="total-price">
            合计：<strong class="price-highlight">¥{{ totalPrice.toLocaleString() }}</strong>
          </span>
        </div>
        <button class="checkout-btn" @click="checkout" :disabled="selectedCount === 0">
          结算 ({{ selectedTotalCount }})
        </button>
      </div>
    </footer>
  </div>
</template>

<style scoped>
* {
  box-sizing: border-box;
  margin: 0;
  padding: 0;
}

.cart-container {
  max-width: 1200px;
  margin: 0 auto;
  padding: 20px;
  background: #f5f5f5;
  min-height: 100vh;
}

.header {
  background: white;
  padding: 20px 30px;
  border-radius: 8px;
  margin-bottom: 20px;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.05);
}

.header h1 {
  font-size: 24px;
  color: #333;
  margin-bottom: 8px;
}

.cart-info {
  color: #999;
  font-size: 14px;
}

.cart-content {
  background: white;
  border-radius: 8px;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.05);
  overflow: hidden;
}

.empty-cart {
  padding: 80px 20px;
  text-align: center;
}

.empty-icon {
  font-size: 80px;
  margin-bottom: 20px;
}

.empty-text {
  font-size: 18px;
  color: #999;
  margin-bottom: 20px;
}

.shop-btn {
  padding: 12px 30px;
  background: linear-gradient(135deg, #ff5000 0%, #ff6b00 100%);
  color: white;
  border: none;
  border-radius: 20px;
  font-size: 16px;
  cursor: pointer;
  transition: all 0.3s;
}

.shop-btn:hover {
  transform: translateY(-2px);
  box-shadow: 0 4px 12px rgba(255, 80, 0, 0.3);
}

.list-header {
  display: grid;
  grid-template-columns: 40px 420px 120px 150px 120px 80px;
  gap: 20px;
  padding: 15px 30px;
  background: #fafafa;
  border-bottom: 1px solid #eee;
  color: #666;
  font-size: 14px;
  font-weight: 500;
  align-items: center;
}

.list-body {
  max-height: calc(100vh - 350px);
  overflow-y: auto;
}

.cart-item {
  display: grid;
  grid-template-columns: 40px 420px 120px 150px 120px 80px;
  gap: 20px;
  padding: 20px 30px;
  border-bottom: 1px solid #eee;
  align-items: center;
  transition: background 0.2s;
}

.cart-item:hover {
  background: #fafafa;
}

.checkbox-label {
  display: flex;
  align-items: center;
  gap: 8px;
  cursor: pointer;
}

.checkbox-label input[type="checkbox"] {
  width: 18px;
  height: 18px;
  cursor: pointer;
}

.item-info {
  display: flex;
  gap: 15px;
  align-items: center;
}

.item-image {
  width: 80px;
  height: 80px;
  object-fit: cover;
  border-radius: 4px;
  border: 1px solid #eee;
}

.item-details {
  flex: 1;
}

.item-name {
  font-size: 14px;
  color: #333;
  margin-bottom: 8px;
  line-height: 1.4;
  display: -webkit-box;
  -webkit-line-clamp: 2;
  -webkit-box-orient: vertical;
  overflow: hidden;
}

.item-sku {
  font-size: 12px;
  color: #999;
}

.item-price {
  color: #666;
  font-size: 14px;
}

.quantity-control {
  display: flex;
  align-items: center;
  border: 1px solid #ddd;
  border-radius: 4px;
  overflow: hidden;
}

.qty-btn {
  width: 30px;
  height: 28px;
  border: none;
  background: #f5f5f5;
  cursor: pointer;
  font-size: 16px;
  transition: background 0.2s;
}

.qty-btn:hover:not(:disabled) {
  background: #e5e5e5;
}

.qty-btn:disabled {
  color: #ccc;
  cursor: not-allowed;
}

.qty-number {
  width: 40px;
  text-align: center;
  font-size: 14px;
  color: #333;
}

.item-total {
  color: #ff5000;
  font-weight: 600;
  font-size: 16px;
}

.delete-btn {
  padding: 6px 16px;
  background: transparent;
  border: 1px solid #ddd;
  color: #666;
  border-radius: 4px;
  cursor: pointer;
  font-size: 13px;
  transition: all 0.2s;
}

.delete-btn:hover {
  border-color: #ff5000;
  color: #ff5000;
}

.cart-footer {
  position: fixed;
  bottom: 0;
  left: 0;
  right: 0;
  background: white;
  padding: 15px 20px;
  box-shadow: 0 -2px 10px rgba(0, 0, 0, 0.1);
  display: flex;
  justify-content: center;
  max-width: 1200px;
  margin: 0 auto;
}

.footer-left,
.footer-right {
  display: flex;
  align-items: center;
  gap: 20px;
}

.footer-right {
  margin-left: auto;
}

.clear-btn {
  padding: 8px 20px;
  background: transparent;
  border: 1px solid #ddd;
  color: #666;
  border-radius: 4px;
  cursor: pointer;
  font-size: 14px;
  transition: all 0.2s;
}

.clear-btn:hover {
  border-color: #ff5000;
  color: #ff5000;
}

.total-info {
  display: flex;
  flex-direction: column;
  align-items: flex-end;
  gap: 4px;
}

.total-text {
  color: #666;
  font-size: 14px;
}

.total-text strong {
  color: #ff5000;
}

.total-price {
  font-size: 14px;
  color: #333;
}

.price-highlight {
  font-size: 22px;
  color: #ff5000;
  margin-left: 8px;
}

.checkout-btn {
  padding: 12px 40px;
  background: linear-gradient(135deg, #ff5000 0%, #ff6b00 100%);
  color: white;
  border: none;
  border-radius: 20px;
  font-size: 16px;
  font-weight: 600;
  cursor: pointer;
  transition: all 0.3s;
}

.checkout-btn:hover:not(:disabled) {
  transform: translateY(-2px);
  box-shadow: 0 4px 12px rgba(255, 80, 0, 0.3);
}

.checkout-btn:disabled {
  background: #ccc;
  cursor: not-allowed;
}

@media (max-width: 1200px) {
  .list-header,
  .cart-item {
    grid-template-columns: 40px 1fr 100px 120px 100px 60px;
    gap: 10px;
    padding: 15px 20px;
  }
  
  .item-info {
    grid-column: 2;
  }
}

@media (max-width: 768px) {
  .cart-container {
    padding: 10px;
  }
  
  .list-header {
    display: none;
  }
  
  .cart-item {
    grid-template-columns: 1fr;
    gap: 15px;
    padding: 15px;
  }
  
  .checkbox-label {
    justify-content: flex-start;
  }
  
  .item-info {
    display: flex;
  }
  
  .cart-footer {
    flex-direction: column;
    padding: 15px;
  }
  
  .footer-left,
  .footer-right {
    width: 100%;
    justify-content: space-between;
  }
  
  .total-info {
    align-items: flex-start;
  }
}
</style>
