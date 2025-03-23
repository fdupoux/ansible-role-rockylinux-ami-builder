FROM alpine:3.20
RUN apk add --no-cache --update bash curl openssh-client ansible py3-boto3
RUN mkdir -p /workdir
RUN adduser -u 1000 -S user -G users
RUN mkdir -p /home/user/.ssh
RUN chown user:users /home/user/.ssh
RUN chmod 700 /home/user/.ssh
WORKDIR /home/user/
RUN chown -R user:users /home/user/
USER user
