<!DOCTYPE html>
<html lang="id">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>PalmLogistik Pro - Manajemen Bulanan</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <script src="https://unpkg.com/lucide@latest"></script>
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
        
        <!-- Sidebar -->
        <aside class="hidden md:flex flex-col w-72 bg-white border-r border-slate-200 p-6">
            <div class="flex items-center space-x-3 mb-10 text-emerald-600">
                <div class="w-10 h-10 bg-emerald-600 rounded-xl flex items-center justify-center shadow-lg shadow-emerald-200">
                    <i data-lucide="truck" class="text-white"></i>
                </div>
                <h1 class="text-xl font-black tracking-tighter uppercase">PalmLogistik</h1>
            </div>
            
            <nav class="space-y-1 flex-1">
                <button onclick="changeTab('dashboard')" class="nav-btn w-full flex items-center space-x-3 px-4 py-3 rounded-xl transition-all" data-tab="dashboard">
                    <i data-lucide="layout-dashboard"></i>
                    <span>Dashboard KPI</span>
                </button>
                <button onclick="changeTab('history')" class="nav-btn w-full flex items-center space-x-3 px-4 py-3 rounded-xl transition-all" data-tab="history">
                    <i data-lucide="history"></i>
                    <span>Buku Timbang</span>
                </button>
                <button onclick="changeTab('inventory')" class="nav-btn w-full flex items-center space-x-3 px-4 py-3 rounded-xl transition-all" data-tab="inventory">
                    <i data-lucide="package-search"></i>
                    <span>Stok & Losses</span>
                </button>
                <button onclick="changeTab('expenses')" class="nav-btn w-full flex items-center space-x-3 px-4 py-3 rounded-xl transition-all" data-tab="expenses">
                    <i data-lucide="fuel"></i>
                    <span>Biaya Ops</span>
                </button>
                <button onclick="changeTab('report')" class="nav-btn w-full flex items-center space-x-3 px-4 py-3 rounded-xl transition-all" data-tab="report">
                    <i data-lucide="receipt-text"></i>
                    <span>Laba Rugi</span>
                </button>
            </nav>

            <div class="mt-auto bg-slate-900 rounded-2xl p-5 text-white">
                <p class="text-[10px] opacity-60 uppercase font-black mb-1">Stok TBS Fisik (Update)</p>
                <h4 class="text-xl font-bold" id="sidebar-stock">0 KG</h4>
            </div>
        </aside>

        <!-- Main Content -->
        <main class="flex-1 overflow-y-auto p-4 md:p-8" id="content-area">
            
            <!-- Dashboard -->
            <div id="tab-dashboard" class="tab-content max-w-5xl mx-auto space-y-8">
                <header class="flex justify-between items-start">
                    <div>
                        <h2 class="text-3xl font-black text-slate-800 tracking-tight">Performa RAM</h2>
                        <p class="text-slate-500 font-medium italic">Sistem Detail Timbangan V2.2</p>
                    </div>
                    <button onclick="exportJSON()" class="flex items-center gap-2 px-4 py-2 bg-white border border-slate-200 rounded-xl text-xs font-bold text-slate-600">
                        <i data-lucide="download" size="14"></i> Backup JSON
                    </button>
                </header>

                <div class="grid grid-cols-1 md:grid-cols-4 gap-4">
                    <div class="bg-white p-5 rounded-2xl border border-slate-100 shadow-sm">
                        <p class="text-[10px] font-black text-slate-400 uppercase mb-2">Total Masuk</p>
                        <p class="text-xl font-bold text-slate-800" id="stat-tonase-in">0 KG</p>
                    </div>
                    <div class="bg-white p-5 rounded-2xl border border-slate-100 shadow-sm">
                        <p class="text-[10px] font-black text-slate-400 uppercase mb-2">Total Keluar</p>
                        <p class="text-xl font-bold text-slate-800" id="stat-tonase-out">0 KG</p>
                    </div>
                    <div class="bg-white p-5 rounded-2xl border border-slate-100 shadow-sm border-l-4 border-l-amber-500">
                        <p class="text-[10px] font-black text-amber-600 uppercase mb-2">Stok Saat Ini</p>
                        <p class="text-xl font-bold text-slate-800" id="stat-stock">0 KG</p>
                    </div>
                    <div class="bg-emerald-600 p-5 rounded-2xl shadow-lg text-white">
                        <p class="text-[10px] font-black opacity-70 uppercase mb-2">Total Laba Bersih</p>
                        <p class="text-xl font-bold" id="stat-net-profit">Rp 0</p>
                    </div>
                </div>

                <div class="grid grid-cols-1 lg:grid-cols-2 gap-8">
                    <!-- Form Input Detail -->
                    <div class="bg-white p-6 rounded-3xl shadow-sm border border-slate-100">
                        <div class="flex items-center justify-between mb-6">
                            <h3 class="text-lg font-bold flex items-center gap-2"><i data-lucide="scale"></i> Input Timbangan Detail</h3>
                            <div class="flex bg-slate-100 p-1 rounded-xl">
                                <button onclick="setTransType('pembelian')" id="btn-pembelian" class="px-4 py-1.5 rounded-lg text-[10px] font-black uppercase transition-all">Beli</button>
                                <button onclick="setTransType('penjualan')" id="btn-penjualan" class="px-4 py-1.5 rounded-lg text-[10px] font-black uppercase transition-all">Jual</button>
                            </div>
                        </div>

                        <form id="trans-form" class="space-y-4">
                            <div class="grid grid-cols-2 gap-4">
                                <div class="space-y-1">
                                    <label class="text-[10px] font-black uppercase text-slate-400 ml-1">Tanggal</label>
                                    <input required type="date" id="transDate" class="w-full px-4 py-3 bg-slate-50 border border-slate-200 rounded-xl outline-none" />
                                </div>
                                <div class="space-y-1">
                                    <label class="text-[10px] font-black uppercase text-slate-400 ml-1">Pihak (Petani/PKS)</label>
                                    <input required type="text" id="partyName" placeholder="..." class="w-full px-4 py-3 bg-slate-50 border border-slate-200 rounded-xl outline-none" />
                                </div>
                            </div>

                            <div class="grid grid-cols-3 gap-4">
                                <div class="space-y-1">
                                    <label class="text-[10px] font-black uppercase text-slate-400 ml-1">Bruto (KG)</label>
                                    <input required type="number" id="grossWeight" placeholder="0" oninput="recalculate()" class="w-full px-4 py-3 bg-slate-50 border border-slate-200 rounded-xl outline-none font-bold text-slate-700" />
                                </div>
                                <div class="space-y-1">
                                    <label class="text-[10px] font-black uppercase text-slate-400 ml-1">Tarra (KG)</label>
                                    <input required type="number" id="tareWeight" placeholder="0" oninput="recalculate()" class="w-full px-4 py-3 bg-slate-50 border border-slate-200 rounded-xl outline-none font-bold text-slate-700" />
                                </div>
                                <div class="space-y-1">
                                    <label class="text-[10px] font-black uppercase text-slate-400 ml-1">Netto 1 (KG)</label>
                                    <input readonly type="number" id="netto1" placeholder="0" class="w-full px-4 py-3 bg-slate-100 border border-slate-200 rounded-xl outline-none font-bold text-slate-500" />
                                </div>
                            </div>

                            <div class="grid grid-cols-2 gap-4">
                                <div class="space-y-1">
                                    <label class="text-[10px] font-black uppercase text-slate-400 ml-1">Potongan (%)</label>
                                    <input type="number" step="0.01" id="deduction" placeholder="Contoh: 2.5" oninput="recalculate()" class="w-full px-4 py-3 bg-slate-50 border border-slate-200 rounded-xl outline-none" />
                                </div>
                                <div class="space-y-1">
                                    <label class="text-[10px] font-black uppercase text-slate-400 ml-1">Harga (Rp/KG)</label>
                                    <input required type="number" id="price" placeholder="0" oninput="recalculate()" class="w-full px-4 py-3 bg-slate-50 border border-slate-200 rounded-xl outline-none font-bold text-emerald-700" />
                                </div>
                            </div>

                            <div class="bg-slate-900 p-4 rounded-2xl text-white space-y-2">
                                <div class="flex justify-between text-xs opacity-70">
                                    <span>Netto Akhir:</span>
                                    <span id="final-netto-preview">0 KG</span>
                                </div>
                                <div class="flex justify-between font-black text-lg">
                                    <span>Total Bayar:</span>
                                    <span id="calc-preview">Rp 0</span>
                                </div>
                            </div>
                            <button type="submit" class="w-full py-4 rounded-2xl font-black text-white bg-emerald-600 shadow-lg hover:bg-emerald-700 transition-all">SIMPAN TRANSAKSI</button>
                        </form>
                    </div>

                    <!-- Aktivitas Terakhir -->
                    <div class="bg-white p-6 rounded-3xl shadow-sm border border-slate-100">
                        <h3 class="text-lg font-bold mb-6">Aktivitas Terkini</h3>
                        <div id="recent-activities" class="space-y-4 max-h-[500px] overflow-y-auto pr-2 custom-scrollbar"></div>
                    </div>
                </div>
            </div>

            <!-- Buku Timbang (Harian) -->
            <div id="tab-history" class="tab-content hidden max-w-6xl mx-auto space-y-6">
                <div class="flex flex-col md:flex-row justify-between items-start md:items-center gap-4">
                    <div>
                        <h2 class="text-2xl font-black text-slate-800 tracking-tight uppercase">Buku Timbang Detail</h2>
                        <p class="text-xs text-slate-500 font-bold">Laporan rincian timbangan harian</p>
                    </div>
                    <div class="flex items-center gap-3 bg-white p-2 rounded-2xl shadow-sm border border-slate-100">
                        <label class="text-[10px] font-black uppercase text-slate-400 px-2">Pilih Tanggal:</label>
                        <input type="date" id="history-date-filter" onchange="renderHistory()" class="bg-slate-50 border-none outline-none text-sm font-bold p-2 rounded-xl">
                    </div>
                </div>

                <!-- Summary Cards Harian -->
                <div class="grid grid-cols-2 md:grid-cols-4 gap-4">
                    <div class="bg-white p-4 rounded-2xl border border-slate-100">
                        <p class="text-[9px] font-black text-slate-400 uppercase">Total Beli (KG)</p>
                        <p id="day-ton-in" class="text-lg font-black text-slate-800">0</p>
                    </div>
                    <div class="bg-white p-4 rounded-2xl border border-slate-100">
                        <p class="text-[9px] font-black text-slate-400 uppercase">Total Jual (KG)</p>
                        <p id="day-ton-out" class="text-lg font-black text-slate-800">0</p>
                    </div>
                    <div class="bg-white p-4 rounded-2xl border border-slate-100">
                        <p class="text-[9px] font-black text-slate-400 uppercase">Uang Keluar</p>
                        <p id="day-cash-out" class="text-lg font-black text-rose-500">Rp 0</p>
                    </div>
                    <div class="bg-white p-4 rounded-2xl border border-slate-100">
                        <p class="text-[9px] font-black text-slate-400 uppercase">Uang Masuk</p>
                        <p id="day-cash-in" class="text-lg font-black text-emerald-600">Rp 0</p>
                    </div>
                </div>

                <div class="bg-white rounded-3xl border border-slate-100 shadow-sm overflow-hidden">
                    <div class="overflow-x-auto">
                        <table class="w-full text-left min-w-[1000px]">
                            <thead class="bg-slate-50 text-[10px] font-black text-slate-400 uppercase border-b">
                                <tr>
                                    <th class="px-4 py-4">Status</th>
                                    <th class="px-4 py-4">Pihak</th>
                                    <th class="px-4 py-4">Bruto</th>
                                    <th class="px-4 py-4">Tarra</th>
                                    <th class="px-4 py-4 text-emerald-600">Net 1</th>
                                    <th class="px-4 py-4">Pot%</th>
                                    <th class="px-4 py-4 font-bold text-slate-800">Net Final</th>
                                    <th class="px-4 py-4">Harga</th>
                                    <th class="px-4 py-4 text-right">Total (Rp)</th>
                                    <th class="px-4 py-4 text-center">Aksi</th>
                                </tr>
                            </thead>
                            <tbody id="history-table-body" class="divide-y divide-slate-100"></tbody>
                        </table>
                    </div>
                    <div id="history-empty-state" class="hidden py-20 text-center">
                        <div class="w-16 h-16 bg-slate-100 rounded-full flex items-center justify-center mx-auto mb-4 opacity-50">
                            <i data-lucide="calendar-x" class="text-slate-400"></i>
                        </div>
                        <p class="text-slate-400 font-bold italic">Tidak ada transaksi pada tanggal ini</p>
                    </div>
                </div>
            </div>

            <!-- Stok & Losses (Inventory Control) -->
            <div id="tab-inventory" class="tab-content hidden max-w-5xl mx-auto space-y-8">
                <div class="flex justify-between items-center">
                    <h2 class="text-2xl font-black text-slate-800 tracking-tight uppercase">Kontrol Stok & Losses</h2>
                </div>

                <div class="grid grid-cols-1 md:grid-cols-3 gap-6">
                    <!-- Penyesuaian Form -->
                    <div class="md:col-span-1 bg-white p-6 rounded-3xl shadow-sm border border-slate-100 space-y-4">
                        <h3 class="text-sm font-black text-slate-400 uppercase tracking-widest">Revisi Stok Manual</h3>
                        <form id="stock-adj-form" class="space-y-4">
                            <div class="space-y-1">
                                <label class="text-[10px] font-black uppercase text-slate-400 ml-1">Tanggal</label>
                                <input required type="date" id="adjDate" class="w-full px-4 py-3 bg-slate-50 border border-slate-200 rounded-xl outline-none" />
                            </div>
                            <div class="space-y-1">
                                <label class="text-[10px] font-black uppercase text-slate-400 ml-1">Jumlah Revisi (KG)</label>
                                <input required type="number" id="adjAmount" placeholder="Contoh: -50 untuk susut" class="w-full px-4 py-3 bg-slate-50 border border-slate-200 rounded-xl outline-none font-bold" />
                                <p class="text-[9px] text-slate-400 mt-1 italic">* Gunakan tanda minus (-) untuk menyatakan losses/susut.</p>
                            </div>
                            <div class="space-y-1">
                                <label class="text-[10px] font-black uppercase text-slate-400 ml-1">Keterangan</label>
                                <input required type="text" id="adjNote" placeholder="Misal: Penyesuaian akhir bulan" class="w-full px-4 py-3 bg-slate-50 border border-slate-200 rounded-xl outline-none" />
                            </div>
                            <button type="submit" class="w-full py-4 rounded-2xl font-black text-white bg-slate-900 shadow-lg hover:bg-black transition-all">SIMPAN REVISI</button>
                        </form>
                    </div>

                    <!-- Riwayat Penyesuaian -->
                    <div class="md:col-span-2 bg-white p-6 rounded-3xl shadow-sm border border-slate-100">
                        <h3 class="text-sm font-black text-slate-400 uppercase tracking-widest mb-4">Riwayat Koreksi Stok</h3>
                        <div class="overflow-hidden rounded-2xl border border-slate-50">
                            <table class="w-full text-left">
                                <thead class="bg-slate-50 text-[10px] font-black text-slate-400 uppercase">
                                    <tr>
                                        <th class="px-4 py-3">Tanggal</th>
                                        <th class="px-4 py-3">Keterangan</th>
                                        <th class="px-4 py-3 text-right">Jumlah (KG)</th>
                                        <th class="px-4 py-3 text-center">Aksi</th>
                                    </tr>
                                </thead>
                                <tbody id="adjustment-table-body" class="divide-y divide-slate-50 text-xs"></tbody>
                            </table>
                        </div>
                    </div>
                </div>
            </div>

            <!-- Biaya Operasional -->
            <div id="tab-expenses" class="tab-content hidden max-w-3xl mx-auto space-y-6">
                <h2 class="text-2xl font-black uppercase italic">Biaya Ops</h2>
                <form id="exp-form" class="bg-white p-6 rounded-3xl border border-slate-100 shadow-sm space-y-4">
                    <div class="grid grid-cols-1 md:grid-cols-2 gap-4">
                        <input required type="date" id="expDate" class="w-full px-4 py-3 bg-slate-50 border border-slate-200 rounded-xl outline-none" />
                        <input required type="text" id="expNote" placeholder="Keterangan (Solar, Gaji, dll)" class="w-full px-4 py-3 bg-slate-50 border border-slate-200 rounded-xl outline-none" />
                    </div>
                    <div class="flex flex-col md:flex-row gap-4">
                        <input required type="number" id="expAmount" placeholder="Jumlah Rp" class="flex-1 px-4 py-3 bg-slate-50 border border-slate-200 rounded-xl outline-none" />
                        <button class="bg-slate-900 text-white px-8 py-3 rounded-xl font-bold uppercase tracking-wide">Input Biaya</button>
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

            <!-- Laporan Bulanan -->
            <div id="tab-report" class="tab-content hidden max-w-4xl mx-auto space-y-6">
                <div class="flex flex-col md:flex-row justify-between items-start md:items-center gap-4">
                    <h2 class="text-2xl font-black uppercase italic">Arsip Laporan Bulanan</h2>
                    <div class="flex flex-wrap gap-2">
                        <select id="filter-month" onchange="renderReport()" class="bg-white border border-slate-200 px-3 py-2 rounded-xl text-sm font-bold outline-none"></select>
                        <select id="filter-year" onchange="renderReport()" class="bg-white border border-slate-200 px-3 py-2 rounded-xl text-sm font-bold outline-none"></select>
                        <button onclick="downloadExcel()" class="flex items-center gap-2 bg-emerald-600 text-white px-5 py-2 rounded-xl font-bold shadow-md hover:bg-emerald-700 transition-all text-sm">
                            <i data-lucide="file-spreadsheet" size="16"></i> Excel
                        </button>
                    </div>
                </div>

                <div class="bg-white rounded-[2rem] shadow-xl border border-slate-100 p-8 space-y-8">
                    <div class="text-center border-b pb-6">
                        <div class="w-16 h-16 bg-emerald-100 text-emerald-600 rounded-2xl flex items-center justify-center mx-auto mb-4">
                             <i data-lucide="truck" size="32"></i>
                        </div>
                        <h2 class="text-2xl font-black uppercase tracking-tighter text-slate-800">PalmLogistik - Laporan Laba Rugi</h2>
                        <p class="text-slate-500 font-bold" id="report-period-label"></p>
                    </div>
                    <div id="pnl-content"></div>
                </div>
            </div>
        </main>

        <!-- Mobile Nav -->
        <nav class="md:hidden fixed bottom-0 left-0 right-0 bg-white border-t border-slate-100 flex items-center justify-around py-4 z-40">
            <button onclick="changeTab('dashboard')" class="mobile-nav-btn" data-tab="dashboard"><i data-lucide="layout-dashboard"></i></button>
            <button onclick="changeTab('history')" class="mobile-nav-btn" data-tab="history"><i data-lucide="history"></i></button>
            <button onclick="changeTab('inventory')" class="mobile-nav-btn" data-tab="inventory"><i data-lucide="package-search"></i></button>
            <button onclick="changeTab('expenses')" class="mobile-nav-btn" data-tab="expenses"><i data-lucide="fuel"></i></button>
            <button onclick="changeTab('report')" class="mobile-nav-btn" data-tab="report"><i data-lucide="receipt-text"></i></button>
        </nav>
    </div>

    <script>
        // --- Core State ---
        let transactions = JSON.parse(localStorage.getItem('palm_v2_trans')) || [];
        let expenses = JSON.parse(localStorage.getItem('palm_v2_exp')) || [];
        let adjustments = JSON.parse(localStorage.getItem('palm_v2_adj')) || [];
        let currentType = 'pembelian';

        const MONTH_NAMES = ["Januari", "Februari", "Maret", "April", "Mei", "Juni", "Juli", "Agustus", "September", "Oktober", "November", "Desember"];
        const getTodayStr = () => new Date().toISOString().split('T')[0];

        // --- Helpers ---
        const formatIDR = (n) => new Intl.NumberFormat('id-ID', { style: 'currency', currency: 'IDR', minimumFractionDigits: 0 }).format(n);
        const formatDate = (d) => new Date(d).toLocaleDateString('id-ID', { day: '2-digit', month: 'short', year: 'numeric' });

        const saveData = () => {
            localStorage.setItem('palm_v2_trans', JSON.stringify(transactions));
            localStorage.setItem('palm_v2_exp', JSON.stringify(expenses));
            localStorage.setItem('palm_v2_adj', JSON.stringify(adjustments));
            updateUI();
        };

        // --- UI Logic ---
        const updateUI = () => {
            refreshStats();
            renderRecent();
            renderHistory();
            renderAdjustmentTable();
            renderExpenseTable();
            initReportFilters();
            renderReport();
            lucide.createIcons();
        };

        const refreshStats = () => {
            const tonIn = transactions.filter(t => t.type === 'pembelian').reduce((a, b) => a + b.netWeight, 0);
            const tonOut = transactions.filter(t => t.type === 'penjualan').reduce((a, b) => a + b.netWeight, 0);
            const adjTotal = adjustments.reduce((a, b) => a + b.amount, 0);
            
            const currentStock = (tonIn - tonOut) + adjTotal;

            const buyVal = transactions.filter(t => t.type === 'pembelian').reduce((a, b) => a + b.totalPrice, 0);
            const sellVal = transactions.filter(t => t.type === 'penjualan').reduce((a, b) => a + b.totalPrice, 0);
            const expTotal = expenses.reduce((a, b) => a + b.amount, 0);

            document.getElementById('stat-tonase-in').innerText = `${tonIn.toLocaleString()} KG`;
            document.getElementById('stat-tonase-out').innerText = `${tonOut.toLocaleString()} KG`;
            document.getElementById('stat-stock').innerText = `${currentStock.toLocaleString()} KG`;
            document.getElementById('stat-net-profit').innerText = formatIDR((sellVal - buyVal) - expTotal);
            document.getElementById('sidebar-stock').innerText = `${currentStock.toLocaleString()} KG`;
        };

        const renderRecent = () => {
            const list = transactions.slice().sort((a,b) => new Date(b.date) - new Date(a.date)).slice(0, 6);
            document.getElementById('recent-activities').innerHTML = list.map(t => `
                <div class="p-4 bg-slate-50 rounded-2xl border border-slate-100 flex justify-between items-center text-xs">
                    <div>
                        <p class="text-[9px] font-black text-slate-400 uppercase">${formatDate(t.date)} • ${t.type}</p>
                        <p class="font-bold text-slate-800">${t.partyName}</p>
                        <p class="text-[10px] font-medium text-slate-500">${(t.netWeight || 0).toLocaleString()} KG @ ${formatIDR(t.pricePerKg)}</p>
                    </div>
                    <div class="text-right">
                        <p class="font-black ${t.type === 'pembelian' ? 'text-rose-500' : 'text-emerald-600'}">
                            ${t.type === 'pembelian' ? '-' : '+'}${formatIDR(t.totalPrice)}
                        </p>
                    </div>
                </div>
            `).join('') || '<p class="text-center py-10 text-slate-400 italic">Belum ada transaksi</p>';
        };

        const renderHistory = () => {
            const filterDate = document.getElementById('history-date-filter').value;
            let list = transactions.slice().sort((a,b) => new Date(b.date) - new Date(a.date));
            
            if (filterDate) {
                list = list.filter(t => t.date === filterDate);
            }

            // Summary Harian
            const dIn = list.filter(t => t.type === 'pembelian').reduce((a,b) => a + b.netWeight, 0);
            const dOut = list.filter(t => t.type === 'penjualan').reduce((a,b) => a + b.netWeight, 0);
            const cOut = list.filter(t => t.type === 'pembelian').reduce((a,b) => a + b.totalPrice, 0);
            const cIn = list.filter(t => t.type === 'penjualan').reduce((a,b) => a + b.totalPrice, 0);

            document.getElementById('day-ton-in').innerText = `${dIn.toLocaleString()} KG`;
            document.getElementById('day-ton-out').innerText = `${dOut.toLocaleString()} KG`;
            document.getElementById('day-cash-out').innerText = formatIDR(cOut);
            document.getElementById('day-cash-in').innerText = formatIDR(cIn);

            const tableBody = document.getElementById('history-table-body');
            const emptyState = document.getElementById('history-empty-state');

            if (list.length === 0) {
                tableBody.innerHTML = '';
                emptyState.classList.remove('hidden');
            } else {
                emptyState.classList.add('hidden');
                tableBody.innerHTML = list.map((t, i) => `
                    <tr class="text-xs">
                        <td class="px-4 py-4">
                            <span class="px-2 py-1 rounded-lg font-black uppercase text-[8px] ${t.type === 'pembelian' ? 'bg-rose-100 text-rose-600' : 'bg-emerald-100 text-emerald-600'}">${t.type}</span>
                            <p class="text-[9px] text-slate-400 mt-1">${formatDate(t.date)}</p>
                        </td>
                        <td class="px-4 py-4 font-bold text-slate-700">${t.partyName}</td>
                        <td class="px-4 py-4 text-slate-500">${(t.grossWeight || 0).toLocaleString()}</td>
                        <td class="px-4 py-4 text-slate-500">${(t.tareWeight || 0).toLocaleString()}</td>
                        <td class="px-4 py-4 text-emerald-600 font-bold">${(t.netto1 || 0).toLocaleString()}</td>
                        <td class="px-4 py-4 text-slate-500">${(t.deduction || 0)}%</td>
                        <td class="px-4 py-4 font-black bg-slate-50 text-slate-800">${(t.netWeight || 0).toLocaleString()}</td>
                        <td class="px-4 py-4 text-slate-600 font-medium">${formatIDR(t.pricePerKg)}</td>
                        <td class="px-4 py-4 text-right font-black text-slate-900">${formatIDR(t.totalPrice)}</td>
                        <td class="px-4 py-4 text-center">
                            <button onclick="deleteItem('trans', ${transactions.indexOf(t)})" class="w-8 h-8 rounded-full bg-slate-50 text-slate-300 hover:text-rose-500 hover:bg-rose-50 flex items-center justify-center transition-all mx-auto"><i data-lucide="trash-2" size="14"></i></button>
                        </td>
                    </tr>
                `).join('');
            }
            lucide.createIcons();
        };

        const renderAdjustmentTable = () => {
            const list = adjustments.slice().sort((a,b) => new Date(b.date) - new Date(a.date));
            document.getElementById('adjustment-table-body').innerHTML = list.map((adj, i) => `
                <tr>
                    <td class="px-4 py-3 text-slate-500">${formatDate(adj.date)}</td>
                    <td class="px-4 py-3 font-semibold">${adj.note}</td>
                    <td class="px-4 py-3 text-right font-bold ${adj.amount < 0 ? 'text-rose-500' : 'text-emerald-600'}">${adj.amount > 0 ? '+' : ''}${adj.amount.toLocaleString()} KG</td>
                    <td class="px-4 py-3 text-center">
                        <button onclick="deleteItem('adj', ${adjustments.indexOf(adj)})" class="text-slate-300 hover:text-rose-500"><i data-lucide="trash-2" size="14"></i></button>
                    </td>
                </tr>
            `).join('') || '<tr><td colspan="4" class="text-center py-6 text-slate-400">Belum ada revisi manual</td></tr>';
        };

        const renderExpenseTable = () => {
            const list = expenses.slice().sort((a,b) => new Date(b.date) - new Date(a.date));
            document.getElementById('expense-table-body').innerHTML = list.map((e, i) => `
                <tr class="text-sm">
                    <td class="px-6 py-4 text-slate-500">${formatDate(e.date)}</td>
                    <td class="px-6 py-4 font-bold">${e.note}</td>
                    <td class="px-6 py-4 text-right font-black text-rose-500">${formatIDR(e.amount)}</td>
                    <td class="px-6 py-4 text-center">
                        <button onclick="deleteItem('exp', ${expenses.indexOf(e)})" class="text-slate-300 hover:text-rose-500"><i data-lucide="trash-2" size="16"></i></button>
                    </td>
                </tr>
            `).join('');
        };

        // --- Monthly Logic ---
        const initReportFilters = () => {
            const monthSel = document.getElementById('filter-month');
            const yearSel = document.getElementById('filter-year');
            if (monthSel.options.length > 0) return;

            MONTH_NAMES.forEach((m, i) => {
                const opt = new Option(m, i);
                if (i === new Date().getMonth()) opt.selected = true;
                monthSel.add(opt);
            });

            const currentYear = new Date().getFullYear();
            for (let y = currentYear - 2; y <= currentYear + 2; y++) {
                const opt = new Option(y, y);
                if (y === currentYear) opt.selected = true;
                yearSel.add(opt);
            }
        };

        const renderReport = () => {
            const m = parseInt(document.getElementById('filter-month').value);
            const y = parseInt(document.getElementById('filter-year').value);
            
            document.getElementById('report-period-label').innerText = `Periode: ${MONTH_NAMES[m]} ${y}`;

            const filteredTrans = transactions.filter(t => {
                const d = new Date(t.date);
                return d.getMonth() === m && d.getFullYear() === y;
            });

            const filteredExp = expenses.filter(e => {
                const d = new Date(e.date);
                return d.getMonth() === m && d.getFullYear() === y;
            });

            const filteredAdj = adjustments.filter(a => {
                const d = new Date(a.date);
                return d.getMonth() === m && d.getFullYear() === y;
            });

            const buyTotal = filteredTrans.filter(t => t.type === 'pembelian').reduce((a,b) => a + b.totalPrice, 0);
            const sellTotal = filteredTrans.filter(t => t.type === 'penjualan').reduce((a,b) => a + b.totalPrice, 0);
            const tonIn = filteredTrans.filter(t => t.type === 'pembelian').reduce((a,b) => a + b.netWeight, 0);
            const tonOut = filteredTrans.filter(t => t.type === 'penjualan').reduce((a,b) => a + b.netWeight, 0);
            const adjSum = filteredAdj.reduce((a,b) => a + b.amount, 0);
            
            const expSum = filteredExp.reduce((a,b) => a + b.amount, 0);
            const net = (sellTotal - buyTotal) - expSum;

            document.getElementById('pnl-content').innerHTML = `
                <div class="space-y-6">
                    <div class="grid grid-cols-2 gap-4 pb-6 border-b border-dashed">
                        <div>
                            <p class="text-[10px] font-black text-slate-400 uppercase">TBS Masuk</p>
                            <p class="text-lg font-bold">${tonIn.toLocaleString()} KG</p>
                        </div>
                        <div class="text-right">
                            <p class="text-[10px] font-black text-slate-400 uppercase">TBS Keluar</p>
                            <p class="text-lg font-bold">${tonOut.toLocaleString()} KG</p>
                        </div>
                    </div>

                    <div class="bg-amber-50 p-4 rounded-2xl flex justify-between items-center text-xs">
                         <span class="font-bold text-amber-700 uppercase">Koreksi Stok / Losses</span>
                         <span class="font-black ${adjSum < 0 ? 'text-rose-600' : 'text-emerald-600'}">${adjSum.toLocaleString()} KG</span>
                    </div>
                    
                    <div class="space-y-3">
                        <div class="flex justify-between items-center text-sm">
                            <span class="font-bold text-slate-600">Total Penjualan TBS</span>
                            <span class="font-black text-emerald-600">${formatIDR(sellTotal)}</span>
                        </div>
                        <div class="flex justify-between items-center text-sm">
                            <span class="font-bold text-slate-600">Total Pembelian TBS</span>
                            <span class="font-black text-rose-500">(${formatIDR(buyTotal)})</span>
                        </div>
                        <div class="flex justify-between items-center p-4 bg-slate-50 rounded-2xl">
                            <span class="font-black text-xs uppercase tracking-widest text-slate-400">Margin Kotor</span>
                            <span class="font-black text-slate-800 text-lg">${formatIDR(sellTotal - buyTotal)}</span>
                        </div>
                    </div>

                    <div>
                        <p class="text-[10px] font-black text-slate-400 uppercase mb-3">Biaya Operasional Bulan Ini</p>
                        <div class="space-y-2">
                            ${filteredExp.map(e => `
                                <div class="flex justify-between text-xs font-medium border-b border-slate-50 pb-1">
                                    <span class="text-slate-500">${e.note}</span>
                                    <span class="text-slate-700">${formatIDR(e.amount)}</span>
                                </div>
                            `).join('') || '<p class="text-xs italic text-slate-400">Tidak ada pengeluaran</p>'}
                        </div>
                        <div class="flex justify-between mt-3 font-bold text-rose-500 text-sm">
                            <span>Total Pengeluaran</span>
                            <span>(${formatIDR(expSum)})</span>
                        </div>
                    </div>

                    <div class="pt-6 border-t-4 border-double border-slate-100 flex justify-between items-center">
                        <h4 class="text-xl font-black uppercase italic">Laba Bersih</h4>
                        <span class="text-3xl font-black ${net >= 0 ? 'text-emerald-600' : 'text-rose-600'}">${formatIDR(net)}</span>
                    </div>
                </div>
            `;
            lucide.createIcons();
        };

        // --- Interaction ---
        const changeTab = (tabId) => {
            document.querySelectorAll('.tab-content').forEach(t => t.classList.add('hidden'));
            document.getElementById('tab-' + tabId).classList.remove('hidden');
            document.querySelectorAll('.nav-btn, .mobile-nav-btn').forEach(btn => {
                const active = btn.dataset.tab === tabId;
                btn.classList.toggle('bg-emerald-600', active && btn.classList.contains('nav-btn'));
                btn.classList.toggle('text-white', active && btn.classList.contains('nav-btn'));
                btn.classList.toggle('text-emerald-600', active && !btn.classList.contains('nav-btn'));
                btn.classList.toggle('text-slate-400', !active);
            });
            if(tabId === 'report') renderReport();
            if(tabId === 'history') {
                if(!document.getElementById('history-date-filter').value) {
                    document.getElementById('history-date-filter').value = getTodayStr();
                }
                renderHistory();
            }
            lucide.createIcons();
        };

        const setTransType = (type) => {
            currentType = type;
            const isBuy = type === 'pembelian';
            document.getElementById('btn-pembelian').className = `px-4 py-1.5 rounded-lg text-[10px] font-black uppercase transition-all ${isBuy ? 'bg-white text-emerald-600 shadow-sm' : 'text-slate-400'}`;
            document.getElementById('btn-penjualan').className = `px-4 py-1.5 rounded-lg text-[10px] font-black uppercase transition-all ${!isBuy ? 'bg-white text-blue-600 shadow-sm' : 'text-slate-400'}`;
            recalculate();
        };

        const recalculate = () => {
            const bruto = parseFloat(document.getElementById('grossWeight').value) || 0;
            const tare = parseFloat(document.getElementById('tareWeight').value) || 0;
            const netto1 = bruto - tare;
            
            const potPercent = parseFloat(document.getElementById('deduction').value) || 0;
            const price = parseFloat(document.getElementById('price').value) || 0;
            
            const finalNetto = netto1 - (netto1 * (potPercent / 100));
            
            document.getElementById('netto1').value = netto1;
            document.getElementById('final-netto-preview').innerText = `${finalNetto.toLocaleString()} KG`;
            document.getElementById('calc-preview').innerText = formatIDR(finalNetto * price);
        };

        // --- Form Handlers ---
        document.getElementById('trans-form').onsubmit = (e) => {
            e.preventDefault();
            const bruto = parseFloat(document.getElementById('grossWeight').value);
            const tare = parseFloat(document.getElementById('tareWeight').value);
            const netto1 = bruto - tare;
            const pot = parseFloat(document.getElementById('deduction').value) || 0;
            const netWeight = netto1 - (netto1 * (pot / 100));
            const price = parseFloat(document.getElementById('price').value);

            transactions.push({
                partyName: document.getElementById('partyName').value,
                grossWeight: bruto,
                tareWeight: tare,
                netto1: netto1,
                deduction: pot,
                netWeight: netWeight,
                pricePerKg: price,
                totalPrice: netWeight * price,
                type: currentType,
                date: document.getElementById('transDate').value
            });
            saveData();
            e.target.reset();
            document.getElementById('transDate').value = getTodayStr();
            recalculate();
        };

        document.getElementById('stock-adj-form').onsubmit = (e) => {
            e.preventDefault();
            adjustments.push({
                date: document.getElementById('adjDate').value,
                amount: parseFloat(document.getElementById('adjAmount').value),
                note: document.getElementById('adjNote').value
            });
            saveData();
            e.target.reset();
            document.getElementById('adjDate').value = getTodayStr();
        };

        document.getElementById('exp-form').onsubmit = (e) => {
            e.preventDefault();
            expenses.push({
                note: document.getElementById('expNote').value,
                amount: parseFloat(document.getElementById('expAmount').value),
                date: document.getElementById('expDate').value
            });
            saveData();
            e.target.reset();
            document.getElementById('expDate').value = getTodayStr();
        };

        const deleteItem = (type, index) => {
            if(!confirm('Hapus data ini secara permanen?')) return;
            if(type === 'trans') transactions.splice(index, 1);
            else if(type === 'exp') expenses.splice(index, 1);
            else if(type === 'adj') adjustments.splice(index, 1);
            saveData();
        };

        const downloadExcel = () => {
            const m = parseInt(document.getElementById('filter-month').value);
            const y = parseInt(document.getElementById('filter-year').value);
            const mName = MONTH_NAMES[m];

            const filteredTrans = transactions.filter(t => {
                const d = new Date(t.date);
                return d.getMonth() === m && d.getFullYear() === y;
            });

            const wb = XLSX.utils.book_new();
            XLSX.utils.book_append_sheet(wb, XLSX.utils.json_to_sheet(filteredTrans), "Transaksi");
            XLSX.utils.book_append_sheet(wb, XLSX.utils.json_to_sheet(adjustments), "Penyesuaian_Stok");
            XLSX.writeFile(wb, `Laporan_PalmLogistik_${mName}_${y}.xlsx`);
        };

        const exportJSON = () => {
            const blob = new Blob([JSON.stringify({transactions, expenses, adjustments}, null, 2)], {type: 'application/json'});
            const a = document.createElement('a');
            a.href = URL.createObjectURL(blob);
            a.download = `PalmLogistik_Backup_${getTodayStr()}.json`;
            a.click();
        };

        window.onload = () => {
            document.getElementById('transDate').value = getTodayStr();
            document.getElementById('expDate').value = getTodayStr();
            document.getElementById('adjDate').value = getTodayStr();
            document.getElementById('history-date-filter').value = getTodayStr();
            setTransType('pembelian');
            changeTab('dashboard');
            updateUI();
        };
    </script>
</body>
</html>

