# ApkSigner-Automated 🚀

Repositori ini berfungsi sebagai pusat penandatanganan (*signing*) APK otomatis berbasis **GitHub Actions** yang mendukung alur kerja antar-repositori (*Cross-Repository*). 

Dengan memisahkan proses kompilasi kode (*build*) di repositori aplikasi dan proses penandatanganan di repositori ini, berkas Keystore rilis (`.jks` / `.keystore`) kamu akan tetap aman, terisolasi, dan tidak tercampur dalam kode sumber proyek utama.

---

## 📦 Alur Kerja (Cara Kerja Sistem)

Sistem ini menggunakan metode **Push-and-Pull** yang aman melalui GitHub API:
1. **Repositori Aplikasi (`repoApk`)** selesai melakukan *build* APK debug/unsigned, lalu menitipkannya di server Artifact GitHub.
2. **`repoApk`** mengirim sinyal otomatis (*Repository Dispatch*) ke repositori ini membawa data ID Run terkait.
3. **`ApkSigner-Automated`** mendeteksi sinyal, lalu memantau (`gh run watch`) hingga repositori sebelah benar-benar selesai mengunggah file.
4. Berkas APK mentah diunduh secara privat, ditandatangani menggunakan Keystore rahasia milikmu, dan hasilnya diunggah kembali sebagai **Artifact Siap Pakai**.

---

## 🔐 Persiapan Rahasia (Repository Secrets)

Sebelum memulai, kamu wajib mendaftarkan beberapa variabel rahasia di menu **Settings -> Secrets and variables -> Actions** pada masing-masing repositori.

### 1. Token Akses Privat (PAT)
Buat sebuah **Personal Access Token (Fine-grained)** melalui pengaturan akun GitHub kamu (**Settings -> Developer Settings -> Personal Access Tokens**).
* **Izin Akses (Permissions):** Berikan akses `Contents: Read & Write` dan `Actions: Read & Write`.
* **Repository Access:** Pilih *All Repositories* atau pilih repositori aplikasi kamu dan repositori `ApkSigner-Automated`.

> ⚠️ **PENTING:** Simpan token ini dengan nama **`TOKEN_AKSES_PRIVAT`** di **KEDUA** repositori (Repositori Aplikasi dan `ApkSigner-Automated`).

### 2. Secrets Spesifik di `ApkSigner-Automated`
Daftarkan kunci-kunci berikut di dalam repositori penandatangan ini:
* `SIGNING_KEY`: Kode base64 dari file Keystore kamu (`.jks` atau `.keystore`).
* `ALIAS`: Nama alias Keystore kamu.
* `KEY_STORE_PASSWORD`: Kata sandi Keystore.
* `KEY_PASSWORD`: Kata sandi Kunci/Alias.

---

## 🛠️ Cara Penggunaan

### Langkah 1: Konfigurasi di Repositori Aplikasi (`repoApk`)
Tambahkan potongan perintah ini di bagian akhir file `.github/workflows/android.yml` milik repositori aplikasimu (tepat setelah proses *build* selesai):

```yaml
    - name: Upload Unsigned APK
      uses: actions/upload-artifact@v4
      with:
        name: apk-mentah-kamu
        path: app/build/outputs/apk/debug/app-debug.apk # Sesuaikan dengan letak output build-mu

    - name: Kirim Sinyal ke Signer Repositori
      run: |
        curl -X POST \
          -H "Accept: application/vnd.github+json" \
          -H "Authorization: Bearer ${{ secrets.TOKEN_AKSES_PRIVAT }}" \
          [https://api.github.com/repos/$](https://api.github.com/repos/$){{ github.repository_owner }}/ApkSigner-Automated/dispatches \
          -d '{"event_type": "mulai_tanda_tangan", "client_payload": { "run_id": "${{ github.run_id }}", "repo_name": "${{ github.repository }}" }}'
