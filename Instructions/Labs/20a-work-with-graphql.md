---
lab:
  title: Microsoft Fabric에서 GraphQL용 API로 작업
  module: Get started with GraphQL in Microsoft Fabric
---

# Microsoft Fabric에서 GraphQL용 API로 작업

GraphQL용 Microsoft Fabric API는 널리 채택되고 친숙한 API 기술을 사용하여 여러 데이터 원본을 빠르고 효율적으로 쿼리할 수 있는 데이터 액세스 계층입니다. API를 사용하면 백엔드 데이터 원본의 세부 정보를 추상화하여 애플리케이션의 논리에 집중하고 클라이언트에 필요한 모든 데이터를 단일 호출로 제공할 수 있습니다. GraphQL은 간단한 쿼리 언어와 쉽게 조작할 수 있는 결과 집합을 사용하여 애플리케이션이 Fabric의 데이터에 액세스하는 데 걸리는 시간을 최소화합니다.

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

1. 모든 쿼리 탭을 닫습니다.

## GraphQL용 API 만들기

먼저 판매 주문 데이터를 노출하도록 GraphQL 엔드포인트를 설정합니다. 이 엔드포인트를 사용하면 날짜, 고객 및 제품과 같은 다양한 매개 변수를 기반으로 판매 주문을 쿼리할 수 있습니다.

1. Fabric 포털에서 작업 영역으로 이동하여 **+ 새 항목**을 선택합니다.
1. **데이터 개발** 섹션으로 이동하여 **GraphQL용 API**를 선택합니다.
1. 이름을 제공한 다음, **만들기**를 선택합니다.
1. GraphQL용 API의 기본 페이지에서 **데이터 원본 선택**을 선택합니다.
1. 연결 옵션을 선택하라는 메시지가 표시되면 **SSO(Single Sign-On) 인증을 사용하여 Fabric 데이터 원본에 연결**을 선택합니다.
1. **연결할 데이터 선택** 페이지에서 이전에 만든 `AdventureWorksLT` 데이터베이스를 선택합니다.
1. **연결**을 선택합니다.
1. **데이터 선택** 페이지에서 `SalesLT.Product` 테이블을 선택합니다. 
1. 데이터를 미리 보기하고 **로드**를 선택합니다.
1. **엔드포인트 복사**를 선택하고 공용 URL 링크를 니다. 이 작업은 필요하지 않지만 API 주소를 복사하는 위치입니다.

## 변형 사용 안 함

이제 API가 만들어졌으므로 이 시나리오에서 작업을 읽기 위한 판매 데이터만 공개하려고 합니다.

1. GraphQL용 API의 **스키마 탐색기**에서 **변형**을 펼칩니다.
1. 각 변형 옆의 **...**(줄임표)를 선택하고 **사용 안 함**을 선택합니다.

이렇게 하면 API를 통해 데이터를 수정하거나 업데이트할 수 없습니다. 즉, 데이터는 읽기 전용이며 사용자는 데이터를 보거나 쿼리할 수 있지만 변경은 할 수 없습니다.

## GraphQL을 사용하여 데이터 쿼리

이제 GraphQL을 사용하여 데이터를 쿼리하여 이름이 *"HL 로드 프레임"* 으로 시작하는 모든 제품을 찾아보겠습니다.

1. GraphQL 쿼리 편집기에서 다음 쿼리를 입력하고 실행합니다.

```json
query {
  products(filter: { Name: { startsWith: "HL Road Frame" } }) {
    items {
      ProductModelID
      Name
      ListPrice
      Color
      Size
      ModifiedDate
    }
  }
}
```

이 쿼리에서는 제품이 기본 유형이며 `ProductModelID`, `Name`, `ListPrice`, `Color`, `Size`, `ModifiedDate`에 대한 필드를 포함합니다. 이 쿼리는 이름이 *"HL 로드 프레임."* 으로 시작하는 제품 목록을 반환합니다.

> **추가 정보**: 플랫폼에서 사용 가능한 다른 구성 요소에 대해 자세히 알아보려면 Microsoft Fabric 설명서에서 [GraphQL용 Microsoft Fabric API란 무엇인가요?](https://learn.microsoft.com/fabric/data-engineering/api-graphql-overview)를 참조합니다.

이 연습에서는 Microsoft Fabric의 GraphQL을 사용하여 SQL 데이터베이스에서 데이터를 만들고, 쿼리하고, 노출했습니다.

## 리소스 정리

데이터베이스 탐색을 마쳤으면 이 연습을 위해 만든 작업 영역을 삭제할 수 있습니다.

1. 왼쪽 막대에서 작업 영역의 아이콘을 선택하여 포함된 모든 항목을 봅니다.
2. 도구 모음의 **...** 메뉴에서 **작업 영역 설정**을 선택합니다.
3. **일반** 섹션에서 **이 작업 영역 제거**를 선택합니다.

