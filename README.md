<!DOCTYPE html>
<html lang="id">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>PalmLogistik - Manajemen RAM TBS</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <script src="https://unpkg.com/lucide@latest"></script>
    <!-- Firebase SDKs -->
    <script src="https://www.gstatic.com/firebasejs/11.0.2/firebase-app-compat.js"></script>
    <script src="https://www.gstatic.com/firebasejs/11.0.2/firebase-auth-compat.js"></script>
    <script src="https://www.gstatic.com/firebasejs/11.0.2/firebase-firestore-compat.js"></script>
    <style>
        @import url('https://fonts.googleapis.com/css2?family=Plus+Jakarta+Sans:wght@400;600;700;800&display=swap');
        body { font-family: 'Plus Jakarta Sans', sans-serif; }
        .custom-scrollbar::-webkit-scrollbar { width: 4px; }
        .custom-scrollbar::-webkit-scrollbar-thumb { background: #cbd5e1; border-radius: 10px; }
        [v-cloak] { display: none; }
        .offline-badge { display: none; }
        body.is-offline .offline-badge { display: flex; }
    </style>
</head>
<body class="bg-slate-50 text-slate-900 pb-20 md:pb-0">
    <!-- Status Bar Offline -->
    <div class="offline-badge fixed top-0 left-0 w-full bg-amber-500 text-white text-[10px] font-bold py-1 justify-center items-center z-[100] gap-2">
        <i data-lucide="wifi-off" size="12"></i> MODE OFFLINE: Data akan disinkronkan saat internet kembali.
    </div>

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
                <p class="text-[10px] opacity-60 uppercase font-black mb-1">Stok TBS di RAM</p>
                <h4 class="text-xl font-bold" id="sidebar-stock">0 KG</h4>
            </div>
        </aside>

        <!-- Main Content -->
        <main class="flex-1 overflow-y-auto p-4 md:p-8" id="content-area">
            <!-- Loading Overlay -->
            <div id="loading-overlay" class="fixed inset-0 bg-white z-50 flex flex-col items-center justify-center">
                <div class="animate-spin text-emerald-600 mb-4"><i data-lucide="loader-2" size="48"></i></div>
                <p class="text-slate-500 font-bold">Menyiapkan Database...</p>
            </div>

            <!-- Tab: Dashboard -->
            <div id="tab-dashboard" class="tab-content max-w-5xl mx-auto space-y-8 animate-in fade-in">
                <header>
                    <h2 class="text-3xl font-black text-slate-800 tracking-tight">Performa RAM</h2>
                    <p class="text-slate-500 font-medium">Metrik kunci operasional hari ini</p>
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
                    <div class="bg-white p-5 rounded-2xl border border-slate-100 shadow-sm" id="stock-card">
                        <p class="text-[10px] font-black text-amber-600 uppercase mb-2 flex items-center gap-1"><i data-lucide="boxes" size="12"></i> Stok di RAM</p>
                        <p class="text-xl font-bold text-slate-800" id="stat-stock">0 <span class="text-xs font-normal">KG</span></p>
                    </div>
                    <div class="bg-emerald-600 p-5 rounded-2xl shadow-lg shadow-emerald-100 text-white">
                        <p class="text-[10px] font-black opacity-70 uppercase mb-2">Laba Bersih</p>
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
                            <input required type="text" id="partyName" placeholder="Nama Petani / PKS" class="w-full px-4 py-3 bg-slate-50 border border-slate-200 rounded-xl outline-none focus:border-emerald-500" />
                            <div class="grid grid-cols-2 gap-4">
                                <input required type="number" id="grossWeight" placeholder="Bruto (KG)" oninput="recalculate()" class="w-full px-4 py-3 bg-slate-50 border border-slate-200 rounded-xl outline-none" />
                                <input type="number" id="deduction" placeholder="Potongan %" oninput="recalculate()" class="w-full px-4 py-3 bg-slate-50 border border-slate-200 rounded-xl outline-none" />
                            </div>
                            <input required type="number" id="price" placeholder="Harga/KG" oninput="recalculate()" class="w-full px-4 py-3 bg-slate-50 border border-slate-200 rounded-xl outline-none" />
                            <div class="bg-emerald-50 p-4 rounded-2xl text-emerald-800 text-sm font-bold flex justify-between">
                                <span>Estimasi Total:</span>
                                <span id="calc-preview">Rp 0</span>
                            </div>
                            <button type="submit" id="submit-btn" class="w-full py-4 rounded-2xl font-black text-white shadow-lg bg-emerald-600 hover:bg-emerald-700 transition-all">SIMPAN DATA</button>
                        </form>
                    </div>

                    <div class="bg-white p-6 rounded-3xl shadow-sm border border-slate-100">
                        <h3 class="text-lg font-bold mb-6 flex items-center gap-2"><i data-lucide="history" class="text-slate-400"></i> Aktivitas RAM</h3>
                        <div id="recent-activities" class="space-y-4 max-h-[400px] overflow-y-auto pr-2 custom-scrollbar">
                            <!-- JS Dynamic -->
                        </div>
                    </div>
                </div>
            </div>

            <!-- Tab: History -->
            <div id="tab-history" class="tab-content hidden max-w-5xl mx-auto space-y-6 animate-in slide-in-from-bottom-4">
                <h2 class="text-2xl font-black">Buku Timbang Lengkap</h2>
                <div class="bg-white rounded-3xl border border-slate-100 shadow-sm overflow-hidden">
                    <div class="overflow-x-auto">
                        <table class="w-full text-left">
                            <thead class="bg-slate-50 text-[10px] font-black text-slate-400 uppercase border-b">
                                <tr>
                                    <th class="px-6 py-4">Tipe</th>
                                    <th class="px-6 py-4">Pihak</th>
                                    <th class="px-6 py-4">Bersih</th>
                                    <th class="px-6 py-4 text-right">Total</th>
                                    <th class="px-6 py-4 text-center">Aksi</th>
                                </tr>
                            </thead>
                            <tbody id="history-table-body" class="divide-y divide-slate-100">
                                <!-- JS Dynamic -->
                            </tbody>
                        </table>
                    </div>
                </div>
            </div>

            <!-- Tab: Expenses -->
            <div id="tab-expenses" class="tab-content hidden max-w-3xl mx-auto space-y-6">
                <header>
                    <h2 class="text-2xl font-black">Biaya Operasional</h2>
                </header>
                <form id="exp-form" class="bg-white p-6 rounded-3xl border border-slate-100 shadow-sm flex flex-col md:flex-row gap-4">
                    <input required type="text" id="expNote" placeholder="Keterangan Biaya" class="flex-1 px-4 py-3 bg-slate-50 border border-slate-200 rounded-xl outline-none" />
                    <input required type="number" id="expAmount" placeholder="Jumlah (IDR)" class="w-full md:w-48 px-4 py-3 bg-slate-50 border border-slate-200 rounded-xl outline-none" />
                    <button class="bg-slate-900 text-white px-8 py-3 rounded-xl font-bold">Tambah</button>
                </form>
                <div class="bg-white rounded-3xl border border-slate-100 shadow-sm overflow-hidden">
                    <table class="w-full text-left">
                        <tbody id="expense-table-body" class="divide-y divide-slate-100"></tbody>
                    </table>
                </div>
            </div>

            <!-- Tab: Report -->
            <div id="tab-report" class="tab-content hidden max-w-4xl mx-auto space-y-8">
                <div class="bg-white rounded-[2rem] shadow-xl border border-slate-100 p-8 space-y-8">
                    <h2 class="text-center text-3xl font-black">Laporan Laba Rugi</h2>
                    <div id="pnl-content">
                        <!-- JS Dynamic -->
                    </div>
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
        // --- Konfigurasi Firebase ---
        // Variabel __firebase_config disediakan oleh lingkungan eksekusi
        const firebaseConfig = JSON.parse(__firebase_config);
        firebase.initializeApp(firebaseConfig);
        const auth = firebase.auth();
        const db = firebase.firestore();
        const appId = typeof __app_id !== 'undefined' ? __app_id : 'palm-logistik-repo';
        
        // Aktifkan Offline Persistence
        db.enablePersistence().catch((err) => {
            if (err.code == 'failed-precondition') {
                console.warn("Firestore: Multiple tabs open, persistence disabled.");
            } else if (err.code == 'unimplemented') {
                console.warn("Firestore: Browser not supported for persistence.");
            }
        });

        let currentUser = null;
        let currentType = 'pembelian';
        let allTransactions = [];
        let allExpenses = [];

        // Deteksi Status Koneksi
        window.addEventListener('online', () => document.body.classList.remove('is-offline'));
        window.addEventListener('offline', () => document.body.classList.add('is-offline'));
        if (!navigator.onLine) document.body.classList.add('is-offline');

        // --- Autentikasi ---
        const initAuth = async () => {
            if (typeof __initial_auth_token !== 'undefined' && __initial_auth_token) {
                await auth.signInWithCustomToken(__initial_auth_token);
            } else {
                await auth.signInAnonymously();
            }
        };

        auth.onAuthStateChanged(user => {
            currentUser = user;
            if (user) startSync();
        });

        // --- Sinkronisasi Data ---
        const startSync = () => {
            if (!currentUser) return;
            const userPath = `artifacts/${appId}/users/${currentUser.uid}`;
            
            // Listen Transaksi
            db.collection(`${userPath}/tbs_transactions`)
              .onSnapshot({ includeMetadataChanges: true }, snap => {
                allTransactions = snap.docs.map(d => ({id: d.id, ...d.data()}))
                    .sort((a,b) => new Date(b.createdAt) - new Date(a.createdAt));
                updateUI();
                document.getElementById('loading-overlay').classList.add('hidden');
            }, (error) => console.error("Snapshot error:", error));

            // Listen Biaya
            db.collection(`${userPath}/expenses`)
              .onSnapshot({ includeMetadataChanges: true }, snap => {
                allExpenses = snap.docs.map(d => ({id: d.id, ...d.data()}))
                    .sort((a,b) => new Date(b.createdAt) - new Date(a.createdAt));
                updateUI();
            }, (error) => console.error("Snapshot error:", error));
        };

        const formatIDR = (n) => new Intl.NumberFormat('id-ID', { style: 'currency', currency: 'IDR', minimumFractionDigits: 0 }).format(n);

        // --- Update Tampilan ---
        const updateUI = () => {
            const tonIn = allTransactions.filter(t => t.type === 'pembelian').reduce((a, b) => a + (Number(b.netWeight) || 0), 0);
            const tonOut = allTransactions.filter(t => t.type === 'penjualan').reduce((a, b) => a + (Number(b.netWeight) || 0), 0);
            const buyVal = allTransactions.filter(t => t.type === 'pembelian').reduce((a, b) => a + (Number(b.totalPrice) || 0), 0);
            const sellVal = allTransactions.filter(t => t.type === 'penjualan').reduce((a, b) => a + (Number(b.totalPrice) || 0), 0);
            const expVal = allExpenses.reduce((a, b) => a + (Number(b.amount) || 0), 0);
            const stock = tonIn - tonOut;
            const netProfit = (sellVal - buyVal) - expVal;

            document.getElementById('stat-tonase-in').innerHTML = `${tonIn.toLocaleString()} <span class="text-xs font-normal">KG</span>`;
            document.getElementById('stat-tonase-out').innerHTML = `${tonOut.toLocaleString()} <span class="text-xs font-normal">KG</span>`;
            document.getElementById('stat-stock').innerHTML = `${stock.toLocaleString()} <span class="text-xs font-normal">KG</span>`;
            document.getElementById('stat-net-profit').innerText = formatIDR(netProfit);
            document.getElementById('sidebar-stock').innerText = `${stock.toLocaleString()} KG`;

            const recentContainer = document.getElementById('recent-activities');
            recentContainer.innerHTML = allTransactions.slice(0, 10).map(t => `
                <div class="flex items-center justify-between p-4 bg-slate-50 rounded-2xl border border-slate-100">
                    <div class="flex items-center space-x-3">
                        <div class="p-2 rounded-xl ${t.type === 'pembelian' ? 'bg-emerald-100 text-emerald-600' : 'bg-blue-100 text-blue-600'}">
                            <i data-lucide="${t.type === 'pembelian' ? 'arrow-down-right' : 'arrow-up-right'}"></i>
                        </div>
                        <div>
                            <p class="text-sm font-black text-slate-800">${t.partyName}</p>
                            <p class="text-[10px] font-bold text-slate-400 uppercase">${(t.netWeight || 0).toLocaleString()} KG • ${t.type}</p>
                        </div>
                    </div>
                    <p class="text-sm font-black ${t.type === 'pembelian' ? 'text-rose-500' : 'text-emerald-600'}">
                        ${t.type === 'pembelian' ? '-' : '+'}${formatIDR(t.totalPrice || 0)}
                    </p>
                </div>
            `).join('');

            document.getElementById('history-table-body').innerHTML = allTransactions.map(t => `
                <tr class="text-sm">
                    <td class="px-6 py-4"><span class="px-2 py-0.5 rounded text-[10px] font-black uppercase ${t.type === 'pembelian' ? 'bg-emerald-50 text-emerald-600' : 'bg-blue-50 text-blue-600'}">${t.type}</span></td>
                    <td class="px-6 py-4 font-bold">${t.partyName}</td>
                    <td class="px-6 py-4">${(t.netWeight || 0).toLocaleString()} KG</td>
                    <td class="px-6 py-4 text-right font-black">${formatIDR(t.totalPrice || 0)}</td>
                    <td class="px-6 py-4 text-center">
                        <button onclick="deleteDoc('tbs_transactions', '${t.id}')" class="text-slate-300 hover:text-rose-500 transition-colors"><i data-lucide="trash-2" size="16"></i></button>
                    </td>
                </tr>
            `).join('');

            document.getElementById('expense-table-body').innerHTML = allExpenses.map(e => `
                <tr class="hover:bg-slate-50">
                    <td class="px-6 py-4 text-sm font-bold text-slate-700">${e.note}</td>
                    <td class="px-6 py-4 text-right font-black text-rose-500">${formatIDR(e.amount || 0)}</td>
                    <td class="px-6 py-4 text-center">
                        <button onclick="deleteDoc('expenses', '${e.id}')" class="text-slate-300 hover:text-rose-500 transition-colors"><i data-lucide="trash-2" size="16"></i></button>
                    </td>
                </tr>
            `).join('');

            document.getElementById('pnl-content').innerHTML = `
                <div class="space-y-6">
                    <div class="flex justify-between py-2 border-b"><span>Penjualan TBS (PKS)</span><span class="font-bold">${formatIDR(sellVal)}</span></div>
                    <div class="flex justify-between py-2 border-b text-rose-500"><span>Pembelian TBS (Petani)</span><span class="font-bold">(${formatIDR(buyVal)})</span></div>
                    <div class="flex justify-between py-4 bg-slate-50 px-4 rounded-xl font-black"><span>LABA KOTOR (Margin)</span><span>${formatIDR(sellVal - buyVal)}</span></div>
                    <div class="py-2">
                        <p class="text-xs font-black text-slate-400 uppercase mb-2">Beban Operasional</p>
                        ${allExpenses.length > 0 ? allExpenses.map(e => `<div class="flex justify-between text-sm py-1"><span>${e.note}</span><span>${formatIDR(e.amount || 0)}</span></div>`).join('') : '<p class="text-xs text-slate-400 italic">Belum ada biaya</p>'}
                    </div>
                    <div class="flex justify-between py-6 border-t-4 border-double border-slate-200 mt-4 items-center">
                        <span class="text-xl font-black">LABA BERSIH</span>
                        <span class="text-3xl font-black text-emerald-600">${formatIDR(netProfit)}</span>
                    </div>
                </div>
            `;

            lucide.createIcons();
        };

        // --- Logika Interaksi ---
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
                btn.classList.toggle('active-nav', isActive);
                if(btn.classList.contains('nav-btn')) {
                    btn.classList.toggle('bg-emerald-600', isActive);
                    btn.classList.toggle('text-white', isActive);
                    btn.classList.toggle('shadow-lg', isActive);
                }
            });
            lucide.createIcons();
        };

        // Simpan Transaksi
        document.getElementById('trans-form').onsubmit = async (e) => {
            e.preventDefault();
            if (!currentUser) return;

            const bruto = parseFloat(document.getElementById('grossWeight').value);
            const pot = parseFloat(document.getElementById('deduction').value) || 0;
            const net = bruto - (bruto * (pot / 100));
            const price = parseFloat(document.getElementById('price').value);

            const path = `artifacts/${appId}/users/${currentUser.uid}/tbs_transactions`;
            try {
                await db.collection(path).add({
                    partyName: document.getElementById('partyName').value,
                    grossWeight: bruto,
                    deduction: pot,
                    netWeight: net,
                    pricePerKg: price,
                    totalPrice: net * price,
                    type: currentType,
                    createdAt: new Date().toISOString()
                });
                e.target.reset();
                recalculate();
            } catch (err) {
                console.error("Error saving transaction:", err);
            }
        };

        // Simpan Biaya
        document.getElementById('exp-form').onsubmit = async (e) => {
            e.preventDefault();
            if (!currentUser) return;
            
            const path = `artifacts/${appId}/users/${currentUser.uid}/expenses`;
            try {
                await db.collection(path).add({
                    note: document.getElementById('expNote').value,
                    amount: parseFloat(document.getElementById('expAmount').value),
                    createdAt: new Date().toISOString()
                });
                e.target.reset();
            } catch (err) {
                console.error("Error saving expense:", err);
            }
        };

        // Hapus Dokumen
        const deleteDoc = async (coll, id) => {
            if (!currentUser) return;
            const path = `artifacts/${appId}/users/${currentUser.uid}/${coll}`;
            await db.collection(path).doc(id).delete();
        };

        // Start App
        window.onload = () => {
            initAuth();
            lucide.createIcons();
        };
    </script>
</body>
</html>

