# サンプルなので、全くこのとおりでなくて問題ありません！！
FROM nginx:latest

COPY ./html /usr/share/nginx/html

## ↓なくても動きます: EXPOSE命令はドキュメントとしての役割しかありません。実際にポートを開けるのは、実行時の -p コマンドです
EXPOSE 80
## ↓なくても動きます: nginxのベースイメージ(FROM命令で指定)の中で指定されているため、自分で書かなくても問題ありません
CMD ["nginx", "-g", "daemon off;"]