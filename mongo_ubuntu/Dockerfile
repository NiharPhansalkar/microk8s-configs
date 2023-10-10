# This assumes you have a node backend which can be run using npm run dev
FROM ubuntu:23.04 as intermediate

RUN apt update && \
    apt install -y git

ARG SSH_PRIVATE_KEY
RUN mkdir /root/.ssh && \
    echo "${SSH_PRIVATE_KEY}" > /root/.ssh/id_ed25519 && \
    chmod 400 /root/.ssh/id_ed25519

# Make sure domain is accepted
RUN touch /root/.ssh/known_hosts
RUN ssh-keyscan github.com >> /root/.ssh/known_hosts

RUN git clone 'git@github.com:Username/Repo.git'

FROM ubuntu:23.04

ENV DEBIAN_FRONTEND=noninteractive

# Essentials
RUN apt update && \
    apt install -y build-essential && \
    apt install -y curl && \
    apt install -y git && \
    apt install -y libssl-dev && \
    apt install -y lsb-release && \
    apt install -y man && \
    apt install -y manpages-dev && \
    apt install -y manpages-posix-dev && \
    apt install -y npm && \
    apt install -y plocate && \
    apt install -y vim-gtk3

# Extra packages required for LFS start
# First for yacc second for textinfo
RUN apt install -y byacc && \
    apt install -y texinfo && \
    apt install -y m4 && \
    apt install -y gawk
#Extra packages end

# For vim plug and plug extensions
RUN git clone 'https://github.com/NiharPhansalkar/ankit-dotfiles.git' && \
    make -C ankit-dotfiles/ install && \
    rm -rf ankit-dotfiles/

# node & npm
RUN sed -i '/globalconfig/d' ~/.npmrc && sed -i '/prefix/d' ~/.npmrc
RUN apt-get update && \
    apt-get install wget curl ca-certificates rsync -y
RUN curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.3/install.sh | bash
ENV NVM_DIR=/root/.nvm
RUN . "$NVM_DIR/nvm.sh" && nvm install node

COPY --from=intermediate /Repo /home/Repo
RUN cd /home/Repo && \
    git switch preferred_branch

WORKDIR /home/Repo
RUN if [ -d "node_modules" ]; then rm -rf node_modules; fi
RUN if [ -f "package_lock.json" ]; then rm -rf package_lock.json; fi
RUN npm cache clean --force && \
    npm install

CMD ["npm", "run", "dev"]
