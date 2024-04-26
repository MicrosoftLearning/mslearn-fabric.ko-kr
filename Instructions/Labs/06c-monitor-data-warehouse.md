---
lab:
  title: Microsoft Fabric에서 데이터 웨어하우스 모니터링
  module: Monitor a data warehouse in Microsoft Fabric
---

# Microsoft Fabric에서 데이터 웨어하우스 모니터링

Microsoft Fabric에서 데이터 웨어하우스는 대규모 분석을 위한 관계형 데이터베이스를 제공합니다. Microsoft Fabric의 데이터 웨어하우스에는 활동 및 쿼리를 모니터링하는 데 사용할 수 있는 동적 관리 뷰가 포함됩니다.

이 랩을 완료하는 데 약 **30**분이 걸립니다.

> **참고**: 이 연습을 완료하려면 [Microsoft Fabric 평가판](https://learn.microsoft.com/fabric/get-started/fabric-trial)이 필요합니다.

## 작업 영역 만들기

패브릭에서 데이터를 사용하기 전에 패브릭 평가판을 사용하도록 설정된 작업 영역을 만듭니다.

1. [Microsoft Fabric 홈페이지](https://app.fabric.microsoft.com)에서 **Synapse 데이터 웨어하우스**를 선택합니다.
1. 왼쪽 메뉴 모음에서 **작업 영역**을 선택합니다(아이콘은 와 유사함).
1. Fabric 용량이 포함된 라이선스 모드(*평가판*, *프리미엄* 또는 *Fabric*)를 선택하여 원하는 이름으로 새 작업 영역을 만듭니다.
1. 새 작업 영역이 열리면 비어 있어야 합니다.

    ![Fabric의 빈 작업 영역 스크린샷.](./Images/new-workspace.png)

## 샘플 데이터 웨어하우스 만들기

작업 영역이 있으므로 이제 데이터 웨어하우스를 만들어야 합니다.

1. 왼쪽 아래에서 **데이터 웨어하우스** 환경이 선택되어 있는지 확인합니다.
1. **홈** 페이지에서 **샘플 웨어하우스**를 선택하고 **sample-dw**라는 새 데이터 웨어하우스를 만듭니다.

    1분 정도 지나면 택시 승차 분석 시나리오에 대한 샘플 데이터로 새 웨어하우스가 만들어지고 채워집니다.

    ![새 웨어하우스의 스크린샷.](./Images/sample-data-warehouse.png)

## 동적 관리 뷰 살펴보기

Microsoft Fabric 데이터 웨어하우스에는 데이터 웨어하우스 인스턴스에서 현재 작업을 식별하는 데 사용할 수 있는 DMV(동적 관리 뷰)가 포함됩니다.

1. **sample-dw** 데이터 웨어하우스 페이지의 **새 SQL 쿼리** 드롭다운 목록에서 **새 SQL 쿼리**를 선택합니다.
1. 비어 있는 새 쿼리 창에 다음 Transact-SQL 코드를 입력하여 **sys.dm_exec_connections** DMV를 쿼리합니다.

    ```sql
   SELECT * FROM sys.dm_exec_connections;
    ```

1. **&#9655; 실행** 단추를 사용하여 SQL 스크립트를 실행하고 데이터 웨어하우스에 대한 모든 연결의 세부 정보가 포함된 결과를 봅니다.
1. 다음과 같이 SQL 코드를 수정하여 **sys.dm_exec_sessions** DMV를 쿼리합니다.

    ```sql
   SELECT * FROM sys.dm_exec_sessions;
    ```

1. 수정된 쿼리를 실행하고 모든 인증된 세션의 세부 정보를 표시하는 결과를 봅니다.
1. 다음과 같이 SQL 코드를 수정하여 **sys.dm_exec_requests** DMV를 쿼리합니다.

    ```sql
   SELECT * FROM sys.dm_exec_requests;
    ```

1. 수정된 쿼리를 실행하고 결과를 확인하여 데이터 웨어하우스에서 실행 중인 모든 요청의 세부 정보를 표시합니다.
1. DMV를 조인하도록 SQL 코드를 수정하고 다음과 같이 동일한 데이터베이스에서 현재 실행 중인 요청에 대한 정보를 반환합니다.

    ```sql
   SELECT connections.connection_id,
    sessions.session_id, sessions.login_name, sessions.login_time,
    requests.command, requests.start_time, requests.total_elapsed_time
   FROM sys.dm_exec_connections AS connections
   INNER JOIN sys.dm_exec_sessions AS sessions
       ON connections.session_id=sessions.session_id
   INNER JOIN sys.dm_exec_requests AS requests
       ON requests.session_id = sessions.session_id
   WHERE requests.status = 'running'
       AND requests.database_id = DB_ID()
   ORDER BY requests.total_elapsed_time DESC;
    ```

1. 수정된 쿼리를 실행하고 결과를 확인합니다. 이 쿼리는 데이터베이스에서 실행 중인 모든 쿼리의 세부 정보를 표시합니다(이 쿼리 포함).
1. **새 SQL 쿼리** 드롭다운 목록에서 **새 SQL 쿼리**를 선택하여 두 번째 쿼리 탭을 추가합니다. 그런 다음 새 빈 쿼리 탭에서 다음 코드를 실행합니다.

    ```sql
   WHILE 1 = 1
       SELECT * FROM Trip;
    ```

1. 쿼리를 실행 상태로 두고 코드가 포함된 탭으로 돌아가 DMV를 쿼리하고 다시 실행합니다. 이번에는 결과에 다른 탭에서 실행 중인 두 번째 쿼리가 포함되어야 합니다. 해당 쿼리의 경과된 시간을 확인합니다.
1. 몇 초 정도 기다렸다가 코드를 다시 실행하여 DMV를 다시 쿼리합니다. 다른 탭의 쿼리에 대한 경과된 시간이 증가해야 합니다.
1. 쿼리가 아직 실행 중인 두 번째 쿼리 탭으로 돌아가서 **X 취소**를 선택하여 취소합니다.
1. DMV를 쿼리하는 코드가 있는 탭으로 돌아가서 쿼리를 다시 실행하여 두 번째 쿼리가 더 이상 실행되지 않는지 확인합니다.
1. 모든 쿼리 탭을 닫습니다.

> **추가 정보**: DMV 사용에 대한 자세한 내용은 Microsoft Fabric 설명서에서 [DMV를 사용하여 연결, 세션 및 요청 모니터링을 참조](https://learn.microsoft.com/fabric/data-warehouse/monitor-using-dmv)하세요.

## 쿼리 인사이트 탐색

Microsoft 패브릭 데이터 웨어하우스는 데이터 웨어하우스에서 실행 중인 쿼리에 대한 세부 정보를 제공하는 특수한 보기 집합인 *쿼리 인사이트*를 제공합니다.

1. **sample-dw** 데이터 웨어하우스 페이지의 **새 SQL 쿼리** 드롭다운 목록에서 **새 SQL 쿼리**를 선택합니다.
1. 비어 있는 새 쿼리 창에 다음 Transact-SQL 코드를 입력하여 **exec_requests_history** 뷰를 쿼리합니다.

    ```sql
   SELECT * FROM queryinsights.exec_requests_history;
    ```

1. **&#9655; 실행** 버튼을 사용하여 SQL 스크립트를 실행하고 이전에 실행된 쿼리의 세부 정보가 포함된 결과를 봅니다.
1. 다음과 같이 **frequently_run_queries** 보기를 쿼리하도록 SQL 코드를 수정합니다.

    ```sql
   SELECT * FROM queryinsights.frequently_run_queries;
    ```

1. 수정된 쿼리를 실행하고 결과를 확인하여 자주 실행되는 쿼리의 세부 정보를 표시합니다.
1. 다음과 같이 SQL 코드를 수정하여 **long_running_queries** 보기를 쿼리합니다.

    ```sql
   SELECT * FROM queryinsights.long_running_queries;
    ```

1. 수정된 쿼리를 실행하고 결과를 확인하여 모든 쿼리 및 해당 기간에 대한 세부 정보를 표시합니다.

> **추가 정보**: 쿼리 인사이트 사용에 대한 자세한 내용은 Microsoft 패브릭 설명서의 [패브릭 데이터 웨어하우징에서 쿼리 인사이트](https://learn.microsoft.com/fabric/data-warehouse/query-insights)를 참조하세요.


## 리소스 정리

이 연습에서는 동적 관리 뷰 및 쿼리 인사이트를 사용하여 Microsoft Fabric 데이터 웨어하우스의 활동을 모니터링했습니다.

데이터 웨어하우스 탐색을 마쳤으면 이 연습을 위해 만든 작업 영역을 삭제할 수 있습니다.

1. 왼쪽 막대에서 작업 영역의 아이콘을 선택하여 포함된 모든 항목을 봅니다.
2. 도구 모음의 **...** 메뉴에서 **작업 영역 설정**을 선택합니다.
3. **일반** 섹션에서 **이 작업 영역 제거**를 선택합니다.
