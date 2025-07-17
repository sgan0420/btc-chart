<script setup>
import { ref } from 'vue'

// Reactive data
const currentPrice = ref('Loading...')
const priceChange = ref(0)
const selectedTimeframe = ref('1D')

// Timeframe options
const timeframes = ['1H', '4H', '1D', '1W', '1M']

// Functions (will be implemented later)
const selectTimeframe = (timeframe) => {
  selectedTimeframe.value = timeframe
  console.log('Selected timeframe:', timeframe)
}

const refreshChart = () => {
  console.log('Refreshing chart...')
}
</script>

<template>
  <div class="app">
    <!-- Header -->
    <header class="header">
      <div class="header-content">
        <div class="logo-section">
          <h1 class="logo">₿TC Chart</h1>
          <span class="subtitle">Real-time Bitcoin Trading Chart</span>
        </div>
        <div class="price-section">
          <div class="price">{{ currentPrice }}</div>
          <div class="price-change" :class="{ positive: priceChange > 0, negative: priceChange < 0 }">
            {{ priceChange > 0 ? '+' : '' }}{{ priceChange.toFixed(2) }}%
          </div>
        </div>
      </div>
    </header>

    <!-- Main Content -->
    <main class="main-content">
      <!-- Chart Controls -->
      <div class="chart-controls">
        <div class="timeframe-buttons">
          <button v-for="timeframe in timeframes" :key="timeframe" :class="{ active: selectedTimeframe === timeframe }"
            @click="selectTimeframe(timeframe)" class="timeframe-btn">
            {{ timeframe }}
          </button>
        </div>
        <button @click="refreshChart" class="refresh-btn">
          <span class="refresh-icon">⟳</span>
          Refresh
        </button>
      </div>

      <!-- Chart Container -->
      <div class="chart-container">
        <div class="chart-placeholder">
          <div class="placeholder-content">
            <div class="chart-icon">📈</div>
            <h3>Bitcoin Candlestick Chart</h3>
            <p>Chart will be displayed here</p>
          </div>
        </div>
      </div>

      <!-- Chart Info -->
      <div class="chart-info">
        <div class="info-item">
          <span class="label">Pair:</span>
          <span class="value">BTC/USD</span>
        </div>
        <div class="info-item">
          <span class="label">Source:</span>
          <span class="value">CoinGecko</span>
        </div>
        <div class="info-item">
          <span class="label">Timeframe:</span>
          <span class="value">{{ selectedTimeframe }}</span>
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
}

/* Header Styles */
.header {
  background: #2a2a2a;
  border-bottom: 1px solid #404040;
  padding: 1rem 0;
  position: sticky;
  top: 0;
  z-index: 100;
}

.header-content {
  max-width: 1200px;
  margin: 0 auto;
  padding: 0 1rem;
  display: flex;
  justify-content: space-between;
  align-items: center;
}

.logo-section {
  display: flex;
  flex-direction: column;
}

.logo {
  font-size: 1.8rem;
  font-weight: bold;
  margin: 0;
  color: #f7931a;
  text-shadow: 0 0 10px rgba(247, 147, 26, 0.3);
}

.subtitle {
  font-size: 0.9rem;
  color: #888;
  margin-top: 0.2rem;
}

.price-section {
  text-align: right;
}

.price {
  font-size: 1.5rem;
  font-weight: bold;
  color: #fff;
}

.price-change {
  font-size: 1rem;
  margin-top: 0.2rem;
}

.price-change.positive {
  color: #00ff88;
}

.price-change.negative {
  color: #ff4444;
}

/* Main Content */
.main-content {
  flex: 1;
  max-width: 1200px;
  margin: 0 auto;
  padding: 2rem 1rem;
  width: 100%;
  box-sizing: border-box;
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

.timeframe-btn {
  background: #3a3a3a;
  border: 1px solid #555;
  color: #fff;
  padding: 0.5rem 1rem;
  border-radius: 6px;
  cursor: pointer;
  transition: all 0.3s ease;
  font-size: 0.9rem;
}

.timeframe-btn:hover {
  background: #4a4a4a;
  border-color: #f7931a;
}

.timeframe-btn.active {
  background: #f7931a;
  border-color: #f7931a;
  color: #000;
  font-weight: bold;
}

.refresh-btn {
  background: #2a5934;
  border: 1px solid #4a7c59;
  color: #fff;
  padding: 0.5rem 1rem;
  border-radius: 6px;
  cursor: pointer;
  transition: all 0.3s ease;
  display: flex;
  align-items: center;
  gap: 0.5rem;
}

.refresh-btn:hover {
  background: #4a7c59;
  transform: translateY(-1px);
}

.refresh-icon {
  font-size: 1.2rem;
}

/* Chart Container */
.chart-container {
  background: #2a2a2a;
  border: 1px solid #404040;
  border-radius: 12px;
  padding: 2rem;
  margin-bottom: 2rem;
  min-height: 400px;
  display: flex;
  align-items: center;
  justify-content: center;
}

.chart-placeholder {
  text-align: center;
  color: #888;
}

.placeholder-content {
  max-width: 300px;
}

.chart-icon {
  font-size: 4rem;
  margin-bottom: 1rem;
}

.chart-placeholder h3 {
  margin: 0 0 1rem 0;
  color: #fff;
}

.chart-placeholder p {
  margin: 0;
  font-size: 0.9rem;
}

/* Chart Info */
.chart-info {
  display: flex;
  justify-content: space-around;
  background: #2a2a2a;
  border: 1px solid #404040;
  border-radius: 8px;
  padding: 1rem;
  flex-wrap: wrap;
  gap: 1rem;
}

.info-item {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 0.3rem;
}

.label {
  font-size: 0.8rem;
  color: #888;
  text-transform: uppercase;
  letter-spacing: 0.5px;
}

.value {
  font-weight: bold;
  color: #fff;
}

/* Footer */
.footer {
  background: #2a2a2a;
  border-top: 1px solid #404040;
  padding: 1.5rem 0;
  margin-top: auto;
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
  .header-content {
    flex-direction: column;
    gap: 1rem;
    text-align: center;
  }

  .logo {
    font-size: 1.5rem;
  }

  .price {
    font-size: 1.3rem;
  }

  .chart-controls {
    flex-direction: column;
    align-items: stretch;
  }

  .timeframe-buttons {
    justify-content: center;
  }

  .chart-container {
    padding: 1rem;
    min-height: 300px;
  }

  .chart-info {
    flex-direction: column;
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

  .timeframe-btn,
  .refresh-btn {
    padding: 0.4rem 0.8rem;
    font-size: 0.8rem;
  }

  .logo {
    font-size: 1.3rem;
  }

  .chart-icon {
    font-size: 3rem;
  }
}
</style>
