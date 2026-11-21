# Speaker notes · Unlock agentic capabilities in your app with ML Kit GenAI

Deck: `2026-mlkit-genai-agentic.html` · 44 slide · target ±30–35 menit

Cara baca:
- **▶** = klik / tekan panah kanan (ada animasi atau step berikutnya di slide yang sama)
- *Teks miring* = catatan untuk diri sendiri, bukan untuk diucapkan
- Istilah teknis sengaja tetap bahasa Inggris (function calling, tool, agent, dll.)

---

## Pembuka

### 1 · Title
Halo semuanya, assalamu'alaikum! Terima kasih sudah datang.

Kenalin, saya Isfa, Mobile Platform Architect dan GDE Android. Kalau mau ngobrol lanjut setelah sesi, bisa cari saya di X, `isfhaani`.

Hari ini kita akan bahas gimana caranya bikin aplikasi Android kita punya kemampuan *agentic*, jadi bukan cuma bisa ngobrol, tapi bisa *ngerjain sesuatu*, dan semuanya jalan **on-device** pakai Gemini Nano lewat ML Kit GenAI.

*Sebut juga: opini di sini pendapat pribadi, bukan mewakili kantor.*

### 2 · Android Dev Summit, last month
Sedikit cerita dulu. Bulan lalu saya berkesempatan datang ke Android Dev Summit.

*Ceritakan 1–2 momen dari foto: suasana, orang yang ditemui, sesi yang seru.*

Dari banyak sesi di sana, ada satu yang paling nempel di kepala saya, dan itu yang jadi topik kita hari ini.

### 3 · The session
Ini sesinya: *Unlock agentic capabilities in your app with ML Kit GenAI*. Videonya sudah ada di YouTube, teman-teman bisa tonton lengkapnya nanti.

Saya sudah minta izin ke Victoria Lestari, salah satu pembicaranya, untuk membawakan materi ini ke komunitas Android di Indonesia. Jadi hari ini saya bawakan versi saya, dengan contoh aplikasi dan konteks yang lebih dekat dengan kita.

### 4 · Outline
Alurnya begini:
1. Kenalan dulu dengan GenAI on-device.
2. Lalu kita naik level pelan-pelan: **single-step** function calling,
3. **multi-step**, model merangkai beberapa tool sekaligus,
4. **multi-turn**, agent yang ingat percakapan,
5. sampai **System AppFunctions**, di mana asisten sistem seperti Gemini bisa memanggil fungsi di aplikasi kita.
6. Terakhir TL;DR untuk dibawa pulang.

---

## Bagian 0 · On-device GenAI

### 5 · Divider: On-device GenAI
Kita mulai dari fondasinya: ML Kit, Gemini Nano, dan AICore.

### 6 · ML Kit GenAI has two layers
ML Kit GenAI itu punya dua lapisan.

Lapisan pertama: **API siap pakai** untuk tugas-tugas umum, seperti summarization, proofreading, rewriting, image description, dan speech recognition. Tinggal pakai, nggak perlu mikirin prompt.

▶ Lapisan kedua: **akses langsung ke foundation model-nya**, yaitu Gemini Nano, lewat Prompt API. Di sini kita yang pegang kendali: prompt kita, tool kita, alur kita. Dan di lapisan inilah agent hidup. Fokus kita hari ini ada di lapisan kedua ini.

### 7 · On-device and cloud
Contoh nyatanya sudah ada di HP kita.

Di sisi kiri, yang jalan **on-device** dengan Gemini Nano: ringkasan telepon otomatis, deteksi panggilan penipuan secara real-time. Datanya sensitif dan butuh cepat, jadi cocok diproses di perangkat.

Di sisi kanan, yang jalan di **cloud**: Ask Maps, atau Gemini yang ngobrol panjang soal ide tema ulang tahun. Ini butuh pengetahuan dunia yang luas dan model yang lebih besar.

▶ Ada juga contoh on-device yang lain: mencari info dari screenshot kita sendiri. "Tempat burger di Berkeley itu namanya apa ya?" Dijawab dari screenshot yang tersimpan, tanpa keluar dari HP.

Intinya: dua-duanya saling melengkapi. Hari ini kita fokus ke yang kiri, yang on-device.

### 8 · Gemini Nano 4 is available today
Kabar baiknya, Gemini Nano 4 sudah tersedia sekarang, di beberapa perangkat flagship.

### 9 · 100M+
Dan ini angka yang menurut saya penting: ditargetkan lebih dari **100 juta** perangkat Android punya Gemini Nano 4 di awal 2027. Ini angka dari Android Dev Summit.

Jadi ini bukan lagi fitur eksperimen yang cuma bisa dicoba di satu-dua HP. Sebentar lagi ini jadi kemampuan yang bisa kita andalkan di banyak perangkat pengguna kita.

### 10 · Agentic AI
Lalu, apa artinya "agentic"?

Agentic AI bisa **merencanakan dan mengeksekusi** tugas yang kompleks di dalam aplikasi kita. Bukan cuma menjawab, tapi bertindak.

▶ Dan fitur kuncinya, **function calling**, sudah bisa dicoba lewat AICore Developer Preview.

### 11 · Two ways to call Gemini Nano
Ada dua cara memanggil Gemini Nano untuk function calling.

Pertama, **direct access** lewat `prompt-api`. Kita dapat tipe-tipe untuk function dan riwayat percakapan, tapi sisanya kita kerjakan sendiri: kita yang mengeksekusi fungsi, menyimpan riwayat, mengulang panggilan, dan membuat guardrail.

▶ Kedua, **agentic flow** lewat `agent-api`, masih alpha. Di sini framework yang mengurus semuanya: eksekusi fungsi otomatis, simpan riwayat, mengatur loop orkestrasinya, plus guardrail otomatis.

Dua-duanya tetap jalan di atas Gemini Nano lewat AICore. Hari ini kita mostly pakai `agent-api`, dan nanti di akhir kita lihat kapan sebaiknya turun ke `prompt-api`.

### 12 · Money Tracker (sample app)
Biar nggak cuma teori, saya bikin sample app: **Money Tracker**.

Ini aplikasi pencatat pengeluaran yang membaca email notifikasi dari bank, dan punya asisten yang ditenagai ML Kit GenAI dan AppFunctions, semuanya on-device. Source code-nya ada di GitHub, link-nya ada di slide terakhir juga.

### 13 · From one call to a system assistant
Dari aplikasi ini, kita akan naik empat level:
1. **Single-step**: satu prompt, satu tool call.
2. **Multi-step**: model merangkai beberapa tool call dalam satu giliran.
3. **Multi-turn**: riwayat percakapan terbawa antar giliran.
4. **System AppFunction**: agent sistem memanggil fungsi aplikasi kita.

---

## Bagian 1 · Single-step

### 14 · Divider: 01 Single-step
Level pertama: satu prompt, satu tool call.

### 15 · Demo: Single-step
*Jalankan demo, atau tunjuk screenshot-nya.*

Saya tanya: "How much did I spend on transport this month?"

Gemini Nano memanggil satu fungsi, `getSpendingSummary` dengan kategori Transport, dan jawabannya: Rp 50.000 untuk transportasi, dari 2 transaksi.

Kelihatannya sederhana, tapi mari kita lihat apa yang terjadi di baliknya.

### 16 · What happens under the hood
Mulai dari prompt pengguna.

▶ Kalau model **tidak punya tool**, jawabannya ya cuma: "Sorry, I can't help with that." Model nggak tahu data keuangan kita.

▶ Sekarang kita kasih daftar tool dari aplikasi: `addExpense`, `listBankAccounts`, `getSpendingSummary`, `getRecentExpenses`.

▶ Model membaca prompt plus daftar tool, lalu memutuskan sendiri: panggil `getSpendingSummary`, lengkap dengan parameternya, tanggal awal bulan sampai hari ini, kategori Transport. Model yang menentukan tool dan argumennya, aplikasi kita yang menjalankan.

### 17 · Android MCP
Tool-tool ini datang dari mana? Dari **AppFunctions**. Anggap saja ini semacam "MCP-nya Android".

Mulai dari aplikasi kita.
▶ Aplikasi mengekspos **services**,
▶ **data** dan **actions**.
▶ Semua itu didaftarkan ke Android OS, ke sebuah **registry**, tempat pusat yang mencatat kemampuan setiap aplikasi.
▶ Hasilnya: pengguna bisa menyelesaikan tugas lewat agent dan interaksi tingkat sistem, tanpa harus membuka aplikasinya satu per satu.

### 18 · Android Intelligence System
Dan semua ini adalah bagian dari visi yang lebih besar: **Android Intelligence System**.

Android sekarang bukan cuma sistem operasi yang menjalankan aplikasi, tapi sistem yang *cerdas*, yang bisa memahami maksud pengguna lalu menghubungkannya ke kemampuan aplikasi-aplikasi di dalamnya. AppFunctions adalah cara aplikasi kita ikut masuk ke sistem itu.

### 19 · @AppFunction (code)
Ini contoh kode untuk mengekspos sebuah tool. Cukup fungsi `suspend` biasa, diberi anotasi `@AppFunction`.

▶ Yang penting dicatat: **KDoc-nya adalah deskripsi tool yang dibaca model.** Jadi tulis KDoc seperti kita menjelaskan ke rekan kerja: kapan dipakai, satuan nilainya apa (di sini rupiah utuh, "25k" berarti 25000), dan contoh kategorinya.

▶ Lalu access level, di sini `ACCESS_LEVEL_SELF`, menentukan **siapa yang boleh memanggil** fungsi ini. Nanti kita bahas lagi di bagian System AppFunctions.

### 20 · Agentic workflow building blocks
Ada empat komponen utama:
**AppFunctions**: mendaftarkan tool yang bisa dipanggil agent.
▶ **Agent**: konfigurasi agent, yaitu nama, instruksi, dan tool-nya.
▶ **AgentRunner**: yang menjalankan dan mengelola sesi eksekusi.
▶ **AgentEvent**: event yang keluar selama eksekusi, seperti pemanggilan fungsi, responsnya, dan teks.

### 21 · Check support (code)
Satu hal yang wajib: **cek dulu apakah perangkatnya mendukung function calling.** Ambil client-nya, cek `getAvailableFeatures()`, kalau ada `FUNCTION_CALLING` baru kita jalankan. Kalau tidak, siapkan fallback yang baik. Ingat, belum semua perangkat punya.

### 22 · Run a single turn (code)
Kalau didukung, langkahnya ada empat.

▶ Pertama, bangun konfigurasi agent pakai `Agent.builder()`:
▶ beri nama,
▶ beri instruksi, dan aktifkan AppFunctions. Dengan satu baris `enableAppFunctions()`, semua fungsi tadi otomatis jadi tool.
▶ Kedua, buat `AgentRunner`.
▶ Ketiga, siapkan prompt dari input pengguna.
▶ Keempat, jalankan `runSingleTurnAsync`,
▶ dan kumpulkan token teksnya untuk ditampilkan ke UI.

Selesai. Itu level satu.

---

## Bagian 2 · Multi-step

### 23 · Divider: 02 Multi-step
Level dua: model merangkai beberapa tool dalam satu giliran.

### 24 · Demo: Multi-step
*Jalankan demo.*

Sekarang saya bilang: "Add 50k lunch on my Jago."

Perhatikan, model belum tahu ID rekening "Jago" itu berapa. Jadi dia **cari dulu** daftar rekening, ketemu Bank Jago, baru menambahkan pengeluarannya. Hasilnya: Rp 50.000 untuk makan siang, dari Bank Jago.

### 25 · The model chains tools on its own
Ini jejaknya.

Panggilan pertama: `listBankAccounts()`. Hasilnya BCA dengan ID 1 dan Bank Jago dengan ID 2.

▶ Panggilan kedua: `addExpense` dengan jumlah 50.000, kategori Food & Drink, `bankAccountId` 2, catatan "lunch".

▶ Lalu model menutup dengan jawaban ke pengguna.

Dua tool call, satu giliran pengguna. Kita nggak menulis logika "cari bank dulu, baru tambah". Modelnya yang merencanakan.

### 26 · Multi-step function calling (diagram)
Secara alur, begini yang terjadi antara aplikasi dan Gemini Nano:
▶ Aplikasi mengirim query plus daftar tool.
▶ Model membalas dengan tool call pertama.
▶ Aplikasi mengeksekusi, lalu mengirim lagi: query, tool, call 1, dan respons 1.
▶ Dan seterusnya, bisa berulang beberapa kali,
▶ sampai tool call ke-n,
▶ hingga akhirnya model mengembalikan jawaban final.

Perhatikan konteksnya terus bertambah di setiap putaran. Di model kecil on-device, ini penting untuk diingat.

### 27 · Tool descriptions (code)
Kenapa model tahu harus panggil `listBankAccounts` dulu? Karena kita **memberitahunya lewat KDoc.**

▶ Di `listBankAccounts` kita tulis: pakai ini untuk mengubah nama bank seperti "my Jago" jadi `bankAccountId`. Di `addExpense` kita tulis: kalau pengguna punya beberapa bank, ambil ID-nya dari `listBankAccounts` dulu.

Deskripsi tool yang jelas = rangkaian tool yang benar.

### 28 · Guardrails & events (code)
Dua hal tambahan di level ini.

▶ Pertama, **guardrail**: `setMaxTurns(5)`. Ini pengaman supaya model tidak terjebak loop memanggil tool terus-menerus. Di instruksi juga kita kasih konteks seperti satuan rupiah dan tanggal hari ini.

▶ Kedua, **event**. Saat dijalankan, kita bisa dengar `AgentEvent`: `FunctionCall` untuk menampilkan "sedang menjalankan tool…", `FunctionResponse` saat selesai, dan `TextMessage` untuk jawabannya. Jadi UI bisa memberi feedback live ke pengguna, nggak cuma loading kosong.

---

## Bagian 3 · Multi-turn

### 29 · Divider: 03 Multi-turn
Level tiga: riwayat terbawa antar giliran.

### 30 · Demo: Multi-turn
*Jalankan demo.*

Saya bilang: "Add 25k coffee." Agent-nya **bertanya balik**: "Pakai rekening yang mana?" Saya jawab "BCA." Lalu saya ralat: "Actually make it 30k."

Agent paham bahwa "it" itu kopi yang barusan, dan mengubahnya jadi Rp 30.000. Ini hanya mungkin kalau agent mengingat percakapan sebelumnya.

### 31 · runSingleTurnAsync vs runAsync
Sebelum lihat kodenya, ada dua cara menjalankan agent yang perlu dibedakan.

`runSingleTurnAsync()` dipanggil langsung di `AgentRunner`. Sifatnya **stateless**: setiap panggilan mulai dari nol, tidak ingat apa-apa. Tapi di dalam satu giliran itu model tetap bisa merangkai beberapa tool, seperti demo multi-step tadi. Cocok untuk perintah sekali jalan.

▶ `runAsync()` dipanggil di sebuah **session**. Sifatnya **stateful**: riwayat percakapan disimpan otomatis antar giliran. Inilah yang membuat agent bisa bertanya balik dan pengguna bisa meralat, seperti demo kopi tadi. Cocok untuk chat, dan bisa di-reset dengan `session.delete()`.

### 32 · Sessions (code)
Caranya dengan **session**.

▶ Ambil atau buat session per chat dengan `getOrCreateSession(chatId)`. Setiap kali `session.runAsync`, riwayat otomatis tersimpan antar giliran, jadi kita tidak perlu mengelola history sendiri. Dan saat pengguna menekan "New conversation", tinggal `session.delete()`.

### 33 · Need more control? Go direct.
Nah, `agent-api` itu nyaman karena framework yang menjalankan loop-nya.

▶ Tapi kalau kita butuh kontrol lebih, misalnya mengatur sendiri apa yang masuk ke riwayat, turun ke **direct access** dengan `prompt-api`. Konsekuensinya, semua kita kerjakan sendiri: eksekusi fungsi, simpan riwayat, loop, dan guardrail.

### 34 · Custom history (code)
Contoh kapan direct access berguna.

▶ Di giliran pertama, kita eksekusi fungsinya langsung tanpa panggilan model kedua. Lalu di giliran kedua, kita kirim **hanya prompt pengguna** sebagai riwayat, tanpa detail tool call dan responsnya. Tujuannya menghemat token konteks, yang di model on-device itu terbatas.

### 35 · Going further: complex use cases
Dari sini, kemungkinannya makin luas:
▶ **Multi-agent**: sub-agent khusus yang saling bekerja sama dan mendelegasikan tugas.
▶ **Hybrid routing**: memilih secara dinamis antara Gemini Nano on-device dan model di cloud.
▶ **Custom memory dan context compaction**: meringkas atau memangkas riwayat panjang supaya muat di batas konteks on-device.

---

## Bagian 4 · System AppFunctions

### 36 · Divider: 04 System AppFunctions
Level terakhir: dari aksi di dalam aplikasi, ke asisten tingkat sistem.

### 37 · One function, two kinds of agents
Ingat access level di anotasi tadi? Di sini dia jadi penting.

▶ Dengan `ACCESS_LEVEL_SELF`, fungsi kita hanya bisa dipanggil oleh **agent di dalam aplikasi kita sendiri**, dengan UI asisten kita sendiri.

▶ Dengan `ACCESS_LEVEL_ANDROID_TRUSTED`, fungsi yang **sama** juga bisa dipanggil oleh **agent sistem**, misalnya Gemini di Android.

Satu fungsi, dua jenis agent.

### 38 · Going system-wide? Register first.
Tapi ada satu hal yang saya baru tahu dan menurut saya penting.

Langkah pertama memang mengekspos AppFunctions dengan access level `ANDROID_TRUSTED`.

▶ Tapi itu belum cukup. Supaya sistem tahu agent mana yang boleh memanggil fungsi aplikasi kita, **aplikasi kita harus didaftarkan dulu lewat Play Console**. Formulirnya ada di link ini, silakan scan QR-nya.

▶ Setelah disetujui, baru fungsi kita **bisa ditemukan dan dipanggil** oleh agent sistem seperti Gemini.

*Beri jeda sebentar supaya audiens sempat scan QR.*

### 39 · Demo: System assistant
Ini contoh hasilnya, dari sample app Jetpacker di sesi aslinya.

Pengguna bilang ke Gemini: "@Jetpacker, tolong tambahkan kopi tujuh dolar ke pengeluaran trip."

Gemini terhubung ke Jetpacker dan memanggil AppFunction-nya, **tanpa membuka aplikasinya sama sekali**.

▶ Dan kopi $7 sudah tercatat di daftar pengeluaran.

---

## Penutup

### 40 · Demo (live)
Oke, cukup slide-nya. Sekarang kita lihat langsung Money Tracker jalan di HP, semuanya on-device.

*Pindah ke layar HP. Urutan aman: tanya pengeluaran transport (single-step) → "add 50k lunch on my Jago" (multi-step) → "add 25k coffee", jawab "BCA", lalu "actually make it 30k" (multi-turn). Kalau demo gagal, kembali ke slide demo 15, 24, dan 30.*

### 41 · Recap: Four levels of agentic
Rangkuman perjalanan kita hari ini:
1. **Single-step**: satu prompt, satu tool call.
2. **Multi-step**: model merangkai beberapa tool dalam satu giliran.
3. **Multi-turn**: session menjaga riwayat antar giliran.
4. **System AppFunction**: agent sistem memanggil fungsi aplikasi kita.

Teman-teman bisa mulai dari level satu, lalu naik sesuai kebutuhan aplikasinya.

### 42 · TL;DR
Kalau cuma bisa ingat tiga hal:
▶ **Jaga set tool tetap fokus.** Modelnya kecil, jadi kasih sedikit tool saja dengan deskripsi yang jelas.
▶ **Pasang guardrail max turns** dengan `setMaxTurns()`, supaya tidak loop tanpa henti.
▶ **Cek ketersediaan dan siapkan fallback.** Belum semua perangkat punya kemampuan ini.

### 43 · Start building today
Silakan mulai dari sini:
- Source code sample app Money Tracker di GitHub,
- Android agent skills di `goo.gle/android-skills`,
- dan AICore Developer Preview di `goo.gle/aicore-dev-preview`.

Scan QR ini untuk membuka slide ini, semua link-nya ada di dalam. *Beri jeda untuk scan.*

### 44 · Thanks!
Terima kasih banyak, barakallah fiik!

Kalau ada pertanyaan, silakan sekarang, atau sapa saya setelah sesi, atau di X `isfhaani` dan di isfa.dev.
