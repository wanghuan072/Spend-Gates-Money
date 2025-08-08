<template>
  <article class="product-card" role="article">
    <div class="product-image">
      <img :src="product.image" :alt="`${product.name} product image`" />
      <div class="image-overlay" aria-hidden="true"></div>
    </div>

    <div class="product-info">
      <h3 class="product-name">{{ product.name }}</h3>
      <p class="product-price" aria-label="Product price">
        {{ formatCurrency(product.price) }}
      </p>
    </div>

    <div class="product-controls" role="group" aria-label="Product purchase controls">
      <button
        class="control-btn sell-btn"
        @click="handleSell"
        :disabled="quantity === 0"
        aria-label="Sell one unit of this product"
        :aria-describedby="`quantity-${product.id}`"
      >
        Sell
      </button>

      <div class="quantity-display" :id="`quantity-${product.id}`" aria-label="Current quantity">
        <span class="quantity-value" role="text">{{ quantity }}</span>
      </div>

      <button
        class="control-btn buy-btn"
        @click="handleBuy"
        :disabled="!canAfford"
        aria-label="Buy one unit of this product"
        :aria-describedby="`quantity-${product.id}`"
      >
        Buy
      </button>
    </div>
  </article>
</template>

<script setup>
import { computed } from 'vue'
import { useGameStore } from '../stores/gameStore'

const props = defineProps({
  product: {
    type: Object,
    required: true,
  },
})

const gameStore = useGameStore()
const quantity = computed(() => gameStore.getProductQuantity(props.product.id))

const canAfford = computed(() => {
  const totalCost = props.product.price * 1 // 每次只购买1个
  return gameStore.balance >= totalCost
})

const formatCurrency = (amount) => {
  return gameStore.formatCurrency(amount)
}

// 处理购买 - 增加数量并减少余额
const handleBuy = () => {
  if (gameStore.balance >= props.product.price) {
    gameStore.buyProduct(props.product.id, 1)
  }
}

// 处理出售 - 减少数量并增加余额
const handleSell = () => {
  if (quantity.value > 0) {
    gameStore.sellProduct(props.product.id, 1)
  }
}
</script>

<style scoped>
.product-card {
  background: rgba(255, 255, 255, 0.95);
  backdrop-filter: blur(20px);
  border-radius: 20px;
  padding: 24px;
  box-shadow: 0 10px 30px rgba(0, 0, 0, 0.08);
  border: 1px solid rgba(255, 255, 255, 0.3);
  transition: all 0.3s ease;
  position: relative;
  overflow: hidden;
}

.product-card::before {
  content: '';
  position: absolute;
  top: 0;
  left: 0;
  right: 0;
  height: 3px;
  background: linear-gradient(90deg, #a8edea, #fed6e3, #ffecd2, #fcb69f);
  background-size: 400% 400%;
  animation: gradientShift 3s ease infinite;
}

.product-card:hover {
  transform: translateY(-8px);
  box-shadow: 0 20px 40px rgba(0, 0, 0, 0.12);
}

@keyframes gradientShift {
  0% {
    background-position: 0% 50%;
  }
  50% {
    background-position: 100% 50%;
  }
  100% {
    background-position: 0% 50%;
  }
}

.product-image {
  width: 100%;
  height: 160px;
  margin-bottom: 20px;
  border-radius: 16px;
  overflow: hidden;
  background: linear-gradient(135deg, #f5f7fa 0%, #c3cfe2 100%);
  position: relative;
}

.product-image img {
  width: 100%;
  height: 100%;
  object-fit: cover;
  transition: transform 0.3s ease;
}

.product-card:hover .product-image img {
  transform: scale(1.05);
}

.image-overlay {
  position: absolute;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background: linear-gradient(135deg, rgba(168, 237, 234, 0.2), rgba(254, 214, 227, 0.2));
  opacity: 0;
  transition: opacity 0.3s ease;
}

.product-card:hover .image-overlay {
  opacity: 1;
}

.product-info {
  margin-bottom: 20px;
}

.product-name {
  font-size: 18px;
  font-weight: 700;
  color: #2d3748;
  margin-bottom: 8px;
  line-height: 1.4;
  min-height: 50px;
  display: flex;
  align-items: center;
}

.product-price {
  font-size: 20px;
  font-weight: 800;
  background: linear-gradient(135deg, #667eea, #764ba2);
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
  background-clip: text;
  margin-bottom: 0;
}

.product-controls {
  display: flex;
  gap: 12px;
  align-items: center;
}

.control-btn {
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 6px;
  padding: 12px 16px;
  border: none;
  border-radius: 12px;
  font-weight: 600;
  font-size: 14px;
  cursor: pointer;
  transition: all 0.3s ease;
  min-width: 80px;
  position: relative;
  overflow: hidden;
}

/* .control-btn::before {
  content: '';
  position: absolute;
  top: 0;
  left: -100%;
  width: 100%;
  height: 100%;
  background: linear-gradient(90deg, transparent, rgba(255, 255, 255, 0.2), transparent);
  transition: left 0.5s ease;
} */

.control-btn:hover::before {
  left: 100%;
}

.buy-btn {
  background: linear-gradient(135deg, #667eea, #764ba2);
  color: white;
  box-shadow: 0 4px 12px rgba(102, 126, 234, 0.2);
}

.buy-btn:hover:not(:disabled) {
  transform: translateY(-2px);
  box-shadow: 0 8px 20px rgba(102, 126, 234, 0.3);
}

.buy-btn:disabled {
  background: linear-gradient(135deg, #cbd5e0, #a0aec0);
  cursor: not-allowed;
  transform: none;
  box-shadow: none;
}

.sell-btn {
  background: linear-gradient(135deg, #ff9a9e, #fecfef);
  color: #2d3748;
  box-shadow: 0 4px 12px rgba(255, 154, 158, 0.2);
}

.sell-btn:hover:not(:disabled) {
  transform: translateY(-2px);
  box-shadow: 0 8px 20px rgba(255, 154, 158, 0.3);
}

.sell-btn:disabled {
  background: linear-gradient(135deg, #cbd5e0, #a0aec0);
  cursor: not-allowed;
  transform: none;
  box-shadow: none;
}

.quantity-display {
  flex: 1;
  display: flex;
  align-items: center;
  justify-content: center;
  padding: 12px 16px;
  background: rgba(160, 174, 192, 0.1);
  border: 2px solid rgba(160, 174, 192, 0.2);
  border-radius: 12px;
  font-size: 14px;
  transition: all 0.3s ease;
}

.quantity-display:hover {
  background: rgba(160, 174, 192, 0.15);
  border-color: rgba(160, 174, 192, 0.3);
}

.quantity-value {
  color: #2d3748;
  font-weight: 800;
  font-size: 20px;
  background: linear-gradient(135deg, #667eea, #764ba2);
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
  background-clip: text;
}

/* 响应式设计 */
@media (max-width: 1024px) {
  .product-card {
    padding: 20px;
  }

  .product-image {
    height: 140px;
    margin-bottom: 16px;
  }

  .product-name {
    font-size: 16px;
    min-height: 44px;
  }

  .product-price {
    font-size: 18px;
  }

  .control-btn {
    padding: 10px 14px;
    font-size: 13px;
    min-width: 70px;
  }

  .quantity-display {
    padding: 10px 14px;
  }

  .quantity-value {
    font-size: 18px;
  }
}

@media (max-width: 768px) {
  .product-card {
    padding: 10px;
  }

  .product-image {
    height: 120px;
    margin-bottom: 10px;
  }

  .product-name {
    font-size: 14px;
    min-height: 40px;
    margin-bottom: 6px;
  }

  .product-price {
    font-size: 16px;
  }

  .product-controls {
    gap: 8px;
  }

  .control-btn {
    padding: 8px 12px;
    font-size: 12px;
    min-width: 60px;
  }

  .quantity-display {
    padding: 8px 12px;
  }

  .quantity-value {
    font-size: 16px;
  }
}
</style> 