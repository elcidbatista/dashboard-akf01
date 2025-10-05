<!DOCTYPE html>
<html lang="pt-BR">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Dashboard AKF01 - Dinâmico</title>
    <!-- Carregando Tailwind CSS para Estilização -->
    <script src="https://cdn.jsdelivr.net/npm/tailwindcss@2.2.19/dist/tailwind.min.js"></script>
    <!-- Carregando Chart.js para Visualização de Dados -->
    <script src="https://cdn.jsdelivr.net/npm/chart.js@3.7.0/dist/chart.min.js"></script>
    <!-- Configuração de fontes e cores para Tailwind -->
    <script>
        tailwind.config = {
            theme: {
                extend: {
                    colors: {
                        'primary-light': '#F3F4F6',
                        'card-bg': '#ffffff',
                        'accent-blue': '#1e40af',
                        'accent-green': '#10b981',
                        'dark-text': '#1f2937',
                        'equipamento-color': '#ef4444',
                        'shift-color': '#8b5cf6',
                    },
                    fontFamily: {
                        sans: ['Inter', 'sans-serif'],
                    }
                }
            }
        }
    </script>
    <style>
        /* Estilo para animação de pulso no status durante atualização automática */
        @keyframes pulse-animation {
            0% { opacity: 1; }
            50% { opacity: 0.6; }
            100% { opacity: 1; }
        }
        .animate-pulse-status {
            animation: pulse-animation 1.5s infinite;
        }
    </style>
</head>
<body class="bg-primary-light text-dark-text font-sans p-4 md:p-10">

    <div class="max-w-7xl mx-auto">
        <!-- Título do Dashboard -->
        <header class="mb-10">
            <h1 class="text-4xl font-extrabold text-accent-blue pb-2">
                Painel de Controle de Manutenção AQF / AKF01 (Dinâmico)
            </h1>
            <p id="data-status" class="text-sm text-gray-500 font-semibold mt-1">
                Aguardando dados CSV. Cole os dados brutos da planilha abaixo para iniciar.
            </p>
            <p class="text-gray-500 mt-1">Análise de Quebras (AKF01) e Performance de Reparo (MTTR).</p>
        </header>
        
        <!-- Área de Input do CSV -->
        <section class="bg-card-bg p-6 rounded-xl shadow-lg mb-8 border-l-4 border-yellow-500">
            <h2 class="text-xl font-semibold mb-4 text-yellow-700">Área de Atualização de Dados</h2>
            
            <!-- Opção 1: Colar CSV Manual (Mais Confiável) -->
            <h3 class="text-lg font-medium mb-2 text-dark-text">1. Colar CSV (Método Confiável)</h3>
            <textarea id="csv-input" rows="4" placeholder="Cole aqui o conteúdo BRUTO da sua planilha (CSV) da aba AKF01."
                class="w-full p-3 border border-gray-300 rounded-lg focus:ring-accent-blue focus:border-accent-blue font-mono text-sm mb-4"></textarea>
            
            <button onclick="handleDataLoad()" 
                class="bg-accent-blue hover:bg-blue-700 text-white font-bold py-2 px-6 rounded-lg shadow-md transition duration-200">
                <span id="update-button-text">Atualizar Dashboard Manualmente</span>
            </button>
            
            <hr class="my-6 border-gray-200">

            <!-- Opção 2: URL de Atualização Automática (Experimental) -->
            <h3 class="text-lg font-medium mb-2 text-dark-text">2. Atualização Automática (Experimental)</h3>
            <p class="text-sm text-gray-600 mb-2">Cole o **link de exportação CSV** do Google Sheets (deve terminar em `.../export?format=csv`).</p>
            <!-- A URL de exportação CSV do usuário foi inserida como valor inicial -->
            <input type="url" id="csv-url-input" 
                   value="https://docs.google.com/spreadsheets/d/e/2PACX-1vQIdVL0z50aM7j_5S1gl1-GuQDYajXoI187wdB5HHBuMizH7Cjzi1uMnvRJmr7iPwsEpU0pVL1xRAiG/pub?gid=2048681344&single=true&output=csv"
                   placeholder="Ex: https://docs.google.com/spreadsheets/d/.../export?format=csv"
                   class="w-full p-3 border border-gray-300 rounded-lg focus:ring-accent-blue focus:border-accent-blue font-mono text-sm mb-4">

            <div class="flex items-center space-x-4">
                 <button id="auto-refresh-button" onclick="toggleAutoRefresh()" 
                    class="bg-accent-green hover:bg-green-700 text-white font-bold py-2 px-6 rounded-lg shadow-md transition duration-200">
                    Ativar Atualização a Cada 5 Min
                </button>
                <p id="auto-refresh-status" class="text-sm font-semibold text-gray-500">Status: Inativo.</p>
            </div>

            <p id="error-message" class="text-red-500 mt-4 hidden">Erro ao processar os dados. Verifique se o formato CSV está correto.</p>
            <p class="text-xs text-red-500 mt-2">⚠️ O método de URL pode falhar devido a restrições de segurança (CORS) em alguns navegadores. **O método de colar é sempre o mais garantido.**</p>
        </section>

        <!-- Seção de Cartões KPI - RESTAURADA A APARÊNCIA ANTERIOR (SHADOW-2XL E BORDER-B-8) -->
        <section id="kpi-cards" class="grid grid-cols-2 lg:grid-cols-4 gap-6 mb-12">
            <!-- Cartão 1: Quantidade de Natureza da Quebra -->
            <div class="bg-card-bg p-6 rounded-xl shadow-2xl border-b-8 border-accent-blue transition duration-300 hover:shadow-xl hover:scale-[1.03]">
                <p class="text-sm font-semibold uppercase text-gray-500">Ocorrências Válidas</p>
                <p id="kpi-ocorrencias" class="text-5xl font-extrabold mt-2 text-dark-text">0</p>
                <p class="text-xs text-green-600 mt-1">Total de registros de quebra.</p>
            </div>
            <!-- Cartão 2: Total de Downtime (Min) -->
            <div class="bg-card-bg p-6 rounded-xl shadow-2xl border-b-8 border-red-500 transition duration-300 hover:shadow-xl hover:scale-[1.03]">
                <p class="text-sm font-semibold uppercase text-gray-500">Total Downtime (Min)</p>
                <p id="kpi-downtime" class="text-5xl font-extrabold mt-2 text-dark-text">0</p>
                <p class="text-xs text-red-600 mt-1">Impacto Crítico em Produção.</p>
            </div>
            <!-- Cartão 3: MTTR (Minutos) -->
            <div class="bg-card-bg p-6 rounded-xl shadow-2xl border-b-8 border-accent-green transition duration-300 hover:shadow-xl hover:scale-[1.03]">
                <p class="text-sm font-semibold uppercase text-gray-500">MTTR (Minutos)</p>
                <p id="kpi-mttr" class="text-5xl font-extrabold mt-2 text-dark-text">0.00</p>
                <p class="text-xs text-accent-green mt-1">Tempo Médio para Reparo.</p>
            </div>
            <!-- Cartão 4: Equipamento Crítico -->
            <div class="bg-card-bg p-6 rounded-xl shadow-2xl border-b-8 border-yellow-500 transition duration-300 hover:shadow-xl hover:scale-[1.03]">
                <p class="text-sm font-semibold uppercase text-gray-500">Equipamento Crítico</p>
                <p id="kpi-top-equipamento" class="text-4xl font-bold mt-2 text-dark-text">N/A</p>
                <p id="kpi-top-ocorrencia" class="text-sm mt-1 text-gray-700 italic truncate" title="Nenhuma descrição de falha.">
                    Nenhuma descrição de falha.
                </p>
                <p class="text-xs text-yellow-600 mt-2">Maior Downtime e sua última falha registrada.</p>
            </div>
        </section>

        <!-- Seção de Gráficos (Row 1: 3 Colunas) -->
        <section id="charts" class="grid grid-cols-1 lg:grid-cols-3 gap-6">
            <div class="lg:col-span-1 bg-card-bg p-6 rounded-xl shadow-lg">
                <h3 class="text-xl font-semibold mb-4 text-dark-text">1. Distribuição de Quebras por Natureza</h3>
                <div class="h-64">
                    <canvas id="chartNatureza"></canvas>
                </div>
            </div>
            <div class="lg:col-span-1 bg-card-bg p-6 rounded-xl shadow-lg">
                <h3 class="text-xl font-semibold mb-4 text-dark-text">2. Top 5 Setores por Tempo de Parada</h3>
                <div class="h-64">
                    <canvas id="chartSetores"></canvas>
                </div>
            </div>
            <div class="lg:col-span-1 bg-card-bg p-6 rounded-xl shadow-lg">
                <h3 class="text-xl font-semibold mb-4 text-dark-text">3. Top 5 Mantenedores por Ocorrências</h3>
                <div class="h-64">
                    <canvas id="chartTecnicos"></canvas>
                </div>
            </div>
        </section>

        <!-- Seção de Gráficos (Row 2: 3 Colunas) -->
        <section id="charts-row2" class="grid grid-cols-1 lg:grid-cols-3 gap-6 mt-6">
            <div class="lg:col-span-1 bg-card-bg p-6 rounded-xl shadow-lg">
                <h3 class="text-xl font-semibold mb-4 text-dark-text">4. Tendência Mensal de Falhas (Frequência)</h3>
                <div class="h-80">
                    <canvas id="chartTendencia"></canvas>
                </div>
            </div>
            <div class="lg:col-span-1 bg-card-bg p-6 rounded-xl shadow-lg">
                <h3 class="text-xl font-semibold mb-4 text-dark-text">5. Ocorrências por Turno (Frequência)</h3>
                <div class="h-80">
                    <canvas id="chartTurno"></canvas>
                </div>
            </div>
            <div class="lg:col-span-1 bg-card-bg p-6 rounded-xl shadow-lg">
                <h3 class="text-xl font-semibold mb-4 text-dark-text">6. Top 5 Equipamentos por Downtime (Min)</h3>
                <div class="h-80">
                    <canvas id="chartEquipamentos"></canvas>
                </div>
            </div>
        </section>

    </div>

    <script>
        // Variáveis globais para rastrear o estado dos dados e gráficos
        const dataStatusEl = document.getElementById('data-status');
        const errorMessageEl = document.getElementById('error-message');
        const updateButtonTextEl = document.getElementById('update-button-text');
        
        // Novas Variáveis Globais para Auto Refresh
        let charts = {}; // Objeto para armazenar instâncias de Chart.js
        let autoRefreshInterval = null;
        const refreshIntervalMinutes = 5;
        const autoRefreshStatusEl = document.getElementById('auto-refresh-status');
        const autoRefreshButtonEl = document.getElementById('auto-refresh-button');
        const csvUrlInputEl = document.getElementById('csv-url-input');
        
        // ***************************************************************
        // FUNÇÕES DE UTILIDADE DE DATAS E TEMPOS
        // ***************************************************************
        
        // Função para converter HH:MM:SS para minutos desde o início do dia
        function timeToMinutes(timeStr) {
            if (!timeStr) return 0;
            try {
                const parts = timeStr.trim().split(':');
                if (parts.length < 2) return 0;
                
                const hours = parseInt(parts[0]) || 0;
                const minutes = parseInt(parts[1]) || 0;
                const seconds = parseInt(parts[2]) || 0;

                return hours * 60 + minutes + (seconds / 60);
            } catch (e) {
                return 0;
            }
        }

        // Função para calcular o downtime em minutos entre Hora Inicio e Hora Fim
        function calculateDowntime(startTime, endTime) {
            if (!startTime || !endTime) return 0;

            const startMins = timeToMinutes(startTime);
            const endMins = timeToMinutes(endTime);

            if (startMins === 0 || endMins === 0) return 0;

            let downtime = endMins - startMins;
            // Caso o reparo tenha cruzado a meia-noite
            if (downtime < 0) {
                downtime += (24 * 60); 
            }

            return Math.round(downtime);
        }

        // Função para normalizar cabeçalhos: remove espaços extras, dois pontos e quebras de linha
        function normalizeHeader(header) {
             return header
                .trim()
                .replace(/:\s*$/, '') // Remove trailing colon and spaces (ex: "Setor: ")
                .replace(/\s+/g, ' ') 
                .replace(/\n/g, ' '); 
        }

        // ***************************************************************
        // PARSE E AGREGAÇÃO DE DADOS
        // ***************************************************************

        // Função para parsear o CSV string de forma robusta
        function parseCSV(csvText) {
            let lines = csvText.trim().split('\n').filter(line => line.trim() !== '');

            if (lines.length < 2) {
                console.error("CSV tem menos de duas linhas (cabeçalho + dados).");
                return [];
            }

            const headerLine = lines[0];
            // Remove as aspas duplas, depois divide por vírgula
            const rawHeaders = headerLine.split(',').map(h => h.trim().replace(/"/g, ''));
            const headers = rawHeaders.map(normalizeHeader);
            
            const records = [];
            const dataLines = lines.slice(1);
            // Regex para lidar com campos entre aspas que contém vírgulas (necessário para CSV de Sheets)
            const csvFieldRegex = /(?:"((?:[^"]|"")*)"|([^,]*))(?:,|$)/g;

            dataLines.forEach(line => {
                if (line.trim().split(',').every(cell => cell.trim() === '')) {
                    return; 
                }

                let values = [];
                let match;
                csvFieldRegex.lastIndex = 0; 
                
                while ((match = csvFieldRegex.exec(line)) !== null) {
                    let value = match[1] !== undefined ? match[1].replace(/""/g, '"') : match[2];
                    values.push((value || '').trim());
                    
                    if (match.index === line.length) {
                        break; 
                    }
                }
                
                // Filtro de validação: A linha só é válida se tiver 'NATUREZA DA QUEBRA' preenchida
                const natureValue = values[0] ? values[0].trim() : '';
                if (natureValue === '') {
                    return; 
                }

                let record = {};
                for (let j = 0; j < headers.length; j++) {
                    const key = headers[j];
                    const rawValue = values[j] ? values[j].trim() : ''; 
                    record[key] = rawValue;
                }
                records.push(record);
            });

            console.log("Total de registros válidos processados:", records.length);
            return records;
        }

        // Função para analisar o CSV e agregar os dados necessários
        function processAndAggregateData(csvText) {
            const records = parseCSV(csvText);
            const aggregates = {
                natureza: {},
                setorDowntime: {},
                mantenedorCounts: {},
                tendencia: {},
                turnoCounts: {},
                equipamentoDowntime: {},
                equipamentoOcorrencia: {}, 
                allDowntimes: []
            };
            
            const natureColors = {
                'ELÉTRICO': '#ef4444',
                'MECÂNICO': '#1e40af',
                'AUTOMAÇÃO': '#fbbf24',
                'OUTROS': '#059669'
            };

            const monthNames = ['Jan', 'Fev', 'Mar', 'Abr', 'Mai', 'Jun', 'Jul', 'Ago', 'Set', 'Out', 'Nov', 'Dez'];
            const monthlyTrends = {}; 

            // Mapeamento dos campos baseado nos cabeçalhos exatos (após normalização)
            const FIELD_MAP = {
                NATUREZA: 'NATUREZA DA QUEBRA',
                DATA_INICIO: 'Data Inicio',
                HORA_INICIO: 'Hora Inicio',
                HORA_FIM: 'Hora fim',
                SETOR: 'Setor',
                EQUIPAMENTO: 'Equipamento',
                MANTENEDOR: 'MANTEDEDOR',
                TURNO: 'TURNO',
                OCORRENCIA: 'DESCRIÇÃO DA OCORRÊNCIA'
            };

            records.forEach(record => {
                const nature = (record[FIELD_MAP.NATUREZA] || 'OUTROS').toUpperCase().trim();
                const setor = (record[FIELD_MAP.SETOR] || 'N/A').toUpperCase().trim();
                const equipamento = (record[FIELD_MAP.EQUIPAMENTO] || 'N/A').toUpperCase().trim();
                const ocorrencia = (record[FIELD_MAP.OCORRENCIA] || 'Sem descrição').trim();
                
                // Normaliza o TURNO (ex: 'TURNO 1' vs 'TURNO 1 MANHÃ')
                const turno = (record[FIELD_MAP.TURNO] || 'N/A').toUpperCase().trim().replace(/ MANHÃ| TARDE| NOITE/g, '').trim();
                
                // O campo MANTEDEDOR pode ter múltiplos nomes separados por vírgula
                const mantenedorList = record[FIELD_MAP.MANTENEDOR] ? record[FIELD_MAP.MANTENEDOR].split(/[,;]/).map(m => m.trim()).filter(m => m) : [];
                
                const downtime = calculateDowntime(record[FIELD_MAP.HORA_INICIO], record[FIELD_MAP.HORA_FIM]);

                if (downtime > 0) {
                    aggregates.allDowntimes.push(downtime);
                }

                // Agregações
                aggregates.natureza[nature] = (aggregates.natureza[nature] || 0) + 1;
                aggregates.setorDowntime[setor] = (aggregates.setorDowntime[setor] || 0) + downtime;
                
                mantenedorList.forEach(mantenedor => {
                    mantenedor = mantenedor.toUpperCase().trim();
                    if (mantenedor && mantenedor !== 'N/A' && mantenedor !== 'FINALIZADA' && mantenedor !== 'PENDENTE') { 
                        aggregates.mantenedorCounts[mantenedor] = (aggregates.mantenedorCounts[mantenedor] || 0) + 1;
                    }
                });

                // Lógica de Tendência Mensal
                const dataInicio = record[FIELD_MAP.DATA_INICIO];
                let year, month;

                if (dataInicio) {
                    if (dataInicio.includes('-')) { // Formato YYYY-MM-DD
                        [year, month] = dataInicio.split('-');
                    } else if (dataInicio.includes('/')) { // Formato DD/MM/YYYY
                        const parts = dataInicio.split('/');
                        if (parts.length === 3) {
                            [ , month, year] = parts; 
                        }
                    }

                    const monthKey = parseInt(month);
                    if (!isNaN(monthKey) && year) {
                        const trendKey = `${year}-${String(monthKey).padStart(2, '0')}`; // Chave: YYYY-MM
                        monthlyTrends[trendKey] = (monthlyTrends[trendKey] || 0) + 1;
                    }
                }

                aggregates.turnoCounts[turno] = (aggregates.turnoCounts[turno] || 0) + 1;
                
                // Agregação de Downtime e Ocorrência Crítica
                if (equipamento !== 'N/A' && equipamento !== '') {
                    const currentDowntime = aggregates.equipamentoDowntime[equipamento] || 0;
                    aggregates.equipamentoDowntime[equipamento] = currentDowntime + downtime;

                    const existingRecord = aggregates.equipamentoOcorrencia[equipamento] || { downtime: 0, ocorrencia: 'Sem descrição' };
                    
                    if (downtime > existingRecord.downtime) {
                        aggregates.equipamentoOcorrencia[equipamento] = { downtime: downtime, ocorrencia: ocorrencia };
                    } else if (aggregates.equipamentoDowntime[equipamento] === currentDowntime + downtime && existingRecord.ocorrencia === 'Sem descrição') {
                         aggregates.equipamentoOcorrencia[equipamento] = { downtime: downtime, ocorrencia: ocorrencia };
                    }
                }
            });


            // --- Formatação da Tendência Mensal ---
            
            // Ordena cronologicamente (YYYY-MM)
            const sortedTrendKeys = Object.keys(monthlyTrends).sort();

            const trendLabels = sortedTrendKeys.map(key => {
                const [yearNum, monthNum] = key.split('-');
                return `${monthNames[parseInt(monthNum) - 1]}/${yearNum % 100}`; // Ex: Jan/25
            });
            const trendCounts = sortedTrendKeys.map(key => monthlyTrends[key]);


            // --- Formatação final para Chart.js e KPI ---
            
            const naturezaSorted = Object.entries(aggregates.natureza).sort(([, a], [, b]) => b - a);
            const setoresSorted = Object.entries(aggregates.setorDowntime)
                .filter(([label]) => label !== 'N/A' && label.trim() !== '')
                .sort(([, a], [, b]) => b - a).slice(0, 5);

            const mantenedorSorted = Object.entries(aggregates.mantenedorCounts)
                .filter(([label]) => label !== 'N/A' && label.trim() !== '')
                .sort(([, a], [, b]) => b - a).slice(0, 5);
            
            // LÓGICA ATUALIZADA: Focar apenas nos Turnos 1, 2 e 3 na ordem correta
            const allowedTurnos = ['TURNO 1', 'TURNO 2', 'TURNO 3'];
            const turnoLabels = allowedTurnos.filter(label => aggregates.turnoCounts[label]); 
            const turnoCounts = turnoLabels.map(label => aggregates.turnoCounts[label] || 0);

            const equipamentosSorted = Object.entries(aggregates.equipamentoDowntime)
                .filter(([label]) => label !== 'N/A' && label.trim() !== '')
                .sort(([, a], [, b]) => b - a).slice(0, 5);
            
            // Encontra a ocorrência crítica do Top 1 Equipamento
            const topEquipamentoLabel = equipamentosSorted.length > 0 ? equipamentosSorted[0][0] : null;
            let topEquipamentoOcorrencia = 'Nenhuma descrição registrada.';
            
            if (topEquipamentoLabel) {
                // Percorre todos os registros do top equipamento e pega a ocorrência do registro com maior downtime INDIVIDUAL
                let maxSingleDowntime = 0;
                let criticalOcorrencia = 'Sem descrição';

                records.filter(r => (r[FIELD_MAP.EQUIPAMENTO] || 'N/A').toUpperCase().trim() === topEquipamentoLabel)
                       .forEach(r => {
                           const downtime = calculateDowntime(r[FIELD_MAP.HORA_INICIO], r[FIELD_MAP.HORA_FIM]);
                           if (downtime > maxSingleDowntime) {
                               maxSingleDowntime = downtime;
                               criticalOcorrencia = (r[FIELD_MAP.OCORRENCIA] || 'Sem descrição').trim();
                           }
                       });
                
                if (criticalOcorrencia !== 'Sem descrição') {
                    topEquipamentoOcorrencia = criticalOcorrencia;
                }
            }


            const realData = {
                totalOcorrencias: records.length,
                totalDowntime: aggregates.allDowntimes.reduce((sum, current) => sum + current, 0),

                calculateMTTR: function() {
                    const ocorrenciasComDowntime = aggregates.allDowntimes.length;
                    if (ocorrenciasComDowntime > 0) {
                        return (this.totalDowntime / ocorrenciasComDowntime).toFixed(2);
                    }
                    return '0.00';
                },

                topEquipamentoOcorrencia: topEquipamentoOcorrencia, 

                naturezaData: {
                    labels: naturezaSorted.map(([label]) => label),
                    counts: naturezaSorted.map(([, count]) => count),
                    colors: naturezaSorted.map(([label]) => natureColors[label] || natureColors['OUTROS'])
                },
                setoresData: {
                    labels: setoresSorted.map(([label]) => label),
                    downtime: setoresSorted.map(([, downtime]) => downtime),
                    color: '#1e40af' 
                },
                technicianData: {
                    labels: mantenedorSorted.map(([label]) => label),
                    counts: mantenedorSorted.map(([, count]) => count),
                    color: '#fbbf24'
                },
                tendenciaData: {
                    labels: trendLabels, 
                    counts: trendCounts, 
                    color: '#059669'
                },
                shiftData: {
                    labels: turnoLabels,
                    counts: turnoCounts,
                    color: '#8b5cf6' 
                },
                equipamentoData: {
                    labels: equipamentosSorted.map(([label]) => label),
                    downtime: equipamentosSorted.map(([, downtime]) => downtime),
                    color: '#ef4444' 
                }
            };
            
            return realData;
        }
        
        // ***************************************************************
        // RENDERIZAÇÃO E ATUALIZAÇÃO DO DASHBOARD
        // ***************************************************************

        // Função para destruir gráficos antigos
        function destroyCharts() {
            Object.values(charts).forEach(chart => {
                if (chart) chart.destroy();
            });
            charts = {};
        }

        // Função para renderizar os gráficos e KPIs com os dados processados
        function renderCharts(data) {
            
            destroyCharts(); // Limpa gráficos existentes

            // Verifica se os dados são válidos
            if (!data || data.totalOcorrencias === 0) {
                dataStatusEl.textContent = 'AVISO: Dados carregados, mas não foram encontrados registros válidos para a aba AKF01.';
                dataStatusEl.classList.remove('text-green-600', 'animate-pulse-status');
                dataStatusEl.classList.add('text-yellow-700');
                
                // Zera KPIs se não houver dados
                document.getElementById('kpi-ocorrencias').textContent = '0';
                document.getElementById('kpi-downtime').textContent = '0';
                document.getElementById('kpi-mttr').textContent = '0.00';
                document.getElementById('kpi-top-equipamento').textContent = 'N/A';
                document.getElementById('kpi-top-ocorrencia').textContent = 'Nenhuma descrição de falha.';
                document.getElementById('kpi-top-ocorrencia').title = 'Nenhuma descrição de falha.';
                return; 
            }

            // Atualiza status
            dataStatusEl.textContent = `✅ Dados de ${data.totalOcorrencias.toLocaleString('pt-BR')} ocorrências carregados com sucesso.`;
            dataStatusEl.classList.remove('text-yellow-700');
            dataStatusEl.classList.add('text-green-600');
            
            // Se estiver em modo auto, mantém a pulsação
            if(autoRefreshInterval) {
                 dataStatusEl.classList.add('animate-pulse-status');
            }


            // Cálculo MTTR
            const mttr = data.calculateMTTR();
            const topEquipamento = data.equipamentoData.labels.length > 0 ? data.equipamentoData.labels[0] : 'N/A';

            // Preencher os elementos KPI no HTML
            document.getElementById('kpi-ocorrencias').textContent = data.totalOcorrencias.toLocaleString('pt-BR');
            document.getElementById('kpi-downtime').textContent = data.totalDowntime.toLocaleString('pt-BR');
            document.getElementById('kpi-mttr').textContent = mttr;
            
            // Atualização dos dados do Equipamento Crítico (Top 1)
            document.getElementById('kpi-top-equipamento').textContent = topEquipamento;
            document.getElementById('kpi-top-ocorrencia').textContent = data.topEquipamentoOcorrencia;
            document.getElementById('kpi-top-ocorrencia').title = data.topEquipamentoOcorrencia;


            // Configurações de tema CLARO para Chart.js
            const lightChartConfig = {
                responsive: true,
                maintainAspectRatio: false,
                plugins: {
                    legend: {
                        labels: { color: '#374151' }
                    },
                    title: { display: false }
                },
                scales: {
                    y: {
                        grid: { color: '#e5e7eb' },
                        ticks: { color: '#4b5563' }
                    },
                    x: {
                        grid: { color: '#e5e7eb' },
                        ticks: { color: '#4b5563' }
                    }
                }
            };
            
            // 1. Gráfico de Distribuição de Quebras por Natureza (Donut Chart)
            const ctxNatureza = document.getElementById('chartNatureza').getContext('2d');
            charts.natureza = new Chart(ctxNatureza, {
                type: 'doughnut',
                data: {
                    labels: data.naturezaData.labels,
                    datasets: [{
                        data: data.naturezaData.counts,
                        backgroundColor: data.naturezaData.colors,
                        borderColor: '#ffffff',
                        borderWidth: 3,
                    }]
                },
                options: {
                    responsive: true,
                    maintainAspectRatio: false,
                    plugins: {
                        legend: {
                            position: 'right',
                            labels: { color: lightChartConfig.plugins.legend.labels.color }
                        },
                        tooltip: {
                            callbacks: {
                                label: function(context) {
                                    const total = context.dataset.data.reduce((a, b) => a + b, 0);
                                    const value = context.parsed;
                                    const percentage = ((value / total) * 100).toFixed(1) + '%';
                                    return `${context.label}: ${value} ocorrências (${percentage})`;
                                }
                            }
                        }
                    }
                }
            });

            // 2. Gráfico de Top 5 Setores por Downtime (Horizontal Bar Chart)
            const ctxSetores = document.getElementById('chartSetores').getContext('2d');
            charts.setores = new Chart(ctxSetores, {
                type: 'bar',
                data: {
                    labels: data.setoresData.labels,
                    datasets: [{
                        label: 'Tempo de Parada (Minutos)',
                        data: data.setoresData.downtime,
                        backgroundColor: data.setoresData.color,
                        borderRadius: 4,
                    }]
                },
                options: {
                    ...lightChartConfig,
                    indexAxis: 'y',
                    scales: {
                        x: {
                            ...lightChartConfig.scales.x,
                            title: { display: true, text: 'Minutos', color: '#4b5563' }
                        }
                    },
                    plugins: { ...lightChartConfig.plugins, legend: { display: false } }
                }
            });

            // 3. Gráfico de Top 5 Mantenedores por Ocorrências (Horizontal Bar Chart)
            const ctxTecnicos = document.getElementById('chartTecnicos').getContext('2d');
            charts.tecnicos = new Chart(ctxTecnicos, {
                type: 'bar',
                data: {
                    labels: data.technicianData.labels,
                    datasets: [{
                        label: 'Ocorrências Atendidas',
                        data: data.technicianData.counts,
                        backgroundColor: data.technicianData.color,
                        borderRadius: 4,
                    }]
                },
                options: {
                    ...lightChartConfig,
                    indexAxis: 'y',
                    scales: {
                        x: {
                            ...lightChartConfig.scales.x,
                            title: { display: true, text: 'Contagem', color: '#4b5563' }
                        }
                    },
                    plugins: { ...lightChartConfig.plugins, legend: { display: false } }
                }
            });

            // 4. Gráfico de Tendência Mensal de Falhas (Line Chart)
            const ctxTendencia = document.getElementById('chartTendencia').getContext('2d');
            charts.tendencia = new Chart(ctxTendencia, {
                type: 'line',
                data: {
                    labels: data.tendenciaData.labels,
                    datasets: [{
                        label: 'Número de Ocorrências',
                        data: data.tendenciaData.counts,
                        borderColor: data.tendenciaData.color,
                        backgroundColor: 'rgba(5, 150, 105, 0.1)',
                        borderWidth: 3,
                        tension: 0.4,
                        fill: true,
                        pointRadius: 5,
                        pointBackgroundColor: data.tendenciaData.color,
                    }]
                },
                options: {
                    ...lightChartConfig,
                    scales: {
                        y: {
                            ...lightChartConfig.scales.y,
                            title: { display: true, text: 'Contagem de Falhas', color: '#4b5563' },
                            beginAtZero: true
                        }
                    }
                }
            });

            // 5. Gráfico de Ocorrências por Turno (Vertical Bar Chart)
            const ctxTurno = document.getElementById('chartTurno').getContext('2d');
            charts.turno = new Chart(ctxTurno, {
                type: 'bar',
                data: {
                    labels: data.shiftData.labels,
                    datasets: [{
                        label: 'Total de Ocorrências',
                        data: data.shiftData.counts,
                        backgroundColor: data.shiftData.color,
                        borderRadius: 4,
                    }]
                },
                options: {
                    ...lightChartConfig,
                    scales: {
                        y: {
                            ...lightChartConfig.scales.y,
                            title: { display: true, text: 'Contagem', color: '#4b5563' },
                            beginAtZero: true
                        },
                        x: {
                            ...lightChartConfig.scales.x,
                            // Garante que os rótulos longos de turno sejam visíveis
                            maxRotation: 45, 
                            minRotation: 45
                        }
                    }
                }
            });

            // 6. Gráfico de Top 5 Equipamentos por Downtime (Horizontal Bar Chart)
            const ctxEquipamentos = document.getElementById('chartEquipamentos').getContext('2d');
            charts.equipamentos = new Chart(ctxEquipamentos, {
                type: 'bar',
                data: {
                    labels: data.equipamentoData.labels,
                    datasets: [{
                        label: 'Downtime (Minutos)',
                        data: data.equipamentoData.downtime,
                        backgroundColor: data.equipamentoData.color,
                        borderRadius: 4,
                    }]
                },
                options: {
                    ...lightChartConfig,
                    indexAxis: 'y',
                    scales: {
                        x: {
                            ...lightChartConfig.scales.x,
                            title: { display: true, text: 'Minutos de Parada', color: '#4b5563' }
                        }
                    }
                }
            });
        }
        
        // ***************************************************************
        // FUNÇÕES DE ATUALIZAÇÃO AUTOMÁTICA (FETCH API)
        // ***************************************************************
        
        // Função para buscar dados de uma URL
        async function fetchDataFromUrl(url) {
            updateButtonTextEl.textContent = 'Buscando URL...';
            try {
                const response = await fetch(url);
                if (!response.ok) {
                    throw new Error(`Erro HTTP: ${response.status} (${response.statusText}).`);
                }
                const csvText = await response.text();
                
                // Coloca o texto buscado na área de input manual para referência
                document.getElementById('csv-input').value = csvText.trim();
                
                // Processa e renderiza
                const data = processAndAggregateData(csvText);
                renderCharts(data);
                
                updateButtonTextEl.textContent = 'Atualização concluída.';
                setTimeout(() => {
                    if (autoRefreshInterval === null) {
                       updateButtonTextEl.textContent = 'Atualizar Dashboard Manualmente';
                    }
                }, 1500);

            } catch (e) {
                console.error("Erro ao buscar dados da URL:", e);
                const errorMsg = `Falha na busca (CORS/URL): ${e.message}. Use o método de Colar CSV.`;
                errorMessageEl.textContent = errorMsg;
                errorMessageEl.classList.remove('hidden');
                
                updateButtonTextEl.textContent = 'Falha na Busca.';
                setTimeout(() => {
                    updateButtonTextEl.textContent = 'Atualizar Dashboard Manualmente';
                }, 1500);
            }
        }

        // Função para alternar o modo de atualização automática
        function toggleAutoRefresh() {
            const url = csvUrlInputEl.value.trim();

            if (autoRefreshInterval) {
                // Desativar
                clearInterval(autoRefreshInterval);
                autoRefreshInterval = null;
                autoRefreshButtonEl.textContent = `Ativar Atualização a Cada ${refreshIntervalMinutes} Min`;
                autoRefreshButtonEl.classList.remove('bg-red-500', 'hover:bg-red-700');
                autoRefreshButtonEl.classList.add('bg-accent-green', 'hover:bg-green-700');
                autoRefreshStatusEl.textContent = 'Status: Inativo.';
                dataStatusEl.classList.remove('animate-pulse-status');
            } else {
                // Ativar
                if (!url) {
                    errorMessageEl.textContent = 'Por favor, insira uma URL de exportação CSV válida para ativar o modo automático.';
                    errorMessageEl.classList.remove('hidden');
                    return;
                }
                
                // Força a primeira busca
                fetchDataFromUrl(url); 

                // Configura o intervalo
                autoRefreshInterval = setInterval(() => {
                    fetchDataFromUrl(url);
                }, refreshIntervalMinutes * 60 * 1000); // 5 minutos em milissegundos

                autoRefreshButtonEl.textContent = `Desativar Atualização Automática`;
                autoRefreshButtonEl.classList.remove('bg-accent-green', 'hover:bg-green-700');
                autoRefreshButtonEl.classList.add('bg-red-500', 'hover:bg-red-700');
                autoRefreshStatusEl.textContent = `Status: Ativo (próxima em ${refreshIntervalMinutes} min).`;
                dataStatusEl.classList.add('animate-pulse-status'); // Adiciona um efeito visual de atualização

                // Remove a mensagem de erro se a URL for válida
                errorMessageEl.classList.add('hidden');
            }
        }
        
        // Função principal para iniciar o carregamento dos dados
        function handleDataLoad() {
            const csvInput = document.getElementById('csv-input').value;
            errorMessageEl.classList.add('hidden');
            
            // Se o modo automático estiver ativo, usamos a função de URL.
            if (autoRefreshInterval) {
                const url = csvUrlInputEl.value.trim();
                if (url) {
                    fetchDataFromUrl(url);
                    return;
                }
            }
            
            // Continua com o processamento manual do CSV
            updateButtonTextEl.textContent = 'Processando...';
            
            if (!csvInput.trim()) {
                errorMessageEl.textContent = 'Por favor, cole os dados CSV no campo acima.';
                errorMessageEl.classList.remove('hidden');
                updateButtonTextEl.textContent = 'Atualizar Dashboard Manualmente';
                return;
            }

            try {
                const data = processAndAggregateData(csvInput);
                renderCharts(data);
                updateButtonTextEl.textContent = 'Atualizado com Sucesso!';
                // Reseta o texto do botão após um breve momento
                setTimeout(() => {
                    updateButtonTextEl.textContent = 'Atualizar Dashboard Manualmente';
                }, 1500);

            } catch (e) {
                console.error("Erro ao processar o CSV:", e);
                errorMessageEl.textContent = 'Erro de processamento: O formato CSV pode estar incorreto. Detalhes: ' + e.message;
                errorMessageEl.classList.remove('hidden');
                updateButtonTextEl.textContent = 'Atualizar Dashboard Manualmente';
            }
        }

        // Função de inicialização
        function initDashboard() {
            // Verifica se há dados no editor do arquivo anterior para facilitar o teste inicial
            const initialCSV = `NATUREZA DA QUEBRA,Data Inicio:  ,ID ORDEM,Hora Inicio: ,Hora fim: ,SOLICITANTE:,Numero da O.M: ,TURNO,Setor: ,Equipamento: ,DESCRIÇÃO DA OCORRÊNCIA:,2 - Ação de Remoção de Sintoma,MANTEDEDOR,STATUS,HORA DE ATENDIMENTO,OBSERVAÇAO/PENDÊNÇIA,MATERIAL UTILIZADO,- ANALISE DAS CAUSAS DA FALHA/QUEBRA (5 POR QUE),POR QUE (1)?,POR QUE (2)?,POR QUE (3)?,POR QUE (4)?,POR QUE (5)?,Pode gerar uma LPP ?,"Equipe de análise:\n",Gatilho,FOTO
,,,,,,,,,,,,,,,,,,,,,,,,,
MECÂNICO,2025-03-11,f7e94e7e,14:10:02,15:10:00,Luciano,,TURNO 1,PH,tc 601A,desalinhamento da correia,Alinhamento da correia de transporte,Demetrio,,14:30:00,verificar o funcionamento,,,,,,,,,,2025-03-11 16:19:23,
ELÉTRICO,2025-03-11,a73751c6,17:41:00,18:01:00,Henrique,,TURNO 1,PH,TC-609,Falha no sensor de movimento de rotação,Troca do sensor,Demetrio,,17:41:00,,,,,,,SIM,teste1,teste2,,,,,,2025-05-07 05:08:29,
MECÂNICO,2025-03-11,3998b5a0,19:15:00,19:40:00,Henrique,,TURNO 1,PH,TC-609,Falha no motor 10cv/15cv,Troca do motor,Demetrio,,19:15:00,,,,,,,,,,,
ELÉTRICO,2025-03-12,71d2b9d0,03:30:00,04:20:00,Henrique,,TURNO 2,PH,TC-609,quebra de eixo,Troca do eixo,Demetrio,,03:30:00,,,,,,,SIM,teste1,teste2,,,,,,2025-05-07 05:08:29,
MECÂNICO,2025-03-12,b055998a,05:00:00,05:20:00,Henrique,,TURNO 2,PH,tc 601A,Rolamento com folga,Troca do rolamento,Demetrio,,05:00:00,,,,,,,,,,,
AUTOMAÇÃO,2025-03-12,25633b49,08:30:00,09:45:00,Luciano,,TURNO 1,PH,TC-609,sensor de presença,Troca do sensor,Demetrio,,08:30:00,,,,,,,,,,,
MECÂNICO,2025-03-12,504892c9,10:30:00,11:30:00,Luciano,,TURNO 1,PH,tc 601A,correia raspando,Ajuste da correia,Demetrio,,10:30:00,,,,,,,,,,,
ELÉTRICO,2025-03-12,1c801e0a,14:00:00,15:15:00,Henrique,,TURNO 1,PH,TC-609,Falha na ventilação,Limpeza na ventilação,Demetrio,,14:00:00,,,,,,,,,,,
AUTOMAÇÃO,2025-03-13,91a6d7f8,07:00:00,07:30:00,Luciano,,TURNO 2,PH,TC-609,Problema na lógica,Revisão da lógica,Demetrio,,07:00:00,,,,,,,,,,,
MECÂNICO,2025-03-13,f4a01c43,11:45:00,12:15:00,Henrique,,TURNO 1,PH,tc 601A,desalinhamento da correia,Alinhamento da correia,Demetrio,,11:45:00,,,,,,,,,,,
ELÉTRICO,2025-03-13,63b0a2c1,16:30:00,17:30:00,Luciano,,TURNO 1,PH,TC-609,Motor com cheiro de queimado,Troca do motor,Demetrio,,16:30:00,,,,,,,,,,,
AUTOMAÇÃO,2025-03-14,a2e1d7b3,01:00:00,02:15:00,Henrique,,TURNO 3,PH,TC-609,Falha no inversor,Troca do inversor,Demetrio,,01:00:00,,,,,,,,,,,
MECÂNICO,2025-03-14,5f187d9c,04:45:00,05:30:00,Luciano,,TURNO 3,PH,tc 601A,Quebra de eixo,Troca do eixo,Demetrio,,04:45:00,,,,,,,,,,,
ELÉTRICO,2025-10-04,337b9a32,16:34:00,17:00:00,Jeorge,,TURNO 2 TARDE,PH,tc 601A,queima eletrica da ventoinha do redutor,"limpeza na tampa frontal O travamento impede a rotação do sistema de ventilação, cessando a exaustão e exigindo atenção imediata para evitar queima elétrica ao tentar forçar a partida.","ELCID , DAVISON",FINALIZADA ,,,,,,,,,,,,,\nAUTOMAÇÃO,2025-10-03,6f8e994c,11:56:12,14:56:19,Marcos,,TURNO 1 MANHÃ,PERFUME,medidor de vazao da linha 05,medidor de vazao da linha 05 sem  esta medindo,"Diagnóstico: Foi confirmado, utilizando o software da Endress+Hauser, que o medidor apresenta falhas internas e sujeira no tubo interno.\n\nAção Necessária: O plano é agendar uma janela de produção para a retirada e manutenção/substituição do equipamento.","ELCID , MARCOS , DANILO , JEFFERSON , DAVISON",PENDENTE,,,,,,,,,,,,,\nMECÂNICO,2025-10-03,25a9ba4f,18:00:00,18:20:00,Jeorge,,TURNO 2 TARDE,EMBALAGEM,Empacot 2,Caçamba com passagem de pó,"Foi verificado que a vedacao mecânica está com defeito , foi retirado para usinar e desativado a caçamba 1 de funcionamento.",DANILO,,,,,,,,,,,,,,\n`;

            const csvInputEl = document.getElementById('csv-input');
            csvInputEl.value = initialCSV.trim();
            handleDataLoad(); // Carrega os dados iniciais automaticamente
        }

        window.onload = initDashboard;

    </script>
</body>
</html>
