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
  zip \
  gnupg2 \
  gh \
  jq \
  nano \
  vim \
  wget \
  tmux \
  && apt-get clean && rm -rf /var/lib/apt/lists/*

# Install uv
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

# Set up git with delta as the pager and zdiff3 for merge conflicts
RUN git config --global core.pager delta && \
  git config --global interactive.diffFilter 'delta --color-only' && \
  git config --global delta.navigate true && \
  git config --global delta.side-by-side true && \
  git config --global delta.line-numbers true && \
  git config --global delta.dark true && \
  git config --global merge.conflictStyle zdiff3

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

# Install SDKMAN for SDK version management
RUN curl -s "https://get.sdkman.io?ci=true" -o /tmp/sdkman-install.sh && \
  sed -i '/^find "$sdkman_src_folder"/d' /tmp/sdkman-install.sh && \
  sed -i '/^find "$sdkman_libexec_folder"/d' /tmp/sdkman-install.sh && \
  bash /tmp/sdkman-install.sh && \
  rm /tmp/sdkman-install.sh

# Install nvm for Node version management
ENV NVM_DIR=/home/node/.nvm
RUN curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.40.4/install.sh | bash && \
  rm -rf $NVM_DIR/.cache $NVM_DIR/.nvexternalsrc 2>/dev/null || true

# Source SDKMAN and nvm in shell
RUN echo "source /home/node/.sdkman/bin/sdkman-init.sh" >> /home/node/.bashrc && \
  echo "source /home/node/.sdkman/bin/sdkman-init.sh" >> /home/node/.zshrc && \
  echo "export NVM_DIR=/home/node/.nvm" >> /home/node/.bashrc && \
  echo "[ -s \"$NVM_DIR/nvm.sh\" ] && \. \"$NVM_DIR/nvm.sh\"" >> /home/node/.bashrc && \
  echo "export NVM_DIR=/home/node/.nvm" >> /home/node/.zshrc && \
  echo "[ -s \"$NVM_DIR/nvm.sh\" ] && \. \"$NVM_DIR/nvm.sh\"" >> /home/node/.zshrc

# Install Java, Maven, and Gradle
RUN bash -c "source /home/node/.sdkman/bin/sdkman-init.sh && \
  sdk install java 21.0.10-tem && \
  sdk install maven && \
  sdk install gradle"

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