---
lab:
  title: Microsoft Fabric에서 데이터 과학 시작
  module: Get started with data science in Microsoft Fabric
---

# Microsoft Fabric에서 데이터 과학 시작

이 랩에서는 데이터를 수집하고, Notebook의 데이터를 탐색하고, 데이터 랭글러를 사용하여 데이터를 처리하고, 두 가지 유형의 모델을 학습합니다. 이러한 모든 단계를 수행하면 Microsoft Fabric의 데이터 과학 기능을 탐색할 수 있습니다.

이 랩을 완료하면 기계 학습 및 모델 추적에 대한 실습 경험을 얻고 Microsoft Fabric에서 Notebook *, *Data Wrangler*, *실험* 및 *모델을* 사용하는 *방법을 알아봅니다.

이 랩을 완료하는 데 약 **20**분이 걸립니다.

> **참고**: 이 연습을 완료하려면 [Microsoft Fabric 평가판](https://learn.microsoft.com/fabric/get-started/fabric-trial)이 필요합니다.

## 작업 영역 만들기

패브릭에서 데이터를 사용하기 전에 패브릭 평가판을 사용하도록 설정된 작업 영역을 만듭니다.

1. 브라우저의 [https://app.fabric.microsoft.com](https://app.fabric.microsoft.com)에서 Microsoft Fabric 홈페이지로 이동합니다.
1. Synapse 데이터 과학** 선택합니다**.
1. 왼쪽 메뉴 모음에서 **작업 영역**을 선택합니다(아이콘은 와 유사함).
1. Fabric 용량이 포함된 라이선스 모드(*평가판*, *프리미엄* 또는 *Fabric*)를 선택하여 원하는 이름으로 새 작업 영역을 만듭니다.
1. 새 작업 영역이 열리면 비어 있어야 합니다.

    ![Fabric의 빈 작업 영역 스크린샷.](./Images/new-workspace.png)

## Notebook 만들기

코드를 실행하려면 Notebook*을 *만들 수 있습니다. Notebook은 여러 언어로 코드를 작성하고 실행할 수 있는 대화형 환경을 제공합니다.

1. **Synapse 데이터 과학** 홈페이지에서 새 **Notebook**을 만듭니다.

    몇 초 후에 단일 *셀*이 포함된 새 Notebook이 열립니다. Notebook은 *코드* 또는 *markdown*(서식이 지정된 텍스트)을 포함할 수 있는 하나 이상의 셀로 구성됩니다.

1. 첫 번째 셀(현재 *코드* 셀)을 선택한 다음 오른쪽 상단에 있는 동적 도구 모음에서 **M&#8595;** 단추를 눌러 셀을 *markdown* 셀로 변환합니다.

    셀이 markdown 셀로 변경되면 포함된 텍스트가 렌더링됩니다.

1. **&#128393;**(편집) 단추를 사용하여 셀을 편집 모드로 전환한 다음 콘텐츠를 삭제하고 다음 텍스트를 입력합니다.

    ```text
   # Data science in Microsoft Fabric
    ```

## 데이터 가져오기

이제 코드를 실행하여 데이터를 가져와 모델을 학습시킬 준비가 되었습니다. Azure Open Datasets의 [당뇨병 데이터 세트](https://learn.microsoft.com/azure/open-datasets/dataset-diabetes?tabs=azureml-opendatasets?azure-portal=true)를 사용하여 작업합니다. 데이터를 로드한 후에는 데이터를 Pandas 데이터 프레임으로 변환합니다. 데이터 프레임이란 행과 열로 정리된 데이터 작업을 위한 일반적인 구조입니다.

1. Notebook에서 최신 셀 출력 아래의 **+ 코드** 아이콘을 사용하여 Notebook에 새 코드 셀을 추가합니다.

    > **팁**: **+ 코드** 아이콘을 보려면 마우스를 현재 셀의 출력 바로 아래 왼쪽으로 이동합니다. 아니면 메뉴 모음의 **편집** 탭에서 **+ 코드 셀 추가**를 선택합니다.

1. 새 코드 셀에 다음 코드를 입력합니다.

    ```python
   # Azure storage access info for open dataset diabetes
   blob_account_name = "azureopendatastorage"
   blob_container_name = "mlsamples"
   blob_relative_path = "diabetes"
   blob_sas_token = r"" # Blank since container is Anonymous access
    
   # Set Spark config to access  blob storage
   wasbs_path = f"wasbs://%s@%s.blob.core.windows.net/%s" % (blob_container_name, blob_account_name, blob_relative_path)
   spark.conf.set("fs.azure.sas.%s.%s.blob.core.windows.net" % (blob_container_name, blob_account_name), blob_sas_token)
   print("Remote blob path: " + wasbs_path)
    
   # Spark read parquet, note that it won't load any data yet by now
   df = spark.read.parquet(wasbs_path)
    ```

1. 셀 왼쪽의 **&#9655;셀 실행** 단추를 이용하여 실행합니다. 또는 키보드를 눌러 `SHIFT` + `ENTER` 셀을 실행할 수 있습니다.

    > **참고**: 이 세션에서 Spark 코드를 실행한 것은 이번이 처음이기 때문에 Spark 풀이 시작되어야 합니다. 이는 세션의 첫 번째 실행이 완료되는 데 1분 정도 걸릴 수 있음을 의미합니다. 후속 실행은 더 빨라질 것입니다.

1. 셀 출력 아래의 **+ 코드** 아이콘을 사용하여 Notebook에 새 코드 셀을 추가하고 그 안에 다음 코드를 입력합니다.

    ```python
   display(df)
    ```

1. 셀 명령이 완료되면 셀 아래의 출력을 검토합니다. 다음과 유사한 출력을 확인할 수 있습니다.

    |나이|SEX|BMI|BP|S1|S2|S3|S4|S5|S6|Y|
    |---|---|---|--|--|--|--|--|--|--|--|
    |59|2|32.1|101.0|157|93.2|38.0|4.0|4.8598|87|151|
    |48|1|21.6|87.0|183|103.2|70.0|3.0|3.8918|69|75|
    |72|2|30.5|93.0|156|93.6|41.0|4.0|4.6728|85|141|
    |24|1|25.3|84.0|198|131.4|40.0|5.0|4.8903|89|206|
    |50|1|23.0|101.0|192|125.4|52.0|4.0|4.2905|80|135|
    | ... | ... | ... | ... | ... | ... | ... | ... | ... | ... | ... |

    출력은 당뇨병 데이터 세트의 행과 열을 보여줍니다.

1. 렌더링된 테이블의 맨 위에는 테이블 **** 과 **차트라는 두 개의 탭이 있습니다**. 차트**를 선택합니다**.
1. 차트의 **오른쪽 위에 있는 보기 옵션을** 선택하여 시각화를 변경합니다.
1. 차트를 다음 설정으로 변경합니다.
    * **차트 종류**: `Box plot`
    * **키**: *비워 둡니다.*
    * **값**: `Y`
1. 적용**을 선택하여 **새 시각화를 렌더링하고 출력을 탐색합니다.

## 데이터 준비

이제 데이터를 수집하고 탐색했으므로 데이터를 변환할 수 있습니다. Notebook에서 코드를 실행하거나 데이터 랭글러를 사용하여 코드를 생성할 수 있습니다.

1. 데이터는 Spark 데이터 프레임으로 로드됩니다. 데이터 랭글러를 시작하려면 데이터를 Pandas 데이터 프레임으로 변환해야 합니다. Notebook에서 다음 코드를 실행합니다.

    ```python
   df = df.toPandas()
   df.head()
    ```

1. Notebook 리본에서 데이터를 선택한 **다음 데이터 랭글러** 드롭다운에서 데이터 프레임 변환을 선택합니다**.**
1. `df` 데이터 세트를 선택합니다. 데이터 랭글러가 시작되면 요약** 패널에서 데이터 프레임에 대한 설명적인 개요가 **생성됩니다.

    현재 레이블 열은 `Y`연속 변수입니다. Y를 예측하는 기계 학습 모델을 학습하려면 회귀 모델을 학습시켜야 합니다. Y의 (예측) 값은 해석하기 어려울 수 있습니다. 대신, 우리는 누군가가 당뇨병 개발을 위한 낮은 리스크 또는 고위험인지 예측하는 분류 모형을 훈련하는 것을 탐구할 수 있습니다. 분류 모델을 학습하려면 다음 값을 `Y`기반으로 이진 레이블 열을 만들어야 합니다.

1. `Y` 데이터 랭글러에서 열을 선택합니다. bin의 빈도는 감소합니다 `220-240` . 75번째 백분위수 `211.5` 는 히스토그램에 있는 두 영역의 전환과 대체로 일치합니다. 이 값을 낮은 위험 및 높은 위험의 임계값으로 사용하겠습니다.
1. 작업 패널로 **이동하여 수식을** 확장**한 다음 수식**에서 열 만들기를 선택합니다**.**
1. 다음 설정을 사용하여 새 열을 만듭니다.
    * **열 이름**: `Risk`
    * **열 수식**: `(df['Y'] > 211.5).astype(int)`
1. 미리 보기에 추가된 새 열을 `Risk` 검토합니다. 값 `1` 이 있는 행 수가 모든 행의 약 25%(75번째 백분위 `Y`수)여야 하는지 확인합니다.
1. **적용**을 선택합니다.
1. Notebook**에 코드 추가를 선택합니다**.
1. 데이터 랭글러에서 생성한 코드를 사용하여 셀을 실행합니다.
1. 새 셀에서 다음 코드를 실행하여 열의 `Risk` 모양이 예상대로 표시되는지 확인합니다.

    ```python
   df_clean.describe()
    ```

## 기계 학습 모델 학습

이제 데이터를 준비했으므로 이 데이터를 사용하여 당뇨병을 예측하는 기계 학습 모델을 학습시킬 수 있습니다. 데이터 세트를 사용하여 두 가지 유형의 모델을 학습시킬 수 있습니다. 회귀 모델(예측 `Y`) 또는 분류 모델(예측 `Risk`). scikit-learn 라이브러리를 사용하여 모델을 학습시키고 MLflow를 사용하여 모델을 추적합니다.

### 회귀 모델 학습

1. 다음 코드를 실행하여 데이터를 학습 및 테스트 데이터 세트로 분할하고 예측하려는 레이블 `Y` 에서 기능을 분리합니다.

    ```python
   from sklearn.model_selection import train_test_split
    
   X, y = df_clean[['AGE','SEX','BMI','BP','S1','S2','S3','S4','S5','S6']].values, df_clean['Y'].values
    
   X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.30, random_state=0)
    ```

1. Notebook에 다른 새 코드 셀을 추가하고 여기에 다음 코드를 입력한 후 실행합니다.

    ```python
   import mlflow
   experiment_name = "diabetes-regression"
   mlflow.set_experiment(experiment_name)
    ```

    이 코드는 라는 `diabetes-regression`MLflow 실험을 만듭니다. 모델은 이 실험에서 추적됩니다.

1. Notebook에 다른 새 코드 셀을 추가하고 여기에 다음 코드를 입력한 후 실행합니다.

    ```python
   from sklearn.linear_model import LinearRegression
    
   with mlflow.start_run():
      mlflow.autolog()
    
      model = LinearRegression()
      model.fit(X_train, y_train)
    ```

    이 코드는 선형 회귀를 사용하여 회귀 모델을 학습시킵니다. 매개 변수, 메트릭, 아티팩트가 MLflow를 통해 자동으로 로깅됩니다.

### 분류 모델 학습

1. 다음 코드를 실행하여 데이터를 학습 및 테스트 데이터 세트로 분할하고 예측하려는 레이블 `Risk` 에서 기능을 분리합니다.

    ```python
   from sklearn.model_selection import train_test_split
    
   X, y = df_clean[['AGE','SEX','BMI','BP','S1','S2','S3','S4','S5','S6']].values, df_clean['Risk'].values
    
   X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.30, random_state=0)
    ```

1. Notebook에 다른 새 코드 셀을 추가하고 여기에 다음 코드를 입력한 후 실행합니다.

    ```python
   import mlflow
   experiment_name = "diabetes-classification"
   mlflow.set_experiment(experiment_name)
    ```

    이 코드는 라는 `diabetes-classification`MLflow 실험을 만듭니다. 모델은 이 실험에서 추적됩니다.

1. Notebook에 다른 새 코드 셀을 추가하고 여기에 다음 코드를 입력한 후 실행합니다.

    ```python
   from sklearn.linear_model import LogisticRegression
    
   with mlflow.start_run():
       mlflow.sklearn.autolog()

       model = LogisticRegression(C=1/0.1, solver="liblinear").fit(X_train, y_train)
    ```

    이 코드는 로지스틱 회귀를 사용하여 분류 모델을 학습합니다. 매개 변수, 메트릭, 아티팩트가 MLflow를 통해 자동으로 로깅됩니다.

## 실험 살펴보기

Microsoft Fabric은 모든 실험을 추적하며, 이를 사용자가 시각적으로 탐색할 수 있도록 해 줍니다.

1. 왼쪽의 허브 메뉴 모음에서 작업 영역으로 이동합니다.
1. `diabetes-regression` 실험을 선택하여 엽니다.

    > **팁:** 로깅된 실험 실행이 표시되지 않으면 페이지를 새로 고칩니다.

1. 실행 메트릭을 **** 검토하여 회귀 모델이 정확한지 알아보세요.
1. 홈페이지로 다시 이동하여 실험을 선택하여 `diabetes-classification` 엽니다.
1. **실행 메트릭을 검토하여 분류 모델의 정확도를 살펴봅니다**. 메트릭 유형은 다른 유형의 모델을 학습할 때 다릅니다.

## 모델 저장

실험에서 학습한 기계 학습 모델을 비교한 후 가장 성능이 좋은 모델을 선택할 수 있습니다. 최상의 성능을 보여주는 모델을 사용하려면 모델을 저장하고 이를 사용하여 예측을 생성합니다.

1. 실험 리본에서 ML 모델**로 저장을 선택합니다**.
1. 새로 열린 팝업 창에서 새 ML 모델** 만들기를 선택합니다**.
1. `model` 폴더를 선택합니다.
1. 모델 이름을 `model-diabetes`로 지정하고 **저장**을 선택합니다.
1. 모델을 만들 때 화면 오른쪽 위에 표시되는 알림에서 **ML 모델 보기**를 선택합니다. 창을 새로 고칠 수도 있습니다. 저장된 모델은 ML 모델 버전**에서 **연결됩니다.

모델, 실험, 실험 실행이 링크되어 있으므로 모델이 어떻게 학습되었는지 검토할 수 있습니다.

## Notebook 저장 및 Spark 세션 종료

이제 모델 학습 및 평가를 마쳤으므로 의미 있는 이름을 사용해 Notebook을 저장하고 Spark 세션을 종료할 수 있습니다.

1. 전자 필기장 메뉴 모음에서 ⚙️ **설정** 아이콘을 사용하여 전자 필기장 설정을 봅니다.
2. Notebook **이름**을 **모델 학습 및 비교**로 설정한 다음 설정 창을 닫습니다.
3. Notebook 메뉴에서 **세션 중지**를 선택하여 Spark 세션을 종료합니다.

## 리소스 정리

이 연습에서는 Notebook을 만들고 기계 학습 모델을 학습시켰습니다. scikit-learn을 사용하여 모델 및 MLflow를 학습시켜 성능을 추적했습니다.

모델과 실험을 다 살펴보았다면 이 연습을 위해 만든 작업 영역을 삭제해도 됩니다.

1. 왼쪽 막대에서 작업 영역의 아이콘을 선택하여 포함된 모든 항목을 봅니다.
2. 도구 모음의 **...** 메뉴에서 **작업 영역 설정**을 선택합니다.
3. **기타** 섹션에서 **이 작업 영역 제거**를 선택합니다.
