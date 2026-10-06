# DDE D62M57A0 (E70 3.0sd): корисні job-и (Toolset32, `d_motor.grp`)

## Тільки читання: безпечно
| Job | Що показує | Норма на холодному незаведеному двигуні |
|---|---|---|
| `ident` | ідентифікація блоку | — |
| `fs_lesen`, `fs_lesen_detail`, `is_lesen`, `is_lesen_detail` | помилки / info-пам'ять | — |
| `status_ubatt` | напруга бортмережі | ≥12.4 В |
| `status_atmosphaerendruck` | атмосферний тиск | ~950–1030 гПа |
| `status_ladedruck_ist` / `_soll` | наддув фактичний / заданий | ist ≈ атмосферному |
| `status_luftmasse_ist` / `_soll` | маса повітря | ≈ 0 |
| `status_raildruck_ist` / `_soll` | тиск у рейці | ≈ 0 |
| `status_kuehlmitteltemperatur`, `status_ansauglufttemperatur`, `status_ladelufttemperatur`, `status_kraftstofftemperatur`, `status_umgebungstemperatur` | температури | усі ≈ температурі повітря |
| `status_differenzdruck_csf` | перепад тиску на DPF | ≈ 0 |
| `status_abgastemperatur_csf` / `_kat` | температура вихлопу | ≈ температурі повітря |
| `status_laufunruhe_llr_menge` / `_drehzahl` | нерівномірність / корекція подачі по циліндрах | (на ХХ) ±2–3 mg |
| `status_regeneration_csf`, `status_restlaufstrecke_csf`, `status_partikelfilter_verbaut` | стан DPF | — |
| `status_kilometerstand`, `status_betriebsstundenzaehler` | пробіг / мотогодини | — |
| `abgleich_ima_lesen` | IMA-коди форсунок | — |
| `status_glf`, `status_glf2` | ймовірно стан системи розжарення (перевірити) | — |

## НЕ запускати без потреби: змінюють дані
`fs_loeschen`, `*_loeschen`, `lernwerte_ruecksetzen`, `cbs_reset`, `steuern_*`, `*_prog*`, `*_schreiben`, `abgleich_verstellen*`, `steuergeraete_reset`, `steuern_eep_defekt_reset`
