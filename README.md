# AI
<!doctype html>
<html lang="en">
<head>
  <meta charset="utf-8" />
  <meta name="viewport" content="width=device-width,initial-scale=1" />
  <title>JACK — Login</title>
  <style>
    :root{--blue:#000080;--bg:#f2f2f2}
    *{box-sizing:border-box}
    body{margin:0;font-family:Inter,Segoe UI,Arial,sans-serif;background:var(--bg)}
    .header{background:var(--blue);color:#fff;padding:14px 16px;text-align:center;font-weight:800;letter-spacing:2px;font-size:20px;position:sticky;top:0;z-index:20}
    .container{max-width:420px;margin:40px auto;padding:12px}
    .card{background:#fff;border-radius:12px;padding:22px;box-shadow:0 8px 30px rgba(2,6,23,0.08);text-align:center}
    .title{font-size:28px;margin:6px 0 10px}
    .subtitle{color:#555;margin-bottom:12px}
    input[type="text"],input[type="password"],input[type="number"]{
      width:100%;padding:12px;border-radius:8px;border:1px solid #d0d0d0;
      margin:8px 0;font-size:15px
    }
    .btn{display:inline-block;width:100%;padding:12px;border-radius:8px;background:var(--blue);
      color:#fff;border:0;font-weight:700;cursor:pointer;margin-top:12px;font-size:16px}
    .btn:hover{background:#0010cc}
    .hidden{display:none}
    .welcome-name{font-size:28px;margin:12px 0}
    .name-style{
      font-size:32px;font-weight:900;
      background:linear-gradient(90deg,#ff0000,#ff9900,#33cc33,#3399ff,#9933ff);
      -webkit-background-clip:text;-webkit-text-fill-color:transparent;
      text-shadow:2px 2px 4px rgba(0,0,0,0.3);letter-spacing:2px;
    }
    .qr{margin-top:18px}
    .qr img{width:200px;height:200px;border-radius:10px;border:6px solid var(--blue);background:#fff;padding:6px}
    .logout,.scanner-btn{display:inline-block;margin-top:18px;padding:10px 18px;background:#444;color:#fff;border-radius:8px;text-decoration:none}
    video{width:100%;margin-top:12px;border-radius:8px}
  </style>
</head>
<body>

  <!-- Top header -->
  <div class="header">🌟 <span class="name-style">WELCOME JACK</span> 🌟</div>

  <div class="container">
    <!-- LOGIN -->
    <div id="loginCard" class="card">
      <div class="title"><span class="name-style">JACK</span></div>
      <div class="subtitle">Enter your mobile number and PIN</div>
      <input id="mobileField" type="text" placeholder="Enter your mobile number" />
      <input id="pinField" type="password" placeholder="Enter your PIN" />
      <button class="btn" onclick="sendCode()">Send Code</button>

      <!-- Confirm Code Box -->
      <div id="codeBox" class="hidden">
        <input id="codeField" type="number" placeholder="Enter confirmation code" />
        <button class="btn" onclick="handleLogin()">Confirm & Login</button>
      </div>
    </div>

    <!-- WELCOME -->
    <div id="welcomeCard" class="card hidden">
      <div class="welcome-name" id="welcomeName">
        <span class="name-style">WELCOME JACK</span> 🔞
      </div>
      <div class="subtitle">You have successfully logged in.</div>

      <div class="qr">
        <h3>Your QR Code</h3>
        <img id="qrImage" src="" alt="QR Code">
      </div>

      <!-- Scanner Button -->
      <a href="#" class="scanner-btn" onclick="toggleScanner();return false;">Open Scanner</a>
      <div id="scannerBox" class="hidden">
        <video id="preview"></video>
      </div>

      <!-- YouTube Link -->
      <div style="margin-top:15px;">
        <a href="https://youtube.com/@aiai-g5b?si=kBMRFoeYFpFhhT8f" 
           target="_blank" 
           style="display:inline-block;padding:10px 18px;background:#cc0000;color:#fff;
                  border-radius:8px;text-decoration:none;font-weight:600;">
           ▶ Visit My YouTube
        </a>
      </div>

      <!-- Logout -->
      <a href="#" class="logout" onclick="doLogout();return false;">Log Out</a>
    </div>
  </div>

<!-- QR Scanner Library -->
<script src="https://unpkg.com/html5-qrcode" type="text/javascript"></script>

<script>
  let generatedCode=null;

  function sendCode(){
    const mobile=document.getElementById("mobileField").value.trim();
    const pin=document.getElementById("pinField").value.trim();

    if(mobile==="" || pin===""){
      alert("❌ Please enter mobile number and PIN first.");
      return;
    }

    // Random 6-digit code generate
    generatedCode=Math.floor(100000+Math.random()*900000);
    alert("✅ Your confirmation code is: "+generatedCode);

    // Show confirm code input box
    document.getElementById("codeBox").classList.remove("hidden");
  }

  function handleLogin(){
    const code=document.getElementById("codeField").value.trim();

    if(code==="" || code!=generatedCode){
      alert("❌ Invalid confirmation code.");
      return;
    }

    // Success → show welcome
    document.getElementById("loginCard").classList.add("hidden");
    document.getElementById("welcomeCard").classList.remove("hidden");

    const mobile=document.getElementById("mobileField").value.trim();
    const pin=document.getElementById("pinField").value.trim();

    const data=`${mobile} | PIN: ${pin} | Logged in at ${new Date().toLocaleString()}`;
    document.getElementById("qrImage").src=
      "https://api.qrserver.com/v1/create-qr-code/?size=300x300&data="+encodeURIComponent(data);

    // Reset
    document.getElementById("mobileField").value="";
    document.getElementById("pinField").value="";
    document.getElementById("codeField").value="";
    document.getElementById("codeBox").classList.add("hidden");
    generatedCode=null;
  }

  function doLogout(){
    document.getElementById("welcomeCard").classList.add("hidden");
    document.getElementById("loginCard").classList.remove("hidden");
    stopScanner();
  }

  // QR Scanner
  let scanner=null;
  function toggleScanner(){
    const box=document.getElementById("scannerBox");
    if(box.classList.contains("hidden")){
      box.classList.remove("hidden");
      startScanner();
    }else{
      box.classList.add("hidden");
      stopScanner();
    }
  }

  function startScanner(){
    scanner=new Html5Qrcode("preview");
    scanner.start({facingMode:"environment"},{fps:10,qrbox:250},
      qrCodeMessage=>{
        alert("Scanned: "+qrCodeMessage);
        stopScanner();
        document.getElementById("scannerBox").classList.add("hidden");
      });
  }

  function stopScanner(){
    if(scanner){scanner.stop().catch(()=>{});scanner=null;}
  }
</script>

</body>
</html>
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>PDF Upload Form</title>
    <style>
        body {
            font-family: Arial, sans-serif;
            margin: 20px;
        }
        .error-message {
            color: red;
            margin-top: 10px;
        }
    </style>
</head>
<body>

    <h1>Upload PDF Document</h1>

    <form id="pdfUploadForm" action="/upload_pdf" method="post" enctype="multipart/form-data">
        <div>
            <label for="pdfFile">Choose PDF file:</label>
            <input type="file" id="pdfFile" name="pdfFile" accept="application/pdf">
        </div>
        <div class="error-message" id="fileError"></div>
        <div>
            <button type="submit">Upload PDF</button>
        </div>
    </form>

    <script>
        document.getElementById('pdfUploadForm').addEventListener('submit', function(event) {
            const fileInput = document.getElementById('pdfFile');
            const fileError = document.getElementById('fileError');
            const file = fileInput.files[0];

            fileError.textContent = ''; // Clear previous errors

            if (!file) {
                fileError.textContent = 'Please select a file to upload.';
                event.preventDefault(); // Prevent form submission
                return;
            }

            // Check if the file is a PDF
            if (file.type !== 'application/pdf') {
                fileError.textContent = 'Only PDF documents are allowed.';
                event.preventDefault(); // Prevent form submission
                return;
            }

            // Note: Preventing image documents *within* a PDF is a server-side task.
            // Client-side JavaScript cannot inspect the content of a PDF file.
            // This client-side code only validates the file type as 'application/pdf'.
        });
    </script>

</body>
</html>
