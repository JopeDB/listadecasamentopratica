# listadecasamentopratica
Tem o objetivo de tornar a lista do casamento um momento leve e agradavel para o casal
<!DOCTYPE html>
<html lang="pt-BR">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Checklist da Casa Nova</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
    <script src="https://cdn.jsdelivr.net/npm/chartjs-plugin-datalabels@2.2.0/dist/chartjs-plugin-datalabels.min.js"></script>
    <link rel="preconnect" href="https://fonts.googleapis.com">
    <link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
    <link href="https://fonts.googleapis.com/css2?family=Inter:wght@400;700;900&display=swap" rel="stylesheet">
    <style>
        html { scroll-behavior: smooth; }
        body { font-family: 'Inter', sans-serif; background-color: #f8fafc; }
        .chart-container { position: relative; width: 100%; max-width: 400px; margin: auto; height: 280px; }
        canvas { cursor: pointer; }
        @media (min-width: 768px) { .chart-container { height: 320px; } }
        .loader { border: 4px solid #f3f3f3; border-top: 4px solid #2dd4bf; border-radius: 50%; width: 40px; height: 40px; animation: spin 1s linear infinite; }
        @keyframes spin { 0% { transform: rotate(0deg); } 100% { transform: rotate(360deg); } }
        label.completed { text-decoration: line-through; color: #94a3b8; }
        .highlight-section { animation: highlight-animation 1.5s ease-out; }
        @keyframes highlight-animation {
            0% { box-shadow: 0 0 0 0px rgba(45, 212, 191, 0.5); }
            40% { box-shadow: 0 0 0 12px rgba(45, 212, 191, 0); }
            100% { box-shadow: 0 0 0 0px rgba(45, 212, 191, 0); }
        }
        [contenteditable="true"]:focus { outline: 2px solid #5eead4; background-color: #f0fdfa; border-radius: 4px; padding: 0 4px; }
        .delete-btn, .delete-category-btn, .details-btn { opacity: 0.3; transition: opacity 0.2s ease-in-out; }
        li:hover .delete-btn, .category-card summary:hover .delete-category-btn, li:hover .details-btn { opacity: 1; }
        .item-details { max-height: 0; overflow: hidden; transition: max-height 0.5s ease-out, padding 0.5s ease-out; }
        .item-details.open { max-height: 250px; padding-top: 0.75rem; }
        .priority-alta { border-left: 4px solid #ef4444; }
        .priority-media { border-left: 4px solid #f59e0b; }
        .priority-baixa { border-left: 4px solid #3b82f6; }
        .dragging { opacity: 0.5; border: 2px dashed #2dd4bf; }
        summary::-webkit-details-marker { display: none; }
        summary { list-style: none; }
        details[open] .details-arrow { transform: rotate(180deg); }
        .filter-btn.active { background-color: #14b8a6; color: white; }
        
        @media print {
            body { background-color: white; color: black; }
            header, footer, #dashboard, #ia-sugestoes, #profile, #search-section, .add-item-input, .add-item-btn, .add-category-btn, .delete-btn, .delete-category-btn, .details-btn, .details-arrow, .filter-btn, .sort-btn, #onboarding, .print-hidden, .cursor-move, #confirmation-modal {
                display: none !important;
            }
            main, section, .grid {
                display: block !important;
                gap: 0 !important;
            }
            .category-card, details {
                box-shadow: none !important;
                border: 1px solid #e2e8f0;
                margin-bottom: 1rem;
                page-break-inside: avoid;
            }
            details > div {
                display: block !important;
            }
            details {
                height: auto;
            }
            details > summary { 
                cursor: default !important;
                border-bottom: 1px solid #e2e8f0;
            }
            .item-details { display: none !important; }
            li { page-break-inside: avoid; padding-top: 4px !important; padding-bottom: 4px !important; }
            [contenteditable="true"] { pointer-events: none; border: none; padding: 0; }
            label::before { content: '☐ '; font-family: sans-serif; margin-right: 6px; }
            input[type="checkbox"]:checked + label::before { content: '☑ '; }
            input[type="checkbox"] { display: none !important; }

            body.print-list-view .category-card {
                border: none; box-shadow: none; padding: 0; margin-bottom: 1.5rem;
            }
            body.print-list-view details > summary {
                padding: 0; margin-bottom: 0.5rem; border-bottom: 2px solid black;
            }
            body.print-list-view .category-card h3 {
                font-size: 1.5rem; font-weight: bold;
            }
            body.print-list-view .category-progress-text,
            body.print-list-view .category-progress {
                display: none;
            }
            body.print-list-view ul {
                list-style-type: none; padding-left: 0;
            }
            body.print-list-view li {
                padding: 2px 0 !important; border-bottom: 1px dotted #ccc; display: flex !important;
            }
            body.print-list-view .priority-alta,
            body.print-list-view .priority-media,
            body.print-list-view .priority-baixa {
                border-left: none;
            }
        }
    </style>
</head>
<body class="bg-slate-50 text-slate-800">
    <div id="confirmation-modal" class="fixed inset-0 bg-black bg-opacity-50 z-50 flex justify-center items-center hidden">
        <div class="bg-white rounded-lg p-8 shadow-2xl max-w-sm w-full">
            <h3 class="text-xl font-bold text-slate-800">Confirmar Exclusão</h3>
            <p class="text-slate-600 mt-2">Você tem certeza? Esta ação não pode ser desfeita.</p>
            <div class="mt-6 flex justify-end gap-4">
                <button id="modal-cancel-btn" class="px-4 py-2 bg-slate-200 text-slate-800 rounded-lg hover:bg-slate-300">Cancelar</button>
                <button id="modal-confirm-btn" class="px-4 py-2 bg-rose-600 text-white rounded-lg hover:bg-rose-700">Excluir</button>
            </div>
        </div>
    </div>

    <div class="container mx-auto p-4 md:p-8 max-w-7xl">
        <header class="my-8 md:my-12 flex flex-col sm:flex-row justify-between sm:items-center gap-4 print-hidden">
            <div class="text-center sm:text-left">
                <h1 class="text-4xl md:text-5xl font-black text-teal-900">🏡 Checklist da Casa Nova</h1>
                <p class="mt-2 text-lg md:text-xl text-teal-700">Seu guia completo para montar o lar dos sonhos.</p>
            </div>
            <div class="flex gap-2">
                <button id="print-list-btn" class="bg-slate-600 text-white font-bold py-3 px-6 rounded-lg hover:bg-slate-700 transition-colors duration-300 flex items-center gap-2 self-center sm:self-auto">
                    Imprimir Lista
                </button>
                <button id="print-btn" class="bg-teal-500 text-white font-bold py-3 px-6 rounded-lg hover:bg-teal-600 transition-colors duration-300 flex items-center gap-2 self-center sm:self-auto">
                    Imprimir Cards
                </button>
            </div>
        </header>

        <main>
            <section id="profile" class="mb-12 md:mb-16 print-hidden">
                <div class="bg-white rounded-xl shadow-sm p-6 md:p-8 flex flex-col md:flex-row items-center gap-6">
                    <div class="relative">
                        <img id="profile-pic" src="https://placehold.co/150x150/d1fae5/14b8a6?text=Foto" class="w-36 h-36 rounded-full object-cover border-4 border-teal-200">
                        <label for="upload-pic" class="absolute bottom-0 right-0 bg-teal-500 text-white rounded-full p-2 cursor-pointer hover:bg-teal-600 transition-transform hover:scale-110">
                            <svg xmlns="http://www.w3.org/2000/svg" width="20" height="20" fill="currentColor" viewBox="0 0 16 16"><path d="M15 12a1 1 0 0 1-1 1H2a1 1 0 0 1-1-1V6a1 1 0 0 1 1-1h1.172a3 3 0 0 0 2.12-.879l.83-.828A1 1 0 0 1 6.827 3h2.344a1 1 0 0 1 .707.293l.828.828A3 3 0 0 0 12.828 5H14a1 1 0 0 1 1 1v6zM2 4a2 2 0 0 0-2 2v6a2 2 0 0 0 2 2h12a2 2 0 0 0 2-2V6a2 2 0 0 0-2-2h-1.172a2 2 0 0 1-1.414-.586l-.828-.828A2 2 0 0 0 9.172 2H6.828a2 2 0 0 0-1.414.586l-.828.828A2 2 0 0 1 3.172 4H2z"/><path d="M8 11a2.5 2.5 0 1 1 0-5 2.5 2.5 0 0 1 0 5zm0 1a3.5 3.5 0 1 0 0-7 3.5 3.5 0 0 0 0 7zM3 6.5a.5.5 0 1 1-1 0 .5.5 0 0 1 1 0z"/></svg>
                        </label>
                        <input type="file" id="upload-pic" class="hidden" accept="image/*">
                    </div>
                    <div class="text-center md:text-left">
                        <div class="flex items-center justify-center md:justify-start gap-2 flex-wrap">
                            <h2 id="name1" contenteditable="true" class="text-3xl font-bold text-teal-800 focus:outline-none focus:ring-2 focus:ring-teal-400 rounded px-2">Nome 1</h2>
                            <span class="text-3xl font-bold text-teal-500">&</span>
                            <h2 id="name2" contenteditable="true" class="text-3xl font-bold text-teal-800 focus:outline-none focus:ring-2 focus:ring-teal-400 rounded px-2">Nome 2</h2>
                        </div>
                        <p class="mt-2 text-slate-600">Começando uma nova jornada juntos!</p>
                        <div id="countdown-container" class="mt-4 text-center md:text-left">
                            <div id="countdown-display" class="text-2xl font-bold text-teal-600"></div>
                            <div class="mt-2">
                                <label for="wedding-date" class="text-sm text-slate-500">Data do Casamento:</label>
                                <input type="date" id="wedding-date" class="p-1 border border-slate-300 rounded-md text-sm">
                            </div>
                        </div>
                    </div>
                </div>
            </section>

            <section id="onboarding" class="mb-8 p-4 bg-teal-50 border-l-4 border-teal-500 text-teal-800 rounded-r-lg print-hidden">
                <div class="flex">
                    <div class="py-1"><svg class="fill-current h-6 w-6 text-teal-500 mr-4" xmlns="http://www.w3.org/2000/svg" viewBox="0 0 20 20"><path d="M2.93 17.07A10 10 0 1 1 17.07 2.93 10 10 0 0 1 2.93 17.07zM9 11v4h2v-4H9zm0-4h2v2H9V7z"/></svg></div>
                    <div>
                        <p class="font-bold">Dicas de Uso</p>
                        <p class="text-sm">Arraste as categorias para reordená-las. Clique em ⚙️ para ver os detalhes financeiros de um item. Use os filtros para focar no que é importante!</p>
                    </div>
                    <button id="close-onboarding" class="ml-auto text-2xl font-bold">&times;</button>
                </div>
            </section>
            
            <section id="dashboard" class="mb-12 md:mb-16 print-hidden">
                <details class="bg-white rounded-xl shadow-sm open:ring-1 open:ring-black/5">
                    <summary class="text-xl font-bold text-teal-800 p-6 cursor-pointer flex justify-between items-center">
                        Painel de Controle
                        <span class="text-teal-500 text-2xl transition-transform duration-300 details-arrow">&#9660;</span>
                    </summary>
                    <div class="p-6 border-t">
                        <div id="sumario">
                            <div class="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-4 gap-4">
                                <div class="bg-teal-50 p-4 rounded-lg text-center"><span id="total-items-count" class="text-5xl font-black text-teal-800">0</span><p class="text-lg text-teal-700 font-bold">Itens na Lista</p></div>
                                <div class="bg-rose-50 p-4 rounded-lg text-center"><span id="remaining-items-count" class="text-5xl font-black text-rose-800">0</span><p class="text-lg text-rose-700 font-bold">Itens Pendentes</p></div>
                                <div class="bg-emerald-50 p-4 rounded-lg text-center"><span id="total-gasto-count" class="text-4xl font-black text-emerald-800">R$ 0,00</span><p class="text-lg text-emerald-700 font-bold">Total Gasto</p></div>
                                <div class="bg-amber-50 p-4 rounded-lg text-center"><span id="total-ganho-count" class="text-4xl font-black text-amber-800">R$ 0,00</span><p class="text-lg text-amber-700 font-bold">Total Economizado</p></div>
                            </div>
                            <div class="mt-6 border-t pt-6">
                                <h3 class="text-xl font-bold text-teal-800">Planejador de Orçamento</h3>
                                <div class="mt-4 grid grid-cols-1 md:grid-cols-3 gap-4 items-end">
                                    <div><label for="budget-input" class="block text-sm font-medium text-slate-700">Orçamento Total (R$)</label><input type="number" id="budget-input" class="mt-1 block w-full p-2 border border-slate-300 rounded-lg" placeholder="Ex: 5000"></div>
                                    <div class="md:col-span-2"><div class="w-full bg-slate-200 rounded-full h-8"><div id="budget-progress-bar" class="bg-teal-500 h-8 rounded-full text-white flex items-center justify-center font-bold transition-all duration-500" style="width: 0%;">0%</div></div><p id="budget-feedback" class="text-sm text-slate-600 mt-1 text-center">Defina um orçamento para começar.</p></div>
                                </div>
                            </div>
                        </div>
                        <div class="grid grid-cols-1 lg:grid-cols-2 gap-8 items-center mt-6 border-t pt-6">
                            <div><h3 class="text-xl font-bold text-teal-800 text-center mb-4">Visão Essencial</h3><div class="chart-container"><canvas id="essentialItemsChart"></canvas></div></div>
                            <div><h3 class="text-xl font-bold text-teal-800 text-center mb-4">Visão Completa</h3><div class="chart-container"><canvas id="completeItemsChart"></canvas></div></div>
                        </div>
                    </div>
                </details>
            </section>

            <section id="search-section" class="mb-8 print-hidden">
                <div class="relative">
                    <input type="text" id="search-input" placeholder="Buscar item em todas as listas..." class="w-full p-4 pl-12 text-lg border border-slate-300 rounded-xl shadow-sm focus:ring-2 focus:ring-teal-400 focus:outline-none">
                    <div class="absolute inset-y-0 left-0 pl-4 flex items-center pointer-events-none">
                        <svg class="h-6 w-6 text-slate-400" xmlns="http://www.w3.org/2000/svg" viewBox="0 0 20 20" fill="currentColor"><path fill-rule="evenodd" d="M8 4a4 4 0 100 8 4 4 0 000-8zM2 8a6 6 0 1110.89 3.476l4.817 4.817a1 1 0 01-1.414 1.414l-4.816-4.816A6 6 0 012 8z" clip-rule="evenodd" /></svg>
                    </div>
                </div>
            </section>
            
            <section id="essenciais" class="mb-12 md:mb-16">
                 <div class="flex flex-col md:flex-row justify-between md:items-center mb-6 gap-4">
                    <h2 class="text-3xl font-bold text-teal-800 text-center md:text-left">✅ A Lista Essencial</h2>
                    <div class="flex gap-2 self-center md:self-auto" data-type="essential">
                        <button class="filter-btn text-sm py-2 px-4 bg-white rounded-full shadow-sm" data-filter="all">Todos</button>
                        <button class="filter-btn text-sm py-2 px-4 bg-white rounded-full shadow-sm flex items-center gap-1" data-filter="alta"><span class="w-3 h-3 rounded-full bg-rose-500"></span>Alta</button>
                        <button class="filter-btn text-sm py-2 px-4 bg-white rounded-full shadow-sm flex items-center gap-1" data-filter="media"><span class="w-3 h-3 rounded-full bg-amber-500"></span>Média</button>
                        <button class="filter-btn text-sm py-2 px-4 bg-white rounded-full shadow-sm flex items-center gap-1" data-filter="baixa"><span class="w-3 h-3 rounded-full bg-blue-500"></span>Baixa</button>
                    </div>
                 </div>
                <div id="essential-categories-grid" class="grid grid-cols-1 xl:grid-cols-2 gap-6"></div>
                <div class="mt-6 print-hidden">
                    <button id="add-essential-category-btn" class="add-category-btn w-full bg-teal-500 text-white font-bold py-3 px-6 rounded-lg hover:bg-teal-600 transition-colors duration-300 flex items-center justify-center text-lg">+ Adicionar Categoria Essencial</button>
                </div>
            </section>

            <section id="completa" class="mb-12 md:mb-16">
                 <div class="flex flex-col md:flex-row justify-between md:items-center mb-6 gap-4">
                    <h2 class="text-3xl font-bold text-teal-800 text-center md:text-left">✨ A Lista Completa</h2>
                    <div class="flex gap-2 self-center md:self-auto" data-type="complete">
                        <button class="filter-btn text-sm py-2 px-4 bg-white rounded-full shadow-sm" data-filter="all">Todos</button>
                        <button class="filter-btn text-sm py-2 px-4 bg-white rounded-full shadow-sm flex items-center gap-1" data-filter="alta"><span class="w-3 h-3 rounded-full bg-rose-500"></span>Alta</button>
                        <button class="filter-btn text-sm py-2 px-4 bg-white rounded-full shadow-sm flex items-center gap-1" data-filter="media"><span class="w-3 h-3 rounded-full bg-amber-500"></span>Média</button>
                        <button class="filter-btn text-sm py-2 px-4 bg-white rounded-full shadow-sm flex items-center gap-1" data-filter="baixa"><span class="w-3 h-3 rounded-full bg-blue-500"></span>Baixa</button>
                    </div>
                 </div>
                <div id="complete-categories-grid" class="grid grid-cols-1 xl:grid-cols-2 gap-6"></div>
                <div class="mt-6 print-hidden">
                    <button id="add-complete-category-btn" class="add-category-btn w-full bg-teal-500 text-white font-bold py-3 px-6 rounded-lg hover:bg-teal-600 transition-colors duration-300 flex items-center justify-center text-lg">+ Adicionar Categoria na Lista Completa</button>
                </div>
            </section>

            <section id="tarefas" class="mb-12 md:mb-16">
                <div class="flex flex-col md:flex-row justify-between md:items-center mb-6 gap-4">
                    <h2 class="text-3xl font-bold text-teal-800 text-center md:text-left">📋 Tarefas da Mudança</h2>
                    <div class="flex gap-2 self-center md:self-auto" data-type="tasks">
                         <button class="sort-btn text-sm py-2 px-4 bg-white rounded-full shadow-sm" data-sort="pending">Pendentes</button>
                         <button class="sort-btn text-sm py-2 px-4 bg-white rounded-full shadow-sm" data-sort="alpha">A-Z</button>
                    </div>
                </div>
                <div id="tasks-categories-grid" class="grid grid-cols-1 xl:grid-cols-2 gap-6"></div>
                <div class="mt-6 print-hidden">
                    <button id="add-task-category-btn" class="add-category-btn w-full bg-teal-500 text-white font-bold py-3 px-6 rounded-lg hover:bg-teal-600 transition-colors duration-300 flex items-center justify-center text-lg">+ Adicionar Categoria de Tarefas</button>
                </div>
            </section>

        </main>

        <footer class="text-center mt-12 md:mt-16 py-8 border-t-2 border-teal-200">
            <p class="text-lg font-bold text-teal-800">Parabéns pela nova fase e divirta-se montando seu lar! 💖</p>
        </footer>
    </div>

    <script>
    document.addEventListener('DOMContentLoaded', () => {
        Chart.register(ChartDataLabels);
        const STORAGE_KEY = 'checklistCasaNovaData';
        const ONBOARDING_KEY = 'checklistOnboardingDismissed';
        const pastelPalette = ['#6EE7B7', '#7DD3FC', '#FBCFE8', '#FDE68A', '#A5B4FC', '#C4B5FD', '#A7F3D0', '#FECACA', '#BFDBFE'];
        const darkPastelPalette = ['#047857', '#0369A1', '#9D174D', '#9A3412', '#4338CA', '#6D28D9', '#065F46', '#991B1B', '#1E40AF'];
        const tooltipTitleCallback = (tooltipItems) => {
            const item = tooltipItems[0];
            let label = item.chart.data.labels[item.dataIndex];
            return Array.isArray(label) ? label.join(' ') : label;
        };
        const essentialCtx = document.getElementById('essentialItemsChart').getContext('2d');
        const essentialChart = new Chart(essentialCtx, { type: 'doughnut', data: { labels: [], datasets: [{ data: [], backgroundColor: pastelPalette, borderColor: '#f8fafc', borderWidth: 4 }] }, options: { responsive: true, maintainAspectRatio: false, cutout: '60%', plugins: { legend: { position: 'top' }, tooltip: { callbacks: { title: tooltipTitleCallback } }, datalabels: { formatter: (value) => value, color: 'white', font: { weight: 'bold', size: 16 } } } } });
        const completeCtx = document.getElementById('completeItemsChart').getContext('2d');
        const completeChart = new Chart(completeCtx, { type: 'bar', data: { labels: [], datasets: [{ data: [], backgroundColor: pastelPalette, borderRadius: 6 }] }, options: { responsive: true, maintainAspectRatio: false, indexAxis: 'y', scales: { x: { beginAtZero: true, grid: { display: false } }, y: { grid: { display: false } } }, plugins: { legend: { display: false }, tooltip: { callbacks: { title: tooltipTitleCallback } }, datalabels: { anchor: 'end', align: 'end', offset: 8, color: (context) => darkPastelPalette[context.dataIndex % darkPastelPalette.length], font: { weight: 'bold' } } } } });
        function highlightAndScroll(targetId) { const targetElement = document.getElementById(targetId); if (targetElement) { if(!targetElement.open) { targetElement.open = true; } document.querySelectorAll('.highlight-section').forEach(el => el.classList.remove('highlight-section')); targetElement.classList.add('highlight-section'); targetElement.scrollIntoView({ behavior: 'smooth', block: 'center' }); targetElement.addEventListener('animationend', () => targetElement.classList.remove('highlight-section'), { once: true }); } }
        function setupChartClickHandler(chart, containerId) { chart.options.onClick = (event, elements) => { if (elements.length > 0) { const index = elements[0].index; const container = document.getElementById(containerId); const sections = container.querySelectorAll('.category-card'); if (sections[index]) { highlightAndScroll(sections[index].id); } } }; }
        setupChartClickHandler(essentialChart, 'essential-categories-grid');
        setupChartClickHandler(completeChart, 'complete-categories-grid');
        const mainContainer = document.querySelector('main');
        const essentialGrid = document.getElementById('essential-categories-grid');
        const completeGrid = document.getElementById('complete-categories-grid');
        const tasksGrid = document.getElementById('tasks-categories-grid');
        const budgetInput = document.getElementById('budget-input');
        const budgetProgressBar = document.getElementById('budget-progress-bar');
        const budgetFeedback = document.getElementById('budget-feedback');
        let appData = {};
        let elementToDelete = null;
        let draggedElement = null;
        const initialData = {
            profile: { name1: "Nome 1", name2: "Nome 2", image: "https://placehold.co/150x150/d1fae5/14b8a6?text=Foto", weddingDate: null },
            budget: 0,
            essential: [
                { id: 'cat-ess-1', title: '🛌 Quarto', isOpen: true, items: [ {id: 'item-ess-1-1', text: 'Cama + Colchão', checked: false, tipo: 'Comprado', est: 0, real: 0, priority: 'alta'}, {id: 'item-ess-1-2', text: 'Travesseiros', checked: false, tipo: 'Comprado', est: 0, real: 0, priority: 'alta'}, {id: 'item-ess-1-3', text: 'Jogo de lençol e cobertor', checked: false, tipo: 'Comprado', est: 0, real: 0, priority: 'alta'}, {id: 'item-ess-1-4', text: 'Guarda-roupa ou cômoda', checked: false, tipo: 'Comprado', est: 0, real: 0, priority: 'alta'}, {id: 'item-ess-1-5', text: 'Cabides', checked: false, tipo: 'Comprado', est: 0, real: 0, priority: 'media'}, {id: 'item-ess-1-6', text: 'Cortina ou Blackout', checked: false, tipo: 'Comprado', est: 0, real: 0, priority: 'media'} ] },
                { id: 'cat-ess-2', title: '🍳 Cozinha', isOpen: true, items: [ {id: 'item-ess-2-1', text: 'Geladeira', checked: false, tipo: 'Comprado', est: 0, real: 0, priority: 'alta'}, {id: 'item-ess-2-2', text: 'Fogão ou cooktop + Forno', checked: false, tipo: 'Comprado', est: 0, real: 0, priority: 'alta'}, {id: 'item-ess-2-3', text: 'Micro-ondas', checked: false, tipo: 'Comprado', est: 0, real: 0, priority: 'media'}, {id: 'item-ess-2-4', text: 'Panelas básicas', checked: false, tipo: 'Comprado', est: 0, real: 0, priority: 'alta'}, {id: 'item-ess-2-5', text: 'Kit de utensílios (concha, etc)', checked: false, tipo: 'Comprado', est: 0, real: 0, priority: 'alta'}, {id: 'item-ess-2-6', text: 'Sacos de lixo', checked: false, tipo: 'Comprado', est: 0, real: 0, priority: 'alta'}, {id: 'item-ess-2-7', text: 'Formas de gelo', checked: false, tipo: 'Comprado', est: 0, real: 0, priority: 'baixa'}, {id: 'item-ess-2-8', text: 'Talheres/pratos descartáveis (1º dia)', checked: false, tipo: 'Comprado', est: 0, real: 0, priority: 'alta'}, ] },
                { id: 'cat-ess-3', title: '🚽 Banheiro', isOpen: true, items: [ {id: 'item-ess-3-1', text: 'Toalhas de banho e rosto', checked: false, tipo: 'Comprado', est: 0, real: 0, priority: 'alta'}, {id: 'item-ess-3-2', text: 'Papel higiênico', checked: false, tipo: 'Comprado', est: 0, real: 0, priority: 'alta'}, {id: 'item-ess-3-3', text: 'Sabonete', checked: false, tipo: 'Comprado', est: 0, real: 0, priority: 'alta'}, {id: 'item-ess-3-4', text: 'Shampoo/Condicionador', checked: false, tipo: 'Comprado', est: 0, real: 0, priority: 'alta'}, {id: 'item-ess-3-5', text: 'Pasta e escova de dentes', checked: false, tipo: 'Comprado', est: 0, real: 0, priority: 'alta'}, ]},
                { id: 'cat-ess-4', title: '🧺 Lavanderia', isOpen: true, items: [ {id: 'item-ess-4-1', text: 'Máquina de lavar', checked: false, tipo: 'Comprado', est: 0, real: 0, priority: 'alta'}, {id: 'item-ess-4-2', text: 'Varal e pregadores', checked: false, tipo: 'Comprado', est: 0, real: 0, priority: 'alta'}, {id: 'item-ess-4-3', text: 'Cesto de roupa suja', checked: false, tipo: 'Comprado', est: 0, real: 0, priority: 'media'}, {id: 'item-ess-4-4', text: 'Balde', checked: false, tipo: 'Comprado', est: 0, real: 0, priority: 'media'}, {id: 'item-ess-4-5', text: 'Vassoura, rodo e pá', checked: false, tipo: 'Comprado', est: 0, real: 0, priority: 'alta'} ]},
                { id: 'cat-ess-5', title: '🛋️ Sala', isOpen: true, items: [ {id: 'item-ess-5-1', text: 'Sofá ou cadeiras', checked: false, tipo: 'Comprado', est: 0, real: 0, priority: 'alta'}, {id: 'item-ess-5-2', text: 'Rack ou painel para a TV', checked: false, tipo: 'Comprado', est: 0, real: 0, priority: 'media'} ]},
                { id: 'cat-ess-6', title: '🛠️ Geral (Primeiro Dia)', isOpen: true, items: [ {id: 'item-ess-6-1', text: 'Lâmpadas', checked: false, tipo: 'Comprado', est: 0, real: 0, priority: 'alta'}, {id: 'item-ess-6-2', text: 'Extensão elétrica e adaptadores', checked: false, tipo: 'Comprado', est: 0, real: 0, priority: 'alta'}, {id: 'item-ess-6-3', text: 'Kit de primeiros socorros', checked: false, tipo: 'Comprado', est: 0, real: 0, priority: 'media'}, {id: 'item-ess-6-4', text: 'Roteador Wi-Fi', checked: false, tipo: 'Comprado', est: 0, real: 0, priority: 'alta'}, {id: 'item-ess-6-5', text: 'Rolo de papel toalha', checked: false, tipo: 'Comprado', est: 0, real: 0, priority: 'alta'}, ]}
            ],
            complete: [
                { id: 'cat-com-1', title: '🛋️ Móveis & Conforto', isOpen: true, items: [ {id: 'item-com-1-1', text: 'Mesa de jantar com cadeiras', checked: false, tipo: 'Comprado', est: 0, real: 0, priority: 'media'}, {id: 'item-com-1-2', text: 'Tapetes para os ambientes', checked: false, tipo: 'Comprado', est: 0, real: 0, priority: 'baixa'}, {id: 'item-com-1-3', text: 'Cortinas para as janelas', checked: false, tipo: 'Comprado', est: 0, real: 0, priority: 'media'}, {id: 'item-com-1-4', text: 'Criados-mudos para o quarto', checked: false, tipo: 'Comprado', est: 0, real: 0, priority: 'baixa'}, {id: 'item-com-1-5', text: 'Luminárias de piso ou abajures', checked: false, tipo: 'Comprado', est: 0, real: 0, priority: 'baixa'}, {id: 'item-com-1-6', text: 'Sapateira', checked: false, tipo: 'Comprado', est: 0, real: 0, priority: 'baixa'}, {id: 'item-com-1-7', text: 'Prateleiras extras', checked: false, tipo: 'Comprado', est: 0, real: 0, priority: 'baixa'} ]},
                { id: 'cat-com-2', title: '🔌 Eletros & Utensílios', isOpen: true, items: [ {id: 'item-com-2-1', text: 'Liquidificador', checked: false, tipo: 'Comprado', est: 0, real: 0, priority: 'media'}, {id: 'item-com-2-2', text: 'Batedeira', checked: false, tipo: 'Comprado', est: 0, real: 0, priority: 'baixa'}, {id: 'item-com-2-3', text: 'Sanduicheira', checked: false, tipo: 'Comprado', est: 0, real: 0, priority: 'media'}, {id: 'item-com-2-4', text: 'Cafeteira ou chaleira elétrica', checked: false, tipo: 'Comprado', est: 0, real: 0, priority: 'media'}, {id: 'item-com-2-5', text: 'Filtro de água ou purificador', checked: false, tipo: 'Comprado', est: 0, real: 0, priority: 'media'}, {id: 'item-com-2-6', text: 'Grill ou Chapa', checked: false, tipo: 'Comprado', est: 0, real: 0, priority: 'baixa'}, {id: 'item-com-2-7', text: 'Processador de alimentos', checked: false, tipo: 'Comprado', est: 0, real: 0, priority: 'baixa'} ]},
                { id: 'cat-com-3', title: '🎨 Decoração', isOpen: true, items: [ {id: 'item-com-3-1', text: 'Quadros e pôsteres', checked: false, tipo: 'Comprado', est: 0, real: 0, priority: 'baixa'}, {id: 'item-com-3-2', text: 'Vasos com plantas', checked: false, tipo: 'Comprado', est: 0, real: 0, priority: 'baixa'}, {id: 'item-com-3-3', text: 'Almofadas decorativas', checked: false, tipo: 'Comprado', est: 0, real: 0, priority: 'baixa'}, {id: 'item-com-3-4', text: 'Mantas para o sofá', checked: false, tipo: 'Comprado', est: 0, real: 0, priority: 'baixa'}, {id: 'item-com-3-5', text: 'Porta-retratos', checked: false, tipo: 'Comprado', est: 0, real: 0, priority: 'baixa'} ]},
                { id: 'cat-com-8', title: '💻 Home Office', isOpen: true, items: [ {id: 'item-com-8-1', text: 'Cadeira ergonômica', checked: false, tipo: 'Comprado', est: 0, real: 0, priority: 'alta'}, {id: 'item-com-8-2', text: 'Mesa de trabalho espaçosa', checked: false, tipo: 'Comprado', est: 0, real: 0, priority: 'alta'}, {id: 'item-com-8-3', text: 'Luminária de mesa', checked: false, tipo: 'Comprado', est: 0, real: 0, priority: 'media'}, {id: 'item-com-8-4', text: 'Organizador de cabos', checked: false, tipo: 'Comprado', est: 0, real: 0, priority: 'baixa'}, {id: 'item-com-8-5', text: 'Suporte para monitor/notebook', checked: false, tipo: 'Comprado', est: 0, real: 0, priority: 'media'} ]},
                { id: 'cat-com-9', title: '🐾 Pets', isOpen: true, items: [ {id: 'item-com-9-1', text: 'Caminha', checked: false, tipo: 'Comprado', est: 0, real: 0, priority: 'alta'}, {id: 'item-com-9-2', text: 'Potes de comida e água', checked: false, tipo: 'Comprado', est: 0, real: 0, priority: 'alta'}, {id: 'item-com-9-3', text: 'Caixa de areia / Tapete higiênico', checked: false, tipo: 'Comprado', est: 0, real: 0, priority: 'alta'}, {id: 'item-com-9-4', text: 'Brinquedos', checked: false, tipo: 'Comprado', est: 0, real: 0, priority: 'media'}, {id: 'item-com-9-5', text: 'Coleira e guia para passeios', checked: false, tipo: 'Comprado', est: 0, real: 0, priority: 'media'} ]},
                { id: 'cat-com-10', title: '🌿 Jardim / Varanda', isOpen: true, items: [ {id: 'item-com-10-1', text: 'Mangueira e esguicho', checked: false, tipo: 'Comprado', est: 0, real: 0, priority: 'media'}, {id: 'item-com-10-2', text: 'Ferramentas de jardinagem', checked: false, tipo: 'Comprado', est: 0, real: 0, priority: 'baixa'}, {id: 'item-com-10-3', text: 'Móveis para área externa', checked: false, tipo: 'Comprado', est: 0, real: 0, priority: 'baixa'}, {id: 'item-com-10-4', text: 'Churrasqueira', checked: false, tipo: 'Comprado', est: 0, real: 0, priority: 'media'} ]},
                { id: 'cat-com-11', title: '🎉 Boas-vindas', isOpen: true, items: [ {id: 'item-com-11-1', text: 'Velas aromáticas ou difusor', checked: false, tipo: 'Comprado', est: 0, real: 0, priority: 'baixa'}, {id: 'item-com-11-2', text: 'Caixa de som Bluetooth', checked: false, tipo: 'Comprado', est: 0, real: 0, priority: 'media'}, {id: 'item-com-11-3', text: 'Taças e bebida para comemorar', checked: false, tipo: 'Comprado', est: 0, real: 0, priority: 'alta'}, {id: 'item-com-11-4', text: 'Lanches práticos (1ª noite)', checked: false, tipo: 'Comprado', est: 0, real: 0, priority: 'alta'} ]}
            ],
            tasks: [
                 { id: 'cat-task-1', title: 'Burocracias', isOpen: true, items: [ {id: 'item-task-1-1', text: 'Mudar titularidade das contas (luz, água)', checked: false}, {id: 'item-task-1-2', text: 'Contratar/Agendar internet', checked: false}, {id: 'item-task-1-3', text: 'Registrar novo endereço (bancos, etc)', checked: false} ]},
                 { id: 'cat-task-2', title: 'Logística', isOpen: true, items: [ {id: 'item-task-2-1', text: 'Contratar empresa de mudança/frete', checked: false}, {id: 'item-task-2-2', text: 'Comprar caixas e materiais de embalagem', checked: false}, {id: 'item-task-2-3', text: 'Medir os cômodos e móveis', checked: false}, {id: 'item-task-2-4', text: 'Descongelar geladeira antes da mudança', checked: false} ]}
            ]
        };
        function saveData() { localStorage.setItem(STORAGE_KEY, JSON.stringify(appData)); }
        function loadData() {
            const savedData = localStorage.getItem(STORAGE_KEY);
            appData = savedData ? JSON.parse(savedData) : JSON.parse(JSON.stringify(initialData));
            document.getElementById('profile-pic').src = appData.profile.image;
            document.getElementById('name1').textContent = appData.profile.name1;
            document.getElementById('name2').textContent = appData.profile.name2;
            document.getElementById('wedding-date').value = appData.profile.weddingDate || '';
            renderAll();
        }
        function updateAll() { updateSummary(); updateCharts(); saveData(); }
        function updateSummary() {
            let totalComprado = 0, totalGanhado = 0, totalItems = 0, checkedItems = 0;
            ['essential', 'complete', 'tasks'].forEach(type => { (appData[type] || []).forEach(cat => { (cat.items || []).forEach(item => { totalItems++; if(item.checked) checkedItems++; if(type !== 'tasks'){ const valor = parseFloat(item.real) || 0; if (item.tipo === 'Comprado') totalComprado += valor; else totalGanhado += valor;} }); }); });
            document.getElementById('total-items-count').textContent = totalItems;
            document.getElementById('remaining-items-count').textContent = totalItems - checkedItems;
            document.getElementById('total-gasto-count').textContent = totalComprado.toLocaleString('pt-BR', { style: 'currency', currency: 'BRL' });
            document.getElementById('total-ganho-count').textContent = totalGanhado.toLocaleString('pt-BR', { style: 'currency', currency: 'BRL' });
            budgetInput.value = appData.budget || '';
            const budget = parseFloat(appData.budget) || 0;
            if (budget > 0) {
                const percentage = Math.min((totalComprado / budget) * 100, 100);
                const remaining = budget - totalComprado;
                budgetProgressBar.style.width = `${percentage}%`; budgetProgressBar.textContent = `${percentage.toFixed(0)}%`;
                if (remaining >= 0) { budgetFeedback.textContent = `Você ainda tem ${remaining.toLocaleString('pt-BR', { style: 'currency', currency: 'BRL' })} disponíveis.`; budgetFeedback.className = 'text-sm text-slate-600 mt-1 text-center';
                } else { budgetFeedback.textContent = `Atenção: você ultrapassou o orçamento em ${Math.abs(remaining).toLocaleString('pt-BR', { style: 'currency', currency: 'BRL' })}.`; budgetFeedback.className = 'text-sm text-rose-600 font-bold mt-1 text-center'; }
                budgetProgressBar.classList.toggle('bg-teal-500', percentage <= 80);
                budgetProgressBar.classList.toggle('bg-amber-500', percentage > 80 && percentage < 100);
                budgetProgressBar.classList.toggle('bg-rose-600', percentage >= 100);
            } else { budgetProgressBar.style.width = `0%`; budgetProgressBar.textContent = `0%`; budgetFeedback.textContent = `Defina um orçamento para começar.`; }
        }
        function updateCharts() {
            essentialChart.data.labels = appData.essential.map(c => c.title); essentialChart.data.datasets[0].data = appData.essential.map(c => c.items.length); essentialChart.update();
            completeChart.data.labels = appData.complete.map(c => c.title); completeChart.data.datasets[0].data = appData.complete.map(c => c.items.length); completeChart.update();
        }
        function createItemLi(item, isTask = false) {
            const li = document.createElement('li');
            const checkedClass = item.checked ? 'completed' : '';
            const priorityClass = isTask ? '' : `priority-${item.priority || 'baixa'}`;
            li.className = `flex flex-col py-2 px-3 rounded-md transition-colors duration-200 hover:bg-slate-50 ${priorityClass}`;
            li.dataset.itemId = item.id;
            const itemContent = `<div class="flex items-center flex-1 min-w-0"><input id="${item.id}" type="checkbox" class="h-5 w-5 rounded border-slate-300 text-teal-500 focus:ring-teal-500 flex-shrink-0" ${item.checked ? 'checked' : ''}><label for="${item.id}" class="ml-3 text-slate-700 transition-colors duration-200 truncate ${checkedClass}" contenteditable="true">${item.text}</label></div>`;
            const itemControls = isTask ? `<div class="flex items-center gap-2 flex-shrink-0 self-end"><button class="delete-btn text-rose-500 hover:text-rose-700 font-bold text-xl p-2">×</button></div>` : `<div class="flex items-center gap-2 flex-shrink-0 self-end mt-2 md:mt-0"><button class="details-btn p-2 text-slate-400 hover:text-teal-600">⚙️</button><button class="delete-btn text-rose-500 hover:text-rose-700 font-bold text-xl p-2">×</button></div>`;
            const itemDetails = isTask ? '' : `<div class="item-details w-full pl-8"><div class="flex flex-col gap-2 items-stretch sm:flex-row sm:flex-wrap sm:items-center sm:justify-end"><select class="priority-select text-sm rounded border-slate-300 focus:ring-teal-500 focus:border-teal-500 w-full sm:w-auto"><option value="baixa" ${item.priority === 'baixa' ? 'selected' : ''}>Baixa</option><option value="media" ${item.priority === 'media' ? 'selected' : ''}>Média</option><option value="alta" ${item.priority === 'alta' ? 'selected' : ''}>Alta</option></select><select class="tipo-aquisicao-select text-sm rounded border-slate-300 focus:ring-teal-500 focus:border-teal-500 w-full sm:w-auto"><option ${item.tipo === 'Comprado' ? 'selected' : ''}>Comprado</option><option ${item.tipo === 'Ganhado' ? 'selected' : ''}>Ganhado</option></select><input type="number" value="${item.est || ''}" placeholder="Est." class="w-full sm:w-20 valor-estimado-input text-sm p-1 border rounded border-slate-300"><input type="number" value="${item.real || ''}" placeholder="Real" class="w-full sm:w-20 valor-real-input text-sm p-1 border rounded border-slate-300"></div></div>`;
            li.innerHTML = `<div class="flex w-full justify-between items-center">${itemContent}${itemControls}</div>${itemDetails}`;
            return li;
        }
        function updateCategoryProgress(card) {
            const items = card.querySelectorAll('li');
            const checkedItems = card.querySelectorAll('input[type="checkbox"]:checked');
            const progressText = card.querySelector('.category-progress-text');
            const progressBar = card.querySelector('.category-progress .bg-teal-500');
            if (!progressText) return;
            const total = items.length;
            const checked = checkedItems.length;
            progressText.textContent = `(${checked}/${total})`;
            const percentage = total > 0 ? (checked / total) * 100 : 0;
            if (progressBar) progressBar.style.width = `${percentage}%`;
        }
        function renderCategory(category, container, isTask = false) {
            const card = document.createElement('details');
            card.id = category.id;
            card.className = 'category-card bg-white rounded-xl shadow-sm';
            if (category.isOpen !== false) card.setAttribute('open', '');
            card.setAttribute('draggable', true);
            const summary = document.createElement('summary');
            summary.className = 'p-6 flex justify-between items-center list-none cursor-pointer';
            summary.innerHTML = `<div class="flex items-center gap-3 cursor-move"><h3 class="text-2xl font-bold text-teal-800" contenteditable="true">${category.title}</h3><span class="category-progress-text text-sm font-bold text-slate-500"></span></div><div class="flex items-center"><span class="text-teal-500 text-2xl transition-transform duration-300 details-arrow">&#9660;</span><button class="delete-category-btn text-rose-500 hover:text-rose-700 font-bold text-2xl p-1 ml-2" title="Excluir categoria">×</button></div>`;
            card.appendChild(summary);
            const contentDiv = document.createElement('div');
            contentDiv.className = 'px-6 pb-6 border-t border-slate-200';
            const progress = isTask ? '' : `<div class="category-progress mt-4"><div class="w-full bg-slate-200 rounded-full h-2.5"><div class="bg-teal-500 h-2.5 rounded-full" style="width: 0%;"></div></div></div>`;
            const list = document.createElement('ul'); list.className = 'mt-4 space-y-2';
            (category.items || []).forEach(item => list.appendChild(createItemLi(item, isTask)));
            contentDiv.innerHTML = progress;
            contentDiv.appendChild(list);
            const addItemForm = document.createElement('div');
            addItemForm.className = 'mt-4 flex gap-2';
            addItemForm.innerHTML = `<input type="text" class="add-item-input w-full p-2 border border-slate-300 rounded-lg text-sm" placeholder="Adicionar ${isTask ? 'tarefa' : 'item'}..."><button class="add-item-btn bg-teal-500 text-white font-bold px-4 rounded-lg hover:bg-teal-600">+</button>`;
            contentDiv.appendChild(addItemForm);
            card.appendChild(contentDiv);
            container.appendChild(card);
        }
        function renderAll() {
            essentialGrid.innerHTML = ''; completeGrid.innerHTML = ''; tasksGrid.innerHTML = '';
            appData.essential.forEach(cat => renderCategory(cat, essentialGrid));
            appData.complete.forEach(cat => renderCategory(cat, completeGrid));
            appData.tasks.forEach(cat => renderCategory(cat, tasksGrid, true));
            document.querySelectorAll('.category-card').forEach(updateCategoryProgress);
            updateAll();
        }
        mainContainer.addEventListener('click', (e) => {
            const categoryCard = e.target.closest('.category-card');
            if(e.target.matches('.details-btn')) { e.target.closest('li').querySelector('.item-details').classList.toggle('open'); return; }
            if (!categoryCard) return;
            const categoryId = categoryCard.id;
            const dataType = categoryCard.closest('#essential-categories-grid') ? 'essential' : categoryCard.closest('#complete-categories-grid') ? 'complete' : 'tasks';
            const category = appData[dataType].find(c => c.id === categoryId);
            if (e.target.closest('.delete-btn')) { const itemId = e.target.closest('li').dataset.itemId; category.items = category.items.filter(i => i.id !== itemId); } 
            else if (e.target.closest('.add-item-btn')) { const input = categoryCard.querySelector('.add-item-input'); if (input.value.trim()) { const newItem = { id: 'item-' + crypto.randomUUID(), text: input.value.trim(), checked: false }; if (dataType !== 'tasks') { Object.assign(newItem, {tipo: 'Comprado', est: 0, real: 0, priority: 'baixa'}); } category.items.push(newItem); input.value = ''; } } 
            else if (e.target.closest('.delete-category-btn')) { elementToDelete = { type: dataType, id: categoryId }; document.getElementById('confirmation-modal').classList.remove('hidden'); return; } 
            else { return; }
            renderAll();
        });
        mainContainer.addEventListener('input', (e) => {
            const li = e.target.closest('li'); if (!li) return;
            const categoryCard = e.target.closest('.category-card'); const categoryId = categoryCard.id; const itemId = li.dataset.itemId;
            const dataType = categoryCard.closest('#essential-categories-grid') ? 'essential' : categoryCard.closest('#complete-categories-grid') ? 'complete' : 'tasks';
            const category = appData[dataType].find(c => c.id === categoryId); const item = category.items.find(i => i.id === itemId);
            if (e.target.matches('input[type="checkbox"]')) { item.checked = e.target.checked; li.querySelector('label').classList.toggle('completed', item.checked); } 
            else if (e.target.matches('.tipo-aquisicao-select')) { item.tipo = e.target.value; } 
            else if (e.target.matches('.valor-estimado-input')) { item.est = e.target.value; } 
            else if (e.target.matches('.valor-real-input')) { item.real = e.target.value; } 
            else if (e.target.matches('.priority-select')) { item.priority = e.target.value; li.className = `flex flex-col py-2 px-3 rounded-md transition-colors duration-200 hover:bg-slate-50 priority-${item.priority}`; }
            updateCategoryProgress(categoryCard); updateSummary(); saveData();
        });
        mainContainer.addEventListener('blur', (e) => { if (e.target.matches('[contenteditable="true"]')) { const targetId = e.target.id; if(targetId === 'name1' || targetId === 'name2'){ appData.profile[targetId] = e.target.textContent; saveData(); return; } const categoryCard = e.target.closest('.category-card'); const categoryId = categoryCard.id; const dataType = categoryCard.closest('#essential-categories-grid') ? 'essential' : categoryCard.closest('#complete-categories-grid') ? 'complete' : 'tasks'; const category = appData[dataType].find(c => c.id === categoryId); if (e.target.matches('h3')) { category.title = e.target.textContent; } else if (e.target.matches('label')) { const itemId = e.target.closest('li').dataset.itemId; const item = category.items.find(i => i.id === itemId); item.text = e.target.textContent; } updateCharts(); saveData(); } }, true);
        mainContainer.addEventListener('toggle', (e) => { if (e.target.matches('.category-card')) { const categoryCard = e.target; const categoryId = categoryCard.id; const dataType = categoryCard.closest('#essential-categories-grid') ? 'essential' : categoryCard.closest('#complete-categories-grid') ? 'complete' : 'tasks'; const category = appData[dataType].find(c => c.id === categoryId); if (category) { category.isOpen = categoryCard.open; saveData(); } } }, true);
        budgetInput.addEventListener('input', () => { appData.budget = budgetInput.value; updateSummary(); saveData(); });
        function addCategory(type) { appData[type].push({ id: 'cat-' + crypto.randomUUID(), title: 'Nova Categoria', isOpen: true, items: [] }); renderAll(); }
        document.getElementById('add-essential-category-btn').addEventListener('click', () => addCategory('essential'));
        document.getElementById('add-complete-category-btn').addEventListener('click', () => addCategory('complete'));
        document.getElementById('add-task-category-btn').addEventListener('click', () => addCategory('tasks'));
        const modal = document.getElementById('confirmation-modal');
        document.getElementById('modal-cancel-btn').addEventListener('click', () => { modal.classList.add('hidden'); elementToDelete = null; });
        document.getElementById('modal-confirm-btn').addEventListener('click', () => { if (elementToDelete) { appData[elementToDelete.type] = appData[elementToDelete.type].filter(cat => cat.id !== elementToDelete.id); renderAll(); } modal.classList.add('hidden'); elementToDelete = null; });
        document.getElementById('print-btn').addEventListener('click', () => {
            document.body.classList.remove('print-list-view');
            window.print();
        });
        document.getElementById('print-list-btn').addEventListener('click', () => {
            document.body.classList.add('print-list-view');
            window.print();
            document.body.classList.remove('print-list-view');
        });
        [essentialGrid, completeGrid, tasksGrid].forEach(grid => {
            grid.addEventListener('dragstart', e => { if (e.target.classList.contains('category-card')) { draggedElement = e.target; e.dataTransfer.setData('text/plain', e.target.id); setTimeout(() => e.target.classList.add('dragging'), 0); } });
            grid.addEventListener('dragend', e => { if(draggedElement) draggedElement.classList.remove('dragging'); });
            grid.addEventListener('dragover', e => { e.preventDefault(); const target = e.target.closest('.category-card'); if (target && target !== draggedElement) { const rect = target.getBoundingClientRect(); const next = (e.clientY - rect.top) / rect.height > .5; if(next) { target.parentNode.insertBefore(draggedElement, target.nextSibling); } else { target.parentNode.insertBefore(draggedElement, target); } } });
            grid.addEventListener('drop', e => { e.preventDefault(); if (draggedElement) { draggedElement.classList.remove('dragging'); const dataType = grid.id.includes('essential') ? 'essential' : grid.id.includes('complete') ? 'complete' : 'tasks'; const newOrderIds = [...grid.querySelectorAll('.category-card')].map(card => card.id); appData[dataType].sort((a, b) => newOrderIds.indexOf(a.id) - newOrderIds.indexOf(b.id)); updateAll(); } });
        });
        document.querySelectorAll('.filter-btn, .sort-btn').forEach(btn => {
            btn.addEventListener('click', (e) => {
                const parent = e.target.parentElement;
                parent.querySelectorAll('.active').forEach(b => b.classList.remove('active'));
                e.target.classList.add('active');
                const type = parent.dataset.type;
                const grid = document.getElementById(`${type}-categories-grid`);
                const filter = e.target.dataset.filter;
                if(filter){
                    grid.querySelectorAll('li').forEach(li => {
                        const priority = li.className.match(/priority-(\w+)/);
                        if(filter === 'all' || (priority && priority[1] === filter)) li.style.display = 'flex';
                        else li.style.display = 'none';
                    });
                }
                const sort = e.target.dataset.sort;
                if(sort){
                    grid.querySelectorAll('.category-card').forEach(card => {
                        const list = card.querySelector('ul');
                        const items = Array.from(list.querySelectorAll('li'));
                        items.sort((a, b) => {
                            if(sort === 'alpha'){
                                const textA = a.querySelector('label').textContent.trim().toLowerCase();
                                const textB = b.querySelector('label').textContent.trim().toLowerCase();
                                return textA.localeCompare(textB);
                            } else if(sort === 'pending'){
                                const checkedA = a.querySelector('input').checked;
                                const checkedB = b.querySelector('input').checked;
                                return checkedA - checkedB;
                            }
                        });
                        items.forEach(item => list.appendChild(item));
                    });
                }
            });
        });
        const onboarding = document.getElementById('onboarding');
        if(localStorage.getItem(ONBOARDING_KEY)) onboarding.style.display = 'none';
        document.getElementById('close-onboarding').addEventListener('click', () => {
             onboarding.style.display = 'none';
             localStorage.setItem(ONBOARDING_KEY, 'true');
        });
        const dashboardDetails = document.querySelector('#dashboard details');
        if (window.innerWidth < 768) { dashboardDetails.removeAttribute('open'); }
        const uploadPic = document.getElementById('upload-pic');
        uploadPic.addEventListener('change', (e) => {
            const file = e.target.files[0];
            if (file) {
                const reader = new FileReader();
                reader.onload = (event) => {
                    const imageUrl = event.target.result;
                    document.getElementById('profile-pic').src = imageUrl;
                    appData.profile.image = imageUrl;
                    saveData();
                };
                reader.readAsDataURL(file);
            }
        });
        const weddingDateInput = document.getElementById('wedding-date');
        weddingDateInput.addEventListener('input', (e) => {
            appData.profile.weddingDate = e.target.value;
            saveData();
        });
        function updateCountdown() {
            const countdownDisplay = document.getElementById('countdown-display');
            const weddingDateStr = appData.profile.weddingDate;
            if (!weddingDateStr) {
                countdownDisplay.innerHTML = '💍 Defina a data do casamento!';
                return;
            }
            const weddingDate = new Date(weddingDateStr + "T00:00:00");
            const now = new Date();
            const diff = weddingDate - now;
            if (diff <= 0) {
                countdownDisplay.innerHTML = "🎉 O grande dia chegou!";
                return;
            }
            const days = Math.floor(diff / (1000 * 60 * 60 * 24));
            const hours = Math.floor((diff % (1000 * 60 * 60 * 24)) / (1000 * 60 * 60));
            const minutes = Math.floor((diff % (1000 * 60 * 60)) / (1000 * 60));
            const seconds = Math.floor((diff % (1000 * 60)) / 1000);
            countdownDisplay.innerHTML = `Faltam: <span class="text-rose-500">${days}d ${hours}h ${minutes}m ${seconds}s</span>`;
        }
        setInterval(updateCountdown, 1000);
        const searchInput = document.getElementById('search-input');
        searchInput.addEventListener('input', (e) => {
            const searchTerm = e.target.value.toLowerCase().trim();
            const allGrids = [essentialGrid, completeGrid, tasksGrid];
            allGrids.forEach(grid => {
                grid.querySelectorAll('.category-card').forEach(card => {
                    let categoryHasVisibleItem = false;
                    card.querySelectorAll('li').forEach(item => {
                        const itemText = item.querySelector('label').textContent.toLowerCase();
                        if (itemText.includes(searchTerm)) {
                            item.style.display = 'flex';
                            categoryHasVisibleItem = true;
                        } else {
                            item.style.display = 'none';
                        }
                    });
                    if (categoryHasVisibleItem) {
                        card.style.display = 'block';
                        card.open = true;
                    } else {
                        card.style.display = 'none';
                    }
                });
            });
            if (searchTerm === '') {
                allGrids.forEach(grid => {
                    grid.querySelectorAll('.category-card').forEach(card => {
                        card.style.display = '';
                        const categoryId = card.id;
                        const dataType = grid.id.replace('-categories-grid', '');
                        const categoryData = appData[dataType].find(c => c.id === categoryId);
                        if (categoryData) {
                            card.open = categoryData.isOpen !== false;
                        }
                        card.querySelectorAll('li').forEach(item => {
                            item.style.display = '';
                        });
                    });
                });
                document.querySelectorAll('.filter-btn.active, .sort-btn.active').forEach(btn => btn.click());
            }
        });
        loadData();
    });
    </script>
</body>
</html>
