<script setup>
import { ref, computed, watch } from 'vue'

// 购物车商品数据
const cartItems = ref([
  {
    id: 1,
    shopName: 'Apple官方旗舰店',
    shopId: 101,
    items: [
      {
        id: 1,
        name: 'iPhone 15 Pro Max 256GB 钛金属原色',
        price: 9999,
        originalPrice: 10999,
        quantity: 1,
        image: 'https://doc-fd.zol-img.com.cn/t_s2000x2000/g7/M00/0A/00/ChMkK2fBV4GIeDa0AAAxLe1DWM4AApdjAHZ4CcAADFF356.jpg',
        checked: true,
        promotion: '满5000减300',
        stock: 999
      }
    ]
  },
  {
    id: 2,
    shopName: '小米官方旗舰店',
    shopId: 102,
    items: [
      {
        id: 2,
        name: '小米13 Ultra 12GB+256GB 黑色 徕卡光学',
        price: 5999,
        originalPrice: 6499,
        quantity: 1,
        image: 'https://d.ifengimg.com/q70/p2.ifengimg.com/a/2017_11/1ff9d6013e82b83_size260_w900_h599.jpg',
        checked: true,
        promotion: '限时直降',
        stock: 50
      },
      {
        id: 3,
        name: '小米平板6 Pro 12.4英寸 8GB+256GB',
        price: 2499,
        originalPrice: 2699,
        quantity: 2,
        image: 'https://d.ifengimg.com/q70/p2.ifengimg.com/a/2017_11/1ff9d6013e82b83_size260_w900_h599.jpg',
        checked: false,
        promotion: '满3000减200',
        stock: 100
      }
    ]
  },
  {
    id: 3,
    shopName: '华为官方旗舰店',
    shopId: 103,
    items: [
      {
        id: 4,
        name: '华为Mate 60 Pro 12GB+512GB 雅川青',
        price: 6999,
        originalPrice: 7499,
        quantity: 1,
        image: 'https://img95.699pic.com/photo/50467/0328.jpg_wh300.jpg!/fh/300/quality/90',
        checked: false,
        promotion: '新品上市',
        stock: 0,
        tags: ['缺货']
      }
    ]
  }
])

// 全选状态
const selectAll = ref(false)

// 修改商品数量
const updateQuantity = (item, delta) => {
  const newQuantity = item.quantity + delta
  if (newQuantity >= 1 && newQuantity <= 99) {
    item.quantity = newQuantity
  }
}

// 商品失焦时校验数量
const handleQuantityBlur = (item) => {
  if (item.quantity < 1) item.quantity = 1
  if (item.quantity > 99) item.quantity = 99
}

// 切换单个商品选中状态
const toggleItemCheck = (shop, item) => {
  item.checked = !item.checked
  updateShopCheckState(shop)
  updateSelectAllState()
}

// 切换店铺全选状态
const toggleShopCheck = (shop) => {
  const newChecked = !isShopAllChecked(shop)
  shop.items.forEach(item => {
    item.checked = newChecked
  })
  updateSelectAllState()
}

// 判断店铺是否全选
const isShopAllChecked = (shop) => {
  return shop.items.length > 0 && shop.items.every(item => item.checked)
}

// 判断店铺是否部分选中
const isShopPartChecked = (shop) => {
  const checkedItems = shop.items.filter(item => item.checked)
  return checkedItems.length > 0 && checkedItems.length < shop.items.length
}

// 更新店铺选中状态
const updateShopCheckState = (shop) => {
  // 实时计算店铺选中状态
}

// 更新全局全选状态
const updateSelectAllState = () => {
  selectAll.value = cartItems.value.every(shop => 
    shop.items.length > 0 && shop.items.every(item => item.checked)
  )
}

// 切换全局全选
const toggleSelectAll = () => {
  selectAll.value = !selectAll.value
  cartItems.value.forEach(shop => {
    shop.items.forEach(item => {
      item.checked = selectAll.value
    })
  })
}

// 删除商品
const deleteItem = (shop, itemId) => {
  const index = shop.items.findIndex(item => item.id === itemId)
  if (index > -1) {
    shop.items.splice(index, 1)
    // 如果店铺没有商品了，移除店铺
    if (shop.items.length === 0) {
      const shopIndex = cartItems.value.findIndex(s => s.id === shop.id)
      if (shopIndex > -1) {
        cartItems.value.splice(shopIndex, 1)
      }
    }
    updateSelectAllState()
  }
}

// 移入收藏夹
const addToFavorite = (shop, item) => {
  if (confirm(`确定要将"${item.name}"移入收藏夹吗？`)) {
    deleteItem(shop, item.id)
    alert('已移入收藏夹')
  }
}

// 清空购物车
const clearCart = () => {
  const selectedItems = getAllSelectedItems()
  if (selectedItems.length === 0) {
    alert('请先选择要清空的商品')
    return
  }
  if (confirm(`确定要清空选中的 ${selectedItems.length} 件商品吗？`)) {
    cartItems.value = []
    selectAll.value = false
  }
}

// 获取所有选中的商品
const getAllSelectedItems = () => {
  const selected = []
  cartItems.value.forEach(shop => {
    shop.items.forEach(item => {
      if (item.checked) {
        selected.push(item)
      }
    })
  })
  return selected
}

// 选中的商品数量
const selectedCount = computed(() => {
  return getAllSelectedItems().length
})

// 选中的商品总数量
const selectedTotalCount = computed(() => {
  return getAllSelectedItems().reduce((total, item) => total + item.quantity, 0)
})

// 选中的商品总价
const totalPrice = computed(() => {
  return getAllSelectedItems().reduce((total, item) => total + item.price * item.quantity, 0)
})

// 优惠金额
const discountAmount = computed(() => {
  return getAllSelectedItems().reduce((total, item) => {
    return total + (item.originalPrice - item.price) * item.quantity
  }, 0)
})

// 结算
const checkout = () => {
  const selected = getAllSelectedItems()
  if (selected.length === 0) {
    alert('请先选择要结算的商品')
    return
  }
  
  // 检查是否有缺货商品
  const outOfStockItems = selected.filter(item => item.stock === 0)
  if (outOfStockItems.length > 0) {
    alert('选中的商品中有缺货商品，请先取消选择')
    return
  }
  
  alert(`结算成功！\n共 ${selectedTotalCount.value} 件商品\n商品金额：¥${(totalPrice.value + discountAmount.value).toLocaleString()}\n优惠金额：¥${discountAmount.value.toLocaleString()}\n实付金额：¥${totalPrice.value.toLocaleString()}`)
}

// 编辑商品规格
const editItem = (item) => {
  alert('编辑商品规格功能')
}

// 查找相似商品
const findSimilar = (item) => {
  alert(`查找与"${item.name}"相似的商品`)
}

// 监听商品选中状态变化，自动更新全选
watch(() => cartItems.value, () => {
  updateSelectAllState()
}, { deep: true })
</script>

<template>
  <div class="cart-container">
    <!-- 顶部导航 -->
    <div class="top-nav">
      <div class="nav-content">
        <div class="nav-left">
          <h1 class="logo">淘宝购物车</h1>
          <span class="nav-divider">|</span>
          <span class="welcome-text">Hi，欢迎来到淘宝</span>
        </div>
        <div class="nav-right">
          <a href="#" class="nav-link">我的淘宝</a>
          <a href="#" class="nav-link">购物车 <span class="badge">{{ totalCount }}</span></a>
          <a href="#" class="nav-link">收藏夹</a>
          <a href="#" class="nav-link">商品分类</a>
        </div>
      </div>
    </div>

    <!-- 搜索栏 -->
    <div class="search-bar">
      <div class="search-box">
        <input type="text" placeholder="搜索购物车中的商品" class="search-input" />
        <button class="search-btn">搜索</button>
      </div>
    </div>

    <!-- 购物车主内容 -->
    <div class="cart-main">
      <!-- 购物车为空 -->
      <div v-if="totalCount === 0" class="empty-cart">
        <div class="empty-image">🛒</div>
        <h2 class="empty-title">您的购物车还是空的</h2>
        <p class="empty-desc">快去挑选喜欢的商品吧</p>
        <button class="shop-btn">去逛逛</button>
        <button class="view-history-btn">查看购物记录</button>
      </div>

      <!-- 购物车列表 -->
      <div v-else class="cart-content">
        <!-- 购物车头部 -->
        <div class="cart-header">
          <label class="checkbox-label">
            <input type="checkbox" v-model="selectAll" @change="toggleSelectAll" />
            <span>全选</span>
          </label>
          <span class="header-item header-info">商品信息</span>
          <span class="header-item header-price">单价</span>
          <span class="header-item header-quantity">数量</span>
          <span class="header-item header-total">金额</span>
          <span class="header-item header-action">操作</span>
        </div>

        <!-- 店铺列表 -->
        <div v-for="shop in cartItems" :key="shop.id" class="shop-group">
          <!-- 店铺头部 -->
          <div class="shop-header">
            <label class="checkbox-label">
              <input 
                type="checkbox" 
                :checked="isShopAllChecked(shop)"
                :indeterminate="isShopPartChecked(shop)"
                @change="toggleShopCheck(shop)"
              />
              <svg class="shop-icon" viewBox="0 0 1024 1024">
                <path d="M128 192h768v64H128zM128 448h768v64H128zM128 704h768v64H128z"/>
              </svg>
              <span class="shop-name">{{ shop.shopName }}</span>
              <span class="shop-link">进店逛逛 ></span>
            </label>
          </div>

          <!-- 商品列表 -->
          <div v-for="item in shop.items" :key="item.id" 
               class="cart-item"
               :class="{ 'out-of-stock': item.stock === 0 }">
            <label class="checkbox-label">
              <input 
                type="checkbox" 
                v-model="item.checked" 
                @change="toggleItemCheck(shop, item)"
                :disabled="item.stock === 0"
              />
            </label>
            
            <div class="item-image-box">
              <img :src="item.image" :alt="item.name" class="item-image" />
              <div v-if="item.tags && item.tags.length" class="item-tags">
                <span v-for="tag in item.tags" :key="tag" class="tag">{{ tag }}</span>
              </div>
            </div>

            <div class="item-info">
              <a href="#" class="item-name" @click.prevent>{{ item.name }}</a>
              <div class="item-sku">
                <span>规格：默认规格</span>
                <a href="#" class="edit-sku" @click.prevent="editItem(item)">修改</a>
              </div>
              <div v-if="item.promotion" class="promotion-tag">
                <span class="tag-icon">🎁</span>
                {{ item.promotion }}
              </div>
              <div class="item-actions">
                <a href="#" class="action-link" @click.prevent="findSimilar(item)">找相似</a>
                <a href="#" class="action-link" @click.prevent="addToFavorite(shop, item)">移入收藏夹</a>
              </div>
            </div>

            <div class="item-price">
              <span class="price-symbol">¥</span>
              <span class="price-value">{{ item.price.toLocaleString() }}</span>
              <div v-if="item.originalPrice > item.price" class="original-price">
                ¥{{ item.originalPrice.toLocaleString() }}
              </div>
            </div>

            <div class="item-quantity">
              <div class="quantity-control" :class="{ 'disabled': item.stock === 0 }">
                <button class="qty-btn minus" 
                        @click="updateQuantity(item, -1)"
                        :disabled="item.quantity <= 1 || item.stock === 0">-</button>
                <input type="number" 
                       v-model.number="item.quantity"
                       @blur="handleQuantityBlur(item)"
                       @keyup.enter="handleQuantityBlur(item)"
                       class="qty-input"
                       :disabled="item.stock === 0"
                       min="1"
                       max="99" />
                <button class="qty-btn plus"
                        @click="updateQuantity(item, 1)"
                        :disabled="item.stock === 0 || item.quantity >= 99">+</button>
              </div>
              <span v-if="item.stock > 0" class="stock-info">库存{{ item.stock }}件</span>
              <span v-else class="out-of-stock-text">暂时缺货</span>
            </div>

            <div class="item-total">
              <span class="price-symbol">¥</span>
              <span class="price-value">{{ (item.price * item.quantity).toLocaleString() }}</span>
              <div v-if="item.originalPrice > item.price" class="save-price">
                已省¥{{ ((item.originalPrice - item.price) * item.quantity).toLocaleString() }}
              </div>
            </div>

            <div class="item-action">
              <button class="action-btn favorite" @click="addToFavorite(shop, item)">
                ♥ 收藏
              </button>
              <button class="action-btn delete" @click="deleteItem(shop, item.id)">
                删除
              </button>
            </div>
          </div>
        </div>
      </div>
    </div>

    <!-- 底部结算栏 -->
    <footer class="cart-footer" v-if="totalCount > 0">
      <div class="footer-content">
        <div class="footer-left">
          <label class="checkbox-label">
            <input type="checkbox" v-model="selectAll" @change="toggleSelectAll" />
            <span>全选</span>
          </label>
          <button class="footer-btn" @click="clearCart">清空购物车</button>
          <button class="footer-btn share-btn">分享</button>
        </div>
        
        <div class="footer-right">
          <div class="footer-info">
            <div class="selected-count">
              已选商品 <strong>{{ selectedCount }}</strong> 件
            </div>
            <div v-if="discountAmount > 0" class="discount-info">
              <span>商品金额：¥{{ (totalPrice + discountAmount).toLocaleString() }}</span>
              <span class="discount-amount">优惠：-¥{{ discountAmount.toLocaleString() }}</span>
            </div>
            <div class="total-price">
              合计：<strong class="price-highlight">¥{{ totalPrice.toLocaleString() }}</strong>
            </div>
          </div>
          <button class="checkout-btn" @click="checkout" :disabled="selectedCount === 0">
            结算
          </button>
        </div>
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
  min-height: 100vh;
  background: #f5f5f5;
  padding-bottom: 80px;
}

/* 顶部导航 */
.top-nav {
  background: #f5f5f5;
  border-bottom: 1px solid #e8e8e8;
}

.nav-content {
  max-width: 1200px;
  margin: 0 auto;
  padding: 8px 20px;
  display: flex;
  justify-content: space-between;
  align-items: center;
  font-size: 12px;
  color: #666;
}

.nav-left {
  display: flex;
  align-items: center;
  gap: 15px;
}

.logo {
  font-size: 18px;
  font-weight: bold;
  color: #ff5000;
  margin: 0;
}

.nav-divider {
  color: #ddd;
}

.welcome-text {
  color: #999;
}

.nav-right {
  display: flex;
  gap: 20px;
}

.nav-link {
  color: #666;
  text-decoration: none;
  transition: color 0.2s;
}

.nav-link:hover {
  color: #ff5000;
}

.badge {
  background: #ff5000;
  color: white;
  padding: 2px 6px;
  border-radius: 10px;
  margin-left: 4px;
}

/* 搜索栏 */
.search-bar {
  background: linear-gradient(135deg, #ff5000 0%, #ff6b00 100%);
  padding: 30px 20px;
}

.search-box {
  max-width: 600px;
  margin: 0 auto;
  display: flex;
  border-radius: 20px;
  overflow: hidden;
  box-shadow: 0 4px 12px rgba(255, 80, 0, 0.3);
}

.search-input {
  flex: 1;
  padding: 12px 20px;
  border: none;
  outline: none;
  font-size: 14px;
}

.search-btn {
  padding: 12px 30px;
  background: #ff5000;
  color: white;
  border: none;
  font-size: 14px;
  cursor: pointer;
  transition: background 0.2s;
}

.search-btn:hover {
  background: #e04800;
}

/* 购物车主内容 */
.cart-main {
  max-width: 1200px;
  margin: 20px auto;
  padding: 0 20px;
}

/* 空购物车 */
.empty-cart {
  background: white;
  padding: 80px 20px;
  text-align: center;
  border-radius: 8px;
}

.empty-image {
  font-size: 120px;
  margin-bottom: 20px;
  opacity: 0.3;
}

.empty-title {
  font-size: 24px;
  color: #333;
  margin-bottom: 10px;
}

.empty-desc {
  color: #999;
  margin-bottom: 30px;
}

.shop-btn {
  padding: 12px 40px;
  background: linear-gradient(135deg, #ff5000 0%, #ff6b00 100%);
  color: white;
  border: none;
  border-radius: 20px;
  font-size: 16px;
  cursor: pointer;
  margin-right: 15px;
  transition: all 0.3s;
}

.shop-btn:hover {
  transform: translateY(-2px);
  box-shadow: 0 4px 12px rgba(255, 80, 0, 0.3);
}

.view-history-btn {
  padding: 12px 40px;
  background: white;
  color: #666;
  border: 1px solid #ddd;
  border-radius: 20px;
  font-size: 16px;
  cursor: pointer;
  transition: all 0.3s;
}

.view-history-btn:hover {
  border-color: #ff5000;
  color: #ff5000;
}

/* 购物车内容 */
.cart-content {
  background: white;
  border-radius: 8px;
  overflow: hidden;
}

/* 购物车头部 */
.cart-header {
  display: flex;
  padding: 12px 20px;
  background: #fafafa;
  border-bottom: 1px solid #eee;
  color: #666;
  font-size: 12px;
}

.header-item {
  text-align: center;
}

.header-info {
  flex: 1;
  text-align: left;
  padding-left: 15px;
}

.header-price {
  width: 120px;
}

.header-quantity {
  width: 140px;
}

.header-total {
  width: 120px;
}

.header-action {
  width: 100px;
}

/* 店铺组 */
.shop-group {
  border-bottom: 1px solid #eee;
}

.shop-group:last-child {
  border-bottom: none;
}

.shop-header {
  display: flex;
  align-items: center;
  padding: 15px 20px;
  background: #f8f8f8;
  border-bottom: 1px solid #eee;
}

.shop-icon {
  width: 16px;
  height: 16px;
  fill: #ff5000;
  margin: 0 5px;
}

.shop-name {
  color: #333;
  font-size: 14px;
  font-weight: 500;
}

.shop-link {
  color: #ff5000;
  font-size: 12px;
  margin-left: 10px;
  text-decoration: none;
}

.shop-link:hover {
  text-decoration: underline;
}

/* 商品项 */
.cart-item {
  display: flex;
  padding: 20px;
  border-bottom: 1px solid #f0f0f0;
  transition: background 0.2s;
}

.cart-item:hover {
  background: #fafafa;
}

.cart-item:last-child {
  border-bottom: none;
}

.cart-item.out-of-stock {
  opacity: 0.6;
  background: #f9f9f9;
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
  accent-color: #ff5000;
}

.checkbox-label input[type="checkbox"]:disabled {
  cursor: not-allowed;
}

/* 商品图片 */
.item-image-box {
  position: relative;
  width: 80px;
  height: 80px;
  margin: 0 15px;
}

.item-image {
  width: 80px;
  height: 80px;
  object-fit: cover;
  border: 1px solid #eee;
  border-radius: 4px;
}

.item-tags {
  position: absolute;
  bottom: 0;
  left: 0;
  right: 0;
  display: flex;
  gap: 2px;
}

.tag {
  background: #ff5000;
  color: white;
  font-size: 10px;
  padding: 2px 4px;
  border-radius: 2px;
}

/* 商品信息 */
.item-info {
  flex: 1;
  min-width: 0;
}

.item-name {
  color: #333;
  font-size: 13px;
  line-height: 1.4;
  display: -webkit-box;
  -webkit-line-clamp: 2;
  -webkit-box-orient: vertical;
  overflow: hidden;
  text-decoration: none;
  margin-bottom: 8px;
  transition: color 0.2s;
}

.item-name:hover {
  color: #ff5000;
}

.item-sku {
  font-size: 12px;
  color: #999;
  margin-bottom: 8px;
}

.edit-sku {
  color: #ff5000;
  text-decoration: none;
  margin-left: 8px;
  cursor: pointer;
}

.promotion-tag {
  font-size: 12px;
  color: #ff5000;
  margin-bottom: 8px;
  display: flex;
  align-items: center;
  gap: 4px;
}

.tag-icon {
  font-size: 12px;
}

.item-actions {
  display: flex;
  gap: 15px;
  font-size: 12px;
}

.action-link {
  color: #666;
  text-decoration: none;
  transition: color 0.2s;
}

.action-link:hover {
  color: #ff5000;
}

/* 价格 */
.item-price {
  width: 120px;
  text-align: center;
  font-size: 14px;
}

.price-symbol {
  font-size: 12px;
}

.price-value {
  font-size: 14px;
  color: #333;
}

.original-price {
  font-size: 12px;
  color: #999;
  text-decoration: line-through;
  margin-top: 4px;
}

/* 数量 */
.item-quantity {
  width: 140px;
  text-align: center;
}

.quantity-control {
  display: inline-flex;
  align-items: center;
  border: 1px solid #ddd;
  border-radius: 4px;
  overflow: hidden;
  background: white;
}

.quantity-control.disabled {
  opacity: 0.5;
}

.qty-btn {
  width: 28px;
  height: 26px;
  border: none;
  background: #f5f5f5;
  cursor: pointer;
  font-size: 16px;
  color: #666;
  transition: background 0.2s;
}

.qty-btn:hover:not(:disabled) {
  background: #e8e8e8;
}

.qty-btn:disabled {
  color: #ccc;
  cursor: not-allowed;
}

.qty-input {
  width: 40px;
  height: 26px;
  border: none;
  text-align: center;
  font-size: 13px;
  outline: none;
  color: #333;
}

.qty-input:disabled {
  background: #f5f5f5;
  color: #999;
}

.stock-info {
  display: block;
  font-size: 12px;
  color: #999;
  margin-top: 5px;
}

.out-of-stock-text {
  display: block;
  font-size: 12px;
  color: #ff5000;
  margin-top: 5px;
}

/* 小计 */
.item-total {
  width: 120px;
  text-align: center;
}

.item-total .price-value {
  color: #ff5000;
  font-weight: 600;
  font-size: 16px;
}

.save-price {
  font-size: 12px;
  color: #ff5000;
  margin-top: 4px;
}

/* 操作 */
.item-action {
  width: 100px;
  text-align: center;
  display: flex;
  flex-direction: column;
  gap: 8px;
  align-items: center;
}

.action-btn {
  padding: 6px 12px;
  background: transparent;
  border: 1px solid #ddd;
  border-radius: 4px;
  font-size: 12px;
  cursor: pointer;
  transition: all 0.2s;
  color: #666;
}

.action-btn:hover {
  border-color: #ff5000;
  color: #ff5000;
}

.action-btn.favorite:hover {
  border-color: #ff6b6b;
  color: #ff6b6b;
}

/* 底部结算栏 */
.cart-footer {
  position: fixed;
  bottom: 0;
  left: 0;
  right: 0;
  background: white;
  box-shadow: 0 -2px 10px rgba(0, 0, 0, 0.1);
  z-index: 100;
}

.footer-content {
  max-width: 1200px;
  margin: 0 auto;
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 12px 20px;
}

.footer-left,
.footer-right {
  display: flex;
  align-items: center;
  gap: 15px;
}

.footer-btn {
  padding: 6px 16px;
  background: transparent;
  border: 1px solid #ddd;
  border-radius: 4px;
  color: #666;
  font-size: 13px;
  cursor: pointer;
  transition: all 0.2s;
}

.footer-btn:hover {
  border-color: #ff5000;
  color: #ff5000;
}

.footer-info {
  display: flex;
  align-items: flex-end;
  gap: 15px;
}

.selected-count {
  font-size: 13px;
  color: #666;
}

.selected-count strong {
  color: #ff5000;
  font-size: 15px;
}

.discount-info {
  display: flex;
  flex-direction: column;
  gap: 2px;
  font-size: 12px;
  color: #999;
}

.discount-amount {
  color: #ff5000;
}

.total-price {
  font-size: 14px;
  color: #333;
}

.price-highlight {
  font-size: 22px;
  color: #ff5000;
  font-weight: 600;
  margin-left: 5px;
}

.checkout-btn {
  padding: 12px 50px;
  background: linear-gradient(135deg, #ff5000 0%, #ff6b00 100%);
  color: white;
  border: none;
  border-radius: 20px;
  font-size: 16px;
  font-weight: 600;
  cursor: pointer;
  transition: all 0.3s;
  min-width: 120px;
}

.checkout-btn:hover:not(:disabled) {
  transform: translateY(-2px);
  box-shadow: 0 4px 12px rgba(255, 80, 0, 0.3);
}

.checkout-btn:disabled {
  background: #ccc;
  cursor: not-allowed;
}

/* 响应式 */
@media (max-width: 1200px) {
  .item-info {
    min-width: 200px;
  }
}

@media (max-width: 768px) {
  .nav-content {
    padding: 8px 15px;
  }
  
  .nav-right {
    display: none;
  }
  
  .cart-header {
    display: none;
  }
  
  .cart-item {
    flex-wrap: wrap;
    padding: 15px;
  }
  
  .checkbox-label {
    width: 100%;
    margin-bottom: 10px;
  }
  
  .item-image-box {
    margin: 0 10px 0 0;
  }
  
  .item-info {
    flex: 1;
    min-width: calc(100% - 95px);
    margin-bottom: 15px;
  }
  
  .item-price,
  .item-quantity,
  .item-total,
  .item-action {
    width: auto;
    flex: 1;
    min-width: 80px;
  }
  
  .footer-content {
    flex-wrap: wrap;
    padding: 10px 15px;
  }
  
  .footer-left {
    width: 100%;
    justify-content: space-between;
    margin-bottom: 10px;
  }
  
  .footer-info {
    gap: 8px;
  }
  
  .checkout-btn {
    padding: 10px 30px;
    min-width: auto;
  }
}
</style>
