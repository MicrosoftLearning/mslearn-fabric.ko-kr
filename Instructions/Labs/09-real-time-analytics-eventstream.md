---
lab:
  title: Microsoft Fabric에서 Eventstream 시작
  module: Get started with Eventstream in Microsoft Fabric
---
# RTA(실시간 분석)에서 Eventstream 시작

Eventstream은 코드가 없는 환경을 사용하여 실시간 이벤트를 캡처, 변환 및 다양한 대상으로 라우팅하는 Microsoft Fabric의 기능입니다. 변환이 필요한 경우 이벤트 데이터 원본, 라우팅 대상 및 이벤트 프로세서를 eventstream에 추가할 수 있습니다. Microsoft Fabric의 EventStore는 클러스터의 이벤트를 기본 지정한 시점에 클러스터 또는 워크로드의 상태를 이해하는 방법을 제공하는 모니터링 옵션입니다. 클러스터의 각 엔터티 및 엔터티 형식에 사용할 수 있는 이벤트에 대해 EventStore 서비스를 쿼리할 수 있습니다. 즉, 클러스터, 노드, 애플리케이션, 서비스, 파티션 및 파티션 복제본(replica) 같은 다양한 수준에서 이벤트를 쿼리할 수 있습니다. EventStore 서비스에는 클러스터의 이벤트 간에 상관 관계를 지정하는 기능도 있습니다. 서로 영향을 줄 수 있는 여러 엔터티에서 동시에 작성된 이벤트를 살펴보면 EventStore 서비스는 이러한 이벤트를 연결하여 클러스터에서 활동의 원인을 식별하는 데 도움이 될 수 있습니다. Microsoft Fabric 클러스터의 모니터링 및 진단 또 다른 옵션은 EventFlow를 사용하여 이벤트를 집계하고 수집하는 것입니다.

이 랩을 완료하는 데 약 **30**분이 소요됩니다.

> **참고**: 이 연습을 완료하려면 Microsoft Fabric 평가판[이 필요합니다](https://learn.microsoft.com/fabric/get-started/fabric-trial).

## 작업 영역 만들기

패브릭에서 데이터를 사용하기 전에 패브릭 평가판을 사용하도록 설정된 작업 영역을 만듭니다.

1. Microsoft Fabric에 로그인[하고 Power BI**를 선택합니다**.`https://app.fabric.microsoft.com`](https://app.fabric.microsoft.com)
2. 왼쪽 메뉴 모음에서 **작업 영역**을 선택합니다(아이콘은 와 유사함).
3. 선택한 이름으로 새 작업 영역을 만들고 패브릭 용량(*평가판*, *프리미엄* 또는 *패브릭*)이 포함된 라이선스 모드를 선택합니다.
4. 새 작업 영역이 열리면 다음과 같이 비어 있어야 합니다.

   ![Power BI의 빈 작업 영역 스크린샷](./Images/new-workspace.png)
5. Power BI 포털의 왼쪽 아래에서 Power BI 아이콘을 **선택하고 실시간 분석 환경으로 **전환합니다**.**

## 시나리오

Fabric eventstreams를 사용하면 이벤트 데이터를 한 곳에서 쉽게 관리할 수 있습니다. 실시간 이벤트 데이터를 수집, 변환 및 원하는 형식으로 다른 대상으로 보낼 수 있습니다. 번거로움 없이 Eventstreams를 Azure Event Hubs, KQL 데이터베이스 및 Lakehouse에 연결할 수도 있습니다.

이 랩은 주식 시장 데이터라는 샘플 스트리밍 데이터를 기반으로 합니다. 주식 시장 샘플 데이터는 시간, 기호, 가격, 거래량 등과 같은 미리 설정된 스키마 열이 있는 증권 거래소의 데이터 세트입니다. 이 샘플 데이터를 사용하여 주가의 실시간 이벤트를 시뮬레이션하고 KQL 데이터베이스와 같은 다양한 대상으로 분석합니다.

실시간 분석 스트리밍 및 쿼리 기능을 사용하여 주식 통계에 대한 주요 질문에 답변합니다. 이 시나리오에서는 KQL 데이터베이스와 같은 일부 구성 요소를 독립적으로 수동으로 만드는 대신 마법사를 최대한 활용하겠습니다.

이 자습서에서는 다음 작업을 수행하는 방법을 알아봅니다.

- KQL 데이터베이스 만들기
- OneLake에 데이터 복사 사용
- Eventstream 만들기
- Eventstream에서 KQL 데이터베이스로 데이터 스트리밍
- KQL 및 SQL을 사용하여 데이터 탐색

## KQL 데이터베이스 만들기

1. **실시간 분석** 내에서 KQL 데이터베이스** 상자를 선택합니다**.

   ![kqldatabase 선택 이미지](./Images/select-kqldatabase.png)

2. KQL 데이터베이스의 이름을 지정**하라**는 메시지가 표시됩니다.

   ![이름 kqldatabase 이미지](./Images/name-kqldatabase.png)

3. KQL 데이터베이스에 MyStockData와 같이 **기억할 이름을 지정하고 Create** 키를 누릅니**** 다.

1. **데이터베이스 세부 정보** 패널에서 연필 아이콘을 선택하여 OneLake에서 가용성을 설정합니다.

   ![onlake 사용 이미지](./Images/enable-onelake-availability.png)

2. 단추를 활성**으로 전환한 다음 완료**를 **선택**해야 합니다.

 > **참고:** 폴더를 선택할 필요가 없습니다. Fabric에서 폴더를 만듭니다.

   ![Onelake 토글 사용 이미지](./Images/enable-onelake-toggle.png)

## Eventstream 만들기

1. 메뉴 모음에서 실시간 분석을** 선택합니다**(아이콘은 rta 로고](./Images/rta_logo.png)와 유사)![
2. 새로 만들기**에서 **EventStream(미리 보기)을 선택합니다 **.**

   ![eventstream 선택 이미지](./Images/select-eventstream.png)

3. 이벤트 스트림의 이름을 지정**하라**는 메시지가 표시됩니다. EventStream에 MyStockES와 같이 **기억할 이름을 지정하고 **만들기** 단추를 누릅니**다.

   ![이름 eventstream 이미지](./Images/name-eventstream.png)

## 이벤트 스트림 원본 및 대상 설정

1. Eventstream 캔버스의 드롭다운 목록에서 새 원본**을 선택한 **다음, 샘플 데이터를** 선택합니다**.

   ![EventStream 캔버스 이미지](./Images/real-time-analytics-canvas.png)

2. 다음 표와 같이 샘플 데이터의 값을 입력한 다음 추가**를 선택합니다**.

   | 필드       | 권장 값 |
   | ----------- | ----------------- |
   | 원본 이름 | StockData         |
   | 샘플 데이터 | 주식 시장      |

3. 이제 새 대상을 선택하여 대상**을 추가한 **다음, KQL 데이터베이스를 선택합니다 **.**

   ![EventStream 대상 이미지](./Images/new-kql-destination.png)

4. KQL 데이터베이스 구성에서 다음 표를 사용하여 구성을 완료합니다.

   | 필드            | 권장 값                              |
   | ---------------- | ---------------------------------------------- |
   | 대상 이름 | MyStockData                                    |
   | 작업 영역        | KQL 데이터베이스를 만든 작업 영역 |
   | KQL 데이터베이스     | MyStockData                                    |
   | 대상 테이블| MyStockData                                    |
   | 입력 데이터 형식| Json                                           |

3. **추가**를 선택합니다.

> **참고**: 데이터 수집이 즉시 시작됩니다.

모든 단계가 녹색 검사 표시로 표시될 때까지 기다립니다. Eventsream에서 페이지 제목 **연속 수집이 설정된 것을 볼 수 있습니다.** 그런 다음, 닫기를** 선택하여 **Eventstream 페이지로 돌아갑니다.

> **참고**: Eventstream 연결이 빌드되고 설정된 후 테이블을 보려면 페이지를 새로 고쳐야 할 수 있습니다.

## KQL 쿼리

KQL(Kusto 쿼리 언어)은 데이터를 처리하고 결과를 반환하기 위한 읽기 전용 요청입니다. 요청은 쉽게 읽고 작성하고 자동화할 수 있는 데이터 흐름 모델을 사용하여 일반 텍스트로 서술됩니다. 쿼리는 항상 특정 테이블 또는 데이터베이스의 컨텍스트에서 실행됩니다. 최소한 쿼리는 원본 데이터 참조와 순서대로 적용된 하나 이상의 쿼리 연산자로 구성되며, 이는 연산자를 구분하기 위해 파이프 문자(|)를 사용하여 시각적으로 표시됩니다. Kusto 쿼리 언어 대한 자세한 내용은 Kusto 쿼리 언어(KQL) 개요를 참조[하세요.](https://learn.microsoft.com/en-us/azure/data-explorer/kusto/query/?context=%2Ffabric%2Fcontext%2Fcontext)

> **참고**: KQL 편집기에서는 구문과 Inellisense 강조 표시가 함께 제공되므로 KQL(Kusto 쿼리 언어)에 대한 지식을 빠르게 얻을 수 있습니다.

1. MyStockData**라는 **새로 만들어지고 하이드레이션된 KQL 데이터베이스로 이동합니다.
2. 데이터 트리의 MyStockData 테이블에서 [...] 추가 메뉴를 선택합니다. 그런 다음 쿼리 테이블을 > 100 레코드를 표시합니다.

   ![KQL 쿼리 집합 이미지](./Images/kql-query-sample.png)

3. 샘플 쿼리는 테이블 컨텍스트가 **이미 채워진 데이터** 탐색 창에서 열립니다. 이 첫 번째 쿼리는 take 연산자를 사용하여 샘플 수의 레코드를 반환하며 데이터 구조와 가능한 값을 먼저 살펴보는 데 유용합니다. 자동 채워진 샘플 쿼리는 자동으로 실행됩니다. 결과 창에서 쿼리 결과를 볼 수 있습니다.

   ![KQL 쿼리 결과 이미지](./Images/kql-query-results.png)

4. 데이터 트리로 돌아가서 다음 쿼리를 선택합니다. 여기서는 where 연산자와 연산자 사이를 사용하여 지난 24시간 동안 수집된 레코드를 반환합니다.

   ![지난 24일 KQL 쿼리 결과 이미지](./Images/kql-query-results-last24.png)

> **참고**: 쿼리 제한을 초과했다는 경고가 표시될 수 있습니다. 이 동작은 데이터베이스로 스트리밍되는 데이터의 양에 따라 달라집니다.

기본 제공 쿼리 함수를 사용하여 계속 탐색하여 데이터를 숙지할 수 있습니다.

## 샘플 SQL 쿼리

쿼리 편집기에서는 기본 쿼리 Kusto 쿼리 언어(KQL) 외에도 T-SQL 사용을 지원합니다. T-SQL은 KQL을 사용할 수 없는 도구에 유용할 수 있습니다. 자세한 내용은 T-SQL을 사용하여 데이터 쿼리를 참조 [하세요.](https://learn.microsoft.com/en-us/azure/data-explorer/t-sql)

1. 데이터 트리로 돌아가서 MyStockData 테이블에서 [...] 추가 메뉴를** 선택합니다**. 쿼리 테이블을 > **SQL > 100** 레코드를 표시합니다.

   ![sql 쿼리 샘플 이미지](./Images/sql-query-sample.png)

2. 쿼리 내 어딘가에 커서를 놓고 실행을** 선택**하거나 Shift + Enter를 누릅니**다**.

   ![sql 쿼리 결과 이미지](./Images/sql-query-results.png)

빌드 기능으로 계속 탐색하고 SQL 또는 KQL을 사용하여 데이터를 숙지할 수 있습니다. 그러면 단원이 끝납니다.

## 리소스 정리

이 연습에서는 KQL 데이터베이스를 만들고 eventstream을 사용하여 연속 스트리밍을 설정했습니다. 그 후 KQL 및 SQL을 사용하여 데이터를 쿼리했습니다. KQL 데이터베이스 탐색을 마쳤으면 이 연습에 대해 만든 작업 영역을 삭제할 수 있습니다.
1. 왼쪽의 막대에서 작업 영역의 아이콘을 선택합니다.
2. 도구 모음의 ... 메뉴에서 작업 영역 설정을 선택합니다.
3. 기타 섹션에서 이 작업 영역 제거를 선택합니다.
