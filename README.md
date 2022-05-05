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

## minio files

replicateされたファイルは、単一のsqlite3データベースファイルではなく、以下のようになっていた。

```bash
$ wget https://dl.min.io/client/mc/release/linux-amd64/mc
$ chmod +x ./mc
$ ./mc alias set minio http://localhost:9000 minioadmin minioadmin
$ ./mc ls -r mybkt minio
[2022-05-05 21:22:45 JST]   418B STANDARD mybkt/fruits.db/generations/0f67fb6531a7c63b/snapshots/00000000.snapshot.lz4
[2022-05-05 21:22:45 JST]   483B STANDARD mybkt/fruits.db/generations/0f67fb6531a7c63b/wal/00000000_00000000.wal.lz4
[2022-05-05 21:23:46 JST]   119B STANDARD mybkt/fruits.db/generations/0f67fb6531a7c63b/wal/00000001_00000000.wal.lz4
[2022-05-05 21:24:53 JST]   127B STANDARD mybkt/fruits.db/generations/0f67fb6531a7c63b/wal/00000001_00001038.wal.lz4
[2022-05-05 21:24:53 JST]   119B STANDARD mybkt/fruits.db/generations/0f67fb6531a7c63b/wal/00000002_00000000.wal.lz4
[2022-05-05 21:29:23 JST]   145B STANDARD mybkt/fruits.db/generations/0f67fb6531a7c63b/wal/00000002_00001038.wal.lz4
[2022-05-05 21:29:23 JST]   119B STANDARD mybkt/fruits.db/generations/0f67fb6531a7c63b/wal/00000003_00000000.wal.lz4
[2022-05-05 21:32:34 JST]   157B STANDARD mybkt/fruits.db/generations/0f67fb6531a7c63b/wal/00000003_00001038.wal.lz4
[2022-05-05 21:32:34 JST]   115B STANDARD mybkt/fruits.db/generations/0f67fb6531a7c63b/wal/00000004_00000000.wal.lz4
[2022-05-05 21:35:46 JST]   168B STANDARD mybkt/fruits.db/generations/0f67fb6531a7c63b/wal/00000004_00001038.wal.lz4
[2022-05-05 21:35:46 JST]   119B STANDARD mybkt/fruits.db/generations/0f67fb6531a7c63b/wal/00000005_00000000.wal.lz4
[2022-05-05 21:31:59 JST]   472B STANDARD mybkt/fruits.db/generations/2190b1717ec0b853/snapshots/00000000.snapshot.lz4
[2022-05-05 21:31:59 JST]   119B STANDARD mybkt/fruits.db/generations/2190b1717ec0b853/wal/00000000_00000000.wal.lz4
[2022-05-05 21:33:11 JST]   157B STANDARD mybkt/fruits.db/generations/2190b1717ec0b853/wal/00000000_00001038.wal.lz4
[2022-05-05 21:33:11 JST]   119B STANDARD mybkt/fruits.db/generations/2190b1717ec0b853/wal/00000001_00000000.wal.lz4
```

