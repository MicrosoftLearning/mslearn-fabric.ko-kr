---
lab:
  title: Apache Spark에서 델타 테이블 사용
  module: Work with Delta Lake tables in Microsoft Fabric
---

# Apache Spark에서 델타 테이블 사용

Microsoft Fabric 레이크하우스의 테이블은 Apache Spark용 오픈 소스 *Delta Lake* 형식을 기반으로 합니다. Delta Lake는 일괄 처리 및 스트리밍 데이터 작업 모두에 대한 관계형 의미 체계에 대한 지원을 추가하고 Apache Spark를 사용하여 데이터 레이크의 기본 파일을 기반으로 하는 테이블에서 데이터를 처리하고 쿼리할 수 있는 레이크하우스 아키텍처를 만들 수 있습니다.

이 연습을 완료하는 데 약 **40**분 정도 소요됩니다.

> **참고**: 이 연습을 완료하려면 [Microsoft Fabric 평가판](https://learn.microsoft.com/fabric/get-started/fabric-trial)이 필요합니다.

## 작업 영역 만들기

패브릭에서 데이터를 사용하기 전에 패브릭 평가판을 사용하도록 설정된 작업 영역을 만듭니다.

1. [Microsoft Fabric 홈페이지](https://app.fabric.microsoft.com/home?experience=fabric)(`https://app.fabric.microsoft.com/home?experience=fabric`)에서 **Synapse 데이터 엔지니어링**을 선택합니다.
2. 왼쪽 메뉴 모음에서 **작업 영역**을 선택합니다(아이콘은 와 유사함).
3. Fabric 용량이 포함된 라이선스 모드(*평가판*, *프리미엄* 또는 *Fabric*)를 선택하여 원하는 이름으로 새 작업 영역을 만듭니다.
4. 새 작업 영역이 열리면 비어 있어야 합니다.

    ![Fabric의 빈 작업 영역 스크린샷.](./Images/new-workspace.png)

## 레이크하우스 만들기 및 데이터 업로드

이제 작업 영역이 있으므로 분석하려는 데이터에 대한 데이터 레이크하우스를 만들어야 합니다.

1. **Synapse Data Engineering** 홈페이지에서 원하는 이름으로 새 **레이크하우스**를 만듭니다.

    잠시 후 새로운 빈 레이크하우스가 나타납니다. 분석을 위해 일부 데이터를 데이터 레이크하우스에 수집해야 합니다. 이 작업을 수행하는 방법에는 여러 가지가 있지만 이 연습에서는 텍스트 파일을 로컬 컴퓨터(또는 해당하는 경우 랩 VM)에 다운로드한 다음 레이크하우스에 업로드하기만 하면 됩니다.

1. 이 연습을 위한 [데이터 파일](https://github.com/MicrosoftLearning/dp-data/raw/main/products.csv)을 `https://github.com/MicrosoftLearning/dp-data/raw/main/products.csv`에서 다운로드하여 로컬 컴퓨터(또는 해당하는 경우 랩 VM)에 **products.csv**로 저장합니다.

1. 레이크하우스가 포함된 웹 브라우저 탭으로 돌아가 **탐색기** 창의 **Files** 폴더에 대한 **...** 메뉴에서 **새 하위 폴더**를 선택하고 **products**라는 폴더를 만듭니다.

1. **제품** 폴더의 **...** 메뉴에서 **업로드** 및 **파일 업로드**를 선택한 다음 로컬 컴퓨터(또는 해당하는 경우 랩 VM)의 **products.csv** 파일을 레이크하우스에 업로드합니다.
1. 파일을 업로드한 후 **제품** 폴더를 선택합니다. 다음과 같이 **products.csv** 파일이 업로드되었는지 확인합니다.

    ![레이크하우스에 업로드된 products.csv 파일의 스크린샷.](./Images/products-file.png)

## 데이터프레임에서 데이터 탐색

1. 데이터레이크에 있는 **제품** 폴더의 콘텐츠를 보는 동안 **홈** 페이지의 **Notebook 열기** 메뉴에서 **새 Notebook**을 선택합니다.

    몇 초 후에 단일 *셀*이 포함된 새 Notebook이 열립니다. Notebook은 *코드* 또는 *markdown*(서식이 지정된 텍스트)을 포함할 수 있는 하나 이상의 셀로 구성됩니다.

2. 몇 가지 간단한 코드가 포함된 Notebook의 기존 셀을 선택한 다음 오른쪽 상단에 있는 **&#128465;**(*삭제*) 아이콘을 사용하여 제거합니다. 이 코드는 필요하지 않습니다.
3. **탐색기** 창에서 **레이크하우스**를 확장한 다음 레이크하우스의 **파일** 목록을 확장하고 이전에 업로드한 **products.csv** 파일을 보여주는 새 창을 표시하도록 **products** 폴더를 선택합니다.

    ![파일 창이 있는 Notebook의 스크린샷.](./Images/notebook-products.png)

4. **products.csv**의 **...** 메뉴에서 **데이터 로드** > **Spark**를 선택합니다. 다음 코드를 포함하는 새 코드 셀을 Notebook에 추가해야 합니다.

    ```python
   df = spark.read.format("csv").option("header","true").load("Files/products/products.csv")
   # df now is a Spark DataFrame containing CSV data from "Files/products/products.csv".
   display(df)
    ```

    > **팁**: **<<** 아이콘을 사용하여 왼쪽에 있는 파일이 포함된 창을 숨길 수 있습니다. 그렇게 하면 Notebook에 집중하는 데 도움이 됩니다.

5. 셀 왼쪽의 **&#9655;**(*셀 실행l*) 단추를 이용하여 실행합니다.

    > **참고**: 이 Notebook에서 Spark 코드를 처음 실행했으므로 Spark 세션을 시작해야 합니다. 이는 첫 번째 실행이 완료되는 데 1분 정도 걸릴 수 있음을 의미합니다. 후속 실행은 더 빨라질 것입니다.

6. 셀 명령이 완료되면 셀 아래의 출력을 검토합니다. 다음과 유사한 출력을 확인할 수 있습니다.

    | 색인 | ProductID | ProductName | 범주 | ListPrice |
    | -- | -- | -- | -- | -- |
    | 1 | 771 | Mountain-100 Silver, 38 | 산악용 자전거 | 3399.9900 |
    | 2 | 772 | Mountain-100 Silver, 42 | 산악용 자전거 | 3399.9900 |
    | 3 | 773 | Mountain-100 Silver, 44 | 산악용 자전거 | 3399.9900 |
    | ... | ... | ... | ... | ... |

## 델타 테이블 만들기

`saveAsTable` 메서드를 사용하여 데이터프레임을 델타 테이블로 저장할 수 있습니다. Delta Lake는 *관리* 테이블과 *외부* 테이블 만들기를 모두 지원합니다.

### *관리되는* 테이블 만들기

*관리* 테이블은 스키마 메타데이터와 데이터 파일이 모두 Fabric에서 관리되는 테이블입니다. 테이블의 데이터 파일은 **Tables** 폴더에 만들어집니다.

1. 첫 번째 코드 셀에서 반환된 결과 아래에 새 코드 셀이 없는 경우 **+ 코드** 아이콘을 사용하여 새 코드 셀을 추가합니다.

    > **팁**: **+ 코드** 아이콘을 보려면 마우스를 현재 셀의 출력 바로 아래 왼쪽으로 이동합니다. 아니면 메뉴 모음의 **편집** 탭에서 **+ 코드 셀 추가**를 선택합니다.

2. 새로운 셀에 다음 코드를 입력하고 실행합니다.

    ```python
   df.write.format("delta").saveAsTable("managed_products")
    ```

3. **레이크하우스 탐색기** 창의 **테이블** 폴더에 대한 **...** 메뉴에서 **새로 고침**을 선택합니다. 그런 다음 **Tables** 노드를 확장하고 **managed_products** 테이블이 만들어졌는지 확인합니다.

### *외부 테이블* 만들기

레이크하우스의 메타스토어에 스키마 메타데이터가 정의되어 있는 *외부* 테이블을 만들 수도 있지만 데이터 파일은 외부 위치에 저장됩니다.

1. 또 다른 새 코드 셀을 추가하고 다음 코드를 추가합니다.

    ```python
   df.write.format("delta").saveAsTable("external_products", path="abfs_path/external_products")
    ```

2. **레이크하우스 탐색기** 창의 **Files** 폴더에 대한 **...** 메뉴에서 **ABFS 경로 복사**를 선택합니다.

    ABFS 경로는 레이크하우스의 OneLake 스토리지에 있는 **Files** 폴더에 대한 정규화된 경로입니다. 다음과 유사합니다.

    *abfss://workspace@tenant-onelake.dfs.fabric.microsoft.com/lakehousename.Lakehouse/Files*

3. 코드 셀에 입력한 코드에서 **abfs_path**를 클립보드에 복사한 경로로 바꾸면 코드가 데이터 프레임을 **Files** 폴더 위치의 **external_products** 폴더에 데이터 파일이 포함된 외부 테이블로 저장합니다. 전체 경로는 다음과 유사해야 합니다.

    *abfss://workspace@tenant-onelake.dfs.fabric.microsoft.com/lakehousename.Lakehouse/Files/external_products*

4. **레이크하우스 탐색기** 창의 **테이블** 폴더에 대한 **...** 메뉴에서 **새로 고침**을 선택합니다. 그런 다음 **Tables** 노드를 확장하고 **external_products** 테이블이 만들어졌는지 확인합니다.

5. **레이크하우스 탐색기** 창의 **Files** 폴더에 대한 **...** 메뉴에서 **새로 고침**을 선택합니다. 그런 다음 **Files** 노드를 확장하고 테이블의 데이터 파일에 대해 **external_products** 폴더가 만들어졌는지 확인합니다.

### *관리* 테이블과 *외부* 테이블 비교

관리되는 테이블과 외부 테이블의 차이점을 살펴보겠습니다.

1. 다른 코드 셀을 추가하고 다음 코드를 실행합니다.

    ```sql
   %%sql

   DESCRIBE FORMATTED managed_products;
    ```

    이 결과에서 테이블의 **Location** 속성을 확인합니다. 이 속성은 **/Tables/managed_products**로 끝나는 레이크하우스의 OneLake 스토리지 경로여야 합니다(전체 경로를 보려면 **데이터 형식** 열을 넓혀야 할 수도 있음).

2. 다음과 같이 **external_products** 테이블의 세부 정보를 표시하도록 `DESCRIBE` 명령을 수정합니다.

    ```sql
   %%sql

   DESCRIBE FORMATTED external_products;
    ```

    이 결과에서 테이블의 **Location** 속성을 확인합니다. 이 속성은 **/Files/external_products**로 끝나는 레이크하우스의 OneLake 스토리지 경로여야 합니다(전체 경로를 보려면 **데이터 형식** 열을 넓혀야 할 수도 있음).

    관리형 테이블의 파일은 레이크하우스의 OneLake 스토리지에 있는 **Tables** 폴더에 저장됩니다. 이 경우 만들어진 테이블에 대한 Parquet 파일과 **delta_log** 폴더를 저장하기 위해 **managed_products**라는 폴더가 만들어졌습니다.

3. 다른 코드 셀을 추가하고 다음 코드를 실행합니다.

    ```sql
   %%sql

   DROP TABLE managed_products;
   DROP TABLE external_products;
    ```

4. **레이크하우스 탐색기** 창의 **테이블** 폴더에 대한 **...** 메뉴에서 **새로 고침**을 선택합니다. 그런 다음 **Tables** 노드를 확장하고 나열된 테이블이 없는지 확인합니다.

5. **레이크하우스 탐색기** 창에서 **Files** 폴더를 확장하고 **external_products**가 삭제되지 않았는지 확인합니다. 이전에 **external_products** 테이블에 있었던 데이터에 대한 Parquet 데이터 파일과 **_delta_log** 폴더를 보려면 이 폴더를 선택합니다. 외부 테이블의 테이블 메타데이터가 삭제되었지만 파일은 영향을 받지 않았습니다.

### SQL을 사용하여 테이블 만들기

1. 다른 코드 셀을 추가하고 다음 코드를 실행합니다.

    ```sql
   %%sql

   CREATE TABLE products
   USING DELTA
   LOCATION 'Files/external_products';
    ```

2. **레이크하우스 탐색기** 창의 **테이블** 폴더에 대한 **...** 메뉴에서 **새로 고침**을 선택합니다. 그런 다음 **Tables** 노드를 확장하고 **products**라는 새 테이블이 나열되는지 확인합니다. 그런 다음 테이블을 확장하여 해당 스키마가 **external_products** 폴더에 저장된 원래 데이터 프레임과 일치하는지 확인합니다.

3. 다른 코드 셀을 추가하고 다음 코드를 실행합니다.

    ```sql
   %%sql

   SELECT * FROM products;
   ```

## 테이블 버전 관리 살펴보기

델타 테이블의 트랜잭션 기록은 **delta_log** 폴더의 JSON 파일에 저장됩니다. 이 트랜잭션 로그를 사용하여 데이터 버전 관리를 관리할 수 있습니다.

1. Notebook에 새 코드 셀을 추가하고 다음 코드를 실행합니다.

    ```sql
   %%sql

   UPDATE products
   SET ListPrice = ListPrice * 0.9
   WHERE Category = 'Mountain Bikes';
    ```

    이 코드는 산악 자전거 가격의 10% 인하를 구현합니다.

2. 다른 코드 셀을 추가하고 다음 코드를 실행합니다.

    ```sql
   %%sql

   DESCRIBE HISTORY products;
    ```

    결과에는 테이블에 기록된 트랜잭션 내역이 표시됩니다.

3. 다른 코드 셀을 추가하고 다음 코드를 실행합니다.

    ```python
   delta_table_path = 'Files/external_products'

   # Get the current data
   current_data = spark.read.format("delta").load(delta_table_path)
   display(current_data)

   # Get the version 0 data
   original_data = spark.read.format("delta").option("versionAsOf", 0).load(delta_table_path)
   display(original_data)
    ```

    결과에는 두 개의 데이터 프레임이 표시됩니다. 하나는 가격 인하 이후의 데이터를 포함하고 다른 하나는 데이터의 원본 버전을 표시합니다.

## 스트리밍 데이터에 델타 테이블 사용

Delta Lake는 스트리밍 데이터를 지원합니다. 델타 테이블은 Spark 구조적 스트리밍 API를 사용하여 생성된 데이터 스트림의 싱크 또는 원본일 수 있습니다. 이 예제에서는 시뮬레이션된 IoT(사물 인터넷) 시나리오에서 델타 테이블을 일부 스트리밍 데이터의 싱크로 사용합니다.

1. Notebook에 새 코드 셀을 추가합니다. 그런 다음 새 셀에서 다음 코드를 추가하고 실행합니다.

    ```python
   from notebookutils import mssparkutils
   from pyspark.sql.types import *
   from pyspark.sql.functions import *

   # Create a folder
   inputPath = 'Files/data/'
   mssparkutils.fs.mkdirs(inputPath)

   # Create a stream that reads data from the folder, using a JSON schema
   jsonSchema = StructType([
   StructField("device", StringType(), False),
   StructField("status", StringType(), False)
   ])
   iotstream = spark.readStream.schema(jsonSchema).option("maxFilesPerTrigger", 1).json(inputPath)

   # Write some event data to the folder
   device_data = '''{"device":"Dev1","status":"ok"}
   {"device":"Dev1","status":"ok"}
   {"device":"Dev1","status":"ok"}
   {"device":"Dev2","status":"error"}
   {"device":"Dev1","status":"ok"}
   {"device":"Dev1","status":"error"}
   {"device":"Dev2","status":"ok"}
   {"device":"Dev2","status":"error"}
   {"device":"Dev1","status":"ok"}'''
   mssparkutils.fs.put(inputPath + "data.txt", device_data, True)
   print("Source stream created...")
    ```

    만든 원본 스트림... 메시지가 출력되는지 확인합니다. 방금 실행한 코드는 가상의 IoT 디바이스의 판독값을 나타내는 일부 데이터가 저장된 폴더를 기반으로 스트리밍 데이터 원본을 만들었습니다.

2. 새 코드 셀에서 다음 코드를 추가하고 실행합니다.

    ```python
   # Write the stream to a delta table
   delta_stream_table_path = 'Tables/iotdevicedata'
   checkpointpath = 'Files/delta/checkpoint'
   deltastream = iotstream.writeStream.format("delta").option("checkpointLocation", checkpointpath).start(delta_stream_table_path)
   print("Streaming to delta sink...")
    ```

    이 코드는 스트리밍 디바이스 데이터를 델타 형식으로 **iotdevicedata**라는 폴더에 작성합니다. 폴더 위치에 대한 경로가 **Tables** 폴더에 있으므로 해당 폴더에 대한 테이블이 자동으로 만들어집니다.

3. 새 코드 셀에서 다음 코드를 추가하고 실행합니다.

    ```sql
   %%sql

   SELECT * FROM IotDeviceData;
    ```

    이 코드는 스트리밍 원본의 디바이스 데이터를 포함하는 **IotDeviceData** 테이블을 쿼리합니다.

4. 새 코드 셀에서 다음 코드를 추가하고 실행합니다.

    ```python
   # Add more data to the source stream
   more_data = '''{"device":"Dev1","status":"ok"}
   {"device":"Dev1","status":"ok"}
   {"device":"Dev1","status":"ok"}
   {"device":"Dev1","status":"ok"}
   {"device":"Dev1","status":"error"}
   {"device":"Dev2","status":"error"}
   {"device":"Dev1","status":"ok"}'''

   mssparkutils.fs.put(inputPath + "more-data.txt", more_data, True)
    ```

    이 코드는 스트리밍 원본에 더 많은 가상 디바이스 데이터를 작성합니다.

5. 다음 코드가 포함된 셀을 다시 실행합니다.

    ```sql
   %%sql

   SELECT * FROM IotDeviceData;
    ```

    이 코드는 이제 스트리밍 원본에 추가된 추가 데이터를 포함해야 하는 **IotDeviceData** 테이블을 다시 쿼리합니다.

6. 새 코드 셀에서 다음 코드를 추가하고 실행합니다.

    ```python
   deltastream.stop()
    ```

    이 코드는 스트림을 중지합니다.

## 리소스 정리

이 연습에서는 Microsoft Fabric에서 델타 테이블을 사용하여 작업하는 방법을 알아보았습니다.

레이크하우스 탐색을 마쳤으면 이 연습을 위해 만든 작업 영역을 삭제할 수 있습니다.

1. 왼쪽 막대에서 작업 영역의 아이콘을 선택하여 포함된 모든 항목을 봅니다.
2. 도구 모음의 **...** 메뉴에서 **작업 영역 설정**을 선택합니다.
3. **일반** 섹션에서 **이 작업 영역 제거**를 선택합니다.
