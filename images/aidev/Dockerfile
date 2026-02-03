FROM ubuntu:24.04

SHELL ["/bin/bash", "-o", "pipefail", "-c"]
ENV DEBIAN_FRONTEND=noninteractive

# Core apt packages (gcc is included in build-essential)
RUN apt-get update \
  && apt-get install -y --no-install-recommends \
    ca-certificates \
    curl \
    git \
    build-essential \
    openssh-client \
    unzip \
    locales \
    tzdata \
    telnet \
    sudo \
    tmux \
    ripgrep \
    fd-find \
    tree \
    vim \
  && ln -sf /usr/bin/fdfind /usr/bin/fd \
  && rm -rf /var/lib/apt/lists/*

# Node.js 24 via NodeSource
RUN curl -fsSL https://deb.nodesource.com/setup_24.x | bash - \
  && apt-get install -y --no-install-recommends nodejs \
  && rm -rf /var/lib/apt/lists/*

# Locale
RUN sed -i 's/# en_US.UTF-8 UTF-8/en_US.UTF-8 UTF-8/' /etc/locale.gen \
  && locale-gen \
  && update-locale LANG=en_US.UTF-8 LC_ALL=en_US.UTF-8
ENV LANG=en_US.UTF-8
ENV LC_ALL=en_US.UTF-8
ENV TZ=UTC
RUN ln -snf /usr/share/zoneinfo/$TZ /etc/localtime && echo $TZ > /etc/timezone

# Create user eks with sudo privileges
RUN useradd -m -s /bin/bash eks \
  && echo 'eks ALL=(ALL) NOPASSWD:ALL' >> /etc/sudoers

# Switch to eks user
USER eks

# oh-my-bash
RUN bash -c "$(curl -fsSL https://raw.githubusercontent.com/ohmybash/oh-my-bash/master/tools/install.sh)" || true
RUN sed -i 's/^OSH_THEME=.*/OSH_THEME="brainy"/' /home/eks/.bashrc

COPY --chown=eks:eks .bashrcx /home/eks/.bashrcx
RUN echo 'source /home/eks/.bashrcx' >> /home/eks/.bashrc

COPY --chown=eks:eks .tmux.conf /home/eks/.tmux.conf

# mise (project toolchain manager)
RUN curl https://mise.run | sh
ENV PATH="/home/eks/.local/bin:${PATH}"

# zoxide (smart cd)
RUN curl -sSfL https://raw.githubusercontent.com/ajeetdsouza/zoxide/main/install.sh | sh

# pnpm
RUN sudo npm install -g pnpm@latest

# uv / uvx
RUN curl -LsSf https://astral.sh/uv/install.sh | sh

# bun / bunx
RUN curl -fsSL https://bun.com/install | bash
ENV PATH="/home/eks/.bun/bin:${PATH}"

# rustup (official installer)
RUN curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | bash -s -- -y
ENV PATH="/home/eks/.cargo/bin:${PATH}"

# AI tools
RUN curl -fsSL https://claude.ai/install.sh | bash
RUN sudo npm i -g @openai/codex

WORKDIR /home/eks
CMD ["bash"]
