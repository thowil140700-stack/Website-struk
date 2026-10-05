# Website-struk
Struk otomatis
<!DOCTYPE html>
<html lang="id">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Struk Agen Mini ATM Gemini AI</title>
<style>
body {
    font-family: Arial, sans-serif;
    background-color: #f0f2f5;
    margin: 0;
    padding: 20px;
    display: flex;
    gap: 20px;
    justify-content: center;
    flex-wrap: wrap;
}
.container {
    background: white;
    padding: 20px;
    border-radius: 8px;
    box-shadow: 0 2px 8px rgba(0,0,0,0.1);
    width: 420px;
}
textarea, input, button, select {
    width: 100%;
    margin-bottom: 12px;
    padding: 10px;
    box-sizing: border-box;
    border: 1px solid #ccc;
    border-radius: 4px;
}
button {
    background-color: #1a73e8;
    color: white;
    border: none;
    cursor: pointer;
    font-weight: bold;
}
button:hover { background-color: #1557b0; }
.btn-print { background-color: #28a745; }
.btn-print:hover { background-color: #1e7e34; }

/* Struk Thermal Style (58mm / 80mm) */
.receipt {
    background: #fff;
    width: 300px;
    padding: 15px;
    border: 1px solid #ddd;
    font-family: 'Courier New', Courier, monospace;
    font-size: 11px;
    line-height: 1.3;
}
.receipt-header { text-align: center; margin-bottom: 10px; }
.receipt-header h2 { margin: 0; font-size: 15px; text-transform: uppercase; }
.receipt-header p { margin: 2px 0; font-size: 10px; }
.line { border-bottom: 1px dashed #000; margin: 8px 0; }
.data-table { width: 100%; border-collapse: collapse; }
.data-table td { padding: 2px 0; vertical-align: top; }
.text-right { text-align: right; }
.bold { font-weight: bold; }
.status { text-align: center; font-size: 13px; font-weight: bold; margin: 6px 0; }
.receipt-footer { text-align: center; font-size: 10px; margin-top: 5px; }

/* Layout cetak khusus printer thermal */
@media print {
    body * { visibility: hidden; }
    #receiptArea, #receiptArea * { visibility: visible; }
    #receiptArea {
        position: absolute;
        left: 0;
        top: 0;
        width: 100%;
        border: none;
        padding: 0;
    }
}
</style>
</head>
<body>

<!-- Panel Input / Menu Konfigurasi -->
<div class="container">
    <h2>Pencetak Struk Gemini AI</h2>
    
    <label>API Key Google Gemini:</label>
    <input type="password" id="apiKey" placeholder="AIzaSy..." />
    <small style="color: #666; display: block; margin-top: -8px; margin-bottom: 10px; font-size: 11px;">
        Ambil gratis di <a href="https://aistudio.google.com/" target="_blank">Google AI Studio</a>
    </small>
    
    <label>Nama Toko / Loket:</label>
    <input type="text" id="agentName" value="TOKO ALADIN CELL" placeholder="Contoh: TOKO ALADIN CELL" />
    
    <label>Alamat Toko / Loket:</label>
    <input type="text" id="storeAddress" value="Jl. Raya No. 123, Kota Anda" placeholder="Contoh: Jl. Merdeka No. 45" />
    
    <label>Ucapan Terima Kasih / Catatan Kaki:</label>
    <input type="text" id="footerMessage" value="TERIMA KASIH ATAS KUNJUNGAN ANDA" placeholder="Contoh: TERIMA KASIH" />
    
    <label>Upload Gambar Bukti / Screenshot Transaksi:</label>
    <input type="file" id="imageInput" accept="image/*" />
    
    <label>Atau Teks Bukti / SMS / Pesan (Opsional):</label>
    <textarea id="inputText" rows="3" placeholder="Contoh: Sukses transfer ke BCA 1234567890 a.n Budi Santoso sebesar Rp 500.000..."></textarea>
    
    <button onclick="generateReceipt()">Olah Struk dengan Gemini AI</button>
    <button class="btn-print" onclick="window.print()">Cetak Struk Thermal</button>
</div>

<!-- Area Tampilan Struk -->
<div class="receipt" id="receiptArea">
    <div class="receipt-header">
        <h2 id="dispAgentName">AGEN MINI ATM</h2>
        <p id="dispStoreAddress">Alamat Toko</p>
        <p id="dispDate">TANGGAL: -</p>
    </div>
    <div class="line"></div>
    <div class="status" id="dispStatus">-- STRUK TRANSAKSI --</div>
    <div class="line"></div>
    <table class="data-table" id="receiptDetails">
        <!-- Rincian otomatis oleh AI -->
    </table>
    <div class="line"></div>
    <table class="data-table">
        <tr><td>NOMINAL</td><td class="text-right" id="dispNominal">Rp 0</td></tr>
        <tr><td>BIAYA ADMIN</td><td class="text-right" id="dispAdmin">Rp 0</td></tr>
        <tr class="bold"><td>TOTAL BAYAR</td><td class="text-right" id="dispTotal">Rp 0</td></tr>
    </table>
    <div class="line"></div>
    <div class="receipt-footer" id="dispFooter">
        TERIMA KASIH<br>
        SIMPAN STRUK INI SEBAGAI BUKTI PEMBAYARAN YANG SAH
    </div>
</div>

<script>
async function fileToBase64(file) {
    return new Promise((resolve, reject) => {
        const reader = new FileReader();
        reader.readAsDataURL(file);
        reader.onload = () => resolve(reader.result);
        reader.onerror = error => reject(error);
    });
}

async function generateReceipt() {
    const apiKey = document.getElementById('apiKey').value;
    const agentName = document.getElementById('agentName').value || "AGEN MINI ATM";
    const storeAddress = document.getElementById('storeAddress').value || "-";
    const footerMessage = document.getElementById('footerMessage').value || "TERIMA KASIH";
    const imageInput = document.getElementById('imageInput').files[0];
    const inputText = document.getElementById('inputText').value;

    if (!apiKey || (!imageInput && !inputText)) {
        alert("Harap isi API Key Gemini dan masukkan gambar atau teks transaksi!");
        return;
    }

    let contentsParts = [];
    
    let promptText = `Ekstrak teks bukti transaksi agen bank/mini ATM berikut menjadi format JSON yang valid.
PENTING:
1. JANGAN masukkan unsur 'ADMIN' atau 'BIAYA ADMIN' ke dalam array 'detail' di bawah, karena komponen admin sudah disediakan tempat khususnya sendiri di properti 'admin' terpisah (agar tidak tercetak dobel).
2. Ekstrak tanggal dan jam transaksi yang tertera pada gambar/teks ke dalam properti "tanggal" (misal: "27-02-2026 14:35:20"). Jika tidak ditemukan, kosongkan atau isi dengan waktu sekarang.

Format JSON wajib persis seperti ini:
{
  "jenis_transaksi": "TRANSFER BANK / TARIK TUNAI / TOKEN PLN / DLL",
  "status": "BERHASIL / SUKSES",
  "tanggal": "DD-MM-YYYY HH:MM:SS",
  "detail": [
    {"label": "BANK TUJUAN", "value": "BCA"},
    {"label": "NO. REK/ID", "value": "1234567890"},
    {"label": "NAMA", "value": "BUDI SANTOSO"},
    {"label": "NO. REF/SN", "value": "9876543210"}
  ],
  "nominal": 500000,
  "admin": 5000,
  "total": 505000
}`;

    if (inputText) {
        promptText += `\nTeks Tambahan: "${inputText}"`;
    }

    contentsParts.push({ text: promptText });

    if (imageInput) {
        const base64Full = await fileToBase64(imageInput);
        const mimeType = imageInput.type;
        const base64Data = base64Full.split(',')[1]; // Ambil data base64 murni tanpa prefix data:image/...
        
        contentsParts.push({
            inline_data: {
                mime_type: mimeType,
                data: base64Data
            }
        });
    }

    try {
        // Menggunakan model gemini-2.5-flash dengan konfigurasi output JSON
        const response = await fetch(`https://generativelanguage.googleapis.com/v1beta/models/gemini-2.5-flash:generateContent?key=${apiKey}`, {
            method: 'POST',
            headers: {
                'Content-Type': 'application/json'
            },
            body: JSON.stringify({
                contents: [
                    {
                        parts: contentsParts
                    }
                ],
                generationConfig: {
                    temperature: 0.1,
                    responseMimeType: "application/json"
                }
            })
        });

        const data = await response.json();
        
        if (data.error) {
            throw new Error(data.error.message);
        }

        const contentText = data.candidates[0].content.parts[0].text.trim();
        const res = JSON.parse(contentText);
        
        renderReceipt(res, agentName, storeAddress, footerMessage);
    } catch (error) {
        console.error(error);
        alert("Gagal memproses data dengan Gemini AI: " + error.message);
    }
}

function renderReceipt(data, agentName, storeAddress, footerMessage) {
    document.getElementById('dispAgentName').innerText = agentName;
    document.getElementById('dispStoreAddress').innerText = storeAddress;
    
    document.getElementById('dispFooter').innerHTML = `${footerMessage}<br><span style="font-size:9px;">SIMPAN STRUK INI SEBAGAI BUKTI PEMBAYARAN YANG SAH</span>`;
    
    const transactionDate = data.tanggal ? data.tanggal : new Date().toLocaleString('id-ID');
    document.getElementById('dispDate').innerText = "TGL: " + transactionDate;
    
    document.getElementById('dispStatus').innerText = `${data.jenis_transaksi}\n[${data.status}]`;
    
    const detailsTable = document.getElementById('receiptDetails');
    detailsTable.innerHTML = '';
    
    if (data.detail && Array.isArray(data.detail)) {
        data.detail.forEach(item => {
            const labelUpper = item.label.toUpperCase();
            if (labelUpper.includes('ADMIN')) {
                return; // Mencegah admin tercetak dobel di tabel tengah
            }

            const row = `<tr>
                <td width="40%">${item.label}</td>
                <td width="5%">:</td>
                <td width="55%" class="bold">${item.value}</td>
            </tr>`;
            detailsTable.innerHTML += row;
        });
    }

    document.getElementById('dispNominal').innerText = "Rp " + (data.nominal || 0).toLocaleString('id-ID');
    document.getElementById('dispAdmin').innerText = "Rp " + (data.admin || 0).toLocaleString('id-ID');
    document.getElementById('dispTotal').innerText = "Rp " + (data.total || 0).toLocaleString('id-ID');
}
</script>

</body>
</html>
