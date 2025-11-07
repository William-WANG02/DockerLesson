FROM docker.cnb.cool/looc/default-dev-env:latest

RUN curl -fsSL https://code-server.dev/install.sh | sh &&\
    code-server --install-extension bierner.markdown-mermaid &&\
    code-server --install-extension ms-azuretools.vscode-docker
