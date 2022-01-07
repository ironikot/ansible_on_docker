# Amazon EC2
FROM ubuntu:latest


RUN apt-get update && \
    apt-get install -y software-properties-common && \
    apt-get update 

# patでOpenSSｈサーバをインストールする 上書きするか？って聞かれるからYESを突っ込ませる。
RUN apt -y install openssh-server 
RUN yes | ssh-keygen -f /etc/ssh/ssh_host_rsa_key -N '' -t rsa
RUN yes | ssh-keygen -f /etc/ssh/ssh_host_ecdsa_key -N '' -t ecdsa
# sshd起動時に公開鍵が必要なため、作成
RUN ssh-keygen -A
RUN mkdir -p var/run/sshd
# passwordを付けて、Ansibleで暗号化を試す
RUN echo 'root:root' |chpasswd

RUN sed -ri 's/^#?PermitRootLogin\s+.*/PermitRootLogin yes/' /etc/ssh/sshd_config
RUN sed -ri 's/UsePAM yes/#UsePAM yes/g' /etc/ssh/sshd_config

RUN mkdir /root/.ssh

EXPOSE 22
# sshdを起動
CMD ["/usr/sbin/sshd", "-D"]