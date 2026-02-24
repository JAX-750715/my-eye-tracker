<!DOCTYPE html>
<html lang="zh-Hant">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>小瞳學 Pro+ 完整追蹤</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
    <style>
        body { font-family: 'PingFang TC', 'Microsoft JhengHei', sans-serif; background-color: #f8fafc; }
        .glass-card { background: white; border-radius: 1rem; box-shadow: 0 4px 6px -1px rgb(0 0 0 / 0.1); }
        input::-webkit-outer-spin-button, input::-webkit-inner-spin-button { -webkit-appearance: none; margin: 0; }
    </style>
</head>
<body class="p-4 md:p-6 text-slate-800">

    <div class="max-w-3xl mx-auto">
        <header class="flex items-center justify-between mb-6">
            <div>
                <h1 class="text-2xl font-bold text-slate-800">👁️ 小瞳學 Pro+</h1>
                <p class="text-xs text-slate-500 italic">全方位視力數據庫 (度數/散光/眼軸)</p>
            </div>
            <button onclick="toggleModal()" class="bg-indigo-600 hover:bg-indigo-700 text-white px-5 py-2 rounded-xl shadow-md transition font-medium">
                + 錄入新紀錄
            </button>
        </header>

        <div class="glass-card p-4 mb-6">
            <div class="flex justify-between items-center mb-4 px-2">
                <h3 class="font-bold text-slate-600">視力發展趨勢</h3>
                <div class="flex gap-3 text-[10px] font-bold">
                    <span class="flex items-center text-blue-500"><span class="w-2 h-2 bg-blue-500 rounded-full mr-1"></span>度數</span>
                    <span class="flex items-center text-orange-500"><span class="w-2 h-2 bg-orange-500 rounded-full mr-1"></span>眼軸</span>
                </div>
            </div>
            <div class="relative h-64 md:h-80">
                <canvas id="proChart"></canvas>
            </div>
        </div>

        <div class="glass-card overflow-hidden">
            <div class="bg-slate-50 p-4 border-b flex justify-between items-center">
                <h3 class="font-bold text-sm uppercase tracking-wider text-slate-500">歷史檢查清單</h3>
                <span id="recordCount" class="text-xs bg-slate-200 px-2 py-0.5 rounded-full text-slate-600">0 筆</span>
            </div>
            <div id="historyList" class="divide-y divide-slate-100 max-h-96 overflow-y-auto">
                </div>
        </div>
    </div>

    <div id="modal" class="hidden fixed inset-0 bg-slate-900/60 backdrop-blur-sm flex items-center justify-center p-4 z-50">
        <div class="bg-white rounded-2xl p-6 w-full max-w-lg shadow-2xl overflow-y-auto max-h-[90vh]">
            <div class="flex justify-between items-center mb-6">
                <h2 class="text-xl font-bold">新增檢查紀錄</h2>
                <button onclick="toggleModal()" class="text-slate-400 p-2">✕</button>
            </div>
            
            <form id="recordForm" class="space-y-6">
                <div>
                    <label class="block text-sm font-semibold mb-1 text-slate-600">檢查日期</label>
                    <input type="date" id="date" required class="w-full border-slate-200 border rounded-xl p-3 focus:ring-2 ring-indigo-500 outline-none">
                </div>

                <div class="p-4 bg-blue-50/50 rounded-2xl border border-blue-100">
                    <label class="block font-black text-blue-700 mb-3 flex items-center">
                        <span class="w-2 h-5 bg-blue-500 rounded-full mr-2"></span>左眼 (OS)
                    </label>
                    <div class="grid grid-cols-3 gap-3">
                        <div>
                            <span class="text-[10px] font-bold text-slate-400 uppercase">近視 SPH</span>
                            <input type="number" step="0.25" id="lSph" placeholder="-1.50" required class="w-full border rounded-lg p-2 mt-1">
                        </div>
                        <div>
                            <span class="text-[10px] font-bold text-slate-400 uppercase">散光 CYL</span>
                            <input type="number" step="0.25" id="lCyl" placeholder="-0.50" class="w-full border rounded-lg p-2 mt-1">
                        </div>
                        <div>
                            <span class="text-[10px] font-bold text-slate-400 uppercase">眼軸 AL</span>
                            <input type="number" step="0.01" id="lAL" placeholder="mm" class="w-full border rounded-lg p-2 mt-1">
                        </div>
                    </div>
                </div>

                <div class="p-4 bg-indigo-50/50 rounded-2xl border border-indigo-100">
                    <label class="block font-black text-indigo-700 mb-3 flex items-center">
                        <span class="w-2 h-5 bg-indigo-500 rounded-full mr-2"></span>右眼 (OD)
                    </label>
                    <div class="grid grid-cols-3 gap-3">
                        <div>
                            <span class="text-[10px] font-bold text-slate-400 uppercase">近視 SPH</span>
                            <input type="number" step="0.25" id="rSph" placeholder="-1.25" required class="w-full border rounded-lg p-2 mt-1">
                        </div>
                        <div>
                            <span class="text-[10px] font-bold text-slate-400 uppercase">散光 CYL</span>
                            <input type="number" step="0.25" id="rCyl" placeholder="-0.25" class="w-full border rounded-lg p-2 mt-1">
                        </div>
                        <div>
                            <span class="text-[10px] font-bold text-slate-400 uppercase">眼軸 AL</span>
                            <input type="number" step="0.01" id="rAL" placeholder="mm" class="w-full border rounded-lg p-2 mt-1">
                        </div>
                    </div>
                </div>

                <div class="flex gap-3 pt-2">
                    <button type="button" onclick="toggleModal()" class="flex-1 py-3 text-slate-500 font-medium">取消</button>
                    <button type="submit" class="flex-1 py-3 bg-indigo-600 text-white rounded-xl font-bold hover:bg-indigo-700 shadow-lg shadow-indigo-100">儲存紀錄</button>
                </div>
            </form>
        </div>
    </div>

    <script>
        let myChart = null;
        let records = JSON.parse(localStorage.getItem('myopiaRecordsPlus')) || [];

        window.onload = () => {
            document.getElementById('date').valueAsDate = new Date();
            updateUI();
        };

        function toggleModal() {
            document.getElementById('modal').classList.toggle('hidden');
        }

        document.getElementById('recordForm').onsubmit = (e) => {
            e.preventDefault();
            const newRecord = {
                id: Date.now(),
                date: document.getElementById('date').value,
                lSph: parseFloat(document.getElementById('lSph').value),
                lCyl: parseFloat(document.getElementById('lCyl').value) || 0,
                lAL: parseFloat(document.getElementById('lAL').value) || null,
                rSph: parseFloat(document.getElementById('rSph').value),
                rCyl: parseFloat(document.getElementById('rCyl').value) || 0,
                rAL: parseFloat(document.getElementById('rAL').value) || null
            };
            
            records.push(newRecord);
            records.sort((a, b) => new Date(a.date) - new Date(b.date));
            localStorage.setItem('myopiaRecordsPlus', JSON.stringify(records));
            
            toggleModal();
            updateUI();
            e.target.reset();
            document.getElementById('date').valueAsDate = new Date();
        };

        function deleteRecord(id) {
            if(confirm('確定要刪除這筆紀錄嗎？')) {
                records = records.filter(r => r.id !== id);
                localStorage.setItem('myopiaRecordsPlus', JSON.stringify(records));
                updateUI();
            }
        }

        function updateUI() {
            renderList();
            renderChart();
            document.getElementById('recordCount').innerText = `${records.length} 筆`;
        }

        function renderList() {
            const container = document.getElementById('historyList');
            if(records.length === 0) {
                container.innerHTML = `<div class="p-12 text-center text-slate-300">目前尚無紀錄</div>`;
                return;
            }
            container.innerHTML = records.slice().reverse().map(r => `
                <div class="p-4 hover:bg-slate-50 flex items-center">
                    <div class="flex-1">
                        <div class="text-sm font-bold text-slate-800">${r.date}</div>
                        <div class="grid grid-cols-2 gap-4 mt-2">
                            <div class="text-[11px]">
                                <span class="text-blue-600 font-bold mr-1">L:</span> 
                                <span class="text-slate-700">度 ${r.lSph.toFixed(2)} / 散 ${r.lCyl.toFixed(2)}</span>
                                ${r.lAL ? `<span class="ml-2 text-orange-600">軸 ${r.lAL}mm</span>` : ''}
                            </div>
                            <div class="text-[11px]">
                                <span class="text-indigo-600 font-bold mr-1">R:</span> 
                                <span class="text-slate-700">度 ${r.rSph.toFixed(2)} / 散 ${r.rCyl.toFixed(2)}</span>
                                ${r.rAL ? `<span class="ml-2 text-orange-600">軸 ${r.rAL}mm</span>` : ''}
                            </div>
                        </div>
                    </div>
                    <button onclick="deleteRecord(${r.id})" class="text-slate-300 hover:text-red-400 px-2">✕</button>
                </div>
            `).join('');
        }

        function renderChart() {
            const ctx = document.getElementById('proChart').getContext('2d');
            if (myChart) myChart.destroy();

            myChart = new Chart(ctx, {
                data: {
                    labels: records.map(r => r.date),
                    datasets: [
                        {
                            type: 'line',
                            label: '左度數',
                            data: records.map(r => r.lSph),
                            borderColor: '#3b82f6',
                            tension: 0.2,
                            yAxisID: 'y'
                        },
                        {
                            type: 'line',
                            label: '右度數',
                            data: records.map(r => r.rSph),
                            borderColor: '#6366f1',
                            tension: 0.2,
                            yAxisID: 'y'
                        },
                        {
                            type: 'bar',
                            label: '眼軸',
                            data: records.map(r => r.lAL || r.rAL), // 取平均或其一顯示
                            backgroundColor: 'rgba(249, 115, 22, 0.1)',
                            yAxisID: 'y1'
                        }
                    ]
                },
                options: {
                    responsive: true,
                    maintainAspectRatio: false,
                    scales: {
                        y: { reverse: true, position: 'left', title: { display: true, text: '度數 (D)' } },
                        y1: { position: 'right', min: 20, max: 28, grid: { drawOnChartArea: false }, title: { display: true, text: '眼軸 (mm)' } }
                    },
                    plugins: { legend: { display: false } }
                }
            });
        }
    </script>
</body>
</html>
