# meu-financeiro2
M.O.B.2
<!DOCTYPE html>
<html lang="pt-pt">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Dashboard Financeiro Pro</title>
    <!-- Tailwind CSS para Design Moderno -->
    <script src="https://cdn.tailwindcss.com"></script>
    <!-- Chart.js para Gráficos -->
    <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
    <!-- SheetJS para ler Excel -->
    <script src="https://cdnjs.cloudflare.com/ajax/libs/xlsx/0.18.5/xlsx.full.min.js"></script>
    <!-- Lucide Icons -->
    <script src="https://unpkg.com/lucide@latest"></script>
    <style>
        @import url('https://fonts.googleapis.com/css2?family=Inter:wght@300;400;600;700&display=swap');
        body { font-family: 'Inter', sans-serif; }
        .glass-card {
            background: rgba(255, 255, 255, 0.95);
            backdrop-filter: blur(10px);
            border: 1px solid rgba(229, 231, 235, 1);
        }
        @media (prefers-color-scheme: dark) {
            body { background-color: #0f172a; color: white; }
            .glass-card { background: rgba(30, 41, 59, 0.8); border-color: rgba(51, 65, 85, 1); }
        }
    </style>
</head>
<body class="bg-gray-50 min-h-screen pb-12">

    <!-- Navegação / Header -->
    <nav class="sticky top-0 z-50 glass-card px-6 py-4 mb-8 shadow-sm">
        <div class="max-w-7xl mx-auto flex justify-between items-center">
            <div class="flex items-center gap-2">
                <i data-lucide="wallet" class="text-indigo-600"></i>
                <h1 class="text-xl font-bold tracking-tight">Finanças<span class="text-indigo-600">Smart</span></h1>
            </div>
            <label class="flex items-center gap-2 bg-indigo-600 hover:bg-indigo-700 text-white px-4 py-2 rounded-lg cursor-pointer transition-all shadow-md">
                <i data-lucide="upload-cloud" class="w-4 h-4"></i>
                <span>Importar Excel</span>
                <input type="file" id="excelInput" class="hidden" accept=".xlsx, .xls, .csv">
            </label>
        </div>
    </nav>

    <main class="max-w-7xl mx-auto px-4 sm:px-6 lg:px-8">
        
        <!-- KPIs Principais -->
        <div class="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-4 gap-6 mb-8">
            <div class="glass-card p-6 rounded-2xl shadow-sm">
                <p class="text-sm text-gray-500 mb-1">Total Balanço</p>
                <h3 id="totalBalance" class="text-2xl font-bold">€ 0.00</h3>
                <div class="mt-2 text-xs text-green-500 flex items-center gap-1">
                    <i data-lucide="trending-up" class="w-3 h-3"></i> Atualizado agora
                </div>
            </div>
            <div class="glass-card p-6 rounded-2xl shadow-sm">
                <p class="text-sm text-gray-500 mb-1">Business</p>
                <h3 id="businessTotal" class="text-2xl font-bold text-blue-600">€ 0.00</h3>
            </div>
            <div class="glass-card p-6 rounded-2xl shadow-sm">
                <p class="text-sm text-gray-500 mb-1">Bad Decisions</p>
                <h3 id="badTotal" class="text-2xl font-bold text-red-500">€ 0.00</h3>
            </div>
            <div class="glass-card p-6 rounded-2xl shadow-sm">
                <p class="text-sm text-gray-500 mb-1">Investimentos</p>
                <h3 id="investTotal" class="text-2xl font-bold text-emerald-600">€ 0.00</h3>
            </div>
        </div>

        <!-- Gráficos -->
        <div class="grid grid-cols-1 lg:grid-cols-2 gap-8 mb-8">
            <div class="glass-card p-6 rounded-2xl shadow-sm">
                <h4 class="font-semibold mb-4 flex items-center gap-2">
                    <i data-lucide="pie-chart" class="w-4 h-4 text-indigo-500"></i>
                    Distribuição por Categoria
                </h4>
                <canvas id="categoryChart" height="250"></canvas>
            </div>
            <div class="glass-card p-6 rounded-2xl shadow-sm">
                <h4 class="font-semibold mb-4 flex items-center gap-2">
                    <i data-lucide="bar-chart-3" class="w-4 h-4 text-indigo-500"></i>
                    Fluxo Mensal
                </h4>
                <canvas id="monthlyChart" height="250"></canvas>
            </div>
        </div>

        <!-- Tabela de Dados -->
        <div class="glass-card rounded-2xl overflow-hidden shadow-sm">
            <div class="px-6 py-4 border-b border-gray-100 dark:border-slate-700 flex justify-between items-center">
                <h4 class="font-semibold">Transações Recentes</h4>
                <div id="statusMessage" class="text-xs text-gray-400 italic">Aguardando ficheiro...</div>
            </div>
            <div class="overflow-x-auto">
                <table class="w-full text-left border-collapse">
                    <thead class="bg-gray-50 dark:bg-slate-800/50">
                        <tr>
                            <th class="px-6 py-3 text-xs font-medium text-gray-500 uppercase">Data</th>
                            <th class="px-6 py-3 text-xs font-medium text-gray-500 uppercase">Descrição</th>
                            <th class="px-6 py-3 text-xs font-medium text-gray-500 uppercase">Categoria</th>
                            <th class="px-6 py-3 text-xs font-medium text-gray-500 uppercase text-right">Valor</th>
                        </tr>
                    </thead>
                    <tbody id="transactionTable" class="divide-y divide-gray-100 dark:divide-slate-700">
                        <!-- Linhas geradas dinamicamente -->
                    </tbody>
                </table>
            </div>
        </div>
    </main>

    <script>
        // Inicializar Ícones
        lucide.createIcons();

        let categoryChart, monthlyChart;

        // Listener para Input de Ficheiro
        document.getElementById('excelInput').addEventListener('change', function(e) {
            const file = e.target.files[0];
            if (!file) return;

            const reader = new FileReader();
            reader.onload = function(e) {
                const data = new Uint8Array(e.target.result);
                const workbook = XLSX.read(data, { type: 'array' });
                const firstSheet = workbook.SheetNames[0];
                const jsonData = XLSX.utils.sheet_to_json(workbook.Sheets[firstSheet]);
                
                processData(jsonData);
            };
            reader.readAsArrayBuffer(file);
        });

        function processData(data) {
            const tableBody = document.getElementById('transactionTable');
            tableBody.innerHTML = '';
            
            let stats = {
                balance: 0,
                business: 0,
                bad: 0,
                invest: 0,
                categories: {}
            };

            data.forEach(row => {
                const valor = parseFloat(row.Valor) || 0;
                const cat = (row.Categoria || 'Outros').trim();
                const desc = row.Descrição || 'Sem descrição';
                const dataStr = row.Data || '-';

                // Cálculos
                stats.balance += valor;
                if(cat.toLowerCase().includes('business')) stats.business += valor;
                if(cat.toLowerCase().includes('bad')) stats.bad += valor;
                if(cat.toLowerCase().includes('invest')) stats.invest += valor;

                stats.categories[cat] = (stats.categories[cat] || 0) + Math.abs(valor);

                // Adicionar à Tabela
                const tr = document.createElement('tr');
                tr.className = "hover:bg-gray-50/50 dark:hover:bg-slate-700/30 transition-colors text-sm";
                tr.innerHTML = `
                    <td class="px-6 py-4 whitespace-nowrap">${dataStr}</td>
                    <td class="px-6 py-4 font-medium">${desc}</td>
                    <td class="px-6 py-4">
                        <span class="px-2 py-1 rounded-full text-[10px] font-bold uppercase ${getBadgeColor(cat)}">
                            ${cat}
                        </span>
                    </td>
                    <td class="px-6 py-4 text-right font-mono ${valor < 0 ? 'text-red-500' : 'text-green-500'}">
                        ${valor.toLocaleString('pt-PT', { style: 'currency', currency: 'EUR' })}
                    </td>
                `;
                tableBody.appendChild(tr);
            });

            updateDashboard(stats);
            document.getElementById('statusMessage').innerText = `${data.length} registos importados.`;
        }

        function getBadgeColor(cat) {
            const c = cat.toLowerCase();
            if(c.includes('home')) return 'bg-blue-100 text-blue-600';
            if(c.includes('business')) return 'bg-indigo-100 text-indigo-600';
            if(c.includes('bad')) return 'bg-red-100 text-red-600';
            if(c.includes('invest')) return 'bg-emerald-100 text-emerald-600';
            return 'bg-gray-100 text-gray-600';
        }

        function updateDashboard(stats) {
            // Atualizar KPIs
            document.getElementById('totalBalance').innerText = stats.balance.toLocaleString('pt-PT', { style: 'currency', currency: 'EUR' });
            document.getElementById('businessTotal').innerText = stats.business.toLocaleString('pt-PT', { style: 'currency', currency: 'EUR' });
            document.getElementById('badTotal').innerText = stats.bad.toLocaleString('pt-PT', { style: 'currency', currency: 'EUR' });
            document.getElementById('investTotal').innerText = stats.invest.toLocaleString('pt-PT', { style: 'currency', currency: 'EUR' });

            // Atualizar Gráfico de Pizza
            const ctxPie = document.getElementById('categoryChart').getContext('2d');
            if(categoryChart) categoryChart.destroy();
            categoryChart = new Chart(ctxPie, {
                type: 'doughnut',
                data: {
                    labels: Object.keys(stats.categories),
                    datasets: [{
                        data: Object.values(stats.categories),
                        backgroundColor: ['#4f46e5', '#ef4444', '#10b981', '#f59e0b', '#3b82f6', '#6366f1']
                    }]
                },
                options: { 
                    responsive: true, 
                    maintainAspectRatio: false,
                    plugins: { legend: { position: 'bottom' } }
                }
            });

            // Exemplo de gráfico de barras (simulado)
            const ctxBar = document.getElementById('monthlyChart').getContext('2d');
            if(monthlyChart) monthlyChart.destroy();
            monthlyChart = new Chart(ctxBar, {
                type: 'bar',
                data: {
                    labels: ['Jan', 'Fev', 'Mar', 'Abr', 'Mai'],
                    datasets: [{
                        label: 'Gasto Total',
                        data: [1200, 1900, 3000, 500, 2000],
                        backgroundColor: '#6366f1',
                        borderRadius: 8
                    }]
                },
                options: { responsive: true, maintainAspectRatio: false }
            });
        }
    </script>
</body>
</html>
