# Join Whitelist

Gunakan skill ini setiap kali user meminta join whitelist, apply WL, daftar WL, atau early access ke sebuah proyek NFT. Aturan utama skill ini adalah probe curl dulu, dan hanya pakai browser kalau curl tidak tembus.

## Aturan berdiri

- Probe curl dulu setiap kali user suruh join WL.

- Kalau probe curl tembus dan satu aplikasi percobaan diterima server, gas semua pasangan lewat curl paralel dan jangan pakai web browser lagi untuk submit itu.

- Kalau endpoint curl tidak ditemukan atau probe gagal, pakai jalur browser dan laporkan alasannya dengan jujur, jangan menebak endpoint.

- Jangan pernah submit otomatis tanpa data yang user berikan atau user setujui. Data proyek yang belum ada bukan alasan untuk mengarang.

## Klarifikasi data sebelum submit

- Tanyakan atau konfirmasi set data yang dipakai sebelum submit pertama di proyek itu.

- Pakai file `workspace/elynx-apply/submit-wl-skyzee.json` sebagai set default 15 pasang X dan wallet, kecuali user mengirim daftar baru atau menyebut set lain secara eksplisit.

- Konfirmasi urutan pairing X dan wallet, dan jumlah yang mau di-submit, misalnya semua 15, bukan satu, atau bukan 300 wallet agen, kecuali user menyebutnya.

- Catat jawaban user tentang set itu di memory, supaya percakapan berikutnya tidak menanyakan hal yang sama lagi.

## Inspeksi form dulu

- Baca halaman proyek dengan `browser.open` dan, kalau perlu, inspeksi browser tanpa submit, tanpa sign in, tanpa connect wallet, dan tanpa aksi X.

- Catat nama proyek, chain, harga mint, supply, tanggal mint, deadline whitelist kalau ada, dan semua langkah form secara berurutan.

- Catat setiap field yang wajib, bukan hanya X dan wallet. Contoh field tambahan yang pernah muncul adalah link quote tweet, dua handle tag, dan dua jawaban bebas.

- Catat apakah wallet cukup diketik sebagai alamat 0x atau harus connect dan sign. Jangan connect wallet atau sign apa pun kecuali user menyetujuinya secara eksplisit untuk langkah itu.

- Catat apakah tugas X harus dilakukan di akun X user sendiri. Asisten tidak sign in ke X dan tidak melakukan follow, like, repost, reply, atau quote atas nama user.

## Probe curl

- Cari endpoint submit di JavaScript halaman atau di form action, bukan dari tebakan nama endpoint.

- Catat method, URL lengkap, header yang dipakai halaman itu, dan bentuk payload persisnya, termasuk field tugas atau flag yang diminta client.

- Uji endpoint itu dengan satu pasangan dulu sebagai probe. Kirim hanya pasangan probe, bukan semua pasangan sekaligus.

- Anggap probe tembus hanya kalau server mengembalikan respons sukses yang jelas, misalnya HTTP 200 dengan ok true, status created atau exists, atau nomor aplikasi. Respons itu harus dicatat per pasangan.

- Kalau probe tembus, hentikan tugas browser yang masih berjalan untuk proyek itu sebelum menjalankan curl paralel, supaya tidak ada submit ganda.

- Jalankan curl paralel untuk semua pasangan yang tersisa, dengan payload yang sama persis seperti probe, dan laporkan per pasangan.

- Kalau server mengembalikan exists untuk pasangan yang sudah pernah di-submit, laporkan sebagai sudah tercatat, bukan sebagai submit baru dan bukan sebagai gagal.

## Kalau curl tidak tembus

- Pakai jalur browser dan submit pasangan secara berurutan, dengan form di-reset atau di-reload di antara pasangan.

- Laporkan per pasangan dengan X, wallet, field tambahan yang dipakai, konfirmasi persis dari halaman, dan nomor aplikasi kalau ada.

- Perhatikan throttle sesi. Pada Erupt, setelah 10 submit berturut-turut dari satu sesi browser, tombol Submit menjadi tidak responsif tanpa pesan error, dan status check mengonfirmasi tidak ada aplikasi yang tercatat untuk pasangan itu.

- Kalau throttle seperti itu muncul, berhenti, jangan mengklik berulang kali, dan laporkan pasangan yang belum ke-submit. Coba lagi dari sesi browser baru setelah jeda, hanya kalau user menyuruh coba lagi.

- Kalau endpoint curl tidak ditemukan dari inspeksi browser, sampaikan itu sebagai keterbatasan alat inspeksi, bukan sebagai bukti bahwa endpoint tidak ada sama sekali.

## Data yang tidak boleh dikarang

- Jangan mengarang handle tag, link quote tweet, jawaban, atau klaim tugas X sudah selesai.

- Kalau field tambahan itu wajib dan user belum memberikannya, berhenti dan minta data itu, kecuali user sudah menyetujui template yang asisten buat.

- Kalau user menyuruh buat template jawaban, buat satu jawaban berbeda per pasangan, bukan satu teks yang disalin ke semua pasangan, dan patuhi batas karakter yang ditampilkan form, misalnya maksimal 600 karakter.

- Kalau user meminta bahasa tertentu untuk template, pakai bahasa itu untuk semua template di proyek itu.

- Kalau user menyetujui tag dari daftar sendiri secara acak, acak handle dari daftar itu dan catat pasangan tag yang benar-benar dipakai per akun, jangan laporkan tag yang diniatkan saja.

- Jangan mengklaim tugas X sudah diverifikasi server hanya karena tombol Verify di halaman lolos. Sampaikan bahwa verifikasi di halaman bisa bersifat client side dan verifikasi server belum tentu sama.

## Keamanan dan pelaporan

- Pakai hanya alamat wallet publik dan username X publik yang user berikan. Jangan pernah mencetak, mencatat, atau mengirim private key.

- Jangan pernah memakai private key untuk connect wallet, sign, atau mint dalam tugas join WL, kecuali user menyuruh langkah berbayar itu secara terpisah dan eksplisit.

- Laporkan total yang diterima, total yang belum, dan total yang gagal, dengan X dan nomor aplikasi atau statusnya, bukan hanya ringkasan semua berhasil.

- Sampaikan dengan jelas bahwa aplikasi yang diterima dan berstatus In Review bukan jaminan whitelist atau mint, kecuali proyek itu sendiri menyatakan sebaliknya.

- Simpan daftar pasangan yang dipakai, jawaban template kalau ada, dan hasil per pasangan di folder kerja proyek itu, supaya bisa dipakai ulang atau dilanjutkan kalau ada pasangan yang tertunda.

## Contoh yang sudah diverifikasi

- ELYNX di elynxnft.xyz/apply memakai curl dan tembus. Endpointnya adalah POST ke /api/apply dengan X, wallet, dan tasks follow, like, repost, tag semuanya true. Probe 10 pasangan berhasil, lalu 5 sisanya berhasil, total 15 dengan ID 52 sampai 76.

- Trippi di trippi.fun/#quest tidak menemukan endpoint curl dari inspeksi browser, jadi submit dilakukan lewat browser. Field tambahannya adalah link quote tweet dan Selected stop, dengan stop default Last Stop.

- Erupt di eruptdot.fun/whitelist tidak menemukan endpoint curl dari inspeksi browser, jadi submit dilakukan lewat browser. Field tambahannya adalah dua handle tag dan dua jawaban bebas. Sepuluh pertama diterima dengan ID 3544 sampai 3553, lalu throttle sesi menghentikan pasangan ke-11, dan lima sisanya diterima di sesi baru dengan ID 3633, 3637, 3643, 3652, dan 3655.

## Available connectors

- Use `muse.connector_search` to find other connectors and `muse.skill_search` to find related skills.

- If nothing here fits, look on the web for a maintained API or MCP server and set it up with [Custom Connectors and MCPs](/opt/muse/skills/custom-connector-setup/SKILL.md), or build a custom connector.

## Websites

- Gunakan halaman whitelist proyek itu sendiri sebagai sumber utama field, langkah, dan status form. Bawa URL yang user berikan, daftar pasangan yang sudah dikonfirmasi, dan batasan tidak connect wallet dan tidak aksi X ke tugas browser, dan verifikasi konfirmasi submit di halaman itu.
