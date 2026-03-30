<!DOCTYPE html>
<html>
<head>
<style>
    body {
        background: #0a0a0b;
        color: #e8e8f0;
    }

    .topbar {
        display: flex;
        justify-content: space-between;
        height: 36px;
        background: #111113;
        border-bottom: 1px solid #f0a500;
        align-items: center;
        padding: 0 16px;
    }

    .logo {
        font-size: 13px;
        letter-spacing: 0.2em;
        color: #f0a500;
        font-weight: 600;
    }

    .layout {
        display: grid;
        grid-template-columns: 200px 1fr 250px;
        height: calc(100vh - 36px);
    }

    .sidebar {
        background: #111113;
        border-right: 1px solid #2e2e38;
        padding: 12px;
        overflow-y: auto;
        height: calc(100vh - 36px);
    }

    .center {
        background: #0d0d0f;
        padding: 12px;
        overflow-y: auto;
        height: calc(100vh - 36px);
    }

    .right {
        background: #111113;
        border-left: 1px solid #2e2e38;
        padding: 12px;
        overflow-y: auto;
        height: calc(100vh - 36px);
    }

    .section-title {
        font-size: 10px;
        letter-spacing: 0.15em;
        color: #f0a500;
        margin-bottom: 12px;
    }
    .input-label {
        font-size: 10px;
        color: #686880;
        text-transform: uppercase;
        letter-spacing: 0.1em;
        margin-bottom: 4px;
        margin-top: 12px;
    }

    .ctrl-select {
        width: 100%;
        background: #18181c;
        border: 1px solid #2e2e38;
        color: #e8e8f0;
        font-size: 11px;
        padding: 6px 8px;
    }

    .stat-row {
        display: flex;
        justify-content: space-between;
        padding: 8px 0;
        border-bottom: 1px solid #2e2e38;
    }

    .stat-label {
        font-size: 11px;
        color: #686880;
    }

    .stat-value {
        font-size: 11px;
        color: #e8e8f0;
    }

    .green { color: #00c896; }
    .red   { color: #ff4455; }
</style>
</head>

<body>
<div class="topbar">
    <div class="logo">QUANTERM</div>
    <div id="clock">09:00:00</div>
</div>

<div class="layout">
    <div class="sidebar">
        <p class="section-title">CONTROLS</p>

            <p class="input-label">Ticker</p>
            <select class="ctrl-select" onchange="loadChart(this.value, currentPeriod)">
                <option value="RY.TO">RY.TO — Royal Bank</option>
                <option value="VAB.TO">VAB.TO — Bond ETF</option>
                <option value="VFV.TO">VFV.TO — S&P 500 ETF</option>
                <option value="XEQT.TO">XEQT.TO — Core Equity</option>
                <option value="XIU.TO">XIU.TO — TSX 60</option>
                <option value="ZAG.TO">ZAG.TO — Aggregate Bond</option>
                <option value="ZLB.TO">ZLB.TO — Low Vol ETF</option>
                <option value="AMZN">AMZN — Amazon</option>
                <option value="NVDA">NVDA — Nvidia</option>
            </select>

            <p class="input-label">Period</p>
            <select class="ctrl-select" onchange="loadChart(currentTicker, this.value)">
                <option value="1y">1 Year</option>
                <option value="2y">2 Years</option>
                <option value="5y">5 Years</option>
            </select>

            <p class="input-label">Strategy</p>
            <select class="ctrl-select" id="strategy-sel" onchange="loadChart(currentTicker, currentPeriod)">
                <option value="ma">MA Crossover</option>
                <option value="meanrev">Mean Reversion</option>
            </select>

            <p class="input-label">Slippage per trade</p>
            <input
                type="number"
                id="slippage-input"
                class="ctrl-select"
                value="0.1"
                min="0"
                max="2"
                step="0.05"
                placeholder="0.1%"
                onchange="loadChart(currentTicker, currentPeriod)"
            >
            <p class="input-label" style="color: #444; font-size:9px;">% cost per signal flip</p>

            <div class="sep" style="height:1px; background:#2e2e38; margin: 14px 0;"></div>
            <p class="section-title">PORTFOLIO</p>
            <div id="watchlist-panel"></div>

            <p class="input-label">Account Size ($)</p>
            <input type="number" id="account-size" class="ctrl-select" value="10000" min="1000" step="1000" onchange="loadChart(currentTicker, currentPeriod)">

            <p class="input-label">Risk per Trade (%)</p>
            <input type="number" id="risk-pct" class="ctrl-select" value="2" min="0.5" max="10" step="0.5" onchange="loadChart(currentTicker, currentPeriod)">

            <p class="input-label">Stop Loss (%)</p>
            <input type="number" id="stop-pct" class="ctrl-select" value="5" min="1" max="20" step="0.5" onchange="loadChart(currentTicker, currentPeriod)">

    </div>
    <div class="center">
        <p class="section-title">PRICE CHART</p>
        <div style="position:relative; height: 300px;">
            <canvas id="myChart"></canvas>
        </div>
        <p class="section-title" style="margin-top: 16px;">EQUITY CURVE</p>
        <div style="position:relative; height:180px;">
            <canvas id="equityChart"></canvas>
        </div>
        <p class="section-title" style="margin-top: 16px;">TRADE LOG</p>
        <div style="overflow-x: auto;">
            <table id="trade-log" style="width:100%; border-collapse: collapse; font-size: 11px;">
                <thead>
                    <tr style="border-bottom: 1px solid #2e2e38;">
                        <th style="text-align:left; padding: 6px 8px; color:#686880; font-weight:400;">Entry</th>
                        <th style="text-align:left; padding: 6px 8px; color:#686880; font-weight:400;">Exit</th>
                        <th style="text-align:right; padding: 6px 8px; color:#686880; font-weight:400;">Entry $</th>
                        <th style="text-align:right; padding: 6px 8px; color:#686880; font-weight:400;">Exit $</th>
                        <th style="text-align:right; padding: 6px 8px; color:#686880; font-weight:400;">Return</th>
                    </tr>
                </thead>
                <tbody id="trade-log-body"></tbody>
            </table>
        </div>
        <div style="margin-top: 16px; padding: 12px; background: #18181c; border: 1px solid #2e2e38;">
            <p class="section-title" style="margin-bottom: 10px;">POSITION CALCULATOR</p>
            <div class="stat-row">
                <span class="stat-label">Current Price</span>
                <span class="stat-value" id="pos-price">—</span>
            </div>
            <div class="stat-row">
                <span class="stat-label">Position Size ($)</span>
                <span class="stat-value" id="pos-size">—</span>
            </div>
            <div class="stat-row">
                <span class="stat-label">Shares to Buy</span>
                <span class="stat-value" id="pos-shares">—</span>
            </div>
            <div class="stat-row">
                <span class="stat-label">Stop Loss Price</span>
                <span class="stat-value" id="pos-stop">—</span>
            </div>
            <div class="stat-row">
                <span class="stat-label">Max Loss ($)</span>
                <span class="stat-value red" id="pos-maxloss">—</span>
            </div>
        </div>
        <div style="margin-top: 16px;">
            <p class="section-title">PORTFOLIO COMPARISON</p>
            <table id="comparison-table" style="width:100%; border-collapse: collapse; font-size: 11px;">
                <thead>
                    <tr style="border-bottom: 1px solid #2e2e38; background: #111113;">
                        <th style="text-align:left; padding: 6px 4px; color:#686880; font-weight:400;">Ticker</th>
                        <th style="text-align:right; padding: 6px 4px; color:#686880; font-weight:400;">Price</th>
                        <th style="text-align:right; padding: 6px 4px; color:#686880; font-weight:400;">1Y Return</th>
                        <th style="text-align:right; padding: 6px 4px; color:#686880; font-weight:400;">Volatility</th>
                        <th style="text-align:right; padding: 6px 4px; color:#686880; font-weight:400;">Signal</th>
                    </tr>
                </thead>
                <tbody id="comparison-body">
                    <tr><td colspan="5" style="padding:8px 4px; color:#686880;">loading portfolio...</td></tr>
                </tbody>
            </table>
        </div>
        <p class="section-title" style="margin-top:16px;">PORTFOLIO EQUITY</p>
        <div style="position:relative; height:200px;">
            <canvas id="portfolioChart"></canvas>
        </div>
    </div>
    <div class="right">
    <p class="section-title">ANALYTICS</p>

    <div class="stat-row">
        <span class="stat-label">Sortino Ratio</span>
        <span class="stat-value" id="sortino-val">—</span>
    </div>

    <div class="stat-row">
        <span class="stat-label">Max Drawdown</span>
        <span class="stat-value" id="drawdown-val">—</span>
    </div>

    <div class="stat-row">
        <span class="stat-label">Ann. Return</span>
        <span class="stat-value" id="ann-return-val">—</span>
    </div>

    <div class="stat-row">
        <span class="stat-label">Return (after costs)</span>
        <span class="stat-value" id="net-return-val">—</span>
    </div>

    <div class="stat-row">
        <span class="stat-label">Volatility</span>
        <span class="stat-value" id="vol-val">—</span>
    </div>

    <div class="stat-row">
        <span class="stat-label">Signal</span>
        <span class="stat-value" id="signal-val">—</span>
    </div>

    <div class="stat-row">
        <span class="stat-label">Bench. Corr (XIU)</span>
        <span class="stat-value" id="corr-val">—</span>
    </div>

    <div class="stat-row">
        <span class="stat-label">Win Rate</span>
        <span class="stat-value" id="winrate-val">—</span>
    </div>

    <div class="stat-row">
        <span class="stat-label">Avg Win</span>
        <span class="stat-value" id="avgwin-val">—</span>
    </div>

    <div class="stat-row">
        <span class="stat-label">Avg Loss</span>
        <span class="stat-value" id="avgloss-val">—</span>
    </div>
</div>
</div>
<script src="https://cdnjs.cloudflare.com/ajax/libs/Chart.js/4.4.1/chart.umd.js"></script>

<script>
  let myChart = null;
  let currentTicker = 'RY.TO';
  let currentPeriod = '1y';

  function dailyReturns(closes) {
    const rets = [];
    for (let i = 1; i < closes.length; i++) {
      rets.push((closes[i] - closes[i-1]) / closes[i-1]);
    }
    return rets;
  }

  async function fetchCloses(ticker, range) {
    const url = `https://query1.finance.yahoo.com/v8/finance/chart/${ticker}?range=${range}&interval=1d`;
    const proxy = `https://api.allorigins.win/get?url=${encodeURIComponent(url)}`;
    const res = await fetch(proxy);
    const wrapper = await res.json();
    const raw = JSON.parse(wrapper.contents);
    return raw.chart.result[0].indicators.quote[0].close.filter(v => v !== null);
  }

  function changeColor(v) {
    return parseFloat(v) >= 0 ? '#00c896' : '#ff4455';
  }

  function changeSign(v) {
    return parseFloat(v) >= 0 ? '+' : '';
  }

  function correlation(a, b) {
    const n = Math.min(a.length, b.length);
    const ax = a.slice(0, n);
    const bx = b.slice(0, n);
    const meanA = ax.reduce((s,v) => s+v, 0) / n;
    const meanB = bx.reduce((s,v) => s+v, 0) / n;
    const cov = ax.reduce((s,v,i) => s + (v-meanA)*(bx[i]-meanB), 0) / n;
    const stdA = Math.sqrt(ax.reduce((s,v) => s + (v-meanA)**2, 0) / n);
    const stdB = Math.sqrt(bx.reduce((s,v) => s + (v-meanB)**2, 0) / n);
    return (cov / (stdA * stdB)).toFixed(2);
  }

  function rollingStd(data, window) {
    return data.map((val, idx) => {
      if (idx < window - 1) return null;
      const slice = data.slice(idx - window + 1, idx + 1);
      const mean = slice.reduce((a, b) => a + b, 0) / window;
      const variance = slice.reduce((a, b) => a + (b - mean) ** 2, 0) / window;
      return Math.sqrt(variance);
    });
  }

  function movingAverage(data, window) {
    return data.map((val, idx) => {
      if (idx < window - 1) return null;
      const slice = data.slice(idx - window + 1, idx + 1);
      return slice.reduce((a, b) => a + b, 0) / window;
    });
  }

  async function loadChart(ticker = 'RY.TO', period = '1y') {
    currentTicker = ticker;
    currentPeriod = period;
    const url = `https://query1.finance.yahoo.com/v8/finance/chart/${ticker}?range=${period}&interval=1d`;
    const proxy = `https://api.allorigins.win/get?url=${encodeURIComponent(url)}`;

    const response = await fetch(proxy);
    const wrapper = await response.json();
    const raw = JSON.parse(wrapper.contents);

    const result = raw.chart.result[0];
    const rawCloses = result.indicators.quote[0].close;
    const rawTimestamps = result.timestamp;

    const aligned = rawTimestamps
      .map((t, i) => ({ t, c: rawCloses[i] }))
      .filter(row => row.c !== null && row.c !== undefined);

    const timestamps = aligned.map(row => row.t);
    const closes = aligned.map(row => row.c);

    const labels = timestamps.map(t => {
      const d = new Date(t * 1000);
      return d.toLocaleDateString('en-CA', { month: 'short', day: 'numeric' });
    });

    const fastMA = movingAverage(closes, 20);
    const slowMA = movingAverage(closes, 50);
    const stdDev = rollingStd(closes, 20);

    const strategy = document.getElementById('strategy-sel').value;

    let signal;
    if (strategy === 'meanrev') {
      const lastPrice = closes[closes.length - 1];
      const lastMean = fastMA[fastMA.length - 1];
      const lastStd = stdDev[stdDev.length - 1];
      if (lastPrice < lastMean - 2 * lastStd) signal = 'BUY';
      else if (lastPrice > lastMean + 2 * lastStd) signal = 'SELL';
      else signal = 'HOLD';
    } else {
      signal = fastMA[fastMA.length - 1] > slowMA[slowMA.length - 1] ? 'BUY' : 'SELL';
    }
    const signalColor = signal === 'BUY' ? '#00c896' : signal === 'SELL' ? '#ff4455' : '#f0a500';

    const sigEl = document.getElementById('signal-val');
    sigEl.textContent = signal;
    sigEl.style.color = signalColor;

    const returns = dailyReturns(closes);

    let corr = '—';
    try {
      const benchCloses = await fetchCloses('XIU.TO', period);
      corr = correlation(returns, dailyReturns(benchCloses));
    } catch(e) {
      corr = 'N/A';
    }
    document.getElementById('corr-val').textContent = corr;

    const totalReturn = (closes[closes.length-1] - closes[0]) / closes[0];
    const annReturn = (totalReturn * (252 / closes.length) * 100).toFixed(1);

    const avgReturn = returns.reduce((a,b) => a+b, 0) / returns.length;
    const variance = returns.reduce((a,b) => a + (b-avgReturn)**2, 0) / returns.length;
    const vol = (Math.sqrt(variance) * Math.sqrt(252) * 100).toFixed(1);

    const negReturns = returns.filter(r => r < 0);
    const downVariance = negReturns.reduce((a,b) => a + b**2, 0) / returns.length;
    const sortino = (annReturn / (Math.sqrt(downVariance / returns.length) * Math.sqrt(252) * 100)).toFixed(2);

    let peak = closes[0];
    let maxDD = 0;
    for (let i = 1; i < closes.length; i++) {
      if (closes[i] > peak) peak = closes[i];
      const dd = (closes[i] - peak) / peak;
      if (dd < maxDD) maxDD = dd;
    }
    const drawdown = (maxDD * 100).toFixed(1);

    document.getElementById('sortino-val').textContent = sortino;
    document.getElementById('drawdown-val').textContent = drawdown + '%';
    document.getElementById('ann-return-val').textContent = '+' + annReturn + '%';
    document.getElementById('vol-val').textContent = vol + '%';

    const slippage = parseFloat(document.getElementById('slippage-input').value) / 100;
    const stopPct = parseFloat(document.getElementById('stop-pct').value) / 100;

    // Single backtest loop — equity curve + trade count, stop-loss applied consistently
    const equityHistory = [10000];
    let bal = 10000;
    let pos = 'cash';
    let entryPrice = null;
    let tradeCount = 0;

    for (let i = 50; i < closes.length; i++) {
      const fast = fastMA[i];
      const slow = slowMA[i];
      if (fast === null || slow === null) continue;

      let newPos = fast > slow ? 'long' : 'cash';

      if (pos === 'long' && entryPrice !== null) {
        if ((closes[i] - entryPrice) / entryPrice < -stopPct) newPos = 'cash';
      }

      if (newPos !== pos) {
        bal *= (1 - slippage);
        tradeCount++;
        entryPrice = newPos === 'long' ? closes[i] : null;
      }

      if (newPos === 'long') {
        bal *= (1 + (closes[i] - closes[i-1]) / closes[i-1]);
      }

      pos = newPos;
      equityHistory.push(bal);
    }

    const netReturn = (((bal - 10000) / 10000) * 100).toFixed(1);
    const netEl = document.getElementById('net-return-val');
    netEl.textContent = changeSign(netReturn) + netReturn + '%  (' + tradeCount + ' trades)';
    netEl.style.color = changeColor(netReturn);

    const equityLabels = labels.slice(labels.length - equityHistory.length);
    const buyHold = closes.map(c => 10000 * (c / closes[0]));

    if (window.equityChartInst) window.equityChartInst.destroy();
    window.equityChartInst = new Chart(document.getElementById('equityChart'), {
      type: 'line',
      data: {
        labels: equityLabels,
        datasets: [
          {
            label: 'Strategy ($)',
            data: equityHistory,
            borderColor: '#00c896',
            borderWidth: 1.5,
            pointRadius: 0,
            fill: false
          },
          {
            label: 'Buy & Hold ($)',
            data: buyHold.slice(buyHold.length - equityHistory.length),
            borderColor: '#4488ff',
            borderWidth: 1,
            pointRadius: 0,
            borderDash: [4, 4],
            fill: false
          }
        ]
      },
      options: {
        responsive: true,
        maintainAspectRatio: false,
        interaction: { mode: 'index', intersect: false },
        plugins: {
          legend: { labels: { color: '#686880' } },
          tooltip: {
            mode: 'index',
            intersect: false,
            callbacks: {
              label: ctx => ctx.dataset.label + ': $' + Math.round(ctx.raw).toLocaleString()
            }
          }
        },
        scales: {
          x: { ticks: { color: '#686880', maxTicksLimit: 5 } },
          y: { ticks: { color: '#686880', callback: v => '$' + v.toLocaleString() } }
        }
      }
    });

    const trades2 = [];
    let tradeEntry = null;

    for (let i = 50; i < closes.length; i++) {
      const fast = fastMA[i];
      const slow = slowMA[i];
      if (fast === null || slow === null) continue;

      const newPos = fast > slow ? 'long' : 'cash';
      const prevPos = i > 50 ? (fastMA[i-1] > slowMA[i-1] ? 'long' : 'cash') : 'cash';

      if (prevPos === 'cash' && newPos === 'long') {
        tradeEntry = { date: labels[i], price: closes[i] };
      }

      if (prevPos === 'long' && newPos === 'cash' && tradeEntry) {
        const ret = ((closes[i] - tradeEntry.price) / tradeEntry.price * 100).toFixed(2);
        trades2.push({
          entry: tradeEntry.date,
          exit: labels[i],
          entryPrice: tradeEntry.price.toFixed(2),
          exitPrice: closes[i].toFixed(2),
          ret
        });
        tradeEntry = null;
      }
    }

    const wins = trades2.filter(t => parseFloat(t.ret) > 0);
    const losses = trades2.filter(t => parseFloat(t.ret) < 0);
    const winRate = trades2.length > 0 ? ((wins.length / trades2.length) * 100).toFixed(0) : 0;
    const avgWin = wins.length > 0 ? (wins.reduce((a,t) => a + parseFloat(t.ret), 0) / wins.length).toFixed(2) : 0;
    const avgLoss = losses.length > 0 ? (losses.reduce((a,t) => a + parseFloat(t.ret), 0) / losses.length).toFixed(2) : 0;

    document.getElementById('winrate-val').textContent = winRate + '%';
    document.getElementById('avgwin-val').textContent = '+' + avgWin + '%';
    document.getElementById('avgloss-val').textContent = avgLoss + '%';
    document.getElementById('avgloss-val').style.color = '#ff4455';

    const accountSize = parseFloat(document.getElementById('account-size').value);
    const riskPct = parseFloat(document.getElementById('risk-pct').value) / 100;
    const currentPrice = closes[closes.length - 1];
    const maxLossDollars = accountSize * riskPct;
    const positionSize = maxLossDollars / stopPct;

    document.getElementById('pos-price').textContent = '$' + currentPrice.toFixed(2);
    document.getElementById('pos-size').textContent = '$' + positionSize.toFixed(0);
    document.getElementById('pos-shares').textContent = Math.floor(positionSize / currentPrice) + ' shares';
    document.getElementById('pos-stop').textContent = '$' + (currentPrice * (1 - stopPct)).toFixed(2);
    document.getElementById('pos-maxloss').textContent = '-$' + maxLossDollars.toFixed(0);

    const tbody = document.getElementById('trade-log-body');
    if (trades2.length === 0) {
      tbody.innerHTML = '<tr><td colspan="5" style="padding:8px; color:#686880; text-align:center;">no completed trades</td></tr>';
    } else {
      tbody.innerHTML = trades2.map(t => `
        <tr style="border-bottom: 1px solid #1a1a22;">
          <td style="padding: 6px 8px; color:#a0a0b8;">${t.entry}</td>
          <td style="padding: 6px 8px; color:#a0a0b8;">${t.exit}</td>
          <td style="padding: 6px 8px; text-align:right; color:#e8e8f0;">$${t.entryPrice}</td>
          <td style="padding: 6px 8px; text-align:right; color:#e8e8f0;">$${t.exitPrice}</td>
          <td style="padding: 6px 8px; text-align:right; color:${changeColor(t.ret)};">${changeSign(t.ret)}${t.ret}%</td>
        </tr>`).join('');
    }

    if (myChart) myChart.destroy();
    myChart = new Chart(document.getElementById('myChart'), {
      type: 'line',
      data: {
        labels: labels,
        datasets: [
          {
            label: ticker,
            data: closes,
            borderColor: '#f0a500',
            borderWidth: 1.5,
            pointRadius: 0,
            tension: 0.2
          },
          {
            label: 'Fast MA (20)',
            data: fastMA,
            borderColor: '#4488ff',
            borderWidth: 1,
            pointRadius: 0,
            tension: 0.2,
            borderDash: []
          },
          {
            label: 'Slow MA (50)',
            data: slowMA,
            borderColor: '#ff4455',
            borderWidth: 1,
            pointRadius: 0,
            tension: 0.2,
            borderDash: [4, 4]
          }
        ]
      },
      options: {
        responsive: true,
        maintainAspectRatio: false,
        plugins: { legend: { labels: { color: '#686880' } } },
        scales: {
          x: { ticks: { color: '#686880', maxTicksLimit: 6 } },
          y: { ticks: { color: '#686880' } }
        }
      }
    });
  }

  setInterval(() => {
    const now = new Date();
    const hh = String(now.getHours()).padStart(2, '0');
    const mm = String(now.getMinutes()).padStart(2, '0');
    const ss = String(now.getSeconds()).padStart(2, '0');
    document.getElementById('clock').textContent = `${hh}:${mm}:${ss}`;
  }, 1000);

  const myPortfolio = ['RY.TO','VAB.TO','VFV.TO','XEQT.TO','XIU.TO','ZAG.TO','ZLB.TO','AMZN','NVDA'];

  async function loadWatchlist() {
    const panel = document.getElementById('watchlist-panel');
    panel.innerHTML = '<p style="font-size:10px; color:#686880;">loading...</p>';

    const results = await Promise.all(myPortfolio.map(async ticker => {
      try {
        const closes = await fetchCloses(ticker, '5d');
        const price = closes[closes.length - 1];
        const prev = closes[closes.length - 2];
        const chg = ((price - prev) / prev * 100).toFixed(2);
        return { ticker, price: price.toFixed(2), chg };
      } catch(e) {
        return { ticker, price: '—', chg: '0' };
      }
    }));

    panel.innerHTML = results.map(r => `
      <div onclick="loadChart('${r.ticker}', currentPeriod)" style="
        padding: 6px 0;
        border-bottom: 1px solid #1a1a22;
        cursor: pointer;
        display: flex;
        justify-content: space-between;
        align-items: center;
      ">
        <span style="font-size:11px; color:#e8e8f0;">${r.ticker}</span>
        <div style="text-align:right;">
          <div style="font-size:11px; color:#e8e8f0;">$${r.price}</div>
          <div style="font-size:10px; color:${changeColor(r.chg)};">${changeSign(r.chg)}${r.chg}%</div>
        </div>
      </div>`).join('');
  }

  async function loadPortfolioComparison() {
    const tbody = document.getElementById('comparison-body');

    const results = await Promise.all(myPortfolio.map(async ticker => {
      try {
        const closes = await fetchCloses(ticker, '1y');
        const price = closes[closes.length - 1];
        const annReturn = ((closes[closes.length-1] - closes[0]) / closes[0] * 100).toFixed(1);

        const rets = dailyReturns(closes);
        const mean = rets.reduce((a,b) => a+b, 0) / rets.length;
        const variance = rets.reduce((a,b) => a+(b-mean)**2, 0) / rets.length;
        const vol = (Math.sqrt(variance) * Math.sqrt(252) * 100).toFixed(1);

        const fastMA = movingAverage(closes, 20);
        const slowMA = movingAverage(closes, 50);
        const signal = fastMA[fastMA.length-1] > slowMA[slowMA.length-1] ? 'BUY' : 'SELL';

        return { ticker, price: price.toFixed(2), annReturn, vol, signal };
      } catch(e) {
        return { ticker, price: '—', annReturn: '—', vol: '—', signal: '—' };
      }
    }));

    results.sort((a, b) => parseFloat(b.annReturn) - parseFloat(a.annReturn));

    tbody.innerHTML = results.map(r => `
      <tr style="border-bottom: 1px solid #1a1a22; cursor:pointer;" onclick="loadChart('${r.ticker}', currentPeriod)">
        <td style="padding: 6px 4px; color:#e8e8f0;">${r.ticker}</td>
        <td style="padding: 6px 4px; text-align:right; color:#e8e8f0;">$${r.price}</td>
        <td style="padding: 6px 4px; text-align:right; color:${changeColor(r.annReturn)};">${changeSign(r.annReturn)}${r.annReturn}%</td>
        <td style="padding: 6px 4px; text-align:right; color:#686880;">${r.vol}%</td>
        <td style="padding: 6px 4px; text-align:right; color:${r.signal === 'BUY' ? '#00c896' : '#ff4455'}; font-weight:500;">${r.signal}</td>
      </tr>`).join('');
  }

  loadChart();
  loadWatchlist();
  loadPortfolioComparison();

  async function loadPortfolioEquity() {
    const allData = await Promise.all(myPortfolio.map(async ticker => {
      try {
        const closes = await fetchCloses(ticker, '1y');
        const returns = closes.map(c => (c - closes[0]) / closes[0]);
        return { returns, len: closes.length };
      } catch(e) {
        return null;
      }
    }));

    const valid = allData.filter(d => d !== null);
    const minLen = Math.min(...valid.map(d => d.len));

    const portfolioEquity = [];
    for (let i = 0; i < minLen; i++) {
      const avgReturn = valid.reduce((sum, d) => sum + (d.returns[i] || 0), 0) / valid.length;
      portfolioEquity.push(10000 * (1 + avgReturn));
    }

    // Re-fetch labels for the shortest ticker to align x-axis
    const shortestTicker = myPortfolio[valid.indexOf(valid.find(d => d.len === minLen))];
    const url = `https://query1.finance.yahoo.com/v8/finance/chart/${shortestTicker}?range=1y&interval=1d`;
    const proxy = `https://api.allorigins.win/get?url=${encodeURIComponent(url)}`;
    const res = await fetch(proxy);
    const wrapper2 = await res.json();
    const raw = JSON.parse(wrapper2.contents);
    const result = raw.chart.result[0];
    const sharedLabels = result.timestamp.slice(0, minLen).map(t =>
      new Date(t * 1000).toLocaleDateString('en-CA', { month: 'short', day: 'numeric' })
    );

    if (window.portfolioChartInst) window.portfolioChartInst.destroy();
    window.portfolioChartInst = new Chart(document.getElementById('portfolioChart'), {
      type: 'line',
      data: {
        labels: sharedLabels,
        datasets: [{
          label: 'Portfolio ($)',
          data: portfolioEquity,
          borderColor: '#f0a500',
          borderWidth: 1.5,
          pointRadius: 0,
          fill: true,
          backgroundColor: 'rgba(240,165,0,0.05)'
        }]
      },
      options: {
        responsive: true,
        maintainAspectRatio: false,
        interaction: { mode: 'index', intersect: false },
        plugins: { legend: { labels: { color: '#686880' } } },
        scales: {
          x: { ticks: { color: '#686880', maxTicksLimit: 5 } },
          y: { ticks: { color: '#686880', callback: v => '$' + v.toLocaleString() } }
        }
      }
    });
  }

  loadPortfolioEquity();
</script>
</body>

</html>
