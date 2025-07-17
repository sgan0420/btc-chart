<script setup>
import { ref, onMounted, onUnmounted } from 'vue'
import axios from 'axios'

// Reactive data
const currentPrice = ref('Loading...')
const priceChange = ref(0)
const selectedTimeframe = ref('1D')
const chartType = ref('candlestick')
const isLoading = ref(true)
const lastUpdate = ref(null)

// Timeframe options
const timeframes = ['1H', '4H', '1D', '1W', '1M']

let priceUpdateInterval = null

// Format price for display
const formatPrice = (price) => {
  const num = parseFloat(price)
  if (num >= 1000000) {
    return (num / 1000000).toFixed(2) + 'M'
  } else if (num >= 1000) {
    return (num / 1000).toFixed(1) + 'K'
  } else if (num >= 1) {
    return num.toLocaleString('en-US', { 
      minimumFractionDigits: 2, 
      maximumFractionDigits: 2 
    })
  } else {
    return num.toFixed(6)
  }
}

// Fetch current price from Binance
const fetchCurrentPrice = async () => {
  try {
    const response = await axios.get('https://api.binance.com/api/v3/ticker/24hr?symbol=BTCUSDT')
    const data = response.data
    
    currentPrice.value = '$' + formatPrice(data.lastPrice)
    priceChange.value = parseFloat(data.priceChangePercent)
    lastUpdate.value = new Date()
    isLoading.value = false
    
    console.log('Price updated:', {
      price: data.lastPrice,
      change: data.priceChangePercent,
      time: new Date().toLocaleTimeString()
    })
    
  } catch (error) {
    console.error('Error fetching current price:', error)
    currentPrice.value = 'Error loading price'
    isLoading.value = false
  }
}

// Functions
const selectTimeframe = (timeframe) => {
  selectedTimeframe.value = timeframe
  console.log('Selected timeframe:', timeframe)
}

const refreshChart = async () => {
  console.log('Refreshing price...')
  isLoading.value = true
  await fetchCurrentPrice()
}

// Initialize data and set up auto-refresh
onMounted(async () => {
  console.log('App mounted, fetching initial price...')
  await fetchCurrentPrice()
  
  // Set up auto-refresh every 30 seconds
  priceUpdateInterval = setInterval(async () => {
    await fetchCurrentPrice()
  }, 30000)
})

// Cleanup on unmount
onUnmounted(() => {
  if (priceUpdateInterval) {
    clearInterval(priceUpdateInterval)
  }
})
</script>

<template>
  <div class="app">
    <!-- Header -->
    <header class="header">
      <div class="header-content">
        <div class="logo-section">
          <h1 class="logo">₿TC Chart</h1>
        </div>
      </div>
    </header>

    <!-- Main Content -->
    <main class="main-content">
      <!-- Price Section -->
      <div class="price-section">
        <span class="pair-label price">BTC/USD</span>
        <span class="price">{{ currentPrice }}</span>
        <span class="price-change" :class="{ positive: priceChange > 0, negative: priceChange < 0, zero: priceChange === 0 }">
          {{ priceChange > 0 ? '+' : '' }}{{ priceChange.toFixed(2) }}%
        </span>
      </div>

      <!-- Chart Controls -->
      <div class="chart-controls">
        <div class="timeframe-buttons">
          <button v-for="timeframe in timeframes" :key="timeframe" :class="{ active: selectedTimeframe === timeframe }"
            @click="selectTimeframe(timeframe)" class="timeframe-btn">
            {{ timeframe }}
          </button>
        </div>
        <div class="control-buttons">
          <select v-model="chartType" class="chart-type-select">
            <option value="candlestick">📊 Candlestick</option>
            <option value="line">📈 Line Chart</option>
          </select>
          <button @click="refreshChart" class="refresh-btn">
            <span class="refresh-icon">⟳</span>
            Refresh
          </button>
        </div>
      </div>

      <!-- Chart Container -->
      <div class="chart-container">
        <div class="chart-placeholder">
          <div class="placeholder-content">
            <div class="chart-icon">📈</div>
            <h3>Bitcoin Chart</h3>
            <p>Chart will be displayed here</p>
          </div>
        </div>
      </div>
    </main>

    <!-- Footer -->
    <footer class="footer">
      <div class="footer-content">
        <p>&copy; 2025 Shijie Gan</p>
        <div class="footer-links">
          <a href="https://shijiegan.vercel.app/" target="_blank" class="footer-link">View Author</a>
        </div>
      </div>
    </footer>
  </div>
</template>

<style scoped>
.app {
  min-height: 100vh;
  display: flex;
  flex-direction: column;
  background: #1a1a1a;
  color: #ffffff;
  overflow: hidden;
}

/* Header Styles */
.header {
  background: #2a2a2a;
  border-bottom: 1px solid #404040;
  padding: 1rem 0;
  flex-shrink: 0;
}

.header-content {
  max-width: 1200px;
  margin: 0 auto;
  padding: 0 1rem;
  display: flex;
  justify-content: center;
  align-items: center;
}

.logo {
  font-size: 1.8rem;
  font-weight: bold;
  margin: 0;
  color: #f7931a;
  text-shadow: 0 0 10px rgba(247, 147, 26, 0.3);
}

/* Main Content */
.main-content {
  flex: 1;
  max-width: 1200px;
  margin: 0 auto;
  padding: 2rem 1rem;
  width: 100%;
  box-sizing: border-box;
  display: flex;
  flex-direction: column;
  min-height: 0;
}

/* Price Section */
.price-section {
  text-align: center;
  margin-bottom: 1.5rem;
  display: flex;
  align-items: center;
  gap: 1rem;
}

.price {
  font-size: 2rem;
  font-weight: bold;
  color: #fff;
  margin: 0;
}

.price-change {
  font-size: 1rem;
  margin: 0;
  padding: 0.2rem 0.4rem;
  border-radius: 6px;
  font-weight: 600;
  min-width: 60px;
  text-align: center;
}

.price-change.positive {
  color: #ffffff;
  background-color: #00ff88;
}

.price-change.negative {
  color: #ffffff;
  background-color: #ff4444;
}

.price-change.zero {
  color: #ffffff;
  background-color: #666666;
}

.pair-label {
  font-size: 1.1rem;
  color: #fff;
  opacity: 0.7;
  font-weight: 500;
}

.pair-label.price {
  font-size: 2rem;
  font-weight: bold;
  color: #fff;
  letter-spacing: 0.5px;
}

/* Chart Controls */
.chart-controls {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 2rem;
  flex-wrap: wrap;
  gap: 1rem;
}

.timeframe-buttons {
  display: flex;
  gap: 0.5rem;
  flex-wrap: wrap;
}

.control-buttons {
  display: flex;
  gap: 0.5rem;
  align-items: center;
}

.timeframe-btn,
.refresh-btn {
  background: #3a3a3a;
  border: 1px solid #555;
  color: #fff;
  padding: 0.5rem 1rem;
  border-radius: 6px;
  cursor: pointer;
  transition: all 0.3s ease;
  font-size: 0.9rem;
  min-height: 40px;
  display: flex;
  align-items: center;
  gap: 0.3rem;
}

.refresh-btn {
  background: #00c97a;
  border: 1px solid #00c97a;
  color: #fff;
  padding: 0.5rem 1rem;
  border-radius: 6px;
  cursor: pointer;
  transition: all 0.3s ease;
  font-size: 0.9rem;
  min-height: 40px;
  display: flex;
  align-items: center;
  gap: 0.3rem;
}

.refresh-btn:hover {
  background: #00e68a;
  border-color: #00e68a;
}

.chart-type-select {
  background: #3a3a3a;
  border: 1px solid #555;
  color: #fff;
  padding: 0.5rem 1rem;
  border-radius: 6px;
  cursor: pointer;
  transition: all 0.3s ease;
  font-size: 0.9rem;
  min-height: 40px;
  min-width: 140px;
}

.chart-type-select:focus,
.timeframe-btn:hover,
.refresh-btn:hover {
  background: #4a4a4a;
  border-color: #f7931a;
  outline: none;
}

.timeframe-btn.active {
  background: #f7931a;
  border-color: #f7931a;
  color: #000;
}

.refresh-icon {
  font-size: 1rem;
}

/* Chart Container */
.chart-container {
  background: #2a2a2a;
  border: 1px solid #404040;
  border-radius: 12px;
  padding: 2rem;
  flex: 1;
  display: flex;
  align-items: center;
  justify-content: center;
  min-height: 0;
  margin-bottom: 1rem;
}

.chart-placeholder {
  text-align: center;
  width: 100%;
}

.placeholder-content {
  color: #888;
}

.chart-icon {
  font-size: 4rem;
  margin-bottom: 1rem;
  opacity: 0.7;
}

.placeholder-content h3 {
  margin: 0.5rem 0;
  color: #fff;
  font-size: 1.2rem;
}

.placeholder-content p {
  margin: 0;
  color: #888;
  font-size: 0.9rem;
}

/* Footer */
.footer {
  background: #2a2a2a;
  border-top: 1px solid #404040;
  padding: 1.5rem 0;
  flex-shrink: 0;
}

.footer-content {
  max-width: 1200px;
  margin: 0 auto;
  padding: 0 1rem;
  display: flex;
  justify-content: space-between;
  align-items: center;
  flex-wrap: wrap;
  gap: 1rem;
}

.footer-content p {
  margin: 0;
  color: #888;
  font-size: 0.9rem;
}

.footer-links {
  display: flex;
  gap: 2rem;
}

.footer-link {
  color: #888;
  text-decoration: none;
  font-size: 0.9rem;
  transition: color 0.3s ease;
}

.footer-link:hover {
  color: #f7931a;
}

/* Mobile Responsive */
@media (max-width: 768px) {
  .main-content {
    padding: 1rem;
  }

  .price-section {
    gap: 1rem;
    justify-content: center;
  }

  .price {
    font-size: 1.8rem;
  }

  .price-change {
    font-size: 0.9rem;
    padding: 0.2rem 0.4rem;
  }

  .chart-controls {
    flex-direction: column;
    align-items: stretch;
    gap: 1rem;
  }

  .timeframe-buttons {
    justify-content: center;
  }

  .control-buttons {
    justify-content: center;
  }

  .chart-container {
    padding: 1.5rem;
  }

  .footer-content {
    flex-direction: column;
    text-align: center;
  }

  .footer-links {
    justify-content: center;
  }
}

@media (max-width: 480px) {
  .main-content {
    padding: 1rem 0.5rem;
  }

  .price-section {
    gap: 0.8rem;
    flex-wrap: wrap;
  }

  .price {
    font-size: 1.6rem;
  }

  .price-change {
    font-size: 0.8rem;
    padding: 0.15rem 0.3rem;
    min-width: 50px;
  }

  .timeframe-btn,
  .refresh-btn,
  .chart-type-select {
    padding: 0.4rem 0.8rem;
    font-size: 0.8rem;
    min-height: 36px;
  }

  .chart-type-select {
    min-width: 120px;
  }

  .logo {
    font-size: 1.5rem;
  }

  .chart-icon {
    font-size: 3rem;
  }

  .chart-container {
    padding: 1rem;
  }
}
</style>
