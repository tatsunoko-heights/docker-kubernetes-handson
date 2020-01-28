# サンプルなので、全くこのとおりでなくて問題ありません！
FROM ubuntu:latest

# Dockerイメージを軽量にするためには、RUNをできるだけ1回にまとめるのがコツ！
# 複数行に分けて書いても動作はします。
RUN apt-get update && \
    apt-get install -y python && \
    echo "print('Hello')" >/root/hello.py

ENTRYPOINT [ "python", "/root/hello.py" ]