# [利用方法] ScalarDB Cluster - 主要機能の設定例と関連ドキュメント

updated: 2025-08-26 19:48:19 JST

## 対象製品/バージョン

- ScalarDB Cluster v3.16 -

## 質問

ScalarDB Cluster で、よく利用される機能に関する設定例を教えてください。

## 回答

本書では、以下の構成項目に関する設定方法と、それに関連する公式ドキュメントへのリンクを掲載しています。

[設定のサンプルはこちら](/files/scalardb/deploy/scalardb_configuration_commonly_used_features/scalardb-cluster-custom-values.yaml)


- ライセンス設定
- マルチストレージトランザクション設定
- SQLインターフェース設定
- 認証・認可設定
- 保存データの暗号化設定
- 通信経路の暗号化設定

詳細は各項目の公式ドキュメントリンクをご参照ください。

設定で利用される各種パラメータは、Kubernetes リソースの Secret を利用することも可能です。詳細は下記をご覧ください。
- [How to use Secret resources to pass credentials as environment variables into the properties file](https://scalardb.scalar-labs.com/docs/latest/helm-charts/use-secret-for-credentials/)



### ライセンス設定
- [ScalarDB Enterprise Edition Licensing](https://scalardb.scalar-labs.com/docs/latest/scalar-licensing/#scalardb-enterprise-edition)

以下は、Trialライセンス利用時の設定例です。Productionライセンスを使用する際は、license_key の値に加え、前述のリンクから取得した license_check_cert_pem を適切な値に差し替えてください。
```
    # Licensing Configuration
    # Documentation:
    #   - https://scalardb.scalar-labs.com/docs/latest/scalar-licensing/#scalardb-enterprise-edition
    scalar.db.cluster.node.licensing.license_key={"organization_name":"CoE / Test","expiration_date_time":"2024-10-10T11:15:15.769+09:00[Asia/Tokyo]","product_name":"ScalarDB Cluster","product_version":3,"license_type":"trial","signature":"MEUCIQ......../g="}

    # license_check_cert_pem
    scalar.db.cluster.node.licensing.license_check_cert_pem=-----\nMIICIzCCAcigAwIBAgIIKT9LIGX1TJQwCgYIKoZIzj0EAwIwZzELMAkGA1UEBhMC\nSlAxDjAMBgNVBAgTBVRva3lvMREwDwYDVQQHEwhTaGluanVrdTEVMBMGA1UEChMM\nU2NhbGFyLCBJbmMuMR4wHAYDVQQDExV0cmlhbC5zY2FsYXItbGFicy5jb20wHhcN\nMjMxMTE2MDcxMDM5WhcNMjQwMjE1MTMxNTM5WjBnMQswCQYDVQQGEwJKUDEOMAwG\nA1UECBMFVG9reW8xETAPBgNVBAcTCFNoaW5qdWt1MRUwEwYDVQQKEwxTY2FsYXIs\nIEluYy4xHjAcBgNVBAMTFXRyaWFsLnNjYWxhci1sYWJzLmNvbTBZMBMGByqGSM49\nAgEGCCqGSM49AwEHA0IABBSkIYAk7r5FRDf5qRQ7dbD3ib5g3fb643h4hqCtK+lC\nwM4AUr+PPRoquAy+Ey2sWEvYrWtl2ZjiYyyiZw8slGCjXjBcMA4GA1UdDwEB/wQE\nAwIFoDAdBgNVHSUEFjAUBggrBgEFBQcDAQYIKwYBBQUHAwIwDAYDVR0TAQH/BAIw\nADAdBgNVHQ4EFgQUbFyOWFrsjkkOvjw6vK3gGUADGOcwCgYIKoZIzj0EAwIDSQAw\nRgIhAKwigOb74z9BdX1+dUpeVG8WrzLTIqdIU0w+9jhAueXoAiEA6cniJ3qsP4j7\nsck62kHnFpH1fCUOc/b/B8ZtfeXI2Iw=\n-----END CERTIFICATE-----
```


### マルチストレージトランザクション設定
- [Multi-Storage Transactions](https://scalardb.scalar-labs.com/docs/latest/multi-storage-transactions/)

MySQL と PostgreSQLの二つのデータベースを利用する例を紹介します。

マルチストレージトランザクションをサポートする為には、 scalar.db.transaction_manager=consensus-commit に設定します。
加えて、 scalar.db.storage=multi-storage を宣言した上で、各ストレージの名前を scalar.db.multi_storage.storage に定義します。ここでは、mysql と postgres というストレージ名を定義します。
　
```
    # Transaction Management Configuration
    # Documentation:
    #   - https://scalardb.scalar-labs.com/docs/latest/configurations/#general-configurations
    #   - https://scalardb.scalar-labs.com/docs/latest/api-guide/#transaction-api
    # Available options:
    #   - consensus-commit: Full ACID transactions with consensus-based commit protocol
    #   - single-crud-operation: To run non-transactional storage operations
    scalar.db.transaction_manager=consensus-commit

    # Storage Configuration - Multi-storage setup
    # Documentation: https://scalardb.scalar-labs.com/docs/latest/multi-storage-transactions/
    # Available storage types:
    #   - multi-storage: Multiple storage backends with namespace mapping
    scalar.db.storage=multi-storage
    
    # Define available storage backend names
    scalar.db.multi_storage.storages=mysql,postgres 
```
先に定義したストレージ名毎に、接続するデータベースへの接続情報を設定します。
- [Storage-related configurations](https://scalardb.scalar-labs.com/docs/latest/scalardb-cluster/scalardb-cluster-configurations?utm_source=chatbot&utm_medium=support&utm_campaign=ai-chatbot#storage-related-configurations)

NoSQL関連の設定、それぞれの関連パラメータにつきましては、前述のリンクよりドキュメントサイトにて確認可能です。こちらの説明は、シングルストレージの設定が記載されています。
その為、マルチストレージの設定では、この部分の読み替えが必要です。

例えば、
```
scalar.db.storage
```
の設定は、マルチストレージでは
```
scalar.db.multi_storage.storages.`<strage_name>`.storage
```
のように multi_storage.storages.`<strage_name>` が付加された表記となります。


また、本書の例では、 cross_partition_scan を有効にしています。ordering の設定は、JDBCデータベースのみ利用可能です。
- [Cross-partition scan configurations](https://scalardb.scalar-labs.com/docs/latest/scalardb-cluster/scalardb-cluster-configurations#cross-partition-scan-configurations)

まずは、mysql に関する設定例です。

記載のデータベースのコンタクトポイント、データベース名、ユーザ名、パスワードはサンプル設定です。お手元の環境に合わせて変更してください。

```
    # Definitions for each storage: scalar.db.multi_storage.storages.<one of scalar.db.multi_storage.storages>.<parameter name>
    # mysql Storage Configuration
    # JDBC driver options for MySQL:
    #   - jdbc: Standard JDBC driver for MySQL/MariaDB
    scalar.db.multi_storage.storages.mysql.storage=jdbc
    
    # Please change the following connection details according to your environment
    # MySQL connection URL
    scalar.db.multi_storage.storages.mysql.contact_points=jdbc:mysql://mysql.coe.scalar.local:3306/storage_mysql
    
    # MySQL username from environment
    scalar.db.multi_storage.storages.mysql.username=scalaradmin
    
    # MySQL password from environment
    scalar.db.multi_storage.storages.mysql.password=scalaradmin

    # Cross-partition scan options (performance vs consistency trade-offs):
    #   - true: Enable cross-partition operations (may impact performance)
    #   - false: Disable cross-partition operations (better performance, limited queries)
    scalar.db.multi_storage.storages.mysql.cross_partition_scan.enabled=true
    scalar.db.multi_storage.storages.mysql.cross_partition_scan.filtering.enabled=true
    scalar.db.multi_storage.storages.mysql.cross_partition_scan.ordering.enabled=true
```
続いてpostgres に関する設定です。

記載のデータベースのコンタクトポイント、データベース名、ユーザ名、パスワードはサンプル設定です。お手元の環境に合わせて変更してください。

```
    # postgres Storage Configuration
    # JDBC driver options for PostgreSQL:
    #   - jdbc: Standard JDBC driver for PostgreSQL
    scalar.db.multi_storage.storages.postgres.storage=jdbc
    # Please change the following connection details according to your environment
    # PostgreSQL connection URL
    scalar.db.multi_storage.storages.postgres.contact_points=jdbc:postgresql://postgres.coe.scalar.local:5432/storage_postgres

    # PostgreSQL username from environment
    scalar.db.multi_storage.storages.postgres.username=scalaradmin

    # PostgreSQL password from environment
    scalar.db.multi_storage.storages.postgres.password=scalaradmin

    # Cross-partition scan options (performance vs consistency trade-offs):
    #   - true: Enable cross-partition operations (may impact performance)
    #   - false: Disable cross-partition operations (better performance, limited queries)
  　scalar.db.multi_storage.storages.postgres.cross_partition_scan.enabled=true
    scalar.db.multi_storage.storages.postgres.cross_partition_scan.filtering.enabled=true
    scalar.db.multi_storage.storages.postgres.cross_partition_scan.ordering.enabled=true
```

最後に、ネームスペースとストレージのマッピング（namespace_mapping）と、デフォルトストレージ（default_storage）の設定を行います。

ScalarDBでは、ネームスペースを通じてストレージにアクセスします。

ネームスペースとストレージのマッピング（namespace_mapping）では、ScalarDBで利用するネームスペースを管理するストレージを指定します。
例では、ns_postgres というネームスペースに postgres ストレージを、ns_mysql というネームスペースに mysql ストレージを設定しています。

デフォルトストレージ（default_storage）は、ネームスペースとストレージのマッピング（namespace_mapping）で定義されていないネームスペースが指定された場合、ここで定義されたストレージが、そのネームスペース用に利用されます。

```
    # Namespace to Storage Mapping - Route different namespaces to specific storage backends
    # Format: namespace1:storage1,namespace2:storage2
    # Special namespaces:
    #   - coordinator: Used for transaction coordination metadata
    # Please change the namespace mapping according to your application requirements
    scalar.db.multi_storage.namespace_mapping=ns_postgres:postgres,ns_mysql:mysql

    # Default Storage Backend
    # Fallback storage for namespaces not explicitly mapped above
    scalar.db.multi_storage.default_storage=postgres
```

### SQLインターフェース設定
- [ScalarDB SQL](https://scalardb.scalar-labs.com/docs/latest/scalardb-cluster/scalardb-cluster-configurations/#sql-related-configurations)

ScalarDB Cluster SQL を有効にするには、scalar.db.sql.enabledを true に設定します。

```
    # SQL Interface Configuration
    # Documentation: https://scalardb.scalar-labs.com/docs/latest/scalardb-cluster/scalardb-cluster-configurations/#sql-related-configurations
    # SQL interface options:
    #   - true: Enable ScalarDB SQL interface for SQL queries
    #   - false: Disable SQL interface (use only native ScalarDB API)
    scalar.db.sql.enabled=true

```

### 認証・認可設定
- [ScalarDB Auth with SQL](https://scalardb.scalar-labs.com/docs/latest/scalardb-cluster/scalardb-auth-with-sql/)

認証と認可を有効にするには、scalar.db.cluster.auth.enabled を true に設定します。

:::info
前述のリンク先ドキュメントに記載されている通り、認証および認可を有効にする場合は、内部的にパーティション間スキャンが実行されます。そのため、システム名前空間（デフォルトは scalardb）において、scalar.db.cross_partition_scan.enabled を true に設定する必要があります。
:::


```
    # Authentication Configuration
    # Documentation: https://scalardb.scalar-labs.com/docs/latest/scalardb-cluster/scalardb-auth-with-sql/
    # Authentication options:
    #   - true: Enable authentication and authorization (recommended for production)
    #   - false: Disable authentication (development/testing only)
    scalar.db.cluster.auth.enabled=true
```


### 保存データの暗号化設定
- [Data Encryption](https://scalardb.scalar-labs.com/docs/latest/scalardb-cluster/encrypt-data-at-rest/)
- [Encryption configurations (optional based on your environment)](https://scalardb.scalar-labs.com/docs/latest/helm-charts/configure-custom-values-scalardb-cluster#encryption-configurations-optional-based-on-your-environment)

Self-encryption の設定例です。本機能を利用する際には、 scalardbCluster.scalardbClusterNodeProperties と scalardbCluster.encryption の、２か所の設定が必要になります。

scalardbCluster.scalardbClusterNodeProperties
```
    # Encryption Configuration for ScalarDB application internal behavior
    # Documentation: https://scalardb.scalar-labs.com/docs/latest/scalardb-cluster/encrypt-wire-communications
    # Encryption at rest options:
    #   - true: Enable data encryption at rest
    #   - false: Disable encryption (data stored in plain text)
    scalar.db.cluster.encryption.enabled=true
    scalar.db.cluster.encryption.type=self
    scalar.db.cluster.encryption.delete_data_encryption_key_on_drop_table.enabled=false
    scalar.db.cluster.encryption.self.kubernetes.secret.namespace_name=default
```

scalardbCluster.encryption
```
  # Data Encryption Configuration (affects RBAC permissions and volume mounts)
  # Documentation: https://scalardb.scalar-labs.com/docs/latest/scalardb-cluster/data-encryption/
  # Infrastructure-level encryption options:
  #   - true: Enable encryption support in Kubernetes (creates RBAC, volumes)
  #   - false: Disable encryption infrastructure support
  encryption:
    # Enable encryption support in Kubernetes
    enabled: true
    # Encryption type (self-managed)
    type: self
```



### 通信経路の暗号化設定
- [Wire Encryption](https://scalardb.scalar-labs.com/docs/latest/scalardb-cluster/encrypt-wire-communications)

本例では、証明書関連の情報をKubernetes リソースの Secret を利用して設定しています。

- [How to Create Private Key and Certificate Files for TLS Connections in Scalar Products](https://scalardb.scalar-labs.com/docs/latest/scalar-kubernetes/HowToCreateKeyAndCertificateFiles/)


下記ファイル名は設定例です。任意のファイル名で置き換えてることが可能です。
- cacert.pem
- cluster_cert.pem
- cluster_key.pem
- envoy_cert.pem
- envoy_key.pem

```
kubectl create secret generic ca-cert --from-file=ca.crt=cacert.pem -n default
kubectl create secret generic cluster-cert --from-file=tls.crt=cluster_cert.pem -n default
kubectl create secret generic cluster-key --from-file=tls.key=cluster_key.pem -n default
kubectl create secret generic envoy-cert --from-file=tls.crt=envoy_cert.pem -n default
kubectl create secret generic envoy-key --from-file=tls.key=envoy_key.pem -n default
```

scalardb-cluster-custom-values.yaml 内では、下記の３か所を設定します。

envoy.tls
```
  # TLS encryption settings for Envoy proxy
  tls:
    downstream:
      enabled: true
      certChainSecret: "envoy-tls-cert"
      privateKeySecret: "envoy-tls-key"
    upstream:
      enabled: true
      overrideAuthority: "cluster.coe.scalar.local"
      caRootCertSecret: "scalardb-cluster-tls-ca"
```

scalardbCluster.scalardbClusterNodeProperties
```
    # Encryption Configuration for ScalarDB application
    # Documentation: https://scalardb.scalar-labs.com/docs/latest/scalardb-cluster/encrypt-wire-communications
    # Encryption at rest options:
    #   - true: Enable data encryption at rest
    #   - false: Disable encryption (data stored in plain text)
    scalar.db.cluster.encryption.enabled=true
    scalar.db.cluster.encryption.type=self
    scalar.db.cluster.encryption.delete_data_encryption_key_on_drop_table.enabled=false
    scalar.db.cluster.encryption.self.kubernetes.secret.namespace_name=default
```

scalardbCluster.tls
```
  # TLS Configuration for ScalarDB Cluster
  # Documentation: https://scalardb.scalar-labs.com/docs/latest/scalardb-cluster/encrypt-wire-communications
  # Infrastructure-level TLS options:
  #   - true: Enable TLS for gRPC endpoints and health checks
  #   - false: Disable TLS at infrastructure level
  tls:
    # Enable TLS for gRPC endpoints and health checks
    enabled: true

    # CA root certificate secret name
    caRootCertSecret: "scalardb-cluster-tls-ca"

    # Certificate chain secret name
    certChainSecret: "scalardb-cluster-tls-cert"

    # Private key secret name
    privateKeySecret: "scalardb-cluster-tls-key"

    # Override authority for TLS verification
    overrideAuthority: "cluster.coe.scalar.local"
```


## 関連する Scalar ドキュメント
- [ScalarDB Enterprise Edition Licensing](https://scalardb.scalar-labs.com/docs/latest/scalar-licensing/#scalardb-enterprise-edition)
- [Multi-Storage Transactions](https://scalardb.scalar-labs.com/docs/latest/multi-storage-transactions/)
- [Storage-related configurations](https://scalardb.scalar-labs.com/docs/latest/scalardb-cluster/scalardb-cluster-configurations?utm_source=chatbot&utm_medium=support&utm_campaign=ai-chatbot#storage-related-configurations)
- [Cross-partition scan configurations](https://scalardb.scalar-labs.com/docs/latest/scalardb-cluster/scalardb-cluster-configurations#cross-partition-scan-configurations)
- [ScalarDB SQL](https://scalardb.scalar-labs.com/docs/latest/scalardb-cluster/scalardb-cluster-configurations/#sql-related-configurations)
- [ScalarDB Auth with SQL](https://scalardb.scalar-labs.com/docs/latest/scalardb-cluster/scalardb-auth-with-sql/)
- [Data Encryption](https://scalardb.scalar-labs.com/docs/latest/scalardb-cluster/encrypt-data-at-rest/)
- [Encryption configurations (optional based on your environment)](https://scalardb.scalar-labs.com/docs/latest/helm-charts/configure-custom-values-scalardb-cluster#encryption-configurations-optional-based-on-your-environment)
- [Wire Encryption](https://scalardb.scalar-labs.com/docs/latest/scalardb-cluster/encrypt-wire-communications)
- [How to Create Private Key and Certificate Files for TLS Connections in Scalar Products](https://scalardb.scalar-labs.com/docs/latest/scalar-kubernetes/HowToCreateKeyAndCertificateFiles/)
- [How to use Secret resources to pass credentials as environment variables into the properties file](https://scalardb.scalar-labs.com/docs/latest/helm-charts/use-secret-for-credentials/)

## 参考情報 (外部ドキュメント)

なし