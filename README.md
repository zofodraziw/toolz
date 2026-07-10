<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Maker's Mixed-Media BoM Calculator</title>
    <style>
        :root {
            --bg-color: #0f172a;
            --card-bg: #1e293b;
            --text-main: #f8fafc;
            --text-muted: #94a3b8;
            --accent: #3b82f6;
            --accent-hover: #2563eb;
            --border: #334155;
            --success: #10b981;
            --section-bg: #0f172a;
        }

        * { box-sizing: border-box; font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif; }

        body {
            background-color: var(--bg-color);
            color: var(--text-main);
            padding: 15px;
            margin: 0;
            /* Removed flex and center justification to prevent left-side clipping on mobile */
        }

        .container {
            background-color: var(--card-bg);
            max-width: 900px;
            width: 100%;
            margin: 0 auto; /* Centers the container safely */
            border-radius: 12px;
            box-shadow: 0 10px 15px -3px rgba(0,0,0,0.5);
            padding: 30px;
        }

        h1 { text-align: center; color: var(--text-main); font-size: 26px; margin-bottom: 5px; }
        p.subtitle { text-align: center; color: var(--text-muted); margin-bottom: 25px; }

        .tabs { display: flex; gap: 10px; margin-bottom: 20px; overflow-x: auto; padding-bottom: 5px; }
        .tab-btn {
            flex: 1; padding: 12px; border: none; background: var(--section-bg);
            border-radius: 6px; cursor: pointer; font-weight: 600; color: var(--text-muted);
            transition: 0.2s; white-space: nowrap; border: 1px solid var(--border);
        }
        .tab-btn.active { background: var(--accent); color: white; border-color: var(--accent); }

        .grid { display: grid; grid-template-columns: 1fr 1fr; gap: 25px; }

        .section-box {
            background: var(--section-bg); border: 1px solid var(--border);
            border-radius: 8px; padding: 15px; margin-bottom: 15px;
        }

        h3 { font-size: 16px; margin-top: 0; color: var(--accent); border-bottom: 1px solid var(--border); padding-bottom: 8px; margin-bottom: 15px; }

        .input-group { display: flex; flex-direction: column; margin-bottom: 12px; width: 100%; }
        .input-row { display: flex; gap: 15px; }
        .input-row > div { flex: 1; }
        
        label { font-size: 13px; font-weight: 600; margin-bottom: 5px; color: var(--text-muted); }
        input[type="number"], input[type="range"] {
            padding: 8px; border: 1px solid var(--border); border-radius: 4px; font-size: 16px; /* 16px prevents iOS zoom bug */
            background: var(--card-bg); color: var(--text-main); width: 100%;
        }
        input[type="number"]:focus { outline: none; border-color: var(--accent); }
        
        .range-wrap { display: flex; align-items: center; gap: 15px; }
        .range-wrap span { font-weight: bold; width: 50px; color: var(--text-main); }

        .results {
            position: sticky; top: 20px;
            background: var(--section-bg); border: 1px solid var(--border);
            border-radius: 8px; padding: 20px;
        }
        .result-row { display: flex; justify-content: space-between; margin-bottom: 10px; font-size: 15px; }
        .result-row.sub { font-size: 13px; color: var(--text-muted); padding-left: 10px; margin-bottom: 5px; }
        .result-row.total { font-weight: bold; border-top: 1px solid var(--border); padding-top: 10px; margin-top: 5px; }
        .result-row.highlight { font-size: 22px; color: var(--success); font-weight: bold; margin-top: 15px; }

        /* Mobile Responsive Adjustments */
        @media (max-width: 768px) {
            .grid { grid-template-columns: 1fr; gap: 15px; }
            .container { padding: 15px; }
            h1 { font-size: 22px; }
            
            /* Stack the input rows vertically on mobile */
            .input-row { flex-direction: column; gap: 0; }
            
            .tabs { margin-bottom: 15px; }
            .results { position: static; margin-top: 20px; }
        }
    </style>
</head>
<body>

<div class="container">
    <h1>🚀 Mixed-Media BoM Calculator</h1>
    <p class="subtitle">Advanced costing for multi-material & hardware-integrated builds</p>

    <div class="tabs">
        <button class="tab-btn active" onclick="loadPreset('phoneStand')">📱 Tact. Phone Stand</button>
        <button class="tab-btn" onclick="loadPreset('badgeDuo')">⚔️ Mag-Badge Duo</button>
        <button class="tab-btn" onclick="loadPreset('headphoneStand')">🎧 Cyber Headphone Dock</button>
    </div>

    <div class="grid">
        <!-- Inputs Column -->
        <div>
            <div class="section-box">
                <h3>3D Printing (Dual Extrusion / Multi-Part)</h3>
                <div class="input-row">
                    <div class="input-group">
                        <label>Primary PLA/PETG ($/kg)</label>
                        <input type="number" id="fil1Cost" step="1" oninput="calculate()">
                    </div>
                    <div class="input-group">
                        <label>Weight (g)</label>
                        <input type="number" id="fil1Weight" step="1" oninput="calculate()">
                    </div>
                </div>
                <div class="input-row">
                    <div class="input-group">
                        <label>Accent TPU/Carbon ($/kg)</label>
                        <input type="number" id="fil2Cost" step="1" oninput="calculate()">
                    </div>
                    <div class="input-group">
                        <label>Weight (g)</label>
                        <input type="number" id="fil2Weight" step="1" oninput="calculate()">
                    </div>
                </div>
            </div>

            <div class="section-box">
                <h3>Laser Cutting (Mixed Media)</h3>
                <div class="input-row">
                    <div class="input-group">
                        <label>Acrylic Sheet ($)</label>
                        <input type="number" id="laser1Cost" step="0.5" oninput="calculate()">
                    </div>
                    <div class="input-group">
                        <label>Fraction Used (0-1)</label>
                        <input type="number" id="laser1Frac" step="0.05" oninput="calculate()">
                    </div>
                </div>
                <div class="input-row">
                    <div class="input-group">
                        <label>Wood/Plywood Sheet ($)</label>
                        <input type="number" id="laser2Cost" step="0.5" oninput="calculate()">
                    </div>
                    <div class="input-group">
                        <label>Fraction Used (0-1)</label>
                        <input type="number" id="laser2Frac" step="0.05" oninput="calculate()">
                    </div>
                </div>
            </div>

            <div class="section-box">
                <h3>Hardware & Assembly</h3>
                <div class="input-row">
                    <div class="input-group">
                        <label>Magnets ($ ea)</label>
                        <input type="number" id="magCost" step="0.05" oninput="calculate()">
                    </div>
                    <div class="input-group">
                        <label>Qty</label>
                        <input type="number" id="magQty" step="1" oninput="calculate()">
                    </div>
                </div>
                <div class="input-row">
                    <div class="input-group">
                        <label>LED Modules ($ ea)</label>
                        <input type="number" id="ledCost" step="0.5" oninput="calculate()">
                    </div>
                    <div class="input-group">
                        <label>Qty</label>
                        <input type="number" id="ledQty" step="1" oninput="calculate()">
                    </div>
                </div>
                <div class="input-group">
                    <label>Misc (Glue, Lanyards, Screws) ($)</label>
                    <input type="number" id="miscHardware" step="0.1" oninput="calculate()">
                </div>
            </div>

            <div class="section-box">
                <h3>Overhead & Pricing Strategy</h3>
                <div class="input-row">
                    <div class="input-group">
                        <label>Print Time (hrs)</label>
                        <input type="number" id="printTime" step="0.5" oninput="calculate()">
                    </div>
                    <div class="input-group">
                        <label>Laser Time (hrs)</label>
                        <input type="number" id="laserTime" step="0.1" oninput="calculate()">
                    </div>
                </div>
                <div class="input-group">
                    <label>Machine Rate ($/hr for power/wear)</label>
                    <input type="number" id="machineRate" step="0.1" oninput="calculate()">
                </div>
                <div class="input-group">
                    <label>Target Net Margin (%)</label>
                    <div class="range-wrap">
                        <input type="range" id="margin" min="10" max="90" value="60" oninput="updateMarginLabel()">
                        <span id="marginLabel">60%</span>
                    </div>
                </div>
            </div>
        </div>

        <!-- Outputs Column -->
        <div>
            <div class="results">
                <h3>Cost Breakdown</h3>
                
                <div class="result-row"><span>3D Print Materials:</span><span id="outPrintTotal">$0.00</span></div>
                <div class="result-row sub"><span>Primary:</span><span id="outFil1">$0.00</span></div>
                <div class="result-row sub"><span>Accent:</span><span id="outFil2">$0.00</span></div>
                
                <div class="result-row" style="margin-top:10px;"><span>Laser Materials:</span><span id="outLaserTotal">$0.00</span></div>
                <div class="result-row sub"><span>Acrylic:</span><span id="outLaser1">$0.00</span></div>
                <div class="result-row sub"><span>Wood:</span><span id="outLaser2">$0.00</span></div>

                <div class="result-row" style="margin-top:10px;"><span>Hardware & Misc:</span><span id="outHardTotal">$0.00</span></div>

                <div class="result-row total">
                    <span>Total Material Cost:</span>
                    <span id="outTotalMat">$0.00</span>
                </div>
                
                <br>
                <div class="result-row">
                    <span>Machine Overhead (Print + Laser):</span>
                    <span id="outOverhead">$0.00</span>
                </div>
                <div class="result-row total" style="color:var(--accent);">
                    <span>Prime Production Cost:</span>
                    <span id="outPrimeCost">$0.00</span>
                </div>

                <br><hr style="border: 0; border-top: 1px solid var(--border);"><br>

                <h3>Sales Projections</h3>
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
        phoneStand: { fil1Cost: 20, fil1Weight: 100, fil2Cost: 40, fil2Weight: 15, laser1Cost: 3, laser1Frac: 0, laser2Cost: 2, laser2Frac: 0.15, magCost: 0, magQty: 0, ledCost: 0, ledQty: 0, miscHardware: 0.5, printTime: 3.5, laserTime: 0.1, machineRate: 0.5, margin: 60 },
        badgeDuo: { fil1Cost: 22, fil1Weight: 45, fil2Cost: 35, fil2Weight: 10, laser1Cost: 4, laser1Frac: 0.2, laser2Cost: 0, laser2Frac: 0, magCost: 0.25, magQty: 4, ledCost: 0, ledQty: 0, miscHardware: 1.0, printTime: 1.5, laserTime: 0.2, machineRate: 0.5, margin: 70 },
        headphoneStand: { fil1Cost: 25, fil1Weight: 280, fil2Cost: 45, fil2Weight: 40, laser1Cost: 5, laser1Frac: 0.4, laser2Cost: 3, laser2Frac: 0.25, magCost: 0, magQty: 0, ledCost: 3.50, ledQty: 1, miscHardware: 2.0, printTime: 12, laserTime: 0.5, machineRate: 0.5, margin: 55 }
    };

    function loadPreset(key) {
        document.querySelectorAll('.tab-btn').forEach(btn => btn.classList.remove('active'));
        event.target.classList.add('active');

        const p = presets[key];
        document.getElementById('fil1Cost').value = p.fil1Cost;
        document.getElementById('fil1Weight').value = p.fil1Weight;
        document.getElementById('fil2Cost').value = p.fil2Cost;
        document.getElementById('fil2Weight').value = p.fil2Weight;
        document.getElementById('laser1Cost').value = p.laser1Cost;
        document.getElementById('laser1Frac').value = p.laser1Frac;
        document.getElementById('laser2Cost').value = p.laser2Cost;
        document.getElementById('laser2Frac').value = p.laser2Frac;
        
        document.getElementById('magCost').value = p.magCost;
        document.getElementById('magQty').value = p.magQty;
        document.getElementById('ledCost').value = p.ledCost;
        document.getElementById('ledQty').value = p.ledQty;
        document.getElementById('miscHardware').value = p.miscHardware;

        document.getElementById('printTime').value = p.printTime;
        document.getElementById('laserTime').value = p.laserTime;
        document.getElementById('machineRate').value = p.machineRate;
        document.getElementById('margin').value = p.margin;
        
        updateMarginLabel();
    }

    function updateMarginLabel() {
        document.getElementById('marginLabel').innerText = document.getElementById('margin').value + '%';
        calculate();
    }

    function calculate() {
        const f1C = parseFloat(document.getElementById('fil1Cost').value) || 0;
        const f1W = parseFloat(document.getElementById('fil1Weight').value) || 0;
        const f2C = parseFloat(document.getElementById('fil2Cost').value) || 0;
        const f2W = parseFloat(document.getElementById('fil2Weight').value) || 0;

        const l1C = parseFloat(document.getElementById('laser1Cost').value) || 0;
        const l1F = parseFloat(document.getElementById('laser1Frac').value) || 0;
        const l2C = parseFloat(document.getElementById('laser2Cost').value) || 0;
        const l2F = parseFloat(document.getElementById('laser2Frac').value) || 0;

        const mC = parseFloat(document.getElementById('magCost').value) || 0;
        const mQ = parseFloat(document.getElementById('magQty').value) || 0;
        const ledC = parseFloat(document.getElementById('ledCost').value) || 0;
        const ledQ = parseFloat(document.getElementById('ledQty').value) || 0;
        const misc = parseFloat(document.getElementById('miscHardware').value) || 0;

        const pTime = parseFloat(document.getElementById('printTime').value) || 0;
        const lTime = parseFloat(document.getElementById('laserTime').value) || 0;
        const mRate = parseFloat(document.getElementById('machineRate').value) || 0;
        const margin = parseFloat(document.getElementById('margin').value) || 0;

        const costF1 = (f1W / 1000) * f1C;
        const costF2 = (f2W / 1000) * f2C;
        const totalPrint = costF1 + costF2;

        const costL1 = l1C * l1F;
        const costL2 = l2C * l2F;
        const totalLaser = costL1 + costL2;

        const totalHard = (mC * mQ) + (ledC * ledQ) + misc;
        const totalMat = totalPrint + totalLaser + totalHard;

        const overhead = (pTime + lTime) * mRate;
        const primeCost = totalMat + overhead;
        
        const retailPrice = primeCost / (1 - (margin / 100));
        const profit = retailPrice - primeCost;

        const format = (num) => '$' + num.toFixed(2);

        document.getElementById('outFil1').innerText = format(costF1);
        document.getElementById('outFil2').innerText = format(costF2);
        document.getElementById('outPrintTotal').innerText = format(totalPrint);

        document.getElementById('outLaser1').innerText = format(costL1);
        document.getElementById('outLaser2').innerText = format(costL2);
        document.getElementById('outLaserTotal').innerText = format(totalLaser);

        document.getElementById('outHardTotal').innerText = format(totalHard);
        document.getElementById('outTotalMat').innerText = format(totalMat);
        document.getElementById('outOverhead').innerText = format(overhead);
        document.getElementById('outPrimeCost').innerText = format(primeCost);
        
        document.getElementById('outProfit').innerText = format(profit);
        document.getElementById('outRetail').innerText = format(retailPrice);
    }

    window.onload = () => loadPreset('phoneStand');
</script>

</body>
</html>
