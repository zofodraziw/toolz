<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Maker's BoM & Cost Calculator</title>
    <style>
        :root {
            --bg-color: #f4f4f9;
            --card-bg: #ffffff;
            --text-main: #333;
            --accent: #4f46e5;
            --accent-hover: #4338ca;
            --border: #e5e7eb;
            --success: #10b981;
        }

        * { box-sizing: border-box; font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif; }

        body {
            background-color: var(--bg-color);
            color: var(--text-main);
            padding: 20px;
            display: flex;
            justify-content: center;
        }

        .container {
            background-color: var(--card-bg);
            max-width: 800px;
            width: 100%;
            border-radius: 12px;
            box-shadow: 0 4px 6px rgba(0,0,0,0.1);
            padding: 30px;
        }

        h1 { text-align: center; color: var(--text-main); font-size: 24px; margin-bottom: 5px; }
        p.subtitle { text-align: center; color: #6b7280; margin-bottom: 25px; }

        .tabs { display: flex; gap: 10px; margin-bottom: 20px; }
        .tab-btn {
            flex: 1; padding: 10px; border: none; background: #f3f4f6;
            border-radius: 6px; cursor: pointer; font-weight: 600; color: #4b5563;
            transition: 0.2s;
        }
        .tab-btn.active { background: var(--accent); color: white; }

        .grid { display: grid; grid-template-columns: 1fr 1fr; gap: 20px; }
        @media (max-width: 600px) { .grid { grid-template-columns: 1fr; } }

        .input-group { display: flex; flex-direction: column; margin-bottom: 15px; }
        label { font-size: 14px; font-weight: 600; margin-bottom: 5px; color: #374151; }
        input[type="number"], input[type="range"] {
            padding: 10px; border: 1px solid var(--border); border-radius: 6px; font-size: 16px;
        }
        .range-wrap { display: flex; align-items: center; gap: 15px; }
        .range-wrap span { font-weight: bold; width: 50px; }

        .results {
            background: #f8fafc; border: 1px solid var(--border);
            border-radius: 8px; padding: 20px; margin-top: 20px;
        }
        .result-row { display: flex; justify-content: space-between; margin-bottom: 10px; font-size: 16px; }
        .result-row.total { font-weight: bold; border-top: 1px solid var(--border); padding-top: 10px; }
        .result-row.highlight { font-size: 20px; color: var(--success); font-weight: bold; margin-top: 15px; }

    </style>
</head>
<body>

<div class="container">
    <h1>🚀 Maker's BoM & Cost Calculator</h1>
    <p class="subtitle">Calculate actual costs and set retail prices</p>

    <div class="tabs">
        <button class="tab-btn active" onclick="loadPreset('phoneStand')">📱 Phase 1: Phone Stand</button>
        <button class="tab-btn" onclick="loadPreset('badgeDuo')">⚔️ Phase 2: Badge Duo</button>
        <button class="tab-btn" onclick="loadPreset('headphoneStand')">🎧 Phase 3: Headphone Stand</button>
    </div>

    <div class="grid">
        <div>
            <h3>Material Inputs</h3>
            <div class="input-group">
                <label>Filament Spool Cost ($ per kg)</label>
                <input type="number" id="spoolCost" step="1" oninput="calculate()">
            </div>
            <div class="input-group">
                <label>Filament Weight Used (grams)</label>
                <input type="number" id="weight" step="1" oninput="calculate()">
            </div>
            <div class="input-group">
                <label>Laser Material Sheet Cost ($)</label>
                <input type="number" id="sheetCost" step="0.5" oninput="calculate()">
            </div>
            <div class="input-group">
                <label>Fraction of Sheet Used (0.0 to 1.0)</label>
                <input type="number" id="sheetFraction" step="0.05" oninput="calculate()">
            </div>
            <div class="input-group">
                <label>Hardware Add-ons (Magnets, LEDs, etc.) ($)</label>
                <input type="number" id="hardware" step="0.1" oninput="calculate()">
            </div>

            <h3 style="margin-top:20px;">Overhead & Profit</h3>
            <div class="input-group">
                <label>Print Time (Hours)</label>
                <input type="number" id="time" step="0.5" oninput="calculate()">
            </div>
            <div class="input-group">
                <label>Machine Wear/Electricity ($ per Hour)</label>
                <input type="number" id="rate" step="0.1" oninput="calculate()">
            </div>
            <div class="input-group">
                <label>Target Profit Margin (%)</label>
                <div class="range-wrap">
                    <input type="range" id="margin" min="10" max="90" value="50" oninput="updateMarginLabel()">
                    <span id="marginLabel">50%</span>
                </div>
            </div>
        </div>

        <div>
            <div class="results">
                <h3 style="margin-top:0;">Cost Breakdown</h3>
                <div class="result-row">
                    <span>3D Print Material:</span>
                    <span id="outPrintMat">$0.00</span>
                </div>
                <div class="result-row">
                    <span>Laser Material:</span>
                    <span id="outLaserMat">$0.00</span>
                </div>
                <div class="result-row">
                    <span>Hardware:</span>
                    <span id="outHardware">$0.00</span>
                </div>
                <div class="result-row total">
                    <span>Total Material Cost:</span>
                    <span id="outTotalMat">$0.00</span>
                </div>
                
                <br>
                <div class="result-row">
                    <span>Machine Overhead:</span>
                    <span id="outOverhead">$0.00</span>
                </div>
                <div class="result-row total" style="color:var(--accent);">
                    <span>Prime Production Cost:</span>
                    <span id="outPrimeCost">$0.00</span>
                </div>

                <br><hr style="border: 0; border-top: 1px solid var(--border);"><br>

                <h3 style="margin-top:0;">Sales Projections</h3>
                <div class="result-row">
                    <span>Target Profit (Net):</span>
                    <span id="outProfit">$0.00</span>
                </div>
                <div class="result-row highlight">
                    <span>Suggested Retail Price:</span>
                    <span id="outRetail">$0.00</span>
                </div>
            </div>
        </div>
    </div>
</div>

<script>
    const presets = {
        phoneStand: { spoolCost: 25, weight: 120, sheetCost: 3, sheetFraction: 0.1, hardware: 0, time: 3, rate: 0.5, margin: 60 },
        badgeDuo: { spoolCost: 40, weight: 65, sheetCost: 3, sheetFraction: 0.2, hardware: 1.5, time: 2, rate: 0.5, margin: 70 },
        headphoneStand: { spoolCost: 25, weight: 350, sheetCost: 5, sheetFraction: 0.5, hardware: 4.5, time: 9, rate: 0.5, margin: 55 }
    };

    function loadPreset(key) {
        document.querySelectorAll('.tab-btn').forEach(btn => btn.classList.remove('active'));
        event.target.classList.add('active');

        const p = presets[key];
        document.getElementById('spoolCost').value = p.spoolCost;
        document.getElementById('weight').value = p.weight;
        document.getElementById('sheetCost').value = p.sheetCost;
        document.getElementById('sheetFraction').value = p.sheetFraction;
        document.getElementById('hardware').value = p.hardware;
        document.getElementById('time').value = p.time;
        document.getElementById('rate').value = p.rate;
        document.getElementById('margin').value = p.margin;
        
        updateMarginLabel();
    }

    function updateMarginLabel() {
        document.getElementById('marginLabel').innerText = document.getElementById('margin').value + '%';
        calculate();
    }

    function calculate() {
        const spoolCost = parseFloat(document.getElementById('spoolCost').value) || 0;
        const weight = parseFloat(document.getElementById('weight').value) || 0;
        const sheetCost = parseFloat(document.getElementById('sheetCost').value) || 0;
        const sheetFraction = parseFloat(document.getElementById('sheetFraction').value) || 0;
        const hardware = parseFloat(document.getElementById('hardware').value) || 0;
        const time = parseFloat(document.getElementById('time').value) || 0;
        const rate = parseFloat(document.getElementById('rate').value) || 0;
        const margin = parseFloat(document.getElementById('margin').value) || 0;

        const printCost = (weight / 1000) * spoolCost;
        const laserCost = sheetCost * sheetFraction;
        const totalMat = printCost + laserCost + hardware;
        const overhead = time * rate;
        const primeCost = totalMat + overhead;
        
        // Price calculation based on desired margin percentage
        const retailPrice = primeCost / (1 - (margin / 100));
        const profit = retailPrice - primeCost;

        const format = (num) => '$' + num.toFixed(2);

        document.getElementById('outPrintMat').innerText = format(printCost);
        document.getElementById('outLaserMat').innerText = format(laserCost);
        document.getElementById('outHardware').innerText = format(hardware);
        document.getElementById('outTotalMat').innerText = format(totalMat);
        document.getElementById('outOverhead').innerText = format(overhead);
        document.getElementById('outPrimeCost').innerText = format(primeCost);
        document.getElementById('outProfit').innerText = format(profit);
        document.getElementById('outRetail').innerText = format(retailPrice);
    }

    // Initialize with Phone Stand
    window.onload = () => loadPreset('phoneStand');
</script>

</body>
</html>
