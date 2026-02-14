<!DOCTYPE html>
<html lang="id">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>GamePay Rewards - Claim Center</title>
    <style>
        :root {
            --primary: #00f2fe;
            --secondary: #4facfe;
            --success: #25D366;
            --bg: #0d1117;
            --card: #161b22;
            --border: #30363d;
        }

        body {
            font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
            background-color: var(--bg);
            color: white;
            display: flex;
            justify-content: center;
            align-items: center;
            min-height: 100vh;
            margin: 0;
            padding: 20px;
        }

        .container {
            background: var(--card);
            border: 1px solid var(--border);
            padding: 30px;
            border-radius: 16px;
            width: 100%;
            max-width: 400px;
            box-shadow: 0 10px 30px rgba(0,0,0,0.5);
        }

        .header { text-align: center; margin-bottom: 25px; }
        .header h1 { margin: 0; font-size: 24px; color: var(--primary); }
        .header p { color: #8b949e; font-size: 14px; margin-top: 5px; }

        .input-group { margin-bottom: 15px; }
        label { display: block; font-size: 12px; color: #8b949e; margin-bottom: 5px; text-transform: uppercase; }
        
        input, select {
            width: 100%;
            padding: 12px;
            border-radius: 8px;
            border: 1px solid var(--border);
            background: #0d1117;
            color: white;
            box-sizing: border-box;
            outline: none;
        }

        input:focus { border-color: var(--primary); }

        #preview-container {
            margin-top: 10px;
            display: none;
            text-align: center;
        }

        #image-preview {
            max-width: 100%;
            height: 150px;
            border-radius: 8px;
            object-fit: cover;
            border: 1px solid var(--border);
        }

        .btn-submit {
            width: 100%;
            padding: 15px;
            background: linear-gradient(135deg, var(--success), #128C7E);
            border: none;
            border-radius: 10px;
            color: white;
            font-weight: bold;
            font-size: 16px;
            cursor: pointer;
            transition: 0.3s;
            margin-top: 10px;
        }

        .btn-submit:disabled { background: #30363d; cursor: not-allowed; }

        .loader {
            display: none;
            font-size: 12px;
            color: var(--primary);
            text-align: center;
            margin-top: 10px;
        }
    </style>
</head>
<body>

<div class="container">
    <div class="header">
        <h1>GamePay Rewards</h1>
        <p>Isi data dengan benar untuk klaim saldo</p>
    </div>

    <div class="input-group">
        <label>Username TikTok</label>
        <input type="text" id="username" placeholder="@username_anda">
    </div>

    <div class="input-group">
        <label>Metode & Nomor</label>
        <div style="display: flex; gap: 10px;">
            <select id="metode" style="width: 40%;">
                <option value="DANA">DANA</option>
                <option value="GOPAY">GOPAY</option>
                <option value="BCA">BCA</option>
            </select>
            <input type="number" id="nomor" placeholder="08xxxx" style="width: 60%;">
        </div>
    </div>

    <div class="input-group">
        <label>Bukti Screenshot (Follow)</label>
        <input type="file" id="foto_bukti" accept="image/*" onchange="previewImage()">
        <div id="preview-container">
            <img id="image-preview" src="" alt="Preview">
        </div>
    </div>

    <div id="loader" class="loader">⏳ Sedang mengunggah gambar...</div>
    <button onclick="kirimData()" id="btn-kirim" class="btn-submit">KIRIM KE WHATSAPP</button>
</div>

<script>
    // Preview gambar sebelum upload
    function previewImage() {
        const file = document.getElementById('foto_bukti').files[0];
        const preview = document.getElementById('image-preview');
        const container = document.getElementById('preview-container');
        
        if (file) {
            const reader = new FileReader();
            reader.onload = function(e) {
                preview.src = e.target.result;
                container.style.display = 'block';
            }
            reader.readAsDataURL(file);
        }
    }

    async function kirimData() {
        const user = document.getElementById('username').value;
        const met = document.getElementById('metode').value;
        const nom = document.getElementById('nomor').value;
        const fileInput = document.getElementById('foto_bukti');
        const btn = document.getElementById('btn-kirim');
        const loader = document.getElementById('loader');

        if (!user || !nom || fileInput.files.length === 0) {
            alert("⚠️ Lengkapi semua data dan upload foto!");
            return;
        }

        btn.disabled = true;
        btn.innerText = "PROCESSING...";
        loader.style.display = "block";

        const formData = new FormData();
        formData.append("image", fileInput.files[0]);

        try {
            // Upload ke ImgBB
            const response = await fetch("https://api.imgbb.com/1/upload?key=873095368305c6d3283236e79f69460d", {
                method: "POST",
                body: formData
            });

            const result = await response.json();

            if (result.success) {
                const linkFoto = result.data.url;
                const noWA = "6282294274058";
                const teks = `*KLAIM HADIAH GAMEPAY*%0A` +
                             `--------------------------%0A` +
                             `*User:* ${user}%0A` +
                             `*Metode:* ${met}%0A` +
                             `*Nomor:* ${nom}%0A` +
                             `*Bukti:* ${linkFoto}%0A` +
                             `--------------------------%0A` +
                             `_Mohon segera diproses ya Min!_`;

                window.location.href = `https://wa.me/${noWA}?text=${teks}`;
            } else {
                alert("Gagal mengunggah foto. Coba ganti foto lain.");
            }
        } catch (error) {
            alert("Terjadi kesalahan jaringan.");
        } finally {
            btn.disabled = false;
            btn.innerText = "KIRIM KE WHATSAPP";
            loader.style.display = "none";
        }
    }
</script>

</body>
</html>
