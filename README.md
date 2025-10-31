<!DOCTYPE html>
<html lang="hi">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>JACK Chrome</title>
<style>
  body {
    margin: 0;
    background: #0f1724;
    font-family: "Segoe UI", sans-serif;
    color: white;
    display: flex;
    flex-direction: column;
    height: 100vh;
    overflow: hidden;
  }
  header {
    background: #1e293b;
    color: white;
    text-align: center;
    padding: 10px 0;
    font-size: 20px;
    font-weight: bold;
    letter-spacing: 1px;
    box-shadow: 0 2px 4px rgba(0,0,0,0.5);
    position: sticky;
    top: 0;
    z-index: 10;
  }
  #searchBar {
    display: flex;
    justify-content: center;
    align-items: center;
    padding: 10px;
    background: #1e293b;
    gap: 8px;
  }
  input {
    width: 80%;
    padding: 10px;
    border-radius: 20px;
    border: none;
    outline: none;
    font-size: 16px;
  }
  button {
    padding: 10px 16px;
    border-radius: 20px;
    border: none;
    background: #2563eb;
    color: white;
    cursor: pointer;
    font-weight: bold;
  }
  button:hover {
    background: #1d4ed8;
  }
  #answerBox {
    background: #ffffff11;
    border-radius: 10px;
    margin: 10px auto;
    width: 90%;
    padding: 10px;
    display: none;
    text-align: left;
    color: #fff;
    font-size: 15px;
    box-shadow: 0 0 10px #0004;
  }
  iframe {
    flex: 1;
    width: 100%;
    border: none;
    background: white;
  }
</style>
</head>
<body>
  <header>🌐 JACK Chrome</header>

  <div id="searchBar">
    <input type="text" id="urlInput" placeholder="Type a question or URL..." />
    <button onclick="browse()">Go</button>
  </div>

  <div id="answerBox"></div>

  <iframe id="browserFrame" src="https://www.google.com/webhp?igu=1"></iframe>

<script>
const answerBox = document.getElementById('answerBox');
const browserFrame = document.getElementById('browserFrame');

function isQuestion(text) {
  return /(who|what|when|where|why|how|kaun|kya|kab|kahan|कौन|क्या|कब|कहाँ)/i.test(text);
}

function normalizeUrl(input) {
  input = input.trim();
  if (!input) return '';
  if (/^[a-zA-Z]+:\/\//.test(input)) return input;
  if (/^[^\s]+\.[^\s]{2,}/.test(input)) {
    return input.startsWith('http') ? input : 'https://' + input;
  }
  return '';
}

async function fetchAnswer(q) {
  answerBox.style.display = "block";
  answerBox.innerHTML = "🔍 Searching answer...";

  try {
    // Try Wikipedia summary
    let res = await fetch(`https://en.wikipedia.org/api/rest_v1/page/summary/${encodeURIComponent(q)}`);
    if (res.ok) {
      let data = await res.json();
      if (data.extract) {
        answerBox.innerHTML = `<b>${data.title}</b><br>${data.extract}`;
        return;
      }
    }

    // Try DuckDuckGo instant answer
    res = await fetch(`https://api.duckduckgo.com/?q=${encodeURIComponent(q)}&format=json`);
    let data = await res.json();
    if (data.AbstractText) {
      answerBox.innerHTML = `<b>${data.Heading}</b><br>${data.AbstractText}`;
      return;
    }

    // No instant answer found
    answerBox.innerHTML = "❌ No direct answer found. Showing Google search...";
  } catch (e) {
    answerBox.innerHTML = "⚠️ Unable to fetch answer. Showing Google search...";
  }
}

async function browse() {
  const input = document.getElementById('urlInput').value.trim();
  if (!input) return;

  const url = normalizeUrl(input);
  const isQ = isQuestion(input);

  if (isQ) await fetchAnswer(input);

  if (url) {
    browserFrame.src = url;
  } else {
    const q = encodeURIComponent(input);
    browserFrame.src = `https://www.google.com/search?q=${q}`;
  }
}

document.getElementById('urlInput').addEventListener('keypress', e => {
  if (e.key === 'Enter') browse();
});
</script>
</body>
</html>
