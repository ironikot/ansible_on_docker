FROM ubuntu:latest

ENV ANSIBLE_HOST_KEY_CHECKING False

RUN apt update && \
    apt install -y software-properties-common && \
    apt update &&\
    apt install -y ansible &&\
    apt install -y ssh &&\
    apt install -y sshpass
# sshpassはSSHでPWを使った接続では必要みたい。
# 編集して遊ぶためにvimを入れる。
RUN apt install -y vim

CMD /bin/bash
