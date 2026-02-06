<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Sistema Integrado - Fast Food Service Co</title>
    
    <a href="https://github.com/romerobohorquezjhon-ship-it/corefeelin">View on GitHub</a>
    <script src="https://cdn.tailwindcss.com"></script>
    <link href="https://fonts.googleapis.com/css2?family=Inter:wght@300;400;600;800;900&display=swap" rel="stylesheet">
    
    <style>
        :root {
            --color-primary: #06b6d4;
            --color-primary-dark: #0891b2;
            --color-background: #111827;
            --color-text: #f9fafb;
            --color-card: #1f2937;
        }
        
        body {
            font-family: 'Inter', sans-serif;
            background-color: var(--color-background);
            color: var(--color-text);
            background-image: url('https://images.unsplash.com/photo-1555396273-367ea4eb4db5?ixlib=rb-4.0.3&auto=format&fit=crop&w=1374&q=80');
            background-size: cover;
            background-position: center;
            background-attachment: fixed;
            position: relative;
            margin: 0;
            padding: 0;
        }

        body::before {
            content: '';
            position: absolute;
            top: 0; left: 0; right: 0; bottom: 0;
            background: rgba(17, 24, 39, 0.96);
            z-index: 0;
        }

        .btn-primary {
            background-color: var(--color-primary);
            color: white;
            padding: 0.75rem 1.5rem;
            border-radius: 0.5rem;
            font-weight: 600;
            transition: all 0.2s;
            box-shadow: 0 4px 6px -1px rgba(0, 0, 0, 0.1);
            cursor: pointer;
        }

        .btn-primary:hover {
            background-color: var(--color-primary-dark);
            transform: translateY(-1px);
        }

        .card {
            background-color: var(--color-card);
            border-radius: 1rem;
            padding: 1.5rem;
            box-shadow: 0 10px 15px -3px rgba(0, 0, 0, 0.5);
            border: 1px solid #374151;
            position: relative;
            z-index: 10;
        }

        .input-style {
            background-color: #374151;
            color: var(--color-text);
            border: 1px solid #4b5563;
            padding: 0.75rem;
            border-radius: 0.5rem;
            width: 100%;
            outline: none;
        }

        .input-style:focus {
            border-color: var(--color-primary);
        }

        .modal-overlay {
            background-color: rgba(0, 0, 0, 0.85);
            backdrop-filter: blur(4px);
        }

        .custom-scrollbar::-webkit-scrollbar { width: 5px; }
        .custom-scrollbar::-webkit-scrollbar-track { background: transparent; }
        .custom-scrollbar::-webkit-scrollbar-thumb { background: #4b5563; border-radius: 10px; }
        
        .order-card-kitchen {
            border-left: 4px solid var(--color-primary);
            animation: slideIn 0.3s ease-out;
        }

        @keyframes slideIn {
            from { opacity: 0; transform: translateX(20px); }
            to { opacity: 1; transform: translateX(0); }
        }

        #creator-watermark {
            position: fixed;
            bottom: 10px;
            right: 10px;
            font-size: 10px;
            opacity: 0.3;
            color: white;
            z-index: 50;
        }
    </style>
</head>
<body class="min-h-screen flex flex-col items-center p-4">

    <!-- Header Dinámico -->
    <header id="app-header" class="w-full max-w-xl mb-6 hidden z-10">
        <div class="flex justify-between items-center bg-gray-900/80 p-4 rounded-xl border border-gray-700 shadow-xl backdrop-blur-md">
            <div>
                <p id="role-label" class="text-[10px] text-cyan-400 uppercase font-black tracking-widest"></p>
                <p id="user-display-name" class="text-lg font-bold text-white"></p>
            </div>
            <div class="text-right">
                <p class="text-[10px] text-gray-500 uppercase font-black tracking-widest">Fecha</p>
                <p id="header-date" class="text-sm font-medium text-gray-300"></p>
            </div>
        </div>
    </header>

    <main id="app" class="w-full max-w-xl z-10 relative">
        <!-- Pantallas dinámicas aquí -->
    </main>

    <div id="creator-watermark">FAST FOOD SERVICE CO. v1.0</div>

    <!-- Modal de Notificaciones -->
    <div id="message-modal" class="fixed inset-0 hidden items-center justify-center z-50 modal-overlay p-4">
        <div class="bg-gray-800 border border-gray-700 p-8 rounded-2xl shadow-2xl w-full max-w-sm text-center">
            <h3 id="modal-title" class="text-xl font-bold mb-2 text-cyan-400"></h3>
            <p id="modal-body" class="mb-6 text-gray-300 font-medium"></p>
            <button onclick="closeModal()" class="btn-primary w-full uppercase tracking-widest text-xs">Aceptar</button>
        </div>
    </div>

    <script>
        // --- CONSTANTES Y ESTADO ---
        const appElement = document.getElementById('app');
        const headerElement = document.getElementById('app-header');
        
        let currentRole = ''; 
        let currentUser = '';
        let currentScreen = 'roleSelection';
        let pendingOrders = []; 

        let currentOrder = {
            waiter: '',
            location: { floor: '', table: '' },
            items: [],
            total: 0,
            timestamp: null
        };

        const MENU = {
            "Desayunos": [
                { name: 'Calentao con Huevo', price: 12000 },
                { name: 'Arepa con Queso', price: 8000 },
                { name: 'Chocolate Santafereño', price: 7000 }
            ],
            "Almuerzos": [
                { name: 'Bandeja Paisa', price: 28000 },
                { name: 'Ajiaco Santafereño', price: 25000 },
                { name: 'Sancocho de Gallina', price: 22000 }
            ],
            "Comidas Rápidas": [
                { name: 'Empanadas (3 un.)', price: 9000 },
                { name: 'Salchipapa Especial', price: 15000 },
                { name: 'Perro Caliente', price: 14000 }
            ]
        };

        // --- FUNCIONES DE UTILIDAD ---
        const formatCurrency = (val) => new Intl.NumberFormat('es-CO', { 
            style: 'currency', 
            currency: 'COP', 
            minimumFractionDigits: 0 
        }).format(val);

        const showModal = (title, body) => {
            document.getElementById('modal-title').textContent = title;
            document.getElementById('modal-body').textContent = body;
            const modal = document.getElementById('message-modal');
            modal.classList.remove('hidden');
            modal.classList.add('flex');
        };

        const closeModal = () => {
            const modal = document.getElementById('message-modal');
            modal.classList.add('hidden');
            modal.classList.remove('flex');
        };

        const updateHeader = () => {
            if (!currentUser) {
                headerElement.classList.add('hidden');
                return;
            }
            headerElement.classList.remove('hidden');
            document.getElementById('role-label').textContent = currentRole === 'mesero' ? 'Mesero Activo' : 'Jefe de Cocina';
            document.getElementById('user-display-name').textContent = currentUser;
            document.getElementById('header-date').textContent = new Date().toLocaleDateString('es-CO', { day: 'numeric', month: 'short' });
        };

        // --- NAVEGACIÓN ---
        const goToScreen = (screen) => {
            currentScreen = screen;
            render();
            window.scrollTo(0, 0);
        };

        // --- LÓGICA DE USUARIO ---
        window.selectRole = (role) => {
            currentRole = role;
            goToScreen('login');
        };

        window.login = () => {
            const input = document.getElementById('name-input');
            const name = input ? input.value.trim() : "";
            if (name.length < 3) return showModal('Error', 'Ingresa un nombre válido (mín. 3 letras).');
            
            currentUser = name;
            updateHeader();
            
            if (currentRole === 'mesero') {
                currentOrder.waiter = currentUser;
                goToScreen('selectLocation');
            } else {
                goToScreen('kitchenDashboard');
            }
        };

        window.logout = () => {
            currentRole = '';
            currentUser = '';
            updateHeader();
            goToScreen('roleSelection');
        };

        // --- LÓGICA DE MESERO ---
        window.confirmLocation = () => {
            const floor = document.getElementById('floor-select').value;
            const table = document.getElementById('table-select').value;
            if (!floor || !table) return showModal('Faltan Datos', 'Selecciona el piso y la mesa.');
            
            currentOrder.location = { floor, table };
            goToScreen('waiterMenu');
        };

        window.addToOrder = (name, price) => {
            const existing = currentOrder.items.find(i => i.name === name);
            if (existing) {
                existing.quantity++;
            } else {
                currentOrder.items.push({ name, price, quantity: 1 });
            }
            calculateOrderTotal();
            render();
        };

        window.updateQuantity = (name, delta) => {
            const item = currentOrder.items.find(i => i.name === name);
            if (item) {
                item.quantity += delta;
                if (item.quantity <= 0) {
                    currentOrder.items = currentOrder.items.filter(i => i.name !== name);
                }
            }
            calculateOrderTotal();
            render();
        };

        const calculateOrderTotal = () => {
            currentOrder.total = currentOrder.items.reduce((s, i) => s + (i.price * i.quantity), 0);
        };

        window.sendOrderToKitchen = () => {
            if (currentOrder.items.length === 0) return showModal('Pedido Vacío', 'Agrega productos antes de enviar.');
            
            const newOrder = {
                ...JSON.parse(JSON.stringify(currentOrder)), // Clonar objeto
                id: Date.now(),
                timestamp: new Date().toLocaleTimeString([], { hour: '2-digit', minute: '2-digit' })
            };
            
            pendingOrders.push(newOrder);
            showModal('¡Enviado!', `El pedido para la Mesa ${currentOrder.location.table} está en cocina.`);
            
            // Limpiar pedido y volver
            currentOrder.items = [];
            currentOrder.total = 0;
            goToScreen('selectLocation');
        };

        // --- LÓGICA DE COCINA ---
        window.completeOrder = (orderId) => {
            pendingOrders = pendingOrders.filter(o => o.id !== orderId);
            render();
        };

        // --- RENDERIZADO DE PANTALLAS ---
        const render = () => {
            let html = '';

            switch(currentScreen) {
                case 'roleSelection':
                    html = `
                        <div class="card text-center p-10 border-t-4 border-cyan-500">
                            <h1 class="text-3xl font-black text-white mb-2">BIENVENIDOS</h1>
                            <p class="text-xs text-gray-400 mb-10 tracking-[0.3em] uppercase">Gestión de Pedidos</p>
                            <div class="grid gap-4">
                                <button onclick="selectRole('mesero')" class="btn-primary py-6 text-lg flex items-center justify-center gap-3">
                                    <span>👤</span> PERFIL MESERO
                                </button>
                                <button onclick="selectRole('cocinero')" class="bg-amber-600 hover:bg-amber-700 text-white font-black py-6 rounded-lg text-lg flex items-center justify-center gap-3 transition-colors">
                                    <span>🍳</span> PERFIL COCINERO
                                </button>
                            </div>
                        </div>
                    `;
                    break;

                case 'login':
                    html = `
                        <div class="card p-8">
                            <button onclick="goToScreen('roleSelection')" class="text-xs text-cyan-400 font-bold mb-4 uppercase">← Volver</button>
                            <h2 class="text-2xl font-black text-white mb-2 uppercase">Identificación</h2>
                            <p class="text-xs text-gray-400 mb-6 uppercase tracking-widest">Ingresa tu nombre para el turno</p>
                            <input type="text" id="name-input" placeholder="Nombre completo" class="input-style mb-4 text-center text-lg">
                            <button onclick="login()" class="btn-primary w-full py-4 uppercase font-black tracking-widest">Iniciar Sesión</button>
                        </div>
                    `;
                    break;

                case 'selectLocation':
                    html = `
                        <div class="card border-l-4 border-cyan-500">
                            <h2 class="text-xl font-black text-white mb-6 uppercase">Ubicación del Servicio</h2>
                            <div class="space-y-4">
                                <select id="floor-select" class="input-style" onchange="document.getElementById('table-select').disabled = false">
                                    <option value="" disabled selected>Selecciona Piso</option>
                                    <option value="Piso 1">Piso 1 (Principal)</option>
                                    <option value="Piso 2">Piso 2 (VIP)</option>
                                    <option value="Terraza">Terraza</option>
                                </select>
                                <select id="table-select" class="input-style" disabled>
                                    <option value="" disabled selected>Selecciona Mesa</option>
                                    ${[1,2,3,4,5,6,7,8].map(n => `<option value="${n}">Mesa ${n}</option>`).join('')}
                                </select>
                                <button onclick="confirmLocation()" class="btn-primary w-full py-4 font-black uppercase tracking-widest">Abrir Comanda</button>
                                <button onclick="logout()" class="w-full text-[10px] text-gray-500 font-bold mt-4 uppercase">Cerrar Sesión</button>
                            </div>
                        </div>
                    `;
                    break;

                case 'waiterMenu':
                    let menuContent = '';
                    for (let cat in MENU) {
                        menuContent += `<h3 class="text-[10px] font-black text-cyan-400 mt-6 mb-2 uppercase tracking-widest">${cat}</h3>`;
                        MENU[cat].forEach(item => {
                            menuContent += `
                                <div class="flex items-center justify-between bg-gray-800/40 p-3 rounded-lg border border-gray-700 mb-2">
                                    <div>
                                        <p class="text-xs font-bold text-white">${item.name}</p>
                                        <p class="text-[10px] text-cyan-400 font-black">${formatCurrency(item.price)}</p>
                                    </div>
                                    <button onclick="addToOrder('${item.name}', ${item.price})" class="bg-cyan-600 hover:bg-cyan-500 w-10 h-10 rounded-lg font-black transition-colors text-white">+</button>
                                </div>
                            `;
                        });
                    }

                    const cartSummary = currentOrder.items.map(i => `
                        <div class="flex justify-between items-center bg-gray-900/60 p-2 rounded-lg mb-2 text-[10px]">
                            <span class="font-bold text-white">${i.name}</span>
                            <div class="flex items-center gap-2">
                                <button onclick="updateQuantity('${i.name}', -1)" class="text-red-400 font-bold px-2">-</button>
                                <span class="text-cyan-400 font-bold min-w-[20px] text-center">${i.quantity}</span>
                                <button onclick="updateQuantity('${i.name}', 1)" class="text-green-400 font-bold px-2">+</button>
                            </div>
                        </div>
                    `).join('') || '<p class="text-center text-gray-500 text-[9px] italic py-2">No hay items</p>';

                    html = `
                        <div class="card p-5">
                            <div class="flex justify-between items-center mb-4 border-b border-gray-700 pb-2">
                                <h2 class="text-sm font-black text-white uppercase">Mesa ${currentOrder.location.table} (${currentOrder.location.floor})</h2>
                                <button onclick="goToScreen('selectLocation')" class="text-[9px] text-gray-500 font-bold uppercase">Cambiar Mesa</button>
                            </div>
                            
                            <div class="max-h-[40vh] overflow-y-auto custom-scrollbar pr-2">
                                ${menuContent}
                            </div>

                            <div class="mt-6 pt-4 border-t-2 border-dashed border-gray-700">
                                <h4 class="text-[9px] font-black text-gray-400 mb-2 uppercase tracking-widest text-center">Resumen de Pedido</h4>
                                <div class="max-h-32 overflow-y-auto custom-scrollbar">${cartSummary}</div>
                                <div class="mt-4 flex justify-between items-center">
                                    <div>
                                        <p class="text-[9px] text-gray-500 uppercase">Total a Pagar</p>
                                        <p class="text-xl font-black text-cyan-400">${formatCurrency(currentOrder.total)}</p>
                                    </div>
                                    <button onclick="sendOrderToKitchen()" class="btn-primary px-6 py-4 uppercase font-black text-xs">Enviar Cocina</button>
                                </div>
                            </div>
                        </div>
                    `;
                    break;

                case 'kitchenDashboard':
                    const kitchenCards = pendingOrders.length > 0 
                        ? pendingOrders.map(o => `
                            <div class="card order-card-kitchen mb-4 bg-gray-800/80">
                                <div class="flex justify-between items-start mb-3 border-b border-gray-700 pb-2">
                                    <div>
                                        <h4 class="text-sm font-black text-white uppercase">Mesa ${o.location.table} - ${o.location.floor}</h4>
                                        <p class="text-[9px] text-gray-400">Mesero: ${o.waiter} • ${o.timestamp}</p>
                                    </div>
                                    <span class="bg-amber-900/30 text-amber-400 text-[8px] px-2 py-1 rounded border border-amber-600 font-black uppercase">En Cola</span>
                                </div>
                                <div class="space-y-1 mb-4">
                                    ${o.items.map(i => `
                                        <div class="flex justify-between text-xs">
                                            <span class="text-gray-300"><span class="font-black text-cyan-400">${i.quantity}x</span> ${i.name}</span>
                                        </div>
                                    `).join('')}
                                </div>
                                <button onclick="completeOrder(${o.id})" class="w-full bg-green-600 hover:bg-green-700 text-white font-black py-3 rounded-lg text-xs uppercase transition-colors">Entregar Pedido</button>
                            </div>
                        `).join('')
                        : `<div class="card text-center py-20 opacity-50"><p class="italic text-gray-400">No hay órdenes pendientes...</p></div>`;

                    html = `
                        <div class="w-full">
                            <div class="flex justify-between items-center mb-6">
                                <h2 class="text-2xl font-black text-white uppercase tracking-tighter">Monitor de Cocina</h2>
                                <button onclick="logout()" class="text-[10px] text-gray-500 font-bold uppercase">Salir</button>
                            </div>
                            <div class="grid gap-2">
                                ${kitchenCards}
                            </div>
                        </div>
                    `;
                    break;
            }

            appElement.innerHTML = html;
        };

        // Iniciar
        document.addEventListener('DOMContentLoaded', () => {
            render();
        });
    </script>
</body>

</html>
