---
lab:
  title: Spark 및 Microsoft Fabric Notebook을 사용하여 데이터 수집
  module: Ingest data with Spark and Microsoft Fabric notebooks
---

# Spark 및 Microsoft Fabric Notebook을 사용하여 데이터 수집

이 랩에서는 Microsoft Fabric Notebook을 만들고 PySpark를 사용하여 Azure Blob Storage 경로에 연결한 다음 쓰기 최적화를 사용하여 레이크하우스에 데이터를 로드합니다.

이 랩을 완료하는 데 약 **30**분이 걸립니다.

이 환경에서는 여러 Notebook 코드 셀에 코드를 빌드합니다. 이는 사용자 환경에서 수행할 방법을 반영하지 못할 수 있지만 디버깅에는 유용할 수 있습니다.

또한 샘플 데이터 세트로 작업하므로 최적화에는 대규모 프로덕션에서 볼 수 있는 내용이 반영되지 않습니다. 하지만 여전히 개선점을 볼 수 있으며 밀리초 단위로 중요한 경우 최적화가 핵심입니다.

> **참고**: 이 연습을 완료하려면 [Microsoft Fabric 평가판](https://learn.microsoft.com/fabric/get-started/fabric-trial)이 필요합니다.

## 작업 영역 만들기

패브릭에서 데이터를 사용하기 전에 패브릭 평가판을 사용하도록 설정된 작업 영역을 만듭니다.

1. [Microsoft Fabric 홈페이지](https://app.fabric.microsoft.com)에서 **Synapse 데이터 엔지니어링**을 선택합니다.
1. 왼쪽 메뉴 모음에서 **작업 영역**을 선택합니다(아이콘은 와 유사함).
1. Fabric 용량이 포함된 라이선스 모드(*평가판*, *프리미엄* 또는 *Fabric*)를 선택하여 원하는 이름으로 새 작업 영역을 만듭니다.
1. 새 작업 영역이 열리면 비어 있어야 합니다.

    ![Fabric의 빈 작업 영역 스크린샷.](./Images/new-workspace.png)

## 작업 영역 및 레이크하우스 대상 만들기

먼저 레이크하우스에 새 레이크하우스와 대상 폴더를 만듭니다.

1. 작업 영역에서 **+ 새 > 레이크하우스**를 선택하고 이름을 제공하고 **만들기**를 선택합니다.

    > **참고:** **테이블** 또는 **파일**이 없는 새 레이크하우스를 만드는 데 몇 분 정도 걸릴 수 있습니다.

    ![새 레이크하우스의 스크린샷](Images/new-lakehouse.png)

1. **파일**에서 **[...]** 을 선택하여 **RawData**라고 명명된 **새 하위 폴더**를 만듭니다.

1. 레이크하우스 내의 레이크하우스 탐색기에서 **RawData > ... > 속성**을 선택합니다.

1. 나중에 사용할 수 있도록 **RawData** 폴더의 **ABFS 경로**를 빈 메모장에 복사합니다. `abfss://{workspace_name}@onelake.dfs.fabric.microsoft.com/{lakehouse_name}.Lakehouse/Files/{folder_name}/{file_name}`과(와) 유사하게 표시됩니다.

이제 레이크하우스와 RawData 대상 폴더가 있는 작업 영역이 생깁니다.

## Fabric Notebook 만들기 및 외부 데이터 로드

새 Fabric Notebook을 만들고 PySpark를 사용하여 외부 데이터 원본에 연결합니다.

1. 레이크하우스의 위쪽 메뉴에서 전자 필기장 **열기 > 새 Notebook**을 선택합니다. Notebook이 만들어지면 열립니다.

    >  **팁:** 이 Notebook 내에서 레이크하우스 탐색기에 액세스할 수 있으며, 이 연습을 완료하면 새로 고쳐 진행 상황을 확인할 수 있습니다.

1. 기본 셀에서 코드가 **PySpark(Python)** 로 설정됩니다.

1. 코드 셀에 다음 코드를 삽입합니다.
    - 연결 문자열에 대한 매개 변수 선언
    - 연결 문자열 빌드
    - DataFrame으로 데이터 가져오기

    ```Python
    # Azure Blob Storage access info
    blob_account_name = "azureopendatastorage"
    blob_container_name = "nyctlc"
    blob_relative_path = "yellow"
    
    # Construct connection path
    wasbs_path = f'wasbs://{blob_container_name}@{blob_account_name}.blob.core.windows.net/{blob_relative_path}'
    print(wasbs_path)
    
    # Read parquet data from Azure Blob Storage path
    blob_df = spark.read.parquet(wasbs_path)
    ```

1. 코드 셀 옆에 있는 **&#9655; 셀 실행**을 선택하여 데이터를 DataFrame에 연결하고 읽습니다.

    **예상 결과:** 명령이 성공하고 `wasbs://nyctlc@azureopendatastorage.blob.core.windows.net/yellow`을(를) 인쇄합니다.

    > **참고:** Spark 세션은 첫 번째 코드 실행에서 시작되므로 완료하는 데 시간이 더 오래 걸릴 수 있습니다.

1. 파일에 데이터를 쓰려면 이제 **RawData** 폴더에 대한 **ABFS 경로**가 필요합니다.

1. **새 코드 셀**에 다음 코드를 삽입합니다.

    ```python
        # Declare file name    
        file_name = "yellow_taxi"
    
        # Construct destination path
        output_parquet_path = f"**InsertABFSPathHere**/{file_name}"
        print(output_parquet_path)
        
        # Load the first 1000 rows as a Parquet file
        blob_df.limit(1000).write.mode("overwrite").parquet(output_parquet_path)
    ```

1. **RawData** ABFS 경로를 추가하고 **&#9655; 셀 실행**을 선택하여 yellow_taxi.parquet 파일에 1000개의 행을 씁니다.

1. **output_parquet_path**가 `abfss://Spark@onelake.dfs.fabric.microsoft.com/DPDemo.Lakehouse/Files/RawData/yellow_taxi`과(와) 유사하게 표시됩니다.

1. Lakehouse Explorer에서 데이터 로드를 확인하려면 **파일 > ... > 새로 고침**을 선택합니다.

이제 **yellow_taxi.parquet** "file"(*파티션 파일이 들어 있는 폴더로 표시됨*)이 있는 새 **RawData** 폴더가 표시됩니다.

## 델타 테이블로 데이터 변환 및 로드

아마도 데이터 수집 작업이 파일을 로드하는 것만으로 끝나지 않을 것입니다. 레이크하우스의 델타 테이블은 확장성 있는 유연한 쿼리와 스토리지를 허용하므로 델타 테이블을 하나 만들겠습니다.

1. 새 코드 셀을 만들고 다음 코드를 삽입합니다.

    ```python
    from pyspark.sql.functions import col, to_timestamp, current_timestamp, year, month
    
    # Read the parquet data from the specified path
    raw_df = spark.read.parquet(output_parquet_path)   
    
    # Add dataload_datetime column with current timestamp
    filtered_df = raw_df.withColumn("dataload_datetime", current_timestamp())
    
    # Filter columns to exclude any NULL values in storeAndFwdFlag
    filtered_df = filtered_df.filter(raw_df["storeAndFwdFlag"].isNotNull())
    
    # Load the filtered data into a Delta table
    table_name = "yellow_taxi"  # Replace with your desired table name
    filtered_df.write.format("delta").mode("append").saveAsTable(table_name)
    
    # Display results
    display(filtered_df.limit(1))
    ```

1. 코드 셀 옆에 있는 **&#9655; 셀 실행**을 선택합니다.

    - 그러면 델타 테이블에 데이터가 로드된 때를 기록하는 타임스탬프 열 **dataload_datetime**이 추가됩니다.
    - **storeAndFwdFlag**에서 NULL 값 필터링
    - 델타 테이블에 필터링된 데이터 로드
    - 유효성 검사를 위해 단일 행 표시

1. 다음 이미지와 유사하게 표시되는 결과를 검토하고 확인합니다.

    ![단일 행을 표시하는 성공적인 출력의 스크린샷](Images/notebook-transform-result.png)

이제 성공적으로 외부 데이터에 연결하고, parquet 파일에 쓰고, 데이터를 DataFrame에 로드하고, 데이터를 변환하고, 델타 테이블에 로드했습니다.

## SQL 쿼리를 사용하여 델타 테이블 데이터 분석

이 랩은 데이터 수집에 중점을 두어 *추출, 변환, 로드* 프로세스를 설명하지만, 데이터를 미리 보는 것도 중요합니다.

1. 새 코드 셀을 만들고 아래 코드를 삽입합니다.

    ```python
    # Load table into df
    delta_table_name = "yellow_taxi"
    table_df = spark.read.format("delta").table(delta_table_name)
    
    # Create temp SQL table
    table_df.createOrReplaceTempView("yellow_taxi_temp")
    
    # SQL Query
    table_df = spark.sql('SELECT * FROM yellow_taxi_temp')
    
    # Display 10 results
    display(table_df.limit(10))
    ```

1. 또 다른 코드 셀을 만들고 이 코드도 삽입합니다.

    ```python
    # Load table into df
    delta_table_name = "yellow_taxi_opt"
    opttable_df = spark.read.format("delta").table(delta_table_name)
    
    # Create temp SQL table
    opttable_df.createOrReplaceTempView("yellow_taxi_opt")
    
    # SQL Query to confirm
    opttable_df = spark.sql('SELECT * FROM yellow_taxi_opt')
    
    # Display results
    display(opttable_df.limit(10))
    ```

1. 이제 두 쿼리 중 첫 번째 쿼리에 대해 **셀 실행** 단추 옆에 있는 &#9660; 화살표를 선택하고 드롭다운에서 **이 셀과 아래 실행**을 선택합니다.

    그러면 마지막 두 개의 코드 셀이 실행됩니다. 최적화되지 않은 데이터가 있는 테이블과 최적화된 데이터가 있는 테이블을 쿼리하는 것 간의 실행 시간 차이를 확인합니다.

## 리소스 정리

이 연습에서는 Fabric에서 PySpark와 함께 Notebook을 사용하여 데이터를 로드하고 Parquet에 저장했습니다. 그런 다음, Parquet 파일을 사용하여 추가로 데이터를 변환했습니다. 마지막으로 SQL을 사용하여 델타 테이블을 쿼리했습니다.

탐색을 마치면 이 연습에서 만든 작업 영역을 삭제해도 됩니다.

1. 왼쪽 막대에서 작업 영역의 아이콘을 선택하여 포함된 모든 항목을 봅니다.
2. 도구 모음의 **...** 메뉴에서 **작업 영역 설정**을 선택합니다.
3. **일반** 섹션에서 **이 작업 영역 제거**를 선택합니다.
