FROM ubuntu:24.04

SHELL ["/bin/bash", "-o", "pipefail", "-c"]
ENV DEBIAN_FRONTEND=noninteractive
ENV HOMEBREW_NO_ANALYTICS=1
ENV HOMEBREW_NO_AUTO_UPDATE=1

# Core apt packages
RUN apt-get update \
  && apt-get install -y --no-install-recommends \
    ca-certificates \
    curl \
    git \
    build-essential \
    openssh-client \
    openssh-server \
    pkg-config \
    libssl-dev \
    unzip \
    locales \
    tzdata \
    telnet \
    sudo \
  && rm -rf /var/lib/apt/lists/*

# Locale
RUN sed -i 's/# en_US.UTF-8 UTF-8/en_US.UTF-8 UTF-8/' /etc/locale.gen \
  && locale-gen \
  && update-locale LANG=en_US.UTF-8 LC_ALL=en_US.UTF-8
ENV LANG=en_US.UTF-8
ENV LC_ALL=en_US.UTF-8
ENV TZ=UTC
RUN ln -snf /usr/share/zoneinfo/$TZ /etc/localtime && echo $TZ > /etc/timezone

# Create user aidev with sudo privileges
RUN useradd -m -s /bin/bash aidev \
  && echo 'aidev:123456' | chpasswd \
  && echo 'aidev ALL=(ALL) NOPASSWD:ALL' >> /etc/sudoers

# Configure SSH
RUN mkdir -p /run/sshd \
  && ssh-keygen -t rsa -f /etc/ssh/ssh_host_rsa_key -N '' \
  && ssh-keygen -t ecdsa -f /etc/ssh/ssh_host_ecdsa_key -N '' \
  && ssh-keygen -t ed25519 -f /etc/ssh/ssh_host_ed25519_key -N '' \
  && sed -i 's/#PasswordAuthentication yes/PasswordAuthentication yes/' /etc/ssh/sshd_config \
  && sed -i 's/#PermitRootLogin prohibit-password/PermitRootLogin no/' /etc/ssh/sshd_config \
  && mkdir -p /home/aidev/.ssh \
  && chmod 700 /home/aidev/.ssh \
  && chown aidev:aidev /home/aidev/.ssh

EXPOSE 22

# Switch to aidev user
USER aidev

# oh-my-bash
RUN bash -c "$(curl -fsSL https://raw.githubusercontent.com/ohmybash/oh-my-bash/master/tools/install.sh)" || true
RUN sed -i 's/^OSH_THEME=.*/OSH_THEME="brainy"/' /home/aidev/.bashrc

RUN echo 'source /home/aidev/.bashrcx' >> /home/aidev/.bashrc

# Homebrew
RUN NONINTERACTIVE=1 /bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
ENV PATH="/home/linuxbrew/.linuxbrew/bin:/home/linuxbrew/.linuxbrew/sbin:${PATH}"

COPY --chown=aidev:aidev Brewfile /tmp/Brewfile
RUN brew bundle --file /tmp/Brewfile

# Git LFS
RUN git lfs install

# node / npm / npx / pnpm
RUN brew install node@24 || brew install node
ENV PATH="/home/linuxbrew/.linuxbrew/opt/node@24/bin:/home/linuxbrew/.linuxbrew/opt/node/bin:${PATH}"
RUN npm install -g pnpm@latest

COPY --chown=aidev:aidev .tmux.conf /home/aidev/.tmux.conf

# uv / uvx
RUN curl -LsSf https://astral.sh/uv/install.sh | sh
ENV PATH="/home/aidev/.local/bin:${PATH}"

# bun / bunx
RUN curl -fsSL https://bun.com/install | bash
ENV PATH="/home/aidev/.bun/bin:${PATH}"

# rustup (official installer)
RUN curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | bash -s -- -y
ENV PATH="/home/aidev/.cargo/bin:${PATH}"

# cargo-watch
RUN cargo install cargo-watch --locked

# AI tools
RUN curl -fsSL https://claude.ai/install.sh | bash
RUN npm i -g @openai/codex

WORKDIR /home/aidev
CMD ["/usr/sbin/sshd", "-D"]
