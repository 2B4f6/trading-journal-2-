// Load trades from LocalStorage
let trades = JSON.parse(localStorage.getItem("trades")) || [];

// DOM elements
const form = document.getElementById("trade-form");
const tradesTableBody = document.querySelector("#trades-table tbody");
const totalTradesEl = document.getElementById("total-trades");
const totalGainEl = document.getElementById("total-gain");
const averageGainEl = document.getElementById("average-gain");
const totalRiskRewardEl = document.getElementById("total-risk-reward");

const strategyFilter = document.getElementById("strategy-filter");
const platformFilter = document.getElementById("platform-filter");

const gainChartCtx = document.getElementById("gainChart").getContext('2d');
let gainChart;

// Render trades table and summary
function renderTrades() {
    tradesTableBody.innerHTML = "";

    // Populate filter dropdowns
    const strategies = [...new Set(trades.map(t => t.strategy).filter(s => s))];
    const platforms = [...new Set(trades.map(t => t.platform).filter(p => p))];

    strategyFilter.innerHTML = '<option value="">All</option>' + strategies.map(s => `<option value="${s}">${s}</option>`).join('');
    platformFilter.innerHTML = '<option value="">All</option>' + platforms.map(p => `<option value="${p}">${p}</option>`).join('');

    // Apply filters
    const filteredTrades = trades.filter(t => {
        return (!strategyFilter.value || t.strategy === strategyFilter.value) &&
               (!platformFilter.value || t.platform === platformFilter.value);
    });

    let totalGain = 0;
    let totalRisk = 0;

    filteredTrades.forEach(trade => {
        totalGain += parseFloat(trade.profitLoss) || 0;
        totalRisk += parseFloat(trade.risk) || 0;

        let row = tradesTableBody.insertRow();
        row.insertCell(0).innerText = parseFloat(trade.profitLoss).toFixed(2);
        row.cells[0].className = trade.profitLoss >= 0 ? "profit" : "loss";
        row.insertCell(1).innerText = trade.dailyGain || "";
        row.insertCell(2).innerText = trade.strategy || "";
        row.insertCell(3).innerText = trade.risk || "";
        row.insertCell(4).innerText = trade.platform || "";
        row.insertCell(5).innerText = trade.confidence || "";
        row.insertCell(6).innerText = trade.emotion || "";
        const imgCell = row.insertCell(7);
        if (trade.screenshot) {
            const img = document.createElement("img");
            img.src = trade.screenshot;
            imgCell.appendChild(img);
        }
        row.insertCell(8).innerText = trade.risk ? (parseFloat(trade.profitLoss)/parseFloat(trade.risk)).toFixed(2) : "";
    });

    totalTradesEl.innerText = filteredTrades.length;
    totalGainEl.innerText = totalGain.toFixed(2);
    averageGainEl.innerText = filteredTrades.length ? (totalGain / filteredTrades.length).toFixed(2) : 0;
    totalRiskRewardEl.innerText = totalRisk ? (totalGain / totalRisk).toFixed(2) : 0;

    updateChart(filteredTrades);
}

// Handle form submit
form.addEventListener("submit", e => {
    e.preventDefault();
    const formData = new FormData(form);
    const screenshotFile = formData.get("screenshot");

    if (screenshotFile && screenshotFile.size > 0) {
        const reader = new FileReader();
        reader.onload = function(event) {
            addTradeToList(formData, event.target.result);
        };
        reader.readAsDataURL(screenshotFile);
    } else {
        addTradeToList(formData, null);
    }
});

function addTradeToList(formData, screenshotData) {
    const newTrade = {
        profitLoss: parseFloat(formData.get("profitLoss")),
        dailyGain: parseFloat(formData.get("dailyGain")) || 0,
        strategy: formData.get("strategy"),
        risk: parseFloat(formData.get("risk")) || 0,
        platform: formData.get("platform"),
        confidence: parseFloat(formData.get("confidence")) || 0,
        emotion: formData.get("emotion") || 5,
        screenshot: screenshotData
    };
    trades.push(newTrade);
    localStorage.setItem("trades", JSON.stringify(trades));
    form.reset();
    renderTrades();
}

// Filters
strategyFilter.addEventListener("change", renderTrades);
platformFilter.addEventListener("change", renderTrades);

// Chart.js setup
function updateChart(tradeList) {
    const labels = tradeList.map((t,i)=>`Trade ${i+1}`);
    const data = tradeList.map(t=>t.profitLoss);
    
    if (gainChart) gainChart.destroy();
    
    gainChart = new Chart(gainChartCtx, {
        type: 'line',
        data: {
            labels: labels,
            datasets: [{
                label: 'Profit/Loss',
                data: data,
                backgroundColor: 'rgba(16, 185, 129, 0.3)',
                borderColor: 'rgba(16, 185, 129, 1)',
                borderWidth: 2,
                fill: true,
                tension: 0.2
            }]
        },
        options: {
            responsive: true,
            plugins: { legend: { display: false } },
            scales: { y: { beginAtZero: true } }
        }
    });
}

// Initial render
renderTrades();
