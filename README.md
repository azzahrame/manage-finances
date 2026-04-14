# manage-finances
aplikasi mengelola keuangan yang mudah untuk mahasiswa
import React, { useState, useEffect, useMemo } from 'react';
import { 
  Home, 
  Wallet, 
  PieChart, 
  Target, 
  Moon, 
  Sun, 
  Plus, 
  Trash2, 
  LogOut,
  AlertCircle
} from 'lucide-react';

/* ==============================================================================
   STRUKTUR API & DATABASE (NODE.JS + EXPRESS + MONGODB)
   Bagian ini adalah panduan jika Anda ingin membuat backend aslinya.

   1. Skema MongoDB (Mongoose):
      - UserSchema: { name, email, password }
      - TransactionSchema: { userId, type (income/expense), amount, category, date, note }
      - BudgetSchema: { userId, category, limitMonth, year }
      - GoalSchema: { userId, name, targetAmount, currentAmount, deadline }

   2. Struktur Route Express (API Endpoints):
      - POST /api/auth/register      -> Registrasi user
      - POST /api/auth/login         -> Login user & return JWT token
      - GET /api/transactions        -> Ambil semua transaksi user
      - POST /api/transactions       -> Tambah transaksi baru
      - DELETE /api/transactions/:id -> Hapus transaksi
      - GET /api/budgets             -> Ambil anggaran bulanan
      - POST /api/budgets            -> Set anggaran baru
      - GET /api/goals               -> Ambil daftar target tabungan
      - PUT /api/goals/:id/add       -> Tambah saldo ke target tabungan

   3. Contoh Data JSON (Response dari API):
      {
        "id": "t1",
        "type": "expense",
        "amount": 25000,
        "category": "Makanan",
        "date": "2026-04-08",
        "note": "Makan siang di kantin"
      }
============================================================================== */

// --- DATA DUMMY AWAL (Untuk Simulasi) ---
const initialTransactions = [
  { id: 1, type: 'income', amount: 1500000, category: 'Uang Saku', date: '2026-04-01', note: 'Bulanan dari Ortu' },
  { id: 2, type: 'expense', amount: 200000, category: 'Transportasi', date: '2026-04-02', note: 'Isi Bensin & Ojol' },
  { id: 3, type: 'expense', amount: 450000, category: 'Makanan', date: '2026-04-05', note: 'Makan minggu pertama' },
  { id: 4, type: 'expense', amount: 150000, category: 'Pendidikan', date: '2026-04-06', note: 'Beli buku tulis & print tugas' },
];

const initialBudgets = [
  { category: 'Makanan', limit: 800000 },
  { category: 'Transportasi', limit: 300000 },
  { category: 'Pendidikan', limit: 200000 },
  { category: 'Hiburan', limit: 150000 },
];

const initialGoals = [
  { id: 1, name: 'Beli Laptop Baru', target: 5000000, saved: 1500000 },
  { id: 2, name: 'Liburan Akhir Semester', target: 1000000, saved: 200000 },
];

const categories = ['Makanan', 'Transportasi', 'Pendidikan', 'Hiburan', 'Kos', 'Uang Saku', 'Lainnya'];

export default function App() {
  // --- STATE MANAGEMENT ---
  const [isAuthenticated, setIsAuthenticated] = useState(false);
  const [darkMode, setDarkMode] = useState(false);
  const [activeTab, setActiveTab] = useState('dashboard');
  
  const [transactions, setTransactions] = useState(initialTransactions);
  const [budgets, setBudgets] = useState(initialBudgets);
  const [goals, setGoals] = useState(initialGoals);

  // Form States
  const [newTx, setNewTx] = useState({ type: 'expense', amount: '', category: 'Makanan', date: '', note: '' });

  // --- HELPER FUNCTIONS ---
  // Fungsi format mata uang Rupiah
  const formatRp = (angka) => {
    return new Intl.NumberFormat('id-ID', { style: 'currency', currency: 'IDR', minimumFractionDigits: 0 }).format(angka);
  };

  // Kalkulasi Keuangan
  const { totalIncome, totalExpense, balance, expensesByCategory } = useMemo(() => {
    let inc = 0;
    let exp = 0;
    const catMap = {};

    transactions.forEach(t => {
      if (t.type === 'income') {
        inc += t.amount;
      } else {
        exp += t.amount;
        catMap[t.category] = (catMap[t.category] || 0) + t.amount;
      }
    });

    return { 
      totalIncome: inc, 
      totalExpense: exp, 
      balance: inc - exp,
      expensesByCategory: catMap
    };
  }, [transactions]);

  // --- HANDLERS ---
  const handleLogin = (e) => {
    e.preventDefault();
    setIsAuthenticated(true);
  };

  const handleAddTransaction = (e) => {
    e.preventDefault();
    if (!newTx.amount || !newTx.date) return;
    
    const transaction = {
      id: Date.now(),
      ...newTx,
      amount: parseInt(newTx.amount)
    };
    
    setTransactions([transaction, ...transactions].sort((a, b) => new Date(b.date) - new Date(a.date)));
    setNewTx({ type: 'expense', amount: '', category: 'Makanan', date: '', note: '' }); // Reset form
  };

  const handleDeleteTransaction = (id) => {
    setTransactions(transactions.filter(t => t.id !== id));
  };

  const handleAddToGoal = (id, amount) => {
    setGoals(goals.map(g => g.id === id ? { ...g, saved: g.saved + amount } : g));
  };

  // --- KOMPONEN AUTENTIKASI ---
  if (!isAuthenticated) {
    return (
      <div className={`min-h-screen flex items-center justify-center p-4 transition-colors duration-300 ${darkMode ? 'bg-slate-900 text-white' : 'bg-rose-50 text-slate-800'}`}>
        <div className={`w-full max-w-md p-8 rounded-3xl shadow-xl ${darkMode ? 'bg-slate-800' : 'bg-white'}`}>
          <div className="text-center mb-8">
            <h1 className="text-3xl font-bold bg-clip-text text-transparent bg-gradient-to-r from-teal-400 to-indigo-500 mb-2">
              MahaCuan.
            </h1>
            <p className={`text-sm ${darkMode ? 'text-slate-400' : 'text-slate-500'}`}>Atur uang sakumu dengan cerdas</p>
          </div>
          <form onSubmit={handleLogin} className="space-y-4">
            <div>
              <label className="block text-sm mb-1 opacity-80">Email / NIM</label>
              <input type="text" defaultValue="mahasiswa@kampus.ac.id" className={`w-full p-3 rounded-xl border ${darkMode ? 'bg-slate-700 border-slate-600' : 'bg-slate-50 border-slate-200'} focus:outline-none focus:ring-2 focus:ring-teal-400`} />
            </div>
            <div>
              <label className="block text-sm mb-1 opacity-80">Password</label>
              <input type="password" defaultValue="password123" className={`w-full p-3 rounded-xl border ${darkMode ? 'bg-slate-700 border-slate-600' : 'bg-slate-50 border-slate-200'} focus:outline-none focus:ring-2 focus:ring-teal-400`} />
            </div>
            <button type="submit" className="w-full py-3 mt-4 rounded-xl font-semibold text-white bg-gradient-to-r from-teal-400 to-indigo-500 hover:opacity-90 transition-opacity shadow-lg shadow-teal-500/30">
              Masuk
            </button>
            <p className="text-center text-xs mt-4 opacity-60">Belum punya akun? Daftar di sini</p>
          </form>
        </div>
        
        {/* Toggle Dark Mode Float */}
        <button onClick={() => setDarkMode(!darkMode)} className="absolute top-4 right-4 p-3 rounded-full bg-opacity-20 backdrop-blur-md bg-white shadow-sm">
          {darkMode ? <Sun size={20} className="text-yellow-400" /> : <Moon size={20} className="text-slate-600" />}
        </button>
      </div>
    );
  }

  // --- MAIN LAYOUT APP ---
  return (
    <div className={`min-h-screen transition-colors duration-300 pb-20 md:pb-0 md:pl-64 ${darkMode ? 'bg-slate-900 text-slate-100' : 'bg-slate-50 text-slate-800'}`}>
      
      {/* SIDEBAR (Desktop) & BOTTOM NAV (Mobile) */}
      <nav className={`fixed z-50 transition-colors duration-300 ${darkMode ? 'bg-slate-800 border-slate-700' : 'bg-white border-slate-200'} 
        bottom-0 left-0 w-full border-t md:border-t-0 md:border-r md:w-64 md:h-screen md:top-0 md:flex md:flex-col shadow-[0_-4px_20px_-15px_rgba(0,0,0,0.1)] md:shadow-xl`}>
        
        <div className="hidden md:block p-6">
          <h1 className="text-2xl font-bold bg-clip-text text-transparent bg-gradient-to-r from-teal-400 to-indigo-500">MahaCuan.</h1>
          <p className="text-xs opacity-60 mt-1">Student Finance</p>
        </div>

        <div className="flex flex-row justify-around md:flex-col md:gap-2 md:p-4 w-full h-16 md:h-auto items-center md:items-stretch">
          {[
            { id: 'dashboard', icon: Home, label: 'Dashboard' },
            { id: 'transactions', icon: Wallet, label: 'Transaksi' },
            { id: 'budget', icon: PieChart, label: 'Anggaran' },
            { id: 'goals', icon: Target, label: 'Target' },
          ].map(item => (
            <button 
              key={item.id}
              onClick={() => setActiveTab(item.id)}
              className={`flex flex-col md:flex-row items-center justify-center md:justify-start gap-1 md:gap-3 p-2 md:px-4 md:py-3 rounded-xl transition-all ${
                activeTab === item.id 
                  ? `${darkMode ? 'bg-indigo-500/20 text-indigo-400' : 'bg-indigo-50 text-indigo-600 font-medium'}` 
                  : `opacity-60 hover:opacity-100 ${darkMode ? 'hover:bg-slate-700' : 'hover:bg-slate-100'}`
              }`}
            >
              <item.icon size={20} />
              <span className="text-[10px] md:text-sm">{item.label}</span>
            </button>
          ))}
        </div>

        <div className="hidden md:block mt-auto p-4 border-t border-slate-200 dark:border-slate-700">
          <button onClick={() => setIsAuthenticated(false)} className="flex items-center gap-3 w-full p-2 text-rose-500 hover:bg-rose-50 dark:hover:bg-rose-500/10 rounded-xl transition-colors">
            <LogOut size={20} /> <span className="text-sm">Keluar</span>
          </button>
        </div>
      </nav>

      {/* HEADER UTAMA */}
      <header className="sticky top-0 z-40 backdrop-blur-md bg-opacity-80 p-4 md:px-8 md:py-6 flex justify-between items-center">
        <div>
          <h2 className="text-xl md:text-2xl font-bold capitalize">{activeTab === 'transactions' ? 'Pencatatan' : activeTab}</h2>
          <p className="text-sm opacity-60 hidden md:block">Pantau keuanganmu agar tidak boncos akhir bulan.</p>
        </div>
        <div className="flex items-center gap-3">
          <button onClick={() => setDarkMode(!darkMode)} className={`p-2 rounded-full ${darkMode ? 'bg-slate-800' : 'bg-white'} shadow-sm`}>
            {darkMode ? <Sun size={18} className="text-yellow-400" /> : <Moon size={18} className="text-slate-600" />}
          </button>
          <div className="w-10 h-10 rounded-full bg-gradient-to-tr from-teal-300 to-indigo-300 flex items-center justify-center text-white font-bold shadow-md">
            AM
          </div>
        </div>
      </header>

      {/* KONTEN UTAMA */}
      <main className="p-4 md:p-8 max-w-6xl mx-auto space-y-6">
        
        {/* --- VIEW: DASHBOARD --- */}
        {activeTab === 'dashboard' && (
          <div className="animate-in fade-in slide-in-from-bottom-4 duration-500">
            {/* Kartu Saldo */}
            <div className={`p-6 rounded-3xl shadow-lg bg-gradient-to-r ${darkMode ? 'from-indigo-900 to-slate-800' : 'from-indigo-500 to-teal-400'} text-white mb-6 relative overflow-hidden`}>
              <div className="absolute top-0 right-0 p-8 opacity-10">
                <Wallet size={120} />
              </div>
              <p className="text-sm md:text-base opacity-80 mb-1">Total Saldo Saat Ini</p>
              <h3 className="text-3xl md:text-4xl font-bold mb-6">{formatRp(balance)}</h3>
              
              <div className="flex gap-4 md:gap-8">
                <div className="bg-white/10 p-3 rounded-2xl backdrop-blur-sm flex-1">
                  <p className="text-xs opacity-80 mb-1">Pemasukan Bulan Ini</p>
                  <p className="font-semibold text-teal-200">{formatRp(totalIncome)}</p>
                </div>
                <div className="bg-white/10 p-3 rounded-2xl backdrop-blur-sm flex-1">
                  <p className="text-xs opacity-80 mb-1">Pengeluaran Bulan Ini</p>
                  <p className="font-semibold text-rose-200">{formatRp(totalExpense)}</p>
                </div>
              </div>
            </div>

            {/* Visualisasi Sederhana: Pengeluaran per Kategori */}
            <h4 className="font-semibold text-lg mb-4">Pengeluaran Berdasarkan Kategori</h4>
            <div className={`p-5 rounded-3xl ${darkMode ? 'bg-slate-800' : 'bg-white'} shadow-sm space-y-4`}>
              {Object.keys(expensesByCategory).length === 0 ? (
                <p className="text-center opacity-50 py-4">Belum ada data pengeluaran</p>
              ) : (
                Object.entries(expensesByCategory).sort((a,b) => b[1]-a[1]).map(([cat, amount], idx) => {
                  const percentage = ((amount / totalExpense) * 100).toFixed(1);
                  // Variasi warna pastel berdasarkan index
                  const colors = ['bg-rose-400', 'bg-orange-400', 'bg-amber-400', 'bg-teal-400', 'bg-indigo-400'];
                  const colorClass = colors[idx % colors.length];

                  return (
                    <div key={cat}>
                      <div className="flex justify-between text-sm mb-1">
                        <span>{cat}</span>
                        <span className="font-medium">{formatRp(amount)} ({percentage}%)</span>
                      </div>
                      <div className={`w-full h-3 rounded-full ${darkMode ? 'bg-slate-700' : 'bg-slate-100'}`}>
                        <div className={`h-3 rounded-full ${colorClass}`} style={{ width: `${percentage}%` }}></div>
                      </div>
                    </div>
                  )
                })
              )}
            </div>
          </div>
        )}

        {/* --- VIEW: TRANSACTIONS --- */}
        {activeTab === 'transactions' && (
          <div className="grid md:grid-cols-3 gap-6 animate-in fade-in slide-in-from-bottom-4 duration-500">
            
            {/* Form Tambah Transaksi */}
            <div className={`p-6 rounded-3xl md:col-span-1 h-fit ${darkMode ? 'bg-slate-800' : 'bg-white'} shadow-sm`}>
              <h3 className="font-bold mb-4 flex items-center gap-2"><Plus size={18} /> Tambah Data</h3>
              <form onSubmit={handleAddTransaction} className="space-y-4 text-sm">
                
                {/* Switch Tipe */}
                <div className="flex p-1 rounded-xl bg-slate-100 dark:bg-slate-700">
                  <button type="button" onClick={() => setNewTx({...newTx, type: 'expense'})} 
                    className={`flex-1 py-2 rounded-lg text-center transition-all ${newTx.type === 'expense' ? 'bg-white dark:bg-slate-600 shadow-sm font-medium text-rose-500' : 'opacity-60'}`}>
                    Pengeluaran
                  </button>
                  <button type="button" onClick={() => setNewTx({...newTx, type: 'income'})} 
                    className={`flex-1 py-2 rounded-lg text-center transition-all ${newTx.type === 'income' ? 'bg-white dark:bg-slate-600 shadow-sm font-medium text-teal-500' : 'opacity-60'}`}>
                    Pemasukan
                  </button>
                </div>

                <div>
                  <label className="block mb-1 opacity-70">Nominal (Rp)</label>
                  <input type="number" required placeholder="Contoh: 50000" 
                    value={newTx.amount} onChange={(e) => setNewTx({...newTx, amount: e.target.value})}
                    className={`w-full p-3 rounded-xl border ${darkMode ? 'bg-slate-700 border-slate-600' : 'bg-slate-50 border-slate-200'} focus:ring-2 focus:ring-indigo-400 outline-none`} />
                </div>

                <div>
                  <label className="block mb-1 opacity-70">Kategori</label>
                  <select value={newTx.category} onChange={(e) => setNewTx({...newTx, category: e.target.value})}
                    className={`w-full p-3 rounded-xl border ${darkMode ? 'bg-slate-700 border-slate-600' : 'bg-slate-50 border-slate-200'} outline-none`}>
                    {categories.map(c => <option key={c} value={c}>{c}</option>)}
                  </select>
                </div>

                <div>
                  <label className="block mb-1 opacity-70">Tanggal</label>
                  <input type="date" required 
                    value={newTx.date} onChange={(e) => setNewTx({...newTx, date: e.target.value})}
                    className={`w-full p-3 rounded-xl border ${darkMode ? 'bg-slate-700 border-slate-600 [color-scheme:dark]' : 'bg-slate-50 border-slate-200'} outline-none`} />
                </div>

                <div>
                  <label className="block mb-1 opacity-70">Catatan (Opsional)</label>
                  <input type="text" placeholder="Beli bakso..." 
                    value={newTx.note} onChange={(e) => setNewTx({...newTx, note: e.target.value})}
                    className={`w-full p-3 rounded-xl border ${darkMode ? 'bg-slate-700 border-slate-600' : 'bg-slate-50 border-slate-200'} outline-none`} />
                </div>

                <button type="submit" className="w-full py-3 rounded-xl font-semibold text-white bg-indigo-500 hover:bg-indigo-600 transition-colors shadow-lg shadow-indigo-500/30">
                  Simpan Transaksi
                </button>
              </form>
            </div>

            {/* Daftar Riwayat Transaksi */}
            <div className="md:col-span-2 space-y-4">
              <h3 className="font-bold flex items-center justify-between">
                <span>Riwayat Terakhir</span>
                <span className="text-xs font-normal opacity-60">Total data: {transactions.length}</span>
              </h3>
              
              {transactions.length === 0 ? (
                <div className={`p-8 text-center rounded-3xl ${darkMode ? 'bg-slate-800' : 'bg-white'} opacity-60`}>
                  Belum ada transaksi bulan ini.
                </div>
              ) : (
                transactions.map(t => (
                  <div key={t.id} className={`p-4 rounded-2xl flex items-center justify-between shadow-sm hover:shadow-md transition-shadow ${darkMode ? 'bg-slate-800' : 'bg-white'}`}>
                    <div className="flex items-center gap-4">
                      <div className={`p-3 rounded-full ${t.type === 'income' ? 'bg-teal-100 text-teal-600 dark:bg-teal-900/30 dark:text-teal-400' : 'bg-rose-100 text-rose-600 dark:bg-rose-900/30 dark:text-rose-400'}`}>
                        {t.type === 'income' ? <Plus size={20} /> : <Wallet size={20} />}
                      </div>
                      <div>
                        <p className="font-semibold text-sm md:text-base">{t.category}</p>
                        <div className="flex gap-2 text-xs opacity-60 mt-1">
                          <span>{t.date}</span>
                          {t.note && <span>• {t.note}</span>}
                        </div>
                      </div>
                    </div>
                    <div className="flex items-center gap-4">
                      <span className={`font-bold ${t.type === 'income' ? 'text-teal-500' : 'text-rose-500'}`}>
                        {t.type === 'income' ? '+' : '-'}{formatRp(t.amount)}
                      </span>
                      <button onClick={() => handleDeleteTransaction(t.id)} className="p-2 text-slate-400 hover:text-rose-500 transition-colors">
                        <Trash2 size={16} />
                      </button>
                    </div>
                  </div>
                ))
              )}
            </div>
          </div>
        )}

        {/* --- VIEW: BUDGETING --- */}
        {activeTab === 'budget' && (
          <div className="animate-in fade-in slide-in-from-bottom-4 duration-500">
            <div className="flex items-center justify-between mb-6">
              <p className="opacity-80 text-sm">Batas pengeluaran agar tetap hemat.</p>
            </div>
            
            <div className="grid md:grid-cols-2 gap-6">
              {budgets.map(b => {
                const spent = expensesByCategory[b.category] || 0;
                const percentage = Math.min((spent / b.limit) * 100, 100);
                
                // Logika peringatan warna
                let barColor = 'bg-teal-400';
                let alertMsg = null;
                if (percentage >= 100) {
                  barColor = 'bg-rose-500';
                  alertMsg = "Anggaran terlampaui!";
                } else if (percentage >= 80) {
                  barColor = 'bg-amber-400';
                  alertMsg = "Hati-hati, hampir habis.";
                }

                return (
                  <div key={b.category} className={`p-6 rounded-3xl shadow-sm ${darkMode ? 'bg-slate-800' : 'bg-white'}`}>
                    <div className="flex justify-between items-start mb-4">
                      <h4 className="font-bold text-lg">{b.category}</h4>
                      {alertMsg && (
                        <span className="flex items-center gap-1 text-xs font-medium text-rose-500 bg-rose-100 dark:bg-rose-900/30 px-2 py-1 rounded-lg">
                          <AlertCircle size={12} /> {alertMsg}
                        </span>
                      )}
                    </div>
                    
                    <div className="flex justify-between text-sm mb-2">
                      <span className="opacity-70">Terpakai: <span className="font-semibold text-slate-900 dark:text-white">{formatRp(spent)}</span></span>
                      <span className="opacity-70">Batas: {formatRp(b.limit)}</span>
                    </div>
                    
                    <div className={`w-full h-4 rounded-full overflow-hidden ${darkMode ? 'bg-slate-700' : 'bg-slate-100'}`}>
                      <div className={`h-full transition-all duration-1000 ${barColor}`} style={{ width: `${percentage}%` }}></div>
                    </div>
                    <p className="text-right text-xs mt-2 opacity-50">{percentage.toFixed(0)}% digunakan</p>
                  </div>
                )
              })}
            </div>
          </div>
        )}

        {/* --- VIEW: GOALS (Tabungan) --- */}
        {activeTab === 'goals' && (
          <div className="animate-in fade-in slide-in-from-bottom-4 duration-500 space-y-6">
            
            {goals.map(g => {
              const progress = Math.min((g.saved / g.target) * 100, 100);
              return (
                <div key={g.id} className={`p-6 rounded-3xl shadow-sm border-l-4 border-indigo-400 ${darkMode ? 'bg-slate-800' : 'bg-white'}`}>
                  <div className="flex flex-col md:flex-row md:items-center justify-between gap-4 mb-4">
                    <div>
                      <h4 className="text-xl font-bold">{g.name}</h4>
                      <p className="text-sm opacity-60 mt-1">Terkumpul: {formatRp(g.saved)} / {formatRp(g.target)}</p>
                    </div>
                    
                    {/* Tombol Simulas Tambah Tabungan */}
                    <button 
                      onClick={() => handleAddToGoal(g.id, 50000)}
                      disabled={progress >= 100}
                      className="px-4 py-2 bg-indigo-100 text-indigo-600 dark:bg-indigo-900/30 dark:text-indigo-300 rounded-xl text-sm font-semibold hover:bg-indigo-200 transition-colors disabled:opacity-50"
                    >
                      + Rp 50.000
                    </button>
                  </div>

                  <div className="relative pt-2">
                    <div className={`w-full h-4 rounded-full overflow-hidden ${darkMode ? 'bg-slate-700' : 'bg-slate-100'}`}>
                      <div className="h-full bg-gradient-to-r from-indigo-400 to-teal-400 transition-all duration-1000" style={{ width: `${progress}%` }}></div>
                    </div>
                    {/* Icon Target di ujung bar */}
                    <div 
                      className="absolute top-0 -ml-3 mt-1.5 transition-all duration-1000 bg-white dark:bg-slate-800 p-0.5 rounded-full shadow"
                      style={{ left: `${progress}%` }}
                    >
                      <Target size={16} className={progress >= 100 ? 'text-teal-500' : 'text-indigo-400'} />
                    </div>
                  </div>
                  <p className="text-center text-xs mt-3 font-medium opacity-70">
                    {progress >= 100 ? 'Target Tercapai! 🎉' : `Progress: ${progress.toFixed(1)}%`}
                  </p>
                </div>
              )
            })}
          </div>
        )}

      </main>
    </div>
  );
}
