<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Maya Voice Assistant</title>
  <script src="https://tailwindcss.com"></script>
</head>
<body class="bg-slate-900 text-white min-h-screen flex flex-col justify-between items-center p-6 font-sans">

  <!-- Header -->
  <header class="text-center mt-8">
    <h1 class="text-3xl font-bold tracking-tight text-teal-400">MAYA</h1>
    <p class="text-slate-400 text-sm mt-1">Your Mobile AI Voice Assistant</p>
  </header>

  <!-- Output Display -->
  <main class="w-full max-w-md bg-slate-800 rounded-2xl p-6 shadow-xl flex-grow my-6 flex flex-col justify-end space-y-4 overflow-y-auto">
    <div id="user-bubble" class="hidden self-end bg-teal-600 text-white px-4 py-2 rounded-2xl max-w-[80%] text-sm shadow"></div>
    <div id="maya-bubble" class="self-start bg-slate-700 text-slate-200 px-4 py-2 rounded-2xl max-w-[80%] text-sm shadow">
      Tap the microphone and say something to get started.
    </div>
  </main>

  <!-- Control Panel -->
  <footer class="w-full max-w-md flex flex-col items-center mb-8">
    <button id="mic-btn" class="w-20 h-20 bg-teal-500 hover:bg-teal-400 active:scale-95 transition rounded-full flex items-center justify-center shadow-lg shadow-teal-500/30">
      <!-- Microphone Icon -->
      <svg id="mic-icon" class="w-8 h-8 text-white" fill="none" stroke="currentColor" viewBox="0 0 24 24">
        <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M19 11a7 7 0 01-7 7m0 0a7 7 0 01-7-7m7 7v4m0 0H8m4 0h4m-4-8a3 3 0 01-3-3V5a3 3 0 116 0v6a3 3 0 01-3 3z"></path>
      </svg>
    </button>
    <p id="status" class="text-xs text-slate-500 mt-3 font-medium uppercase tracking-wider">Idle</p>
  </footer>

  <script>
    // Configuration
    const GEMINI_API_KEY = "YOUR_GEMINI_API_KEY"; // Replace with your proxy server endpoint or key for development
    
    // UI Elements
    const micBtn = document.getElementById('mic-btn');
    const micIcon = document.getElementById('mic-icon');
    const statusText = document.getElementById('status');
    const userBubble = document.getElementById('user-bubble');
    const mayaBubble = document.getElementById('maya-bubble');

    // Speech Recognition Setup (Web Speech API)
    const SpeechRecognition = window.SpeechRecognition || window.webkitSpeechRecognition;
    if (!SpeechRecognition) {
      alert("Your browser does not support Speech Recognition. Please use Safari on iOS or Chrome on Android.");
    }

    const recognition = new SpeechRecognition();
    recognition.continuous = false;
    recognition.lang = 'en-US';
    recognition.interimResults = false;

    let isListening = false;

    // Toggle Listening Mode
    micBtn.addEventListener('click', () => {
      if (!isListening) {
        recognition.start();
      } else {
        recognition.stop();
      }
    });

    recognition.onstart = () => {
      isListening = true;
      statusText.innerText = "Listening...";
      micBtn.classList.replace('bg-teal-500', 'bg-red-500');
    };

    recognition.onend = () => {
      isListening = false;
      statusText.innerText = "Processing...";
      micBtn.classList.replace('bg-red-500', 'bg-teal-500');
    };

    recognition.onresult = async (event) => {
      const transcript = event.results[0][0].transcript;
      
      // Update User UI
      userBubble.innerText = transcript;
      userBubble.classList.remove('hidden');
      
      // Send to AI
      await generateAIResponse(transcript);
    };

    // Connect to Google Gemini API
    async function generateAIResponse(userInput) {
      statusText.innerText = "Maya is thinking...";
      try {
        const response = await fetch(`https://googleapis.com{GEMINI_API_KEY}`, {
          method: 'POST',
          headers: { 'Content-Type': 'application/json' },
          body: JSON.stringify({
            contents: [{ parts: [{ text: `You are Maya, a witty and fast mobile voice assistant. Give a short, concise response (max 2 sentences) to: "${userInput}"` }] }]
          })
        });

        const data = await response.json();
        const aiText = data.candidates[0].content.parts[0].text;
        
        // Update Maya UI
        mayaBubble.innerText = aiText;
        statusText.innerText = "Idle";

        // Trigger Voice Output
        speak(aiText);
      } catch (error) {
        console.error(error);
        mayaBubble.innerText = "Sorry, I couldn't reach my servers. Please check your API key setup.";
        statusText.innerText = "Error";
      }
    }

    // Text to Speech Output
    function speak(text) {
      const utterance = new SpeechSynthesisUtterance(text);
      utterance.lang = 'en-US';
      // Pick a female/natural voice variant if available in browser
      const voices = window.speechSynthesis.getVoices();
      const preferredVoice = voices.find(v => v.name.includes('Google') || v.name.includes('Samantha'));
      if (preferredVoice) utterance.voice = preferredVoice;
      
      window.speechSynthesis.speak(utterance);
    }
  </script>
</body>
</html>
