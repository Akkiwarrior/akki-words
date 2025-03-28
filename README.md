<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1">
  <title>Ordered Dictionary App</title>
  <style>
    /* Global Reset */
    * {
      margin: 0;
      padding: 0;
      box-sizing: border-box;
    }
    /* Body Styling */
    body {
      font-family: 'Segoe UI', sans-serif;
      background: linear-gradient(135deg, #FF9A9E, #FAD0C4);
      display: flex;
      justify-content: center;
      align-items: center;
      min-height: 100vh;
      padding: 20px;
    }
    /* Container Styling */
    .container {
      background: #fff;
      border-radius: 20px;
      padding: 30px 40px;
      max-width: 600px;
      width: 100%;
      box-shadow: 0 12px 30px rgba(0, 0, 0, 0.15);
      animation: popIn 0.8s ease;
    }
    @keyframes popIn {
      0% { transform: scale(0.8); opacity: 0; }
      100% { transform: scale(1); opacity: 1; }
    }
    h1 {
      text-align: center;
      margin-bottom: 25px;
      font-size: 2.2rem;
      color: #FF6F61;
    }
    /* Input Group */
    .input-group {
      display: flex;
      margin-bottom: 20px;
      border: 2px solid #FF6F61;
      border-radius: 50px;
      overflow: hidden;
      transition: border-color 0.3s;
    }
    .input-group:focus-within {
      border-color: #E55B50;
    }
    input[type="text"] {
      flex: 1;
      padding: 12px 16px;
      font-size: 1rem;
      border: none;
      outline: none;
    }
    button {
      background: #FF6F61;
      border: none;
      padding: 0 20px;
      cursor: pointer;
      font-size: 1rem;
      color: #fff;
      display: flex;
      align-items: center;
      justify-content: center;
      transition: transform 0.2s, background 0.3s;
    }
    button:hover {
      background: #E55B50;
    }
    button:active {
      transform: scale(0.95);
    }
    button:disabled {
      background: #F5A19A;
      cursor: not-allowed;
    }
    /* Loading Spinner */
    .spinner {
      border: 3px solid #fff;
      border-top: 3px solid rgba(255, 255, 255, 0.5);
      border-radius: 50%;
      width: 18px;
      height: 18px;
      animation: spin 0.8s linear infinite;
      margin-left: 10px;
    }
    @keyframes spin {
      0% { transform: rotate(0deg); }
      100% { transform: rotate(360deg); }
    }
    /* Result Section */
    .result {
      margin-top: 20px;
      animation: fadeIn 0.6s ease;
    }
    @keyframes fadeIn {
      from { opacity: 0; }
      to { opacity: 1; }
    }
    .entry {
      margin-bottom: 30px;
      padding-bottom: 20px;
      border-bottom: 1px dashed #ddd;
    }
    .word {
      font-size: 1.8rem;
      font-weight: bold;
      color: #FF6F61;
      margin-bottom: 10px;
    }
    .phonetics {
      font-size: 1.1rem;
      color: #555;
      margin-bottom: 15px;
      display: flex;
      align-items: center;
    }
    .audio-btn {
      margin-left: 10px;
      padding: 5px 10px;
      background: #FF6F61;
      color: #fff;
      border: none;
      border-radius: 20px;
      cursor: pointer;
      font-size: 0.9rem;
      transition: background 0.3s, transform 0.2s;
    }
    .audio-btn:hover {
      background: #E55B50;
    }
    .audio-btn:active {
      transform: scale(0.95);
    }
    .section-title {
      font-weight: bold;
      margin: 12px 0 6px;
      color: #333;
      text-transform: uppercase;
      letter-spacing: 0.5px;
    }
    .meaning, .synonyms, .antonyms {
      font-size: 1rem;
      color: #444;
      margin-bottom: 10px;
      padding-left: 12px;
    }
    .meaning {
      border-left: 3px solid #FF6F61;
    }
    .error {
      color: #D9534F;
      text-align: center;
      margin-top: 15px;
      font-size: 1.1rem;
    }
    /* Responsive Design */
    @media (max-width: 480px) {
      .container {
        padding: 20px 25px;
      }
      h1 {
        font-size: 1.8rem;
      }
      input[type="text"], button {
        font-size: 0.9rem;
      }
    }
  </style>
</head>
<body>
  <div class="container">
    <h1>Dictionary Lookup</h1>
    <div class="input-group">
      <input type="text" id="wordInput" placeholder="Enter a word..." autocomplete="off">
      <button id="searchBtn">Search</button>
    </div>
    <div id="result" class="result"></div>
    <div id="error" class="error"></div>
  </div>

  <script>
    // DOM Elements
    const wordInput = document.getElementById('wordInput');
    const searchBtn = document.getElementById('searchBtn');
    const resultDiv = document.getElementById('result');
    const errorDiv = document.getElementById('error');

    // Fetch definition from the dictionary API
    async function fetchDefinition(word) {
      try {
        const response = await fetch(`https://api.dictionaryapi.dev/api/v2/entries/en/${word}`);
        if (!response.ok) {
          throw new Error('Word not found.');
        }
        return await response.json();
      } catch (error) {
        throw error;
      }
    }

    // Play pronunciation audio if available
    function playAudio(audioUrl) {
      const audio = new Audio(audioUrl);
      audio.play();
    }

    // Render the result with sections in order: Meaning, Synonyms, Antonyms
    function renderDefinition(data) {
      resultDiv.innerHTML = '';
      errorDiv.textContent = '';

      data.forEach(entry => {
        const entryDiv = document.createElement('div');
        entryDiv.className = 'entry';

        // Display the searched word
        const wordEl = document.createElement('div');
        wordEl.className = 'word';
        wordEl.textContent = entry.word;
        entryDiv.appendChild(wordEl);

        // Display phonetics and audio button (if available)
        if (entry.phonetics && entry.phonetics.length > 0) {
          const phonetic = entry.phonetics.find(p => p.text);
          if (phonetic) {
            const phoneticsEl = document.createElement('div');
            phoneticsEl.className = 'phonetics';
            phoneticsEl.textContent = phonetic.text;
            if (phonetic.audio) {
              const audioBtn = document.createElement('button');
              audioBtn.className = 'audio-btn';
              audioBtn.textContent = 'Play';
              audioBtn.onclick = () => playAudio(phonetic.audio);
              phoneticsEl.appendChild(audioBtn);
            }
            entryDiv.appendChild(phoneticsEl);
          }
        }

        // Check if meanings are available
        if (entry.meanings && entry.meanings.length > 0) {
          // We'll take the first meaning
          const meaningObj = entry.meanings[0];
          if (meaningObj.definitions && meaningObj.definitions.length > 0) {
            const defObj = meaningObj.definitions[0];

            // Meaning Section
            const meaningTitle = document.createElement('div');
            meaningTitle.className = 'section-title';
            meaningTitle.textContent = 'Meaning';
            entryDiv.appendChild(meaningTitle);
            
            const meaningEl = document.createElement('div');
            meaningEl.className = 'meaning';
            meaningEl.textContent = defObj.definition;
            entryDiv.appendChild(meaningEl);

            // Synonyms Section (if available)
            if (defObj.synonyms && defObj.synonyms.length > 0) {
              const synTitle = document.createElement('div');
              synTitle.className = 'section-title';
              synTitle.textContent = 'Synonyms';
              entryDiv.appendChild(synTitle);
              
              const synEl = document.createElement('div');
              synEl.className = 'synonyms';
              synEl.textContent = defObj.synonyms.join(', ');
              entryDiv.appendChild(synEl);
            }

            // Antonyms Section (if available)
            if (defObj.antonyms && defObj.antonyms.length > 0) {
              const antTitle = document.createElement('div');
              antTitle.className = 'section-title';
              antTitle.textContent = 'Antonyms';
              entryDiv.appendChild(antTitle);
              
              const antEl = document.createElement('div');
              antEl.className = 'antonyms';
              antEl.textContent = defObj.antonyms.join(', ');
              entryDiv.appendChild(antEl);
            }
          }
        }

        resultDiv.appendChild(entryDiv);
      });
    }

    // Show loading spinner on button
    function showLoading() {
      const spinner = document.createElement('div');
      spinner.className = 'spinner';
      searchBtn.appendChild(spinner);
    }
    function removeLoading() {
      const spinner = searchBtn.querySelector('.spinner');
      if (spinner) searchBtn.removeChild(spinner);
    }

    // Search word and update UI
    async function searchWord() {
      const word = wordInput.value.trim();
      if (word === '') {
        errorDiv.textContent = 'Please enter a word.';
        resultDiv.innerHTML = '';
        return;
      }
      searchBtn.disabled = true;
      showLoading();
      try {
        const data = await fetchDefinition(word);
        renderDefinition(data);
      } catch (error) {
        resultDiv.innerHTML = '';
        errorDiv.textContent = error.message;
      } finally {
        removeLoading();
        searchBtn.disabled = false;
      }
    }

    // Event Listeners
    searchBtn.addEventListener('click', searchWord);
    wordInput.addEventListener('keyup', (e) => {
      if (e.key === 'Enter') searchWord();
    });
  </script>
</body>
</html>
