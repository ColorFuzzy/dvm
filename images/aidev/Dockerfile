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

# Create user eks with sudo privileges
RUN useradd -m -s /bin/bash eks \
  && echo 'eks ALL=(ALL) NOPASSWD:ALL' >> /etc/sudoers

# Switch to eks user
USER eks

# oh-my-bash
RUN bash -c "$(curl -fsSL https://raw.githubusercontent.com/ohmybash/oh-my-bash/master/tools/install.sh)" || true
RUN sed -i 's/^OSH_THEME=.*/OSH_THEME="brainy"/' /home/eks/.bashrc

RUN echo 'source /home/eks/.bashrcx' >> /home/eks/.bashrc

# Homebrew
RUN NONINTERACTIVE=1 /bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
ENV PATH="/home/linuxbrew/.linuxbrew/bin:/home/linuxbrew/.linuxbrew/sbin:${PATH}"

COPY --chown=eks:eks Brewfile /tmp/Brewfile
RUN brew bundle --file /tmp/Brewfile

# Git LFS
RUN git lfs install

# node / npm / npx / pnpm
RUN brew install node@24 || brew install node
ENV PATH="/home/linuxbrew/.linuxbrew/opt/node@24/bin:/home/linuxbrew/.linuxbrew/opt/node/bin:${PATH}"
RUN npm install -g pnpm@latest

COPY --chown=eks:eks .tmux.conf /home/eks/.tmux.conf

# uv / uvx
RUN curl -LsSf https://astral.sh/uv/install.sh | sh
ENV PATH="/home/eks/.local/bin:${PATH}"

# bun / bunx
RUN curl -fsSL https://bun.com/install | bash
ENV PATH="/home/eks/.bun/bin:${PATH}"

# rustup (official installer)
RUN curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | bash -s -- -y
ENV PATH="/home/eks/.cargo/bin:${PATH}"

# AI tools
RUN curl -fsSL https://claude.ai/install.sh | bash
RUN npm i -g @openai/codex

WORKDIR /home/eks
CMD ["bash"]
