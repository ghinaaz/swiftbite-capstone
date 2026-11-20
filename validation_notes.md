# Technical Evidence - SwiftBite Capstone
Variant ID : SB-P19
Peserta    : AzizahGhina

## 1. Input inventory
- PM2   : `swiftbite_pm2_integrated_order_level.csv` | rows=335 | unique order_id=335 | null=0 | dup=0
- Reviews: `swiftbite_reviews.csv` | rows=192 | unique order_id=192 | dup=0
- Kolom terdeteksi untuk analisis: {"order_ts": "order_timestamp", "restaurant": "restaurant_name", "success_amt": "successful_payment_amount", "refund_amt": "refunded_payment_amount", "delivery_status": "delivery_status", "is_late": "is_late", "est_ts": "promised_delivery_minutes", "act_ts": "delivered_at"}

## 2. Join & grain (LEFT JOIN PM2 -> Reviews on order_id, validate one_to_one)
- PM2 rows               : 335
- Capstone rows          : 335
- Unique order_id        : 335
- Matched review order_id: 192
- Unmatched Reviews      : 0
- Duplicate order_id     : 0
- Order No Review        : 143 | Reviewed - No Text: 11 | Reviewed - With Text: 181
- Direct PII field count : 0
- Total nominal sebelum  : {'subtotal_amount': np.int64(40199000), 'discount_amount': np.int64(2225000), 'total_amount': np.int64(42218000), 'successful_payment_amount': np.int64(38845000), 'refunded_payment_amount': np.int64(1684000), 'net_payment_amount': np.int64(37161000)}
- Total nominal sesudah  : {'subtotal_amount': np.int64(40199000), 'discount_amount': np.int64(2225000), 'total_amount': np.int64(42218000), 'successful_payment_amount': np.int64(38845000), 'refunded_payment_amount': np.int64(1684000), 'net_payment_amount': np.int64(37161000)}
- Final judgment         : PASS

## 3. Definisi KPI
- **Total Orders**: COUNT DISTINCT order_id pada populasi terpilih
- **Net Payment Value**: SUM(successful_payment_amount) - SUM(refunded_payment_amount); failed attempts tidak menambah success
- **Review Coverage**: (order dengan review / seluruh order) x 100%; No Review masuk denominator
- **Late Delivery Rate**: (delivered order terlambat / delivered order dengan informasi waktu valid) x 100%; cancelled/incomplete bukan 'tepat waktu'
- **AOV (opsional)**: net payment / order dengan success_amt > 0

KPI tanpa filter: Total Orders=335, Net Payment=37,161,000.00,
Review Coverage=57.31%, Late Delivery Rate=31.19% (92/295).

## 4. Sentimen
- Metode: pemetaan manual teks review unik (dibaca satu per satu) ke Positive/Neutral/Negative; kamus rule-based dengan negasi sebagai cadangan. Label dari isi teks, bukan rating. Kebijakan: kalimat bernada campuran (pujian + keluhan) dilabeli ... (ISI sesuai keputusanmu, misalnya Negative atau Neutral)
- Hasil: {'Positive': 79, 'Neutral': 63, 'Negative': 39, 'No Text': 11}
- Verifikasi manual: n=64, kesesuaian=0.8125
- Keterbatasan: kamus tidak menangkap sarkasme/konteks; label Unknown dipakai bila teks ada tetapi tidak meyakinkan.

## 5. Uji dashboard (logika filter vs pandas)
- tanpa filter: rows=335 | match_pandas=True
- bulan=2026-03: rows=129 | match_pandas=True
- restoran=SwiftBite Partner 19: rows=15 | match_pandas=True
- bulan=2026-03 & restoran=SwiftBite Partner 19: rows=6 | match_pandas=True
- sentimen=Negative: rows=39 | match_pandas=True
- kombinasi nol baris: rows=0 | match_pandas=True

Uji manual di browser (lengkapi sendiri): [ ] filter bulan  [ ] filter zona  [ ] filter sentimen  [ ] kombinasi  [ ] nol baris -> N/A  [ ] HTML dibuka dari folder final

## 6. QA akhir
- rows == PM2 rows: PASS
- unique order_id == rows: PASS
- order_id null == 0: PASS
- No PII / raw customer_id: PASS
- No Review != No Text: PASS
- No Review -> sentiment kosong: PASS
- label sentimen valid: PASS
- net payment total unchanged: PASS

## 7. Catatan perubahan / keputusan (termasuk yang dibantu AI Agent)
- Pemeriksaan format key: tidak ada spasi tersembunyi di PM2 maupun Reviews; tidak ada transformasi pada key.
- Label payment_method_final diseragamkan di memori (Bank Transfer->Bank_Transfer, E Wallet->E_Wallet), 2 baris terdampak. File sumber tidak diubah.
- 181 review diberi label lewat tabel pemetaan 31 teks unik (dibaca manual satu per satu); kamus rule-based hanya cadangan.
- 64 label sentimen dikoreksi manual dari sampel verifikasi.
- Label payment_method_final diseragamkan di memori (Bank Transfer->Bank_Transfer, E Wallet->E_Wallet). File sumber tidak diubah.

## 8. Catatan tambahan
- Review tidak sejalan dengan rating: 22 dari 181 review bertext (12.2%).
- Review bertext: 181 | teks unik: 31 (teks berulang antar order; analisis teks terbatas pada pola ini).
- Late Delivery Rate dihitung dari kolom is_late bawaan PM2 pada delivered order valid; order cancelled tidak dihitung tepat waktu.
- Label payment_method_final diseragamkan di memori (Bank Transfer->Bank_Transfer, E Wallet->E_Wallet); file sumber tidak diubah.
- MIN_N = 10 untuk late rate per restoran; delivered valid per restoran 12-13, sampel kecil sehingga dipakai hati-hati.
- Dashboard final: 5 KPI, 5 grafik (satu per business question), filter bulan, zona pelanggan, dan sentimen/status review.
