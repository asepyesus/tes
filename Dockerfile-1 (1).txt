FROM python:3.11-slim

# Install system deps untuk Playwright Chromium
# Note: skip playwright install-deps karena beberapa package tidak ada di Debian Trixie
RUN apt-get update && apt-get install -y \
    wget curl ca-certificates \
    libnss3 libnspr4 \
    libatk1.0-0 libatk-bridge2.0-0 \
    libcups2 libdrm2 \
    libxkbcommon0 libxcomposite1 libxdamage1 \
    libxfixes3 libxrandr2 libgbm1 \
    libasound2 libpango-1.0-0 libpangocairo-1.0-0 \
    libx11-6 libxcb1 libxext6 libxss1 \
    libglib2.0-0 libdbus-1-3 \
    fonts-freefont-ttf fonts-dejavu-core \
    && rm -rf /var/lib/apt/lists/*

WORKDIR /app

COPY requirements.txt .
RUN pip install --no-cache-dir --upgrade pip && \
    pip install --no-cache-dir -r requirements.txt

# Install hanya browser Chromium, TANPA install-deps (sudah manual di atas)
ENV PLAYWRIGHT_BROWSERS_PATH=/ms-playwright
RUN playwright install chromium

COPY app.py .

EXPOSE 8000

CMD ["gunicorn", "app:app", "--bind", "0.0.0.0:8000", "--workers", "1", "--timeout", "120"]
