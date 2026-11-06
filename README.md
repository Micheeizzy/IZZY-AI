  # IZZY-AI
Un IA 

<!DOCTYPE html>
<html lang="fr">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1">
  <meta name="theme-color" content="#101426">
  <title>IZZY AI</title>
  <link rel="manifest" href="manifest.json">

  <style>
    body {
      margin: 0;
      font-family: Arial, sans-serif;
      background: #0b1020;
      color: white;
    }

    header {
      background: #171f38;
      padding: 20px;
      text-align: center;
    }

    h1 { color: #9988ff; }

    #messages {
      padding: 15px;
      height: 60vh;
      overflow-y: auto;
    }

    .message {
      padding: 12px;
      margin: 10px 0;
      border-radius: 12px;
      max-width: 85%;
      white-space: pre-wrap;
    }

    .bot { background: #202b49; }
    .user {
      background: #6553c7;
      margin-left: auto;
    }

    .controls {
      padding: 15px;
      display: flex;
      gap: 8px;
    }

    input {
      flex: 1;
      min-width: 0;
      padding: 13px;
      border-radius: 10px;
      border: none;
    }

    button {
      background: #8876ff;
      color: white;
      border: none;
      padding: 12px;
      border-radius: 10px;
    }

    video {
      width: 100%;
      max-height: 250px;
    }
  </style>
</head>

<body>
  <header>
    <h1>IZZY AI</h1>
    <p>Ton assistant personnel intelligent</p>
  </header>

  <div id="messages">
    <div class="message bot">
      Bonjour ! Je suis IZZY AI.
      Comment puis-je t'aider aujourd'hui ?
    </div>
  </div>

  <div class="controls">
    <input id="question" placeholder="Écris ton message...">
    <button onclick="envoyer()">Envoyer</button>
  </div>

  <div class="controls">
    <button onclick="parler()">🎙️ Parler</button>
    <button onclick="ouvrirCamera()">📷 Caméra</button>
  </div>

  <video id="camera" autoplay playsinline hidden></video>

  <script>
    const messages = document.getElementById("messages");

    function ajouterMessage(texte, classe) {
      const div = document.createElement("div");
      div.className = "message " + classe;
      div.textContent = texte;
      messages.appendChild(div);
      messages.scrollTop = messages.scrollHeight;
    }

    function envoyer() {
      const champ = document.getElementById("question");
      const texte = champ.value.trim();

      if (!texte) return;

      ajouterMessage(texte, "user");
      champ.value = "";

      setTimeout(() => {
        ajouterMessage(
          "J'ai reçu ton message. Mon moteur d'intelligence artificielle doit encore être connecté pour te répondre réellement.",
          "bot"
        );
      }, 500);
    }

    function parler() {
      const Reconnaissance =
        window.SpeechRecognition ||
        window.webkitSpeechRecognition;

      if (!Reconnaissance) {
        ajouterMessage("La reconnaissance vocale n'est pas disponible.", "bot");
        return;
      }

      const reconnaissance = new Reconnaissance();
      reconnaissance.lang = "fr-FR";

      reconnaissance.onresult = function(event) {
        document.getElementById("question").value =
          event.results[0][0].transcript;
      };

      reconnaissance.start();
    }

    let fluxCamera;

    async function ouvrirCamera() {
      try {
        fluxCamera = await navigator.mediaDevices.getUserMedia({
          video: true
        });

        const video = document.getElementById("camera");
        video.srcObject = fluxCamera;
        video.hidden = false;
      } catch (erreur) {
        ajouterMessage("Autorisation de caméra refusée ou indisponible.", "bot");
      }
    }

    if ("serviceWorker" in navigator) {
      navigator.serviceWorker.register("./sw.js");
    }
  </script>
</body>
</html>


const CACHE = "izzy-ai-v1";

const FILES = [
  "./",
  "./index.html",
  "./manifest.json",
  "./icons/icon-192.png",
  "./icons/icon-512.png"
];

self.addEventListener("install", event => {
  event.waitUntil(
    caches.open(CACHE).then(cache => cache.addAll(FILES))
  );
});

self.addEventListener("fetch", event => {
  if (event.request.method !== "GET") return;

  event.respondWith(
    caches.match(event.request).then(
      response => response || fetch(event.request)
    )
  );
});

