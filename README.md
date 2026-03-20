<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>TripGenie x Dify 5-Step Logic Demo</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <style>
        @keyframes pulse-glow {
            0%, 100% { box-shadow: 0 0 5px rgba(59, 130, 246, 0.5); }
            50% { box-shadow: 0 0 20px rgba(59, 130, 246, 0.8); }
        }
        .active-node {
            animation: pulse-glow 2s infinite;
            border-color: #3b82f6 !important;
            background-color: #eff6ff !important;
            z-index: 10;
        }
        .terminal-scroll::-webkit-scrollbar { width: 4px; }
        .terminal-scroll::-webkit-scrollbar-thumb { background: #4b5563; border-radius: 10px; }
        .no-scrollbar::-webkit-scrollbar { display: none; }

        .luggage-card {
            background: #ffffff;
            border: 1px solid #e5e7eb;
            border-radius: 12px;
            padding: 12px;
            margin-top: 8px;
        }
        .luggage-item {
            display: flex;
            justify-content: space-between;
            align-items: center;
            padding: 8px 0;
            border-bottom: 1px solid #f3f4f6;
        }
        .luggage-item:last-child { border-bottom: none; }
        .counter-group {
            display: flex;
            align-items: center;
            gap: 12px;
        }
        .counter-btn {
            width: 24px;
            height: 24px;
            border: 1px solid #3b82f6;
            border-radius: 50%;
            display: flex;
            align-items: center;
            justify-content: center;
            color: #3b82f6;
            font-size: 16px;
            cursor: pointer;
            transition: all 0.2s;
        }
        .counter-btn:active { background: #eff6ff; transform: scale(0.9); }
        .counter-btn.disabled { border-color: #d1d5db; color: #d1d5db; cursor: not-allowed; }
    </style>
</head>
<body class="bg-slate-50 font-sans h-screen flex items-center justify-center overflow-hidden text-slate-900">

<div class="w-full h-full max-w-[1440px] flex gap-4 p-4">
    
    <!-- LEFT: App Interface (Mobile) -->
    <div class="w-[380px] bg-white rounded-[3rem] border-[10px] border-gray-900 shadow-2xl relative flex flex-col overflow-hidden shrink-0">
        <div class="h-10 flex justify-between px-8 pt-4 items-center">
            <span class="text-xs font-bold font-mono">9:41</span>
            <div class="w-20 h-5 bg-black rounded-full"></div>
            <div class="flex gap-1 text-[10px] font-mono">100%</div>
        </div>

        <div class="flex-1 flex flex-col px-4 py-2">
            <div class="flex items-center mb-4">
                <div class="w-8 h-8 rounded-full bg-blue-50 flex items-center justify-center text-blue-600">
                    <svg class="w-4 h-4" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path d="M15 19l-7-7 7-7" stroke-width="2.5"/></svg>
                </div>
                <div class="ml-2">
                    <h2 class="text-xs font-bold">Transfer: Shanghai Hongqiao</h2>
                    <p class="text-[9px] text-gray-400">Mar 20, 15:00 · 3 Pax · Special Items Included</p>
                </div>
            </div>

            <div class="bg-white rounded-2xl shadow-sm border border-gray-100 flex-1 flex flex-col">
                <div class="p-3 border-b flex items-center gap-2">
                    <div class="w-6 h-6 bg-blue-600 rounded flex items-center justify-center">
                        <svg class="w-4 h-4 text-white" viewBox="0 0 24 24" fill="currentColor"><path d="M12 2L4.5 20.29l.71.71L12 18l6.79 3 .71-.71z"/></svg>
                    </div>
                    <span class="font-bold text-xs text-blue-600">TripGenie Luggage Assistant</span>
                </div>

                <div id="appChat" class="flex-1 p-3 overflow-y-auto space-y-3">
                    <div class="bg-gray-100 p-2.5 rounded-xl rounded-bl-none text-[11px] text-gray-700">
                        Hello! To recommend the most suitable vehicle, please provide your luggage details.
                    </div>
                    <div class="bg-blue-600 p-2.5 rounded-xl rounded-br-none text-[11px] text-white ml-auto max-w-[80%] shadow-sm">
                        I have 4 suitcases and a snowboard. What kind of car can I take?
                    </div>
                </div>

                <!-- Step 2: Interactive Luggage Selector Component -->
                <div id="followUpArea" class="px-4 py-3 border-t bg-blue-50/30 hidden">
                    <div class="flex items-center gap-1.5 mb-2">
                        <svg class="w-3.5 h-3.5 text-blue-600" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path d="M19 11H5m14 0a2 2 0 012 2v6a2 2 0 01-2 2H5a2 2 0 01-2-2v-6a2 2 0 012-2m14 0V9a2 2 0 00-2-2M5 11V9a2 2 0 012-2m0 0V5a2 2 0 012-2h6a2 2 0 012 2v2M7 7h10" stroke-width="2"/></svg>
                        <p class="text-[10px] text-blue-900 font-bold uppercase tracking-tight">Select Luggage Sizes & Quantity</p>
                    </div>
                    
                    <div class="luggage-card shadow-sm border-blue-100">
                        <div class="luggage-item">
                            <div>
                                <div class="text-[11px] font-bold">Carry-on (20'')</div>
                                <div class="text-[9px] text-gray-400">Backpacks / Small bags</div>
                            </div>
                            <div class="counter-group">
                                <button onclick="updateCount('c20', -1)" class="counter-btn disabled" id="btn-c20-minus">-</button>
                                <span class="text-xs font-mono w-4 text-center" id="count-c20">0</span>
                                <button onclick="updateCount('c20', 1)" class="counter-btn">+</button>
                            </div>
                        </div>
                        <div class="luggage-item">
                            <div>
                                <div class="text-[11px] font-bold">Medium (24'')</div>
                                <div class="text-[9px] text-gray-400">Standard checked bag</div>
                            </div>
                            <div class="counter-group">
                                <button onclick="updateCount('c24', -1)" class="counter-btn disabled" id="btn-c24-minus">-</button>
                                <span class="text-xs font-mono w-4 text-center" id="count-c24">0</span>
                                <button onclick="updateCount('c24', 1)" class="counter-btn">+</button>
                            </div>
                        </div>
                        <div class="luggage-item border-none">
                            <div>
                                <div class="text-[11px] font-bold">Large (28'')</div>
                                <div class="text-[9px] text-gray-400">Extra large checked bag</div>
                            </div>
                            <div class="counter-group">
                                <button onclick="updateCount('c28', -1)" class="counter-btn disabled" id="btn-c28-minus">-</button>
                                <span class="text-xs font-mono w-4 text-center" id="count-c28">0</span>
                                <button onclick="updateCount('c28', 1)" class="counter-btn">+</button>
                            </div>
                        </div>
                        
                        <button id="confirmLuggage" onclick="submitLuggage()" class="w-full mt-3 bg-blue-600 text-white py-2 rounded-lg font-bold text-[10px] shadow-sm active:bg-blue-700 transition-colors">
                            Confirm & Find Matches
                        </button>
                    </div>
                </div>

                <div class="p-3 border-t">
                    <button id="findBtn" class="w-full bg-blue-600 text-white py-3 rounded-xl font-bold text-xs shadow-md active:scale-95 transition-all">
                        Find matching ride
                    </button>
                </div>
            </div>
        </div>
        <div class="h-6 flex justify-center items-center pb-2">
            <div class="w-24 h-1 bg-gray-200 rounded-full"></div>
        </div>
    </div>

    <!-- RIGHT: Dify Backend -->
    <div class="flex-1 flex flex-col gap-4 overflow-hidden">
        
        <!-- Terminal (Log) -->
        <div class="h-[40%] bg-slate-900 rounded-2xl border border-slate-800 flex flex-col overflow-hidden shadow-2xl">
            <div class="bg-slate-800 px-4 py-2 flex items-center justify-between border-b border-slate-700">
                <div class="flex gap-1.5">
                    <div class="w-2.5 h-2.5 bg-red-500 rounded-full"></div>
                    <div class="w-2.5 h-2.5 bg-amber-500 rounded-full"></div>
                    <div class="w-2.5 h-2.5 bg-emerald-500 rounded-full"></div>
                </div>
                <span class="text-[9px] font-mono text-slate-400 uppercase tracking-widest">Dify Execution Engine v2.5</span>
            </div>
            <div id="terminal" class="flex-1 p-4 font-mono text-[10.5px] text-emerald-400 overflow-y-auto terminal-scroll leading-relaxed">
                <span class="text-slate-500">// Waiting for user trigger...</span>
            </div>
        </div>

        <!-- Architecture Flow -->
        <div class="flex-1 bg-white rounded-2xl border border-slate-200 p-5 overflow-y-auto shadow-sm flex flex-col gap-4 font-sans">
            
            <div class="grid grid-cols-2 gap-4">
                <div id="step-1" class="border border-slate-100 bg-slate-50/50 p-3 rounded-xl transition-all">
                    <div class="flex items-center gap-2 mb-2 text-blue-600 font-bold text-[11px]">
                        <span class="w-4 h-4 bg-blue-100 rounded-full flex items-center justify-center text-[9px]">1</span>
                        Step 1: Slot Filling
                    </div>
                    <div class="space-y-1 text-[9px] text-slate-500">
                        <div id="slot-city" class="flex justify-between"><span>Destination:</span> <span class="text-slate-800 font-medium">-</span></div>
                        <div id="slot-pax" class="flex justify-between"><span>Passengers:</span> <span class="text-slate-800 font-medium">-</span></div>
                        <div id="slot-luggage" class="flex justify-between"><span>Luggage:</span> <span class="text-slate-800 font-medium">-</span></div>
                        <div id="slot-special" class="flex justify-between"><span>Special Items:</span> <span class="text-slate-800 font-medium">-</span></div>
                    </div>
                </div>
                <div id="step-2" class="border border-slate-100 bg-slate-50/50 p-3 rounded-xl transition-all">
                    <div class="flex items-center gap-2 mb-2 text-amber-600 font-bold text-[11px]">
                        <span class="w-4 h-4 bg-amber-100 rounded-full flex items-center justify-center text-[9px]">2</span>
                        Step 2: Dynamic Component
                    </div>
                    <div class="text-[9px] text-slate-400 italic">
                        Strategy: Display LuggageSelector based on intent
                    </div>
                    <div id="follow-status" class="mt-2 text-[9px] text-amber-700 font-bold hidden">● Deploying LuggageSelector Component</div>
                </div>
            </div>

            <div id="step-3" class="border border-slate-100 bg-slate-50/50 p-3 rounded-xl transition-all">
                <div class="flex items-center gap-2 mb-2 text-emerald-600 font-bold text-[11px]">
                    <span class="w-4 h-4 bg-emerald-100 rounded-full flex items-center justify-center text-[9px]">3</span>
                    Step 3: Knowledge RAG
                </div>
                <div class="flex gap-2">
                    <div id="rag-gen" class="flex-1 bg-white p-2 rounded border border-slate-100 text-[9px] text-center">General Rules</div>
                    <div id="rag-map" class="flex-1 bg-white p-2 rounded border border-slate-100 text-[9px] text-center">City Fleet Map</div>
                    <div id="rag-spec" class="flex-1 bg-white p-2 rounded border border-slate-100 text-[9px] text-center font-bold text-emerald-600 shadow-sm border-emerald-100 bg-emerald-50/20">Special Item Protocol</div>
                </div>
            </div>

            <div id="step-4" class="border border-slate-100 bg-slate-50/50 p-3 rounded-xl transition-all">
                <div class="flex items-center gap-2 mb-2 text-indigo-600 font-bold text-[11px]">
                    <span class="w-4 h-4 bg-indigo-100 rounded-full flex items-center justify-center text-[9px]">4</span>
                    Step 4: Logic Engine
                </div>
                <div class="flex justify-around items-center px-4">
                    <span id="label-fit" class="px-2 py-1 rounded border text-[9px] text-slate-400 font-mono">fit</span>
                    <span id="label-risky" class="px-2 py-1 rounded border text-[9px] text-slate-400 font-mono">risky_fit</span>
                    <span id="label-not" class="px-2 py-1 rounded border text-[9px] text-slate-400 font-mono">not_fit</span>
                    <span id="label-unknown" class="px-2 py-1 rounded border text-[9px] text-slate-400 font-mono">unknown</span>
                </div>
            </div>

            <div id="step-5" class="border border-slate-100 bg-slate-50/50 p-3 rounded-xl transition-all">
                <div class="flex items-center gap-2 mb-2 text-purple-600 font-bold text-[11px]">
                    <span class="w-4 h-4 bg-purple-100 rounded-full flex items-center justify-center text-[9px]">5</span>
                    Step 5: Recommendation Output
                </div>
                <div id="final-out" class="bg-white p-2 border border-purple-100 rounded text-[9px] text-slate-500 italic">
                    Synthesizing optimal fleet solution...
                </div>
            </div>

        </div>
    </div>
</div>

<script>
    const findBtn = document.getElementById('findBtn');
    const terminal = document.getElementById('terminal');
    const appChat = document.getElementById('appChat');
    const followUpArea = document.getElementById('followUpArea');

    const nodes = {
        s1: document.getElementById('step-1'),
        s2: document.getElementById('step-2'),
        s3: document.getElementById('step-3'),
        s4: document.getElementById('step-4'),
        s5: document.getElementById('step-5'),
        followStatus: document.getElementById('follow-status'),
        slotLuggage: document.querySelector('#slot-luggage span:last-child'),
        slotSpecial: document.querySelector('#slot-special span:last-child'),
        slotCity: document.querySelector('#slot-city span:last-child'),
        slotPax: document.querySelector('#slot-pax span:last-child'),
        labelNot: document.getElementById('label-not'),
        labelRisky: document.getElementById('label-risky'),
        labelFit: document.getElementById('label-fit'),
        finalOut: document.getElementById('final-out')
    };

    let lugCounts = { c20: 0, c24: 0, c28: 0 };

    function updateCount(type, delta) {
        const newVal = Math.max(0, lugCounts[type] + delta);
        lugCounts[type] = newVal;
        document.getElementById(`count-${type}`).innerText = newVal;
        
        const minusBtn = document.getElementById(`btn-${type}-minus`);
        if(newVal > 0) minusBtn.classList.remove('disabled');
        else minusBtn.classList.add('disabled');
    }

    function log(text, type = 'info') {
        const div = document.createElement('div');
        div.className = 'mb-1.5';
        const ts = new Date().toLocaleTimeString([], { hour12: false, fractionalSecondDigits: 1 });
        let color = 'text-emerald-400';
        let prefix = 'LOG';
        
        if(type === 'step') { color = 'text-white font-bold'; prefix = 'STEP'; }
        if(type === 'thought') { color = 'text-purple-300'; prefix = 'AI'; }
        if(type === 'act') { color = 'text-amber-300'; prefix = 'ACTION'; }
        if(type === 'res') { color = 'text-blue-300'; prefix = 'RESULT'; }

        div.innerHTML = `<span class="text-slate-600">[${ts}]</span> <span class="bg-slate-800 px-1.5 py-0.5 rounded text-[9px] mr-2 text-slate-400 font-sans tracking-tighter">${prefix}</span> <span class="${color}">${text}</span>`;
        terminal.appendChild(div);
        terminal.scrollTop = terminal.scrollHeight;
    }

    function addAppMsg(text, isUser = false) {
        const div = document.createElement('div');
        div.className = isUser ? 
            "bg-blue-600 p-2.5 rounded-xl rounded-br-none text-[11px] text-white ml-auto max-w-[80%]" : 
            "bg-gray-100 p-2.5 rounded-xl rounded-bl-none text-[11px] text-gray-700";
        div.innerText = text;
        appChat.appendChild(div);
        appChat.scrollTop = appChat.scrollHeight;
    }

    async function sleep(ms) { return new Promise(r => setTimeout(r, ms)); }

    async function startExecution() {
        findBtn.disabled = true;
        findBtn.innerText = "Analyzing Intent...";
        terminal.innerHTML = '';
        
        log("Received input: [4 Suitcases + Snowboard]", "step");
        nodes.s1.classList.add('active-node');
        log("Step 1: Parsing user intent and extraction slots", "thought");
        await sleep(1000);
        nodes.slotLuggage.innerText = "4 Units (Sizes Unknown)";
        nodes.slotSpecial.innerText = "Snowboard x1";
        nodes.slotCity.innerText = "Shanghai Hongqiao";
        nodes.slotPax.innerText = "3 Persons";
        log("Slots Filled: {Destination: Shanghai Hongqiao, Pax: 3, Snowboard: 1}", "res");
        await sleep(600);
        nodes.s1.classList.remove('active-node');

        nodes.s2.classList.add('active-node');
        log("Step 2: Triggering dynamic component logic", "thought");
        nodes.followStatus.classList.remove('hidden');
        await sleep(1000);
        log("Action: Deploying LuggageSelector component to user", "act");
        addAppMsg("I've received your request. To recommend the perfect car (considering the length of your snowboard), please specify the size of your suitcases:");
        followUpArea.classList.remove('hidden');
        
        findBtn.innerText = "Please select in component";
    }

    async function submitLuggage() {
        const total = lugCounts.c20 + lugCounts.c24 + lugCounts.c28;
        if(total === 0) {
            alert("Please select at least one suitcase");
            return;
        }
        
        followUpArea.classList.add('hidden');
        const summary = `${lugCounts.c20}x20'', ${lugCounts.c24}x24'', ${lugCounts.c28}x28''`;
        addAppMsg(`My luggage: ${summary}`, true);
        
        findBtn.innerText = "Continuing analysis...";
        log(`User feedback: ${summary}`, "res");
        nodes.followStatus.innerText = "● Response Received";
        nodes.slotLuggage.innerText = summary;
        await sleep(800);
        nodes.s2.classList.remove('active-node');

        // --- STEP 3: Knowledge RAG ---
        nodes.s3.classList.add('active-node');
        log("Step 3: Querying RAG Knowledge Base", "step");
        log("Query: [Snowboard length storage] [Shanghai fleet capacity]", "act");
        await sleep(1200);
        log("Knowledge: Snowboards (>1.6m) require MPV with rear seats partially folded. Shanghai Hongqiao pick-up allows standard vans.", "res");
        await sleep(500);
        nodes.s3.classList.remove('active-node');

        // --- STEP 4: Logic Engine ---
        nodes.s4.classList.add('active-node');
        log("Step 4: Executing capacity and safety logic", "thought");
        await sleep(1200);
        
        if(lugCounts.c28 >= 2 || total > 4) {
            log("Evaluation: SEDAN/SUV insufficient for 3 pax + board + large bags.", "res");
            nodes.labelNot.classList.add('bg-red-100', 'text-red-700', 'border-red-300');
            nodes.labelNot.innerText = "not_fit (Standard)";
        } else {
            log("Evaluation: Moderate risk. High occupancy.", "res");
            nodes.labelRisky.classList.add('bg-amber-100', 'text-amber-700', 'border-amber-300');
        }
        await sleep(600);
        nodes.s4.classList.remove('active-node');

        // --- STEP 5: Output ---
        nodes.s5.classList.add('active-node');
        log("Step 5: Generating structured response", "step");
        await sleep(1000);
        
        const finalMsg = `【Match Found】Recommended: Luxury 6-Seater MPV (Business Van).
Reason: A standard sedan or SUV cannot accommodate 3 passengers, 4 suitcases, AND a long snowboard simultaneously. An MPV allows partial folding of the 3rd-row seats to secure your snowboard safely.
💡 Pro-tip: Ensure your snowboard is in a protective bag to prevent scratches during transport.`;
        
        nodes.finalOut.innerText = finalMsg;
        nodes.finalOut.classList.remove('italic');
        addAppMsg(finalMsg);
        log("Task complete.", "info");
        
        nodes.s5.classList.remove('active-node');
        findBtn.disabled = false;
        findBtn.innerText = "Restart Simulation";
        findBtn.classList.replace('bg-blue-600', 'bg-emerald-600');
    }

    findBtn.addEventListener('click', () => {
        if(findBtn.innerText.includes("Restart")) {
            location.reload();
        } else {
            startExecution();
        }
    });
</script>

</body>
</html>
