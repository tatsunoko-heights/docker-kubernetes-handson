# サンプルなので、全くこのとおりでなくて問題ありません！！

FROM ubuntu:latest

# ここで最初に話すのもなんなのですが
# Dockerイメージを軽量にするためには、RUNをできるだけ1回にまとめるのがコツ
RUN apt-get update && \
    apt-get install -y python && \
    echo "print('Hello')" >/root/hello.py

# docker run <image>で Helloが表示されるようにしてみました
ENTRYPOINT [ "python", "/root/hello.py" ]