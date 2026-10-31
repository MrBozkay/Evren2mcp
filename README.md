<div align="center">

<img src="docs/cover.png" alt="evren2mcp — EVREN Platformu için MCP sunucusu" width="880"/>

# evren2mcp

**EVREN Yapay Zeka Platformu'nu MCP istemcilerine bağlayan sunucu**

Claude Code · Codex CLI · OpenCode · Cursor · VS Code · Hermes Agent

[![CI](https://github.com/MrBozkay/Evren2mcp/actions/workflows/ci.yml/badge.svg)](https://github.com/MrBozkay/Evren2mcp/actions/workflows/ci.yml)
[![Lisans: MIT](https://img.shields.io/badge/lisans-MIT-blue.svg)](LICENSE)
[![Python](https://img.shields.io/badge/python-3.11%20%E2%80%93%203.14-blue.svg)](https://pypi.org/project/evren2mcp/)
[![MCP](https://img.shields.io/badge/MCP-stdio-4f8ef7.svg)](https://modelcontextprotocol.io)

*"Bu veri setinde küçük bir model eğit, kaça mal olur?"* — istemcin maliyeti
gösterir, onayını bekler, eğitimi başlatır, sonucu ve puan farkını getirir.

**Türkçe** · [English](README.en.md)

</div>

---

> [!IMPORTANT]
> **Resmî değildir.** Bağımsız bir topluluk projesidir; SSB ya da EVREN Platformu
> tarafından geliştirilmemiş, onaylanmamış ve desteklenmemektedir. Sunucu yalnız
> **senin kendi** API anahtarınla çalışır; her işlem hesabına bağlı EVREN
> Platformu Kullanıcı Sözleşmesi'ne tabidir. Sözleşmenin 4.9–4.11. maddeleri
> gereği yüklediğin veri setleri ve modeller platform tarafından model
> eğitiminde kullanılabilir.

## ✨ Neden?

Kredi harcayan **hiçbir iş**, sana gösterilmiş ve senin onayladığın bir teklif
olmadan başlamaz:

- 🧾 **Teklif kapısı** — eğitim öncesi tahmini + en kötü durum tutarı, 10 dk geçerli tek kullanımlık teklif
- 🛡️ **Çift tavan** — tek iş ≤ 5 CR, günlük ≤ 20 CR (platformun kendi kredi defterinden)
- 🔒 **Çift ücret koruması** — belirsiz POST asla tekrar gönderilmez
- 🇹🇷 **Yurt içi** — istem ve yanıt Türkiye'deki H200 kümesinde işlenir

## 📺 Örnek oturum

```text
Sen:     Drone setimde küçük bir modeli ince ayarla. Kaça mal olur?
İstemci: (list_datasets → estimate_training)
         Teklif T-51B553 — tahmini 0,54 CR, en kötü 1,82 CR.
         Bugün harcanan: 0 / 20 CR. Teklif 10 dk geçerli. Başlatayım mı?
Sen:     Evet.
İstemci: (start_training T-51B553) Eğitim başladı: veri hazırlanıyor → eğitiliyor.
         …
İstemci: (get_training) Tamamlandı. Gerçek maliyet 0,14 CR, mAP50 0,356, +15 puan.
```

Maliyet, metrik ve puan 06.10.2026'daki gerçek bir eğitimden; konuşma kısaltılmıştır.

## 📦 Kurulum

**Gerekenler:** [uv](https://docs.astral.sh/uv/) · herhangi bir MCP istemcisi ·
EVREN hesabı ve **Platform** API anahtarı (panel → API anahtarları → Platform)

Anahtarını yalnız senin okuyabileceğin bir dosyaya yaz, ör. `~/.evren2mcp/.env`:

```dotenv
EVREN_PLATFORM_API_KEY=platform-anahtarin
```

Sonra sunucuyu istemcine kaydet (dosyanın tam yolunu ver):

### Claude Code

```bash
claude mcp add evren --scope user -e EVREN_ENV_FILE=/tam/yol/.env \
  -- uvx --from git+https://github.com/MrBozkay/Evren2mcp evren2mcp
claude mcp list   # evren … ✔ Connected
```

### Codex CLI

```bash
codex mcp add evren --env EVREN_ENV_FILE=/tam/yol/.env \
  -- uvx --from git+https://github.com/MrBozkay/Evren2mcp evren2mcp
```

### OpenCode

```bash
opencode mcp add evren --env EVREN_ENV_FILE=/tam/yol/.env \
  -- uvx --from git+https://github.com/MrBozkay/Evren2mcp evren2mcp
```

<details>
<summary><b>opencode.jsonc</b> (eski şema; yeni sürümlerde <code>mcp.servers</code>)</summary>

```jsonc
{
  "mcp": {
    "evren": {
      "type": "local",
      "command": ["uvx", "--from", "git+https://github.com/MrBozkay/Evren2mcp", "evren2mcp"],
      "environment": { "EVREN_ENV_FILE": "/tam/yol/.env" }
    }
  }
}
```

</details>

### Cursor

`.cursor/mcp.json` (proje) ya da `~/.cursor/mcp.json` (global):

```json
{
  "mcpServers": {
    "evren": {
      "command": "uvx",
      "args": ["--from", "git+https://github.com/MrBozkay/Evren2mcp", "evren2mcp"],
      "env": { "EVREN_ENV_FILE": "/tam/yol/.env" }
    }
  }
}
```

### VS Code

`.vscode/mcp.json`:

```json
{
  "servers": {
    "evren": {
      "type": "stdio",
      "command": "uvx",
      "args": ["--from", "git+https://github.com/MrBozkay/Evren2mcp", "evren2mcp"],
      "env": { "EVREN_ENV_FILE": "/tam/yol/.env" }
    }
  }
}
```

### Hermes Agent

```bash
hermes mcp add evren --command uvx --env EVREN_ENV_FILE=/tam/yol/.env \
  --args --from git+https://github.com/MrBozkay/Evren2mcp evren2mcp
```

<details>
<summary><b>Yerel kopyadan</b> (geliştirme için)</summary>

```bash
git clone https://github.com/MrBozkay/Evren2mcp.git && cd evren2mcp
uv sync
cp .env.example .env   # EVREN_PLATFORM_API_KEY'i doldur
claude mcp add evren --scope user -e EVREN_ENV_FILE=<tam-yol>/.env \
  -- uv run --directory <tam-yol> evren2mcp
```

</details>

> [!TIP]
> Kurulumu denemek için istemcine *"evren diagnostics çağır"* de: ayarlarını
> anahtar maskeli gösterir. Windows'ta `C:/Users/sen/.evren2mcp/.env` gibi bir
> yol kullan.

> [!NOTE]
> `egit` ve `durum` akışları (prompt'lar) Claude Code'un `/` menüsünde görünür;
> diğer istemcilerde istem olarak elle yazılır.

## 🧰 Araçlar

| Araç | Ne yapar | Maliyet |
|---|---|:---:|
| `list_datasets` | Setler ve sürümleri | bedava |
| `search_universe` | Herkese açık set/model arama | bedava |
| `get_credits` | Bakiye ve son hareketler | bedava |
| `get_standing` | Puan, seviye, ilk 10 sınırına fark | bedava |
| `estimate_training` | Maliyet tahmini + 10 dk geçerli teklif | bedava |
| `start_training` | Teklifle eğitimi başlatır | **kredi** |
| `get_training` / `list_trainings` | Durum, metrik, hata açıklaması, puan farkı | bedava |
| `stop_training` | Süren eğitimi durdurur (geri alınamaz) | iade |
| `evaluate_model` | Modelin bir bölmedeki notunu ölçer | ~0,04 CR |
| `download_model` | Ağırlığı (`best.pt`) indirir | bedava |
| `check_dataset_zip` | Zip'i yerelde kontrol eder; hiçbir şey gönderilmez | bedava |
| `upload_dataset` | Zip'i yeni **gizli** sete yükler | bedava |
| `diagnostics` | Ayarlar (anahtar maskeli), istemci yetenekleri | bedava |

**Hazır akışlar:** `egit` — rehberli eğitim · `durum` — bakiye, puan, süren eğitimler
**Kaynaklar:** `evren://kurallar` — kredi kazanç kuralları · `evren://gunluk` — yerel eğitim günlüğü

## 🚦 Onay kapısı

```text
estimate_training ──► teklif (ayar parmak izi, tahmini/en kötü tutar, 10 dk)
        │
   sen onaylarsın
        │
start_training(teklif) ──► teklif geçerli, kullanılmamış? ──► tek iş tavanı ──► günlük tavan ──► POST
                                  hayır → ret                  (en kötü durum)   (platform defteri, UTC günü)
```

1. Teklif tek kullanımlıktır ve 10 dakikada düşer; sunucu yeniden başlayınca
   teklifler silinir, eski bir fiyatla iş başlamaz.
2. Tek iş tavanı platformun en kötü durum tahminine göre denetlenir.
3. Günlük tavan, bugünkü harcamayı platformun kendi kredi defterinden okur;
   açık teklifler ve yeni başlamış işler de 10 dk boyunca sayılır. Eşzamanlı
   iki çağrı tavanın altından birlikte geçemez.
4. Platform başlatma isteğine net yanıt vermezse istek **tekrar gönderilmez**
   (çift ücret riski); sunucu işi iş listesinde arar.
5. İstemcin izin katmanı varsa her araç çağrısında izin sorar (ör. Claude Code).

## ⚙️ Ayarlar

Ortam değişkenleri ya da `EVREN_ENV_FILE`'ın gösterdiği dosya (varsayılan:
çalışma klasöründeki `.env`). Süreç ortamındaki değer dosyadakini ezer.

| Değişken | Varsayılan |
|---|---|
| `EVREN_PLATFORM_API_KEY` | *(zorunlu)* |
| `EVREN_ENV_FILE` | `./.env` |
| `EVREN_PROJECT_ID` | ilk proje |
| `EVREN_MAX_JOB_CR` | `5` |
| `EVREN_MAX_DAILY_CR` | `20` |
| `EVREN_JOURNAL_PATH` | `~/.evren2mcp/journal.jsonl` |
| `EVREN_DOWNLOAD_DIR` | `~/.evren2mcp/models` |
| `EVREN_MAX_UPLOAD_MB` | `2048` |

## 🔐 Güvenlik ve gizlilik

- Anahtar yalnız HTTPS ile API alan adına gider; başka alan adındaki indirme
  bağlantılarına gitmez. Her çıktıda ve hata mesajında maskelenir.
- İstekler dakikada 20 ile ve patlama yumuşatmayla sınırlanır; platformun hız
  sınırı zorlanmaz (şartlar sınırı atlatmayı yasaklıyor).
- Eğitim günlüğü ve indirilen modeller senin makinende kalır.
- Açık bildirimler için [SECURITY.md](SECURITY.md).

## 🚫 Bilerek olmayanlar

Silme, herkese açık yapma, etiketleme, kullanım şartlarını onaylama, anahtar
yönetimi. Bunlar panelde kullanıcının elinde kalır.

## 🛠️ Geliştirme

Katkı rehberi: [CONTRIBUTING.md](CONTRIBUTING.md) · Tasarım:
[docs/tasarim.md](docs/tasarim.md) · Platformda ölçülen davranış:
[docs/canli-dogrulama.md](docs/canli-dogrulama.md)

```bash
git clone https://github.com/MrBozkay/Evren2mcp.git && cd evren2mcp
uv sync
uv run pytest -q        # 124 test
uv run ruff check src tests
uv run mypy
```

## 📄 Lisans

[MIT](LICENSE) — © 2026 MrBozkay

---

<div align="center">

**EVREN Yapay Zeka Platformu** · e-Devlet girişi: [evren.ssyz.org.tr](https://evren.ssyz.org.tr)

</div>
