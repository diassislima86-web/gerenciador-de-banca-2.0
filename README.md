# gerenciador-de-banca-2.0
diassis lima
<!DOCTYPE html>
<html lang="pt-PT">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Gestor de Banca</title>
    <!-- Carregamento do Tailwind CSS para estilização -->
    <script src="https://cdn.tailwindcss.com"></script>
    <!-- Carregamento do Chart.js para o Gráfico -->
    <script src="https://cdn.jsdelivr.net/npm/chart.js@4.4.3/dist/chart.umd.min.js"></script>
    <link href="https://fonts.googleapis.com/css2?family=Inter:wght@100..900&display=swap" rel="stylesheet">
    <style>
        :root {
            --color-primary: #10b981; /* Esmeralda 500 (vitória) */
            --color-secondary: #f43f5e; /* Rosa 500 (derrota) */
            --color-gale: #f59e0b; /* Âmbar 500 (alerta) */
            --color-soros: #3b82f6; /* Azul 500 (soros) */
        }
        body {
            font-family: 'Inter', sans-serif;
            background-color: #f3f4f6;
        }
        .card {
            background-color: white;
            border-radius: 12px;
            box-shadow: 0 10px 15px -3px rgba(0, 0, 0, 0.1), 0 4px 6px -2px rgba(0, 4, 0, 0.05);
        }
        .btn-win {
            background-color: var(--color-primary);
            transition: all 0.15s;
        }
        .btn-win:hover {
            background-color: #059669; /* Esmeralda 600 */
            transform: translateY(-1px);
        }
        .btn-loss {
            background-color: var(--color-secondary);
            transition: all 0.15s;
        }
        .btn-loss:hover {
            background-color: #e11d48; /* Rosa 600 */
            transform: translateY(-1px);
        }
        .btn-gale {
            background-color: var(--color-gale);
            transition: all 0.15s;
        }
        .btn-gale:hover {
            background-color: #d97706; /* Âmbar 600 */
            transform: translateY(-1px);
        }
        .btn-soros {
            background-color: var(--color-soros);
            transition: all 0.15s;
        }
        .btn-soros:hover {
            background-color: #2563eb; /* Azul 600 */
            transform: translateY(-1px);
        }
        /* CLASSE PARA DESTAQUE DE ERRO */
        .error-highlight {
            border-color: var(--color-secondary) !important; 
            box-shadow: 0 0 0 3px rgba(244, 63, 94, 0.5) !important;
        }
        #bankrollChartContainer {
            position: relative;
            height: 40vh; /* Altura responsiva */
            width: 100%;
        }
        .llm-loading {
            position: relative;
            pointer-events: none;
            opacity: 0.8;
        }
        .llm-loading::after {
            content: '';
            position: absolute;
            left: 50%;
            top: 50%;
            width: 16px;
            height: 16px;
            border: 2px solid #fff;
            border-top-color: transparent;
            border-radius: 50%;
            animation: spin 1s ease-in-out infinite;
            margin-left: -8px;
            margin-top: -8px;
        }
        @keyframes spin {
            to { transform: rotate(360deg); }
        }
    </style>
</head>
<body class="p-4 sm:p-8">

    <div id="app" class="max-w-6xl mx-auto space-y-8">
        <header class="text-center p-6 card">
            <h1 class="text-3xl font-extrabold text-gray-800">📊 Gestor de Banca</h1>
            <p class="text-gray-500 mt-1">O seu risco e retorno são calculados por cada moeda de forma isolada.</p>
        </header>

        <!-- Secção de Configurações de Risco -->
        <div class="p-6 card border-l-4 border-indigo-500">
            <h2 class="text-xl font-semibold text-gray-800 mb-4">⚙️ Configurações da Banca Atual</h2>
            <div class="grid grid-cols-1 md:grid-cols-4 gap-4">
                
                <!-- Modo de Risco -->
                <div class="col-span-1">
                    <label for="riskModeInput" class="block text-sm font-medium text-gray-700">Modo de Risco</label>
                    <div class="mt-1 flex gap-4">
                        <label class="inline-flex items-center">
                            <input type="radio" name="riskMode" value="percentage" checked
                                   onclick="updateSettings('riskMode', 'percentage'); toggleRiskInputs('percentage')"
                                   class="form-radio text-indigo-600">
                            <span class="ml-2 text-sm text-gray-700">% da Banca</span>
                        </label>
                        <label class="inline-flex items-center">
                            <input type="radio" name="riskMode" value="fixed"
                                   onclick="updateSettings('riskMode', 'fixed'); toggleRiskInputs('fixed')"
                                   class="form-radio text-indigo-600">
                            <span class="ml-2 text-sm text-gray-700">Valor Fixo</span>
                        </label>
                    </div>
                </div>

                <!-- Input Risco % (Visível no modo 'percentage') -->
                <div id="percentageRiskGroup">
                    <label for="riskPercentageInput" class="block text-sm font-medium text-gray-700">Risco Máximo da Banca (%)</label>
                    <div class="relative mt-1 rounded-md shadow-sm">
                        <input type="number" id="riskPercentageInput" min="0.1" step="0.1"
                                class="block w-full rounded-md border-gray-300 pr-10 focus:border-indigo-500 focus:ring-indigo-500"
                                onchange="updateSettings('riskPercentage', this.value)" oninput="updateSettings('riskPercentage', this.value)">
                        <div class="pointer-events-none absolute inset-y-0 right-0 flex items-center pr-3">
                            <span class="text-gray-500">%</span>
                        </div>
                    </div>
                </div>

                <!-- Input Risco Fixo (Visível no modo 'fixed') -->
                <div id="fixedRiskGroup" class="hidden">
                    <label for="fixedUnitValueInput" class="block text-sm font-medium text-gray-700">Valor Fixo da Unidade (Risco / 3)</label>
                    <div class="relative mt-1 rounded-md shadow-sm">
                        <input type="number" id="fixedUnitValueInput" min="0.01" step="0.01"
                                class="block w-full rounded-md border-gray-300 pr-10 focus:border-indigo-500 focus:ring-indigo-500"
                                onchange="updateSettings('fixedUnitValue', this.value)" oninput="updateSettings('fixedUnitValue', this.value)">
                        <div class="pointer-events-none absolute inset-y-0 right-0 flex items-center pr-3">
                            <span id="fixedUnitSymbol" class="text-gray-500">$</span>
                        </div>
                    </div>
                </div>
                
                <!-- Input Retorno % -->
                <div>
                    <label for="returnPercentageInput" class="block text-sm font-medium text-gray-700">Retorno da Operação (Pay-out %)</label>
                    <div class="relative mt-1 rounded-md shadow-sm">
                        <input type="number" id="returnPercentageInput" min="0.1" step="0.1"
                                class="block w-full rounded-md border-gray-300 pr-10 focus:border-indigo-500 focus:ring-indigo-500"
                                onchange="updateSettings('returnPercentage', this.value)" oninput="updateSettings('returnPercentage', this.value)">
                        <div class="pointer-events-none absolute inset-y-0 right-0 flex items-center pr-3">
                            <span class="text-gray-500">%</span>
                        </div>
                    </div>
                </div>

                <!-- Seletor de Moeda -->
                <div>
                    <label for="currencyInput" class="block text-sm font-medium text-gray-700">Moeda da Banca</label>
                    <select id="currencyInput" 
                            class="mt-1 block w-full rounded-md border-gray-300 shadow-sm focus:border-indigo-500 focus:ring-indigo-500"
                            onchange="switchBankroll(this.value)">
                        <option value="USD">Dólar (USD)</option>
                        <option value="EUR">Euro (€)</option>
                        <option value="BRL">Real (R$)</option>
                    </select>
                </div>

            </div>
        </div>

        <!-- Secção de Dados Chave e Regra de Risco / 3 -->
        <div class="grid grid-cols-1 sm:grid-cols-2 lg:grid-cols-4 gap-4">
            <!-- 1. Banca Inicial (Input) -->
            <div class="p-5 card border-l-4 border-gray-400">
                <label for="initialBankrollInput" class="block text-sm font-medium text-gray-500">Banca Inicial</label>
                <input type="number" id="initialBankrollInput" placeholder="Ex: 1000.00"
                       class="mt-1 block w-full text-xl font-semibold border-none focus:ring-0 p-0 text-gray-700"
                       onchange="updateInitialBankroll(this.value)" oninput="updateInitialBankroll(this.value)">
            </div>

            <!-- 2. Banca Atual (Display) -->
            <div class="p-5 card border-l-4 border-green-500">
                <p class="text-sm font-medium text-gray-500">Banca Atual (Saldo)</p>
                <p id="currentBankrollDisplay" class="mt-1 text-2xl font-bold text-green-600">$ 0,00</p>
            </div>

            <!-- 3. Risco Máximo Total (Calculado DINÂMICO) -->
            <div class="p-5 card border-l-4 border-red-500">
                <p class="text-sm font-medium text-gray-500" id="maxRiskLabel">Risco Máximo Total</p>
                <p id="maxRiskTotalDisplay" class="mt-1 text-2xl font-bold text-red-600">$ 0,00</p>
            </div>

            <!-- 4. Valor da Unidade (Calculado) -->
            <div class="p-5 card border-l-4 border-orange-500">
                <p class="text-sm font-medium text-gray-500">Valor da Unidade</p>
                <p id="unitStakeDisplay" class="mt-1 text-2xl font-bold text-orange-600">$ 0,00</p>
            </div>
        </div>

        <!-- Métricas de Desempenho -->
        <div class="grid grid-cols-1 sm:grid-cols-3 gap-4">
            <!-- 5. Total de Vitórias -->
            <div class="p-5 card border-l-4 border-green-700 text-center">
                <p class="text-sm font-medium text-gray-500">Vitórias (W)</p>
                <p id="winCountDisplay" class="mt-1 text-3xl font-bold text-green-700">0</p>
            </div>
             <!-- 6. Total de Derrotas -->
            <div class="p-5 card border-l-4 border-red-700 text-center">
                <p class="text-sm font-medium text-gray-500">Derrotas (L)</p>
                <p id="lossCountDisplay" class="mt-1 text-3xl font-bold text-red-700">0</p>
            </div>
             <!-- 7. Taxa de Acerto (Win Rate) -->
            <div class="p-5 card border-l-4 border-blue-500 text-center">
                <p class="text-sm font-medium text-gray-500">Taxa de Vitórias (W%)</p>
                <p id="winRateDisplay" class="mt-1 text-3xl font-bold text-blue-600">0.00%</p>
            </div>
        </div>

        <!-- Gráfico de Crescimento da Banca -->
        <div class="p-6 card">
            <h2 class="text-xl font-semibold text-gray-800 mb-4">📈 Evolução da Banca</h2>
            <div id="bankrollChartContainer">
                <canvas id="bankrollChart"></canvas>
            </div>
        </div>
        
        <!-- Secção de Registo Rápido (Operações) -->
        <div class="p-6 card">
            <h2 class="text-xl font-semibold text-gray-800 mb-4">⚡ Registo Rápido (Operações)</h2>
            <p class="text-sm text-gray-600 mb-4">
                As transações usam o **Valor da Unidade** e a taxa de retorno de <span id="returnPercentLabel" class="font-bold text-green-600">87%</span>.
            </p>
            
            <!-- Linha de Unidade Padrão e Soros -->
            <div class="border-b pb-4 mb-4">
                <h3 class="text-lg font-semibold text-gray-800 mb-3">Unidade Padrão & Soros</h3>
                <div class="flex flex-wrap gap-4">
                    <!-- 1. Unidade Padrão -->
                    <button onclick="registerAutoTransaction(true, 0)" 
                            class="btn-win text-white font-medium py-3 px-6 rounded-lg shadow-md hover:shadow-lg w-full sm:w-auto">
                        ✅ Vitória (Unidade)
                    </button>
                    <button onclick="registerAutoTransaction(false, 0)" 
                            class="btn-loss text-white font-medium py-3 px-6 rounded-lg shadow-md hover:shadow-lg w-full sm:w-auto">
                        ❌ Derrota (Unidade)
                    </button>

                    <!-- 2. Soros Nível 2 -->
                    <button onclick="registerAutoTransaction(true, 3)" 
                            class="btn-soros text-white font-medium py-3 px-6 rounded-lg shadow-md hover:shadow-lg w-full sm:w-auto">
                        ✅ Vitória (Soros 2)
                    </button>
                    <button onclick="registerAutoTransaction(false, 3)" 
                            class="btn-loss text-white font-medium py-3 px-6 rounded-lg shadow-md hover:shadow-lg w-full sm:w-auto">
                        ❌ Derrota (Soros 2)
                    </button>
                </div>
            </div>


            <!-- Linha de Gale -->
            <div class="mt-4 pt-4">
                <h3 class="text-lg font-semibold text-gray-800 mb-3">Ciclo de Perda (Gale)</h3>
                <p class="text-xs text-red-600 mb-3">AVISO: Use apenas após uma derrota anterior. Estas operações registam uma **PERDA** no respetivo nível.</p>
                <div class="flex flex-wrap gap-4">
                     <!-- 2. Gale 1 -->
                    <button onclick="registerAutoTransaction(false, 1)" 
                            class="btn-gale text-white font-medium py-3 px-6 rounded-lg shadow-md hover:shadow-lg w-full sm:w-auto">
                        ❌ Derrota (Gale 1)
                    </button>
                     <!-- 3. Gale 2 -->
                    <button onclick="registerAutoTransaction(false, 2)" 
                            class="btn-loss text-white font-medium py-3 px-6 rounded-lg shadow-md hover:shadow-lg w-full sm:w-auto">
                        ❌ Derrota (Gale 2)
                    </button>
                    <!-- 4. Vitória de Saída do Gale (para qualquer nível) -->
                    <button onclick="registerAutoTransaction(true, 1)" 
                            class="btn-win text-white font-medium py-3 px-6 rounded-lg shadow-md hover:shadow-lg w-full sm:w-auto">
                        ✅ Vitória (Saída Gale)
                    </button>
                </div>
            </div>
        </div>

        <!-- Tabela de Histórico de Transações -->
        <div class="p-6 card overflow-x-auto">
            <h2 class="text-xl font-semibold text-gray-800 mb-4">🧾 Histórico de Transações</h2>
            <table class="min-w-full divide-y divide-gray-200">
                <thead class="bg-gray-50">
                    <tr>
                        <th class="px-3 py-3 text-left text-xs font-medium text-gray-500 uppercase tracking-wider">Data</th>
                        <th class="px-3 py-3 text-left text-xs font-medium text-gray-500 uppercase tracking-wider">Descrição</th>
                        <th class="px-3 py-3 text-left text-xs font-medium text-gray-500 uppercase tracking-wider">Investido</th>
                        <th class="px-3 py-3 text-left text-xs font-medium text-gray-500 uppercase tracking-wider">Resultado</th>
                        <th class="px-3 py-3 text-left text-xs font-medium text-gray-500 uppercase tracking-wider">Banca Atual</th>
                        <th class="px-3 py-3 text-left text-xs font-medium text-gray-500 uppercase tracking-wider">Ação</th>
                    </tr>
                </thead>
                <tbody id="transactionList" class="bg-white divide-y divide-gray-200">
                    <!-- As transações serão inseridas aqui pelo JavaScript -->
                    <tr>
                        <td colspan="6" class="py-4 text-center text-gray-500">Nenhuma transação registada.</td>
                    </tr>
                </tbody>
            </table>
        </div>

        <!-- Secção de Adicionar Transação Manual (Aportes/Retiradas) -->
        <div class="p-6 card">
            <h2 class="text-xl font-semibold text-gray-800 mb-4">✍️ Adicionar Transação Manual (Aportes/Outros)</h2>
            <div id="llmAnalysisOutput" class="p-3 mb-4 bg-gray-50 border border-gray-200 rounded-lg text-gray-600 hidden">
                <!-- Output da análise LLM aqui -->
            </div>
            <form id="transactionForm" class="grid grid-cols-1 md:grid-cols-4 gap-4">
                <div class="col-span-1 md:col-span-2">
                    <label for="description" class="block text-sm font-medium text-gray-700">Descrição</label>
                    <input type="text" id="description" required placeholder="Ex: Novo Aporte ou Operação Manual" class="mt-1 block w-full rounded-md border-gray-300 shadow-sm focus:border-emerald-500 focus:ring-emerald-500">
                </div>
                <div>
                    <label for="invested" class="block text-sm font-medium text-gray-700">Valor Investido</label>
                    <input type="number" id="invested" required min="0" step="0.01" value="0.00" class="mt-1 block w-full rounded-md border-gray-300 shadow-sm focus:border-emerald-500 focus:ring-emerald-500">
                </div>
                <div>
                    <label for="result" class="block text-sm font-medium text-gray-700">Resultado (L/P/Aporte)</label>
                    <input type="number" id="result" required step="0.01" placeholder="Ex: +500.00 (Aporte) ou -25.00 (Saque)" class="mt-1 block w-full rounded-md border-gray-300 shadow-sm focus:border-emerald-500 focus:ring-emerald-500">
                </div>
                
                <div class="md:col-span-4 flex flex-col sm:flex-row justify-end gap-2 mt-4">
                    <button type="button" onclick="analyzeRiskWithLLM(event)" id="analyzeButton"
                            class="bg-indigo-600 text-white font-medium py-2 px-4 rounded-lg shadow-md hover:bg-indigo-700 hover:shadow-lg transition w-full sm:w-auto">
                        ✨ Análise de Risco LLM
                    </button>
                    <button type="submit" class="btn-win text-white font-medium py-2 px-4 rounded-lg shadow-md hover:shadow-lg w-full sm:w-auto">
                        Registar Transação Manual
                    </button>
                </div>
            </form>
        </div>
    </div>

    <!-- Firebase Imports e Scripts -->
    <script type="module">
        // Importa as funções necessárias do Firebase SDK
        import { initializeApp } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-app.js";
        import { getAuth, signInAnonymously, signInWithCustomToken, onAuthStateChanged } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-auth.js";
        import { getFirestore, doc, setDoc, onSnapshot, updateDoc, arrayRemove, arrayUnion } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-firestore.js";
        import { setLogLevel } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-firestore.js";

        // Variáveis globais fornecidas pelo ambiente Canvas
        const appId = typeof __app_id !== 'undefined' ? __app_id : 'default-app-id';
        const firebaseConfig = typeof __firebase_config !== 'undefined' ? JSON.parse(__firebase_config) : null;
        const initialAuthToken = typeof __initial_auth_token !== 'undefined' ? __initial_auth_token : null;
        // API Key para a API Gemini (deixada em branco para o ambiente Canvas)
        const apiKey = "" 
        const apiUrl = `https://generativelanguage.googleapis.com/v1beta/models/gemini-2.5-flash-preview-05-20:generateContent?key=${apiKey}`;

        let db;
        let auth;
        let userId = 'loading';
        let trackerDocRef; // Será definido dinamicamente em listenForData/switchBankroll
        let unsubscribeSnapshot = null; // Para gerir a subscrição do Firestore
        
        // Dados atuais da aplicação
        let currentData = {
            initialBankroll: 0,
            transactions: [],
            riskPercentage: 5,
            fixedUnitValue: 10.00, // Novo valor fixo por unidade (default USD $10)
            riskMode: 'percentage', // Novo modo de risco
            returnPercentage: 87,
            currency: 'USD' // Default Dólar
        };
        let isAuthReady = false;
        let bankrollChart; // Variável para a instância do Chart.js

        setLogLevel('Debug');

        // Função para formatar números como moeda (Euro, Real ou Dólar)
        const formatCurrency = (value) => {
            if (typeof value !== 'number' || isNaN(value)) return '$ 0.00';
            
            let currencyCode = 'USD';
            let locale = 'en-US';

            if (currentData.currency === 'BRL') {
                currencyCode = 'BRL';
                locale = 'pt-BR';
            } else if (currentData.currency === 'EUR') {
                 currencyCode = 'EUR';
                 locale = 'pt-PT';
            }

            return new Intl.NumberFormat(locale, {
                style: 'currency',
                currency: currencyCode,
                minimumFractionDigits: 2,
                maximumFractionDigits: 2
            }).format(value);
        };

        // Função para alternar a visibilidade dos campos de input de risco
        window.toggleRiskInputs = (mode) => {
            document.getElementById('percentageRiskGroup').classList.toggle('hidden', mode === 'fixed');
            document.getElementById('fixedRiskGroup').classList.toggle('hidden', mode === 'percentage');
            // Atualiza o símbolo no input fixo
            const symbol = currentData.currency === 'BRL' ? 'R$' : (currentData.currency === 'EUR' ? '€' : '$');
            document.getElementById('fixedUnitSymbol').textContent = symbol;
            
            // NOTE: A chamada renderUI() foi removida daqui para evitar o loop infinito
        };

        // Função principal de inicialização do Firebase e autenticação
        const initFirebase = async () => {
            if (firebaseConfig) {
                const app = initializeApp(firebaseConfig);
                db = getFirestore(app);
                auth = getAuth(app);

                // Autenticação
                onAuthStateChanged(auth, async (user) => {
                    if (user) {
                        userId = user.uid;
                    } else {
                        // Se não houver usuário, tenta login com token ou anónimo
                        try {
                            if (initialAuthToken) {
                                await signInWithCustomToken(auth, initialAuthToken);
                            } else {
                                await signInAnonymously(auth);
                            }
                            userId = auth.currentUser.uid;
                        } catch (error) {
                            console.error("Erro ao autenticar no Firebase:", error);
                            userId = crypto.randomUUID(); // Fallback para ID temporário
                        }
                    }

                    isAuthReady = true;
                    // Inicializa com a moeda padrão (USD), lendo o seletor na UI
                    const initialCurrency = document.getElementById('currencyInput').value;
                    switchBankroll(initialCurrency, true); 
                    initChart(); // Inicializa o gráfico
                });
            } else {
                console.error("Configuração do Firebase não encontrada.");
            }
        };

        // Troca o documento da banca para a moeda selecionada e carrega os dados
        window.switchBankroll = (newCurrency, isInitialLoad = false) => {
            if (!isAuthReady) return;
            
            // 1. Altera a moeda no estado local
            currentData.currency = newCurrency;

            // 2. Cancela a escuta anterior (CRUCIAL para multi-documento)
            if (unsubscribeSnapshot) {
                unsubscribeSnapshot();
                unsubscribeSnapshot = null;
            }

            // 3. Define o novo documento (Ex: main_tracker_USD, main_tracker_EUR)
            const docId = `main_tracker_${newCurrency}`;
            trackerDocRef = doc(db, `artifacts/${appId}/users/${userId}/financial_tracker`, docId);

            // 4. Inicia a escuta para o novo documento
            listenForData();

            // 5. Atualiza a UI
            if (!isInitialLoad) {
                document.getElementById('currencyInput').value = newCurrency;
            }
        };
        
        // Inicializa o Gráfico Chart.js
        const initChart = () => {
            // FIX: Destrói o gráfico existente antes de criar um novo, se houver.
            if (bankrollChart) {
                bankrollChart.destroy();
            }

            const ctx = document.getElementById('bankrollChart').getContext('2d');
            bankrollChart = new Chart(ctx, {
                type: 'line',
                data: {
                    labels: ['Início'], // Etiqueta inicial
                    datasets: [{
                        label: 'Banca Atual',
                        data: [0], // Valor inicial
                        borderColor: '#2563eb', // Azul
                        backgroundColor: 'rgba(37, 99, 235, 0.1)',
                        tension: 0.3,
                        pointRadius: 4,
                        pointHoverRadius: 6,
                    }]
                },
                options: {
                    responsive: true,
                    maintainAspectRatio: false,
                    scales: {
                        y: {
                            beginAtZero: false,
                            title: {
                                display: true,
                                text: 'Valor'
                            },
                            ticks: {
                                callback: function(value) {
                                    return formatCurrency(value);
                                }
                            }
                        }
                    },
                    plugins: {
                        legend: {
                            display: false
                        },
                        tooltip: {
                            callbacks: {
                                label: function(context) {
                                    let label = context.dataset.label || '';
                                    if (label) {
                                        label += ': ';
                                    }
                                    if (context.parsed.y !== null) {
                                        label += formatCurrency(context.parsed.y);
                                    }
                                    return label;
                                }
                            }
                        }
                    }
                }
            });
        };

        // Função para escutar as mudanças no Firestore em tempo real
        const listenForData = () => {
            if (!isAuthReady || !trackerDocRef) return;
            
            // Atribui a função de "unsubscribe"
            unsubscribeSnapshot = onSnapshot(trackerDocRef, (docSnap) => {
                const currencyBefore = currentData.currency;
                
                if (docSnap.exists()) {
                    const data = docSnap.data();
                    currentData = {
                        initialBankroll: data.initialBankroll || 0,
                        transactions: (data.transactions || []).filter(t => t.timestamp),
                        // Carrega as configurações específicas do documento da moeda
                        riskPercentage: data.riskPercentage !== undefined ? data.riskPercentage : 5,
                        fixedUnitValue: data.fixedUnitValue !== undefined ? data.fixedUnitValue : 10.00, // Novo
                        riskMode: data.riskMode || 'percentage', // Novo
                        returnPercentage: data.returnPercentage !== undefined ? data.returnPercentage : 87,
                        currency: currencyBefore // Mantém a moeda selecionada, que define o documento
                    };
                } else {
                    // Documento não existe (é a primeira vez que usa esta moeda)
                    // Reseta para os defaults e cria o documento
                    currentData = {
                        initialBankroll: 0,
                        transactions: [],
                        riskPercentage: 5,
                        fixedUnitValue: 10.00,
                        riskMode: 'percentage',
                        returnPercentage: 87,
                        currency: currencyBefore
                    };
                    setDoc(trackerDocRef, currentData).catch(e => console.error(`Erro ao criar documento inicial para ${currencyBefore}:`, e));
                }
                
                // Aplica o modo de risco carregado
                // AQUI FOI O ERRO: Chamamos toggleRiskInputs que chamava renderUI. Agora está correto.
                toggleRiskInputs(currentData.riskMode);
                
                renderUI();
                updateChart(); // Atualiza o gráfico após renderizar a UI
            }, (error) => {
                console.error("Erro ao escutar dados:", error);
            });
        };

        // Atualiza os dados no Gráfico
        const updateChart = () => {
            if (!bankrollChart) return;
            
            // Filtra e ordena as transações pela data mais antiga primeiro para calcular a evolução
            const transactions = currentData.transactions.slice().sort((a, b) => a.timestamp - b.timestamp);
            let currentBankroll = currentData.initialBankroll;

            // 1. Prepara os dados para o gráfico
            const labels = ['Início'];
            const data = [currentBankroll];

            transactions.forEach((t, index) => {
                currentBankroll += t.result;
                labels.push(`Op. ${index + 1} (${new Date(t.timestamp).toLocaleDateString('pt-PT').slice(0, 5)})`);
                data.push(currentBankroll);
            });

            // 2. Atualiza o Chart.js
            bankrollChart.data.labels = labels;
            bankrollChart.data.datasets[0].data = data;
            bankrollChart.update();
        };


        // Função que renderiza a interface do utilizador com os dados atuais
        const renderUI = () => {
            const initialBankroll = currentData.initialBankroll;
            const transactions = currentData.transactions;

            // 1. Atualizar Configurações (Inputs)
            const riskInput = document.getElementById('riskPercentageInput');
            const fixedInput = document.getElementById('fixedUnitValueInput');
            const returnInput = document.getElementById('returnPercentageInput');
            const currencyInput = document.getElementById('currencyInput');
            const maxRiskLabel = document.getElementById('maxRiskLabel');
            
            if (document.activeElement !== riskInput) {
                riskInput.value = currentData.riskPercentage;
            }
            if (document.activeElement !== fixedInput) {
                fixedInput.value = currentData.fixedUnitValue.toFixed(2);
            }
            if (document.activeElement !== returnInput) {
                returnInput.value = currentData.returnPercentage;
            }
            if (document.activeElement !== currencyInput) {
                currencyInput.value = currentData.currency;
            }

            // Define o modo de risco correto nos radio buttons
            document.querySelector(`input[name="riskMode"][value="${currentData.riskMode}"]`).checked = true;
            // NOTE: Chamada toggleRiskInputs(currentData.riskMode) aqui APÓS carregar inputs.

            // 2. Atualizar Banca Inicial (Input)
            const initialInput = document.getElementById('initialBankrollInput');
            if (document.activeElement !== initialInput) { 
                initialInput.value = initialBankroll.toFixed(2);
            }

            // 3. Calcular Banca Atual (Base para o risco dinâmico)
            let currentBankroll = initialBankroll;
            transactions.forEach(t => {
                currentBankroll += t.result;
            });
            const bankrollForRisk = Math.max(0, currentBankroll); // Banca para o cálculo
            window.currentBankrollValue = currentBankroll; // Salva globalmente para o LLM

            let maxRiskTotal = 0;
            let unitStake = 0;

            // 4. Cálculo baseado no Modo de Risco
            if (currentData.riskMode === 'percentage') {
                const riskRatio = currentData.riskPercentage / 100;
                maxRiskTotal = bankrollForRisk * riskRatio;
                unitStake = maxRiskTotal / 3;
                maxRiskLabel.innerHTML = `Risco Máximo Total (<span id="riskPercentLabel">${currentData.riskPercentage}%</span> da Atual) `;
            } else { // fixed mode
                unitStake = currentData.fixedUnitValue;
                maxRiskTotal = unitStake * 3; // O risco máximo total é 3x o valor da unidade fixa
                maxRiskLabel.innerHTML = `Risco Máximo Total (3x Unidade Fixa)`;
            }

            // 5. Atualizar Labels e Displays
            document.getElementById('currentBankrollDisplay').textContent = formatCurrency(currentBankroll);
            document.getElementById('maxRiskTotalDisplay').textContent = formatCurrency(maxRiskTotal);
            document.getElementById('unitStakeDisplay').textContent = formatCurrency(unitStake);
            
            // Atualiza labels de % na UI
            document.getElementById('riskPercentLabel').textContent = `${currentData.riskPercentage}%`;
            document.getElementById('returnPercentLabel').textContent = `${currentData.returnPercentage}%`;


            // 6. Calcular Métricas W/L (Apenas transações automáticas)
            let winCount = 0;
            let lossCount = 0;
            let totalAutoTransactions = 0;

            transactions.forEach(t => {
                // A descrição das transações automáticas contém "Unidade", "Gale" ou "Soros"
                if (t.description.includes("Unidade") || t.description.includes("Gale") || t.description.includes("Soros")) {
                    totalAutoTransactions++;
                    if (t.result > 0) {
                        winCount++;
                    } else if (t.result < 0) {
                        lossCount++;
                    }
                }
            });

            const winRate = totalAutoTransactions > 0 ? (winCount / totalAutoTransactions) * 100 : 0;

            // 7. Atualizar Métricas W/L
            document.getElementById('winCountDisplay').textContent = winCount;
            document.getElementById('lossCountDisplay').textContent = lossCount;
            document.getElementById('winRateDisplay').textContent = `${winRate.toFixed(2)}%`;

            // 8. Renderizar Tabela de Transações
            const transactionList = document.getElementById('transactionList');
            transactionList.innerHTML = ''; 

            if (transactions.length === 0) {
                transactionList.innerHTML = '<tr><td colspan="6" class="py-4 text-center text-gray-500">Nenhuma transação registada.</td></tr>';
                return;
            }

            // Mapeia e inverte a ordem para mostrar o mais recente primeiro
            transactions.slice().sort((a, b) => b.timestamp - a.timestamp).forEach((t) => {
                
                // Recalcula o saldo da banca a cada linha (para o caso de edições)
                const sortedTransactions = currentData.transactions.slice().sort((a, b) => a.timestamp - b.timestamp);
                
                const originalIndex = sortedTransactions.findIndex(st => st.timestamp === t.timestamp && st.result === t.result && st.investedAmount === t.investedAmount);
                
                let rowCurrentBankroll = initialBankroll;
                if (originalIndex !== -1) {
                    for (let i = 0; i <= originalIndex; i++) {
                        rowCurrentBankroll += sortedTransactions[i].result;
                    }
                } else {
                    rowCurrentBankroll = currentBankroll; 
                }
                
                const row = document.createElement('tr');
                const resultColor = t.result >= 0 ? 'text-green-600 font-semibold' : 'text-red-600 font-semibold';
                const investedColor = t.description.includes("Gale") ? 'bg-yellow-50 text-orange-700' : 'text-gray-900';


                row.innerHTML = `
                    <td class="px-3 py-2 whitespace-nowrap text-sm text-gray-500">${new Date(t.timestamp).toLocaleDateString('pt-PT')}</td>
                    <td class="px-3 py-2 text-sm text-gray-900">${t.description}</td>
                    <td class="px-3 py-2 whitespace-nowrap text-sm text-gray-500 ${investedColor}">${formatCurrency(t.investedAmount)}</td>
                    <td class="px-3 py-2 whitespace-nowrap text-sm ${resultColor}">${formatCurrency(t.result)}</td>
                    <td class="px-3 py-2 whitespace-nowrap text-sm text-gray-900 font-bold">${formatCurrency(rowCurrentBankroll)}</td>
                    <td class="px-3 py-2 whitespace-nowrap text-sm text-gray-900">
                        <button onclick="deleteTransaction(${t.timestamp}, ${t.result}, ${t.investedAmount})" 
                                class="text-red-500 hover:text-red-700 p-1 rounded-full hover:bg-red-100 transition duration-150"
                                title="Eliminar Transação">
                            🗑️
                        </button>
                    </td>
                `;
                transactionList.appendChild(row);
            });
        };

        // --- Funções de Ação do Utilizador ---

        // Função para chamar a API Gemini e analisar a transação manual
        window.analyzeRiskWithLLM = async (event) => {
            event.preventDefault();

            const description = document.getElementById('description').value || 'Transação genérica';
            const investedAmount = parseFloat(document.getElementById('invested').value);
            const result = parseFloat(document.getElementById('result').value);
            const currentBankroll = window.currentBankrollValue;

            const outputDiv = document.getElementById('llmAnalysisOutput');
            const analyzeButton = document.getElementById('analyzeButton');

            if (isNaN(investedAmount) || isNaN(result) || currentBankroll <= 0) {
                outputDiv.classList.remove('hidden');
                outputDiv.innerHTML = `<p class="text-red-600">⚠️ Erro: Por favor, defina a Banca Inicial e preencha os campos Investido/Resultado.</p>`;
                return;
            }

            analyzeButton.classList.add('llm-loading');
            analyzeButton.textContent = 'A Analisar...';
            outputDiv.classList.add('hidden');
            outputDiv.innerHTML = '';

            // Determina o símbolo correto para o prompt
            let currencySymbol = currentData.currency === 'BRL' ? 'R$' : (currentData.currency === 'EUR' ? '€' : '$');
            
            // 1. Construir o Prompt para o LLM
            const prompt = `
                Como um analista financeiro conservador, avalie a seguinte transação em relação ao capital total.
                
                - Banca Atual (Capital Total): ${currencySymbol} ${currentBankroll.toFixed(2)}
                - Transação (Investido/Risco): ${currencySymbol} ${investedAmount.toFixed(2)}
                - Resultado Esperado/Lançado: ${currencySymbol} ${result.toFixed(2)}
                - Descrição: ${description}

                Forneça uma análise de risco concisa (máximo 4 frases) em português de Portugal.
                1. Destaque o percentual de risco (% do capital total investido nesta operação).
                2. Comente sobre o impacto desta exposição no capital total.
                3. Use um tom conservador (ex: "é um risco elevado", "abordagem sensata").
            `;
            
            // 2. Configurar a chamada da API
            const payload = {
                contents: [{ parts: [{ text: prompt }] }],
                systemInstruction: {
                    parts: [{ text: "Você é um analista financeiro conservador e conciso. Suas respostas devem ser limitadas a 4 frases, focadas em percentual de risco e preservação de capital." }]
                },
            };

            const maxRetries = 3;
            for (let i = 0; i < maxRetries; i++) {
                try {
                    const response = await fetch(apiUrl, {
                        method: 'POST',
                        headers: { 'Content-Type': 'application/json' },
                        body: JSON.stringify(payload)
                    });

                    if (!response.ok) {
                        throw new Error(`Erro HTTP: ${response.status}`);
                    }

                    const resultJson = await response.json();
                    const generatedText = resultJson.candidates?.[0]?.content?.parts?.[0]?.text || "Erro: Não foi possível obter a análise.";

                    // 3. Exibir o resultado na UI
                    outputDiv.innerHTML = `<p class="font-bold text-gray-800">Análise LLM (Gemini):</p><p>${generatedText}</p>`;
                    outputDiv.classList.remove('hidden');
                    break; // Sai do loop se for bem-sucedido

                } catch (error) {
                    console.error(`Erro na chamada LLM (Tentativa ${i + 1}):`, error);
                    if (i < maxRetries - 1) {
                        // Implementa backoff exponencial
                        await new Promise(resolve => setTimeout(resolve, Math.pow(2, i) * 1000));
                    } else {
                        outputDiv.innerHTML = `<p class="text-red-600">❌ Erro na API Gemini: Não foi possível realizar a análise após ${maxRetries} tentativas.</p>`;
                        outputDiv.classList.remove('hidden');
                    }
                }
            }
            
            // 4. Resetar o botão
            analyzeButton.classList.remove('llm-loading');
            analyzeButton.textContent = '✨ Análise de Risco LLM';
        };

        // Atualiza as configurações de porcentagem ou moeda no Firestore
        window.updateSettings = async (settingKey, value) => {
            if (!isAuthReady || !trackerDocRef) return;

            // Se for moeda, ela já foi trocada pelo switchBankroll
            if (settingKey === 'currency') return; 

            // Atualiza os valores percentuais ou fixos
            let updateValue = value;
            if (settingKey === 'riskPercentage' || settingKey === 'returnPercentage' || settingKey === 'fixedUnitValue') {
                updateValue = parseFloat(value);
                if (isNaN(updateValue) || updateValue <= 0) return;
            }

            try {
                await updateDoc(trackerDocRef, {
                    [settingKey]: updateValue
                });
            } catch (error) {
                // Se o documento não existir, tenta criar
                if (error.code === 'not-found') {
                    // Cuidado: aqui temos que ter certeza que currentData reflete o estado atual da moeda
                    const docToSet = {
                        initialBankroll: currentData.initialBankroll,
                        transactions: currentData.transactions,
                        currency: currentData.currency,
                        riskPercentage: currentData.riskPercentage,
                        fixedUnitValue: currentData.fixedUnitValue,
                        riskMode: currentData.riskMode,
                        returnPercentage: currentData.returnPercentage
                    };
                    docToSet[settingKey] = updateValue;
                    await setDoc(trackerDocRef, docToSet);
                } else {
                    console.error(`Erro ao atualizar configuração ${settingKey}:`, error);
                }
            }
        };


        // Atualiza a Banca Inicial no Firestore
        window.updateInitialBankroll = async (value) => {
            if (!isAuthReady || !trackerDocRef) return;

            const newBankroll = parseFloat(value);
            if (isNaN(newBankroll) || newBankroll < 0) return;

            try {
                await updateDoc(trackerDocRef, {
                    initialBankroll: newBankroll
                });
            } catch (error) {
                // Se o documento não existir, tenta criar
                if (error.code === 'not-found') {
                     await setDoc(trackerDocRef, {
                        ...currentData, // Inclui as configurações da moeda atual
                        initialBankroll: newBankroll
                    });
                } else {
                    console.error("Erro ao atualizar banca inicial:", error);
                }
            }
        };

        // Regista uma transação automática baseada na Unidade ou Soros/Gale
        window.registerAutoTransaction = async (isWin, strategyLevel) => {
            if (!isAuthReady || !trackerDocRef) return;
            
            const initialInput = document.getElementById('initialBankrollInput');
            let bankrollToCheck = currentData.initialBankroll;

            // FIX DE SINCRONIZAÇÃO: Se a banca em currentData for 0, usa o valor do input (caso o usuário tenha digitado rápido)
            if (bankrollToCheck <= 0) {
                const uiBankroll = parseFloat(initialInput.value);
                if (uiBankroll > 0) {
                    await updateInitialBankroll(uiBankroll);
                    bankrollToCheck = uiBankroll;
                    currentData.initialBankroll = uiBankroll; 
                }
            }
            
            if (bankrollToCheck <= 0) {
                console.error("Erro: Banca Inicial não definida. Por favor, defina a Banca Inicial primeiro.");
                initialInput.classList.add('error-highlight');
                setTimeout(() => { initialInput.classList.remove('error-highlight'); }, 3000);
                initialInput.focus();
                return;
            }
            
            // Determina a Unidade de Risco
            let unitStake = 0;
            let currentBankroll = currentData.initialBankroll;
            currentData.transactions.forEach(t => { currentBankroll += t.result; });
            const bankrollForRisk = Math.max(0, currentBankroll); 

            if (currentData.riskMode === 'percentage') {
                const riskRatio = currentData.riskPercentage / 100;
                const maxRiskTotal = bankrollForRisk * riskRatio;
                unitStake = maxRiskTotal / 3;
            } else {
                unitStake = currentData.fixedUnitValue;
            }
            
            if (unitStake <= 0) {
                 console.warn("Valor da Unidade não definido ou zero. Não é possível calcular nova Unidade de risco.");
                 return;
            }

            const returnRatio = currentData.returnPercentage / 100;
            let investedAmount = unitStake;
            let description = '';
            let lossAmount = 0;
            let currencySymbol = currentData.currency === 'BRL' ? 'R$' : (currentData.currency === 'EUR' ? '€' : '$');

            // Filtra as últimas transações de Martingale/Soros
            const allAutoTransactions = currentData.transactions
                .slice()
                .sort((a, b) => b.timestamp - a.timestamp) // Mais recente primeiro
                .filter(t => t.description.includes("Unidade") || t.description.includes("Gale") || t.description.includes("Soros"));

            
            // --- Lógica de Soros (Nível 3) ---
            if (strategyLevel === 3) {
                
                // Espera-se que a última operação (índice 0) seja uma Vitória de Unidade/Soros
                if (allAutoTransactions.length === 0 || allAutoTransactions[0].result <= 0) {
                     console.error("Erro Soros 2: Deve ser usado após uma Vitória de Unidade ou Soros 1.");
                     return;
                }
                
                // O lucro da última operação é o valor a ser reinvestido
                const lastProfit = allAutoTransactions[0].result;
                const lastInvested = allAutoTransactions[0].investedAmount;

                // Soros Nível 2 Stake: Lucro anterior + Unidade Padrão (stake)
                // Usaremos o lucro líquido da última operação como a stake extra
                investedAmount = lastProfit + unitStake;
                lossAmount = investedAmount;

                if (isWin) {
                    result = investedAmount * returnRatio;
                    description = `Vitória (Soros 2)`;
                } else {
                    result = -lossAmount;
                    description = `Derrota (Soros 2) - Perda do Lucro Anterior`;
                }

            // --- Lógica do Gale (Nível 1, 2 e Saída de Ciclo) ---
            } else if (strategyLevel === 1 || strategyLevel === 2 || (isWin && strategyLevel === 1)) {
                
                let accumulatedLoss = 0;

                // 1. Cálculo da Perda Acumulada
                if (strategyLevel === 1) {
                    // Gale 1: Espera-se perda de Unidade
                    if (allAutoTransactions.length === 0 || allAutoTransactions[0].result >= 0 || allAutoTransactions[0].description.includes("Gale")) {
                         console.error("Erro Gale 1: Deve ser usado após uma Derrota de Unidade.");
                         return;
                    }
                    accumulatedLoss = Math.abs(allAutoTransactions[0].result);
                } else if (strategyLevel === 2) {
                    // Gale 2: Espera-se Derrota Unidade + Derrota Gale 1
                    if (allAutoTransactions.length < 2 || allAutoTransactions[0].result >= 0 || allAutoTransactions[1].result >= 0) {
                        console.error("Erro Gale 2: Deve ser usado após Derrota de Unidade e Derrota Gale 1 consecutivas.");
                        return;
                    }
                    accumulatedLoss = Math.abs(allAutoTransactions[0].result) + Math.abs(allAutoTransactions[1].result);
                } else if (isWin && strategyLevel === 1) {
                    // Saída Gale: A stake da vitória deve ser calculada para cobrir a última perda.
                     if (allAutoTransactions.length === 0 || allAutoTransactions[0].result >= 0) {
                         console.error("Erro Vitória Gale: Não há uma perda anterior para recuperar.");
                         return;
                    }
                    accumulatedLoss = Math.abs(allAutoTransactions[0].result);
                }

                
                // 2. Cálculo do Investimento (para lucro + recuperação)
                const desiredProfit = unitStake * returnRatio; 
                investedAmount = (accumulatedLoss + desiredProfit) / returnRatio;
                lossAmount = investedAmount; 
                
                if (isWin) {
                    // Saída do Ciclo Gale (Vitória)
                    result = desiredProfit;
                    description = `Vitória Gale (Recuperou ${formatCurrency(accumulatedLoss)})`;
                } else {
                    // Derrota no Gale (Nível 1 ou 2)
                    result = -lossAmount;
                    description = `Derrota (Gale ${strategyLevel})`;
                }

            // --- Lógica da Unidade Padrão (Vitória/Derrota) ---
            } else {
                // Operação de Unidade Padrão (galeLevel = 0)
                investedAmount = unitStake;
                lossAmount = unitStake;
                
                if (isWin) {
                    // Vitória da Unidade Padrão
                    result = unitStake * returnRatio;
                    description = `Vitória (Unidade)`;
                } else {
                    // Derrota da Unidade Padrão
                    result = -lossAmount * 1.00;
                    description = `❌ Derrota (Unidade)`;
                }
            }


            const newTransaction = {
                timestamp: Date.now(), 
                description,
                investedAmount: parseFloat(investedAmount.toFixed(2)), 
                result: parseFloat(result.toFixed(2)) 
            };

            try {
                await updateDoc(trackerDocRef, {
                    transactions: arrayUnion(newTransaction)
                });
            } catch (error) {
                 if (error.code === 'not-found') {
                    await setDoc(trackerDocRef, {
                        ...currentData, // Inclui as configurações
                        transactions: [newTransaction]
                    });
                } else {
                    console.error("Erro ao adicionar transação automática:", error);
                }
            }
        };

        // Função para eliminar uma transação
        window.deleteTransaction = async (timestamp, result, investedAmount) => {
            if (!isAuthReady || !trackerDocRef) return;

            const transactionToDelete = currentData.transactions.find(t => 
                t.timestamp === timestamp && 
                Math.abs(t.result - result) < 0.01 && 
                Math.abs(t.investedAmount - investedAmount) < 0.01
            );
            
            if (!transactionToDelete) {
                console.error("Erro: Transação não encontrada para eliminação.");
                return;
            }

            try {
                await updateDoc(trackerDocRef, {
                    transactions: arrayRemove(transactionToDelete)
                });
                console.log("Transação eliminada com sucesso.");
            } catch (error) {
                console.error("Erro ao eliminar transação:", error);
            }
        };


        // Adiciona uma nova transação manual ao Firestore (Aportes/Outros)
        document.getElementById('transactionForm').addEventListener('submit', async (e) => {
            e.preventDefault();
            if (!isAuthReady || !trackerDocRef) return;

            const description = document.getElementById('description').value;
            const investedAmount = parseFloat(document.getElementById('invested').value);
            const result = parseFloat(document.getElementById('result').value);

            if (isNaN(investedAmount) || isNaN(result)) {
                console.error("Valores inválidos para investimento/resultado.");
                return;
            }

            const newTransaction = {
                timestamp: Date.now(),
                description,
                investedAmount,
                result 
            };

            try {
                await updateDoc(trackerDocRef, {
                    transactions: arrayUnion(newTransaction)
                });

                document.getElementById('transactionForm').reset();
            } catch (error) {
                 if (error.code === 'not-found') {
                    await setDoc(trackerDocRef, {
                        ...currentData, // Inclui as configurações
                        transactions: [newTransaction]
                    });
                    document.getElementById('transactionForm').reset();
                } else {
                    console.error("Erro ao adicionar transação:", error);
                }
            }
        });

        // Inicia o aplicativo Firebase
        window.onload = initFirebase;

    </script>
}
