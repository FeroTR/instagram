# .github/workflows/daily_post.yml
# Her gün 09:00 Türkiye saatinde (06:00 UTC) çalışır.

name: Instagram Günlük Paylaşım

on:
  schedule:
    - cron: "0 6 * * *"   # 06:00 UTC = 09:00 Türkiye (UTC+3)
  workflow_dispatch:        # Manuel tetikleme (test için)

jobs:
  post:
    runs-on: ubuntu-latest

    steps:
      # ── 1. Repo'yu çek ──────────────────────────────
      - name: Checkout
        uses: actions/checkout@v4

      # ── 2. Python kur ───────────────────────────────
      - name: Python 3.12 kur
        uses: actions/setup-python@v5
        with:
          python-version: "3.12"
          cache: "pip"

      # ── 3. Bağımlılıkları yükle ─────────────────────
      - name: Bağımlılıkları yükle
        run: pip install -r requirements.txt

      # ── 4. Durum dosyasını cache'den geri yükle ─────
      #   (hangi resimde kaldığımızı hatırlıyoruz)
      - name: Resim durumunu yükle
        uses: actions/cache@v4
        with:
          path: last_image_index.json
          key: image-state-${{ runner.os }}
          restore-keys: image-state-

      # ── 5. Botu çalıştır ────────────────────────────
      - name: Botu çalıştır
        env:
          ANTHROPIC_API_KEY:   ${{ secrets.ANTHROPIC_API_KEY }}
          IG_USER_ID:          ${{ secrets.IG_USER_ID }}
          IG_ACCESS_TOKEN:     ${{ secrets.IG_ACCESS_TOKEN }}
          GDRIVE_FOLDER_ID:    ${{ secrets.GDRIVE_FOLDER_ID }}
          GDRIVE_CREDENTIALS:  ${{ secrets.GDRIVE_CREDENTIALS }}
          IMGBB_API_KEY:       ${{ secrets.IMGBB_API_KEY }}
          # DRY_RUN: "true"    # Test için bu satırı aç
        run: python src/bot.py

      # ── 6. Güncellenen durum dosyasını kaydet ───────
      - name: Resim durumunu kaydet
        if: always()
        uses: actions/cache@v4
        with:
          path: last_image_index.json
          key: image-state-${{ runner.os }}-${{ github.run_id }}
