<!DOCTYPE html>
<html lang="id">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>PalmLogistik Lokal - Manajemen RAM TBS</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <script src="https://unpkg.com/lucide@latest"></script>
    <!-- Pustaka SheetJS untuk Export Excel -->
    <script src="https://cdnjs.cloudflare.com/ajax/libs/xlsx/0.18.5/xlsx.full.min.js"></script>
    <style>
        @import url('https://fonts.googleapis.com/css2?family=Plus+Jakarta+Sans:wght@400;600;700;800&display=swap');
        body { font-family: 'Plus Jakarta Sans', sans-serif; }
        .custom-scrollbar::-webkit-scrollbar { width: 4px; }
        .custom-scrollbar::-webkit-scrollbar-thumb { background: #cbd5e1; border-radius: 10px; }
        .tab-content { animation: fadeIn 0.3s ease-in-out; }
        @keyframes fadeIn { from { opacity: 0; transform: translateY(10px); } to { opacity: 1; transform: translateY(0); } }
    </style>
</head>
<body class="bg-slate-50 text-slate-900 pb-20 md:pb-0">

    <div id="app" class="flex flex-col md:flex-row h-screen overflow-hidden">
        
        <!-- Sidebar (Desktop) -->
        <aside class="hidden md:flex flex-col w-72 bg-white border-r border-slate-200 p-6">
            <div class="flex items-center space-x-3 mb-10 text-emerald-600">
                <div class="w-10 h-10 bg-emerald-600 rounded-xl flex items-center justify-center shadow-lg shadow-emerald-200">
                    <i data-lucide="truck" class="text-white"></i>
                </div>
                <h1 class="text-xl font-black tracking-tighter uppercase">PalmLogistik</h1>
            </div>
            
            <nav class="space-y-1 flex-1" id="main-nav">
                <button onclick="changeTab('dashboard')" class="nav-btn w-full flex items-center space-x-3 px-4 py-3 rounded-xl transition-all active-nav bg-emerald-600 text-white shadow-lg" data-tab="dashboard">
                    <i data-lucide="layout-dashboard"></i>
                    <span>Dashboard KPI</span>
                </button>
                <button onclick="changeTab('history')" class="nav-btn w-full flex items-center space-x-3 px-4 py-3 rounded-xl transition-all text-slate-400 hover:bg-slate-50" data-tab="history">
                    <i data-lucide="history"></i>
                    <span>Buku Timbang</span>
                </button>
                <button onclick="changeTab('expenses')" class="nav-btn w-full flex items-center space-x-3 px-4 py-3 rounded-xl transition-all text-slate-400 hover:bg-slate-50" data-tab="expenses">
                    <i data-lucide="fuel"></i>
                    <span>Biaya Ops</span>
                </button>
                <button onclick="changeTab('report')" class="nav-btn w-full flex items-center space-x-3 px-4 py-3 rounded-xl transition-all text-slate-400 hover:bg-slate-50" data-tab="report">
                    <i data-lucide="receipt-text"></i>
                    <span>Laba Rugi</span>
                </button>
            </nav>

            <div class="mt-auto bg-slate-900 rounded-2xl p-5 text-white">
                <p class="text-[10px] opacity-60 uppercase font-black mb-1">Stok TBS di RAM (Lokal)</p>
                <h4 class="text-xl font-bold" id="sidebar-stock">0 KG</h4>
            </div>
        </aside>

        <!-- Main Content -->
        <main class="flex-1 overflow-y-auto p-4 md:p-8" id="content-area">
            
            <!-- Tab: Dashboard -->
            <div id="tab-dashboard" class="tab-content max-w-5xl mx-auto space-y-8">
                <header class="flex justify-between items-start">
                    <div>
                        <h2 class="text-3xl font-black text-slate-800 tracking-tight">Performa RAM</h2>
                        <p class="text-slate-500 font-medium italic">Data tersimpan di perangkat ini</p>
                    </div>
                    <button onclick="exportJSON()" class="flex items-center gap-2 px-4 py-2 bg-white border border-slate-200 rounded-xl text-xs font-bold text-slate-600 hover:bg-slate-50 transition-all">
                        <i data-lucide="download" size="14"></i> Backup Data (JSON)
                    </button>
                </header>

                <div class="grid grid-cols-1 md:grid-cols-4 gap-4">
                    <div class="bg-white p-5 rounded-2xl border border-slate-100 shadow-sm">
                        <p class="text-[10px] font-black text-slate-400 uppercase mb-2">TBS Masuk</p>
                        <p class="text-xl font-bold text-slate-800" id="stat-tonase-in">0 <span class="text-xs font-normal">KG</span></p>
                    </div>
                    <div class="bg-white p-5 rounded-2xl border border-slate-100 shadow-sm">
                        <p class="text-[10px] font-black text-slate-400 uppercase mb-2">TBS Berangkat</p>
                        <p class="text-xl font-bold text-slate-800" id="stat-tonase-out">0 <span class="text-xs font-normal">KG</span></p>
                    </div>
                    <div class="bg-white p-5 rounded-2xl border border-slate-100 shadow-sm">
                        <p class="text-[10px] font-black text-amber-600 uppercase mb-2 flex items-center gap-1"><i data-lucide="boxes" size="12"></i> Stok Saat Ini</p>
                        <p class="text-xl font-bold text-slate-800" id="stat-stock">0 <span class="text-xs font-normal">KG</span></p>
                    </div>
                    <div class="bg-emerald-600 p-5 rounded-2xl shadow-lg shadow-emerald-100 text-white">
                        <p class="text-[10px] font-black opacity-70 uppercase mb-2">Laba Estimasi</p>
                        <p class="text-xl font-bold" id="stat-net-profit">Rp 0</p>
                    </div>
                </div>

                <div class="grid grid-cols-1 lg:grid-cols-2 gap-8">
                    <div class="bg-white p-6 rounded-3xl shadow-sm border border-slate-100">
                        <div class="flex items-center justify-between mb-6">
                            <h3 class="text-lg font-bold flex items-center gap-2"><i data-lucide="scale" class="text-emerald-600"></i> Input Timbangan</h3>
                            <div class="flex bg-slate-100 p-1 rounded-xl" id="trans-type-toggle">
                                <button onclick="setTransType('pembelian')" id="btn-pembelian" class="px-4 py-1.5 rounded-lg text-[10px] font-black uppercase transition-all bg-white text-emerald-600 shadow-sm">Beli</button>
                                <button onclick="setTransType('penjualan')" id="btn-penjualan" class="px-4 py-1.5 rounded-lg text-[10px] font-black uppercase transition-all text-slate-400">Jual</button>
                            </div>
                        </div>

                        <form id="trans-form" class="space-y-4">
                            <!-- Input Tanggal Manual -->
                            <div class="space-y-1">
                                <label class="text-[10px] font-black uppercase text-slate-400 ml-1">Tanggal Transaksi</label>
                                <input required type="date" id="transDate" class="w-full px-4 py-3 bg-slate-50 border border-slate-200 rounded-xl outline-none focus:border-emerald-500" />
                            </div>
                            
                            <input required type="text" id="partyName" placeholder="Nama Petani / PKS" class="w-full px-4 py-3 bg-slate-50 border border-slate-200 rounded-xl outline-none focus:border-emerald-500" />
                            <div class="grid grid-cols-2 gap-4">
                                <input required type="number" id="grossWeight" placeholder="Bruto (KG)" oninput="recalculate()" class="w-full px-4 py-3 bg-slate-50 border border-slate-200 rounded-xl outline-none" />
                                <input type="number" id="deduction" placeholder="Potongan %" oninput="recalculate()" class="w-full px-4 py-3 bg-slate-50 border border-slate-200 rounded-xl outline-none" />
                            </div>
                            <input required type="number" id="price" placeholder="Harga/KG" oninput="recalculate()" class="w-full px-4 py-3 bg-slate-50 border border-slate-200 rounded-xl outline-none" />
                            <div class="bg-emerald-50 p-4 rounded-2xl text-emerald-800 text-sm font-bold flex justify-between">
                                <span>Total Estimasi:</span>
                                <span id="calc-preview">Rp 0</span>
                            </div>
                            <button type="submit" class="w-full py-4 rounded-2xl font-black text-white shadow-lg bg-emerald-600 hover:bg-emerald-700 transition-all">SIMPAN KE PERANGKAT</button>
                        </form>
                    </div>

                    <div class="bg-white p-6 rounded-3xl shadow-sm border border-slate-100">
                        <h3 class="text-lg font-bold mb-6 flex items-center gap-2"><i data-lucide="history" class="text-slate-400"></i> Transaksi Terakhir</h3>
                        <div id="recent-activities" class="space-y-4 max-h-[500px] overflow-y-auto pr-2 custom-scrollbar">
                            <!-- JS Injection -->
                        </div>
                    </div>
                </div>
            </div>

            <!-- Tab: History -->
            <div id="tab-history" class="tab-content hidden max-w-5xl mx-auto space-y-6">
                <h2 class="text-2xl font-black">Buku Timbang</h2>
                <div class="bg-white rounded-3xl border border-slate-100 shadow-sm overflow-hidden">
                    <div class="overflow-x-auto">
                        <table class="w-full text-left">
                            <thead class="bg-slate-50 text-[10px] font-black text-slate-400 uppercase border-b">
                                <tr>
                                    <th class="px-6 py-4">Tanggal</th>
                                    <th class="px-6 py-4">Tipe</th>
                                    <th class="px-6 py-4">Pihak</th>
                                    <th class="px-6 py-4">Bersih</th>
                                    <th class="px-6 py-4 text-right">Total</th>
                                    <th class="px-6 py-4 text-center">Aksi</th>
                                </tr>
                            </thead>
                            <tbody id="history-table-body" class="divide-y divide-slate-100"></tbody>
                        </table>
                    </div>
                </div>
            </div>

            <!-- Tab: Expenses -->
            <div id="tab-expenses" class="tab-content hidden max-w-3xl mx-auto space-y-6">
                <h2 class="text-2xl font-black">Biaya Operasional</h2>
                <form id="exp-form" class="bg-white p-6 rounded-3xl border border-slate-100 shadow-sm space-y-4">
                    <div class="grid grid-cols-1 md:grid-cols-2 gap-4">
                        <div class="space-y-1">
                            <label class="text-[10px] font-black uppercase text-slate-400 ml-1">Tanggal Biaya</label>
                            <input required type="date" id="expDate" class="w-full px-4 py-3 bg-slate-50 border border-slate-200 rounded-xl outline-none" />
                        </div>
                        <div class="space-y-1">
                            <label class="text-[10px] font-black uppercase text-slate-400 ml-1">Keterangan</label>
                            <input required type="text" id="expNote" placeholder="Solar, Gaji, dll" class="w-full px-4 py-3 bg-slate-50 border border-slate-200 rounded-xl outline-none" />
                        </div>
                    </div>
                    <div class="flex flex-col md:flex-row gap-4">
                        <input required type="number" id="expAmount" placeholder="Jumlah (Rp)" class="flex-1 px-4 py-3 bg-slate-50 border border-slate-200 rounded-xl outline-none" />
                        <button class="bg-slate-900 text-white px-8 py-3 rounded-xl font-bold">Tambah Biaya</button>
                    </div>
                </form>
                <div class="bg-white rounded-3xl border border-slate-100 shadow-sm overflow-hidden">
                    <table class="w-full text-left">
                        <thead class="bg-slate-50 text-[10px] font-black text-slate-400 uppercase border-b">
                            <tr>
                                <th class="px-6 py-4">Tanggal</th>
                                <th class="px-6 py-4">Keterangan</th>
                                <th class="px-6 py-4 text-right">Jumlah</th>
                                <th class="px-6 py-4 text-center">Hapus</th>
                            </tr>
                        </thead>
                        <tbody id="expense-table-body" class="divide-y divide-slate-100"></tbody>
                    </table>
                </div>
            </div>

            <!-- Tab: Report -->
            <div id="tab-report" class="tab-content hidden max-w-4xl mx-auto space-y-8">
                <div class="flex justify-between items-center">
                    <h2 class="text-2xl font-black">Laporan Keuangan</h2>
                    <button onclick="downloadExcel()" class="flex items-center gap-2 bg-emerald-600 text-white px-6 py-2 rounded-xl font-bold shadow-lg hover:bg-emerald-700 transition-all">
                        <i data-lucide="file-spreadsheet" size="18"></i> Download Excel
                    </button>
                </div>
                <div class="bg-white rounded-[2rem] shadow-xl border border-slate-100 p-8 space-y-8">
                    <div class="text-center border-b pb-6">
                        <h2 class="text-3xl font-black uppercase tracking-tighter text-emerald-600">PalmLogistik</h2>
                        <p class="text-slate-500 font-bold">Ringkasan Laba Rugi Lokal</p>
                        <p class="text-xs text-slate-400 mt-1 italic" id="report-date-info"></p>
                    </div>
                    <div id="pnl-content"></div>
                </div>
            </div>
        </main>

        <!-- Mobile Nav -->
        <nav class="md:hidden fixed bottom-0 left-0 right-0 bg-white border-t border-slate-100 flex items-center justify-around py-4 z-40">
            <button onclick="changeTab('dashboard')" class="mobile-nav-btn text-emerald-600" data-tab="dashboard"><i data-lucide="layout-dashboard"></i></button>
            <button onclick="changeTab('history')" class="mobile-nav-btn text-slate-300" data-tab="history"><i data-lucide="history"></i></button>
            <button onclick="changeTab('expenses')" class="mobile-nav-btn text-slate-300" data-tab="expenses"><i data-lucide="fuel"></i></button>
            <button onclick="changeTab('report')" class="mobile-nav-btn text-slate-300" data-tab="report"><i data-lucide="receipt-text"></i></button>
        </nav>
    </div>

    <script>
        // --- State Management ---
        let transactions = JSON.parse(localStorage.getItem('palm_transactions')) || [];
        let expenses = JSON.parse(localStorage.getItem('palm_expenses')) || [];
        let currentType = 'pembelian';

        // Helper untuk tanggal hari ini format YYYY-MM-DD
        const getTodayStr = () => new Date().toISOString().split('T')[0];

        const saveData = () => {
            // Sort transaksi berdasarkan tanggal (terbaru di atas)
            transactions.sort((a, b) => new Date(b.date) - new Date(a.date));
            expenses.sort((a, b) => new Date(b.date) - new Date(a.date));
            
            localStorage.setItem('palm_transactions', JSON.stringify(transactions));
            localStorage.setItem('palm_expenses', JSON.stringify(expenses));
            updateUI();
        };

        const formatIDR = (n) => new Intl.NumberFormat('id-ID', { style: 'currency', currency: 'IDR', minimumFractionDigits: 0 }).format(n);
        const formatDate = (d) => new Date(d).toLocaleDateString('id-ID', { day: '2-digit', month: 'short', year: 'numeric' });

        const updateUI = () => {
            const tonIn = transactions.filter(t => t.type === 'pembelian').reduce((a, b) => a + Number(b.netWeight), 0);
            const tonOut = transactions.filter(t => t.type === 'penjualan').reduce((a, b) => a + Number(b.netWeight), 0);
            const buyVal = transactions.filter(t => t.type === 'pembelian').reduce((a, b) => a + Number(b.totalPrice), 0);
            const sellVal = transactions.filter(t => t.type === 'penjualan').reduce((a, b) => a + Number(b.totalPrice), 0);
            const expVal = expenses.reduce((a, b) => a + Number(b.amount), 0);
            const stock = tonIn - tonOut;
            const netProfit = (sellVal - buyVal) - expVal;

            document.getElementById('report-date-info').innerText = `Data hingga: ${new Date().toLocaleDateString('id-ID')}`;

            // Stats
            document.getElementById('stat-tonase-in').innerHTML = `${tonIn.toLocaleString()} <span class="text-xs font-normal">KG</span>`;
            document.getElementById('stat-tonase-out').innerHTML = `${tonOut.toLocaleString()} <span class="text-xs font-normal">KG</span>`;
            document.getElementById('stat-stock').innerHTML = `${stock.toLocaleString()} <span class="text-xs font-normal">KG</span>`;
            document.getElementById('stat-net-profit').innerText = formatIDR(netProfit);
            document.getElementById('sidebar-stock').innerText = `${stock.toLocaleString()} KG`;

            // Recent Dashboard
            const recentContainer = document.getElementById('recent-activities');
            recentContainer.innerHTML = transactions.slice(0, 8).map(t => `
                <div class="flex items-center justify-between p-4 bg-slate-50 rounded-2xl border border-slate-100">
                    <div class="flex items-center space-x-3">
                        <div class="p-2 rounded-xl ${t.type === 'pembelian' ? 'bg-emerald-100 text-emerald-600' : 'bg-blue-100 text-blue-600'}">
                            <i data-lucide="${t.type === 'pembelian' ? 'arrow-down-right' : 'arrow-up-right'}"></i>
                        </div>
                        <div>
                            <p class="text-xs font-bold text-slate-400">${formatDate(t.date)}</p>
                            <p class="text-sm font-black text-slate-800">${t.partyName}</p>
                            <p class="text-[10px] font-bold text-slate-400 uppercase">${t.netWeight.toLocaleString()} KG • ${t.type}</p>
                        </div>
                    </div>
                    <p class="text-sm font-black ${t.type === 'pembelian' ? 'text-rose-500' : 'text-emerald-600'}">
                        ${t.type === 'pembelian' ? '-' : '+'}${formatIDR(t.totalPrice)}
                    </p>
                </div>
            `).join('') || '<p class="text-center text-slate-400 py-10 italic">Belum ada data</p>';

            // History Table
            document.getElementById('history-table-body').innerHTML = transactions.map((t, idx) => `
                <tr class="text-sm">
                    <td class="px-6 py-4 text-slate-500 font-medium">${formatDate(t.date)}</td>
                    <td class="px-6 py-4"><span class="px-2 py-0.5 rounded text-[10px] font-black uppercase ${t.type === 'pembelian' ? 'bg-emerald-50 text-emerald-600' : 'bg-blue-50 text-blue-600'}">${t.type}</span></td>
                    <td class="px-6 py-4 font-bold">${t.partyName}</td>
                    <td class="px-6 py-4">${t.netWeight.toLocaleString()} KG</td>
                    <td class="px-6 py-4 text-right font-black">${formatIDR(t.totalPrice)}</td>
                    <td class="px-6 py-4 text-center">
                        <button onclick="deleteItem('trans', ${idx})" class="text-slate-300 hover:text-rose-500"><i data-lucide="trash-2" size="16"></i></button>
                    </td>
                </tr>
            `).join('');

            // Expense Table
            document.getElementById('expense-table-body').innerHTML = expenses.map((e, idx) => `
                <tr class="hover:bg-slate-50 text-sm">
                    <td class="px-6 py-4 text-slate-500 font-medium">${formatDate(e.date)}</td>
                    <td class="px-6 py-4 font-bold text-slate-700">${e.note}</td>
                    <td class="px-6 py-4 text-right font-black text-rose-500">${formatIDR(e.amount)}</td>
                    <td class="px-6 py-4 text-center">
                        <button onclick="deleteItem('exp', ${idx})" class="text-slate-300 hover:text-rose-500"><i data-lucide="trash-2" size="16"></i></button>
                    </td>
                </tr>
            `).join('');

            // Report UI
            document.getElementById('pnl-content').innerHTML = `
                <div class="space-y-6">
                    <div class="flex justify-between py-2 border-b"><span>Penjualan TBS</span><span class="font-bold">${formatIDR(sellVal)}</span></div>
                    <div class="flex justify-between py-2 border-b text-rose-500"><span>Pembelian TBS</span><span class="font-bold">(${formatIDR(buyVal)})</span></div>
                    <div class="flex justify-between py-4 bg-slate-50 px-4 rounded-xl font-black"><span>MARGIN BRUTO</span><span>${formatIDR(sellVal - buyVal)}</span></div>
                    <div class="py-2">
                        <p class="text-xs font-black text-slate-400 uppercase mb-2">Biaya Operasional</p>
                        ${expenses.length > 0 ? expenses.map(e => `<div class="flex justify-between text-sm py-1"><span>${e.note}</span><span>${formatIDR(e.amount)}</span></div>`).join('') : '<p class="text-xs italic text-slate-400">Belum ada biaya</p>'}
                        <div class="flex justify-between text-sm font-bold border-t mt-2 pt-2 text-rose-600"><span>Total Biaya</span><span>(${formatIDR(expVal)})</span></div>
                    </div>
                    <div class="flex justify-between py-6 border-t-4 border-double border-slate-200 mt-4 items-center">
                        <span class="text-xl font-black uppercase">Laba Bersih</span>
                        <span class="text-3xl font-black text-emerald-600">${formatIDR(netProfit)}</span>
                    </div>
                </div>
            `;
            lucide.createIcons();
        };

        const downloadExcel = () => {
            if (transactions.length === 0 && expenses.length === 0) {
                alert("Tidak ada data untuk diunduh!");
                return;
            }

            const transData = transactions.map(t => ({
                Tanggal: formatDate(t.date),
                Tipe: t.type.toUpperCase(),
                Pihak: t.partyName,
                Bruto_KG: t.grossWeight,
                Potongan_Persen: t.deduction,
                Netto_KG: t.netWeight,
                Harga_Per_KG: t.pricePerKg,
                Total_IDR: t.totalPrice
            }));

            const expData = expenses.map(e => ({
                Tanggal: formatDate(e.date),
                Keterangan: e.note,
                Jumlah_IDR: e.amount
            }));

            const tonIn = transactions.filter(t => t.type === 'pembelian').reduce((a, b) => a + Number(b.netWeight), 0);
            const tonOut = transactions.filter(t => t.type === 'penjualan').reduce((a, b) => a + Number(b.netWeight), 0);
            const buyVal = transactions.filter(t => t.type === 'pembelian').reduce((a, b) => a + Number(b.totalPrice), 0);
            const sellVal = transactions.filter(t => t.type === 'penjualan').reduce((a, b) => a + Number(b.totalPrice), 0);
            const expTotal = expenses.reduce((a, b) => a + Number(b.amount), 0);
            
            const summaryData = [
                ["LAPORAN LABA RUGI PALMLOGISTIK"],
                ["Tanggal Laporan:", new Date().toLocaleString('id-ID')],
                [],
                ["Keterangan", "Nilai"],
                ["Total TBS Masuk", tonIn + " KG"],
                ["Total TBS Keluar", tonOut + " KG"],
                ["Stok Sisa", (tonIn - tonOut) + " KG"],
                [],
                ["Total Penjualan", sellVal],
                ["Total Pembelian", buyVal],
                ["Margin Bruto", sellVal - buyVal],
                ["Total Biaya Ops", expTotal],
                ["LABA BERSIH", (sellVal - buyVal) - expTotal]
            ];

            const wb = XLSX.utils.book_new();
            XLSX.utils.book_append_sheet(wb, XLSX.utils.aoa_to_sheet(summaryData), "Ringkasan");
            XLSX.utils.book_append_sheet(wb, XLSX.utils.json_to_sheet(transData), "Data Transaksi");
            XLSX.utils.book_append_sheet(wb, XLSX.utils.json_to_sheet(expData), "Data Biaya");
            XLSX.writeFile(wb, `Laporan_PalmLogistik_${getTodayStr()}.xlsx`);
        };

        const setTransType = (type) => {
            currentType = type;
            const btnP = document.getElementById('btn-pembelian');
            const btnJ = document.getElementById('btn-penjualan');
            btnP.className = type === 'pembelian' ? 'px-4 py-1.5 rounded-lg text-[10px] font-black uppercase transition-all bg-white text-emerald-600 shadow-sm' : 'px-4 py-1.5 rounded-lg text-[10px] font-black uppercase transition-all text-slate-400';
            btnJ.className = type === 'penjualan' ? 'px-4 py-1.5 rounded-lg text-[10px] font-black uppercase transition-all bg-white text-blue-600 shadow-sm' : 'px-4 py-1.5 rounded-lg text-[10px] font-black uppercase transition-all text-slate-400';
            recalculate();
        };

        const recalculate = () => {
            const bruto = parseFloat(document.getElementById('grossWeight').value) || 0;
            const pot = parseFloat(document.getElementById('deduction').value) || 0;
            const price = parseFloat(document.getElementById('price').value) || 0;
            const net = bruto - (bruto * (pot / 100));
            document.getElementById('calc-preview').innerText = formatIDR(net * price);
        };

        const changeTab = (tabId) => {
            document.querySelectorAll('.tab-content').forEach(t => t.classList.add('hidden'));
            document.getElementById('tab-' + tabId).classList.remove('hidden');
            document.querySelectorAll('.nav-btn, .mobile-nav-btn').forEach(btn => {
                const isActive = btn.dataset.tab === tabId;
                btn.classList.toggle('text-emerald-600', isActive);
                if(btn.classList.contains('nav-btn')) {
                    btn.classList.toggle('bg-emerald-600', isActive);
                    btn.classList.toggle('text-white', isActive);
                    btn.classList.toggle('shadow-lg', isActive);
                }
            });
            lucide.createIcons();
        };

        document.getElementById('trans-form').onsubmit = (e) => {
            e.preventDefault();
            const bruto = parseFloat(document.getElementById('grossWeight').value);
            const pot = parseFloat(document.getElementById('deduction').value) || 0;
            const net = bruto - (bruto * (pot / 100));
            const price = parseFloat(document.getElementById('price').value);

            transactions.push({
                partyName: document.getElementById('partyName').value,
                grossWeight: bruto,
                deduction: pot,
                netWeight: net,
                pricePerKg: price,
                totalPrice: net * price,
                type: currentType,
                date: document.getElementById('transDate').value // Ambil dari input manual
            });
            saveData();
            e.target.reset();
            document.getElementById('transDate').value = getTodayStr();
            recalculate();
        };

        document.getElementById('exp-form').onsubmit = (e) => {
            e.preventDefault();
            expenses.push({
                note: document.getElementById('expNote').value,
                amount: parseFloat(document.getElementById('expAmount').value),
                date: document.getElementById('expDate').value // Ambil dari input manual
            });
            saveData();
            e.target.reset();
            document.getElementById('expDate').value = getTodayStr();
        };

        const deleteItem = (type, index) => {
            if(!confirm('Hapus data ini?')) return;
            if(type === 'trans') transactions.splice(index, 1);
            else expenses.splice(index, 1);
            saveData();
        };

        const exportJSON = () => {
            const data = { transactions, expenses };
            const blob = new Blob([JSON.stringify(data, null, 2)], { type: 'application/json' });
            const url = URL.createObjectURL(blob);
            const a = document.createElement('a');
            a.href = url;
            a.download = `Backup_PalmFinance_${getTodayStr()}.json`;
            a.click();
        };

        window.onload = () => {
            // Set default date hari ini
            document.getElementById('transDate').value = getTodayStr();
            document.getElementById('expDate').value = getTodayStr();
            updateUI();
            lucide.createIcons();
        };
    </script>
</body>
</html>

