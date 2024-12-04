---
lab:
  title: Microsoft Fabric에서 SQL Database 작업
  module: Get started with SQL Database in Microsoft Fabric
---

# Microsoft Fabric에서 SQL Database 작업

Microsoft Fabric의 SQL 데이터베이스는 Azure SQL Database를 기반으로 하는 개발자 친화적인 트랜잭션 데이터베이스로, Fabric에서 운영 데이터베이스를 쉽게 만들 수 있게 해줍니다. Fabric의 SQL 데이터베이스는 SQL Database 엔진을 Azure SQL Database로 사용합니다.

이 랩을 완료하는 데 약 **30**분이 걸립니다.

> **참고**: 이 연습을 완료하려면 [Microsoft Fabric 평가판](https://learn.microsoft.com/fabric/get-started/fabric-trial)이 필요합니다.

## 작업 영역 만들기

패브릭에서 데이터를 사용하기 전에 패브릭 평가판을 사용하도록 설정된 작업 영역을 만듭니다.

1. [Microsoft Fabric 홈페이지](https://app.fabric.microsoft.com/home?experience=fabric)(`https://app.fabric.microsoft.com/home?experience=fabric`)에서 확인합니다.
1. 왼쪽 메뉴 모음에서 **새 작업 영역**을 선택합니다.
1. Fabric 용량이 포함된 라이선스 모드(*평가판*, *프리미엄* 또는 *Fabric*)를 선택하여 원하는 이름으로 새 작업 영역을 만듭니다.
1. 새 작업 영역이 열리면 비어 있어야 합니다.

    ![Fabric의 빈 작업 영역 스크린샷.](./Images/new-workspace.png)

## 샘플 데이터로 SQL데이터베이스 만들기

작업 영역이 있으므로 이제 SQL 데이터베이스를 만들어야 합니다.

1. Fabric 포털의 왼쪽 패널에서 **+ 새 항목**을 선택합니다.
1. **데이터 저장** 섹션으로 이동하여 **SQL Database**를 선택합니다.
1. 데이터베이스 이름으로 **AdventureWorksLT**를 입력하고 **만들기**를 선택합니다.
1. 데이터베이스를 만든 후에는 **샘플 데이터** 카드에서 데이터베이스로 샘플 데이터를 로드할 수 있습니다.

    1분 정도 지나면 데이터베이스가 해당 시나리오에 대한 샘플 데이터로 채워집니다.

    ![샘플 데이터가 로드된 새 데이터베이스의 스크린샷.](./Images/sql-database-sample.png)

## SQL 데이터베이스 쿼리

SQL 쿼리 편집기는 IntelliSense, 코드 완료, 구문 강조 표시, 클라이언트 쪽 구문 분석 및 유효성 검사를 지원합니다. DDL(데이터 정의 언어), DML(데이터 조작 언어) 및 DCL(데이터 컨트롤 언어) 문을 실행할 수 있습니다.

1. **AdventureWorksLT** 데이터베이스 페이지에서 **홈**으로 이동하고 **새 쿼리**를 선택합니다.

1. 새로운 빈 쿼리 창에 다음 T-SQL 코드를 입력하고 실행합니다.

    ```sql
    SELECT 
        p.Name AS ProductName,
        pc.Name AS CategoryName,
        p.ListPrice
    FROM 
        SalesLT.Product p
    INNER JOIN 
        SalesLT.ProductCategory pc ON p.ProductCategoryID = pc.ProductCategoryID
    ORDER BY 
    p.ListPrice DESC;
    ```
    
    이 쿼리는 제품 이름, 범주 및 정가를 내림차순으로 정렬하여 표시하기 위해 `Product` 및 `ProductCategory` 테이블을 병합합니다.

1. 새로운 빈 쿼리 편집기에 다음 T-SQL 코드를 입력하고 실행합니다.

    ```sql
   SELECT 
        c.FirstName,
        c.LastName,
        soh.OrderDate,
        soh.SubTotal
    FROM 
        SalesLT.Customer c
    INNER JOIN 
        SalesLT.SalesOrderHeader soh ON c.CustomerID = soh.CustomerID
    ORDER BY 
        soh.OrderDate DESC;
    ```

    이 쿼리는 주문 날짜 및 소계와 함께 고객 목록을 검색하고 주문 날짜에 따라 내림차순으로 정렬합니다. 

1. 모든 쿼리 탭을 닫습니다.

## 외부 데이터 원본과 데이터 통합

공휴일에 대한 외부 데이터를 판매 주문과 통합합니다. 그 다음, 공휴일과 일치하는 판매 주문을 식별하여 휴일이 판매 활동에 미치는 영향에 대한 인사이트를 제공합니다.

1. **홈**으로 이동하여 **새 쿼리**를 선택합니다.

1. 새로운 빈 쿼리 창에 다음 T-SQL 코드를 입력하고 실행합니다.

    ```sql
    CREATE TABLE SalesLT.PublicHolidays (
        CountryOrRegion NVARCHAR(50),
        HolidayName NVARCHAR(100),
        Date DATE,
        IsPaidTimeOff BIT
    );
    ```

    이 쿼리는 다음 단계를 준비하기 위해 `SalesLT.PublicHolidays` 테이블을 만듭니다.

1. 새로운 빈 쿼리 편집기에 다음 T-SQL 코드를 입력하고 실행합니다.

    ```sql
    INSERT INTO SalesLT.PublicHolidays (CountryOrRegion, HolidayName, Date, IsPaidTimeOff)
    VALUES
        ('Canada', 'Victoria Day', '2024-02-19', 1),
        ('United Kingdom', 'Christmas Day', '2024-12-25', 1),
        ('United Kingdom', 'Spring Bank Holiday', '2024-05-27', 1),
        ('United States', 'Thanksgiving Day', '2024-11-28', 1);
    ```
    
    이 예제에서 이 쿼리는 2024년 캐나다, 영국 및 미국의 공휴일을 `SalesLT.PublicHolidays` 테이블에 삽입합니다.    

1. 새 쿼리 편집기 또는 기존 쿼리 편집기에서 다음 T-SQL 코드를 입력하고 실행합니다.

    ```sql
    -- Insert new addresses into SalesLT.Address
    INSERT INTO SalesLT.Address (AddressLine1, City, StateProvince, CountryRegion, PostalCode, rowguid, ModifiedDate)
    VALUES
        ('123 Main St', 'Seattle', 'WA', 'United States', '98101', NEWID(), GETDATE()),
        ('456 Maple Ave', 'Toronto', 'ON', 'Canada', 'M5H 2N2', NEWID(), GETDATE()),
        ('789 Oak St', 'London', 'England', 'United Kingdom', 'EC1A 1BB', NEWID(), GETDATE());
    
    -- Insert new orders into SalesOrderHeader
    INSERT INTO SalesLT.SalesOrderHeader (
        SalesOrderID, RevisionNumber, OrderDate, DueDate, ShipDate, Status, OnlineOrderFlag, 
        PurchaseOrderNumber, AccountNumber, CustomerID, ShipToAddressID, BillToAddressID, 
        ShipMethod, CreditCardApprovalCode, SubTotal, TaxAmt, Freight, Comment, rowguid, ModifiedDate
    )
    VALUES
        (1001, 1, '2024-12-25', '2024-12-30', '2024-12-26', 1, 1, 'PO12345', 'AN123', 1, (SELECT TOP 1 AddressID FROM SalesLT.Address WHERE AddressLine1 = '789 Oak St'), (SELECT TOP 1 AddressID FROM SalesLT.Address WHERE AddressLine1 = '123 Main St'), 'Ground', '12345', 100.00, 10.00, 5.00, 'New Order 1', NEWID(), GETDATE()),
        (1002, 1, '2024-11-28', '2024-12-03', '2024-11-29', 1, 1, 'PO67890', 'AN456', 2, (SELECT TOP 1 AddressID FROM SalesLT.Address WHERE AddressLine1 = '123 Main St'), (SELECT TOP 1 AddressID FROM SalesLT.Address WHERE AddressLine1 = '456 Maple Ave'), 'Air', '67890', 200.00, 20.00, 10.00, 'New Order 2', NEWID(), GETDATE()),
        (1003, 1, '2024-02-19', '2024-02-24', '2024-02-20', 1, 1, 'PO54321', 'AN789', 3, (SELECT TOP 1 AddressID FROM SalesLT.Address WHERE AddressLine1 = '456 Maple Ave'), (SELECT TOP 1 AddressID FROM SalesLT.Address WHERE AddressLine1 = '789 Oak St'), 'Sea', '54321', 300.00, 30.00, 15.00, 'New Order 3', NEWID(), GETDATE()),
        (1004, 1, '2024-05-27', '2024-06-01', '2024-05-28', 1, 1, 'PO98765', 'AN321', 4, (SELECT TOP 1 AddressID FROM SalesLT.Address WHERE AddressLine1 = '789 Oak St'), (SELECT TOP 1 AddressID FROM SalesLT.Address WHERE AddressLine1 = '789 Oak St'), 'Ground', '98765', 400.00, 40.00, 20.00, 'New Order 4', NEWID(), GETDATE());
    ```

    이 코드는 데이터베이스에 새 주소와 주문을 추가하여 다른 국가의 가상 주문을 시뮬레이션합니다.

1. 새 쿼리 편집기 또는 기존 쿼리 편집기에서 다음 T-SQL 코드를 입력하고 실행합니다.

    ```sql
    SELECT DISTINCT soh.SalesOrderID, soh.OrderDate, ph.HolidayName, ph.CountryOrRegion
    FROM SalesLT.SalesOrderHeader AS soh
    INNER JOIN SalesLT.Address a
        ON a.AddressID = soh.ShipToAddressID
    INNER JOIN SalesLT.PublicHolidays AS ph
        ON soh.OrderDate = ph.Date AND a.CountryRegion = ph.CountryOrRegion
    ```

    쿼리가 각 국가의 공휴일과 일치하는 판매 주문을 식별하는 방법을 확인하여 결과를 잠시 관찰합니다. 이를 통해 주문 패턴과 휴일이 판매 활동에 미치는 잠재적 영향에 대한 중요한 인사이트를 얻을 수 있습니다.

1. 모든 쿼리 탭을 닫습니다.

## 보안 데이터

특정 사용자 그룹이 보고서를 생성하기 위해 미국 데이터에만 액세스할 수 있어야 한다고 가정해 봅니다.

이전에 사용한 쿼리를 기반으로 뷰를 만들고 여기에 필터를 추가하겠습니다.

1. 새로운 빈 쿼리 창에 다음 T-SQL 코드를 입력하고 실행합니다.

    ```sql
    CREATE VIEW SalesLT.vw_SalesOrderHoliday AS
    SELECT DISTINCT soh.SalesOrderID, soh.OrderDate, ph.HolidayName, ph.CountryOrRegion
    FROM SalesLT.SalesOrderHeader AS soh
    INNER JOIN SalesLT.Address a
        ON a.AddressID = soh.ShipToAddressID
    INNER JOIN SalesLT.PublicHolidays AS ph
        ON soh.OrderDate = ph.Date AND a.CountryRegion = ph.CountryOrRegion
    WHERE a.CountryRegion = 'United Kingdom';
    ```

1. 새 쿼리 편집기 또는 기존 쿼리 편집기에서 다음 T-SQL 코드를 입력하고 실행합니다.

    ```sql
    -- Create the role
    CREATE ROLE SalesOrderRole;
    
    -- Grant select permission on the view to the role
    GRANT SELECT ON SalesLT.vw_SalesOrderHoliday TO SalesOrderRole;
    ```

    `SalesOrderRole` 역할에 멤버로 추가된 모든 사용자는 필터링된 보기에만 액세스할 수 있습니다. 이 역할의 사용자가 다른 사용자 개체에 액세스하려고 하면 다음과 유사한 오류 메시지가 표시됩니다.

    ```
    Msg 229, Level 14, State 5, Line 1
    The SELECT permission was denied on the object 'ObjectName', database 'DatabaseName', schema 'SchemaName'.
    ```

> **추가 정보**: 해당 플랫폼에서 사용할 수 있는 다른 구성 요소에 대한 자세한 내용은 Microsoft Fabric 설명서에서 [Microsoft Fabric이란 무엇인가요?](https://learn.microsoft.com/fabric/get-started/microsoft-fabric-overview)를 참조합니다.

이 연습에서는 Microsoft Fabric의 SQL 데이터베이스에서 데이터를 만들고, 쿼리하고, 보호하는 작업을 수행했습니다.

## 리소스 정리

데이터베이스 탐색을 마쳤으면 이 연습을 위해 만든 작업 영역을 삭제할 수 있습니다.

1. 왼쪽 막대에서 작업 영역의 아이콘을 선택하여 포함된 모든 항목을 봅니다.
2. 도구 모음의 **...** 메뉴에서 **작업 영역 설정**을 선택합니다.
3. **일반** 섹션에서 **이 작업 영역 제거**를 선택합니다.
