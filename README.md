# defter

**Çift kayıtlı işlem defteri motoru — sıfırdan.** Bir bakiye tutan sistemin altındaki
katmanı hazır kütüphane kullanmadan yazma çalışması: write-ahead log, çökme sonrası
kurtarma, idempotency, eşzamanlılık kontrolü ve değişmez denetim izi.

> **A learning project.** A double-entry ledger engine written from scratch in Python with
> no external dependencies — WAL, crash recovery, idempotency keys, concurrency control and
> a hash-chained audit trail. Not production software; the point is to implement the
> mechanisms instead of importing them.

## Neden

Bir banka hesabından diğerine para geçerken şu soruların hepsi cevaplanmak zorunda:

- İşlemin ortasında süreç ölürse bakiye ne olur?
- Aynı istek ağ hatası yüzünden iki kere gelirse iki kere mi işlenir?
- İki eşzamanlı transfer aynı bakiyeyi okursa hangisi kazanır?
- Bir kaydın sonradan değiştirilmediğini nasıl kanıtlarsın?

Bu dört soru veritabanlarının ve ödeme sistemlerinin içinde çözülmüş durumda. Bu repo
onları **kullanmak** yerine **uygulamak** için var.

## Mimari

```
                    ┌──────────────┐
   istek ──────────▶│ idempotency  │  aynı anahtar → aynı sonuç, tek kez
                    └──────┬───────┘
                           ▼
                    ┌──────────────┐
                    │     lock     │  hesap bazında kilit
                    └──────┬───────┘
                           ▼
   ┌───────────────────────────────────────────┐
   │              transaction                  │
   │   Σ debit == Σ credit  (aksi hâlde red)   │
   └───────────────┬───────────────────────────┘
                   ▼
            ┌─────────────┐      ┌──────────────┐
            │  WAL (append)│─────▶│   recovery   │  çökme sonrası yeniden oynat
            └──────┬──────┘      └──────────────┘
                   ▼
            ┌─────────────┐
            │ audit chain │  her kayıt öncekinin hash'ini taşır
            └─────────────┘
```

## Yol haritası

Her hafta bir konu; her konunun sonunda çalışan kod, geçen test ve bir öğrenim notu.

| Hafta | Konu | Durum | Not |
|---|---|---|---|
| 1 | Çift kayıt çekirdeği, para gösterimi, değişmezlik | 🔨 sürüyor | [ogrenim/01-cift-kayit.md](ogrenim/01-cift-kayit.md) |
| 2 | Write-ahead log | ⬜ | |
| 3 | Çökme sonrası kurtarma + snapshot | ⬜ | |
| 4 | Idempotency anahtarı | ⬜ | |
| 5 | Eşzamanlılık: kilitleme ve race condition | ⬜ | |
| 6 | Invariant testi (Σborç = Σalacak her zaman) | ⬜ | |
| 7 | Hash zincirli denetim izi | ⬜ | |
| 8 | HTTP API + Docker | ⬜ | |

## Kurulum

```bash
python3 -m venv .venv
.venv/bin/pip install -e ".[dev]"
make test
make lint
```

## Tasarım kararları

Kararların gerekçesi kodda değil burada durur; sonradan "neden böyle yapmışım" sorusunun
cevabı olsun diye.

- **Tutarlar tamsayı (kuruş), float değil.** `0.1 + 0.2 != 0.3` olduğu için para hiçbir
  zaman kayan noktalı sayıyla tutulmaz. En küçük birim tamsayı olarak saklanır.
- **Kayıtlar değiştirilemez.** Yanlış bir kayıt silinmez; ters kayıt (reversal) atılır.
  Denetim izinin anlamı buna bağlı.
- **Bakiye saklanmaz, türetilir.** Hesap bakiyesi kayıtların toplamıdır. Saklanan bakiye
  ile kayıtlar çelişebilir; tek doğru kaynak kayıtlardır. (Hız gerektiğinde snapshot —
  3. hafta.)
- **Dış bağımlılık yok.** Çekirdek yalnızca standart kütüphaneyi kullanır; amaç mekanizmayı
  yazmak.

## Lisans

MIT
