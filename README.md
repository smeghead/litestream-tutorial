# litestream tutorial

https://litestream.io/getting-started/

```bash
wget https://github.com/benbjohnson/litestream/releases/download/v0.3.8/litestream-v0.3.8-linux-amd64.deb
sudo dpkg -i litestream-v0.3.8-linux-amd64.deb
sudo systemctl enable litestream
sudo systemctl start litestream
```


## S3互換のストレージを起動する。

```bash
docker run --rm -p  9000:9000 -p 9001:9001 minio/minio server /data --console-address ":9001"
```

http://localhost:9001/ にアクセスしてログインする。

```
Username: minioadmin
Password: minioadmin
```

新しいバケット mybkt を作る。

## DBを作る

```bash
sqlite3 fruits.db
```

```sql
CREATE TABLE fruits (name TEXT, color TEXT);
INSERT INTO fruits (name, color) VALUES ('apple', 'red');
INSERT INTO fruits (name, color) VALUES ('banana', 'yellow');

```

```bash
export LITESTREAM_ACCESS_KEY_ID=minioadmin
export LITESTREAM_SECRET_ACCESS_KEY=minioadmin
```

## replicate開始

```bash
litestream replicate fruits.db s3://mybkt.localhost:9000/fruits.db
```

backgroundで起動しつづけるイメージ。
fruits.dbが更新されれば、ストレージにコピーされる。(差分だけ？)


## restore

```bash
litestream restore -o fruits2.db s3://mybkt.localhost:9000/fruits.db
```

backupのようなもの。s3のダウンロードと同じことなんだろうか？



