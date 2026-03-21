FROM node:24

# Install basic development tools
RUN apt-get update && apt-get install -y --no-install-recommends \
  less \
  git \
  procps \
  sudo \
  fzf \
  zsh \
  man-db \
  unzip \
  gnupg2 \
  gh \
  jq \
  nano \
  vim \
  wget \
  tmux \
  && apt-get clean && rm -rf /var/lib/apt/lists/*

# Install OpenJDK 21 from Adoptium
ARG JDK_VERSION=21.0.10
RUN ARCH=$(dpkg --print-architecture) && \
  if [ "$ARCH" = "arm64" ]; then \
  JDK_ARCH=aarch64; \
  else \
  JDK_ARCH=x64; \
  fi && \
  wget -q https://github.com/adoptium/temurin21-binaries/releases/download/jdk-21.0.10%2B7/OpenJDK21U-jdk_${JDK_ARCH}_linux_hotspot_21.0.10_7.tar.gz && \
  tar -xzf OpenJDK21U-jdk_${JDK_ARCH}_linux_hotspot_21.0.10_7.tar.gz -C /opt && \
  rm OpenJDK21U-jdk_${JDK_ARCH}_linux_hotspot_21.0.10_7.tar.gz
ENV JAVA_HOME=/opt/jdk-21.0.10+7
ENV PATH=$JAVA_HOME/bin:$PATH

# Install uv and configure Chinese mirror
RUN curl -LsSf https://astral.sh/uv/install.sh | sh && \
  mv /root/.local/bin/uv /usr/local/bin/uv && \
  mv /root/.local/bin/uvx /usr/local/bin/uvx || true

# Ensure default node user has access to /usr/local/share
RUN mkdir -p /usr/local/share/npm-global && \
  chown -R node:node /usr/local/share

ARG USERNAME=node

# Persist bash history.
RUN SNIPPET="export PROMPT_COMMAND='history -a' && export HISTFILE=/commandhistory/.bash_history" \
  && mkdir /commandhistory \
  && touch /commandhistory/.bash_history \
  && chown -R $USERNAME /commandhistory

# Set `DEVCONTAINER` environment variable to help with orientation
ENV DEVCONTAINER=true

# Create workspace and config directories and set permissions
RUN mkdir -p /workspace /home/node/.claude /home/node/.config /home/node/.local && \
  chown -R node:node /workspace /home/node/.claude /home/node/.config /home/node/.local

COPY ./claude/settings.json /home/node/.claude/settings.json
COPY ./claude/.claude.json /home/node/.claude.json

RUN chown -R node:node /home/node/.claude/settings.json /home/node/.claude.json

WORKDIR /workspace

ARG GIT_DELTA_VERSION=0.18.2
RUN ARCH=$(dpkg --print-architecture) && \
  wget "https://github.com/dandavison/delta/releases/download/${GIT_DELTA_VERSION}/git-delta_${GIT_DELTA_VERSION}_${ARCH}.deb" && \
  sudo dpkg -i "git-delta_${GIT_DELTA_VERSION}_${ARCH}.deb" && \
  rm "git-delta_${GIT_DELTA_VERSION}_${ARCH}.deb"

# Set up non-root user
USER node

# Install global packages
ENV NPM_CONFIG_PREFIX=/usr/local/share/npm-global
ENV PATH=$PATH:/usr/local/share/npm-global/bin

# Set the default shell to zsh rather than sh
ENV SHELL=/bin/zsh

# Set the default editor and visual
ENV EDITOR=vim
ENV VISUAL=vim

# Default powerline10k theme
ARG ZSH_IN_DOCKER_VERSION=1.2.0
RUN sh -c "$(wget -O- https://github.com/deluan/zsh-in-docker/releases/download/v${ZSH_IN_DOCKER_VERSION}/zsh-in-docker.sh)" -- \
  -p git \
  -p fzf \
  -p https://github.com/zsh-users/zsh-autosuggestions \
  -p https://github.com/zsh-users/zsh-completions \
  -p https://github.com/zsh-users/zsh-syntax-highlighting \
  -a "source /usr/share/doc/fzf/examples/key-bindings.zsh" \
  -a "source /usr/share/doc/fzf/examples/completion.zsh" \
  -a "export PROMPT_COMMAND='history -a' && export HISTFILE=/commandhistory/.bash_history" \
  -x

# Install Claude
ARG CLAUDE_CODE_VERSION=latest
RUN npm install -g @anthropic-ai/claude-code@${CLAUDE_CODE_VERSION}

# Install OpenCode
ARG OPEN_CODE_VERSION=latest
RUN npm install -g opencode-ai@${OPEN_CODE_VERSION}

# Setup for China mainland
ENV TZ="Asia/Shanghai"
ENV UV_INDEX_URL=https://mirrors.aliyun.com/pypi/simple/
RUN npm config set registry https://registry.npmmirror.com