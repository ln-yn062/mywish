<!DOCTYPE html>
<html lang="zh-CN">
<head>
    <link rel="manifest" href="manifest.json">
    <meta name="theme-color" content="#3B82F6">
    <meta name="apple-mobile-web-app-capable" content="yes">
    <meta name="apple-mobile-web-app-status-bar-style" content="black-translucent">
    <link rel="apple-touch-icon" href="https://cdn-icons-png.flaticon.com/512/1077/1077035.png">
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>2026 WishFlow</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <script src="https://unpkg.com/lucide@latest"></script>
    <style>
        @import url('https://fonts.googleapis.com/css2?family=Inter:wght@400;600;800&display=swap');
        
        body {
            font-family: 'Inter', -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, sans-serif;
            background-color: #f8fafc;
            margin: 0;
            overscroll-behavior-y: contain;
        }

        .app-container {
            width: 100%;
            min-height: 100vh;
            background-color: #f8fafc;
            position: relative;
            display: flex;
            flex-direction: column;
        }

        .scrollbar-hide::-webkit-scrollbar {
            display: none;
        }

        .wish-card {
            transition: all 0.3s cubic-bezier(0.4, 0, 0.2, 1);
            -webkit-tap-highlight-color: transparent;
        }

        .wish-card:active {
            transform: scale(0.98);
        }

        .details-panel {
            max-height: 0;
            overflow: hidden;
            transition: max-height 0.3s ease-out, opacity 0.3s ease-out;
            opacity: 0;
        }

        .details-panel.expanded {
            max-height: 800px;
            opacity: 1;
            margin-top: 12px;
            padding-top: 12px;
            border-top: 1px dashed rgba(16, 185, 129, 0.2);
        }

        @keyframes slideUp {
            from { transform: translateY(100%); }
            to { transform: translateY(0); }
        }

        .animate-slide-up {
            animation: slideUp 0.3s ease-out;
        }

        .progress-ring {
            transition: stroke-dashoffset 0.6s ease-out;
        }

        input[type="date"]::-webkit-calendar-picker-indicator {
            filter: invert(0.5);
            cursor: pointer;
        }

        .stat-card {
            background: linear-gradient(135deg, #ffffff 0%, #f1f5f9 100%);
        }
    </style>
</head>
<body>

    <div class="app-container" id="app">
        <!-- Header -->
        <div class="px-6 pt-10 pb-4 shrink-0 bg-white/50 backdrop-blur-sm sticky top-0 z-50">
            <div class="flex justify-between items-center mb-1">
                <h1 class="text-2xl font-black text-slate-900 tracking-tight">2026愿望单</h1>
                <div class="bg-slate-200/60 p-1 rounded-xl flex items-center" id="mode-toggle">
                    <button onclick="setMode('display')" id="btn-display" class="px-4 py-1.5 rounded-lg text-xs font-bold transition-all bg-white shadow-sm text-blue-600">展示</button>
                    <button onclick="setMode('edit')" id="btn-edit" class="px-4 py-1.5 rounded-lg text-xs font-bold transition-all text-slate-500">编辑</button>
                </div>
            </div>
            <p class="text-[10px] text-blue-500 font-black uppercase tracking-widest">Aesthetic Flow</p>
        </div>

        <!-- Stats Dashboard -->
        <div id="stats-dashboard" class="px-6 mb-6"></div>

        <!-- Main Content -->
        <div id="main-content" class="flex-1 overflow-y-auto px-6 pb-24 space-y-10 scrollbar-hide"></div>

        <!-- Footer Hint -->
        <div id="footer-hint-container" class="fixed bottom-6 left-0 w-full flex justify-center pointer-events-none z-20">
            <div id="footer-hint" class="bg-slate-900/90 text-white px-6 py-2 rounded-full shadow-xl text-[10px] font-bold tracking-widest uppercase">
                Dreaming 2026
            </div>
        </div>

        <!-- Modal: Record Feedback -->
        <div id="modal-record" class="hidden fixed inset-0 z-[80] flex items-end justify-center bg-black/40 backdrop-blur-sm">
            <div class="bg-white w-full p-8 pb-12 animate-slide-up shadow-2xl rounded-t-[32px]">
                <div class="flex justify-between items-center mb-6">
                    <div>
                        <h3 class="text-lg font-bold text-slate-800">记录此刻</h3>
                        <p id="record-modal-subtitle" class="text-xs text-slate-400 font-medium mt-1">定格这份努力</p>
                    </div>
                    <button onclick="closeRecordModal()" class="p-2 bg-slate-100 rounded-full text-slate-400"><i data-lucide="x" class="w-5 h-5"></i></button>
                </div>
                <div class="space-y-6">
                    <div>
                        <label class="text-[10px] font-bold text-slate-400 uppercase tracking-widest block mb-2 px-1">打卡日期</label>
                        <input type="date" id="input-record-date" class="w-full bg-slate-50 p-4 rounded-xl text-base font-semibold outline-none focus:ring-2 ring-emerald-100 border border-transparent">
                    </div>
                    <div>
                        <label class="text-[10px] font-bold text-slate-400 uppercase tracking-widest block mb-2 px-1">感悟与反馈</label>
                        <textarea id="input-record-feedback" placeholder="此刻的心情或进度反馈..." class="w-full bg-slate-50 p-4 rounded-xl text-base font-medium outline-none focus:ring-2 ring-emerald-100 border border-transparent h-24 resize-none"></textarea>
                    </div>
                    <button onclick="confirmRecord()" class="w-full py-4 bg-emerald-600 text-white rounded-xl font-bold text-base shadow-lg active:scale-95 transition-all">确认保存</button>
                </div>
            </div>
        </div>

        <!-- Modal: Add New Item -->
        <div id="modal-add" class="hidden fixed inset-0 z-[60] flex items-end justify-center bg-black/40 backdrop-blur-sm">
            <div class="bg-white w-full p-8 pb-12 animate-slide-up shadow-2xl rounded-t-[32px]">
                <div class="flex justify-between items-center mb-6">
                    <h3 id="add-modal-title" class="text-lg font-bold text-slate-800">添加愿望</h3>
                    <button onclick="closeAddModal()" class="p-2 bg-slate-100 rounded-full text-slate-400"><i data-lucide="x" class="w-5 h-5"></i></button>
                </div>
                <div class="space-y-6">
                    <div>
                        <label class="text-[10px] font-bold text-slate-400 uppercase tracking-widest block mb-2 px-1">名称</label>
                        <input id="input-item-title" placeholder="想实现什么？" class="w-full bg-slate-50 p-4 rounded-xl text-base font-semibold outline-none focus:ring-2 ring-blue-100 border border-transparent">
                    </div>
                    <div id="multiple-options" class="hidden">
                        <label class="text-[10px] font-bold text-slate-400 uppercase tracking-widest block mb-2 px-1">目标次数</label>
                        <div class="flex items-center gap-4">
                            <input type="number" id="input-item-total" value="10" class="w-20 bg-slate-50 p-3 rounded-xl text-center text-lg font-black text-blue-600 outline-none">
                            <span class="text-slate-400 text-sm font-bold">次努力</span>
                        </div>
                    </div>
                    <button onclick="confirmAddItem()" class="w-full py-4 bg-blue-600 text-white rounded-xl font-bold text-base shadow-lg active:scale-95 transition-all">确认添加</button>
                </div>
            </div>
        </div>
    </div>

    <script>
        let mode = 'display';
        let expandedItems = new Set();
        let categories = [
            {
                id: 1,
                name: "自我提升",
                items: [
                    { id: 101, title: "读完 5 本艺术史", type: "multiple", total: 5, current: 2, completed: false, history: [{ date: '2026-03-01', feedback: '读完了第一章' }, { date: '2026-03-05', feedback: '对古典艺术有了新认识' }] },
                    { id: 102, title: "学会萨尔萨舞", type: "once", completed: false, history: [] }
                ]
            },
            {
                id: 2,
                name: "健康生活",
                items: [
                    { id: 201, title: "早睡早起 30 天", type: "multiple", total: 30, current: 12, completed: false, history: [] },
                    { id: 202, title: "完成一次马拉松", type: "once", completed: true, history: [{date: '2026-01-20', feedback: '人生首马！'}] }
                ]
            }
        ];
        let currentRecordContext = null;
        let currentAddContext = null;

        function init() {
            render();
            lucide.createIcons();
        }

        function setMode(newMode) {
            mode = newMode;
            document.getElementById('btn-display').className = mode === 'display' 
                ? 'px-4 py-1.5 rounded-lg text-xs font-bold transition-all bg-white shadow-sm text-blue-600' 
                : 'px-4 py-1.5 rounded-lg text-xs font-bold transition-all text-slate-500';
            
            document.getElementById('btn-edit').className = mode === 'edit' 
                ? 'px-4 py-1.5 rounded-lg text-xs font-bold transition-all bg-white shadow-sm text-blue-600' 
                : 'px-4 py-1.5 rounded-lg text-xs font-bold transition-all text-slate-500';
            
            document.getElementById('footer-hint').textContent = mode === 'edit' ? 'Editing Mode' : 'Dreaming 2026';
            render();
        }

        function getStartOfWeek() {
            const now = new Date();
            const day = now.getDay();
            const diff = now.getDate() - day + (day === 0 ? -6 : 1); 
            const start = new Date(now.setDate(diff));
            start.setHours(0, 0, 0, 0);
            return start;
        }

        function calculateStats() {
            let totalTargets = 0;
            let currentProgress = 0;
            let weeklyChecks = 0;
            const weekStart = getStartOfWeek();

            categories.forEach(cat => {
                cat.items.forEach(item => {
                    if (item.type === 'once') {
                        totalTargets += 1;
                        if (item.completed) currentProgress += 1;
                    } else {
                        totalTargets += item.total;
                        currentProgress += item.current;
                    }
                    if (item.history) {
                        item.history.forEach(record => {
                            if (new Date(record.date) >= weekStart) weeklyChecks++;
                        });
                    }
                });
            });

            return {
                percentage: totalTargets > 0 ? Math.round((currentProgress / totalTargets) * 100) : 0,
                current: currentProgress,
                total: totalTargets,
                weekly: weeklyChecks
            };
        }

        function renderStats() {
            const statsContainer = document.getElementById('stats-dashboard');
            if (mode === 'edit') {
                statsContainer.innerHTML = '';
                return;
            }
            const stats = calculateStats();
            const offset = 126 * (1 - stats.percentage / 100);
            let weeklyColorClass = stats.weekly > 10 ? "text-amber-500" : (stats.weekly > 5 ? "text-blue-600" : "text-slate-900");

            statsContainer.innerHTML = `
                <div class="stat-card p-5 rounded-[24px] border border-white shadow-sm flex items-center justify-between">
                    <div class="flex items-center gap-4">
                        <div class="relative w-16 h-16 flex items-center justify-center shrink-0">
                            <svg class="absolute w-full h-full rotate-[-90deg]">
                                <circle cx="32" cy="32" r="20" fill="transparent" stroke="#E2E8F0" stroke-width="4"/>
                                <circle cx="32" cy="32" r="20" fill="transparent" stroke="#3B82F6" stroke-width="4" stroke-dasharray="126" stroke-dashoffset="${offset}" stroke-linecap="round" class="progress-ring"/>
                            </svg>
                            <span class="text-[11px] font-black text-blue-600">${stats.percentage}%</span>
                        </div>
                        <div>
                            <p class="text-[10px] font-bold text-slate-400 uppercase tracking-widest mb-0.5">总体完成率</p>
                            <p class="text-sm font-black text-slate-800">${stats.current} <span class="text-slate-300">/</span> ${stats.total}</p>
                        </div>
                    </div>
                    <div class="h-10 w-px bg-slate-200 mx-2"></div>
                    <div class="text-right">
                        <p class="text-[10px] font-bold text-slate-400 uppercase tracking-widest mb-0.5">本周打卡</p>
                        <div class="flex items-baseline justify-end gap-1">
                            <span class="text-2xl font-black ${weeklyColorClass}">${stats.weekly}</span>
                            <span class="text-[10px] font-bold text-slate-300">次</span>
                        </div>
                    </div>
                </div>
            `;
        }

        function render() {
            renderStats();
            const container = document.getElementById('main-content');
            container.innerHTML = '';

            categories.forEach(cat => {
                const section = document.createElement('section');
                
                // Calculate Category Progress
                let catTotal = 0;
                let catCurrent = 0;
                cat.items.forEach(i => {
                    if(i.type === 'once') {
                        catTotal += 1;
                        if(i.completed) catCurrent += 1;
                    } else {
                        catTotal += i.total;
                        catCurrent += i.current;
                    }
                });
                
                const header = document.createElement('div');
                header.className = 'flex justify-between items-center px-2 mb-4';
                if (mode === 'edit') {
                    header.innerHTML = `
                        <div class="flex items-center gap-2 flex-1">
                            <div class="w-1 h-4 bg-blue-500"></div>
                            <input onchange="updateCatName(${cat.id}, this.value)" class="bg-transparent border-b border-slate-200 text-base font-bold text-slate-800 outline-none w-full" value="${cat.name}">
                            <div class="text-xs font-bold text-slate-300 px-2 whitespace-nowrap">${catCurrent}/${catTotal}</div>
                            <button onclick="deleteCat(${cat.id})" class="text-slate-300 hover:text-red-400 p-1"><i data-lucide="trash-2" class="w-4 h-4"></i></button>
                        </div>
                    `;
                } else {
                    header.innerHTML = `
                        <div class="flex items-center justify-between w-full">
                            <h2 class="text-base font-bold text-slate-800 flex items-center gap-2"><div class="w-1 h-4 bg-blue-500"></div>${cat.name}</h2>
                            <span class="text-xs font-black text-slate-300 tracking-tighter bg-slate-100 px-2 py-0.5 rounded-full">${catCurrent}/${catTotal}</span>
                        </div>
                    `;
                }
                section.appendChild(header);

                const list = document.createElement('div');
                list.className = 'space-y-3';
                cat.items.forEach(item => {
                    const isExpanded = expandedItems.has(item.id);
                    const bgColorClass = item.completed ? 'bg-emerald-50/50 border-emerald-100' : 'bg-blue-50/50 border-blue-100';
                    const card = document.createElement('div');
                    card.className = `wish-card ${bgColorClass} p-4 border rounded-[20px] flex flex-col shadow-sm`;
                    
                    const mainRow = document.createElement('div');
                    mainRow.className = 'flex items-center gap-4 w-full cursor-pointer';
                    mainRow.onclick = () => handleItemClick(cat.id, item.id);

                    let iconHtml = '';
                    if (item.type === 'once') {
                        iconHtml = item.completed 
                            ? '<i data-lucide="check-circle-2" class="w-6 h-6 text-emerald-500"></i>' 
                            : '<i data-lucide="circle" class="w-6 h-6 text-blue-300"></i>';
                    } else {
                        const offset = 113 * (1 - item.current / item.total);
                        const ringColor = item.completed ? '#10B981' : '#3B82F6';
                        iconHtml = `<div class="relative w-10 h-10 flex items-center justify-center shrink-0">
                            <svg class="absolute w-full h-full rotate-[-90deg]">
                                <circle cx="20" cy="20" r="18" fill="transparent" stroke="#E2E8F0" stroke-width="3"/>
                                <circle cx="20" cy="20" r="18" fill="transparent" stroke="${ringColor}" stroke-width="3" stroke-dasharray="113" stroke-dashoffset="${offset}" stroke-linecap="round" class="progress-ring"/>
                            </svg>
                            <span class="text-[9px] font-black ${item.completed ? 'text-emerald-600' : 'text-blue-600'}">${item.current || 0}</span>
                        </div>`;
                    }

                    mainRow.innerHTML = `
                        <div class="shrink-0">${iconHtml}</div>
                        <div class="flex-1 min-w-0">
                            <p class="font-bold text-slate-800 truncate ${item.completed ? 'text-emerald-900' : ''}">${item.title}</p>
                            ${item.type === 'multiple' ? `
                                <div class="flex items-center gap-2 mt-1">
                                    <div class="flex-1 h-1 bg-slate-200/50 rounded-full overflow-hidden">
                                        <div class="h-full ${item.completed ? 'bg-emerald-400' : 'bg-blue-400'}" style="width: ${(item.current/item.total)*100}%"></div>
                                    </div>
                                    <!-- 去掉了这里的右侧进度文本统计 -->
                                </div>
                            ` : ''}
                        </div>
                        ${mode === 'edit' ? `<button onclick="event.stopPropagation(); deleteItem(${cat.id}, ${item.id})" class="text-slate-300 hover:text-red-400 p-2"><i data-lucide="trash-2" class="w-4 h-4"></i></button>` : ''}
                        ${mode === 'display' && item.history?.length > 0 ? `<i data-lucide="chevron-down" class="w-4 h-4 text-blue-400 transition-transform ${isExpanded ? 'rotate-180' : ''}"></i>` : ''}
                    `;
                    card.appendChild(mainRow);

                    if (item.history?.length > 0) {
                        const details = document.createElement('div');
                        details.className = `details-panel ${isExpanded ? 'expanded' : ''}`;
                        let historyHtml = item.history.map((h, idx) => `
                            <div class="flex gap-3 mb-4 last:mb-0">
                                <div class="flex flex-col items-center shrink-0">
                                    <div class="w-1.5 h-1.5 rounded-full ${item.completed ? 'bg-emerald-400' : 'bg-blue-400'} mt-1.5"></div>
                                    ${idx !== item.history.length - 1 ? `<div class="w-px flex-1 bg-slate-200/50 my-1"></div>` : ''}
                                </div>
                                <div class="flex-1">
                                    <p class="text-[9px] font-black text-slate-400 tracking-tighter mb-0.5">${h.date}</p>
                                    <p class="text-xs text-slate-600 font-medium">${h.feedback}</p>
                                </div>
                            </div>
                        `).reverse().join('');
                        details.innerHTML = `<div class="px-1 py-2"><h4 class="text-[9px] font-bold text-slate-300 uppercase tracking-widest mb-3">记录历程</h4>${historyHtml}</div>`;
                        card.appendChild(details);
                    }
                    list.appendChild(card);
                });

                if (mode === 'edit') {
                    const addBox = document.createElement('div');
                    addBox.className = 'flex gap-2 pt-2';
                    addBox.innerHTML = `
                        <button onclick="openAddModal(${cat.id}, 'once')" class="flex-1 py-3 bg-white border border-slate-200 rounded-xl flex items-center justify-center gap-2 text-slate-400 text-xs font-bold"><i data-lucide="plus" class="w-4 h-4"></i> 单次</button>
                        <button onclick="openAddModal(${cat.id}, 'multiple')" class="flex-1 py-3 bg-white border border-slate-200 rounded-xl flex items-center justify-center gap-2 text-slate-400 text-xs font-bold"><i data-lucide="plus" class="w-4 h-4"></i> 打卡</button>
                    `;
                    list.appendChild(addBox);
                }
                section.appendChild(list);
                container.appendChild(section);
            });

            if (mode === 'edit') {
                const addCat = document.createElement('button');
                addCat.className = 'w-full py-4 bg-slate-900 text-white rounded-xl font-bold flex items-center justify-center gap-2 active:scale-95 transition-all mb-10';
                addCat.innerHTML = `<i data-lucide="layout-grid" class="w-4 h-4"></i> 添加新分类`;
                addCat.onclick = () => { categories.push({ id: Date.now(), name: "新分类", items: [] }); render(); };
                container.appendChild(addCat);
            }
            lucide.createIcons();
        }

        function handleItemClick(catId, itemId) {
            if (mode === 'edit') return;
            const item = categories.find(c => c.id === catId).items.find(i => i.id === itemId);
            if (item.completed) {
                if (expandedItems.has(itemId)) expandedItems.delete(itemId);
                else expandedItems.add(itemId);
                render();
            } else {
                openRecordModal(catId, itemId);
            }
        }

        function openRecordModal(catId, itemId) {
            currentRecordContext = { catId, itemId };
            const today = new Date().toISOString().split('T')[0];
            document.getElementById('input-record-date').value = today;
            document.getElementById('modal-record').classList.remove('hidden');
            document.getElementById('input-record-feedback').focus();
        }

        function closeRecordModal() {
            document.getElementById('modal-record').classList.add('hidden');
            document.getElementById('input-record-feedback').value = '';
            currentRecordContext = null;
        }

        function confirmRecord() {
            const date = document.getElementById('input-record-date').value;
            const feedback = document.getElementById('input-record-feedback').value || "记录了一次努力。";
            const item = categories.find(c => c.id === currentRecordContext.catId).items.find(i => i.id === currentRecordContext.itemId);
            if (!item.history) item.history = [];
            item.history.push({ date, feedback });
            if (item.type === 'once') {
                item.completed = true;
            } else {
                item.current++;
                if (item.current >= item.total) item.completed = true;
            }
            expandedItems.add(item.id);
            closeRecordModal();
            render();
        }

        function updateCatName(id, val) { categories.find(c => c.id === id).name = val; }
        function deleteItem(catId, itemId) {
            const cat = categories.find(c => c.id === catId);
            cat.items = cat.items.filter(i => i.id !== itemId);
            render();
        }
        function deleteCat(id) { categories = categories.filter(c => c.id !== id); render(); }

        function openAddModal(catId, type) {
            currentAddContext = { catId, type };
            document.getElementById('multiple-options').className = type === 'multiple' ? 'block' : 'hidden';
            document.getElementById('modal-add').classList.remove('hidden');
        }
        function closeAddModal() { document.getElementById('modal-add').classList.add('hidden'); }
        function confirmAddItem() {
            const title = document.getElementById('input-item-title').value;
            if (!title) return;
            const cat = categories.find(c => c.id === currentAddContext.catId);
            cat.items.push({
                id: Date.now(),
                title: title,
                type: currentAddContext.type,
                completed: false,
                current: 0,
                total: parseInt(document.getElementById('input-item-total').value) || 10,
                history: []
            });
            closeAddModal();
            render();
        }

        window.onload = init;
        if ('serviceWorker' in navigator) {
    window.addEventListener('load', () => {
        navigator.serviceWorker.register('./sw.js')
            .then(reg => console.log('PWA Registered!'))
            .catch(err => console.log('PWA Registration Failed', err));
    });
}
    </script>
</body>
</html>
