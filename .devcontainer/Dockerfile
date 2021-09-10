FROM tensorflow/tensorflow:2.4.1 as base

RUN apt-get update && apt-get install -y \
    build-essential \
    ffmpeg \
    git \
    jq \
    uuid-runtime \
    vim \
    wget \
    && apt-get clean \
    && rm --recursive --force /var/lib/apt/lists/*

RUN \
    curl -fsSL https://get.docker.com -o get-docker.sh \
    && sh get-docker.sh
RUN groupmod -g 998 docker

# RUN \
#     echo "deb [signed-by=/usr/share/keyrings/cloud.google.gpg] http://packages.cloud.google.com/apt cloud-sdk main" | \
#     tee -a /etc/apt/sources.list.d/google-cloud-sdk.list \
#     && curl https://packages.cloud.google.com/apt/doc/apt-key.gpg | \
#     apt-key --keyring /usr/share/keyrings/cloud.google.gpg add - \
#     && apt-get update -y \
#     && apt-get install google-cloud-sdk -y \
#     && apt-get clean \
#     && rm --recursive --force /var/lib/apt/lists/*

RUN \
    curl -Lo skaffold https://storage.googleapis.com/skaffold/releases/latest/skaffold-linux-amd64 \
    && install skaffold /usr/local/bin/

RUN \
    wget https://dvc.org/deb/dvc.list -O /etc/apt/sources.list.d/dvc.list \
    && wget -qO - https://dvc.org/deb/iterative.asc | apt-key add - \
    && apt-get update \
    && apt-get install -y dvc \
    && apt-get clean \
    && rm --recursive --force /var/lib/apt/lists/*

RUN \
    curl -LO https://github.com/BurntSushi/ripgrep/releases/download/12.1.1/ripgrep_12.1.1_amd64.deb \
    && dpkg -i ripgrep_12.1.1_amd64.deb

RUN pip install \
    black \
    flake8 \
    ipykernel \
    mypy \
    pre-commit \
    pydantic \
    pytest

RUN pip install \
    apache-beam[gcp] \
    ffmpeg-python \
    gcsfs \
    matplotlib \
    pandas \
    tensorflow-io==0.17.0 \
    pandas \
    pyaml \
    scikit-learn \
    scipy

FROM base as ci-container

RUN \
    curl --location https://deb.nodesource.com/setup_12.x | bash \
    && apt-get install -y nodejs \
    && apt-get clean \
    && rm --recursive --force /var/lib/apt/lists/*

RUN \
    npm config set user 0 \
    && npm install --global "@dvcorg/cml@0"

FROM base as dev-container

ARG USER_ID=1000
ARG GROUP_ID=1000
RUN \
    groupadd -r -g ${GROUP_ID} vion \
    && useradd -u ${USER_ID} -g ${GROUP_ID} -m -d /home/vion -s /bin/bash vion \
    && usermod -aG docker vion

RUN \
    mkdir /bash_history \
    && touch /bash_history/.bash_history \
    && chown -R vion /bash_history \
    && echo "export PROMPT_COMMAND='history -a' && export HISTFILE=/bash_history/.bash_history" >> /home/vion/.bashrc

USER vion

RUN mkdir ~/.config

RUN \
    curl https://raw.githubusercontent.com/git/git/master/contrib/completion/git-completion.bash > ~/.git-completion.bash \
    && curl https://raw.githubusercontent.com/git/git/master/contrib/completion/git-prompt.sh > ~/.git-prompt.bash \
    && echo "source ~/.git-completion.bash" >> ~/.bashrc \
    && echo "source ~/.git-prompt.bash" >> ~/.bashrc \
    && echo "export GIT_PS1_SHOWDIRTYSTATE=true" >> ~/.bashrc \
    && echo "export PS1='\w\[\033[33m\]\$(__git_ps1)\[\033[00m\]$ '" >> ~/.bashrc

RUN \
    git clone --depth 1 https://github.com/junegunn/fzf.git ~/.fzf \
    && ~/.fzf/install --all

WORKDIR /workspace
