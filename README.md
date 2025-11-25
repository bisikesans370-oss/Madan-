<!DOCTYPE html>
<html lang="or">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>ବିଜୁଳି ବିଲ୍ ଯାଞ୍ଚ କରନ୍ତୁ</title>
    <!-- Tailwind CSS CDN for styling -->
    <script src="https://cdn.tailwindcss.com"></script>
    <style>
        /* Custom styles for better aesthetics */
        body {
            font-family: 'Inter', sans-serif;
            background-color: #f0f4f8; /* Light blue background */
        }
        .container-card {
            background-color: #ffffff;
            border: 1px solid #e2e8f0;
            box-shadow: 0 10px 25px -5px rgba(0, 0, 0, 0.1);
        }
        .btn-calculate {
            transition: background-color 0.3s ease, transform 0.1s ease;
        }
        .btn-calculate:hover {
            background-color: #1e40af;
        }
        .btn-calculate:active {
            transform: scale(0.98);
        }
        .loading-spinner {
            border: 4px solid rgba(255, 255, 255, 0.3);
            border-top: 4px solid #ffffff;
            border-radius: 50%;
            width: 1.5rem;
            height: 1.5rem;
            animation: spin 1s linear infinite;
        }
        @keyframes spin {
            0% { transform: rotate(0deg); }
            100% { transform: rotate(360deg); }
        }
    </style>
</head>
<body>

    <!-- Main Container (Mobile-first, Responsive Design) -->
    <div class="min-h-screen flex items-center justify-center p-4">
        <div class="container-card w-full max-w-sm mx-auto p-6 md:p-8 rounded-xl">

            <!-- Title in Odia -->
            <h1 class="text-3xl font-extrabold text-center text-gray-900 mb-6 border-b pb-3">ବିଜୁଳି ବିଲ୍ ଯାଞ୍ଚ କରନ୍ତୁ</h1>
            <p class="text-sm text-center text-gray-600 mb-4">ଖର୍ଚ୍ଚ ହୋଇଥିବା ୟୁନିଟ୍ ଦାଖଲ କରନ୍ତୁ:</p>
            
            <!-- Electricity Meter Image Placeholder (with METER text restored) -->
            <div class="flex justify-center mb-6">
                <img 
                    src="https://placehold.co/150x100/374151/ffffff?text=METER" 
                    alt="ବିଜୁଳି ମିଟରର ପ୍ରତିଛବି (Electricity Meter Image)" 
                    class="rounded-lg shadow-md border-2 border-gray-200"
                    onerror="this.onerror=null; this.src='https://placehold.co/150x100/A0A0A0/FFFFFF?text=METER';"
                >
            </div>
            
            <!-- Unit Input Field -->
            <div class="mb-6">
                <label for="units" class="block text-sm font-medium text-gray-700 mb-2">ଖର୍ଚ୍ଚ ହୋଇଥିବା ୟୁନିଟ୍ (Units)</label>
                <input type="number" id="units" min="0" placeholder="ଉଦାହରଣ: ୭୫" class="w-full p-3 border border-gray-300 rounded-lg focus:ring-blue-500 focus:border-blue-500 text-lg shadow-inner">
            </div>

            <!-- Calculate Button -->
            <button id="calculateBtn" class="btn-calculate w-full bg-blue-700 text-white font-bold py-3 rounded-lg shadow-lg hover:shadow-xl focus:outline-none focus:ring-4 focus:ring-blue-300 flex items-center justify-center" disabled>
                <span id="buttonText">ବିଳ୍ ଚେକ୍ କରନ୍ତୁ</span>
                <div id="loadingSpinner" class="loading-spinner ml-2 hidden"></div>
            </button>

            <!-- Result Display Area -->
            <div id="resultOutput" class="mt-8 p-5 bg-blue-100 rounded-xl hidden">
                <p class="text-lg font-medium text-blue-800 mb-2">ଆପଣଙ୍କର ଆକଳିତ ବିଜୁଳି ବିଲ୍ ହେଉଛି:</p>
                <p id="totalBill" class="text-4xl font-extrabold text-blue-700">₹0.00</p>
                <p id="breakdown" class="text-sm text-blue-600 mt-3"></p>
            </div>

            <!-- Error Message Area -->
            <div id="errorMsg" class="mt-4 p-3 bg-red-100 border border-red-400 text-red-700 rounded-lg hidden" role="alert">
                ଦୟାକରି ୟୁନିଟ୍‌ର ବୈଧ ସଂଖ୍ୟା (Valid Number) ଦାଖଲ କରନ୍ତୁ।
            </div>

        </div>
    </div>

    <!-- JavaScript Logic (Core logic and TTS implementation) -->
    <script>
        // --- Utility Functions for TTS (PCM to WAV conversion) ---

        /**
         * Converts a Base64 string to an ArrayBuffer.
         * @param {string} base64 - The Base64 encoded string.
         * @returns {ArrayBuffer}
         */
        function base64ToArrayBuffer(base64) {
            const binary_string = window.atob(base64);
            const len = binary_string.length;
            const bytes = new Uint8Array(len);
            for (let i = 0; i < len; i++) {
                bytes[i] = binary_string.charCodeAt(i);
            }
            return bytes.buffer;
        }

        /**
         * Converts signed PCM 16-bit data into a WAV audio blob.
         * @param {Int16Array} pcm16 - PCM data.
         * @param {number} sampleRate - The sample rate (e.g., 24000).
         * @returns {Blob} The WAV audio blob.
         */
        function pcmToWav(pcm16, sampleRate) {
            const numChannels = 1;
            const bytesPerSample = 2; // 16-bit PCM
            const dataLength = pcm16.length * bytesPerSample;
            const buffer = new ArrayBuffer(44 + dataLength);
            const view = new DataView(buffer);

            let offset = 0;

            // RIFF chunk descriptor
            writeString(view, offset, 'RIFF'); offset += 4;
            view.setUint32(offset, 36 + dataLength, true); offset += 4;
            writeString(view, offset, 'WAVE'); offset += 4;

            // 'fmt ' chunk
            writeString(view, offset, 'fmt '); offset += 4;
            view.setUint32(offset, 16, true); offset += 4; // ChunkSize
            view.setUint16(offset, 1, true); offset += 2;  // AudioFormat (1 for PCM)
            view.setUint16(offset, numChannels, true); offset += 2;
            view.setUint32(offset, sampleRate, true); offset += 4;
            view.setUint32(offset, sampleRate * numChannels * bytesPerSample, true); offset += 4; // ByteRate
            view.setUint16(offset, numChannels * bytesPerSample, true); offset += 2; // BlockAlign
            view.setUint16(offset, 16, true); offset += 2; // BitsPerSample

            // 'data' chunk
            writeString(view, offset, 'data'); offset += 4;
            view.setUint32(offset, dataLength, true); offset += 4;

            // Write PCM data
            for (let i = 0; i < pcm16.length; i++) {
                view.setInt16(offset, pcm16[i], true);
                offset += 2;
            }

            return new Blob([view], { type: 'audio/wav' });
        }

        /** Helper function to write strings to DataView */
        function writeString(view, offset, string) {
            for (let i = 0; i < string.length; i++) {
                view.setUint8(offset + i, string.charCodeAt(i));
            }
        }

        // --- TTS API Call and Playback Function ---

        /**
         * Calls the Gemini TTS API and plays the resulting audio.
         * @param {string} text - The Odia text to speak.
         */
        async function speakBill(text) {
            const apiUrl = "https://generativelanguage.googleapis.com/v1beta/models/gemini-2.5-flash-preview-tts:generateContent";
            const apiKey = ""; // Canvas will provide this automatically

            // Odia translation of "Your bill is [amount] rupees."
            const odiaText = `ଆପଣଙ୍କର ବିଲ୍ ହେଉଛି ${text} ଟଙ୍କା।`;

            const payload = {
                contents: [{
                    parts: [{ text: odiaText }]
                }],
                generationConfig: {
                    responseModalities: ["AUDIO"],
                    speechConfig: {
                        voiceConfig: {
                            prebuiltVoiceConfig: { voiceName: "Kore" }
                        }
                    }
                }
            };

            const maxRetries = 5;
            for (let attempt = 0; attempt < maxRetries; attempt++) {
                try {
                    const response = await fetch(`${apiUrl}?key=${apiKey}`, {
                        method: 'POST',
                        headers: { 'Content-Type': 'application/json' },
                        body: JSON.stringify(payload)
                    });

                    if (!response.ok) {
                        const errorData = await response.json();
                        throw new Error(`API Error: ${response.status} - ${JSON.stringify(errorData)}`);
                    }

                    const result = await response.json();
                    const part = result?.candidates?.[0]?.content?.parts?.[0];
                    
                    if (part && part.inlineData && part.inlineData.data) {
                        const audioData = part.inlineData.data;
                        const mimeType = part.inlineData.mimeType; // Expecting audio/L16;rate=...

                        let sampleRate = 24000; // Default sample rate
                        const rateMatch = mimeType.match(/rate=(\d+)/);
                        if (rateMatch && rateMatch[1]) {
                            sampleRate = parseInt(rateMatch[1], 10);
                        }

                        const pcmData = base64ToArrayBuffer(audioData);
                        // API returns signed PCM16 audio data.
                        const pcm16 = new Int16Array(pcmData);
                        
                        const wavBlob = pcmToWav(pcm16, sampleRate);
                        const audioUrl = URL.createObjectURL(wavBlob);
                        
                        const audio = new Audio(audioUrl);
                        audio.play();

                        return; // Success, exit function
                    } else {
                        throw new Error("Invalid TTS response structure or missing audio data.");
                    }
                } catch (error) {
                    console.error(`TTS Attempt ${attempt + 1} failed:`, error);
                    if (attempt < maxRetries - 1) {
                        // Exponential backoff
                        const delay = Math.pow(2, attempt) * 1000;
                        await new Promise(resolve => setTimeout(resolve, delay));
                    } else {
                        // Log a message to the console about failure
                        console.error("Failed to play voice output after multiple retries. The issue might be with network or browser compatibility.");
                    }
                }
            }
        }

        // --- Calculator Logic ---

        const RATE_LOW = 2.70;  // Rate for first 50 units (₹2.70)
        const RATE_HIGH = 4.00; // Rate above 50 units (₹4.00)
        const CUTOFF_UNITS = 50; // Slab cutoff unit

        const calculateBtn = document.getElementById('calculateBtn');
        const unitsInput = document.getElementById('units');
        const loadingSpinner = document.getElementById('loadingSpinner');
        const buttonText = document.getElementById('buttonText');

        // Enable button when script is ready
        window.onload = function() {
            calculateBtn.disabled = false;
        };
        
        calculateBtn.addEventListener('click', calculateBill);

        /**
         * Calculates the electricity bill based on consumed units.
         */
        async function calculateBill() {
            calculateBtn.disabled = true;
            buttonText.textContent = "ଗଣନା କରୁଛି...";
            loadingSpinner.classList.remove('hidden');

            const units = parseFloat(unitsInput.value);

            const resultOutput = document.getElementById('resultOutput');
            const totalBillElement = document.getElementById('totalBill');
            const breakdownElement = document.getElementById('breakdown');
            const errorMsg = document.getElementById('errorMsg');

            // Hide old results and errors
            resultOutput.classList.add('hidden');
            errorMsg.classList.add('hidden');

            // Input Validation
            if (isNaN(units) || units < 0 || unitsInput.value.trim() === '') {
                errorMsg.classList.remove('hidden');
                resetButtonState();
                return;
            }

            let totalBill = 0;
            let breakdownText = '';

            // Electricity Bill Calculation (Slab-wise)
            if (units <= CUTOFF_UNITS) {
                // Case 1: 50 units or less
                totalBill = units * RATE_LOW;
                // Breakdown translated to Odia
                breakdownText = `ସମ୍ପୂର୍ଣ୍ଣ ${units} ୟୁନିଟ୍ ଉପରେ ₹${RATE_LOW.toFixed(2)} ଦରରେ।`;
            } else {
                // Case 2: More than 50 units
                
                // Charge for the first 50 units (at ₹2.70 rate)
                const chargeLow = CUTOFF_UNITS * RATE_LOW; 
                
                // Remaining units
                const remainingUnits = units - CUTOFF_UNITS;
                
                // Charge for remaining units (at ₹4.00 rate)
                const chargeHigh = remainingUnits * RATE_HIGH;
                
                // Total Bill
                totalBill = chargeLow + chargeHigh;

                // Breakdown translated to Odia
                breakdownText = `
                    ପ୍ରଥମ ${CUTOFF_UNITS} ୟୁନିଟ୍‌ର ବିଲ୍: ${CUTOFF_UNITS} x ₹${RATE_LOW.toFixed(2)} = ₹${chargeLow.toFixed(2)}। 
                    <br>
                    ବାକି ${remainingUnits.toFixed(0)} ୟୁନିଟ୍‌ର ବିଲ୍: ${remainingUnits.toFixed(0)} x ₹${RATE_HIGH.toFixed(2)} = ₹${chargeHigh.toFixed(2)}।
                `;
            }

            const finalBillAmount = totalBill.toFixed(2);

            // 1. Display Results
            totalBillElement.textContent = `₹${finalBillAmount}`;
            breakdownElement.innerHTML = breakdownText;
            resultOutput.classList.remove('hidden');
            
            // 2. Trigger Voice Output (TTS)
            await speakBill(finalBillAmount);
            
            // 3. Reset Button and Input
            unitsInput.value = '';
            resetButtonState();
        }

        function resetButtonState() {
            calculateBtn.disabled = false;
            buttonText.textContent = "ବିଳ୍ ଚେକ୍ କରନ୍ତୁ";
            loadingSpinner.classList.add('hidden');
        }
    </script>
</body>
</html>

