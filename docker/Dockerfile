FROM ubuntu:20.04

LABEL maintainer="Jongkuk Lim <lim.jeikei@gmail.com>"

ENV DEBIAN_FRONTEND=noninteractive
ENV TZ=Asia/Seoul
ENV TERM xterm-256color

ARG UID=1000
ARG GID=1000
RUN groupadd -g $GID -o user && useradd -m -u $UID -g $GID -o -s /bin/bash user

RUN ln -snf /usr/share/zoneinfo/$TZ /etc/localtime && echo $TZ > /etc/timezone
RUN apt-get update && apt-get install -y sudo dialog apt-utils tzdata
RUN echo "%sudo ALL=(ALL) NOPASSWD: ALL" >> /etc/sudoers && echo "user:user" | chpasswd && adduser user sudo

WORKDIR /home/user
USER user

RUN sudo apt-get update && sudo apt-get install -y libgl1-mesa-dev && sudo apt-get -y install jq

ENV NVIDIA_VISIBLE_DEVICES ${NVIDIA_VISIBLE_DEVICES:-all}
ENV NVIDIA_DRIVER_CAPABILITIES ${NVIDIA_DRIVER_CAPABILITIES:+$NVIDIA_DRIVER_CAPABILITIES,}graphics

RUN sudo apt-get update && sudo apt-get -y install wget curl git
RUN curl -s https://raw.githubusercontent.com/JeiKeiLim/my_term/main/run.sh | /bin/bash

RUN sudo apt-get update && sudo apt-get install -y zsh && \
    sh -c "$(curl -fsSL https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh)" "" --unattended && \
    git clone --depth=1 https://github.com/romkatv/powerlevel10k.git ${ZSH_CUSTOM:-$HOME/.oh-my-zsh/custom}/themes/powerlevel10k
RUN echo "\n# Custom settings" >> /home/user/.zshrc && \
    echo "export PATH=/home/user/.local/bin:$PATH" >> /home/user/.zshrc && \
    echo "export LC_ALL=C.UTF-8 && export LANG=C.UTF-8" >> /home/user/.zshrc && \
    sed '11 c\ZSH_THEME=powerlevel10k/powerlevel10k' ~/.zshrc  > tmp.txt && mv tmp.txt ~/.zshrc && \
    echo 'POWERLEVEL9K_DISABLE_CONFIGURATION_WIZARD=true' >> ~/.zshrc

# Change default shell to zsh
RUN sudo chsh -s /bin/zsh

# Install ruby
RUN sudo apt-get install -y ruby-full build-essential zlib1g-dev
RUN echo '# Install Ruby Gems to ~/gems' >> ~/.bashrc && \
    echo 'export GEM_HOME="$HOME/gems"' >> ~/.bashrc && \
    echo 'export PATH="$HOME/gems/bin:$PATH"' >> ~/.bashrc
RUN echo '# Install Ruby Gems to ~/gems' >> ~/.zshrc && \
    echo 'export GEM_HOME="$HOME/gems"' >> ~/.zshrc && \
    echo 'export PATH="$HOME/gems/bin:$PATH"' >> ~/.zshrc

# Install jekyll
RUN /bin/bash -lic "gem install jekyll -v 4.2.2"
RUN /bin/bash -ilc "gem install bundler github-pages tzinfo-data jekyll-algolia"
COPY Gemfile /home/user/Gemfile
RUN /bin/bash -lic "bundle install && bundle update && rm Gemfile.lock Gemfile"
# Run bundle exec jekyll serve -H 0.0.0.0
