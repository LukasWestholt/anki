# This Dockerfile uses three stages.
#   1. Compile anki (and dependencies) and build python wheels.
#   2. Create a virtual environment containing anki and its dependencies.
#   3. Create a final image that only includes anki's virtual environment and required
#      system packages.

ARG PYTHON_VERSION="3.11"
ARG DEBIAN_FRONTEND="noninteractive"

# Build anki.
FROM python:$PYTHON_VERSION AS build

RUN apt-get update && \
    apt-get install -y --no-install-recommends ninja-build rsync && \
    apt-get clean

WORKDIR /opt/anki
COPY . .

ENV CARGO_HOME=/state/rust/cargo
ENV RUSTUP_HOME=/state/rust/rustup
ENV PATH=/state/rust/cargo/bin:$PATH
RUN curl https://sh.rustup.rs -sSf | sh -s -- -y --no-modify-path --default-toolchain none && \
    rustup show
# Build python wheels.
RUN ./tools/build

# Install pre-compiled Anki.
FROM python:${PYTHON_VERSION}-slim as installer
WORKDIR /opt/anki/
COPY --from=build /opt/anki/out/wheels/ wheels/

# Use virtual environment.
# Python is here: /usr/local/bin/python v3.11.9
RUN python -m venv --system-site-packages venv \
    && ./venv/bin/python -m pip install --no-cache-dir setuptools wheel \
    && ./venv/bin/python -m pip install --no-cache-dir /opt/anki/wheels/*.whl

# We use another build stage here so we don't include the wheels in the final image.
FROM debian as final
COPY --from=installer /opt/anki/venv /opt/anki/venv
ENV PATH=/opt/anki/venv/bin:$PATH
# Install run-time dependencies.
RUN apt-get update \
    && apt-get install --yes --no-install-recommends \
    # Installs Python here: /usr/bin/python3 v3.11.2
    python3-pyqt6.qtquick python3-pyqt6.qtwebengine python3-pyqt6.qtmultimedia \
    libasound2 \
    libdbus-1-3 \
    libfontconfig1 \
    libfreetype6 \
    libgl1 \
    libglib2.0-0 \
    libnss3 \
    libxcb-icccm4 \
    libxcb-image0 \
    libxcb-keysyms1 \
    libxcb-randr0 \
    libxcb-render-util0 \
    libxcb-shape0 \
    libxcb-xinerama0 \
    libxcb-xkb1 \
    libxcomposite1 \
    libxcursor1 \
    libxi6 \
    libxkbcommon0 \
    libxkbcommon-x11-0 \
    libxrandr2 \
    libxrender1 \
    libxtst6 \
    && rm -rf /var/lib/apt/lists/* \
    && ln -s /usr/bin/python3 /usr/local/bin/python

ARG USERNAME=anki
ARG USER_UID=1000
ARG USER_GID=$USER_UID

# Create non-root user
RUN groupadd --gid $USER_GID $USERNAME \
    && useradd --create-home --uid $USER_UID --gid $USER_GID -m $USERNAME
USER $USERNAME
WORKDIR /work
ENTRYPOINT ["/opt/anki/venv/bin/anki"]
LABEL maintainer="Lukas Westholt <support@lukaswestholt.de>"
