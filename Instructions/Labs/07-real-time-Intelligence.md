---
lab:
  title: Microsoft Fabric에서 실시간 인텔리전스 시작
  module: Get started with Real-Time Intelligence in Microsoft Fabric
---

# Microsoft Fabric에서 실시간 인텔리전스 시작

Microsoft Fabric은 KQL(Kusto 쿼리 언어)을 사용하여 데이터를 저장하고 쿼리하는 데 사용할 수 있는 런타임을 제공합니다. Kusto는 로그 파일 또는 IoT 디바이스의 실시간 데이터와 같은 시계열 구성 요소를 포함하는 데이터에 최적화되어 있습니다.

이 랩을 완료하는 데 약 **30**분이 소요됩니다.

> **참고**: 이 연습을 완료하려면 [Microsoft Fabric 평가판](https://learn.microsoft.com/fabric/get-started/fabric-trial)이 필요합니다.

## 작업 영역 만들기

패브릭에서 데이터를 사용하기 전에 패브릭 평가판을 사용하도록 설정된 작업 영역을 만듭니다.

1. [Microsoft Fabric 홈페이지](https://app.fabric.microsoft.com/home?experience=fabric)(`https://app.fabric.microsoft.com/home?experience=fabric`)에서 **실시간 인텔리전스**를 선택합니다.
1. 왼쪽 메뉴 모음에서 **작업 영역**을 선택합니다(아이콘은 와 유사함).
1. Fabric 용량이 포함된 라이선스 모드(*평가판*, *프리미엄* 또는 *Fabric*)를 선택하여 원하는 이름으로 새 작업 영역을 만듭니다.
1. 새 작업 영역이 열리면 비어 있어야 합니다.

    ![Fabric의 빈 작업 영역 스크린샷.](./Images/new-workspace.png)

## KQL 데이터베이스용 파일 다운로드

이제 작업 영역이 있으므로 분석할 데이터 파일을 다운로드할 차례입니다.

1. 이 연습을 위한 데이터 파일을 [https://raw.githubusercontent.com/MicrosoftLearning/dp-data/main/sales.csv](https://raw.githubusercontent.com/MicrosoftLearning/dp-data/main/sales.csv)에서 다운로드하여 로컬 컴퓨터(또는 해당하는 경우 랩 VM)에 **sales.csv**로 저장합니다.
1. **Microsoft Fabric** 환경이 있는 브라우저 창으로 돌아갑니다.

## KQL 데이터베이스 만들기

KQL(Kusto 쿼리 언어)은 KQL 데이터베이스에 정의된 테이블의 정적 또는 스트리밍 데이터를 쿼리하는 데 사용됩니다. 판매 데이터를 분석하려면 KQL 데이터베이스에 테이블을 만들고 파일에서 데이터를 수집해야 합니다.

1. 포털의 왼쪽 아래에서 실시간 인텔리전스 환경으로 전환합니다.

    ![환경 전환기 메뉴의 스크린샷.](./Images/fabric-real-time.png)

2. 실시간 인텔리전스 홈페이지에서 선택한 이름으로 새 **Eventhouse**를 만듭니다.

   ![Eventhouse가 강조 표시된 RTI 편집기의 스크린샷](./Images/create-kql-db.png)

   Eventhouse는 여러 프로젝트에서 데이터베이스를 그룹화하고 관리하는 데 사용됩니다. 빈 KQL 데이터베이스가 eventhouse의 이름으로 자동으로 만들어집니다.
   
3. 새 데이터베이스가 만들어지면 KQL 데이터베이스 아래의 왼쪽 목록에서 선택합니다. 그런 다음 **로컬 파일**에서 데이터를 가져오는 옵션을 선택합니다. 마법사에서 다음 옵션을 선택하여 데이터를 새 테이블로 가져옵니다.
    - **대상:**
        - **데이터베이스**: *만든 데이터베이스가 이미 선택되어 있음*
        - **Table**: ***새 테이블*** 왼쪽에 있는 + 기호를 클릭하여 **sales**라는 *이름의 새 테이블 만들기*

        ![새 테이블 마법사 1단계](./Images/import-wizard-local-file-1.png?raw=true)

        - 이제 동일한 창에 **파일을 여기로 끌거나 파일 찾아보기** 하이퍼링크가 나타나는 것을 볼 수 있습니다.

        ![새 테이블 마법사 2단계](./Images/import-wizard-local-file-2.png?raw=true)

        - **sales.csv**를 찾아보거나 화면으로 끌어서 상태 상자가 녹색 확인란으로 바뀔 때까지 기다린 후 **다음**을 선택합니다.

        ![새 테이블 마법사 3단계](./Images/import-wizard-local-file-3.png?raw=true)

        - 이 화면에서는 시스템이 열 머리글을 검색했지만 열 머리글이 첫 번째 행에 있는 것을 볼 수 있습니다. 오류가 발생하지 않도록 하려면 슬라이더를 이 **첫 번째 행은 열 머리글임** 선 위로 이동해야 합니다.
        
        ![새 테이블 마법사 4단계](./Images/import-wizard-local-file-4.png?raw=true)

        - 이 슬라이더를 선택하면 모든 것이 잘 진행된 것으로 보입니다. 패널 오른쪽 하단에 있는 **마침** 단추를 선택합니다.

        ![새 테이블 마법사 5단계](./Images/import-wizard-local-file-5.png?raw=true)

        - 다음을 포함하는 요약 화면의 단계가 완료될 때까지 기다립니다.
            - 테이블 만들기(sales)
            - 매핑 만들기(sales_mapping)
            - 데이터 큐잉
            - 수집
        - **닫기** 단추를 선택합니다.

        ![새 테이블 마법사 6단계](./Images/import-wizard-local-file-6.png?raw=true)

> **참고**: 이 예에서는 파일에서 매우 적은 양의 정적 데이터를 가져왔는데 이 연습의 용도로는 괜찮습니다. 실제로 Kusto를 사용하면 Azure Event Hubs와 같은 스트리밍 원본의 실시간 데이터를 포함하여 훨씬 더 많은 양의 데이터를 분석할 수 있습니다.

## KQL을 사용하여 판매 테이블 쿼리

이제 데이터베이스에 데이터 테이블이 있으므로 KQL 코드를 사용하여 쿼리할 수 있습니다.

1. **판매** 테이블이 강조 표시되어 있는지 확인합니다. 메뉴 모음에서 **쿼리 테이블** 드롭다운을 선택하고 여기에서 **100개 레코드 표시**를 선택합니다.

2. 쿼리와 해당 결과가 포함된 새 창이 열립니다. 

3. 쿼리를 다음과 같이 수정합니다.

    ```kusto
   sales
   | where Item == 'Road-250 Black, 48'
    ```

4. 쿼리를 실행합니다. 그런 다음, *Road-250 Black, 48* 제품에 대한 판매 주문 행만 포함해야 하는 결과를 검토합니다.

5. 쿼리를 다음과 같이 수정합니다.

    ```kusto
   sales
   | where Item == 'Road-250 Black, 48'
   | where datetime_part('year', OrderDate) > 2020
    ```

6. 쿼리를 실행하고 결과를 검토합니다. 이 결과에는 2020년 이후에 이루어진 *Road-250 Black, 48*에 대한 판매 주문만 포함되어야 합니다.

7. 쿼리를 다음과 같이 수정합니다.

    ```kusto
   sales
   | where OrderDate between (datetime(2020-01-01 00:00:00) .. datetime(2020-12-31 23:59:59))
   | summarize TotalNetRevenue = sum(UnitPrice) by Item
   | sort by Item asc
    ```

8. 쿼리를 실행하고 결과를 검토합니다. 이 결과에는 제품 이름의 오름차순으로 2020년 1월 1일부터 12월 31일까지 각 제품의 총 순 수익이 포함됩니다.

## KQL 쿼리 세트에서 Power BI 보고서 만들기

KQL 쿼리 세트를 Power BI 보고서의 기초로 사용할 수 있습니다.

1. 쿼리 집합에 대한 쿼리 워크벤치 편집기에서 쿼리를 실행하고 결과를 기다립니다.
2. **Power BI**를 선택하고 보고서 편집기가 열릴 때까지 기다립니다.
3. 보고서 편집기의 **데이터** 창에서 **Kusto 쿼리 결과**를 확장하고 **Item** 및 **TotalRevenue** 필드를 선택합니다.
4. 보고서 디자인 캔버스에서 추가된 테이블 시각화를 선택한 다음 **시각화** 창에서 **클러스터형 막대형 차트**를 선택합니다.

    ![KQL 쿼리 보고서의 스크린샷.](./Images/kql-report.png)

5. **Power BI** 창의 **파일** 메뉴에서 **저장**을 선택합니다. 그런 다음 레이크하우스 및 KQL 데이터베이스가 **비 업무용** 민감도 레이블을 사용하여 정의된 작업 영역에 보고서를 **Revenue by Item.pbix**로 저장합니다.
6. **Power BI** 창을 닫고 왼쪽 막대에서 작업 영역 아이콘을 선택합니다.

    포함된 모든 항목을 보려면 필요한 경우 작업 영역 페이지를 새로 고칩니다.

7. 작업 영역의 항목 목록에 **항목별 수익** 보고서가 나열되어 있습니다.

## 리소스 정리

이 연습에서는 레이크하우스에 업로드된 데이터를 분석하기 위한 KQL 데이터베이스인 레이크하우스를 만들었습니다. KQL을 사용하여 데이터를 쿼리하고 쿼리 집합을 만든 다음, Power BI 보고서를 만드는 데 사용했습니다.

KQL 데이터베이스 탐색을 마쳤으면 이 연습을 위해 만든 작업 영역을 삭제할 수 있습니다.

1. 왼쪽 막대에서 작업 영역 아이콘을 선택합니다.
2. 도구 모음의 **...** 메뉴에서 **작업 영역 설정**을 선택합니다.
3. **일반** 섹션에서 **이 작업 영역 제거**를 선택합니다.
