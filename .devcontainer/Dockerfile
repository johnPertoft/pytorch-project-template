FROM pytorch/pytorch:1.11.0-cuda11.3-cudnn8-runtime

# System dependencies.
RUN apt-get update && apt-get install -y \
    build-essential \
    curl \
    git \
    jq \
    locales \
    sudo \
    vim \
    wget \
    zsh \
    && apt-get clean \
    && rm --recursive --force /var/lib/apt/lists/*

# Setup locale.
RUN echo "en_US.UTF-8 UTF-8" >> /etc/locale.gen && locale-gen

# Install gcloud cli.
RUN echo "deb [signed-by=/usr/share/keyrings/cloud.google.gpg] http://packages.cloud.google.com/apt cloud-sdk main" \
    | tee -a /etc/apt/sources.list.d/google-cloud-sdk.list \
    && curl https://packages.cloud.google.com/apt/doc/apt-key.gpg \
    | apt-key --keyring /usr/share/keyrings/cloud.google.gpg add - \
    && apt-get update -y && apt-get install google-cloud-sdk -y \
    && apt-get clean \
    && rm --recursive --force /var/lib/apt/lists/*

# Install python dev dependencies in separate environments via
# pipx to avoid dependency conflicts.
ENV PIPX_HOME=/usr/local/py-utils \
    PIPX_BIN_DIR=/usr/local/py-utils/bin
ENV PATH=$PATH:$PIPX_BIN_DIR
RUN pip install pipx && pipx ensurepath
RUN pipx install black==22.3.0 \
    && pipx install dvc[gs]==2.10.1 \
    && pipx install flake8==4.0.1 \
    && pipx install mypy==0.942 \
    && pipx install pytest==7.1.1

# Create non-root user.
ARG USERNAME=vscode
ARG USER_UID=1000
ARG USER_GID=$USER_UID   
RUN \
    groupadd --gid $USER_GID $USERNAME \
    && useradd --uid $USER_UID --gid $USER_GID --create-home --shell /bin/zsh $USERNAME \
    && echo $USERNAME ALL=\(root\) NOPASSWD:ALL > /etc/sudoers.d/$USERNAME \
    && chmod 0440 /etc/sudoers.d/$USERNAME

# Switch to non-root user.
USER $USERNAME

# Install Oh-my-zsh.
RUN sh -c "$(curl -fsSL https://raw.github.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"
COPY custom.zsh-theme /home/$USERNAME/.oh-my-zsh/custom/themes/custom.zsh-theme
RUN sed -i -e 's/ZSH_THEME=.*/ZSH_THEME="custom"/g' ~/.zshrc

# Install fzf.
RUN git clone --depth 1 https://github.com/junegunn/fzf.git ~/.fzf && ~/.fzf/install --all

ENV PATH=$PATH:/home/$USERNAME/.local/bin

# Install Python dependencies.
RUN pip install \
    jupyter==1.0.0 \
    matplotlib==3.5.1 \
    tensorboard==2.8.0 \
    torchdata==0.3.0