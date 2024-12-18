---
lab:
  title: Microsoft Fabric의 Eventstream으로 실시간 데이터 수집
  module: Ingest real-time data with Eventstream in Microsoft Fabric
---
# Microsoft Fabric의 Eventstream으로 실시간 데이터 수집

Eventstream은 실시간 이벤트를 캡처, 변환 및 다양한 대상으로 라우팅하는 Microsoft Fabric의 기능입니다. 이벤트 데이터 원본, 대상 및 변환을 이벤트 스트림에 추가할 수 있습니다.

이 연습에서는 사람들이 도시 내에서 자전거를 대여할 수 있는 자전거 공유 시스템에서 자전거 수거 지점의 관찰 사항 관련하여 이벤트 스트림을 내보내기하는 샘플 데이터 원본에서 데이터를 수집합니다.

이 랩을 완료하는 데 약 **30**분이 소요됩니다.

> **참고**: 이 연습을 완료하려면 [Microsoft Fabric 테넌트](https://learn.microsoft.com/fabric/get-started/fabric-trial)가 필요합니다.

## 작업 영역 만들기

Fabric에서 데이터로 작업하기 전에, Fabric 용량을 사용하도록 설정된 작업 영역을 만들어야 합니다.

1. [Microsoft Fabric 홈페이지](https://app.fabric.microsoft.com/home?experience=fabric)(`https://app.fabric.microsoft.com/home?experience=fabric`)에서 **실시간 인텔리전스**를 선택합니다.
1. 왼쪽 메뉴 모음에서 **작업 영역**을 선택합니다(아이콘은 와 유사함).
1. Fabric 용량이 포함된 라이선스 모드(*평가판*, *프리미엄* 또는 *Fabric*)를 선택하여 원하는 이름으로 새 작업 영역을 만듭니다.
1. 새 작업 영역이 열리면 비어 있어야 합니다.

    ![Fabric의 빈 작업 영역 스크린샷.](./Images/new-workspace.png)

## 이벤트 하우스 만들기

이제 작업 영역이 있으므로 실시간 인텔리전스 솔루션에 필요한 Fabric 항목 만들기를 시작할 수 있습니다. 먼저 이벤트 하우스를 만들어 보겠습니다.

1. 왼쪽 메뉴 모음에서 **홈**을 선택한 다음 실시간 인텔리전스 홈 페이지에서 새 **Eventhouse**를 만들어, 원하는 고유한 이름을 지정합니다.
1. 빈 이벤트 하우스가 새로 표시될 때까지 표시되는 팁이나 프롬프트를 닫습니다.

    ![새 이벤트 하우스의 스크린샷](./Images/create-eventhouse.png)

1. 왼쪽 창에서 이벤트 하우스에는 이벤트 하우스와 이름이 같은 KQL 데이터베이스가 포함되어 있습니다.
1. KQL 데이터베이스를 선택하여 확인합니다.

    현재 데이터베이스에는 테이블이 없습니다. 이 연습의 나머지 부분에서는 이벤트 스트림을 사용하여 실시간 원본에서 테이블로 데이터를 로드합니다.

## Eventstream 만들기

1. KQL 데이터베이스의 기본 페이지에서 **데이터 가져오기**를 선택합니다.
2. 데이터 원본에 대해 **Eventstream** > **새 Eventstream**을 선택합니다. Eventstream의 이름을 `Bicycle-data`(으)로 지정합니다.

    작업 영역에서 새 이벤트 스트림 만들기는 몇 분 안에 완료됩니다. 일단 설정되면 자동으로 기본 편집기로 리디렉션되어 원본을 이벤트 스트림에 통합할 준비가 됩니다.

    ![새 이벤트 스트림의 스크린샷.](./Images//name-eventstream.png)

## 원본 추가

1. Eventstream 캔버스에서 **샘플 데이터 사용**을 선택합니다.
2. 원본 이름을 `Bicycles`(으)로 지정하고 **자전거** 샘플 데이터를 선택합니다.

    스트림이 매핑되고 **이벤트 스트림 캔버스**에 자동으로 표시됩니다.

   ![이벤트 스트림 캔버스 검토](./Images/real-time-intelligence-eventstream-sourced.png)

## 대상 추가

1. **이벤트 변환 또는 대상 추가** 드롭다운 목록의 **대상** 섹션에서 **Eventhouse**를 선택합니다.
1. **Eventhouse** 창에서 다음 설정 옵션을 구성합니다.
   - **데이터 수집 모드:**: 수집 전 이벤트 처리
   - **목적지 이름:**`bikes-table`
   - **작업 영역:***이 연습의 시작 부분에서 만든 작업 영역 선택*
   - **Eventhouse**: *이벤트 하우스 선택*
   - **KQL 데이터베이스:** *KQL 데이터베이스 선택*
   - **대상 테이블:** 이름이 지정된 새 테이블 만들기`bikes`
   - **입력 데이터 식:** JSON

   ![Eventstream 대상 설정.](./Images/kql-database-event-processing-before-ingestion.png)

1. **Eventhouse** 창에서 **저장**을 선택합니다. 
1. 도구 모음에서 **게시**를 선택합니다.
1. 데이터 대상이 활성화될 때까지 1분 정도 기다립니다. 그런 다음 디자인 캔버스에서 **bikes-table** 노드를 선택하고 아래의 **데이터 미리 보기** 창을 확인하여 수집된 최신 데이터를 확인합니다.

   ![이벤트 스트림의 대상 테이블.](./Images/stream-data-preview.png)

1. 몇 분 기다린 다음 **새로 고침** 버튼을 사용하여 **데이터 미리 보기** 창을 새로 고칩니다. 스트림이 영구적으로 실행되므로 새 데이터가 테이블에 추가되었을 수 있습니다.
1. 이벤트 스트림 디자인 캔버스 아래 **데이터 인사이트** 탭에서 캡처된 데이터 이벤트의 세부 정보를 확인할 수 있습니다.

## 캡처된 데이터 쿼리

만든 이벤트 스트림은 자전거 데이터의 샘플 원본에서 데이터를 가져와서 이벤트 하우스의 데이터베이스에 로드합니다. 데이터베이스의 테이블을 쿼리하여 캡처된 데이터를 분석할 수 있습니다.

1. 왼쪽 메뉴 모음에서 KQL 데이터베이스를 선택합니다.
1. **데이터베이스** 탭의 KQL 데이터베이스 도구 모음에서 **새로 고침** 버튼을 사용하여 데이터베이스 아래에 **Bikes** 테이블이 표시될 때까지 보기를 새로 고칩니다. 그런 다음 **Bikes** 테이블을 선택합니다.

   ![KQL 데이터베이스의 테이블.](./Images/kql-table.png)

1. **bikes** 테이블**의 **...** 메뉴에서 **쿼리 테이블** > **지난 24시간 동안 수집된 레코드**를 선택합니다.
1. 쿼리 창에서 다음 쿼리가 생성 및 실행되었으며 그 결과가 아래에 표시되어 있음을 확인합니다.

    ```kql
    // See the most recent data - records ingested in the last 24 hours.
    bikes
    | where ingestion_time() between (now(-1d) .. now())
    ```

1. 쿼리 코드를 선택하고 실행하여 테이블에서 100개의 데이터 행을 확인합니다.

    ![KQL 쿼리 스크린샷.](./Images/kql-query.png)

## 이벤트 데이터 변환

캡처한 데이터는 원본에서 변경되지 않습니다. 많은 시나리오에서 이벤트 스트림의 데이터를 대상으로 로드하기 전에 변환해야 할 수 있습니다.

1. 왼쪽 메뉴 모음에서 **Bicycle-data** 이벤트 스트림을 선택합니다.
1. 도구 모음에서 **편집**을 선택하여 이벤트 스트림을 편집합니다.
1. **이벤트 변환** 메뉴에서 **그룹 기준**을 선택하여 이벤트 스트림에 새 **그룹 기준** 노드를 추가합니다.
1. **Bicycle-data** 노드의 출력에서 새 **그룹 기준** 노드의 입력으로 연결을 끌어온 다음 **그룹 기준** 노드의 *연필* 아이콘을 사용하여 편집합니다.

   ![변환 이벤트에 그룹 기준을 추가합니다. ](./Images/eventstream-add-aggregates.png)

1. **그룹 기준** 설정 섹션의 속성을 구성합니다.
    - **작업 이름:** GroupByStreet
    - **집계 형식:** SUM *선택*
    - **필드:** No_Bikes *선택* *그런 다음 **추가**를 선택하여 No_Bikes의 SUM 함수를 만듭니다.*
    - **그룹 집계 기준(선택 사항):** Street
    - **시간 창**: 연속
    - **기간**: 5초
    - **오프셋**: 0초

    > **참고**: 이 구성에 따라 이벤트 스트림은 5초마다 각 거리의 총 자전거 수를 계산합니다.
      
1. 구성을 저장하고 이벤트 스트림 캔버스로 돌아가면 오류가 표시됩니다(변환을 통해 출력을 어딘가에 저장해야 하기 때문에 그렇습니다).

1. **GroupByStreet** 노드 오른쪽의 **+** 아이콘을 사용하여 새 **Eventhouse** 노드를 추가합니다.
1. 다음 옵션을 사용하여 새 이벤트 하우스 노드를 구성합니다.
   - **데이터 수집 모드:**: 수집 전 이벤트 처리
   - **대상 이름:**`bikes-by-street-table`
   - **작업 영역:***이 연습의 시작 부분에서 만든 작업 영역 선택*
   - **Eventhouse**: *이벤트 하우스 선택*
   - **KQL 데이터베이스:** *KQL 데이터베이스 선택*
   - **대상 테이블:** 이름이 지정된 새 테이블 만들기`bikes-by-street`
   - **입력 데이터 식:** JSON

    ![그룹화된 데이터에 대한 테이블의 스크린샷.](./Images/group-by-table.png)

1. **Eventhouse** 창에서 **저장**을 선택합니다. 
1. 도구 모음에서 **게시**를 선택합니다.
1. 변경 내용이 활성화될 때까지 잠시 1분 정도 기다립니다.
1. 디자인 캔버스에서 **bikes-by-street-table** 노드를 선택하고 캔버스 아래에 있는 **데이터 미리보기** 창을 확인합니다.

    ![그룹화된 데이터에 대한 테이블의 스크린샷.](./Images/stream-table-with-windows.png)

    변환된 데이터에는 지정한 그룹화 필드(**Street**), 지정한 집계(**SUM_no_Bikes**), 이벤트가 발생한 5초 연속 창의 종료 시점을 나타내는 타임스탬프 필드(**Window_End_Time**)가 포함되어 있습니다.

## 변환된 데이터 쿼리

이제 이벤트 스트림에 의해 변환되어 테이블로 로드된 자전거 데이터를 쿼리할 수 있습니다.

1. 왼쪽 메뉴 모음에서 KQL 데이터베이스를 선택합니다.
1. 1. **데이터베이스** 탭의 KQL 데이터베이스 도구 모음에서 **새로 고침** 버튼을 사용하여 데이터베이스 아래에 **bikes-by-street** 테이블이 표시될 때까지 보기를 새로 고칩니다.
1. **bikes-by-street** 테이블의 **...** 메뉴에서 **데이터 쿼리** > **100개 레코드 표시**를 선택합니다.
1. 쿼리 창에서 다음 쿼리가 생성되고 실행되는 것을 확인합니다.

    ```kql
    ['bikes-by-street']
    | take 100
    ```

1. KQL 쿼리를 수정하여 각 5초 시간 범위 내에 거리당 총 자전거 수를 검색합니다.

    ```kql
    ['bikes-by-street']
    | summarize TotalBikes = sum(tolong(SUM_No_Bikes)) by Window_End_Time, Street
    | sort by Window_End_Time desc , Street asc
    ```

1. 수정된 쿼리를 선택하고 실행합니다.

    결과는 각 5초 시간 범위 내에 각 거리에서 관찰된 자전거의 수를 보여 줍니다.

    ![그룹화된 데이터를 반환하는 쿼리의 스크린샷.](./Images/kql-group-query.png)

<!--
## Add an Activator destination

So far, you've used an eventstream to load data into tables in an eventhouse. You can also direct streams to an activator and automate actions based on values in the event data.

1. In the menu bar on the left, return to the **Bicycle-data** eventstream. Then in the eventstream page, on the toolbar, select **Edit**.
1. In the **Add destination** menu, select **Activator**. Then drag a connection from the output of the **Bicycle-data** stream to the input of the new Activator destination.
1. Configure the new Activator destination with the following settings:
    - **Destination name**: `low-bikes-activator`
    - **Workspace**: *Select your workspace*
    - **Activator**: *Create a **new** activator named `low-bikes`*
    - **Input data format**: Json

    ![Screenshot of an Activator destination.](./Images/activator-destination.png)

1. Save the new destination.
1. In the menu bar on the left, select your workspace to see all of the items you have created so far in this exercise - including the new **low-bikes** activator.
1. Select the **low-bikes** activator to view its page, and then on the activator page select **Get data**.
1. On the **select a data source** dialog box, scroll down until you see **Data streams** and then select the **Bicycle-data-stream**.

    ![Screenshot of data sources for an activator.](./Images/select-activator-stream.png)

1. Use the **Next**,  **Connect**, and **Finish** buttons to connect the stream to the activator.

    > **Tip**: If the data preview obscures the **Next** button, close the dialog box, select the stream again, and click **Next** before the preview is rendered.

1. When the stream has been connected, the activator page displays the **Events** tab:

    ![Screenshot of the activator Events page.](./Images/activator-events-page.png)

1. Add a new rule, and configure its definition with the following settings:
    - **Monitor**:
        - **Event**: Bicycle-data-stream-event
    - **Condition**
        - **Condition 1**:
            - **Operation**: Numeric state: Is less than or equal to
            - **Column**: No_Bikes
            - **Value**: 3
            - **Default type**: Same as window size
    - **Action**:
        - **Type**: Email
        - **To**: *The email address for the account you are using in this exercise*
        - **Subject**: `Low bikes`
        - **Headline**: `The number of bikes is low`
        - **Message**: `More bikes are needed.`
        - **Context**: *Select the **Neighborhood**, **Street**, and **No-Bikes** columns.

    ![Screenshot of an activator rule definition.](./Images/activator-rule.png)

1. Save and start the rule.
1. View the **Analytics** tab for the rule, which should show each instance if the condition being met as the stream of events is ingested by your eventstream.

    Each instance will result in an email being sent notifying you of low bikes, which will result in a large numbers of emails, so...

1. On the toolbar, select **Stop** to stop the rule from being processed.

-->

## 리소스 정리

이 연습에서는 이벤트 스트림을 사용하여 데이터베이스에 이벤트 하우스와 채워진 테이블을 만들었습니다.

KQL 데이터베이스 탐색을 마쳤으면 이 연습을 위해 만든 작업 영역을 삭제할 수 있습니다.

1. 왼쪽 막대에서 작업 영역의 아이콘을 선택합니다.
2. 도구 모음에서 **작업 영역 설정**을 선택합니다.
3. **일반** 섹션에서 **이 작업 영역 제거**를 선택합니다.
.
