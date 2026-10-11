# JARVIS Linux

Asisten lokal untuk Ubuntu GNOME, ditulis dengan Python. Status: **fondasi MVP v0.1, eksperimental**.

Sudah tersedia: CLI jadwal (SQLite), intent JSON melalui Ollama lokal, input suara berbasis Enter,
faster-whisper CPU, TTS sementara espeak-ng bahasa Indonesia, dan background reminder engine.
Dynamic Island, D-Bus, wake word, dan integrasi desktop lanjutan belum diimplementasikan.
Lisensi proyek belum dipilih; jangan menganggap proyek sudah berlisensi open-source untuk redistribusi.

## Mulai

Python 3.11+ diperlukan. Dari folder ini:

~~~bash
python3 -m venv .venv
source .venv/bin/activate
python -m pip install -e .
jarvis doctor
# Untuk development, simpan database di folder proyek (sudah masuk .gitignore):
export JARVIS_DATA_DIR="$PWD/.data"
jarvis init
jarvis add "Meeting proyek" "2027-01-01T15:00:00+07:00"
jarvis list --date 2027-01-01
jarvis update 1 "Meeting tim" "2027-01-01T16:00:00+07:00"
jarvis delete 1
~~~

Ganti waktu contoh dengan waktu di masa depan. Penghapusan meminta jawaban 'ya'.
CRUD jadwal bekerja tanpa Ollama atau dependensi suara. Database default:
~/.local/share/jarvis/jarvis.sqlite3. Zona waktu default Asia/Jakarta.

## AI dan suara

Jalankan Ollama secara lokal dan pilih model yang sudah tersedia dengan 'ollama list'.
Model final belum ditentukan; kemampuan bahasa Indonesia dan lisensi harus dievaluasi.

~~~bash
export JARVIS_MODEL="nama-model-lokal-anda"
jarvis ask "Apa jadwalku hari ini?"
jarvis chat
python -m pip install -e '.[voice]'
# Pada Ubuntu, instal dependensi sistem berikut bila belum tersedia:
sudo apt install libportaudio2 espeak-ng libnotify-bin
jarvis voice
~~~

Mode voice: Enter untuk mulai, Enter untuk berhenti (rekaman maksimal 30 detik).
Tinjau transkrip dan ketik 'ya' untuk memprosesnya. Penghapusan tetap memerlukan konfirmasi terpisah.
Audio hanya ditampung di RAM; aplikasi tidak membuat file rekaman.
Model Whisper diunduh saat pertama dipakai jika belum ada di cache. Untuk operasi offline penuh,
siapkan model sebelumnya atau isi JARVIS_WHISPER_MODEL dengan direktori model lokal.
TTS espeak-ng adalah baseline eksperimen, belum pilihan suara final.

## Pengingat

~~~bash
jarvis serve
~~~

Perlu sesi desktop aktif, session D-Bus, dan notify-send. Polling setiap 5 detik;
pengingat default 10 menit sebelum kegiatan. Gunakan '--reminder-minutes 0' untuk tepat waktu.
Pengingat yang terlambat hingga 24 jam dicoba saat startup. Lebih lama ditandai 'missed'.
Pengiriman gagal dicoba ulang setelah lease 120 detik. Pengiriman bersifat at-least-once:
crash setelah notifikasi diterima tetapi sebelum pencatatan dapat menyebabkan duplikasi.
Notifikasi diterima oleh layanan desktop tidak menjamin pengguna melihatnya (misalnya Do Not Disturb).

JARVIS_PROACTIVE=false menonaktifkan pengiriman otomatis tanpa menghapus jadwal.
Aktifkan pengingat otomatis saat login GNOME (tanpa sudo):

~~~bash
source scripts/dev-env.sh
python scripts/install-service.py
systemctl --user status jarvis.service --no-pager
~~~

Service memakai database yang sama dengan CLI. Lihat [panduan instalasi](docs/installation.md).

## Pengembangan

~~~bash
python -m unittest discover -s tests -v
~~~

Test memakai unittest dan dependensi core APScheduler; tidak membutuhkan model, microphone, atau Ollama.
Lihat [arsitektur](docs/architecture.md), [roadmap](docs/roadmap.md), dan
[panduan kontribusi](docs/contributing.md).

Panduan runtime audio lokal dan pengujian microphone: [Pengujian suara](docs/voice.md).
# ai_assistent
