# SpeedRun: Escape From Tsunami — Online Multiplayer (server.js LITE)

Game 3D (Three.js) dengan multiplayer online jarak jauh. Pemain dengan Wi-Fi, kuota Telkomsel, atau operator lain bisa bermain bersama lewat internet.

## Isi project

| File | Fungsi |
|---|---|
| `index.html` | Game lengkap (Singleplayer + Multiplayer online) |
| `server.js` | Backend: WebSocket relay + pengelola room + penyaji `index.html` |
| `package.json` | Dependency (hanya `ws`) dan perintah `npm start` |
| `README.md` | Dokumen ini |
| `cfdedc2208396c584eb11f51327f92c3.jpg` | Gambar bawaan project (tetap disertakan) |

## Cara kerja

- **Host authority**: pemain yang membuat server (HOST) menjalankan simulasi game (bot, tsunami, item) seperti sistem lama. `server.js` hanya mengelola room dan meneruskan pesan, tidak menjalankan game.
- Semua koneksi memakai **WebSocket**. Di halaman HTTPS otomatis memakai **WSS**.
- Penyimpanan hanya **in-memory**. Tidak ada database. Server yang restart berarti room hilang.

## Menjalankan di komputer sendiri

```bash
npm install
npm start
```

Buka `http://localhost:3000`. Server membaca `process.env.PORT` (default 3000) dan mendengarkan di `0.0.0.0`. Untuk mencoba multiplayer, buka dua tab/browser.

Cek status server: `http://localhost:3000/health`.

## Deploy ke GitHub + Suga

1. Upload semua file ke repository GitHub.
2. Di Suga, buat service Node.js dari repository tersebut:
   - Build/install: `npm install`
   - Start: `npm start`
   - Port: gunakan variabel `PORT` dari platform. Jangan ditulis manual.
3. Pastikan hosting mendukung **WebSocket**. HTTPS/WSS disediakan reverse proxy platform.
4. Buka URL publik service tersebut. Game dan backend ada di alamat yang sama, jadi **tidak perlu konfigurasi URL tambahan**.

> Server baru dianggap online setelah benar-benar di-deploy dan URL-nya bisa dibuka. Cek `https://URL-ANDA/health`. Hasilnya `{"ok":true,...}`.

## Konfigurasi URL backend

Jika `index.html` dihosting di tempat yang berbeda dari `server.js` (misalnya GitHub Pages), arahkan game ke backend dengan salah satu cara berikut. Urutan prioritasnya seperti ini:

1. **Parameter URL**: `https://situs-anda/?server=wss://backend-anda.example.com`
2. **Menu game**: Multiplayer → *Connection settings* → isi URL → *Save & Test* (tersimpan di browser).
3. **Di kode**: ubah `var SERVER_URL_DEFAULT='';` di `index.html` menjadi `'wss://backend-anda.example.com'`.
4. Jika kosong semua: memakai alamat yang menyajikan halaman (kasus deploy satu service).

Jika `index.html` dibuka langsung dari file (`file://`), isi URL lewat salah satu cara di atas. Memasukkan `https://...` otomatis diubah menjadi `wss://...`.

## Variabel environment (opsional)

| Variabel | Default | Keterangan |
|---|---|---|
| `PORT` | 3000 | Port (diisi hosting) |
| `ALLOWED_ORIGINS` | kosong (semua) | Daftar origin yang boleh konek, pisahkan dengan koma. Contoh: `https://user.github.io` |
| `MAX_PLAYERS` | 8 | Batas pemain per room (maksimum 8) |
| `MAX_ROOMS` | 300 | Batas jumlah room |
| `MAX_CONN_PER_IP` | 40 | Batas koneksi per IP |
| `RECONNECT_GRACE_MS` | 15000 | Slot pemain dipertahankan selama ini setelah koneksi putus |
| `MSG_RATE_PER_SEC` | 120 | Batas pesan per detik per koneksi |
| `MAX_MSG_BYTES` | 65536 | Ukuran pesan maksimum |
| `TRUST_PROXY` | aktif | Set `0` jika server tidak berada di belakang reverse proxy |

Tidak ada API key, password, atau secret di source code.

## Fitur multiplayer

- **Server List**: server publik beserta nama, jumlah pemain/kapasitas, status (Waiting / Starting / Playing / Full), mode dan map. Ada tombol JOIN dan REFRESH. Daftar diperbarui otomatis tiap 4 detik. Server yang tidak aktif otomatis hilang dari daftar.
- **Create Server**: nama, maksimal pemain (2–8), publik atau pribadi. Pengaturan map, bot, dan turnamen dipilih di lobby.
- **Private Server**: tidak muncul di Server List, hanya bisa dimasuki dengan Server ID (6 karakter, divalidasi server). Ada tombol COPY ID. Pesan jelas jika ID salah atau server penuh.
- **Lobby**: daftar pemain, tanda host, jumlah pemain, chat, tombol START RUN khusus host, late join seperti aturan lama.
- **Koneksi**: indikator Connecting / Connected / Reconnecting / Disconnected, ping (ms), reconnect otomatis (slot dipertahankan 15 detik), timeout koneksi, dan pesan error yang jelas. Satu identitas hanya bisa satu kali di satu room.
- **Host terputus**: jika host kembali dalam 15 detik, game lanjut tanpa putus. Jika tidak, pemain lain tetap bisa bermain secara lokal dan tidak crash.
- **Keamanan dan anti-spam**: whitelist tipe pesan, batas ukuran pesan, batas frekuensi pesan, batas chat, batas koneksi per IP, heartbeat ping-pong, dan pembersihan room kosong atau mati.
- **Hemat data**: posisi dikirim sekitar 12 kali per detik. Pemain yang diam hanya mengirim keep-alive. Pesan host ke semua pemain diserialisasi sekali di server.

## Endpoint HTTP

- `GET /` — game
- `GET /health` — status server
- `GET /api/servers` — daftar server publik (JSON, tanpa server pribadi)

## Catatan

- Hosting gratis sering "tidur" saat tidak dipakai. Koneksi pertama bisa butuh beberapa detik, jadi menu menampilkan indikator *Connecting*.
- Jika hosting memakai beberapa instance, pastikan hanya **1 instance**. Room disimpan di memori satu proses.
- Three.js dimuat dari CDN (`cdnjs.cloudflare.com`), jadi pemain perlu internet.
