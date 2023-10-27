---
title: 단일 이벤트
description: 여정 유효성 검사의 '[!UICONTROL 단일 이벤트]' 유형을 시뮬레이션하기 위한 안내 페이지입니다.
exl-id: 314f967c-e10f-4832-bdba-901424dc2eeb
source-git-commit: 194667c26ed002be166ab91cc778594dc1f09238
workflow-type: ht
source-wordcount: '889'
ht-degree: 100%

---

# 단일 이벤트

## 따라야 할 단계 {#steps-to-follow}

>[!CONTEXTUALHELP]
>id="marketerexp_sampledata_unitaryevent"
>title="사용 방법"
>abstract="자세한 내용은 링크를 클릭하십시오."

>[!IMPORTANT]
>
>이러한 지침은 **[!UICONTROL 플레이북]**&#x200B;에 따라 변경될 수 있으므로 항상 각 **[!UICONTROL 플레이북]**&#x200B;의 샘플 데이터 섹션을 참조하십시오.

## 사전 요구 사항

* 플레이북을 사용하여 **[!UICONTROL 여정]**, **[!UICONTROL 스키마]**, **[!UICONTROL 세그먼트]**, **[!UICONTROL 메시지]** 등과 같은 인스턴스 자산을 생성합니다.

* 생성된 자산은 `Bill Of Material` 페이지에 표시됩니다.

<!-- TODO: attached image needs to change once postman is removed from UI -->
![Bill Of Material 페이지](../assets/bom-page.png)

>[!TIP]
>
>터미널을 사용하여 cURL을 실행하는 경우 cURL을 실행하기 전에 변수 값을 설정하여 개별 cURL에서 해당 값을 바꿀 필요가 없도록 할 수 있습니다.
>예를 들어 `ORG_ID=************@AdobeOrg`과 같이 설정하면 셸에서 모든 `$ORG_ID` 발생이 값으로 자동 대체되므로, cURL을 수정하지 않고도 복사하고, 붙여넣고, 실행할 수 있습니다.
>
> 이 문서 전체에서는 다음 변수가 사용됩니다.
>
> ACCESS_TOKEN
>
> API_KEY
>
> ORG_ID
>
> SANDBOX_NAME
>
> PROFILE_SCHEMA_REF
>
> PROFILE_DATASET_NAME
>
> PROFILE_DATASET_ID
>
> JOURNEY_ID
>
> PROFILE_BASE_CONNECTION_ID
>
> PROFILE_SOURCE_CONNECTION_ID
>
> PROFILE_TARGET_CONNECTION_ID
>
> PROFILE_INLET_URL
>
> CUSTOMER_MOBILE_NUMBER
>
> CUSTOMER_FIRST_NAME
>
> CUSTOMER_LAST_NAME
>
> EMAIL
>
> EVENT_SCHEMA_REF
>
> EVENT_DATASET_NAME
>
> EVENT_DATASET_ID
>
> EVENT_BASE_CONNECTION_ID
>
> EVENT_SOURCE_CONNECTION_ID
>
> EVENT_TARGET_CONNECTION_ID
>
> EVENT_INLET_URL
>
> TIMESTAMP
>
> UNIQUE_EVENT_ID

## IMS 토큰 가져오기

1. 액세스 토큰을 생성하려면 [Experience Platform API 인증 및 액세스](https://experienceleague.adobe.com/docs/experience-platform/landing/platform-apis/api-authentication.html)를 따르십시오.

## 플레이북에서 만든 여정 게시

여정을 게시하는 방법에는 2가지가 있습니다. 다음 중 하나를 선택할 수 있습니다.

1. **AJO UI 사용** - `Bill Of Material Page`에서 여정 링크를 클릭합니다. 이렇게 하면 여정 페이지로 리디렉션되며 **[!UICONTROL 게시]** 버튼을 클릭하면 여정이 게시됩니다.

   ![여정 오브젝트](../assets/journey-object.png)

1. **cURL 사용**

   1. 여정을 게시합니다. 응답에는 다음 단계에서 여정 게시 상태를 가져오는 데 필요한 작업 ID가 포함됩니다.

      ```bash
      curl --location --request POST "https://journey-private.adobe.io/authoring/jobs/journeyVersions/$JOURNEY_ID/deploy" \
      --header "Accept: */*" \
      --header "Authorization: Bearer $ACCESS_TOKEN" \
      --header "x-api-key: $API_KEY" \
      --header "x-gw-ims-org-id: $ORG_ID" \
      --header "x-sandbox-name: $SANDBOX_NAME" \
      --header "Content-Type: application/json" 
      ```

   1. 여정 게시에는 시간이 소요될 수 있으므로 상태를 확인하려면 아래 cURL을 실행하고 `response.status`가 `SUCCESS`가 될 때까지 10~15초 정도 기다려야 합니다.

      ```bash
      curl --location "https://journey-private.adobe.io/authoring/jobs/$JOB_ID" \
      --header "Authorization: Bearer $ACCESS_TOKEN" \
      --header "x-api-key: $API_KEY" \
      --header "x-gw-ims-org-id: $ORG_ID" \
      --header "x-sandbox-name: $SANDBOX_NAME" \
      --header "Content-Type: application/json"
      ```

## 고객 프로필 수집

>[!TIP]
>
>이메일 제공업체가 플러스 이메일을 지원하는 경우 이메일에 `+<variable>`을 추가하여 동일한 이메일 주소를 재사용할 수 있습니다(예: `usertest@email.com`를 `usertest+v1@email.com` 또는 `usertest+24jul@email.com`로 재사용). 이를 통해 동일한 이메일 ID를 사용하면서 매번 새로운 프로필을 가질 수 있습니다.
>
>추신: 플러스 이메일은 이메일 제공업체의 지원이 필요한 구성 가능 기능입니다. 테스트에 사용하기 전에 해당 주소로 이메일을 받을 수 있는지 확인하십시오.

1. 최초 사용자는 **[!DNL customer dataset]** 및 **[!DNL HTTP Streaming Inlet Connection]**&#x200B;을 만들어야 합니다.
1. **[!DNL customer dataset]** 및 **[!DNL HTTP Streaming Inlet Connection]**&#x200B;을 이미 생성한 경우, `5`단계로 이동합니다.
1. 아래 cURL을 실행하여 고객 프로필 데이터 세트를 만듭니다.

   ```bash
   curl --location "https://platform.adobe.io/data/foundation/catalog/dataSet" \
   --header "Authorization: Bearer $ACCESS_TOKEN" \
   --header "x-gw-ims-org-id: $ORG_ID" \
   --header "x-sandbox-name: $SANDBOX_NAME" \
   --header "x-api-key: $API_KEY" \
   --header "Content-Type: application/json" \
   --data '{
       "name": "'$PROFILE_DATASET_NAME'",
       "schemaRef": {
           "id": "'$PROFILE_SCHEMA_REF'",
           "contentType": "application/vnd.adobe.xed-full-notext+json; version=1"
       },
       "tags": {
           "unifiedProfile": [
           "enabled:true"
           ],
           "unifiedIdentity": [
           "enabled:true"
           ]
       },
       "fileDescription": {
           "persisted": true,
           "containerFormat": "parquet",
           "format": "parquet"
       }
   }'
   ```

   응답 포맷은 `"@/dataSets/<PROFILE_DATASET_ID>"`입니다.

1. 다음 단계를 사용하여 **[!DNL HTTP Streaming Inlet Connection]**&#x200B;을 만듭니다.
   1. 기본 연결을 만듭니다.

      ```bash
      curl --location "https://platform.adobe.io/data/foundation/flowservice/connections?Cache-Control=no-cache" \
      --header "Authorization: Bearer $ACCESS_TOKEN" \
      --header "x-gw-ims-org-id: $ORG_ID" \
      --header "x-sandbox-name: $SANDBOX_NAME" \
      --header "x-api-key: $API_KEY" \
      --header "Content-Type: application/json" \
      --data '{
          "name": "AbandonedCartProduct_Base_ConnectionForCustomerProfile_1694458293",
          "description": "Marketer Playground Playbook-Validation Customer Profile Base Connection 1",
          "auth": {
              "specName": "Streaming Connection",
              "params": {
                  "dataType": "xdm"
              }
          },
          "connectionSpec": {
              "id": "bc7b00d6-623a-4dfc-9fdb-f1240aeadaeb",
              "version": "1.0"
          }
      }'
      ```

      응답에서 기본 연결을 가져와 다음 cURL에서 `PROFILE_BASE_CONNECTION_ID` 대신 사용합니다.

   1. 소스 연결을 만듭니다.

      ```bash
      curl --location "https://platform.adobe.io/data/foundation/flowservice/sourceConnections" \
      --header "Authorization: Bearer $ACCESS_TOKEN" \
      --header "x-gw-ims-org-id: $ORG_ID" \
      --header "x-sandbox-name: $SANDBOX_NAME" \
      --header "Content-Type: application/json" \
      --header "x-api-key: $API_KEY" \
      --data '{
          "name": "AbandonedCartProduct_Source_ConnectionForCustomerProfile_1694458318",
          "description": "Marketer Playground Playbook-Validation Customer Profile Source Connection 1",
          "baseConnectionId": "'$PROFILE_BASE_CONNECTION_ID'",
          "connectionSpec": {
              "id": "bc7b00d6-623a-4dfc-9fdb-f1240aeadaeb",
              "version": "1.0"
          }
      }'
      ```

      응답에서 소스 연결 ID를 가져와 `PROFILE_SOURCE_CONNECTION_ID` 대신 사용합니다.

   1. 대상 연결을 만듭니다.

      ```bash
      curl --location "https://platform.adobe.io/data/foundation/flowservice/targetConnections" \
      --header "Authorization: Bearer $ACCESS_TOKEN" \
      --header "x-gw-ims-org-id: $ORG_ID" \
      --header "x-sandbox-name: $SANDBOX_NAME" \
      --header "Content-Type: application/json" \
      --header "x-api-key: $API_KEY" \
      --data '{
          "name": "AbandonedCartProduct_Target_ConnectionForCustomerProfile_1694458407",
          "description": "Marketer Playground Playbook-Validation Customer Profile Target Connection 1",
          "data": {
              "format": "parquet_xdm",
              "schema": {
                  "version": "application/vnd.adobe.xed-full+json;version=1",
                  "id": "'$PROFILE_SCHEMA_REF'"
              },
              "properties": null
          },
          "connectionSpec": {
              "id": "c604ff05-7f1a-43c0-8e18-33bf874cb11c",
              "version": "1.0"
          },
          "params": {
              "dataSetId": "'$PROFILE_DATASET_ID'"
          }
      }'
      ```

      응답에서 대상 연결 ID를 가져와 `PROFILE_TARGET_CONNECTION_ID` 대신 사용합니다.

   1. 데이터 흐름을 만듭니다.

      ```bash
      curl --location "https://platform.adobe.io/data/foundation/flowservice/flows" \
      --header "Authorization: Bearer $ACCESS_TOKEN" \
      --header "x-gw-ims-org-id: $ORG_ID" \
      --header "x-sandbox-name: $SANDBOX_NAME" \
      --header "Content-Type: application/json" \
      --header "x-api-key: $API_KEY" \
      --data '{
          "name": "AbandonedCartProduct_Dataflow_ForCustomerCustomerProfile_1694460528",
          "description": "Marketer Playground Playbook-Validation Customer Profile Dataflow 1",
          "flowSpec": {
              "id": "d8a6f005-7eaf-4153-983e-e8574508b877",
              "version": "1.0"
          },
          "sourceConnectionIds": [
              "'$PROFILE_SOURCE_CONNECTION_ID'"
          ],
          "targetConnectionIds": [
              "'$PROFILE_TARGET_CONNECTION_ID'"
          ]
      }'
      ```

   1. 기본 연결을 가져옵니다. 결과에는 프로필 데이터를 보내는 데 필요한 inletUrl이 포함됩니다.

      ```bash
      curl --location "https://platform.adobe.io/data/foundation/flowservice/connections/$PROFILE_BASE_CONNECTION_ID" \
      --header "Authorization: Bearer $ACCESS_TOKEN" \
      --header "x-gw-ims-org-id: $ORG_ID" \
      --header "x-sandbox-name: $SANDBOX_NAME" \
      --header "Content-Type: application/json" \
      --header "x-api-key: $API_KEY"
      ```

      응답에서 inletUrl을 가져와 `PROFILE_INLET_URL` 대신 사용합니다.

1. 이 단계에서 사용자는 `PROFILE_DATASET_ID` 및 `PROFILE_INLET_URL` 값이 있어야 합니다. 해당 값이 없는 경우 `3`단계 또는 `4`단계를 각각 참조하십시오.
1. 고객을 수집하려면 사용자는 아래 cURL의 `CUSTOMER_MOBILE_NUMBER`, `CUSTOMER_FIRST_NAME`, `CUSTOMER_LAST_NAME` 및 `EMAIL`을 바꿔야 합니다.

   1. `CUSTOMER_MOBILE_NUMBER`는 `+1 000-000-0000` 등과 같은 휴대폰 번호가 될 것입니다.
   1. `CUSTOMER_FIRST_NAME`은 사용자의 이름이 될 것입니다.
   1. `CUSTOMER_LAST_NAME`은 사용자의 성이 될 것입니다.
   1. `EMAIL`은 사용자의 이메일 주소가 될 것입니다. 이는 새로운 프로필을 수집할 수 있도록 고유한 이메일 ID를 사용해야 합니다.

1. 마지막으로 cURL을 실행하여 고객 프로필을 수집합니다. 확인하고자 하는 채널에 따라 `email`, `sms` 또는 `push`에 `body.xdmEntity.consents.marketing.preferred`를 업데이트합니다. 또한 해당하는 `val`을 `y`로 설정합니다.

   ```bash
   curl --location "$PROFILE_INLET_URL?synchronousValidation=true" \
   --header 'Content-Type: application/json' \
   --data-raw '{
       "header": {
           "schemaRef": {
               "id": "'$PROFILE_SCHEMA_REF'",
               "contentType": "application/vnd.adobe.xed-full+json;version=1.0"
           },
           "imsOrgId": "'$ORG_ID'",
           "datasetId": "'$PROFILE_DATASET_ID'",
           "source": {
               "name": "Streaming dataflow - 1694460605"
           }
       },
       "body": {
           "xdmMeta": {
               "schemaRef": {
                   "id": "'$PROFILE_SCHEMA_REF'",
                   "contentType": "application/vnd.adobe.xed-full+json;version=1.0"
               }
           },
           "xdmEntity": {
           "consents": {
               "marketing": {
                   "preferred": "email",
                   "email": {
                       "val": "y"
                   },
                   "push": {
                       "val": "n"
                   },
                   "sms": {
                       "val": "n"
                   }
               }
           },
           "mobilePhone": {
               "number": "'$CUSTOMER_MOBILE_NUMBER'",
               "status": "active"
           },
           "person": {
               "name": {
               "firstName": "'$CUSTOMER_FIRST_NAME'",
               "lastName": "'$CUSTOMER_LAST_NAME'"
               }
           },
           "personalEmail": {
               "address": "'$EMAIL'"
           },
           "testProfile": false
           }
       }
   }'
   ```

## 여정 트리거 이벤트 수집

1. 최초 사용자는 **[!DNL event dataset]** 및 **[!DNL HTTP Streaming Inlet Connection for events]**&#x200B;을 만들어야 합니다.
1. **[!DNL event dataset]** 및 **[!DNL HTTP Streaming Inlet Connection for events]**&#x200B;을 이미 생성한 경우, `5`단계로 이동합니다.
1. 아래 cURL을 실행하여 이벤트 데이터 세트를 만듭니다.

   ```bash
   curl --location "https://platform.adobe.io/data/foundation/catalog/dataSet" \
   --header "Authorization: Bearer $ACCESS_TOKEN" \
   --header "x-gw-ims-org-id: $ORG_ID" \
   --header "x-sandbox-name: $SANDBOX_NAME" \
   --header "x-api-key: $API_KEY" \
   --header "Content-Type: application/json" \
   --data '{
       "name": "'$EVENT_DATASET_NAME'",
       "schemaRef": {
           "id": "'$EVENT_SCHEMA_REF'",
           "contentType": "application/vnd.adobe.xed-full-notext+json; version=1"
       },
       "tags": {
           "unifiedProfile": [
               "enabled:true"
           ],
           "unifiedIdentity": [
               "enabled:true"
           ]
       },
       "fileDescription": {
           "persisted": true,
           "containerFormat": "parquet",
           "format": "parquet"
       }
   }'
   ```

   응답 포맷은 `"@/dataSets/<EVENT_DATASET_ID>"`입니다.

1. 다음 단계를 사용하여 **[!DNL HTTP Streaming Inlet Connection for events]**를 만듭니다.
   <!-- TODO: Is the name unique? If so, we may need to generate and provide in variables.txt-->
   1. 기본 연결을 만듭니다.

      ```bash
      curl --location "https://platform.adobe.io/data/foundation/flowservice/connections?Cache-Control=no-cache" \
      --header "Authorization: Bearer $ACCESS_TOKEN" \
      --header "x-gw-ims-org-id: $ORG_ID" \
      --header "x-sandbox-name: $SANDBOX_NAME" \
      --header "x-api-key: $API_KEY" \
      --header "Content-Type: application/json" \
      --data '{
          "name": "AbandonedCartProduct_Base_ConnectionForAEPDemoSchema_1694461448",
          "description": "Marketer Playground Playbook-Validation AEP Demo Schema Base Connection 1",
          "auth": {
              "specName": "Streaming Connection",
              "params": {
                  "dataType": "xdm"
              }
          },
          "connectionSpec": {
              "id": "bc7b00d6-623a-4dfc-9fdb-f1240aeadaeb",
              "version": "1.0"
          }
      }'
      ```

      응답에서 기본 연결 ID를 가져와 `EVENT_BASE_CONNECTION_ID` 대신 사용합니다.

   1. 소스 연결을 만듭니다.

      ```bash
      curl --location "https://platform.adobe.io/data/foundation/flowservice/sourceConnections" \
      --header "Authorization: Bearer $ACCESS_TOKEN" \
      --header "x-gw-ims-org-id: $ORG_ID" \
      --header "x-sandbox-name: $SANDBOX_NAME" \
      --header "x-api-key: $API_KEY" \
      --header "Content-Type: application/json" \
      --data '{
          "name": "AbandonedCartProduct_Source_ConnectionForAEPDemoSchema_1694461464",
          "description": "Marketer Playground Playbook-Validation AEP Demo Schema Source Connection 1",
          "baseConnectionId": "'$EVENT_BASE_CONNECTION_ID'",
          "connectionSpec": {
              "id": "bc7b00d6-623a-4dfc-9fdb-f1240aeadaeb",
              "version": "1.0"
          }
      }'
      ```

      응답에서 소스 연결 ID를 가져와 `EVENT_SOURCE_CONNECTION_ID` 대신 사용합니다.

   1. 대상 연결을 만듭니다.

      ```bash
      curl --location "https://platform.adobe.io/data/foundation/flowservice/sourceConnections" \
      --header "Authorization: Bearer $ACCESS_TOKEN" \
      --header "x-gw-ims-org-id: $ORG_ID" \
      --header "x-sandbox-name: $SANDBOX_NAME" \
      --header "x-api-key: $API_KEY" \
      --header "Content-Type: application/json" \
      --data '{
          "name": "AbandonedCartProduct_Target_ConnectionForAEPDemoSchema_1694802667",
          "description": "Marketer Playground Playbook-Validation AEP Demo Schema Target Connection 1",
          "data": {
              "format": "parquet_xdm",
              "schema": {
                  "version": "application/vnd.adobe.xed-full+json;version=1",
                  "id": "'$EVENT_SCHEMA_REF'"
              },
              "properties": null
          },
          "connectionSpec": {
              "id": "c604ff05-7f1a-43c0-8e18-33bf874cb11c",
              "version": "1.0"
          },
          "params": {
              "dataSetId": "'$EVENT_DATASET_ID'"
          }
      }'
      ```

      응답에서 대상 연결 ID를 가져와 `EVENT_TARGET_CONNECTION_ID` 대신 사용합니다.

   1. 데이터 흐름을 만듭니다.

      ```bash
      curl --location "https://platform.adobe.io/data/foundation/flowservice/flows" \
      --header "Authorization: Bearer $ACCESS_TOKEN" \
      --header "x-gw-ims-org-id: $ORG_ID" \
      --header "x-sandbox-name: $SANDBOX_NAME" \
      --header "x-api-key: $API_KEY" \
      --header "Content-Type: application/json" \
      --data '{
          "name": "AbandonedCartProduct_Dataflow_ForCustomerAEPDemoSchema_1694461564",
          "description": "Marketer Playground Playbook-Validation AEP Demo Schema Dataflow 1",
          "flowSpec": {
              "id": "d8a6f005-7eaf-4153-983e-e8574508b877",
              "version": "1.0"
          },
          "sourceConnectionIds": [
              "'$EVENT_SOURCE_CONNECTION_ID'"
          ],
          "targetConnectionIds": [
              "'$EVENT_TARGET_CONNECTION_ID'"
          ]
      }'
      ```

   1. 기본 연결을 가져옵니다. 결과에는 프로필 데이터를 보내는 데 필요한 inletUrl이 포함됩니다.

   ```bash
   curl --location "https://platform.adobe.io/data/foundation/flowservice/connections/$EVENT_BASE_CONNECTION_ID" \
       --header "Authorization: Bearer $ACCESS_TOKEN" \
       --header "x-gw-ims-org-id: $ORG_ID" \
       --header "x-sandbox-name: $SANDBOX_NAME" \
       --header "x-api-key: $API_KEY" \
       --header "Content-Type: application/json" 
   ```

   응답에서 inletUrl을 가져와 `EVENT_INLET_URL` 대신 사용합니다.

1. 이 단계에서 사용자는 `EVENT_DATASET_ID` 및 `EVENT_INLET_URL` 값이 있어야 합니다. 해당 값이 없는 경우 `3`단계 또는 `4`단계를 각각 참조하십시오.
1. 이벤트를 수집하려면 사용자가 아래 cURL의 요청 본문에서 `TIMESTAMP`를 변경해야 합니다.

   1. `body.xdmEntity`를 다운로드된 이벤트 json 내용으로 바꿉니다.
   1. `TIMESTAMP`는 타임스탬프를 사용하여 이벤트 발생 시간을 UTC 시간대로 표시합니다(예: `2023-09-05T23:57:00.071+00:00`).
   1. 변수 `UNIQUE_EVENT_ID`에 고유 값을 설정합니다.

   ```bash
   curl --location "$EVENT_INLET_URL?synchronousValidation=true" \
   --header 'Content-Type: application/json' \
   --data-raw '{
       "header": {
           "schemaRef": {
               "id": "'$EVENT_SCHEMA_REF'",
               "contentType": "application/vnd.adobe.xed-full+json;version=1.0"
           },
           "imsOrgId": "'$ORG_ID'",
           "datasetId": "'$EVENT_DATASET_ID'",
           "source": {
               "name": "Streaming dataflow - 8/31/2023 9:04:25 PM"
           }
       },
       "body": {
           "xdmMeta": {
               "schemaRef": {
                   "id": "'$EVENT_SCHEMA_REF'",
                   "contentType": "application/vnd.adobe.xed-full+json;version=1.0"
               }
           },
           "xdmEntity": {
               "endUserIDs": {
                   "_experience": {
                       "aaid": {
                           "id": "'$EMAIL'"
                       },
                       "emailid": {
                           "id": "'$EMAIL'"
                       }
                   }
               },
               "_experience": {
                   "analytics": {
                       "customDimensions": {
                           "eVars": {
                           "eVar235": "AC11147"
                           }
                       }
                   }
               },
               "_id": "'$UNIQUE_EVENT_ID'",
               "commerce": {
                   "productListAdds": {
                       "value": 11498
                   }
               },
               "eventType": "commerce.productListAdds",
               "productListItems": [
                   {
                       "_id": "ACS1620",
                       "SKU": "P1",
                       "_experience": {
                           "analytics": {
                           "customDimensions": {
                               "eVars": {
                                   "eVar1": "Pants"
                               }
                           }
                           }
                       },
                       "currencyCode": "USD",
                       "name": "Sample value",
                       "priceTotal": 30841.13,
                       "product": "https://ns.adobe.com/xdm/common/uri",
                       "productAddMethod": "Sample value",
                       "quantity": 1
                   },
                   {
                       "_id": "ACS1729",
                       "SKU": "P2",
                       "_experience": {
                           "analytics": {
                               "customDimensions": {
                                   "eVars": {
                                       "eVar1": "Galliano"
                                   }
                               }
                           }
                       },
                       "currencyCode": "USD",
                       "name": "Sample value",
                       "priceTotal": 20841.13,
                       "product": "https://ns.adobe.com/xdm/common/uri",
                       "productAddMethod": "Sample value",
                       "quantity": 2
                   }
               ],
               "timestamp": "'$TIMESTAMP'",
               "web": {
                   "webInteraction": {
                       "URL": "https://experienceleague.adobe.com/docs/experience-platform/edge/data-collection/collect-commerce-data.html?lang=en",
                       "name": "Sample value",
                       "region": "Sample value"
                   },
                   "webPageDetails": {
                       "URL": "https://experienceleague.adobe.com/docs/experience-platform/edge/data-collection/collect-commerce-data.html?lang=en",
                       "isErrorPage": false,
                       "isHomePage": false,
                       "name": "Sample value",
                       "pageViews": {
                           "id": "Sample value",
                           "value": 1
                       },
                       "server": "Sample value",
                       "siteSection": "Sample value",
                       "viewName": "Sample value"
                   },
                   "webReferrer": {
                   "URL": "Sample value",
                   "type": "internal"
                   }
               }
           }
       }
   }'
   ```

## 최종 유효성 검사

**[!DNL Ingest the Customer Profile]** `8`단계에서 사용되는 선택한 선호 채널에 메시지가 표시되어야 합니다.

* 선호 채널이 `customer_country_code` 및 `customer_mobile_no`의 `sms`인 경우 `SMS`
* 선호 채널이 `email`의 `email`인 경우 `Email`

또는 `Journey Report`를 확인할 수 있습니다. 확인하려면 `Bill of Materials page`의 `Journey Object`를 클릭하십시오. 그러면 `Journey Details page`로 리디렉션됩니다.

게시된 여정의 경우 사용자에게 **[!UICONTROL 보고서 보기]** 버튼이 표시되어야 합니다.
![여행 보고서 페이지](../assets/journey-report-page.png)


## 정리

`Journey`의 여러 인스턴스를 동시에 실행하지 마십시오. 유효성 검사가 완료된 후 유효성 검사에 대해서만 실행하는 경우 여정을 중지하시기 바랍니다.
