# Boundaries — Türkiye İdari Sınırları (Interaktif Harita + İstatistik Rehberi)

Bu repo iki şey yapar:

1. Çevre, Şehircilik ve İklim Değişikliği Bakanlığı'nın **TUCBS Public API** WMS servisinden gelen il / ilçe / mahalle sınırlarını interaktif Leaflet haritasında gösterir.
2. Bu sınırların üzerine **istatistik veri bağlayıp** choropleth ve cartogram üretmek isteyenler için R rehberi sunar.

> Canlı site: `https://<kullanici-adin>.github.io/<repo-adi>/`

## İçerik

- `index.Rmd` — WMS haritası (ana sayfa)
- `guide.Rmd` — İstatistik veriyle choropleth/cartogram üretme rehberi
- `about.Rmd` — Proje ve veri kaynağı hakkında
- `_site.yml` — Site yapılandırması; çıktı `docs/`'a yazılır
- `.github/workflows/render.yml` — Push'ta otomatik render + Pages deploy
- `data/` — Örnek istatistik veri için yer tutucu

## Lokalde çalıştırmak

R / RStudio'da:

```r
# Bir kerelik bağımlılıklar
install.packages(c("rmarkdown", "leaflet", "htmlwidgets", "pacman",
                   "sf", "dplyr", "geodata", "tmap", "cartogram"))

# Site'i render et
rmarkdown::render_site()
```

Çıktı `docs/` klasörüne yazılır; `docs/index.html`'i tarayıcıda aç.

## GitHub Pages'te yayınlamak

### Yol A — Lokal render + push (basit)

1. `rmarkdown::render_site()` çalıştır.
2. `docs/` klasörünü commit'le.
3. GitHub → **Settings → Pages**:
   - Source: **Deploy from a branch**
   - Branch: **main** — Folder: **/docs**
4. ~1 dakika içinde URL'in yayına alınır.

### Yol B — GitHub Actions (otomatik, önerilen)

Repo'da `.github/workflows/render.yml` mevcut. İlk kurulumu yapmak için:

1. `Settings → Pages` → Source: **GitHub Actions**.
2. `Settings → Actions → General → Workflow permissions`: **Read and write**.
3. `main`'e bir commit push'la. Action çalışır, R kurulur, site render edilir, Pages'e deploy olur.

İlk çalıştırma R paketlerini kurduğu için biraz uzun sürer (5–10 dk); sonraki run'lar cache sayesinde hızlıdır.

## İstatistik haritası — hızlı önizleme

Tam rehber için **`guide.Rmd`** (sitede "Rehber: İstatistik Haritası") sayfasına bak. Çekirdek desen şudur:

```r
library(sf); library(dplyr); library(leaflet); library(geodata)

# 1) Vektör il sınırları (GADM)
tr <- geodata::gadm("TUR", level = 1, path = "data") |> sf::st_as_sf()

# 2) İstatistik veri
nufus <- read.csv("data/il_nufus_2023.csv")  # il, nufus

# 3) Birleştir (isimleri normalleştirerek)
harita <- tr |> left_join(nufus, by = c("NAME_1" = "il"))

# 4) Renklendir
pal <- colorBin("YlOrRd", harita$nufus, bins = 6)
leaflet(harita) |>
  addProviderTiles(providers$CartoDB.Positron) |>
  addPolygons(fillColor = ~pal(nufus), weight = 1,
              color = "white", fillOpacity = 0.8) |>
  addLegend(pal = pal, values = ~nufus, title = "Nüfus")
```

> **Önemli:** WMS servisi raster (PNG) gönderir, vektör değil — yani WMS katmanını
> renge boyayamazsın. İstatistik haritası için ayrı bir **vektör** kaynak gerek
> (örn. GADM, TÜİK shapefile). WMS'i istersen altlık/referans olarak ekleyebilirsin.
> Detay: `guide.Rmd` → "WMS vs Vektör".

## Veri Kaynağı

WMS endpoint: `https://tucbs-public-api.csb.gov.tr/trk_afad_tdth_wms`

| Katman | İçerik | Min ölçek |
|---|---|---|
| 67 | İl sınırı | 1 : 9.250.000 |
| 68 | İlçe sınırı | 1 : 2.000.000 |
| 69 | Mahalle / köy sınırı | 1 : 300.000 |

## Lisans

Kod: MIT. Sınır verileri: veri sahibi kurumun (CSB) koşullarına tabi.
