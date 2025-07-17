<script setup>
import { ref, onMounted, computed } from 'vue'
import VueApexCharts from 'vue3-apexcharts'
import axios from 'axios'

// Reactive data
const currentPrice = ref('Loading...')
const priceChange = ref(0)
const priceChange24h = ref(0)
const selectedTimeframe = ref('1D')
const chartData = ref([])
const lineChartData = ref([])
const isLoading = ref(true)
const lastUpdate = ref(null)
const chartType = ref('candlestick') // 'candlestick' or 'line'

// Timeframe options with Binance intervals
const timeframes = [
  { label: '1H', interval: '1h', limit: 24 },
  { label: '4H', interval: '4h', limit: 24 },
  { label: '1D', interval: '1d', limit: 30 },
  { label: '1W', interval: '1w', limit: 24 },
  { label: '1M', interval: '1M', limit: 12 }
]

// Chart configuration
const chartOptions = computed(() => ({
  chart: {
    type: chartType.value,
    height: '100%',
    background: 'transparent',
    toolbar: {
      show: true,
      tools: {
        download: false,
        selection: false,
        zoom: false,
        zoomin: false,
        zoomout: false,
        pan: false,
        reset: false
      }
    },
    zoom: {
      enabled: false
    }
  },
  theme: {
    mode: 'dark'
  },
  title: {
    text: `BTC/USDT - ${selectedTimeframe.value} (${chartType.value === 'candlestick' ? 'Candlestick' : 'Line'})`,
    align: 'left',
    style: {
      color: '#ffffff',
      fontSize: '16px',
      fontWeight: 'bold'
    }
  },
  xaxis: {
    type: 'datetime',
    labels: {
      style: {
        colors: '#888'
      }
    },
    axisBorder: {
      color: '#404040'
    },
    axisTicks: {
      color: '#404040'
    }
  },
  yaxis: {
    tooltip: {
      enabled: true
    },
    labels: {
      style: {
        colors: '#888'
      },
      formatter: (value) => {
        return '$' + value.toLocaleString('en-US', {
          minimumFractionDigits: 2,
          maximumFractionDigits: 2
        })
      }
    }
  },
  grid: {
    borderColor: '#404040',
    strokeDashArray: 3
  },
  stroke: {
    curve: 'straight', // No smoothing for line chart
    width: chartType.value === 'line' ? 2 : 1,
    colors: ['#f7931a']
  },
  plotOptions: {
    candlestick: {
      colors: {
        upward: '#00ff88',
        downward: '#ff4444'
      },
      wick: {
        useFillColor: true
      }
    }
  },
  tooltip: {
    theme: 'dark',
    custom: function({ seriesIndex, dataPointIndex, w }) {
      if (chartType.value === 'candlestick') {
        const open = w.globals.seriesCandleO[seriesIndex][dataPointIndex]
        const high = w.globals.seriesCandleH[seriesIndex][dataPointIndex]
        const low = w.globals.seriesCandleL[seriesIndex][dataPointIndex]
        const close = w.globals.seriesCandleC[seriesIndex][dataPointIndex]
        const timestamp = w.globals.seriesX[seriesIndex][dataPointIndex]
        
        return `
          <div style="padding: 10px; background: #2a2a2a; border: 1px solid #404040; border-radius: 4px;">
            <div style="font-weight: bold; margin-bottom: 5px;">${new Date(timestamp).toLocaleString()}</div>
            <div>Open: <span style="color: #fff;">$${open.toFixed(2)}</span></div>
            <div>High: <span style="color: #00ff88;">$${high.toFixed(2)}</span></div>
            <div>Low: <span style="color: #ff4444;">$${low.toFixed(2)}</span></div>
            <div>Close: <span style="color: #fff;">$${close.toFixed(2)}</span></div>
          </div>
        `
      } else {
        // Line chart tooltip
        const price = w.globals.series[seriesIndex][dataPointIndex]
        const timestamp = w.globals.seriesX[seriesIndex][dataPointIndex]
        
        return `
          <div style="padding: 10px; background: #2a2a2a; border: 1px solid #404040; border-radius: 4px;">
            <div style="font-weight: bold; margin-bottom: 5px;">${new Date(timestamp).toLocaleString()}</div>
            <div>Price: <span style="color: #f7931a;">$${price.toFixed(2)}</span></div>
          </div>
        `
      }
    }
  }
}))

// Format price for display
const formatPrice = (price) => {
  return '$' + parseFloat(price).toLocaleString('en-US', {
    minimumFractionDigits: 2,
    maximumFractionDigits: 2
  })
}

// Fetch current price from Binance
const fetchCurrentPrice = async () => {
  try {
    const response = await axios.get('https://api.binance.com/api/v3/ticker/24hr?symbol=BTCUSDT')
    const data = response.data
    
    currentPrice.value = formatPrice(data.lastPrice)
    priceChange24h.value = parseFloat(data.priceChangePercent)
    priceChange.value = priceChange24h.value
    
    return parseFloat(data.lastPrice)
  } catch (error) {
    console.error('Error fetching current price:', error)
    currentPrice.value = 'Error'
    return null
  }
}

// Fetch kline data from Binance
const fetchChartData = async (timeframe = '1d', limit = 30) => {
  try {
    isLoading.value = true
    
    const response = await axios.get('https://api.binance.com/api/v3/klines', {
      params: {
        symbol: 'BTCUSDT',
        interval: timeframe,
        limit: limit
      }
    })
    
    const klineData = response.data.map(kline => ({
      x: new Date(kline[0]),
      y: [
        parseFloat(kline[1]), // Open
        parseFloat(kline[2]), // High
        parseFloat(kline[3]), // Low
        parseFloat(kline[4])  // Close
      ]
    }))
    
    // Create line chart data using close prices
    const lineData = response.data.map(kline => ({
      x: new Date(kline[0]),
      y: parseFloat(kline[4]) // Close price
    }))
    
    chartData.value = klineData
    lineChartData.value = lineData
    lastUpdate.value = new Date()
    
    // Update current price with the latest close price
    if (klineData.length > 0) {
      const latestCandle = klineData[klineData.length - 1]
      const latestPrice = latestCandle.y[3] // Close price
      
      // Calculate price change from previous candle
      if (klineData.length > 1) {
        const previousCandle = klineData[klineData.length - 2]
        const previousClose = previousCandle.y[3]
        const change = ((latestPrice - previousClose) / previousClose) * 100
        priceChange.value = change
      }
      
      currentPrice.value = formatPrice(latestPrice)
    }
    
  } catch (error) {
    console.error('Error fetching chart data:', error)
    chartData.value = []
  } finally {
    isLoading.value = false
  }
}

// Select timeframe and fetch data
const selectTimeframe = async (timeframe) => {
  selectedTimeframe.value = timeframe
  const config = timeframes.find(tf => tf.label === timeframe)
  if (config) {
    await fetchChartData(config.interval, config.limit)
  }
}

// Refresh chart data
const refreshChart = async () => {
  const config = timeframes.find(tf => tf.label === selectedTimeframe.value)
  if (config) {
    await fetchChartData(config.interval, config.limit)
  }
  await fetchCurrentPrice()
}

// Toggle chart type
const toggleChartType = () => {
  // Chart type is automatically updated by v-model, no need to toggle
  // This function can be used for any additional logic when chart type changes
}

// Initialize data
onMounted(async () => {
  await fetchCurrentPrice()
  const config = timeframes.find(tf => tf.label === selectedTimeframe.value)
  if (config) {
    await fetchChartData(config.interval, config.limit)
  }
  
  // Set up auto-refresh every 30 seconds
  setInterval(async () => {
    await fetchCurrentPrice()
  }, 30000)
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
        <span class="price">{{ currentPrice }}</span>
        <span class="price-change" :class="{ positive: priceChange > 0, negative: priceChange < 0 }">
          {{ priceChange > 0 ? '+' : '' }}{{ priceChange.toFixed(2) }}%
        </span>
      </div>

      <!-- Chart Controls -->
      <div class="chart-controls">
        <div class="timeframe-buttons">
          <button v-for="timeframe in timeframes" :key="timeframe.label" :class="{ active: selectedTimeframe === timeframe.label }"
            @click="selectTimeframe(timeframe.label)" class="timeframe-btn">
            {{ timeframe.label }}
          </button>
        </div>
        <div class="control-buttons">
          <div class="chart-type-dropdown">
            <select v-model="chartType" @change="toggleChartType" class="chart-type-select">
              <option value="candlestick">📊 Candlestick</option>
              <option value="line">📈 Line Chart</option>
            </select>
          </div>
          <button @click="refreshChart" class="refresh-btn">
            <span class="refresh-icon">⟳</span>
            Refresh
          </button>
        </div>
      </div>

      <!-- Chart Container -->
      <div class="chart-container">
        <div v-if="isLoading" class="chart-loading">
          <div class="loading-content">
            <div class="loading-spinner"></div>
            <p>Loading chart data...</p>
          </div>
        </div>
        <div v-else-if="chartData.length > 0" class="chart-wrapper">
          <VueApexCharts
            :type="chartType"
            :options="chartOptions"
            :series="[{ name: 'BTC/USDT', data: chartType === 'candlestick' ? chartData : lineChartData }]"
            height="100%"
          />
          <div class="last-update" v-if="lastUpdate">
            Last updated: {{ lastUpdate.toLocaleTimeString() }}
          </div>
        </div>
        <div v-else class="chart-error">
          <div class="error-content">
            <div class="error-icon">⚠️</div>
            <h3>Unable to load chart data</h3>
            <p>Please check your connection and try refreshing</p>
            <button @click="refreshChart" class="retry-btn">Retry</button>
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
  height: 100vh;
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
  padding: 0.8rem 0;
  position: sticky;
  top: 0;
  z-index: 100;
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

.logo-section {
  display: flex;
  flex-direction: column;
  text-align: center;
}

.logo {
  font-size: 1.8rem;
  font-weight: bold;
  margin: 0;
  color: #f7931a;
  text-shadow: 0 0 10px rgba(247, 147, 26, 0.3);
}

/* Price Section */
.price-section {
  display: flex;
  align-items: center;
  gap: 1rem;
  margin-bottom: 1rem;
  flex-shrink: 0;
}

.price {
  font-size: 1.8rem;
  font-weight: bold;
  color: #fff;
}

.price-change {
  font-size: 1rem;
  font-weight: bold;
  padding: 0.3rem 0.6rem;
  border-radius: 4px;
}

.price-change.positive {
  color: #00ff88;
  background: rgba(0, 255, 136, 0.1);
}

.price-change.negative {
  color: #ff4444;
  background: rgba(255, 68, 68, 0.1);
}

/* Main Content */
.main-content {
  flex: 1;
  max-width: 1200px;
  margin: 0 auto;
  padding: 1rem;
  width: 100%;
  box-sizing: border-box;
  display: flex;
  flex-direction: column;
  overflow: hidden;
  min-height: 0;
}

/* Chart Controls */
.chart-controls {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 1rem;
  flex-wrap: wrap;
  gap: 1rem;
  flex-shrink: 0;
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

.control-buttons {
  display: flex;
  gap: 0.5rem;
  flex-wrap: wrap;
  align-items: center;
  justify-content: center;
}

.chart-type-dropdown {
  position: relative;
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
  min-width: 140px;
  appearance: none;
  background-image: url("data:image/svg+xml;charset=UTF-8,%3csvg xmlns='http://www.w3.org/2000/svg' viewBox='0 0 24 24' fill='none' stroke='%23fff' stroke-width='2' stroke-linecap='round' stroke-linejoin='round'%3e%3cpolyline points='6,9 12,15 18,9'%3e%3c/polyline%3e%3c/svg%3e");
  background-repeat: no-repeat;
  background-position: right 0.5rem center;
  background-size: 1rem;
  padding-right: 2.5rem;
}

.chart-type-select:hover {
  background-color: #4a4a4a;
  border-color: #f7931a;
}

.chart-type-select:focus {
  outline: none;
  border-color: #f7931a;
  box-shadow: 0 0 0 2px rgba(247, 147, 26, 0.2);
}

.chart-type-select option {
  background: #3a3a3a;
  color: #fff;
  padding: 0.5rem;
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
  height: 36px; /* Match dropdown height */
  box-sizing: border-box;
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
  padding: 1rem;
  margin-bottom: 1rem;
  flex: 1;
  display: flex;
  align-items: center;
  justify-content: center;
  min-height: 0;
  overflow: hidden;
}

.chart-wrapper {
  width: 100%;
  height: 100%;
  display: flex;
  flex-direction: column;
}

.chart-loading, .chart-error {
  display: flex;
  align-items: center;
  justify-content: center;
  height: 400px;
  width: 100%;
}

.loading-content, .error-content {
  text-align: center;
  color: #888;
}

.loading-spinner {
  width: 40px;
  height: 40px;
  border: 4px solid #404040;
  border-top: 4px solid #f7931a;
  border-radius: 50%;
  animation: spin 1s linear infinite;
  margin: 0 auto 1rem;
}

@keyframes spin {
  0% { transform: rotate(0deg); }
  100% { transform: rotate(360deg); }
}

.error-icon {
  font-size: 3rem;
  margin-bottom: 1rem;
}

.retry-btn {
  background: #f7931a;
  border: 1px solid #f7931a;
  color: #000;
  padding: 0.5rem 1rem;
  border-radius: 6px;
  cursor: pointer;
  font-weight: bold;
  margin-top: 1rem;
}

.retry-btn:hover {
  background: #ffb347;
  border-color: #ffb347;
}

.last-update {
  text-align: center;
  font-size: 0.8rem;
  color: #888;
  margin-top: 0.5rem;
  padding-top: 0.5rem;
  border-top: 1px solid #404040;
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

/* Footer */
.footer {
  background: #2a2a2a;
  border-top: 1px solid #404040;
  padding: 1rem 0;
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
  .header {
    padding: 0.6rem 0;
  }

  .header-content {
    flex-direction: column;
    gap: 0.5rem;
    text-align: center;
  }

  .logo {
    font-size: 1.3rem;
  }

  .main-content {
    padding: 0.8rem;
  }

  .price-section {
    flex-direction: row;
    align-items: center;
    justify-content: center;
    gap: 1rem;
    margin-bottom: 0.8rem;
  }

  .price {
    font-size: 1.4rem;
  }

  .chart-controls {
    flex-direction: column;
    align-items: stretch;
    margin-bottom: 0.8rem;
    gap: 0.8rem;
  }

  .timeframe-buttons {
    justify-content: center;
    order: 2;
  }

  .control-buttons {
    justify-content: center;
    order: 1;
  }

  .chart-container {
    padding: 0.8rem;
    margin-bottom: 0.8rem;
  }

  .footer {
    padding: 0.8rem 0;
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
    padding: 0.5rem;
  }

  .logo {
    font-size: 1.2rem;
  }

  .price {
    font-size: 1.2rem;
  }

  .price-change {
    font-size: 0.85rem;
    padding: 0.2rem 0.4rem;
  }

  .timeframe-btn,
  .refresh-btn,
  .chart-type-select {
    padding: 0.3rem 0.6rem;
    font-size: 0.75rem;
    height: 32px;
    box-sizing: border-box;
  }

  .chart-type-select {
    min-width: 120px;
  }

  .chart-container {
    padding: 0.5rem;
  }

  .footer {
    padding: 0.6rem 0;
  }
}
</style>
