<!DOCTYPE html>
<html lang="zh-Hant">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>小瞳學 Pro - 眼軸與近視追蹤</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
    <style>
        body { font-family: 'PingFang TC', 'Microsoft JhengHei', sans-serif; background-color: #f8fafc; }
        .glass-card { background: white; border-radius: 1rem; box-shadow: 0 4px 6px -1px rgb(0 0 0 / 0.1); }
        .tab-active { border-bottom: 3px solid #10b981; color: #059669; font-weight: bold; }
    </style>
</head>
<body class="p-4 md:p-6">

    <div class="max-w-3xl mx-auto">
        <header class="flex items-center justify-between mb-6">
            <div>
                <h1 class="text-2xl font-bold text-slate-800">👁️ 小瞳學專業版</h1>
                <p class="text-xs text-slate-500 italic">近視度數 & 眼軸長協同追蹤</p>
            </div>
            <button onclick="toggleModal()" class="bg-emerald-600 hover:bg-emerald-700 text-white px-5 py-2 rounded-xl shadow-md transition font-medium">
                + 錄入檢查數據
            </button>
        </header>

        <div class="glass-card p-4 mb-6">
            <div class="flex justify-between items-center mb-4">
                <h3 class="text-slate-700 font-bold">視力發展趨勢</h3>
                <div class="text-xs text-slate-400">藍線:度數 | 橙線:眼軸</div>
            </div>
            <div class="relative h-64 md:h-80">
                <canvas id="proChart"></canvas>
            </div>
        </div>

        <div id="summaryCards" class="grid grid-cols-2 md:grid-cols-4 gap-3 mb-6">
            </div>

        <div class="glass-card overflow-hidden">
            <div class="bg-slate-50 p-4 border-b">
                <h3 class="text-slate-700 font-bold text-sm uppercase tracking-wider">檢查歷史明細</h3>
            </div>
            <div id="historyList" class="divide-y divide-slate-100 max-h-96 overflow-y-auto">
                </div>
        </div>
    </div>

    <div id="modal" class="hidden fixed inset-0 bg-slate-900/60 backdrop-blur-sm flex items-center justify-center p-4 z-50">
        <div class="bg-white rounded-2xl p-6 w-full max-w-lg shadow-2xl">
            <div class="flex justify-between items-center mb-6">
                <h2 class="text-xl font-bold text-slate-800">新增視力檢查紀錄</h2>
                <button onclick="toggleModal()" class="text-slate-400">✕</button>
            </div>
            
            <form id="recordForm" class="space-y-5">
                <div>
                    <label class="block text-sm font-semibold text-slate-700 mb-1">檢查日期</label>
                    <input type="date" id="date" required class="w-full border-slate-200 border rounded-xl p-3 focus:ring-2 ring-emerald-500 outline-none">
                </div>

                <div class="grid grid-cols-2 gap-6">
                    <div class="space-y-3 p-3 bg-blue-50/50 rounded-xl">
                        <label class="block font-bold text-blue-700 border-b border-blue-100 pb-1 text-sm">左眼 (OS)</label>
                        <div>
                            <span class="text-xs text-slate-500 uppercase">近視度數</span>
                            <input type="number" step="0.25" id="leftSph" placeholder="-1.50" required class="w-full border rounded-lg p-2 mt-1">
                        </div>
                        <div>
                            <span class="text-xs text-slate-500 uppercase">眼軸長 (mm)</span>
                            <input type="number" step="0.01" id="leftAL" placeholder="23.50" class="w-full border rounded-lg p-2 mt-1">
                        </div>
                    </div>

                    <div class="space-y-3 p-3 bg-emerald-50/50 rounded-xl">
                        <label class="block font-bold text-emerald-700 border-b border-emerald-100 pb-1 text-sm">右眼 (OD)</label>
                        <div>
                            <span class="text-xs text-slate-500 uppercase">近視度數</span>
                            <input type="number" step="0.25" id="rightSph" placeholder="-1.25" required class="w-full border rounded-lg p-2 mt-1">
                        </div>
                        <div>
                            <span class="text-xs text-slate-500 uppercase">眼軸長 (mm)</span>
                            <input type="number" step="0.01" id="rightAL" placeholder="23.45" class="w-full border rounded-lg p-2 mt-1">
                        </div>
                    </div>
                </div>

                <div class="flex gap-3 pt-4">
                    <button type="button" onclick="toggleModal()" class="flex-1 py-3 text-slate-500 font-medium border border-slate-200 rounded-xl">取消</button>
                    <button type="submit" class="flex-1 py-3 bg-emerald-600 text-white rounded-xl font-bold hover:bg-emerald-700 shadow-lg shadow-emerald-200">儲存紀錄</button>
                </div>
            </form>
        </div>
    </div>

    <script>
        let myChart = null;
        let records = JSON.parse(localStorage.getItem('myopiaRecordsPro')) || [];

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
                lSph: parseFloat(document.getElementById('leftSph').value),
                rSph: parseFloat(document.getElementById('rightSph').value),
                lAL: parseFloat(document.getElementById('leftAL').value) || null,
                rAL: parseFloat(document.getElementById('rightAL').value) || null
            };
            
            records.push(newRecord);
            records.sort((a, b) => new Date(a.date) - new Date(b.date));
            localStorage.setItem('myopiaRecordsPro', JSON.stringify(records));
            
            toggleModal();
            updateUI();
            e.target.reset();
            document.getElementById('date').valueAsDate = new Date();
        };

        function deleteRecord(id) {
            if(confirm('確定要刪除這筆紀錄嗎？')) {
                records = records.filter(r => r.id !== id);
                localStorage.setItem('myopiaRecordsPro', JSON.stringify(records));
                updateUI();
            }
        }

        function updateUI() {
            renderSummary();
            renderList();
            renderChart();
        }

        function renderSummary() {
            const container = document.getElementById('summaryCards');
            if (records.length === 0) return;
            const last = records[records.length - 1];
            
            const items = [
                { label: '左眼度數', val: last.lSph.toFixed(2), color: 'text-blue-600' },
                { label: '右眼度數', val: last.rSph.toFixed(2), color: 'text-emerald-600' },
                { label: '左眼軸 (mm)', val: last.lAL ? last.lAL.toFixed(2) : '--', color: 'text-orange-600' },
                { label: '右眼軸 (mm)', val: last.rAL ? last.rAL.toFixed(2) : '--', color: 'text-orange-600' }
            ];

            container.innerHTML = items.map(i => `
                <div class="glass-card p-3 text-center">
                    <p class="text-[10px] text-slate-400 font-bold uppercase">${i.label}</p>
                    <p class="text-xl font-black ${i.color}">${i.val}</p>
                </div>
            `).join('');
        }

        function renderList() {
            const container = document.getElementById('historyList');
            if(records.length === 0) {
                container.innerHTML = `<div class="p-8 text-center text-slate-400">暫無檢查數據，請點擊新增</div>`;
                return;
            }
            container.innerHTML = records.slice().reverse().map(r => `
                <div class="flex items-center justify-between p-4 hover:bg-slate-50 transition">
                    <div class="flex-1">
                        <p class="font-bold text-slate-800">${r.date}</p>
                        <div class="flex gap-4 mt-1 text-xs">
                            <span class="bg-blue-50 text-blue-600 px-2 py-1 rounded">左眼: ${r.lSph.toFixed(2)} D / ${r.lAL || '--'} mm</span>
                            <span class="bg-emerald-50 text-emerald-600 px-2 py-1 rounded">右眼: ${r.rSph.toFixed(2)} D / ${r.rAL || '--'} mm</span>
                        </div>
                    </div>
                    <button onclick="deleteRecord(${r.id})" class="ml-4 p-2 text-slate-300 hover:text-red-500">🗑️</button>
                </div>
            `).join('');
        }

        function renderChart() {
            const ctx = document.getElementById('proChart').getContext('2d');
            if (myChart) myChart.destroy();

            const dates = records.map(r => r.date);

            myChart = new Chart(ctx, {
                data: {
                    labels: dates,
                    datasets: [
                        {
                            type: 'line',
                            label: '左眼度數 (L)',
                            data: records.map(r => r.lSph),
                            borderColor: '#3b82f6',
                            yAxisID: 'y'
                        },
                        {
                            type: 'line',
                            label: '右眼度數 (R)',
                            data: records.map(r => r.rSph),
                            borderColor: '#10b981',
                            yAxisID: 'y'
                        },
                        {
                            type: 'bar',
                            label: '左眼軸 (mm)',
                            data: records.map(r => r.lAL),
                            backgroundColor: 'rgba(249, 115, 22, 0.2)',
                            borderColor: '#f97316',
                            borderWidth: 1,
                            yAxisID: 'y1'
                        },
                        {
                            type: 'bar',
                            label: '右眼軸 (mm)',
                            data: records.map(r => r.rAL),
                            backgroundColor: 'rgba(234, 179, 8, 0.2)',
                            borderColor: '#eab308',
                            borderWidth: 1,
                            yAxisID: 'y1'
                        }
                    ]
                },
                options: {
                    responsive: true,
                    maintainAspectRatio: false,
                    scales: {
                        y: {
                            type: 'linear',
                            display: true,
                            position: 'left',
                            reverse: true,
                            title: { display: true, text: '度數 (D)', font: {size: 10} }
                        },
                        y1: {
                            type: 'linear',
                            display: true,
                            position: 'right',
                            grid: { drawOnChartArea: false },
                            min: 20, 
                            max: 28,
                            title: { display: true, text: '眼軸長 (mm)', font: {size: 10} }
                        }
                    },
                    plugins: {
                        legend: { labels: { boxWidth: 10, font: {size: 11} } }
                    }
                }
            });
        }
    </script>
</body>
</html>
