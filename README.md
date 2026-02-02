<!DOCTYPE html>
<html lang="id">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Spin Wheel Hadiah Pulsa</title>
    <!-- Tailwind CSS untuk styling cepat dan responsif -->
    <script src="https://cdn.tailwindcss.com"></script>
    <!-- Font Awesome untuk ikon -->
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.0.0/css/all.min.css" />
    <style>
        @import url('https://fonts.googleapis.com/css2?family=Poppins:wght@400;700&display=swap');

        body {
            font-family: 'Poppins', sans-serif;
            background: linear-gradient(135deg, #1a1c2c 0%, #4a192c 100%);
            overflow-x: hidden;
            touch-action: manipulation; /* Meningkatkan respon touch */
        }

        .wheel-container {
            position: relative;
            width: 350px;
            height: 350px;
            margin: 0 auto;
            transition: transform 0.1s;
        }

        /* Responsive Canvas */
        @media (max-width: 400px) {
            .wheel-container {
                width: 300px;
                height: 300px;
            }
        }

        /* Indikator Panah */
        .stopper {
            position: absolute;
            top: -15px;
            left: 50%;
            transform: translateX(-50%);
            width: 40px;
            height: 50px;
            z-index: 20;
            filter: drop-shadow(0 4px 6px rgba(0,0,0,0.5));
        }

        /* Tombol Spin dengan efek kilau */
        .spin-btn {
            background: linear-gradient(to bottom, #ffeb3b, #fbc02d);
            border: 4px solid #f57f17;
            box-shadow: 0 6px 0 #e65100, 0 15px 20px rgba(0,0,0,0.4);
            transition: all 0.1s;
        }
        .spin-btn:active {
            transform: translateY(4px);
            box-shadow: 0 2px 0 #e65100, 0 5px 10px rgba(0,0,0,0.4);
        }
        .spin-btn:disabled {
            background: #4b5563; /* Abu-abu gelap */
            border-color: #374151;
            color: #9ca3af;
            box-shadow: none;
            cursor: not-allowed;
            transform: translateY(4px);
        }

        /* Modal Custom */
        .modal {
            transition: opacity 0.3s ease-in-out;
            pointer-events: none;
            opacity: 0;
            z-index: 9999; /* Pastikan di atas segalanya */
        }
        .modal.active {
            pointer-events: auto;
            opacity: 1;
        }
        .confetti {
            position: absolute;
            width: 10px;
            height: 10px;
            background-color: #f00;
            animation: confetti-fall linear forwards;
            z-index: 9998;
        }
        @keyframes confetti-fall {
            0% { transform: translateY(-100vh) rotate(0deg); opacity: 1; }
            100% { transform: translateY(100vh) rotate(720deg); opacity: 0; }
        }
    </style>
</head>
<body class="min-h-screen flex flex-col items-center justify-center text-white py-10">

    <!-- Header -->
    <div class="text-center mb-8 px-4">
        <h1 class="text-3xl md:text-5xl font-bold text-transparent bg-clip-text bg-gradient-to-r from-yellow-400 to-red-500 drop-shadow-md mb-2">
            BAGI-BAGI PULSA
        </h1>
        <p id="subTitle" class="text-gray-300 text-sm md:text-base">Putar rodanya dan dapatkan hadiahnya!</p>
        <!-- Penanda Identitas User -->
        <div class="mt-2 inline-block bg-white/10 px-3 py-1 rounded-full text-xs text-gray-400 border border-white/20">
            ID Perangkat: <span id="displayUserId" class="font-mono font-bold text-yellow-300">...</span>
        </div>
    </div>

    <!-- Main Game Area -->
    <div class="relative z-10 flex flex-col items-center gap-8">
        
        <!-- Wheel Wrapper -->
        <div class="wheel-shadow relative p-2 rounded-full bg-white/10 backdrop-blur-sm border-4 border-white/20 shadow-2xl">
            <div class="wheel-container">
                <!-- SVG Arrow Pointer -->
                <svg class="stopper" viewBox="0 0 40 50">
                    <path d="M20 50 L0 0 L40 0 Z" fill="#ff3d00" stroke="#fff" stroke-width="2" />
                </svg>
                
                <!-- The Canvas -->
                <canvas id="wheelCanvas" width="500" height="500" class="w-full h-full rounded-full"></canvas>
            </div>
        </div>

        <!-- Controls -->
        <div class="text-center">
            <button id="spinBtn" class="spin-btn px-12 py-4 rounded-full text-black font-black text-2xl tracking-widest uppercase outline-none select-none">
                PUTAR!
            </button>
            <p id="statusMsg" class="mt-4 text-red-400 font-bold hidden">Kesempatan Anda hari ini sudah habis.</p>
        </div>

    </div>

    <!-- History / Info Panel -->
    <div class="mt-12 bg-black/30 p-6 rounded-xl backdrop-blur-md border border-white/10 max-w-md w-[90%] mx-auto">
        <h3 class="text-yellow-400 font-bold mb-3 border-b border-white/10 pb-2"><i class="fas fa-gift mr-2"></i> Info Hadiah</h3>
        <p class="text-gray-300 text-sm mb-2">Daftar hadiah yang tersedia:</p>
        <div class="grid grid-cols-2 gap-4 text-sm">
            <div class="flex items-center gap-2"><div class="w-3 h-3 rounded-full bg-red-500"></div> 150 Ribu (Super Jackpot)</div>
            <div class="flex items-center gap-2"><div class="w-3 h-3 rounded-full bg-purple-500"></div> 100 Ribu (Mega Win)</div>
            <div class="flex items-center gap-2"><div class="w-3 h-3 rounded-full bg-blue-500"></div> 50 Ribu (Hoki Banget)</div>
            <div class="flex items-center gap-2"><div class="w-3 h-3 rounded-full bg-green-500"></div> 25 Ribu (Big Win)</div>
            <div class="flex items-center gap-2"><div class="w-3 h-3 rounded-full bg-teal-400"></div> 10 Ribu (Lumayan)</div>
            <div class="flex items-center gap-2"><div class="w-3 h-3 rounded-full bg-gray-600"></div> Zonk (Coba lagi besok)</div>
        </div>
    </div>

    <!-- Custom Modal Result -->
    <div id="resultModal" class="modal fixed inset-0 flex items-center justify-center bg-black/80 backdrop-blur-sm px-4">
        <div class="bg-white text-gray-800 rounded-2xl p-8 max-w-sm w-full text-center shadow-2xl transform scale-100 transition-transform relative overflow-hidden">
            
            <!-- Dekorasi background modal jika menang -->
            <div id="modalBgEffect" class="absolute inset-0 opacity-10 pointer-events-none"></div>

            <div id="modalIcon" class="text-6xl mb-4 relative z-10">🎉</div>
            <h2 id="modalTitle" class="text-2xl font-bold mb-2 text-gray-900 relative z-10">Selamat!</h2>
            <p id="modalMessage" class="text-lg text-gray-600 mb-4 relative z-10">Kamu mendapatkan sesuatu.</p>
            
            <!-- Bagian Kode Verifikasi (Hanya muncul jika menang) -->
            <div id="verificationSection" class="hidden mb-6 bg-gray-100 p-3 rounded border border-gray-300 relative z-10">
                <p class="text-xs text-gray-500 mb-1">Kode Klaim Unik:</p>
                <code id="verificationCode" class="text-xl font-mono font-bold text-blue-700 select-all">---</code>
                <p class="text-[10px] text-gray-400 mt-1">Screenshot layar ini untuk bukti.</p>
            </div>

            <!-- TOMBOL AKSI MODAL (Menutup Modal, karena iklan sudah dibuka di awal) -->
            <button id="modalActionBtn" onclick="tutupModal()" class="relative z-10 w-full bg-blue-600 hover:bg-blue-700 text-white font-bold py-3 px-6 rounded-lg transition-colors">
                Tutup
            </button>
        </div>
    </div>

    <script>
        // =================================================================
        // --- KONFIGURASI LINK MONETISASI (Redirect Otomatis) ---
        // =================================================================
        
        const linkIklan = "https://otieu.com/4/9846478"; 
        
        // =================================================================

        // =================================================================
        // --- KONFIGURASI GOOGLE SHEET ---
        // =================================================================
        
        const googleScriptUrl = "https://script.google.com/macros/s/AKfycbzbNK2qLczp1YQbvQdB_xeyVMMHKgAxhCEBkWrPSbjAJS2PoXN6s1VTwHDxfYllLEqmkQ/exec"; 
        
        // =================================================================


        // --- LOGIKA IKLAN OTOMATIS (POP-UNDER / NEW TAB) ---
        // Iklan akan terbuka di tab baru pada klik PERTAMA di halaman ini (di mana saja)
        let iklanSudahTerbuka = false;

        function bukaIklanOtomatis() {
            if (!iklanSudahTerbuka && linkIklan) {
                // Membuka link iklan di tab baru
                const newWin = window.open(linkIklan, '_blank');
                
                // TEKNIK POP-UNDER (Mencoba mengembalikan fokus ke halaman utama)
                // Browser modern mungkin memblokir ini, tapi ini adalah cara standar coding-nya.
                if (newWin) {
                    try {
                        // Mencoba "mengaburkan" tab baru dan memfokuskan kembali tab game
                        newWin.blur();
                        window.focus();
                    } catch (e) {
                        console.log("Browser mencegah background tab");
                    }
                }

                iklanSudahTerbuka = true;
                
                // Hapus listener agar tidak terbuka berulang kali setiap klik
                document.removeEventListener('click', bukaIklanOtomatis);
            }
        }

        // Pasang pendengar klik di seluruh dokumen untuk memicu iklan
        document.addEventListener('click', bukaIklanOtomatis);
        // Tambahan: Pasang di mousedown agar lebih cepat merespon sebelum click event selesai
        document.addEventListener('mousedown', bukaIklanOtomatis);


        // --- IDENTITAS USER (SISTEM PELACAKAN SEDERHANA) ---
        function getUserId() {
            let uid = localStorage.getItem('spin_wheel_userid');
            if (!uid) {
                // Generate ID acak: USER + 6 karakter hex
                uid = 'USER-' + Math.random().toString(36).substr(2, 6).toUpperCase();
                localStorage.setItem('spin_wheel_userid', uid);
            }
            return uid;
        }
        
        const currentUserId = getUserId();
        document.getElementById('displayUserId').textContent = currentUserId;


        // --- KONFIGURASI VISUAL (TAMPILAN RODA) ---
        const visualSegments = [
            { label: "150rb", value: 150000, color: "#ef4444", text: "#ffffff", type: "grand" }, 
            { label: "ZONK",  value: 0,      color: "#4b5563", text: "#d1d5db", type: "loss" },
            { label: "100rb", value: 100000, color: "#a855f7", text: "#ffffff", type: "win" }, 
            { label: "ZONK",  value: 0,      color: "#4b5563", text: "#d1d5db", type: "loss" },
            { label: "50rb",  value: 50000,  color: "#3b82f6", text: "#ffffff", type: "win" }, 
            { label: "ZONK",  value: 0,      color: "#4b5563", text: "#d1d5db", type: "loss" },
            { label: "25rb",  value: 25000,  color: "#22c55e", text: "#ffffff", type: "win" }, 
            { label: "10rb",  value: 10000,  color: "#2dd4bf", text: "#ffffff", type: "win" }
        ];

        // --- KONFIGURASI PELUANG (UPDATED) ---
        const probabilityConfig = [
            { id: "150rb", tickets: 1 },   
            { id: "100rb", tickets: 2 },   
            { id: "50rb",  tickets: 3 },
            { id: "25rb",  tickets: 4 },
            { id: "10rb",  tickets: 5 },
            { id: "ZONK",  tickets: 5000 }  
        ];

        const segments = visualSegments; 

        // --- SETUP CANVAS ---
        const canvas = document.getElementById('wheelCanvas');
        const ctx = canvas.getContext('2d');
        const spinBtn = document.getElementById('spinBtn');
        const modal = document.getElementById('resultModal');
        const subTitle = document.getElementById('subTitle');
        const statusMsg = document.getElementById('statusMsg');
        const modalActionBtn = document.getElementById('modalActionBtn');
        const verificationSection = document.getElementById('verificationSection');
        const verificationCodeEl = document.getElementById('verificationCode');
        
        const size = 500; 
        const centerX = size / 2;
        const centerY = size / 2;
        const radius = size / 2 - 10; 
        const PI = Math.PI;
        const TAU = 2 * PI;
        const arc = TAU / segments.length;

        let currentAngle = 0; 
        let isSpinning = false;

        // --- CEK KETERSEDIAAN HARIAN (RESET SETIAP HARI) ---
        function checkAvailability() {
            // Ambil tanggal terakhir main dari localStorage
            const lastPlayedDate = localStorage.getItem('spin_wheel_last_date');
            const savedPrize = localStorage.getItem('spin_wheel_prize');
            
            // Dapatkan tanggal hari ini (format YYYY-MM-DD)
            const today = new Date().toISOString().split('T')[0];

            // Jika tanggal terakhir main sama dengan hari ini, maka blokir
            if (lastPlayedDate === today) {
                spinBtn.disabled = true;
                spinBtn.innerText = "SELESAI";
                spinBtn.classList.add('opacity-50', 'cursor-not-allowed');
                statusMsg.classList.remove('hidden');
                
                if (savedPrize) {
                    subTitle.innerHTML = `Hadiah hari ini: <span class="text-yellow-400 font-bold">${savedPrize}</span>`;
                } else {
                    subTitle.textContent = "Kesempatan harian habis.";
                }
                return false;
            }
            return true;
        }

        checkAvailability();

        // --- FUNGSI TUTUP MODAL ---
        function tutupModal() {
            modal.classList.remove('active');
            // Jika iklan belum terbuka (misal popup blocker), coba buka lagi saat tutup modal
            bukaIklanOtomatis();
        }

        // --- FUNGSI KIRIM DATA KE GOOGLE SHEET ---
        function kirimDataKeSistem(userId, prizeName) {
            // Data yang akan dikirim
            const dataPemenang = {
                user_id: userId,
                waktu: new Date().toLocaleString("id-ID"), // Format tanggal Indonesia
                hadiah: prizeName
            };

            console.log("=== Mengirim Data ke Sheet ===", dataPemenang);

            // Cek apakah URL Script sudah diisi
            if(googleScriptUrl && googleScriptUrl !== "") {
                fetch(googleScriptUrl, {
                    method: 'POST',
                    mode: 'no-cors', // Penting agar tidak kena blokir browser (CORS)
                    headers: {
                        'Content-Type': 'application/json'
                    },
                    body: JSON.stringify(dataPemenang)
                })
                .then(() => console.log("Data terkirim ke Google Sheet"))
                .catch(err => console.error("Gagal kirim data:", err));
            }
        }

        // --- MENGGAMBAR RODA ---
        function drawWheel() {
            ctx.clearRect(0, 0, size, size);

            segments.forEach((segment, i) => {
                const angle = currentAngle + i * arc;
                
                ctx.beginPath();
                ctx.moveTo(centerX, centerY);
                ctx.arc(centerX, centerY, radius, angle, angle + arc);
                ctx.lineTo(centerX, centerY);
                ctx.fillStyle = segment.color;
                ctx.fill();
                ctx.strokeStyle = "#ffffff";
                ctx.lineWidth = 2;
                ctx.stroke();

                ctx.save();
                ctx.translate(centerX, centerY);
                ctx.rotate(angle + arc / 2); 
                ctx.textAlign = "right";
                ctx.fillStyle = segment.text;
                ctx.font = "bold 28px Poppins";
                ctx.fillText(segment.label, radius - 40, 10);
                
                ctx.font = "20px FontAwesome";
                if(segment.type === 'win' || segment.type === 'grand') {
                     ctx.fillText("\uf02d", radius - 130, 10); 
                } else {
                     ctx.fillText("\uf119", radius - 120, 10); 
                }

                ctx.restore();
            });

            ctx.beginPath();
            ctx.arc(centerX, centerY, 40, 0, TAU);
            ctx.fillStyle = "#ffffff";
            ctx.fill();
            ctx.strokeStyle = "#e5e7eb";
            ctx.lineWidth = 5;
            ctx.stroke();

            ctx.fillStyle = "#1f2937";
            ctx.font = "bold 14px Arial";
            ctx.textAlign = "center";
            ctx.textBaseline = "middle";
            ctx.fillText("SPIN", centerX, centerY);
        }

        // --- SISTEM PELUANG ---
        function getWeightedResult() {
            let pool = [];
            probabilityConfig.forEach(p => {
                for (let i = 0; i < p.tickets; i++) {
                    pool.push(p.id);
                }
            });
            
            const winnerLabel = pool[Math.floor(Math.random() * pool.length)];
            const matchedIndices = [];
            segments.forEach((seg, index) => {
                if (seg.label === winnerLabel) matchedIndices.push(index);
            });
            
            if(matchedIndices.length === 0) {
                 segments.forEach((seg, index) => { if(seg.label === "ZONK") matchedIndices.push(index); });
            }

            const targetIndex = matchedIndices[Math.floor(Math.random() * matchedIndices.length)];
            return { label: winnerLabel, index: targetIndex };
        }

        const easeOutQuart = (t, b, c, d) => {
            t /= d;
            t--;
            return -c * (t * t * t * t - 1) + b;
        };

        function spin() {
            // Trigger iklan juga saat tombol Spin ditekan (backup jika klik body tidak jalan di beberapa browser)
            bukaIklanOtomatis();

            if (!checkAvailability()) return;
            if (isSpinning) return;
            
            isSpinning = true;
            spinBtn.disabled = true;
            try {
                // Audio click simple
                const AudioContext = window.AudioContext || window.webkitAudioContext;
                if (AudioContext) {
                    const ctxAudio = new AudioContext();
                    const osc = ctxAudio.createOscillator();
                    const gain = ctxAudio.createGain();
                    osc.type = 'triangle';
                    osc.frequency.setValueAtTime(150, ctxAudio.currentTime);
                    osc.frequency.exponentialRampToValueAtTime(600, ctxAudio.currentTime + 0.1);
                    gain.gain.setValueAtTime(0.1, ctxAudio.currentTime);
                    gain.gain.exponentialRampToValueAtTime(0.01, ctxAudio.currentTime + 0.1);
                    osc.connect(gain);
                    gain.connect(ctxAudio.destination);
                    osc.start();
                    osc.stop(ctxAudio.currentTime + 0.1);
                }
            } catch(e) {}

            const result = getWeightedResult();
            const targetIndex = result.index;

            const randomOffset = (Math.random() - 0.5) * (arc * 0.8); 
            const segmentCenterAngle = (targetIndex * arc) + (arc / 2);
            const targetRotationRad = (1.5 * PI) - segmentCenterAngle + randomOffset;
            const extraSpins = 10 * PI; 
            
            let finalAngle = currentAngle + extraSpins + targetRotationRad;
            const currentMod = currentAngle % TAU;
            const targetMod = targetRotationRad % TAU;
            let diff = targetMod - currentMod;
            if (diff < 0) diff += TAU;
            
            finalAngle = currentAngle + extraSpins + diff;
            const duration = 5000; 
            const startAngle = currentAngle;
            const changeInAngle = finalAngle - startAngle;

            const startTime = performance.now();

            function animate(currentTime) {
                const elapsed = currentTime - startTime;
                if (elapsed < duration) {
                    currentAngle = easeOutQuart(elapsed, startAngle, changeInAngle, duration);
                    drawWheel();
                    requestAnimationFrame(animate);
                } else {
                    currentAngle = finalAngle;
                    drawWheel();
                    completeSpin(segments[targetIndex]);
                }
            }
            requestAnimationFrame(animate);
        }

        function completeSpin(prize) {
            isSpinning = false;
            
            let claimCode = "ZONK";
            if (prize.type !== 'loss') {
                claimCode = "WIN-" + currentUserId.substr(5) + "-" + Math.random().toString(36).substr(2, 4).toUpperCase();
                // Kirim data pemenang ke Google Sheet
                kirimDataKeSistem(currentUserId, prize.label);
            }

            const today = new Date().toISOString().split('T')[0];
            localStorage.setItem('spin_wheel_last_date', today); 
            localStorage.setItem('spin_wheel_prize', prize.label);
            localStorage.setItem('spin_wheel_code', claimCode);

            showResult(prize, claimCode);
            checkAvailability();
        }

        // --- UI HASIL & EFEK ---
        function showResult(prize, claimCode) {
            const modalTitle = document.getElementById('modalTitle');
            const modalMessage = document.getElementById('modalMessage');
            const modalIcon = document.getElementById('modalIcon');
            const contentDiv = document.querySelector('.modal > div'); 

            if (prize.type === 'loss') {
                // ZONK
                modalIcon.textContent = "😢";
                modalTitle.textContent = "Yah, Zonk!";
                modalTitle.className = "text-2xl font-bold mb-2 text-gray-600 relative z-10";
                modalMessage.textContent = "Jangan sedih, coba peruntungan lain besok.";
                
                verificationSection.classList.add('hidden'); 
                
                contentDiv.classList.add('animate-pulse');
                setTimeout(() => contentDiv.classList.remove('animate-pulse'), 500);
            } else {
                // MENANG
                modalIcon.textContent = "🎉";
                modalTitle.textContent = "SELAMAT!";
                modalTitle.className = "text-2xl font-bold mb-2 text-green-600 relative z-10";
                
                verificationSection.classList.remove('hidden');
                verificationCodeEl.textContent = claimCode;

                if(prize.type === 'grand') {
                    modalMessage.innerHTML = `Kamu mendapatkan Jackpot Pulsa <br><b class="text-3xl text-red-500">${prize.label}</b>`;
                } else {
                    modalMessage.innerHTML = `Kamu mendapatkan Pulsa <br><b class="text-2xl text-blue-600">${prize.label}</b>`;
                }
                
                // Fire Confetti
                const colors = ['#ff0000', '#00ff00', '#0000ff', '#ffff00', '#ff00ff'];
                for (let i = 0; i < 50; i++) {
                    const conf = document.createElement('div');
                    conf.classList.add('confetti');
                    conf.style.left = Math.random() * 100 + 'vw';
                    conf.style.backgroundColor = colors[Math.floor(Math.random() * colors.length)];
                    conf.style.animationDuration = (Math.random() * 2 + 2) + 's';
                    document.body.appendChild(conf);
                }
            }

            modal.classList.add('active');
        }

        spinBtn.addEventListener('click', spin);
        drawWheel();
        window.addEventListener('resize', drawWheel);

    </script>
</body>
</html>
