
## ***작업 초안**
---
lab:
  title: 실시간 대시보드
  module: Query data from a Kusto Query database in Microsoft Fabric
---

# Microsoft Fabric에서 Kusto 데이터베이스 쿼리 시작

실시간 대시보드를 사용하면 KQL(Kusto 쿼리 언어)을 사용하여 Microsoft Fabric 내에서 인사이트를 수집하여 구조 및 구조화되지 않은 데이터를 모두 검색하고 Power BI 내의 슬라이서와 유사하게 연결할 수 있는 패널 내에서 차트, 산점도, 테이블 등에 렌더링할 수 있습니다. 

이 랩을 완료하는 데 약 **25** 분이 걸립니다.

> **참고**: 이 연습을 완료하려면 Microsoft Fabric 평가판[이 필요합니다](https://learn.microsoft.com/fabric/get-started/fabric-trial).

## 작업 영역 만들기

패브릭에서 데이터를 사용하기 전에 패브릭 평가판을 사용하도록 설정된 작업 영역을 만듭니다.

1. Microsoft Fabric 홈페이지[에서 ](https://app.fabric.microsoft.com)실시간 분석을** 선택합니다**.
1. 왼쪽 메뉴 모음에서 **작업 영역**을 선택합니다(아이콘은 와 유사함).
1. 선택한 이름으로 새 작업 영역을 만들고 패브릭 용량(*평가판*, *프리미엄* 또는 *패브릭*)이 포함된 라이선스 모드를 선택합니다.
1. 새 작업 영역이 열리면 비어 있어야 합니다.

    ![Fabric의 빈 작업 영역 스크린샷](./Images/new-workspace.png)

이 랩에서는 패브릭의 RTA(실시간 분석)를 사용하여 샘플 eventstream에서 KQL 데이터베이스를 만듭니다. 실시간 분석은 RTA의 기능을 탐색하는 데 사용할 수 있는 샘플 데이터 세트를 편리하게 제공합니다. 이 샘플 데이터를 사용하여 KQL 만들기 | 실시간 데이터를 분석하고 다운스트림 프로세스에서 다른 용도를 허용하는 SQL 쿼리 및 쿼리 세트입니다.

## KQL 데이터베이스 만들기

1. **실시간 분석** 내에서 KQL 데이터베이스** 상자를 선택합니다**.

   ![KQL 데이터베이스 선택 이미지](./Images/select-kqldatabase.png)

2. KQL 데이터베이스의 이름을 지정**하라**는 메시지가 표시됩니다.

   ![이름 KQL 데이터베이스 이미지](./Images/name-kqldatabase.png)

3. KQL 데이터베이스에 MyStockData와 같이 **기억해야 하는 이름을 지정하고 Create** 키를 누릅니**** 다.

4. **데이터베이스 세부 정보** 패널에서 연필 아이콘을 선택하여 OneLake에서 가용성을 설정합니다.

   ![Onelake 사용 이미지](./Images/enable-onelake-availability.png)

5. 데이터를** 가져오면 시작 옵션***에서 샘플 데이터*** 상자를 선택합니다**.
 
   ![샘플 데이터가 강조 표시된 선택 옵션 이미지](./Images/load-sample-data.png)

6. **샘플 데이터에 대한 옵션에서 자동차 메트릭 분석** 상자를 선택합니다.

   ![랩에 대한 분석 데이터 선택 이미지](./Images/create-sample-data.png)

7. 데이터 로드가 완료되면 KQL 데이터베이스가 채워지는지 확인할 수 있습니다.

   ![KQL 데이터베이스에 로드되는 데이터](./Images/choose-automotive-operations-analytics.png)

7. 데이터가 로드되면 데이터가 KQL 데이터베이스에 로드되는지 확인합니다. 테이블 오른쪽에 있는 줄임표(...)를 선택하고 쿼리 테이블로 이동하고 **100개의 레코드** 표시를 **선택하여 이 작업을 수행할 수** 있습니다.

    ![RawServerMetrics 테이블에서 상위 100개 파일을 선택하는 이미지](./Images/rawservermetrics-top-100.png)

   > **참고**: 처음 실행할 때 컴퓨팅 리소스를 할당하는 데 몇 초 정도 걸릴 수 있습니다.

    ![데이터에서 100 레코드 이미지](./Images/explore-with-kql-take-100.png)


## 시나리오
이 시나리오에서는 Microsoft Fabric에서 제공하는 샘플 데이터를 기반으로 실시간 대시보드를 만들어 다양한 방법으로 데이터를 표시하고, 변수를 만들고, 이 변수를 사용하여 대시보드 패널을 함께 연결하여 원본 시스템 내에서 발생하는 일에 대한 심층적인 인사이트를 얻을 수 있습니다. 이 모듈에서는 NY Taxi 데이터 세트를 사용하여 자치구별 여정의 현재 세부 정보를 살펴봅니다.

1. 실시간 분석으로 이동**한 다음 패브릭 기본 페이지에서 실시간 대시보드**를 선택합니다**.**

    ![실시간 대시보드를 선택합니다.](./Images/select-real-time-dashboard.png)

1. tne **새 타일** 추가 단추를 누릅니다.

```kusto

Trips
| summarize ["Total Trip Distance"] = sum(trip_distance) by pickup_boroname
| project Borough = case(isempty(pickup_boroname) or isnull(pickup_boroname), "Unidentified", pickup_boroname), ["Total Trip Distance"]
| sort by Borough asc 

```
3. **실행** 단추를 누르고 쿼리에 오류가 없는지 확인합니다.
4. 패널의 오른쪽에서 시각적 개체 서식 탭**을 **선택하고 타일 이름*** 및 ***시각적 개체 유형을*** 완료***합니다.

   ![시각적 개체 서식 타일의 이미지입니다.](./Images/visual-formatting-tile.png)

