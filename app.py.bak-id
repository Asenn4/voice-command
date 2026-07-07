import streamlit as st
from transformers import pipeline
import torch
from Sastrawi.Stemmer.StemmerFactory import StemmerFactory
from Sastrawi.StopWordRemover.StopWordRemoverFactory import StopWordRemoverFactory
from thefuzz import process, fuzz
import webbrowser
import time
import subprocess
from audio_recorder_streamlit import audio_recorder

# ==========================================
# 1. SETUP NLP (Sastrawi & TheFuzz)
# ==========================================
st.set_page_config(page_title="Voice Command Automation", page_icon="🎙️")

@st.cache_resource
def setup_nlp():
    # Inisialisasi Sastrawi Stemmer
    stemmer_factory = StemmerFactory()
    stemmer = stemmer_factory.create_stemmer()
    
    # Inisialisasi Sastrawi Stopword Remover
    stopword_factory = StopWordRemoverFactory()
    stopword_remover = stopword_factory.create_stop_word_remover()
    
    return stemmer, stopword_remover

stemmer, stopword_remover = setup_nlp()

# ==========================================
# 2. DEFINISI INTENT (COMMANDS)
# ==========================================
# Daftar perintah (rules) yang dipahami oleh sistem
COMMANDS = {
    # 🌐 DAFTAR BUKA WEBSITE
    "buka google": {"action": "open_url", "url": "https://www.google.com"},
    "buka youtube": {"action": "open_url", "url": "https://www.youtube.com"},
    "buka github": {"action": "open_url", "url": "https://github.com"},
    "buka chat gpt": {"action": "open_url", "url": "https://chatgpt.com"},
    "buka gemini": {"action": "open_url", "url": "https://gemini.google.com"},
    "buka whatsapp": {"action": "open_url", "url": "https://web.whatsapp.com"},
    
    # 💻 DAFTAR BUKA APLIKASI
    "putar musik": {"action": "run_os_command", "command": "start spotify:", "message": "Membuka Spotify..."},
    "buka vscode": {"action": "run_os_command", "command": "code", "message": "Membuka Visual Studio Code..."},
    "buka terminal": {"action": "run_os_command", "command": "start cmd", "message": "Membuka Terminal..."},
    "cari dokumen": {"action": "run_os_command", "command": "explorer", "message": "Membuka File Explorer..."},
    "buka notepad": {"action": "run_os_command", "command": "notepad", "message": "Membuka Notepad..."},
    "buka kalkulator": {"action": "run_os_command", "command": "calc", "message": "Membuka Kalkulator..."},
    "buka word": {"action": "run_os_command", "command": "start winword", "message": "Membuka Microsoft Word..."},
    "buka excel": {"action": "run_os_command", "command": "start excel", "message": "Membuka Microsoft Excel..."},
    "buka powerpoint": {"action": "run_os_command", "command": "start powerpnt", "message": "Membuka Microsoft PowerPoint..."},
    
    # ⚙️ PERINTAH SISTEM LAINNYA
    "matikan sistem": {"action": "simulate_os", "message": "Sistem dimatikan (Hanya simulasi demi keamanan)..."}
}

# Ambil daftar keyword perintah untuk dicocokkan dengan thefuzz
COMMAND_KEYS = list(COMMANDS.keys())

# ==========================================
# 3. SETUP MODEL WHISPER (ASR)
# ==========================================
@st.cache_resource
def load_asr_model():
    # Menggunakan model lokal yang sudah utuh (lengkap dengan config & tokenizer) (ganti sesuai lokasi model Anda)
    model_dir = r"D:\Jarvis\Production\Hasil_Train"
    
    # Langsung muat modelnya, manfaatkan GPU jika ada untuk Voice Command agar realtime!
    device = 0 if torch.cuda.is_available() else -1
    pipe = pipeline("automatic-speech-recognition", model=model_dir, device=device)
    return pipe

asr_pipeline = load_asr_model()

# ==========================================
# 4. FUNGSI PEMROSESAN UTAMA
# ==========================================
def process_voice_command(audio_bytes):
    # Tahap 1: Speech-to-Text dengan Whisper
    with st.spinner("1. Mengubah Suara menjadi Teks (Whisper STT)..."):
        # Mencegah model halusinasi / berbahasa Inggris
        gen_kwargs = {
            "language": "indonesian",
            "task": "transcribe",
            "condition_on_prev_tokens": False
        }
        transcription_result = asr_pipeline(audio_bytes, generate_kwargs=gen_kwargs)
        raw_text = transcription_result["text"].lower()
    
    st.write(f"**Teks Asli (Raw):** `{raw_text}`")
    
    # Tahap 2: NLP Preprocessing (Sastrawi)
    with st.spinner("2. Melakukan Preprocessing Teks (Sastrawi)..."):
        # Hapus stopwords (misal: "tolong", "coba")
        clean_text = stopword_remover.remove(raw_text)
        # Stemming (mencari kata dasar)
        stemmed_text = stemmer.stem(clean_text)
        
    st.write(f"**Teks Bersih (NLP):** `{stemmed_text}`")
    
    # Tahap 3: Intent Recognition (TheFuzz)
    with st.spinner("3. Mencocokkan Perintah (TheFuzz)..."):
        # Mencari perintah yang paling mirip menggunakan fuzzy string matching
        # scorer=fuzz.token_set_ratio sangat bagus untuk kalimat bolak-balik
        best_match, score = process.extractOne(stemmed_text, COMMAND_KEYS, scorer=fuzz.token_set_ratio)
        
    st.write(f"**Intent Terdeteksi:** `{best_match}` (Tingkat Keyakinan: {score}%)")
    
    # Tahap 4: Eksekusi Perintah
    if score >= 75: # Threshold kecocokan (bisa disesuaikan)
        st.success("Perintah Dikenali! Mengeksekusi aksi...")
        action_data = COMMANDS[best_match]
        
        if action_data["action"] == "open_url":
            st.info(f"Membuka URL: {action_data['url']}")
            # webbrowser berfungsi jika dijalankan di PC lokal (Localhost)
            webbrowser.open_new_tab(action_data['url']) 
        
        elif action_data["action"] == "run_os_command":
            st.info(action_data["message"])
            try:
                # Menjalankan aplikasi secara asinkron agar tidak memblokir Streamlit
                subprocess.Popen(action_data["command"], shell=True)
            except Exception as e:
                st.error(f"Gagal menjalankan aplikasi: {e}")

        elif action_data["action"] == "simulate_os":
            st.warning(f"Menjalankan Perintah OS: {action_data['message']}")
            
    else:
        st.error("Maaf, perintah tidak dikenali atau tingkat keyakinan terlalu rendah.")

# ==========================================
# 5. UI STREAMLIT
# ==========================================
st.title("🎙️ Voice Command Automation")

st.subheader("Daftar Perintah yang Tersedia:")

# Membagi daftar menjadi dua bagian untuk dua kolom
midpoint = (len(COMMAND_KEYS) + 1) // 2
col_list1, col_list2 = st.columns(2)

with col_list1:
    st.markdown("\n".join([f"- `{cmd}`" for cmd in COMMAND_KEYS[:midpoint]]))
    
with col_list2:
    st.markdown("\n".join([f"- `{cmd}`" for cmd in COMMAND_KEYS[midpoint:]]))

st.markdown("---")

# Membungkus dalam kontainer bergaris (Card UI) agar terlihat premium
with st.container(border=True):
    # Membuat judul dan deskripsi rata tengah dengan HTML/CSS
    st.markdown("<h3 style='text-align: center; color: #1E90FF;'>🎙️ Voice Command Center</h3>", unsafe_allow_html=True)
    st.markdown("<p style='text-align: center; color: #888;'><i>Tekan logo mikrofon besar di bawah ini untuk mulai berbicara.</i></p>", unsafe_allow_html=True)
    
    # Membuat 3 kolom, dengan kolom tengah yang lebih proporsional
    col1, col2, col3 = st.columns([3, 1, 3])
    
    with col2:
        st.write("") # Spasi kosong atas
        
        # Widget mikrofon interaktif kustom yang diperbesar dan warnanya dipercantik
        audio_bytes = audio_recorder(
            text="",
            recording_color="#ff4b4b", # Merah terang (Streamlit) saat merekam
            neutral_color="#1E90FF",   # Biru modern saat stand-by
            icon_name="microphone",
            icon_size="4x",            # Ukuran diperbesar menjadi 4x
        )
        
        st.write("") # Spasi kosong bawah

if audio_bytes:
    # Menampilkan pemutar audio opsional
    st.audio(audio_bytes, format="audio/wav")
    
    # Langsung mengeksekusi perintah begitu selesai merekam
    process_voice_command(audio_bytes)
