マイクロサービス Kubernetes サンプル
=====================

[このサンプルを開始するためのドイツ語の手引き](WIE-LAUFEN.md)

このサンプルは、私のマイクロサービスブックのサンプルと類似しています
（[英語版](http://microservices-book.com/) / [ドイツ語版](http://microservices-buch.de/)）。該当のサンプルは https://github.com/ewolff/microservice にあります。

ただし、このデモでは Docker 環境として [Kubernetes](https://kubernetes.io/) を使用しています。Kubernetes はサービスディスカバリーとロードバランシングもサポートしています。Apache httpd をリバースプロキシとして使用し、サービスへの呼び出しをルーティングします。

このプロジェクトでは、Docker コンテナ内で完全なマイクロサービスのデモシステムを作成します。サービスは、Java を使用して Spring と Spring Cloud で実装されています。

このサンプルでは、以下の3つのマイクロサービスを使用しています：
- `Order`：注文を処理します。
- `Customer`：顧客データを管理します。
- `Catalog`：カタログ内の商品を管理します。

実行方法
---------

実行方法については [How to run](HOW-TO-RUN.md) を参照してください。

Apache HTTP ロードバランサー
------------------------

Apache HTTP はデモのウェブページをポート8080で提供します。また、HTTPリクエストをマイクロサービスに転送します。これは厳密には必要ありませんが、各サービスは Minikube ホスト上で独自のポートを持っています。それでも、この設定によりシステム全体のエントリーポイントが一元化されます。Apache HTTP はこのためにリバースプロキシとして構成されています。ロードバランシングは Kubernetes に任せられています。

この Apache HTTP を構成するために、Kubernetes からすべての登録済みサービスを取得する必要があります。DNS を使用してこれを行います。

これがどのように機能するかについては、[microservice-kubernetes-demo/apache](microservice-kubernetes-demo/apache/) サブディレクトリを参照してください。

コードに関する注意事項
-------------------

マイクロサービスは以下の通りです：

- [microservice-kubernetes-demo-catalog](microservice-kubernetes-demo/microservice-kubernetes-demo-catalog) は、アイテムを管理するアプリケーションです。
- [microservice-kubernetes-demo-customer](microservice-kubernetes-demo/microservice-kubernetes-demo-customer) は、顧客を管理します。
- [microservice-kubernetes-demo-order](microservice-kubernetes-demo/microservice-kubernetes-demo-order) は注文処理を行います。このサービスは、microservice-kubernetes-demo-catalog と microservice-kubernetes-demo-customer を利用します。

これらのマイクロサービスは REST を使用して互いに通信します。たとえば、[CatalogClient](microservice-kubernetes-demo/microservice-kubernetes-demo-order/src/main/java/com/ewolff/microservice/order/clients/CatalogClient.java) を参照してください。ホスト名はテスト用にスタブを使用できるように設定可能です。デフォルトは `catalog` で、これは Kubernetes で動作します。他のマイクロサービスは Kubernetes 内蔵の DNS を使用して見つけられます。Kubernetes は IP レベルでのロードバランシングを行います。

これらのマイクロサービスには、それぞれ単独で実行するための Java メインアプリケーションが `src/test/java` にあります。`microservice-demo-order` は、その際に他のサービスのスタブを使用します。また、_コンシューマードリブン契約_ を使用するテストもあります。これにより、サービスが正しいインターフェースを提供していることが確認されます。これらの CDC テストは microservice-demo-order でスタブを検証するために使用されます。`microservice-kubernetes-demo-customer` と `microserivce-kubernetes-demo-catalog` では、実装された REST サービスを検証するために使用されます。

なお、このコードには Kubernetes に依存する部分はありません。