# Snipe Token Launch

Gunakan skill ini setiap kali user minta snipe token launch (beli se-early mungkin, harga murah, presign). Ini metode yang benar — dibangun dari kegagalan snipe $MACHINE (2026-11-06) dan riset ke docs primer (Uniswap, QuickNode, Arbitrum playbook, Flashbots, DeepWiki sniper) + panduan sniper. Catatan: postingan X tidak bisa diambil saat riset (pencarian site:x.com tidak mengembalikan hasil), jadi sumbernya docs dan repo, bukan thread X.

## Aturan inti (kesalahan yang membunuh $MACHINE)

JANGAN PERNAH build, sign, atau quote buy di saat launch (T). Di T, kamu HANYA broadcast. Semua yang lain sudah siap SEBELUM launch:

- Quote di T itu sia-sia — saat tx dieksekusi, harga sudah kabur.
- Slippage 25% hanya tahan kenaikan harga 1.33x. Rumus: kenaikan maksimum yang selamat = `1/(1-t)`. Spike 12x butuh toleransi ~92% (minOut ≈ 8% dari quote harga launch).
- "Transaction creation failed" = RPC/client menolak SEBELUM chain — biasanya gas estimation terhadap kontrak yang belum live, atau RPC publik yang overload/rate-limit. Perbaikannya: pre-sign offline dengan gas hard-coded, broadcast raw bytes saja.

## Presign ladder (metode yang benar)

Jauh sebelum launch:

1. Ambil whitelist Merkle proof dan bake ke calldata. Verifikasi di fork. Kalau proof salah, buy revert apa pun gas/slippage-nya.
2. Dry-run calldata buy yang persis di fork (`anvil --fork-url`) → catat gas yang dipakai. `gasLimit` = dry-run × 1.2–1.3. JANGAN PERNAH `eth_estimateGas` di T (revert sebelum launch).
3. Sign OFFLINE (`viem`/`ethers` `signTransaction`) sebuah ladder varian dengan NONCE YANG SAMA:
   - Varian A: `minOut` = quote harga launch × 0.50 (slippage 50%) — hanya terisi kalau kamu early.
   - Varian B: `minOut` = quote harga launch × 0.10 (slippage 90%) — terisi sampai ~10x.
   - Varian C: `minOut` = 1 wei (isi di harga berapa pun) — HANYA dengan `amountIn` KECIL.
   - `deadline` = waktu launch yang diumumkan + 5–10 menit (jangan T + 20–30 detik; launch bisa telat).
   - `maxFeePerGas` ≥ 2× base fee saat ini.
4. Simpan signed blob di memori. BEKUKAN wallet: TIDAK ADA tx lain dari alamat sniper antara signing dan launch (nonce tidak boleh berubah). Pakai wallet khusus snipe, didanai hanya `amountIn + gas`.
5. Saat launch, tembak SEMUA varian serentak via `eth_sendRawTransaction` ke 3+ RPC. Hanya satu yang landing (yang pertama menghabiskan nonce, sisanya ditolak nonce-too-low).

## Strategi gas (tergantung chain — verifikasi di hari launch)

- Base / OP Stack: Priority Gas Auction (PGA) — sequencer mengurutkan berdasarkan priority fee. Gas MEMBELI prioritas. Set priority fee kelipatan dari p95/p99 tip normal (5–20×), dibatasi ekonomi trade.
- Arbitrum: historis FCFS (gas TIDAK membeli posisi) + Timeboost express lane. TAPI arXiv mencatat Arbitrum memilih mengganti Timeboost dengan PGA pada Sept 2026 — VERIFIKASI docs ordering chain target di hari launch, jangan percaya ringkasan.
- Jangan re-sign di T untuk update gas. Pre-sign dengan fee sudah tinggi. Hanya ganti (nonce sama, fee +≥10%) kalau tidak ter-sequence dalam ~2 blok yang diharapkan.

## Strategi RPC

- Benchmark dari mesin yang mengirim: round-trip `eth_blockNumber`, catat p50/p95/p99. Pilih p99 terendah.
- Koneksi hangat dan persisten: buka menit sebelum launch, heartbeat keep-alive. Jangan buka koneksi baru di T.
- 3+ endpoint paralel, provider DAN infrastruktur berbeda (node privat berbayar, provider berbayar kedua, endpoint sequencer/RPC resmi). Tembak blob yang SAMA ke semuanya serentak (`Promise.allSettled`).
- Pisah jalur baca dan tulis: WSS cepat untuk deteksi, HTTP untuk broadcast. Jangan biarkan poll `eth_call` yang lambat memblokir jalur kirim.
- JANGAN PERNAH pakai RPC publik/shared sebagai jalur kirim utama (429/5xx saat puncak, gagal >80%). Publik = cadangan saja.
- Kalau "Transaction creation failed" muncul: cek raw JSON-RPC error body, ganti endpoint, cek apakah tx hash ternyata sudah diterima. Jangan hantam endpoint yang sama. Jangan retry write membabi buta — cek receipt/nonce dulu.

## Deteksi launch (jangan pernah pakai jam dinding)

- JANGAN PERNAH pakai: waktu yang diumumkan (terbukti telat 19 detik), website/API proyek, DexScreener, Telegram/Discord. Launch ADALAH perubahan state on-chain — deteksi itu.
- Sinyal tercepat:
  1. WSS `eth_subscribe` `logs` di kontrak/router launch (`PairCreated`, `PoolCreated`, `Launched`, `TradingEnabled`, `LiquidityAdded`, `Transfer` dari pool).
  2. WSS `newHeads` + `eth_getBlockByNumber(block, true)`, scan blok baru untuk tx launch/likuiditas.
  3. Poll contract-state: `eth_call` tiap 100–200 ms untuk gate persis yang dicek `buy()` (`launched()` / `tradingEnabled()` / `startTime()` / `getReserves()`). Paling robust untuk router custom.
  4. Arbitrum sequencer feed (`wss://arb1-feed.arbitrum.io/feed`) — menampilkan tx yang diterima sequencer sebelum masuk blok.
- Jalankan 2 koneksi WSS independen (provider berbeda) + poll cadangan. Re-subscribe saat reconnect.
- Target: deteksi → broadcast < 50 ms kerja lokal. Langkah apa pun yang I/O setelah trigger (ambil proof, estimasi gas, ambil nonce, buka koneksi) menambah 50–500 ms dan biasanya membedakan blok 0/1 vs fill 12x.

## Slippage (matematikanya)

- Kenaikan harga maksimum yang selamat = `1/(1-t)`. 25% → 1.33x. 50% → 2x. 75% → 4x. 90% → 10x. 92% → 12.5x. 99% → 100x.
- Pakai ladder (di atas) — jangan naikkan slippage lalu retry setelah spike (tiap retry adalah round trip baru di harga yang lebih buruk).
- Kecilkan buy saat toleransi naik. Buy toleransi 90% harus jauh lebih kecil dari buy toleransi 25%.
- Hitung pajak/fee token di `minOut` (token dengan buy tax 5% tidak bisa terisi di slippage 3%).
- Slippage lebar menyerahkan nilai ke MEV/sandwich — varian `minOut≈0` harus KECIL.

## Sizing (pool kecil = harga bergerak brutal)

- Batasi buy ≤1–2% dari reserve base-asset awal pool. Di 10% kamu sudah membayar impact ~9% sebelum orang lain trade; di 50% ~33%.
- Spike 12x berarti ~70% dari reserve token awal kebeli (x·y=k: menghapus fraksi f dari reserve menaikkan harga spot ~`1/(1-f)²`). Di pool supply 4.444, itu ~3.100 token — beberapa buy saja.
- Harapkan harga launch hilang di blok 1. Kecuali kamu di blok 0/1, kamu membeli momentum trade di kelipatan besar dari harga launch, bukan entry harga launch.
- JANGAN kejar setelah spike dengan size lebih besar / slippage lebih lebar. Itu membeli puncak.
- Cek exit sebelum entry — pool tipis yang sama bergerak sama brutalnya ke bawah saat kamu jual.

## Mode kegagalan (diagnosis + pencegahan)

| Kegagalan | Penyebab | Pencegahan |
|---|---|---|
| SlippageExceeded / INSUFFICIENT_OUTPUT_AMOUNT | Output < amountOutMin; harga bergerak melewati toleransi | Ladder dengan varian minOut ≈ 0; size kecil; masuk blok 0–1 |
| Not launched / trading not enabled | Buy broadcast sebelum flag launch flip (trigger pakai timer) | Gate broadcast pada state on-chain, bukan waktu |
| Expired / Transaction too old | Deadline lewat sebelum inclusion | deadline = T + 5–10 menit; maxFeePerGas tinggi; RPC cepat |
| RPC overload / Transaction creation failed (-32003) | RPC publik 429/5xx; estimasi gas gagal di kontrak belum live; node menolak tx saat pembuatan | Pre-sign offline, gas hard-coded, broadcast raw bytes ke 3+ RPC berbayar paralel |
| Nonce too low / gap / replacement underpriced | Tx lain dari wallet yang sama menghabiskan nonce; bump fee < minimum (~10%) | Wallet khusus, nonce beku; replacement hanya nonce sama +≥10% fee; cek eth_getTransactionCount("pending") sebelum retry |
| Insufficient liquidity / K-invariant failure | Reserve pool terlalu kecil untuk size buy, atau pool belum ada | Size ≤1–2% reserve; verifikasi pool ada dan reserve non-zero di blok yang sama via poll deteksi |
| Out of gas | gasLimit dari estimasi basi; verifikasi Merkle proof / logika router custom lebih mahal dari swap biasa | Hard-code dari fork dry-run calldata persis + 20–30% headroom |
| Wrong proof / not whitelisted | Proof untuk alamat/jumlah yang salah, leaf hashing salah, atau root berubah | Ambil proof di awal, verifikasi di fork, eth_call view function router pre-launch kalau ada |
| Stale / dropped detection | WSS putus dan subscription hilang; poller rate-limit | Dual WSS + poll cadangan, auto-resubscribe, alarm staleness |
| Sequencer outage / chain halt | Sequencer terpusat down | Ketahui fallback L1 delayed-inbox / force-inclusion (lambat, bukan alat snipe) |
| Sandwich / frontrun fill di harga terburuk | minOut lebar + tx terlihat (L1) atau tx yang diurutkan lebih dulu (L2 PGA) | Size kecil di varian toleransi lebar; di L1 pakai private bundle/Flashbots Protect |

## Checklist pre-launch

- [ ] Aturan ordering chain target dikonfirmasi di docs terbaru (PGA vs FCFS vs Timeboost/PGA).
- [ ] Router ABI, signature `buy`, alamat token + pool, view function launch-flag, Merkle root/proof diambil dan diverifikasi di fork.
- [ ] Fork dry-run calldata persis → gas tercatat; gasLimit = dry-run × 1.2–1.3.
- [ ] Ladder di-sign offline (nonce sama, 3 varian minOut, deadline T+5–10 menit, maxFeePerGas ≥2× base fee); blob di memori; wallet beku.
- [ ] 3+ RPC di-benchmark (p95/p99 dari mesin pengirim), koneksi keep-alive hangat; 2 feed deteksi WSS ter-subscribe; poll cadangan jalan.
- [ ] Trigger deteksi diuji (ukur latensi trigger→broadcast; target <50 ms kerja lokal).
- [ ] Buy di-size ≤1–2% dari expected initial base reserve; varian toleransi lebar di-size paling kecil.
- [ ] Tx abort/cancel (nonce sama, 0-value, fee lebih tinggi) disiapkan tapi tidak dikirim.
- [ ] Setelah launch: cek receipt + nonce SEBELUM retry apa pun; log nomor blok, gas dibayar, harga realisasi vs harga launch.

## Kegagalan $MACHINE (2026-11-06) — yang salah

- Launch telat 19 detik (23:00:19, bukan 23:00:00) — trigger jam dinding gagal.
- Harga spike ~12x dalam <1 menit (0.014125 → 0.176 HYPE/MACHINE) — ~70% dari reserve 4.444 kebeli.
- Percobaan 1–3 revert SlippageExceeded di slippage 10–25% — 25% hanya tahan 1.33x.
- Percobaan 4–6 gagal di level RPC ("Transaction creation failed") — RPC publik Montra, tx tidak pernah ter-broadcast.
- Script melakukan quote di T (staticCall) lalu sign + kirim — round trip quote→eksekusi lebih lambat dari pergerakan harga.
- Tidak ada presign ladder, tidak ada broadcast multi-RPC, tidak ada priority gas.
- 0 MACHINE kebeli, 0.2157 HYPE aman (tidak ada dana hilang).
- Perbaikan untuk lain kali: presign ladder + deteksi berbasis state + broadcast raw multi-RPC + varian toleransi lebar ber-size kecil.

## Keamanan dan pelaporan

- Hanya alamat wallet publik. Jangan pernah mencetak, mencatat, atau mengirim private key.
- Tidak ada buy tanpa perintah eksplisit user yang menyebut token, jumlah, chain, dan wallet.
- Jangan pernah klaim snipe pasti berhasil — laporkan hasil fill (atau gagal) dengan jujur beserta harga realisasi.

## Repo referensi (terverifikasi via GitHub API, 2026-11-06)

Repo open-source yang terkenal dan sering dipakai. Dicek langsung — bukan klaim dari artikel.

1. **Supercycled/cake_sniper** — 1.535 stars, 664 forks, Go, "EVM frontrunning tool", update 2026-10-16. Repo EVM frontrunning/liquidity-snipe yang paling banyak di-star di GitHub. Teknik yang dipakai skill ini: (a) akses mempool lewat GETH client sendiri; (b) "bee books" — swarm akun + smart contract sniper on-chain, membanjiri mempool dengan banyak percobaan (semuanya revert kecuali satu) untuk berebut posisi tepat setelah liquidity addition di blok yang sama; (c) komponen Clogger yang dijalankan lebih dulu. Konfirmasi: di EVM publik, balapan adalah mempool + urutan blok, bukan cuma kecepatan kirim. https://github.com/Supercycled/cake_sniper

2. **doraemon8/fourmeme-sniper** (dulu vladmeer/fourmeme-sniper) — BSC, TypeScript, sniper four.meme + PancakeSwap. Repo ini sumber tabel slippage dan price impact di skill ini (via DeepWiki). Fitur: deteksi real-time dari factory four.meme, mempool monitoring untuk 0-block sniping, MEV front-run/back-run opsional, auto-buy instan dengan retry + exponential backoff, dynamic gas pricing, slippage configurable (contoh `SLIPPAGE_BPS=100`, `GAS_LIMIT=500000`). Best practice dari repo ini: pakai premium RPC (QuickNode, Ankr) + WSS untuk akses mempool, mulai dengan buy kecil di testnet, pakai banyak wallet untuk nonce/concurrency. Catatan jujur: GitHub API melaporkan 0 stars (repo baru pindah nama dari vladmeer ke doraemon8), tapi ini repo kanonis four.meme sniper yang didokumentasikan DeepWiki. https://github.com/doraemon8/fourmeme-sniper

3. **mortdeus/solana-copy-sniper-mev-trading-bot** — 4.515 stars, Rust, update 2026-11-06. Repo sniper yang paling banyak di-star secara keseluruhan, tapi Solana (PumpFun, Bonk, Raydium, Hyperliquid). Teknik yang relevan: gRPC shred stream (feed tercepat), bundler, off-chain signing untuk kecepatan, custom sell logic. Prinsipnya (feed latensi terendah + sign offline + broadcast via jalur tercepat) sama dengan yang dipakai skill ini, walau implementasi Solana-nya berbeda. https://github.com/mortdeus/solana-copy-sniper-mev-trading-bot

4. **Patokan komersial (closed-source, bukan repo)**: Banana Gun (88% eksekusi first-block di Ethereum, pangsa pasar puncak 73–94%, 5 chain) dan Unibot (volume sniping $895M+, klaim 6× lebih cepat dari Uniswap). Ini standar industri untuk "bagus" — tapi kodenya tertutup, jadi tidak bisa dijadikan referensi implementasi. Disebut di sini sebagai patokan, bukan sumber teknik.

## Sumber riset

- Uniswap docs (swaps, amountOutMinimum, deadline): https://developers.uniswap.org/docs/get-started/concepts/traders/swaps
- QuickNode — sign tx lokal: https://www.quicknode.com/guides/ethereum-development/transactions/how-to-sign-ethereum-transactions-locally
- Arbitrum Private Bundle Playbook 2026: https://ai-frb.com/blog/arbitrum-playbook-private-bundles
- Base PGA / Flashblocks (arXiv): https://arxiv.org/html/2610.03922v1
- Fourmeme sniper — Slippage and Price Impact (DeepWiki): https://deepwiki.com/doraemon8/fourmeme-sniper/7.4-slippage-and-price-impact
- Mobula — Detecting Snipers & Bundlers: https://docs.mobula.io/almanac/detecting-snipers-bundlers
- OnFinality — RPC timeout/retry: https://onfinality.io/en/learn/ethereum-rpc-timeout
- Laporan riset lengkap: `~/workspace/research_notes/token-launch-snipe-method-20261106-1507/report.md`

## Available connectors

- Use `muse.connector_search` to find other connectors and `muse.skill_search` to find related skills.

- If nothing here fits, look on the web for a maintained API or MCP server and set it up with [Custom Connectors and MCPs](/opt/muse/skills/custom-connector-setup/SKILL.md), or build a custom connector.
