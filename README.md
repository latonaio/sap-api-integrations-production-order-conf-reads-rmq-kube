# sap-api-integrations-production-order-conf-reads-rmq-kube
sap-api-integrations-production-order-conf-reads-rmq-kube は、外部システム(特にエッジコンピューティング環境)をSAPと統合することを目的に、SAP API で 製造記録票データ を取得するマイクロサービスです。      
sap-api-integrations-production-order-conf-reads-rmq-kube には、サンプルのAPI Json フォーマットが含まれています。    
sap-api-integrations-production-order-conf-reads-rmq-kube は、オンプレミス版である（＝クラウド版ではない）SAPS4HANA API の利用を前提としています。クラウド版APIを利用する場合は、ご注意ください。      
https://api.sap.com/api/OP_API_PROD_ORDER_CONFIRMATIO_2_SRV_0001/overview  

## 動作環境  

sap-api-integrations-production-order-conf-reads-rmq-kube は、主にエッジコンピューティング環境における動作にフォーカスしています。  
使用する際は、事前に下記の通り エッジコンピューティングの動作環境（推奨/必須）を用意してください。  
・ エッジ Kubernetes （推奨）    
・ AION のリソース （推奨)    
・ OS: LinuxOS （必須）    
・ CPU: ARM/AMD/Intel（いずれか必須）  
・ RabbitMQ on Kubernetes  
・ RabbitMQ Client  

## クラウド環境での利用

sap-api-integrations-production-order-conf-reads-rmq-kube は、外部システムがクラウド環境である場合にSAPと統合するときにおいても、利用可能なように設計されています。 

## RabbitMQ からの JSON Input

sap-api-integrations-production-order-conf-reads-rmq-kube は、Inputとして、RabbitMQ からのメッセージをJSON形式で受け取ります。 
Input の サンプルJSON は、Inputs フォルダ内にあります。  

## RabbitMQ からのメッセージ受信による イベントドリヴン の ランタイム実行

sap-api-integrations-production-order-conf-reads-rmq-kube は、RabbitMQ からのメッセージを受け取ると、イベントドリヴンでランタイムを実行します。  
AION の仕様では、Kubernetes 上 の 当該マイクロサービスPod は 立ち上がったまま待機状態で当該メッセージを受け取り、（コンテナ起動などの段取時間をカットして）即座にランタイムを実行します。　

## RabbitMQ への JSON Output

sap-api-integrations-production-order-conf-reads-rmq-kube は、Outputとして、RabbitMQ へのメッセージをJSON形式で出力します。  
Output の サンプルJSON は、Outputs フォルダ内にあります。  

## RabbitMQ の マスタサーバ環境

sap-api-integrations-production-order-conf-reads-rmq-kube が利用する RabbitMQ のマスタサーバ環境は、[rabbitmq-on-kubernetes](https://github.com/latonaio/rabbitmq-on-kubernetes) です。  
当該マスタサーバ環境は、同じエッジコンピューティングデバイスに配置されても、別の物理(仮想)サーバ内に配置されても、どちらでも構いません。

## RabbitMQ の Golang Runtime ライブラリ
sap-api-integrations-production-order-conf-reads-rmq-kube は、RabbitMQ の Golang Runtime ライブラリ として、[rabbitmq-golang-client](https://github.com/latonaio/rabbitmq-golang-client)を利用しています。

## デプロイ・稼働
sap-api-integrations-production-order-conf-reads-rmq-kube の デプロイ・稼働 を行うためには、aion-service-definitions の services.yml に、本レポジトリの services.yml を設定する必要があります。

kubectl apply - f 等で Deployment作成後、以下のコマンドで Pod が正しく生成されていることを確認してください。
```
$ kubectl get pods
```


## 本レポジトリ が 対応する API サービス
sap-api-integrations-production-order-conf-reads-rmq-kube が対応する APIサービス は、次のものです。  

* APIサービス概要説明 URL: https://api.sap.com/api/OP_API_PROD_ORDER_CONFIRMATIO_2_SRV_0001/overview  
* APIサービス名(=baseURL): API_PROD_ORDER_CONFIRMATION_2_SRV  

## 本レポジトリ に 含まれる API名
sap-api-integrations-production-order-conf-reads-rmq-kube には、次の API をコールするためのリソースが含まれています。  

* ProdnOrdConf2（製造記録票 - 確認）※製造記録票関連データを取得するために、ToMaterialMovements、ToBatchCharacteristic、と合わせて利用されます。
* ToMaterialMovements（製造記録票 - 入出庫 ※To）
* ToBatchCharacteristic（製造記録票 - ロット特性 ※To）
* ProdnOrdConfMatlDocItm（製造記録票 - 入出庫）※製造記録票関連データを取得するために、ToBatchCharacteristic、と合わせて利用されます。
* ToBatchCharacteristic（製造記録票 - ロット特性 ※To）
* ProdnOrderConfBatchCharc（製造記録票 - ロット特性）

## API への 値入力条件 の 初期値
sap-api-integrations-production-order-conf-reads-rmq-kube において、API への値入力条件の初期値は、入力ファイルレイアウトの種別毎に、次の通りとなっています。  

### SDC レイアウト

* inoutSDC.ProductionOrderConfirmation.OrderID（製造指図）
* inoutSDC.ProductionOrderConfirmation.MaterialMovements.Batch（ロット）
* inoutSDC.ProductionOrderConfirmation.ConfirmationGroup（確認グループ）
* inoutSDC.ProductionOrderConfirmation.Sequence（順序）
* inoutSDC.ProductionOrderConfirmation.OrderOperation（作業）

## SAP API Bussiness Hub の API の選択的コール

Latona および AION の SAP 関連リソースでは、Inputs フォルダ下の sample.json の accepter に取得したいデータの種別（＝APIの種別）を入力し、指定することができます。  
なお、同 accepter にAll(もしくは空白)の値を入力することで、全データ（＝全APIの種別）をまとめて取得することができます。  

* sample.jsonの記載例(1)  

accepter において 下記の例のように、データの種別（＝APIの種別）を指定します。  
ここでは、"ConfByOrderID" が指定されています。    
  
```
	"api_schema": "ProdnOrdConf2",
	"accepter": ["ConfByOrderID"],
	"production_order": "1000020",
	"deleted": false
```
  
* 全データを取得する際のsample.jsonの記載例(2)  

全データを取得する場合、sample.json は以下のように記載します。  

```
	"api_schema": "ProdnOrdConf2",
	"accepter": ["All"],
	"production_order": "1000020",
	"deleted": false
```


## 指定されたデータ種別のコール

accepter における データ種別 の指定に基づいて SAP_API_Caller 内の caller.go で API がコールされます。  
caller.go の func() 毎 の 以下の箇所が、指定された API をコールするソースコードです。  

```
func (c *SAPAPICaller) AsyncGetProductionOrderConfirmation(orderID, batch, confirmationGroup, sequence, orderOperation string, accepter []string) {
	wg := &sync.WaitGroup{}
	wg.Add(len(accepter))
	for _, fn := range accepter {
		switch fn {
		case "ConfByOrderID":
			func() {
				c.ConfByOrderID(orderID)
				wg.Done()
			}()
		case "MaterialMovements":
			func() {
				c.MaterialMovements(batch)
				wg.Done()
			}()
		case "BatchCharacteristic":
			func() {
				c.BatchCharacteristic(batch)
				wg.Done()
			}()
		case "ConfByOrderIDConfGroup":
			func() {
				c.ConfByOrderIDConfGroup(orderID, confirmationGroup)
				wg.Done()
			}()
		case "ConfByOrderIDSeqOp":
			func() {
				c.ConfByOrderIDSeqOp(orderID, sequence, orderOperation)
				wg.Done()
			}()
		default:
			wg.Done()
		}
	}

	wg.Wait()
}
```

## Output  
本マイクロサービスでは、[golang-logging-library](https://github.com/latonaio/golang-logging-library) により、以下のようなデータがJSON形式で出力されます。  
以下の sample.json の例は、SAP 製造記録票 の 確認データ が取得された結果の JSON の例です。  
以下の項目のうち、"ConfirmationGroup" ～ "to_ProdnOrdConfMatlDocItm" は、/SAP_API_Output_Formatter/type.go 内 の Type Confirmation {} による出力結果です。"cursor" ～ "time"は、golang-logging-library による 定型フォーマットの出力結果です。  

```
{
	"cursor": "/Users/latona2/bitbucket/sap-api-integrations-production-order-conf-reads/SAP_API_Caller/caller.go#L63",
	"function": "sap-api-integrations-production-order-conf-reads/SAP_API_Caller.(*SAPAPICaller).ConfByOrderID",
	"level": "INFO",
	"message": [
		{
			"ConfirmationGroup": "102",
			"ConfirmationCount": "1",
			"OrderID": "1000020",
			"Sequence": "0",
			"OrderOperation": "0020",
			"OrderSuboperation": "",
			"OrderType": "YBM1",
			"OrderOperationInternalID": "2",
			"ConfirmationText": "",
			"Language": "",
			"Material": "SG124",
			"OrderPlannedTotalQty": "100",
			"ProductionUnit": "PC",
			"FinalConfirmationType": "X",
			"IsFinalConfirmation": true,
			"OpenReservationsIsCleared": false,
			"IsReversed": false,
			"IsReversal": false,
			"APIConfHasNoGoodsMovements": false,
			"OrderConfirmationRecordType": "L40",
			"ConfirmationEntryDate": "/Date(1473897600000)/",
			"ConfirmationEntryTime": "PT08H46M24S",
			"EnteredByUser": "CB9980000078",
			"LastChangeDate": "",
			"LastChangedByUser": "",
			"ConfirmationExternalEntryDate": "",
			"ConfirmationExternalEntryTime": "PT00H00M00S",
			"EnteredByExternalUser": "",
			"ExternalSystemConfirmation": "00000000-0000-0000-0000-000000000000",
			"Plant": "1710",
			"WorkCenterTypeCode": "A",
			"WorkCenter": "TESTING",
			"CapacityCategoryCode": "",
			"CapacityRequirementSplit": 0,
			"Personnel": "0",
			"TimeRecording": "0",
			"EmployeeWageType": "",
			"EmployeeWageGroup": "",
			"BreakDurationUnit": "",
			"ConfirmedBreakDuration": "0.000",
			"EmployeeSuitability": "",
			"NumberOfEmployees": "0.00",
			"PostingDate": "/Date(1473897600000)/",
			"ConfirmedExecutionStartDate": "/Date(1473897600000)/",
			"ConfirmedExecutionStartTime": "PT08H45M30S",
			"ConfirmedSetupEndDate": "",
			"ConfirmedSetupEndTime": "PT00H00M00S",
			"ConfirmedProcessingStartDate": "",
			"ConfirmedProcessingStartTime": "PT00H00M00S",
			"ConfirmedProcessingEndDate": "",
			"ConfirmedProcessingEndTime": "PT00H00M00S",
			"ConfirmedTeardownStartDate": "",
			"ConfirmedTeardownStartTime": "PT00H00M00S",
			"ConfirmedExecutionEndDate": "/Date(1473897600000)/",
			"ConfirmedExecutionEndTime": "PT08H45M30S",
			"ConfirmationUnit": "PC",
			"ConfirmationYieldQuantity": "100",
			"ConfirmationScrapQuantity": "0",
			"ConfirmationReworkQuantity": "0",
			"VarianceReasonCode": "",
			"OpWorkQuantityUnit1": "MIN",
			"OpConfirmedWorkQuantity1": "30.000",
			"NoFurtherOpWorkQuantity1IsExpd": false,
			"OpWorkQuantityUnit2": "MIN",
			"OpConfirmedWorkQuantity2": "10.000",
			"NoFurtherOpWorkQuantity2IsExpd": false,
			"OpWorkQuantityUnit3": "MIN",
			"OpConfirmedWorkQuantity3": "10.000",
			"NoFurtherOpWorkQuantity3IsExpd": false,
			"OpWorkQuantityUnit4": "",
			"OpConfirmedWorkQuantity4": "0.000",
			"NoFurtherOpWorkQuantity4IsExpd": false,
			"OpWorkQuantityUnit5": "",
			"OpConfirmedWorkQuantity5": "0.000",
			"NoFurtherOpWorkQuantity5IsExpd": false,
			"OpWorkQuantityUnit6": "",
			"OpConfirmedWorkQuantity6": "0.000",
			"NoFurtherOpWorkQuantity6IsExpd": false,
			"BusinessProcessEntryUnit": "",
			"BusinessProcessConfirmedQty": "0.000",
			"NoFurtherBusinessProcQtyIsExpd": false,
			"to_ProdnOrdConfMatlDocItm": "https://sandbox.api.sap.com/s4hanacloud/sap/opu/odata/sap/API_PROD_ORDER_CONFIRMATION_2_SRV/ProdnOrdConf2(ConfirmationGroup='102',ConfirmationCount='1')/to_ProdnOrdConfMatlDocItm"
		},
		{
			"ConfirmationGroup": "101",
			"ConfirmationCount": "1",
			"OrderID": "1000020",
			"Sequence": "0",
			"OrderOperation": "0010",
			"OrderSuboperation": "",
			"OrderType": "YBM1",
			"OrderOperationInternalID": "1",
			"ConfirmationText": "",
			"Language": "",
			"Material": "SG124",
			"OrderPlannedTotalQty": "100",
			"ProductionUnit": "PC",
			"FinalConfirmationType": "X",
			"IsFinalConfirmation": true,
			"OpenReservationsIsCleared": false,
			"IsReversed": false,
			"IsReversal": false,
			"APIConfHasNoGoodsMovements": false,
			"OrderConfirmationRecordType": "L40",
			"ConfirmationEntryDate": "/Date(1473897600000)/",
			"ConfirmationEntryTime": "PT08H44M29S",
			"EnteredByUser": "CB9980000078",
			"LastChangeDate": "",
			"LastChangedByUser": "",
			"ConfirmationExternalEntryDate": "",
			"ConfirmationExternalEntryTime": "PT00H00M00S",
			"EnteredByExternalUser": "",
			"ExternalSystemConfirmation": "00000000-0000-0000-0000-000000000000",
			"Plant": "1710",
			"WorkCenterTypeCode": "A",
			"WorkCenter": "TECHNIC",
			"CapacityCategoryCode": "",
			"CapacityRequirementSplit": 0,
			"Personnel": "0",
			"TimeRecording": "0",
			"EmployeeWageType": "",
			"EmployeeWageGroup": "",
			"BreakDurationUnit": "",
			"ConfirmedBreakDuration": "0.000",
			"EmployeeSuitability": "",
			"NumberOfEmployees": "0.00",
			"PostingDate": "/Date(1473897600000)/",
			"ConfirmedExecutionStartDate": "/Date(1473897600000)/",
			"ConfirmedExecutionStartTime": "PT08H41M56S",
			"ConfirmedSetupEndDate": "",
			"ConfirmedSetupEndTime": "PT00H00M00S",
			"ConfirmedProcessingStartDate": "",
			"ConfirmedProcessingStartTime": "PT00H00M00S",
			"ConfirmedProcessingEndDate": "",
			"ConfirmedProcessingEndTime": "PT00H00M00S",
			"ConfirmedTeardownStartDate": "",
			"ConfirmedTeardownStartTime": "PT00H00M00S",
			"ConfirmedExecutionEndDate": "/Date(1473897600000)/",
			"ConfirmedExecutionEndTime": "PT08H41M56S",
			"ConfirmationUnit": "PC",
			"ConfirmationYieldQuantity": "100",
			"ConfirmationScrapQuantity": "0",
			"ConfirmationReworkQuantity": "0",
			"VarianceReasonCode": "",
			"OpWorkQuantityUnit1": "MIN",
			"OpConfirmedWorkQuantity1": "30.000",
			"NoFurtherOpWorkQuantity1IsExpd": false,
			"OpWorkQuantityUnit2": "MIN",
			"OpConfirmedWorkQuantity2": "10.000",
			"NoFurtherOpWorkQuantity2IsExpd": false,
			"OpWorkQuantityUnit3": "MIN",
			"OpConfirmedWorkQuantity3": "10.000",
			"NoFurtherOpWorkQuantity3IsExpd": false,
			"OpWorkQuantityUnit4": "",
			"OpConfirmedWorkQuantity4": "0.000",
			"NoFurtherOpWorkQuantity4IsExpd": false,
			"OpWorkQuantityUnit5": "",
			"OpConfirmedWorkQuantity5": "0.000",
			"NoFurtherOpWorkQuantity5IsExpd": false,
			"OpWorkQuantityUnit6": "",
			"OpConfirmedWorkQuantity6": "0.000",
			"NoFurtherOpWorkQuantity6IsExpd": false,
			"BusinessProcessEntryUnit": "",
			"BusinessProcessConfirmedQty": "0.000",
			"NoFurtherBusinessProcQtyIsExpd": false,
			"to_ProdnOrdConfMatlDocItm": "https://sandbox.api.sap.com/s4hanacloud/sap/opu/odata/sap/API_PROD_ORDER_CONFIRMATION_2_SRV/ProdnOrdConf2(ConfirmationGroup='101',ConfirmationCount='1')/to_ProdnOrdConfMatlDocItm"
		}
	],
	"time": "2022-01-03T10:03:53.273709+09:00"
}
```