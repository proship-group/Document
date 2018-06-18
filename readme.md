# LINKERをGCPに構築する手順

## 前提条件
- ローカル環境がMacまたはLinux(Ubuntuなど,VMも可)
- ローカル環境にGoogle Cloud SDKとkubectlが導入済み

## 手順

### 環境の廃棄化処理
- [環境の廃棄処理](Linkerマイクロサービスの廃棄化処理.md)

### 前期準備の準備

- [Setting Environment Variables](prepare_envvars.md)

### ミドルウェアのデプロイ
- [MongoDBのデプロイ](deploy_mongodb.md)
- [Neo4jのデプロイ](deploy_neo4j.md)
- [Elasticsearchのデプロイ](deploy_elasticsearch.md)
- [MySQLのデプロイ](deploy_mysql.md)
- [redisのデプロイ](deploy_redis.md)
- [NSQのデプロイ](deploy_nsq.md)

### configctlのデプロイ
- [configctlのデプロイ](deploy_configctl.md)

### apicoreのデプロイ
- [apicoreのデプロイ](deploy_apicore.md)

### atagoのセットアップ
- [atagoのセットアップ](setup_atago.md)

### その他のマイクロサービスのデプロイ

- [adminToolのデプロイ](deploy_adminTool.md)

以下のmicroserviceは、atagoを使ってデプロイしてください。

以下、順不同
- [notificatorのデプロイ](deploy_notificator.md)
- [importerのデプロイ](deploy_importer.md)
- [taskManagerのデプロイ](deploy_taskManager.md)
- [mailfetcherのデプロイ](deploy_mailfetcher.md)
- [actionScriptのデプロイ](deploy_actionScript.md)
- [web-uiのデプロイ](deploy_web-ui.md)
- [b-eee-lpのデプロイ](deploy_b-eee-lp.md)
- [linker-apiのデプロイ](deploy_linker-api.md)

### ingressのデプロイ
- [ingressのデプロイ](deploy_ingress.md)

#### その他の手順

- [GKEクラスタのセットアップ](setup_gke_cluster.md)
- [Create a GCP Service Account](create_service_account.md)
- [Create GCS Buckets](create_gcs_buckets.md)
- [Create SSD Storage Class](create_storage_class.md)
- [各種サービス設定](prepare_services.md)
- [LINKERソースコードのクローン](prepare_source_code.md)
- [Selecting a Node-Pool](selecting_node-pool.md)
- [`devops`VMへの接続設定](`devops`VMへの接続設定.md)