---
lab:
  title: Apache Spark를 사용하여 데이터 분석
  module: Use Apache Spark to work with files in a lakehouse
---

# Apache Spark를 사용하여 데이터 분석

Apache Spark는 분산 데이터 처리를 위한 오픈 소스 엔진으로, Data Lake Storage에서 대량의 데이터를 탐색, 처리, 분석하는 데 널리 사용됩니다. Spark는 Azure HDInsight, Azure Databricks, Azure Synapse Analytics 및 Microsoft Fabric을 비롯한 많은 데이터 플랫폼 제품에서 처리 옵션으로 사용할 수 있습니다. Spark의 이점 중 하나는 Java, Scala, Python, SQL을 포함한 다양한 프로그래밍 언어를 지원한다는 점입니다. Spark는 데이터 정리 및 조작, 통계 분석 및 기계 학습, 데이터 분석 및 시각화를 포함한 데이터 처리 워크로드를 위한 매우 유연한 솔루션입니다.

이 랩을 완료하는 데 약 **45**분이 걸립니다.

> **참고**: 이 연습을 완료하려면 [Microsoft Fabric 평가판](https://learn.microsoft.com/fabric/get-started/fabric-trial)이 필요합니다.

## 작업 영역 만들기

패브릭에서 데이터를 사용하기 전에 패브릭 평가판을 사용하도록 설정된 작업 영역을 만듭니다.

1. [Microsoft Fabric 홈페이지](https://app.fabric.microsoft.com)의 Synapse `https://app.fabric.microsoft.com`데이터 엔지니어** 선택합니다**.
1. 왼쪽 메뉴 모음에서 **작업 영역**을 선택합니다(아이콘은 와 유사함).
1. 선택한 이름으로 새 작업 영역을 만들고 패브릭 용량(*평가판*, *프리미엄* 또는 *패브릭*)이 포함된 **고급** 섹션에서 라이선스 모드를 선택합니다.
1. 새 작업 영역이 열리면 비어 있어야 합니다.

    ![Fabric의 빈 작업 영역 스크린샷.](./Images/new-workspace.png)

## 레이크하우스 만들기 및 파일 업로드

이제 작업 영역이 있으므로 분석하려는 데이터 파일에 대한 데이터 레이크하우스를 만들어야 합니다.

1. **Synapse Data Engineering** 홈페이지에서 원하는 이름으로 새 **레이크하우스**를 만듭니다.

    1분 정도 지나면 빈 레이크하우스가 새로 만들어집니다. 분석을 위해 일부 데이터를 데이터 레이크하우스에 수집해야 합니다. 이 작업을 수행하는 방법에는 여러 가지가 있지만, 이 연습에서는 로컬 컴퓨터(또는 해당하는 경우 랩 VM)의 텍스트 파일 폴더를 다운로드하여 추출한 다음 레이크하우스에 업로드하기만 하면 됩니다.

1. 에서 이 연습의 [데이터 파일을](https://github.com/MicrosoftLearning/dp-data/raw/main/orders.zip) 다운로드하고 추출 `https://github.com/MicrosoftLearning/dp-data/raw/main/orders.zip`합니다.

1. 압축된 보관 파일을 추출한 후 2019.csv, 2020.csv 및**** 2021.csv****** CSV 파일이 포함된 주문이라는 **** 폴더가 있는지 확인합니다.**
1. 레이크하우스가 포함된 웹 브라우저 탭으로 돌아가서 탐색기** 창의 파일** 폴더에 **대한 **...** 메뉴에서 업로드** 및 **업로드 폴더**를 선택한 **다음 로컬 컴퓨터(또는 해당하는 경우 랩 VM)의 주문** 폴더를 레이크하우스에 업로드**** 합니다.
1. 파일이 업로드된 후 파일을 확장하고 **주문 폴더를 선택하고 **다음과 같이 CSV 파일이 업로드되었는지 확인합니다**.**

    ![레이크하우스에 업로드된 파일의 스크린샷](./Images/uploaded-files.png)

## Notebook 만들기

Apache Spark에서 데이터를 사용하려면 Notebook*을 *만들 수 있습니다. Notebook은 코드를 작성하고 실행할 수 있는 대화형 환경을 제공하고(여러 언어로) 노트를 추가하여 문서를 작성할 수 있습니다.

1. 데이터 레이크에서 주문 폴더의 내용을 보는 동안 홈**페이지의 **전자 필기장 열기 메뉴에서 새 전자** 필기장을** 선택합니다**.** **** 

    몇 초 후에 단일 *셀*이 포함된 새 Notebook이 열립니다. Notebook은 *코드* 또는 *markdown*(서식이 지정된 텍스트)을 포함할 수 있는 하나 이상의 셀로 구성됩니다.

2. 첫 번째 셀(현재 *코드* 셀)을 선택한 다음 오른쪽 상단에 있는 동적 도구 모음에서 **M&#8595;** 단추를 눌러 셀을 *markdown* 셀로 변환합니다.

    셀이 markdown 셀로 변경되면 포함된 텍스트가 렌더링됩니다.

3. **&#128393;**(편집) 단추를 사용하여 셀을 편집 모드로 전환한 다음 다음과 같이 markdown을 수정합니다.

    ```
   # Sales order data exploration

   Use the code in this notebook to explore sales order data.
    ```

4. 셀 외부의 전자 필기장 아무 곳이나 클릭하여 편집을 중지하고 렌더링된 markdown을 확인합니다.

## 데이터 프레임에 데이터 로드

이제 데이터를 데이터 프레임*에 *로드하는 코드를 실행할 준비가 되었습니다. Spark의 데이터 프레임은 Python의 Pandas 데이터 프레임과 유사하며 행 및 열에서 데이터를 사용하기 위한 일반적인 구조를 제공합니다.

> **참고**: Spark는 Scala, Java 등을 비롯한 여러 코딩 언어를 지원합니다. 이 연습에서는 Spark 최적화 Python 변형인 PySpark*를 사용합니다*. PySpark는 Spark에서 가장 일반적으로 사용되는 언어 중 하나이며 패브릭 Notebook의 기본 언어입니다.

1. 전자 필기장이 표시되면 탐색기 창에서 **Lakehouses**를 확장**한 다음, 레이크하우스의 파일** 목록을 확장하고 **주문 폴더를** 선택하여 **다음과 같이 CSV 파일이 Notebook 편집기 옆에 나열되도록** 합니다.

    ![파일 창이 있는 전자 필기장의 스크린샷](./Images/notebook-files.png)

1. 2019.csv ...** 메뉴에서 데이터**** > Spark** 로드를 선택합니다.******** 다음 코드를 포함하는 새 코드 셀을 Notebook에 추가해야 합니다.

    ```python
   df = spark.read.format("csv").option("header","true").load("Files/orders/2019.csv")
   # df now is a Spark DataFrame containing CSV data from "Files/orders/2019.csv".
   display(df)
    ```

    > **팁**: 아이콘을 사용하여 **<<** 왼쪽의 Lakehouse 탐색기 창을 숨길 수 있습니다. 이렇게 하면 전자 필기장을 집중하게 됩니다.

1. 셀 왼쪽의 **&#9655;셀 실행** 단추를 이용하여 실행합니다.

    > **참고**: Spark 코드를 처음 실행했으므로 Spark 세션을 시작해야 합니다. 이는 세션의 첫 번째 실행이 완료되는 데 1분 정도 걸릴 수 있음을 의미합니다. 후속 실행은 더 빨라질 것입니다.

1. 셀 명령이 완료되면 셀 아래의 출력을 검토합니다. 다음과 유사한 출력을 확인할 수 있습니다.

    | 색인 | SO43701 | 11 | 2019-07-01 | Christy Zhu | christy12@adventure-works.com | Mountain-100 Silver, 44 | 16 | 3399.99 | 271.9992 |
    | -- | -- | -- | -- | -- | -- | -- | -- | -- | -- |
    | 1 | SO43704 | 1 | 2019-07-01 | Julio Ruiz | julio1@adventure-works.com | Mountain-100 Black, 48 | 1 | 3374.99 | 269.9992 |
    | 2 | SO43705 | 1 | 2019-07-01 | Curtis Lu | curtis9@adventure-works.com | Mountain-100 Silver, 38 | 1 | 3399.99 | 271.9992 |
    | ... | ... | ... | ... | ... | ... | ... | ... | ... | ... |

    출력은 2019.csv 파일의 데이터 행과 열을 보여 줍니다. 그러나 열 머리글은 제대로 표시되지 않습니다. 데이터를 데이터 프레임에 로드하는 데 사용되는 기본 코드는 CSV 파일에 첫 번째 행의 열 이름이 포함되어 있다고 가정하지만, 이 경우 CSV 파일에는 헤더 정보가 없는 데이터만 포함됩니다.

1. 다음과 같이 헤더** 옵션을 false**로 설정**하도록 **코드를 수정합니다.

    ```python
   df = spark.read.format("csv").option("header","false").load("Files/orders/2019.csv")
   # df now is a Spark DataFrame containing CSV data from "Files/orders/2019.csv".
   display(df)
    ```

1. 셀을 다시 실행하고 다음과 유사한 출력을 검토합니다.

   | 색인 | _c0 | _c1 | _c2 | _c3 | _c4 | _c5 | _c6 | _c7 | _c8 |
    | -- | -- | -- | -- | -- | -- | -- | -- | -- | -- |
    | 1 | SO43701 | 11 | 2019-07-01 | Christy Zhu | christy12@adventure-works.com | Mountain-100 Silver, 44 | 16 | 3399.99 | 271.9992 |
    | 2 | SO43704 | 1 | 2019-07-01 | Julio Ruiz | julio1@adventure-works.com | Mountain-100 Black, 48 | 1 | 3374.99 | 269.9992 |
    | 3 | SO43705 | 1 | 2019-07-01 | Curtis Lu | curtis9@adventure-works.com | Mountain-100 Silver, 38 | 1 | 3399.99 | 271.9992 |
    | ... | ... | ... | ... | ... | ... | ... | ... | ... | ... |

    이제 데이터 프레임은 첫 번째 행을 데이터 값으로 올바르게 포함하지만 열 이름은 자동으로 생성되며 그다지 유용하지 않습니다. 데이터를 이해하려면 파일의 데이터 값에 대한 올바른 스키마 및 데이터 형식을 명시적으로 정의해야 합니다.

1. 다음과 같이 코드를 수정하여 스키마를 정의하고 데이터를 로드할 때 적용합니다.

    ```python
   from pyspark.sql.types import *

   orderSchema = StructType([
       StructField("SalesOrderNumber", StringType()),
       StructField("SalesOrderLineNumber", IntegerType()),
       StructField("OrderDate", DateType()),
       StructField("CustomerName", StringType()),
       StructField("Email", StringType()),
       StructField("Item", StringType()),
       StructField("Quantity", IntegerType()),
       StructField("UnitPrice", FloatType()),
       StructField("Tax", FloatType())
       ])

   df = spark.read.format("csv").schema(orderSchema).load("Files/orders/2019.csv")
   display(df)
    ```

1. 수정된 셀을 실행하고 다음과 유사한 출력을 검토합니다.

   | 색인 | SalesOrderNumber | SalesOrderLineNumber | OrderDate | CustomerName | 전자 메일 | Item | 수량 | 단가 | 세금 |
    | -- | -- | -- | -- | -- | -- | -- | -- | -- | -- |
    | 1 | SO43701 | 11 | 2019-07-01 | Christy Zhu | christy12@adventure-works.com | Mountain-100 Silver, 44 | 16 | 3399.99 | 271.9992 |
    | 2 | SO43704 | 1 | 2019-07-01 | Julio Ruiz | julio1@adventure-works.com | Mountain-100 Black, 48 | 1 | 3374.99 | 269.9992 |
    | 3 | SO43705 | 1 | 2019-07-01 | Curtis Lu | curtis9@adventure-works.com | Mountain-100 Silver, 38 | 1 | 3399.99 | 271.9992 |
    | ... | ... | ... | ... | ... | ... | ... | ... | ... | ... |

    이제 데이터 프레임에는 올바른 열 이름이 포함됩니다(인덱**스 외에도 **각 행의 서수 위치에 따라 모든 데이터 프레임의 기본 제공 열임). 열의 데이터 형식은 셀의 시작 부분에서 가져온 Spark SQL 라이브러리에 정의된 표준 형식 집합을 사용하여 지정됩니다.

1. 데이터 프레임에는 2019.csv** 파일의 **데이터만 포함됩니다. 파일 경로가 wild카드를 사용하여 \* orders 폴더의 모든 파일**에서 판매 주문 데이터를 읽도록 코드를 수정합니다**.

    ```python
   from pyspark.sql.types import *

   orderSchema = StructType([
       StructField("SalesOrderNumber", StringType()),
       StructField("SalesOrderLineNumber", IntegerType()),
       StructField("OrderDate", DateType()),
       StructField("CustomerName", StringType()),
       StructField("Email", StringType()),
       StructField("Item", StringType()),
       StructField("Quantity", IntegerType()),
       StructField("UnitPrice", FloatType()),
       StructField("Tax", FloatType())
       ])

   df = spark.read.format("csv").schema(orderSchema).load("Files/orders/*.csv")
   display(df)
    ```

1. 수정된 코드 셀을 실행하고 2019년, 2020년 및 2021년 매출을 포함해야 하는 출력을 검토합니다.

    **참고**: 행의 하위 집합만 표시되므로 모든 연도의 예제를 볼 수 없습니다.

## 데이터 프레임에서 데이터 탐색

데이터 프레임 개체에는 포함된 데이터를 필터링, 그룹화 및 조작하는 데 사용할 수 있는 다양한 함수가 포함되어 있습니다.

### 데이터 프레임 필터링

1. 현재 셀 출력의 왼쪽 아래에 있는 마우스를 이동할 때 나타나는 + 코드 링크를 사용하여 **새 코드** 셀을 추가합니다(또는 메뉴 모음의 편집** 탭에서 **+ 코드 셀** 추가 선택**). 그런 다음, 다음 코드를 입력합니다.

    ```Python
   customers = df['CustomerName', 'Email']
   print(customers.count())
   print(customers.distinct().count())
   display(customers.distinct())
    ```

2. 새 코드 셀을 실행하고 결과를 검토합니다. 다음 세부 정보를 살펴봅니다.
    - 데이터 프레임에서 작업을 수행하면 그 결과는 새 데이터 프레임이 됩니다(이 경우 **df** 데이터 프레임에서 열의 특정 하위 집합을 선택하여 새 **customers** 데이터 프레임이 생성됨).
    - 데이터 프레임은 포함된 데이터를 요약하고 필터링하는 데 사용할 수 있는 **count** 및 **distinct** 함수를 제공합니다.
    - 구문은 `dataframe['Field1', 'Field2', ...]` 열의 하위 집합을 정의하는 약식 방법입니다. **select** 메서드를 사용할 수도 있으므로 위 코드의 첫 번째 줄을 `customers = df.select("CustomerName", "Email")`과 같이 작성할 수 있습니다.

3. 다음과 같이 코드를 수정합니다.

    ```Python
   customers = df.select("CustomerName", "Email").where(df['Item']=='Road-250 Red, 52')
   print(customers.count())
   print(customers.distinct().count())
   display(customers.distinct())
    ```

4. 수정된 코드를 실행하여 *Road-250 Red, 52* 제품을 구매한 고객을 확인합니다. 한 함수의 출력이 다음 함수의 입력이 되도록 여러 함수를 함께 “연결”할 수 있습니다. 이 경우에 **select** 메서드에서 만든 데이터 프레임은 필터링 조건을 적용하는 데 사용되는 **where** 메서드의 소스 데이터 프레임이 됩니다.

### 데이터 프레임에서 데이터 집계 및 그룹화

1. Notebook에 새 코드 셀을 추가하고 Notebook에 다음 코드를 입력합니다.

    ```Python
   productSales = df.select("Item", "Quantity").groupBy("Item").sum()
   display(productSales)
    ```

2. 추가한 코드 셀을 실행하면 결과에 제품별로 그룹화된 주문 수량의 합계가 표시됩니다. **groupBy** 메서드는 행을 *Item*별로 그룹화하고 후속 **sum** 집계 함수는 나머지 모든 숫자 열(이 경우 *Quantity*)에 적용됩니다.

3. Notebook에 다른 새 코드 셀을 추가하고 Notebook에 다음 코드를 입력합니다.

    ```Python
   from pyspark.sql.functions import *

   yearlySales = df.select(year(col("OrderDate")).alias("Year")).groupBy("Year").count().orderBy("Year")
   display(yearlySales)
    ```

4. 추가한 코드 셀을 실행하면 결과에 연간 판매 주문 수가 표시되는지 확인합니다. **select** 메서드에는 OrderDate* 필드의 *연도 구성 요소를 추출하는 SQL **연도** 함수가 포함되어 있습니다(따라서 코드에 Spark SQL 라이브러리에서 함수를 가져오는 import** 문이 포함되어 **있습니다). 그런 다음 별칭 메서드를** 사용하여 **추출된 연도 값에 열 이름을 할당합니다. 그런 다음 데이터를 파생된 *Year* 열로 그룹화하고 각 그룹의 행 수를 계산한 후, 마지막으로 **orderBy** 메서드를 사용하여 결과 데이터 프레임을 정렬합니다.

## Spark를 사용하여 데이터 파일 변환

데이터 엔지니어의 일반적인 작업은 특정 형식 또는 구조로 데이터를 수집하고 추가 다운스트림 처리 또는 분석을 위해 변환하는 것입니다.

### 데이터 프레임 메서드 및 함수를 사용하여 데이터 변환

1. Notebook에 다른 새 코드 셀을 추가하고 Notebook에 다음 코드를 입력합니다.

    ```Python
   from pyspark.sql.functions import *

   ## Create Year and Month columns
   transformed_df = df.withColumn("Year", year(col("OrderDate"))).withColumn("Month", month(col("OrderDate")))

   # Create the new FirstName and LastName fields
   transformed_df = transformed_df.withColumn("FirstName", split(col("CustomerName"), " ").getItem(0)).withColumn("LastName", split(col("CustomerName"), " ").getItem(1))

   # Filter and reorder columns
   transformed_df = transformed_df["SalesOrderNumber", "SalesOrderLineNumber", "OrderDate", "Year", "Month", "FirstName", "LastName", "Email", "Item", "Quantity", "UnitPrice", "Tax"]

   # Display the first five orders
   display(transformed_df.limit(5))
    ```

2. 코드를 실행하여 다음 변환을 사용하여 원래 순서 데이터에서 새 데이터 프레임을 만듭니다.
    - OrderDate** 열에 **따라 Year** 및 **Month** 열을 추가**합니다.
    - CustomerName** 열을 기반으로 **FirstName** 및 **LastName** 열을 추가**합니다.
    - 열을 필터링하고 순서를 변경하여 **CustomerName** 열을 제거합니다.

3. 출력을 검토하고 데이터에 대한 변환이 이루어졌는지 확인합니다.

    Spark SQL 라이브러리의 모든 기능을 사용하여 행 필터링, 열 파생, 제거, 이름 바꾸기 및 기타 필요한 데이터 수정 사항을 적용하여 데이터를 변환할 수 있습니다.

    > **팁**: 데이터 프레임 개체의 [메서드에 대한 자세한 내용은 Spark 데이터 프레임 설명서를](https://spark.apache.org/docs/latest/api/python/reference/pyspark.sql/dataframe.html) 참조하세요.

### 변환된 데이터 저장

1. 다음 코드가 포함된 새 셀을 추가하여 변환된 데이터 프레임을 Parquet 형식으로 저장합니다(이미 있는 경우 데이터 덮어쓰기).

    ```Python
   transformed_df.write.mode("overwrite").parquet('Files/transformed_data/orders')
   print ("Transformed data saved!")
    ```

    > **참고**: 일반적으로 *Parquet* 형식은 분석 저장소에 대한 추가 분석 또는 수집에 사용할 데이터 파일에 선호됩니다. Parquet은 대부분의 대규모 데이터 분석 시스템에서 지원하는 매우 효율적인 형식입니다. 실제로 데이터 변환 요구 사항은 단순히 데이터를 다른 형식(예: CSV)에서 Parquet으로 변환하는 것일 수 있습니다.

2. 셀을 실행하고 데이터가 저장되었다는 메시지를 기다립니다. **그런 다음 왼쪽의 Lakehouses** 창에서 **파일** 노드에 대한 **...** 메뉴에서 새로 고침**을 선택하고 **transformed_orders** 폴더를 선택하여 **주문**이라는 **새 폴더가 포함되어 있는지 확인합니다. 그러면 하나 이상의 Parquet 파일이 포함됩니다.

    ![parquet 파일이 포함된 폴더의 스크린샷](./Images/saved-parquet.png)

3. 다음 코드가 포함된 새 셀을 추가하여 transformed_orders/orders 폴더의 parquet 파일 **에서 새 데이터 프레임을 로드합니다** .

    ```Python
   orders_df = spark.read.format("parquet").load("Files/transformed_data/orders")
   display(orders_df)
    ```

4. 셀을 실행하고 결과에 parquet 파일에서 로드된 순서 데이터가 표시되는지 확인합니다.

### 분할된 파일에 데이터 저장

1. 다음 코드를 사용하여 새 셀을 추가합니다. 는 데이터 프레임을 저장하고 연도** 및 **월**별로 **데이터를 분할합니다.

    ```Python
   orders_df.write.partitionBy("Year","Month").mode("overwrite").parquet("Files/partitioned_data")
   print ("Transformed data saved!")
    ```

2. 셀을 실행하고 데이터가 저장되었다는 메시지를 기다립니다. **그런 다음 왼쪽의 Lakehouses** 창에서 **파일** 노드에 대한 **...** 메뉴에서 새로 고침**을 선택하고 **partitioned_orders** 폴더를 확장**하여 **Year=xxxx***라는 폴더의 계층 구조가 포함되어 있는지 확인합니다. 각 폴더에는 **Month=** xxxx***라는 폴더가 포함되어 있습니다. 매월 폴더에는 해당 월의 주문이 포함된 parquet 파일이 포함되어 있습니다.

    ![분할된 데이터 파일의 계층 구조 스크린샷](./Images/partitioned-files.png)

    데이터 파일 분할은 대량의 데이터를 처리할 때 성능을 최적화하는 일반적인 방법입니다. 이 기술을 사용하면 성능이 크게 향상되고 데이터를 더 쉽게 필터링할 수 있습니다.

3. 다음 코드가 포함된 새 셀을 추가하여 orders.parquet** 파일에서 새 데이터 프레임을 **로드합니다.

    ```Python
   orders_2021_df = spark.read.format("parquet").load("Files/partitioned_data/Year=2021/Month=*")
   display(orders_2021_df)
    ```

4. 셀을 실행하고 결과에 2021년 판매 주문 데이터가 표시되는지 확인합니다. 경로에 지정된 분할 열(**연도** 및 **월**)은 데이터 프레임에 포함되지 않습니다.

## 테이블 및 SQL 작업

앞에서 본 것처럼 데이터 프레임 개체의 네이티브 메서드를 사용하면 파일의 데이터를 매우 효과적으로 쿼리하고 분석할 수 있습니다. 그러나 많은 데이터 분석가가 SQL 구문을 사용하여 쿼리할 수 있는 테이블을 사용하는 것이 더 편합니다. Spark는 *관계형 테이블을 정의할 수 있는 메타스토어를* 제공합니다. 데이터 프레임 개체를 제공하는 Spark SQL 라이브러리는 메타스토어의 테이블을 쿼리하는 데 SQL 문을 사용할 수도 있습니다. 이러한 Spark 기능을 사용하면 데이터 레이크의 유연성과 관계형 데이터 웨어하우스의 구조적 데이터 스키마 및 SQL 기반 쿼리를 결합할 수 있으므로 "data lakehouse"라는 용어를 사용할 수 있습니다.

### 테이블 만들기

Spark 메타스토어의 테이블은 데이터 레이크의 파일에 대한 관계형 추상화입니다. 테이블을 관리할* 수 있습니다*(이 경우 파일은 메타스토어에서 관리됨) 외부 ** (이 경우 테이블은 메타스토어와 독립적으로 관리하는 데이터 레이크의 파일 위치를 참조).

1. Notebook에 새 코드 셀을 추가하고 판매 주문 데이터의 데이터 프레임을 salesorders라는 **테이블로 저장하는 다음 코드를 입력합니다**.

    ```Python
   # Create a new table
   df.write.format("delta").saveAsTable("salesorders")

   # Get the table description
   spark.sql("DESCRIBE EXTENDED salesorders").show(truncate=False)
    ```

    > **참고**: 이 예제에 대한 몇 가지 사항을 주목할 가치가 있습니다. 첫째, 명시적 경로가 제공되지 않으므로 테이블에 대한 파일은 메타스토어에서 관리됩니다. 둘째, 테이블은 델타** 형식으로 **저장됩니다. 여러 파일 형식(CSV, Parquet, Avro 등 포함)을 기반으로 테이블을 만들 수 있지만 *Delta Lake* 는 트랜잭션, 행 버전 관리 및 기타 유용한 기능을 포함하여 테이블에 관계형 데이터베이스 기능을 추가하는 Spark 기술입니다. 패브릭의 데이터 레이크하우스에는 델타 형식으로 테이블을 만드는 것이 좋습니다.

2. 코드 셀을 실행하고 새 테이블의 정의를 설명하는 출력을 검토합니다.

3. Lakehouses 창의 ****테이블** 폴더에 대한 **...** 메뉴에서 새로 고침**을 선택합니다**.** 그런 다음 테이블 노드를 **확장하고 salesorders** 테이블이 **만들어졌는지 확인**합니다.

    ![탐색기의 salesorder 테이블 스크린샷](./Images/table-view.png)

5. **salesorders** 테이블에 대한 **...** 메뉴에서 데이터 > ****Spark** 로드를 선택합니다**.

    다음 예제와 유사한 코드를 포함하는 새 코드 셀이 Notebook에 추가됩니다.

    ```Python
   df = spark.sql("SELECT * FROM [your_lakehouse].salesorders LIMIT 1000")
   display(df)
    ```

6. Spark SQL 라이브러리를 사용하여 PySpark 코드의 salesorder** 테이블에 대해 **SQL 쿼리를 포함하고 쿼리 결과를 데이터 프레임에 로드하는 새 코드를 실행합니다.

### 셀에서 SQL 코드를 실행합니다.

PySpark 코드가 포함된 셀에 SQL 문을 포함시키는 것이 유용하겠지만 데이터 분석가는 SQL에서 직접 작업하려는 경우가 많습니다.

1. Notebook에 새 코드 셀을 추가하고 Notebook에 다음 코드를 입력합니다.

    ```sql
   %%sql
   SELECT YEAR(OrderDate) AS OrderYear,
          SUM((UnitPrice * Quantity) + Tax) AS GrossRevenue
   FROM salesorders
   GROUP BY YEAR(OrderDate)
   ORDER BY OrderYear;
    ```

2. 셀을 실행하고 결과를 검토합니다. 다음과 같은 부분을 확인하세요.
    - 셀의 시작 부분에 있는 `%%sql` 줄(*magic*이라고 함)은 Spark SQL 언어 런타임을 사용하여 PySpark 대신 이 셀에서 코드를 실행해야 함을 나타냅니다.
    - SQL 코드는 이전에 만든 salesorders** 테이블을 참조**합니다.
    - SQL 쿼리의 출력은 셀 아래에 결과로 자동으로 표시됩니다.

> **참고**: Spark SQL 및 데이터 프레임에 대한 자세한 내용은 [Spark SQL 설명서](https://spark.apache.org/docs/2.2.0/sql-programming-guide.html)를 참조하세요.

## Spark를 사용하여 데이터 시각화

그림은 속담처럼 천 단어의 가치가 있으며 차트는 종종 천 행의 데이터보다 낫습니다. Fabric의 Notebook에는 데이터 프레임 또는 Spark SQL 쿼리에서 표시되는 데이터에 대한 기본 제공 차트 뷰가 포함되어 있지만 포괄적인 차트를 위해 설계되지 않았습니다. 그러나 **matplotlib** 및 **seaborn**과 같은 Python 그래픽 라이브러리를 사용하여 데이터 프레임의 데이터에서 차트를 만들 수 있습니다.

### 결과를 차트로 보기

1. Notebook에 새 코드 셀을 추가하고 Notebook에 다음 코드를 입력합니다.

    ```sql
   %%sql
   SELECT * FROM salesorders
    ```

2. 코드를 실행하고 이전에 만든 **salesorders** 뷰에서 데이터를 반환하는지 살펴봅니다.
3. 셀 아래의 결과 섹션에서 **보기** 옵션을 **테이블**에서 **차트**로 변경합니다.
4. 차트의 **오른쪽 위에 있는 차트** 사용자 지정 단추를 사용하여 차트의 옵션 창을 표시합니다. 그런 다음, 옵션을 다음과 같이 설정하고 **적용**을 선택합니다.
    - **차트 종류**: 가로 막대형 차트
    - **키**: Item
    - **값**: Quantity
    - **계열 그룹**: *비워 둠*
    - **집계**: Sum
    - **누적**: *선택되지 않음*

5. 차트가 다음과 유사한지 확인합니다.

    ![총 주문 수량별 제품의 가로 막대형 차트 스크린샷](./Images/notebook-chart.png)

### **matplotlib** 시작

1. Notebook에 새 코드 셀을 추가하고 Notebook에 다음 코드를 입력합니다.

    ```Python
   sqlQuery = "SELECT CAST(YEAR(OrderDate) AS CHAR(4)) AS OrderYear, \
                   SUM((UnitPrice * Quantity) + Tax) AS GrossRevenue \
               FROM salesorders \
               GROUP BY CAST(YEAR(OrderDate) AS CHAR(4)) \
               ORDER BY OrderYear"
   df_spark = spark.sql(sqlQuery)
   df_spark.show()
    ```

2. 코드를 실행하고 연간 수익을 포함하는 Spark 데이터 프레임을 반환하는지 살펴봅니다.

    데이터를 차트로 시각화하려면 먼저 **matplotlib** Python 라이브러리를 사용하여 시작합니다. 이 라이브러리는 다른 많은 라이브러리의 기반이 되는 핵심 그리기 라이브러리이며 차트를 만드는 데 많은 유연성을 제공합니다.

3. Notebook에 새 코드 셀을 추가하고 Notebook에 다음 코드를 추가합니다.

    ```Python
   from matplotlib import pyplot as plt

   # matplotlib requires a Pandas dataframe, not a Spark one
   df_sales = df_spark.toPandas()

   # Create a bar plot of revenue by year
   plt.bar(x=df_sales['OrderYear'], height=df_sales['GrossRevenue'])

   # Display the plot
   plt.show()
    ```

4. 셀을 실행하고 결과를 검토합니다. 이는 매년 총 수익과 함께 세로 막대형 차트로 구성됩니다. 이 차트를 생성하는 데 사용되는 코드의 다음 기능을 확인합니다.
    - **matplotlib** 라이브러리에는 *Pandas* 데이터 프레임이 필요하므로 Spark SQL 쿼리에서 반환되는 *Spark* 데이터 프레임을 이 형식으로 변환해야 합니다.
    - **matplotlib** 라이브러리의 핵심은 **pyplot** 개체입니다. 이것은 대부분의 그리기 기능의 기초입니다.
    - 기본 설정을 사용하면 사용자 지정할 수 있는 상당한 범위가 있는 사용 가능한 차트가 생성됩니다.

5. 다음과 같이 차트를 그리도록 코드를 수정합니다.

    ```Python
   from matplotlib import pyplot as plt

   # Clear the plot area
   plt.clf()

   # Create a bar plot of revenue by year
   plt.bar(x=df_sales['OrderYear'], height=df_sales['GrossRevenue'], color='orange')

   # Customize the chart
   plt.title('Revenue by Year')
   plt.xlabel('Year')
   plt.ylabel('Revenue')
   plt.grid(color='#95a5a6', linestyle='--', linewidth=2, axis='y', alpha=0.7)
   plt.xticks(rotation=45)

   # Show the figure
   plt.show()
    ```

6. 코드 셀을 다시 실행하고 결과를 확인합니다. 이제 차트에 좀 더 많은 정보가 포함되어 있습니다.

    플롯은 기술적으로 **그림**과 함께 포함됩니다. 이전 예제에서는 암시적으로 그림이 작성되었지만 명시적으로 그림을 작성할 수도 있습니다.

7. 다음과 같이 차트를 그리도록 코드를 수정합니다.

    ```Python
   from matplotlib import pyplot as plt

   # Clear the plot area
   plt.clf()

   # Create a Figure
   fig = plt.figure(figsize=(8,3))

   # Create a bar plot of revenue by year
   plt.bar(x=df_sales['OrderYear'], height=df_sales['GrossRevenue'], color='orange')

   # Customize the chart
   plt.title('Revenue by Year')
   plt.xlabel('Year')
   plt.ylabel('Revenue')
   plt.grid(color='#95a5a6', linestyle='--', linewidth=2, axis='y', alpha=0.7)
   plt.xticks(rotation=45)

   # Show the figure
   plt.show()
    ```

8. 코드 셀을 다시 실행하고 결과를 확인합니다. 그림에 따라 플롯의 모양과 크기가 결정됩니다.

    그림에는 각각 고유의 축에 여러 개의 하위 플롯이 포함될 수 있습니다.**

9. 다음과 같이 차트를 그리도록 코드를 수정합니다.

    ```Python
   from matplotlib import pyplot as plt

   # Clear the plot area
   plt.clf()

   # Create a figure for 2 subplots (1 row, 2 columns)
   fig, ax = plt.subplots(1, 2, figsize = (10,4))

   # Create a bar plot of revenue by year on the first axis
   ax[0].bar(x=df_sales['OrderYear'], height=df_sales['GrossRevenue'], color='orange')
   ax[0].set_title('Revenue by Year')

   # Create a pie chart of yearly order counts on the second axis
   yearly_counts = df_sales['OrderYear'].value_counts()
   ax[1].pie(yearly_counts)
   ax[1].set_title('Orders per Year')
   ax[1].legend(yearly_counts.keys().tolist())

   # Add a title to the Figure
   fig.suptitle('Sales Data')

   # Show the figure
   plt.show()
    ```

10. 코드 셀을 다시 실행하고 결과를 확인합니다. 그림에는 코드에 지정된 하위 플롯이 포함되어 있습니다.

> **참고**: matplotlib를 사용하여 그리는 방법에 대한 자세한 내용은 [matplotlib 설명서](https://matplotlib.org/)를 참조하세요.

### **seaborn** 라이브러리 사용

**matplotlib**를 사용하면 여러 종류의 복잡한 차트를 만들 수 있지만 최상의 결과를 얻으려면 몇 가지 복잡한 코드가 필요할 수 있습니다. 이러한 이유로 수년 동안 복잡성을 추상화하고 기능을 향상시키기 위해 matplotlib의 기반으로 많은 새로운 라이브러리가 구축되었습니다. 그러한 라이브러리 중 하나가 **seaborn**입니다.

1. Notebook에 새 코드 셀을 추가하고 Notebook에 다음 코드를 입력합니다.

    ```Python
   import seaborn as sns

   # Clear the plot area
   plt.clf()

   # Create a bar chart
   ax = sns.barplot(x="OrderYear", y="GrossRevenue", data=df_sales)
   plt.show()
    ```

2. 코드를 실행하고 seaborn 라이브러리를 사용하여 가로 막대형 차트를 표시하는지 살펴봅니다.
3. 다음과 같이 코드를 수정합니다.

    ```Python
   import seaborn as sns

   # Clear the plot area
   plt.clf()

   # Set the visual theme for seaborn
   sns.set_theme(style="whitegrid")

   # Create a bar chart
   ax = sns.barplot(x="OrderYear", y="GrossRevenue", data=df_sales)
   plt.show()
    ```

4. 수정된 코드를 실행하고 seaborn을 사용하면 플롯에 일관된 색 테마를 설정할 수 있습니다.

5. 다음과 같이 코드를 다시 수정합니다.

    ```Python
   import seaborn as sns

   # Clear the plot area
   plt.clf()

   # Create a line chart
   ax = sns.lineplot(x="OrderYear", y="GrossRevenue", data=df_sales)
   plt.show()
    ```

6. 수정된 코드를 실행하여 연간 수익을 꺾은선형 차트로 봅니다.

> **참고**: seaborn을 사용한 그리기에 대한 자세한 내용은 [ 설명서](https://seaborn.pydata.org/index.html)를 참조하세요.

## Notebook 저장 및 Spark 세션 종료

이제 데이터 작업을 마쳤으므로 의미 있는 이름으로 Notebook을 저장하고 Spark 세션을 종료할 수 있습니다.

1. 전자 필기장 메뉴 모음에서 ⚙️ **설정** 아이콘을 사용하여 전자 필기장 설정을 봅니다.
2. **전자 필기장의 이름을** 설정하여 **판매 주문을** 탐색한 다음 설정 창을 닫습니다.
3. Notebook 메뉴에서 **세션 중지**를 선택하여 Spark 세션을 종료합니다.

## 리소스 정리

이 연습에서는 Spark를 사용하여 Microsoft Fabric에서 데이터를 사용하는 방법을 알아보았습니다.

레이크하우스 탐색을 마쳤으면 이 연습을 위해 만든 작업 영역을 삭제할 수 있습니다.

1. 왼쪽 막대에서 작업 영역의 아이콘을 선택하여 포함된 모든 항목을 봅니다.
2. 도구 모음의 **...** 메뉴에서 **작업 영역 설정**을 선택합니다.
3. **기타** 섹션에서 **이 작업 영역 제거**를 선택합니다.
