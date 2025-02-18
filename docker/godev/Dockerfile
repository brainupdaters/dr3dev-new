# syntax = docker/dockerfile:experimental

FROM archlinux/base:latest AS builder

RUN pacman -Syu --noconfirm

RUN pacman -Syy && pacman --needed --noconfirm -S sudo git base-devel go && \
	useradd --shell /bin/bash --home-dir /build -m builder && \
	echo "builder ALL=(ALL) NOPASSWD: ALL" > /etc/sudoers.d/builder && \
	chmod 0440 /etc/sudoers.d/builder && \
	sudo -u builder bash -c 'cd ~ && git clone https://aur.archlinux.org/gitflow-avh.git gitflow && cd gitflow && makepkg -si --noconfirm && mv gitflow*.pkg.tar.xz gitflow-avh.pkg.tar.xz' && \
	sudo -u builder bash -c 'cd ~ && git clone https://aur.archlinux.org/gitflow-bashcompletion-avh.git gitflow-bc && cd gitflow-bc && makepkg -s --noconfirm && mv gitflow*.pkg.tar.xz gitflow-bc-avh.pkg.tar.xz'

RUN git clone https://github.com/viant/endly && cd endly && \
	GO111MODULE=on go build -o /build/endly endly/endly.go

FROM archlinux/base:latest

LABEL maintainer Didac Oliveira "didac@drlm.org"

# Install software
COPY --from=builder /build/*/*.pkg.tar.xz /tmp
COPY --from=builder /build/endly /usr/local/bin

RUN pacman -Syu --noconfirm

RUN pacman -Syy && \
	pacman --needed --noconfirm -S sudo git bash-completion neovim python python-neovim npm yarn code ttf-fira-code tmux mariadb-clients busybox openssh docker-compose \
	binutils file findutils make sed gawk gcc util-linux grep unzip && \
	pacman --noconfirm -U /tmp/gitflow-avh.pkg.tar.xz && \
	pacman --noconfirm -U /tmp/gitflow-bc-avh.pkg.tar.xz && \
    rm -rf /tmp/* && \
	pacman -Scc && \
	chmod u+s /bin/busybox && \
	mkdir -vp /usr/local/busybox/bin && \
	busybox --install -s /usr/local/busybox/bin/ && \
	locale-gen en_US.UTF-8 && \
	localedef -i en_US -f UTF-8 en_US.UTF-8

# Create User
ARG MYUSER=developer
ARG	MYUID=1000
ARG	MYGID=1000
ARG DOCKERGID=150
ARG DR3DEV=/tmp/dr3dev

ENV MYUSER=${MYUSER}\
	MYUID=${MYUID} \
	MYGID=${MYGID} \
	DR3DEV=${DR3DEV} \
	LANG=en_US.UTF-8 \
	LC_ALL=en_US.UTF-8 \
	LANGUAGE=en_US.UTF-8 \
	HOME=/home/${MYUSER} \
	GOPATH=/home/${MYUSER}/go \
	PATH=/home/${MYUSER}/go/bin:/usr/local/go/bin:$PATH \
	TERM=xterm-256color

RUN	groupadd --gid ${MYGID} ${MYUSER} && \
	groupmod --gid 151 git && \
	groupmod --gid ${DOCKERGID} docker && \
	useradd --gid ${MYGID} --uid ${MYUID} --home-dir /home/${MYUSER} -m --shell /bin/bash ${MYUSER} && \
	usermod -aG docker ${MYUSER} && \
	echo "${MYUSER} ALL=(ALL) NOPASSWD: ALL" > /etc/sudoers.d/${MYUSER} && \
	chmod 0440 /etc/sudoers.d/${MYUSER} && \
	mkdir -p "$GOPATH/src" "$GOPATH/bin" && chown -R ${MYUSER}:${MYUSER} "$GOPATH"

USER ${MYUSER}
WORKDIR /home/${MYUSER}

# Install Golang
ARG GOLANG_VERSION
ARG GOLANG_ARCH=amd64
RUN curl -fLo go.tar.gz "https://golang.org/dl/go${GOLANG_VERSION}.linux-${GOLANG_ARCH}.tar.gz" && \
		sudo tar -C /usr/local -xzf go.tar.gz && \
		rm go.tar.gz

ENV PATH=$PATH:/usr/local/go/bin:$GOPATH/bin

# Install Protocol Buffers
ARG PROTOBUF_VERSION
ARG PROTOBUF_ARCH
RUN curl -fLo protobuf.zip "https://github.com/protocolbuffers/protobuf/releases/download/v${PROTOBUF_VERSION}/protoc-${PROTOBUF_VERSION}-linux-${PROTOBUF_ARCH}.zip" && \
		sudo unzip -d /usr/local/protoc protobuf.zip && \
		sudo chmod -R +r /usr/local/protoc/include/ && \
		rm protobuf.zip  && \
		go get -v github.com/golang/protobuf/protoc-gen-go

ENV PATH=$PATH:/usr/local/protoc/bin

COPY entrypoint.sh /usr/local/bin/
ENTRYPOINT [ "entrypoint.sh" ]

CMD [ "bash" ]
