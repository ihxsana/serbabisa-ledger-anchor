# Anchor Buku Besar SerbaBisa

Berkas di folder ini adalah **bukti publik** bahwa catatan uang SerbaBisa tidak
pernah diubah diam-diam.

Satu berkas: `ledger-anchor.json`. Isinya **hanya hash** dan agregat. Tidak ada
nama, email, nomor transaksi, atau nominal per pengguna di dalamnya.

## Kenapa ini ada

Buku besar kami memakai rantai hash: setiap mutasi menyimpan hash mutasi
sebelumnya. Itu menangkap penyusup dan bug. Tetapi rantai hash saja **tidak**
menangkap operator sistem — orang yang bisa mengubah data *dan* menghitung ulang
seluruh rantainya. Semua buktinya hidup di database yang sama, di bawah kunci
yang sama.

Menerbitkan rekap ke repo publik menutup celah itu. Begitu sebuah rekap terbit,
operator tidak bisa menariknya kembali. Kalau besok ia memalsukan data lalu
menghitung ulang rekapnya, rekap baru itu tidak akan cocok dengan yang sudah
terbit di sini.

## Apa yang dibuktikan, dan apa yang tidak

**Dibuktikan:** file ini memuat data yang sama persis dengan yang terbit
kemarin. Kalau berubah, riwayat commit git menunjukkan kapan dan apa.

**Tidak dibuktikan:** bahwa angka di dalamnya *benar*, atau bahwa rekap itu
mencakup semua transaksi. Untuk yang pertama kamu butuh audit proses bisnis;
untuk yang kedua kamu butuh akses ke sistem. Repo ini hanya membuktikan
**ketidakberubahan**.

Batas kejujuran yang sama berlaku untuk semua sistem: ini bukan blockchain, tidak
ada desentralisasi, dan tidak ada ribuan pihak yang ikut menjaga. Yang ada: bukti
yang keluar dari tangan operator, dan itu cukup untuk membuat pemalsuan
**terlihat**.

## Verifikasi tanpa mempercayai kami

Kamu tidak perlu menjalankan apa pun milik kami. Cukup satu perintah Python:

```bash
curl -s https://raw.githubusercontent.com/ihxsana/serbabisa-ledger-anchor/main/ledger-anchor.json -o a.json

python3 - <<'PY'
import json, hashlib
d = json.load(open('a.json'))
kanonik = json.dumps(d['days'], separators=(',', ':'), ensure_ascii=False)
hitung = hashlib.sha256(kanonik.encode()).hexdigest()
print('tertulis :', d['commitment'])
print('dihitung :', hitung)
print('COCOK' if hitung == d['commitment'] else 'RUSAK — berkas diubah setelah terbit')
PY
```

Kalau hasilnya `COCOK`, berkas ini memang seperti saat diterbitkan. Untuk memastikan
ia benar-benar rekap resmi hari itu, lihat riwayat commit-nya:

```bash
git clone https://github.com/ihxsana/serbabisa-ledger-anchor
cd serbabisa-ledger-anchor
git log --format='%H %ci %s' -- ledger-anchor.json
```

## Bentuk berkas

```json
{
  "schema": "serbabisa-ledger-anchor/v1",
  "algorithm": "sha256",
  "commitment": "<sha256 dari json kanonik `days`>",
  "days": [
    {
      "day": "2026-11-20",
      "first_entry_id": 2316,
      "last_entry_id": 2326,
      "entry_count": 11,
      "total_amount": 0,
      "merkle_root": "<akar Merkle semua mutasi hari itu>",
      "chain_hash": "<hash rantai terakhir hari itu>",
      "sealed_at": "2026-11-20 17:18:46"
    }
  ],
  "generated_at": "..."
}
```

`commitment` sengaja **tidak** menghitung `generated_at`. Sehingga mengekspor
ulang data yang sama menghasilkan nilai identik — `git diff` yang kosong berarti
"tidak ada yang berubah", bukan sekadar "waktunya beda".

`total_amount` selalu `0` karena buku besar memakai pencatatan ganda: setiap
peristiwa terdiri dari beberapa kaki yang saling meniadakan.

## Seberapa sering diperbarui

Setiap hari setelah rekap harian disegel (00:20 WIB). Kalau tidak ada commit
baru, artinya tidak ada mutasi baru — bukan berarti sistem mati.

## Kontak

Kesalahan, temuan, atau pertanyaan: buka issue di repo ini.
