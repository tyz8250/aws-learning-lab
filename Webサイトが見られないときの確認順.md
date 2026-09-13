# Webサイトが見られないときの確認順

## 1. 名前解決

dig / nslookup

DNSは正しいIPアドレスを返しているか？

## 2. ネットワーク

ping

※ ping失敗だけではサーバーダウンとは言えない

## 3. TCP

443番ポートまで接続できるか？

## 4. AWSネットワーク

- Public IP
- Route Table
- Internet Gateway
- Security Group

## 5. サーバー

Webサーバーのプロセスは起動しているか？

## 6. アプリ

HTTPレスポンスは返っているか？