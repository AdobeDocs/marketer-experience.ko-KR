---
title: 단일 이벤트
description: 여정 유효성 검사의 '[!UICONTROL 단일 이벤트]' 유형을 시뮬레이션하기 위한 안내 페이지입니다.
source-git-commit: 0a52611530513fc036ef56999fde36a32ca0f482
workflow-type: ht
source-wordcount: '749'
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

* Postman 소프트웨어가 설치되어 있어야 합니다.
* 플레이북을 사용하여 **[!UICONTROL 여정]**, **[!UICONTROL 스키마]**, **[!UICONTROL 세그먼트]**, **[!UICONTROL 메시지]** 등과 같은 인스턴스 자산을 생성합니다.

생성된 자산은 `Bill Of Material` 페이지에 표시됩니다.

![Bill Of Material 페이지](../assets/bom-page.png)

## 필요한 컬렉션으로 Postman 준비

1. **[!UICONTROL 사용 사례 플레이북]** 애플리케이션을 방문하십시오.
1. **[!UICONTROL 플레이북]** 세부 정보 페이지를 방문하려면 해당 **[!UICONTROL 플레이북]** 카드를 클릭합니다.
1. **[!UICONTROL Bill Of Material]** 페이지를 방문하여 **[!UICONTROL 샘플 데이터]** 섹션을 찾습니다.
1. UI에서 해당 버튼을 클릭하여 `postman.json`을 다운로드합니다.
1. **[!DNL Postman Software]**&#x200B;에서 `postman.json`을 가져옵니다.
1. 이 유효성 검사를 위한 전용 Postman 환경을 생성합니다(예: `Adobe <PLAYBOOK_NAME>`).

## IMS 토큰 가져오기

>[!NOTE]
>
>모든 환경 변수는 대/소문자를 구분하므로 항상 정확한 변수 이름을 사용해야 합니다.

1. 액세스 토큰을 생성하려면 [Experience Platform API 인증 및 액세스](https://experienceleague.adobe.com/docs/experience-platform/landing/platform-apis/api-authentication.html)를 따르십시오.
1. `ACCESS_TOKEN`이라는 이름의 환경 변수에 액세스 토큰 값을 저장합니다.
1. `API_KEY`, `IMS_ORG` 및 `SANDBOX_NAME`와 같은 기타 인증 관련 값을 환경 변수에 저장합니다.

>[!IMPORTANT]
>
>Postman에서 API를 실행하기 전에 필요한 모든 환경 변수를 추가해야 합니다.

## 플레이북에서 만든 여정 게시

여정을 게시하는 방법에는 2가지가 있습니다. 다음 중 하나를 선택할 수 있습니다.

1. **AJO UI 사용** - `Bill Of Material Page`에서 여정 링크를 클릭합니다. 이렇게 하면 여정 페이지로 리디렉션되며 **[!UICONTROL 게시]** 버튼을 클릭하면 여정이 게시됩니다.

   ![여정 오브젝트](../assets/journey-object.png)

1. **Postman API 사용**

   1. **[!DNL Journey Publish]** > **[!DNL Queue journey publish job]**&#x200B;에서 **[!DNL Publish Journey]** 요청을 트리거합니다.
   1. 여정 게시에는 시간이 소요될 수 있으므로 상태를 확인하려면 여정 게시 상태 API를 실행하고 `response.status`가 `SUCCESS`가 될 때까지 10~15초 정도 기다려야 합니다.

   >[!NOTE]
   >
   >모든 환경 변수는 대/소문자를 구분하므로 항상 정확한 변수 이름을 사용해야 합니다.

## 고객 프로필 수집

>[!TIP]
>
>이메일에 `+<variable>`을 추가하여 동일한 이메일 주소를 재사용할 수 있습니다. 예를 들어 `usertest@email.com`은 `usertest+v1@email.com` 또는 `usertest+24jul@email.com`으로 재사용될 수 있습니다. 이를 통해 동일한 이메일 ID를 사용하면서 매번 새로운 프로필을 가질 수 있습니다.

1. 최초 사용자는 **[!DNL customer dataset]** 및 **[!DNL HTTP Streaming Inlet Connection]**&#x200B;을 만들어야 합니다.
1. **[!DNL customer dataset]** 및 **[!DNL HTTP Streaming Inlet Connection]**&#x200B;을 이미 생성한 경우, `5`단계로 이동합니다.
1. **[!DNL Customer Profile Ingestion]** > **[!DNL Create Customer Profile InletId]** > **[!DNL Create Dataset]**&#x200B;를 트리거하여 **[!DNL customer dataset]**&#x200B;을 생성합니다. 그러면 Postman 환경 변수에 `CustomerProfile_dataset_id`가 저장됩니다.
1. **[!DNL HTTP Streaming Inlet Connection]**&#x200B;을 만들고 **[!DNL Customer Profile Ingestion > Create Customer Profile InletId]** 아래에서 Postman API를 사용하십시오.

   1. `CustomerProfile_dataset_id`은 Postman 환경 변수에서 사용할 수 있어야 합니다. 그렇지 않은 경우 `3`단계를 참조하십시오.
   1. **[!DNL `CREATE Base Connection`]**&#x200B;에서 [!DNL create base connection]으로 트리거합니다.
   1. **[!DNL `CREATE Source Connection`]**&#x200B;에서 [!DNL create source connection]으로 트리거합니다.
   1. **[!DNL `CREATE Target Connection`]**&#x200B;에서 [!DNL create target connection]으로 트리거합니다.
   1. **[!DNL `CREATE Dataflow`]**&#x200B;에서 [!DNL create dataflow]으로 트리거합니다.
   1. **[!DNL `GET Base Connection`]** 트리거 - 이 옵션을 선택하면 `CustomerProfile_inlet_id`가 Postman 환경 변수에 자동으로 저장됩니다.

1. 이 단계에서 Postman 환경 변수에 `CustomerProfile_dataset_id` 및 `CustomerProfile_inlet_id`가 있어야 합니다. 그렇지 않은 경우 각각 `3` 또는 `4`단계를 참조하십시오.
1. 사용자가 고객을 수집하려면 `customer_country_code`, `customer_mobile_no`, `customer_first_name`, `customer_last_name` 및 `email`을 Postman 환경 변수에 저장해야 합니다.

   1. `customer_country_code`는 `91` 또는 `1` 등과 같은 휴대폰 번호의 국가 코드입니다.
   1. `customer_mobile_no`는 `9987654321` 등과 같은 휴대폰 번호가 될 것입니다.
   1. `customer_first_name`은 사용자의 이름이 될 것입니다.
   1. `customer_last_name`은 사용자의 성이 될 것입니다.
   1. `email`은 사용자의 이메일 주소가 될 것입니다. 이는 새로운 프로필을 수집할 수 있도록 고유한 이메일 ID를 사용해야 합니다.

1. Postman 요청 **[!DNL Customer Ingestion]** > **[!DNL Customer Streaming Ingestion]**&#x200B;를 업데이트하여 고객의 선호 채널을 변경합니다. 기본적으로 [!DNL `email`]이 요청으로 구성되어 있습니다.

   ```js
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
   }
   ```

1. 선호 채널을 `sms` 또는 `push`로 변경하고 각 채널 값을 `y`로, `n`을 다른 값으로 변경합니다.

   ```js
   "consents": {
       "marketing": {
           "preferred": "sms",
           "email": {
               "val": "n"
           },
           "push": {
               "val": "n"
           },
           "sms": {
               "val": "y"
           }
       }
   }
   ```

1. 마지막으로 **[!DNL `Customer Profile Ingestion > Customer Profile Streaming Ingestion`]**&#x200B;을 트리거하여 고객 프로필을 수집합니다.

## 이벤트 수집

1. 최초 사용자는 **[!DNL event dataset]** 및 **[!DNL HTTP Streaming Inlet Connection for events]**&#x200B;을 만들어야 합니다.
1. **[!DNL event dataset]** 및 **[!DNL HTTP Streaming Inlet Connection for events]**&#x200B;을 이미 생성한 경우, `5`단계로 이동합니다.
1. **[!DNL `Schemas Data Ingestion > AEP Demo Schema Ingestion > Create AEP Demo Schema InletId > Create Dataset`]**&#x200B;를 트리거하여 **[!DNL event dataset]**&#x200B;를 생성합니다. 그러면 Postman 환경 변수에 `AEPDemoSchema_dataset_id`가 저장됩니다.
1. **[!DNL HTTP Streaming Inlet Connection for events]**&#x200B;을 만들고 **[!DNL Schemas Data Ingestion]** > **[!DNL AEP Demo Schema Ingestion]** > **[!DNL Create AEP Demo Schema InletId]**&#x200B;에서 Postman API를 사용하십시오.

   1. `AEPDemoSchema_dataset_id`은 Postman 환경 변수에서 사용할 수 있어야 합니다. 그렇지 않은 경우 `3`단계를 참조하십시오.
   1. **[!DNL `CREATE Base Connection`]**&#x200B;에서 [!DNL create base connection]으로 트리거합니다.
   1. **[!DNL `CREATE Source Connection`]**&#x200B;에서 [!DNL create source connection]으로 트리거합니다.
   1. **[!DNL `CREATE Target Connection`]**&#x200B;에서 [!DNL create target connection]으로 트리거합니다.
   1. **[!DNL `CREATE Dataflow`]**&#x200B;에서 [!DNL create dataflow]으로 트리거합니다.
   1. **[!DNL `GET Base Connection`]** 트리거 - 이 옵션을 선택하면 `AEPDemoSchema_inlet_id`가 Postman 환경 변수에 자동으로 저장됩니다.

1. 이 단계에서 Postman 환경 변수에 `AEPDemoSchema_dataset_id` 및 `AEPDemoSchema_inlet_id`가 있어야 합니다. 그렇지 않은 경우 각각 `3` 또는 `4`단계를 참조하십시오.
1. 이벤트를 수집하려면 사용자가 Postman에서 **[!DNL Schemas Data Ingestion]** > **[!DNL AEP Demo Schema Ingestion]** > **[!DNL AEP Demo Schema Streaming Ingestion]** 요청 본문의 시간 변수 `timestamp`를 변경해야 합니다.

   1. `timestamp`는 이벤트 발생 시간입니다. 현재 타임스탬프(예: `2023-07-21T16:37:52+05:30`)를 사용하여 필요에 따라 시간대를 조정하십시오.

1. **[!DNL Schemas Data Ingestion > AEP Demo Schema Ingestion > AEP Demo Schema Streaming Ingestion]**&#x200B;을 트리거하여 이벤트를 수집하고 여정이 트리거될 수 있도록 합니다.

## 최종 유효성 검사

**[!DNL Ingest the Customer Profile]** `8`단계에서 사용되는 선택한 선호 채널에 메시지가 표시되어야 합니다.

* 선호 채널이 `customer_country_code` 및 `customer_mobile_no`의 `sms`인 경우 `SMS`
* 선호 채널이 `email`의 `email`인 경우 `Email`

또는 `Journey Report`를 확인할 수 있습니다. 확인하려면 `Bill of Materials page`의 `Journey Object`를 클릭하십시오. 그러면 `Journey Details page`로 리디렉션됩니다.

게시된 여정의 경우 사용자에게 **[!UICONTROL 보고서 보기]** 버튼이 표시되어야 합니다.
![여행 보고서 페이지](../assets/journey-report-page.png)


## 정리

`Journey`의 여러 인스턴스를 동시에 실행하지 마십시오. 유효성 검사가 완료된 후 유효성 검사에 대해서만 실행하는 경우 여정을 중지하시기 바랍니다.
