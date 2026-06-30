# Beranda Pribadi — Login Google + Menu

Aplikasi statis (1 file `index.html`, tanpa build step) dengan login Google
lewat Firebase Authentication, dan data menu tersimpan di Firestore
(per akun, hanya bisa diakses oleh pemiliknya).

## 1. Bikin project Firebase

1. Buka https://console.firebase.google.com → **Add project** → ikuti langkahnya (bisa matikan Google Analytics, gak wajib).
2. Di dashboard project → klik ikon **Web (`</>`)** untuk daftarkan web app.
   - Kasih nama bebas (misal "beranda-pribadi"), gak perlu centang Firebase Hosting.
   - Setelah selesai, Firebase kasih objek `firebaseConfig` — **copy semua isinya**.
3. Buka file `index.html`, cari bagian:
   ```js
   const firebaseConfig = {
     apiKey: "GANTI_DENGAN_API_KEY",
     ...
   };
   ```
   Ganti dengan config asli dari Firebase.

## 2. Aktifkan Google Sign-In

1. Di Firebase Console → **Build → Authentication → Get started**.
2. Tab **Sign-in method** → pilih **Google** → **Enable** → simpan.
3. Masih di Authentication → **Settings → Authorized domains** → nanti setelah deploy ke Netlify, tambahkan domain Netlify ana di sini (misal `nama-app.netlify.app`). Tanpa ini, login akan ditolak.

## 3. Aktifkan Firestore + pasang security rules

1. Firebase Console → **Build → Firestore Database → Create database**.
   - Pilih lokasi server (misal `asia-southeast2` Jakarta), mode **Production**.
2. Setelah dibuat, buka tab **Rules**, ganti isinya dengan:
   ```
   rules_version = '2';
   service cloud.firestore {
     match /databases/{database}/documents {
       match /users/{userId}/menus/{menuId} {
         allow read, write: if request.auth != null && request.auth.uid == userId;
       }
     }
   }
   ```
3. Klik **Publish**.

   Rules ini memastikan: setiap orang cuma bisa baca/tulis menu miliknya sendiri,
   walau database-nya satu untuk semua user.

## 4. Deploy ke Netlify

Sama seperti alur TranskripsAI:

1. Buat folder ini jadi repo GitHub (atau drag & drop langsung ke Netlify).
2. Di https://app.netlify.com → **Add new site → Deploy manually** (kalau drag & drop)
   atau **Import from Git** (kalau pakai repo).
3. Karena tidak ada build step, kosongkan **Build command**, set **Publish directory** ke folder ini (`.` atau tempat `index.html` berada).
4. Setelah deploy, catat domain yang diberikan Netlify (misal `https://beranda-ana.netlify.app`).
5. **Wajib balik ke langkah 2.3** — tambahkan domain Netlify ini ke **Authorized domains** di Firebase Authentication, kalau tidak login Google akan gagal dengan error `auth/unauthorized-domain`.

## Catatan keamanan

- Tidak perlu Cloudflare Worker di versi ini — Firestore security rules sudah cukup untuk membatasi akses per user, karena verifikasi identitas dilakukan Firebase di sisi server.
- `apiKey` di `firebaseConfig` **aman untuk publik** (beda dengan API key Groq di TranskripsAI) — itu cuma identifier project, bukan secret. Yang menjaga keamanan data adalah Firestore rules di atas.
- Kalau nanti mau tambah fitur (misal kategori menu, urutan custom), tinggal sesuaikan struktur dokumen di Firestore dan query-nya di `index.html`.
