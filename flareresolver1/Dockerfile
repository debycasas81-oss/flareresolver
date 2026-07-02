FROM python:3.13-slim-bookworm AS builder

RUN apt-get update \
    && apt-get install -y --no-install-recommends equivs \
    && equivs-control libgl1-mesa-dri \
    && printf 'Section: misc\nPriority: optional\nStandards-Version: 3.9.2\nPackage: libgl1-mesa-dri\nVersion: 99.0.0\nDescription: Dummy package for libgl1-mesa-dri\n' >> libgl1-mesa-dri \
    && equivs-build libgl1-mesa-dri \
    && mv libgl1-mesa-dri_*.deb /libgl1-mesa-dri.deb \
    && equivs-control adwaita-icon-theme \
    && printf 'Section: misc\nPriority: optional\nStandards-Version: 3.9.2\nPackage: adwaita-icon-theme\nVersion: 99.0.0\nDescription: Dummy package for adwaita-icon-theme\n' >> adwaita-icon-theme \
    && equivs-build adwaita-icon-theme \
    && mv adwaita-icon-theme_*.deb /adwaita-icon-theme.deb

FROM python:3.13-slim-bookworm

COPY --from=builder /*.deb /

WORKDIR /app

RUN dpkg -i /libgl1-mesa-dri.deb \
    && dpkg -i /adwaita-icon-theme.deb \
    && apt-get update \
    && apt-get install -y --no-install-recommends chromium chromium-common chromium-driver xvfb dumb-init \
        procps curl xauth \
    && rm -rf /var/lib/apt/lists/* \
    && rm -f /usr/lib/x86_64-linux-gnu/libmfxhw* \
    && rm -f /usr/lib/x86_64-linux-gnu/mfx/* \
    && mv /usr/bin/chromedriver /app/chromedriver || true

COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

RUN mkdir -p "/root/.config/chromium/Crash Reports/pending"

COPY src/ .
COPY package.json ../

EXPOSE 8191

ENTRYPOINT ["/usr/bin/dumb-init", "--"]
CMD ["python", "-u", "/app/flaresolverr.py"]
