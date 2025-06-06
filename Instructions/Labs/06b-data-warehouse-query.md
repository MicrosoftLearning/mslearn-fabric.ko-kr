---
lab:
  title: Microsoft Fabric에서 데이터 웨어하우스 쿼리
  module: Query a data warehouse in Microsoft Fabric
---

# Microsoft Fabric에서 데이터 웨어하우스 쿼리

Microsoft Fabric에서 데이터 웨어하우스는 대규모 분석을 위한 관계형 데이터베이스를 제공합니다. Microsoft Fabric 작업 영역에 기본 제공된 풍부한 환경 집합을 통해 고객은 쉽게 사용할 수 있고 DirectLake 모드에서 Power BI와 통합된 항상 연결된 의미 체계 모델을 통해 인사이트를 얻는 시간을 단축할 수 있습니다. 

이 랩을 완료하는 데 약 **30**분이 걸립니다.

> **참고**: 이 연습을 완료하려면 [Microsoft Fabric 평가판](https://learn.microsoft.com/fabric/get-started/fabric-trial)이 필요합니다.

## 작업 영역 만들기

패브릭에서 데이터를 사용하기 전에 패브릭 평가판을 사용하도록 설정된 작업 영역을 만듭니다.

1. 브라우저에서 [Microsoft Fabric 홈페이지](https://app.fabric.microsoft.com/home?experience=fabric)(`https://app.fabric.microsoft.com/home?experience=fabric`)로 이동하고 Fabric 자격 증명을 사용해 로그인합니다.
1. 왼쪽 메뉴 모음에서 **작업 영역**을 선택합니다(아이콘은 와 유사함).
1. Fabric 용량이 포함된 라이선스 모드(*평가판*, *프리미엄* 또는 *Fabric*)를 선택하여 원하는 이름으로 새 작업 영역을 만듭니다.
1. 새 작업 영역이 열리면 비어 있어야 합니다.

    ![Fabric의 빈 작업 영역 스크린샷.](./Images/new-workspace.png)

## 샘플 데이터 웨어하우스 만들기

작업 영역이 있으므로 이제 데이터 웨어하우스를 만들어야 합니다.

1. 왼쪽 메뉴 모음에서 **만들기**를 선택합니다. *새* 페이지의 *데이터 웨어하우스* 섹션에서 **샘플 웨어하우스**를 선택하고 **sample-dw**라는 새 데이터 웨어하우스를 만듭니다.

    >**참고**: **만들기** 옵션이 사이드바에 고정되지 않은 경우 먼저 줄임표(**...**) 옵션을 선택해야 합니다.

    1분 정도 지나면 택시 승차 분석 시나리오에 대한 샘플 데이터로 새 웨어하우스가 만들어지고 채워집니다.

    ![새 웨어하우스의 스크린샷.](./Images/sample-data-warehouse.png)

## 데이터 웨어하우스 쿼리

SQL 쿼리 편집기는 IntelliSense, 코드 완료, 구문 강조 표시, 클라이언트 쪽 구문 분석 및 유효성 검사를 지원합니다. DDL(데이터 정의 언어), DML(데이터 조작 언어) 및 DCL(데이터 컨트롤 언어) 문을 실행할 수 있습니다.

1. **sample-dw** 데이터 웨어하우스 페이지의 **새 SQL 쿼리** 드롭다운 목록에서 **새 SQL 쿼리**를 선택합니다.

1. 새로운 빈 쿼리 창에 다음 Transact-SQL 코드를 입력합니다.

    ```sql
    SELECT 
    D.MonthName, 
    COUNT(*) AS TotalTrips, 
    SUM(T.TotalAmount) AS TotalRevenue 
    FROM dbo.Trip AS T
    JOIN dbo.[Date] AS D
        ON T.[DateID]=D.[DateID]
    GROUP BY D.MonthName;
    ```

1. **&#9655; 실행** 단추를 눌러 SQL 스크립트를 실행하고 결과를 확인합니다. 이 결과에는 총 이동 횟수와 월별 총 수익이 표시됩니다.

1. 다음 Transact-SQL 코드를 입력합니다.

    ```sql
   SELECT 
    D.DayName, 
    AVG(T.TripDurationSeconds) AS AvgDuration, 
    AVG(T.TripDistanceMiles) AS AvgDistance 
    FROM dbo.Trip AS T
    JOIN dbo.[Date] AS D
        ON T.[DateID]=D.[DateID]
    GROUP BY D.DayName;
    ```

1. 수정된 쿼리를 실행하고 결과를 확인합니다. 이 결과에는 요일별 평균 이동 시간과 거리가 표시됩니다.

1. 다음 Transact-SQL 코드를 입력합니다.

    ```sql
    SELECT TOP 10 
        G.City, 
        COUNT(*) AS TotalTrips 
    FROM dbo.Trip AS T
    JOIN dbo.Geography AS G
        ON T.DropoffGeographyID=G.GeographyID
    GROUP BY G.City
    ORDER BY TotalTrips DESC;
    ```

1. 수정된 쿼리를 실행하고 가장 자주 사용되는 상위 10개의 픽업 및 하차 위치를 보여 주는 결과를 확인합니다.

1. 모든 쿼리 탭을 닫습니다.

## 데이터 일관성 확인

분석 및 의사 결정을 위해 데이터가 정확하고 신뢰할 수 있는지 확인하려면 데이터 일관성을 확인해야 합니다. 일관성 없는 데이터는 부정확한 분석과 잘못된 결과로 이어질 수 있습니다. 

일관성을 확인하기 위해 데이터 웨어하우스를 쿼리하겠습니다.

1. **새 SQL 쿼리** 드롭다운 목록에서 **새 SQL 쿼리**를 선택합니다.

1. 새로운 빈 쿼리 창에 다음 Transact-SQL 코드를 입력합니다.

    ```sql
    -- Check for trips with unusually long duration
    SELECT COUNT(*) FROM dbo.Trip WHERE TripDurationSeconds > 86400; -- 24 hours
    ```

1. 수정된 쿼리를 실행하고 결과를 확인합니다. 이 결과는 기간이 비정상적으로 긴 모든 여행의 세부 정보를 보여 줍니다.

1. **새 SQL 쿼리** 드롭다운 목록에서 **새 SQL 쿼리**를 선택하여 두 번째 쿼리 탭을 추가합니다. 그런 다음 새 빈 쿼리 탭에서 다음 코드를 실행합니다.

    ```sql
    -- Check for trips with negative trip duration
    SELECT COUNT(*) FROM dbo.Trip WHERE TripDurationSeconds < 0;
    ```

1. 새로운 빈 쿼리 창에 다음 Transact-SQL 코드를 입력하고 실행합니다.

    ```sql
    -- Remove trips with negative trip duration
    DELETE FROM dbo.Trip WHERE TripDurationSeconds < 0;
    ```

    > **참고:** 일관되지 않은 데이터를 처리하는 방법에는 여러 가지가 있습니다. 이를 제거하는 대신 평균이나 중앙값과 같은 다른 값으로 바꾸는 것이 한 가지 대안입니다.

1. 모든 쿼리 탭을 닫습니다.

## 보기로 저장

보고서를 생성하기 위해 데이터를 사용할 사용자 그룹에 대해 특정 여행을 필터링해야 한다고 가정해 보겠습니다.

이전에 사용한 쿼리를 기반으로 뷰를 만들고 여기에 필터를 추가하겠습니다.

1. **새 SQL 쿼리** 드롭다운 목록에서 **새 SQL 쿼리**를 선택합니다.

1. 새로운 빈 쿼리 창에 다음 Transact-SQL 코드를 다시 입력하고 실행합니다.

    ```sql
    SELECT 
        D.DayName, 
        AVG(T.TripDurationSeconds) AS AvgDuration, 
        AVG(T.TripDistanceMiles) AS AvgDistance 
    FROM dbo.Trip AS T
    JOIN dbo.[Date] AS D
        ON T.[DateID]=D.[DateID]
    GROUP BY D.DayName;
    ```

1. 쿼리를 수정하여 `WHERE D.Month = 1`을 추가합니다. 그러면 1월의 기록만 포함하도록 데이터가 필터링됩니다. 최종 쿼리는 다음과 같아야 합니다.

    ```sql
    SELECT 
        D.DayName, 
        AVG(T.TripDurationSeconds) AS AvgDuration, 
        AVG(T.TripDistanceMiles) AS AvgDistance 
    FROM dbo.Trip AS T
    JOIN dbo.[Date] AS D
        ON T.[DateID]=D.[DateID]
    WHERE D.Month = 1
    GROUP BY D.DayName
    ```

1. 쿼리에서 SELECT 문의 텍스트를 선택합니다. 그런 다음 **&#9655; 실행** 단추를 클릭하고 **보기로 저장**을 선택합니다.

1. **vw_JanTrip**이라는 새 보기를 만듭니다.

1. **탐색기**에서 **스키마 >> dbo >> 보기**로 이동합니다. 방금 만든 *vw_JanTrip* 보기를 확인합니다.

1. 모든 쿼리 탭을 닫습니다.

> **추가 정보**: 데이터 웨어하우스 쿼리에 대한 자세한 내용은 Microsoft Fabric 설명서에서 [SQL 쿼리 편집기를 사용하여 쿼리](https://learn.microsoft.com/fabric/data-warehouse/sql-query-editor)를 참조하세요.

## 리소스 정리

이 연습에서는 쿼리를 사용하여 Microsoft Fabric 데이터 웨어하우스의 데이터에 대한 인사이트를 얻었습니다.

데이터 웨어하우스 탐색을 마쳤으면 이 연습을 위해 만든 작업 영역을 삭제할 수 있습니다.

1. 왼쪽 막대에서 작업 영역의 아이콘을 선택하여 포함된 모든 항목을 봅니다.
1. **작업 영역 설정**을 선택하고 **일반** 섹션에서 아래로 스크롤하여 **이 작업 영역 제거**를 선택합니다.
1. **삭제**를 선택하여 작업 영역을 삭제합니다.
