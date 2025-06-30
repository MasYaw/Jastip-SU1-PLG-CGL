<!DOCTYPE html>
<html lang="id">
<head>
<meta charset="UTF-8" />
<meta name="viewport" content="width=device-width, initial-scale=1" />
<title>Aplikasi Jastip dengan Scan Barcode</title>
<style>
  body { font-family: Arial, sans-serif; padding: 20px; }
  video { width: 100%; max-width: 400px; border: 1px solid #ccc; }
  input, select, button { padding: 8px; margin: 8px 0; width: 100%; max-width: 400px; }
  table { width: 100%; max-width: 600px; border-collapse: collapse; margin-top: 20px; }
  th, td { border: 1px solid #ccc; padding: 8px; text-align: left; }
  th { background: #eee; }
</style>
</head>
<body>

<h2>Aplikasi Jastip Paket dengan Scan Barcode</h2>

<video id="video" autoplay></video>

<form id="packageForm">
  <label>Nomor Resi (hasil scan otomatis akan muncul di sini):</label><br />
  <input type="text" id="resi" readonly required /><br />

  <label>Nama Pemilik Paket:</label><br />
  <input type="text" id="nama" required /><br />

  <label>COD (Bayar di Tempat):</label><br />
  <select id="cod" required>
    <option value="Tidak">Tidak</option>
    <option value="Ya">Ya</option>
  </select><br />

  <label>Nominal COD (jika Ya):</label><br />
  <input type="number" id="nominalCOD" min="0" value="0" disabled /><br />

  <button type="submit">Simpan Paket</button>
</form>

<h3>Daftar Paket</h3>
<table id="packageTable">
  <thead>
    <tr>
      <th>Nomor Resi</th>
      <th>Nama Pemilik</th>
      <th>COD</th>
      <th>Nominal COD</th>
    </tr>
  </thead>
  <tbody></tbody>
</table>

<script src="https://cdn.jsdelivr.net/npm/@zxing/library@0.18.6/umd/index.min.js"></script>
<script>
  const codeReader = new ZXing.BrowserBarcodeReader();
  const videoElement = document.getElementById('video');
  const resiInput = document.getElementById('resi');
  const codSelect = document.getElementById('cod');
  const nominalInput = document.getElementById('nominalCOD');
  const packageForm = document.getElementById('packageForm');

  codSelect.addEventListener('change', () => {
    if (codSelect.value === 'Ya') {
      nominalInput.disabled = false;
      nominalInput.required = true;
    } else {
      nominalInput.disabled = true;
      nominalInput.required = false;
      nominalInput.value = 0;
    }
  });

  codeReader.decodeFromVideoDevice(null, videoElement, (result, err) => {
    if (result) {
      resiInput.value = result.text;
    }
  });

  function loadPackages() {
    return JSON.parse(localStorage.getItem('packages')) || [];
  }

  function savePackages(data) {
    localStorage.setItem('packages', JSON.stringify(data));
  }

  function renderPackages() {
    const data = loadPackages();
    const tbody = document.querySelector('#packageTable tbody');
    tbody.innerHTML = '';
    data.forEach(item => {
      const tr = document.createElement('tr');
      tr.innerHTML = `
        <td>${item.resi}</td>
        <td>${item.nama}</td>
        <td>${item.cod}</td>
        <td>${item.nominalCOD}</td>
      `;
      tbody.appendChild(tr);
    });
  }

  packageForm.addEventListener('submit', (e) => {
    e.preventDefault();
    const resi = resiInput.value.trim();
    const nama = document.getElementById('nama').value.trim();
    const cod = codSelect.value;
    const nominalCOD = cod === 'Ya' ? Number(nominalInput.value) : 0;

    if (!resi || !nama) {
      alert('Nomor resi dan nama pemilik wajib diisi!');
      return;
    }
    if (cod === 'Ya' && nominalCOD <= 0) {
      alert('Nominal COD harus lebih dari 0 jika COD Ya!');
      return;
    }

    const data = loadPackages();
    data.push({ resi, nama, cod, nominalCOD });
    savePackages(data);
    renderPackages();

    resiInput.value = '';
    packageForm.reset();
    nominalInput.disabled = true;
    nominalInput.value = 0;
  });

  renderPackages();
</script>

</body>
</html>
