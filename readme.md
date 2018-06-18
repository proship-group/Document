# LINKERをGCPに構築する手順

## 前提条件
- ローカル環境がMacまたはLinux(Ubuntuなど,VMも可)
- ローカル環境にGoogle Cloud SDKとkubectlが導入済み

## 手順
### 事前準備
- [各種サービス設定](prepare_services.md)


### GKEクラスタの作成
- [GKEクラスタのセットアップ](setup_gke_cluster.md)

### Service Accounts
- [Create a GCP Service Account](create_service_account.md)

### ソースコードの準備
- [LINKERソースコードのクローン](prepare_source_code.md)
- [Setting Environment Variables](prepare_envvars.md)

### Prerequisites
- [Create GCS Buckets](create_gcs_buckets.md)
- [Create SSD Storage Class](create_storage_class.md)

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
以下、順不同
- [notificatorのデプロイ](deploy_notificator.md)
- [importerのデプロイ](deploy_importer.md)
- [taskManagerのデプロイ](deploy_taskManager.md)
- [mailfetcherのデプロイ](deploy_mailfetcher.md)
- [actionScriptのデプロイ](deploy_actionScript.md)
- [adminToolのデプロイ](deploy_adminTool.md)
- [web-uiのデプロイ](deploy_web-ui.md)
- [b-eee-lpのデプロイ](deploy_b-eee-lp.md)
- [linker-apiのデプロイ](deploy_linker-api.md)

### ingressのデプロイ
- [ingressのデプロイ](deploy_ingress.md)

#### Extra Procedures

- [Selecting a Node-Pool](selecting_node-pool.md)