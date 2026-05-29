# 🎙️ Voice Command Automation (Jarvis)

Aplikasi Asisten Virtual Berbasis Suara (Voice Command) menggunakan **Streamlit**, **Hugging Face Transformers (Whisper)**, dan pemrosesan bahasa alami (NLP) dengan **Sastrawi** untuk memahami perintah berbahasa Indonesia. Aplikasi ini dirancang untuk dapat membuka website, menjalankan aplikasi desktop, dan mempermudah otomatisasi komputer Anda hanya melalui suara.

## ✨ Fitur Utama

- **Speech-to-Text (ASR):** Menggunakan model lokal Whisper yang telah di-fine-tuning khusus untuk bahasa Indonesia agar responsif dan akurat.
- **NLP Preprocessing:** Memanfaatkan *Sastrawi* (Stemmer & Stopword Remover) untuk membersihkan teks dan mencari kata dasar dari perintah suara.
- **Intent Recognition:** Menggunakan algoritma pencocokan string fuzzy (`thefuzz`) yang tahan terhadap variasi atau kalimat yang dibolak-balik.
- **Otomatisasi PC (OS Automation):**
  - 🌐 Membuka situs web (Google, YouTube, GitHub, WhatsApp Web, ChatGPT, dll).
  - 💻 Membuka aplikasi desktop (Spotify, VS Code, Terminal, Notepad, Word, Excel, dll).
- **UI Interaktif:** Tampilan web sederhana dan elegan yang dibangun di atas framework Streamlit dengan tombol mikrofon interaktif.

## 🛠️ Persyaratan Sistem

- Python 3.8 atau lebih baru.
- Mikrofon yang terhubung dan berfungsi dengan baik.
- Disarankan menggunakan GPU yang didukung CUDA (NVIDIA) agar pengenalan suara dapat diproses secara *real-time*, meskipun CPU tetap dapat digunakan.
- Model Whisper lokal yang sudah dilatih/diunduh.

## 📦 Instalasi

1. **Clone repositori ini atau salin folder proyek ke komputer Anda.**
2. **Masuk ke direktori aplikasi:**
   ```bash
   cd App
   ```
3. **Instal semua dependensi yang dibutuhkan:**
   ```bash
   pip install -r requirements.txt
   ```
4. **Siapkan Model Whisper:**
   Pastikan Anda memiliki model Whisper di komputer Anda. Secara default, aplikasi akan membaca model dari direktori `D:\Jarvis\Production\Hasil_Train`. Anda dapat mengubah variabel `model_dir` di file `app.py` jika model Anda berada di folder yang berbeda.

## 🚀 Cara Menggunakan

1. **Jalankan aplikasi Streamlit:**
   ```bash
   streamlit run app.py
   ```
2. Aplikasi akan otomatis terbuka di browser default Anda (biasanya di `http://localhost:8501`).
3. Pada halaman utama, klik ikon **Mikrofon Besar** di tengah layar untuk mulai merekam.
4. Ucapkan perintah Anda dengan jelas. Contoh:
   - *"Tolong dong buka youtube"*
   - *"Coba putar musik"*
   - *"Buka kalkulator"*
   - *"Buka vscode"*
5. Tunggu sejenak, aplikasi akan:
   - Mengubah suara menjadi teks.
   - Membersihkan kata-kata tambahan (seperti "tolong" atau "coba").
   - Mendeteksi niat/intent dari perintah Anda.
   - Mengeksekusi aksi yang sesuai (membuka web/aplikasi).

## 📝 Konfigurasi Perintah (Commands)

Anda dapat dengan mudah menambahkan perintah baru dengan mengedit variabel `COMMANDS` di dalam file `app.py`. 
Contoh menambahkan perintah baru:
```python
COMMANDS = {
    # ... perintah yang sudah ada ...
    "buka facebook": {"action": "open_url", "url": "https://facebook.com"},
    "buka kamera": {"action": "run_os_command", "command": "start microsoft.windows.camera:", "message": "Membuka Kamera..."},
}
```

## 📚 Teknologi yang Digunakan

- [Streamlit](https://streamlit.io/) & [Audio Recorder Streamlit](https://pypi.org/project/audio-recorder-streamlit/)
- [Hugging Face Transformers](https://huggingface.co/docs/transformers/index) (Whisper)
- [PyTorch](https://pytorch.org/)
- [Sastrawi](https://github.com/harisetyo/PySastrawi) (NLP Bahasa Indonesia)
- [TheFuzz](https://github.com/seatgeek/thefuzz) (Fuzzy String Matching)
