---
lab:
  title: Microsoft Fabric에서 MLflow를 사용하여 기계 학습 모델 학습 및 추적
  module: Train and track machine learning models with MLflow in Microsoft Fabric
---

# Microsoft Fabric에서 MLflow를 사용하여 기계 학습 모델 학습 및 추적

이 랩에서는 당뇨병의 정량적 측정값을 예측하는 기계 학습 모델을 학습시킵니다. scikit-learn을 사용하여 회귀 모델을 학습시키고 모델을 추적하고 MLflow와 비교합니다.

이 랩을 완료하면 기계 학습 및 모델 추적에 대한 실무 경험을 쌓게 되며 Microsoft Fabric에서 *Notebook*, *실험*, *모델*을 사용하여 작업하는 방법을 알게 됩니다.

이 랩을 완료하는 데 약 **25**분이 걸립니다.

> **참고**: 이 연습을 완료하려면 [Microsoft Fabric 평가판](https://learn.microsoft.com/fabric/get-started/fabric-trial)이 필요합니다.

## 작업 영역 만들기

패브릭에서 데이터를 사용하기 전에 패브릭 평가판을 사용하도록 설정된 작업 영역을 만듭니다.

1. 브라우저에서 Microsoft Fabric 홈페이지(`https://app.fabric.microsoft.com`)로 이동하고 필요한 경우 Fabric 자격 증명을 사용해 로그인합니다.
1. Fabric 홈페이지에서 **Synapse 데이터 과학**을 선택합니다.
1. 왼쪽 메뉴 모음에서 **작업 영역**을 선택합니다(아이콘은 와 유사함).
1. Fabric 용량이 포함된 라이선스 모드(*평가판*, *프리미엄* 또는 *Fabric*)를 선택하여 원하는 이름으로 새 작업 영역을 만듭니다.
1. 새 작업 영역이 열리면 비어 있어야 합니다.

    ![Fabric의 빈 작업 영역 스크린샷.](./Images/new-workspace.png)

## Notebook 만들기

모델을 학습시키기 위해 *Notebook*을 만들 수 있습니다. Notebook은 여러 언어로 코드를 작성하고 실행할 수 있는 대화형 환경을 제공합니다.

1. **Synapse 데이터 과학** 홈페이지에서 새 **Notebook**을 만듭니다.

    몇 초 후에 단일 *셀*이 포함된 새 Notebook이 열립니다. Notebook은 *코드* 또는 *markdown*(서식이 지정된 텍스트)을 포함할 수 있는 하나 이상의 셀로 구성됩니다.

1. 첫 번째 셀(현재 *코드* 셀)을 선택한 다음 오른쪽 상단에 있는 동적 도구 모음에서 **M&#8595;** 단추를 눌러 셀을 *markdown* 셀로 변환합니다.

    셀이 markdown 셀로 변경되면 포함된 텍스트가 렌더링됩니다.

1. 필요한 경우 **&#128393;**(편집) 단추를 사용하여 셀을 편집 모드로 전환한 다음 내용을 삭제하고 다음 텍스트를 입력합니다.

    ```text
   # Train a machine learning model and track with MLflow
    ```

## 데이터 프레임에 데이터 로드

이제 코드를 실행하여 데이터를 가져와 모델을 학습시킬 준비가 되었습니다. Azure Open Datasets의 [당뇨병 데이터 세트](https://learn.microsoft.com/azure/open-datasets/dataset-diabetes?tabs=azureml-opendatasets?azure-portal=true)를 사용하여 작업합니다. 데이터를 로드한 후에는 데이터를 Pandas 데이터 프레임으로 변환합니다. 데이터 프레임이란 행과 열로 정리된 데이터 작업을 위한 일반적인 구조입니다.

1. Notebook에서 최신 셀 출력 아래의 **+ 코드** 아이콘을 사용하여 Notebook에 새 코드 셀을 추가합니다.

    > **팁**: **+ 코드** 아이콘을 보려면 마우스를 현재 셀의 출력 바로 아래 왼쪽으로 이동합니다. 아니면 메뉴 모음의 **편집** 탭에서 **+ 코드 셀 추가**를 선택합니다.

1. 그 안에 다음 코드를 입력합니다.

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

1. 셀 왼쪽의 **&#9655;셀 실행** 단추를 이용하여 실행합니다. 아니면 키보드에서 **SHIFT** + **ENTER** 키를 눌러 셀을 실행할 수 있습니다.

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

    출력은 당뇨병 데이터 세트의 행과 열을 보여 줍니다.

1. 데이터는 Spark 데이터 프레임으로 로드됩니다. Scikit-learn은 입력 데이터 세트가 Pandas 데이터 프레임일 것으로 예상합니다. 아래 코드를 실행하여 데이터 세트를 Pandas 데이터 프레임으로 변환합니다.

    ```python
   import pandas as pd
   df = df.toPandas()
   df.head()
    ```

## 기계 학습 모델 학습

이제 데이터를 로드했으므로 이를 사용하여 기계 학습 모델을 학습시키고 당뇨병의 정량적 측정값을 예측할 수 있습니다. scikit-learn 라이브러리를 사용하여 회귀 모델을 학습시키고 MLflow를 사용하여 모델을 추적하겠습니다.

1. 다음 코드를 실행하여 데이터를 학습 및 테스트 데이터 세트로 분할하고 예측하려는 레이블에서 기능을 분리합니다.

    ```python
   from sklearn.model_selection import train_test_split
    
   X, y = df[['AGE','SEX','BMI','BP','S1','S2','S3','S4','S5','S6']].values, df['Y'].values
    
   X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.30, random_state=0)
    ```

1. Notebook에 다른 새 코드 셀을 추가하고 여기에 다음 코드를 입력한 후 실행합니다.

    ```python
   import mlflow
   experiment_name = "experiment-diabetes"
   mlflow.set_experiment(experiment_name)
    ```

    이 코드는 **experiment-diabetes**라는 MLflow 실험을 만듭니다. 모델은 이 실험에서 추적됩니다.

1. Notebook에 다른 새 코드 셀을 추가하고 여기에 다음 코드를 입력한 후 실행합니다.

    ```python
   from sklearn.linear_model import LinearRegression
    
   with mlflow.start_run():
      mlflow.autolog()
    
      model = LinearRegression()
      model.fit(X_train, y_train)
    
      mlflow.log_param("estimator", "LinearRegression")
    ```

    이 코드는 선형 회귀를 사용하여 회귀 모델을 학습시킵니다. 매개 변수, 메트릭, 아티팩트가 MLflow를 통해 자동으로 로깅됩니다. 또한 *LinearRegression* 값을 사용하여 **estimator**라는 매개 변수도 로깅됩니다.

1. Notebook에 다른 새 코드 셀을 추가하고 여기에 다음 코드를 입력한 후 실행합니다.

    ```python
   from sklearn.tree import DecisionTreeRegressor
    
   with mlflow.start_run():
      mlflow.autolog()
    
      model = DecisionTreeRegressor(max_depth=5) 
      model.fit(X_train, y_train)
    
      mlflow.log_param("estimator", "DecisionTreeRegressor")
    ```

    이 코드는 의사 결정 트리 회귀 변수를 사용하여 회귀 모델을 학습시킵니다. 매개 변수, 메트릭, 아티팩트가 MLflow를 통해 자동으로 로깅됩니다. 또한 *DecisionTreeRegressor* 값을 사용하여 **estimator**라는 매개 변수도 로깅됩니다.

## MLflow를 사용하여 실험 검색 및 보기

MLflow를 사용하여 모델을 학습시키고 추적한 경우 MLflow 라이브러리를 사용하여 실험과 해당 세부 정보를 검색할 수 있습니다.

1. 모든 실험을 나열하려면 다음 코드를 사용합니다.

    ```python
   import mlflow
   experiments = mlflow.search_experiments()
   for exp in experiments:
       print(exp.name)
    ```

1. 특정 실험을 검색하려면 해당 이름을 사용해 확인할 수 있습니다.

    ```python
   experiment_name = "experiment-diabetes"
   exp = mlflow.get_experiment_by_name(experiment_name)
   print(exp)
    ```

1. 실험 이름을 사용하면 해당 실험의 모든 작업을 확인할 수 있습니다.

    ```python
   mlflow.search_runs(exp.experiment_id)
    ```

1. 작업 실행 및 출력을 보다 쉽게 비교하려면 검색을 구성하여 결과를 정렬하면 됩니다. 예를 들어 다음 셀은 *start_time*을 기준으로 결과를 정렬하고 최대 2개의 결과만 표시합니다.

    ```python
   mlflow.search_runs(exp.experiment_id, order_by=["start_time DESC"], max_results=2)
    ```

1. 마지막으로 여러 모델의 평가 메트릭을 나란히 나타내면 모델을 쉽게 비교할 수 있습니다.

    ```python
   import matplotlib.pyplot as plt
   
   df_results = mlflow.search_runs(exp.experiment_id, order_by=["start_time DESC"], max_results=2)[["metrics.training_r2_score", "params.estimator"]]
   
   fig, ax = plt.subplots()
   ax.bar(df_results["params.estimator"], df_results["metrics.training_r2_score"])
   ax.set_xlabel("Estimator")
   ax.set_ylabel("R2 score")
   ax.set_title("R2 score by Estimator")
   for i, v in enumerate(df_results["metrics.training_r2_score"]):
       ax.text(i, v, str(round(v, 2)), ha='center', va='bottom', fontweight='bold')
   plt.show()
    ```

    다음 이미지와 유사한 출력을 확인할 수 있습니다.

    ![표현된 평가 메트릭의 스크린샷.](./Images/data-science-metrics.png)

## 실험 살펴보기

Microsoft Fabric은 모든 실험을 추적하며, 이를 사용자가 시각적으로 탐색할 수 있도록 해 줍니다.

1. 왼쪽의 메뉴 모음에서 작업 영역으로 이동합니다.
1. **experiment-diabetes** 실험을 선택하여 엽니다.

    > **팁:** 로깅된 실험 실행이 표시되지 않으면 페이지를 새로 고칩니다.

1. **보기** 탭을 선택합니다.
1. **목록 실행**을 선택합니다.
1. 각 확인란에 표시하여 두 개의 마지막 실행을 선택합니다.

    그러면 두 개의 마지막 실행이 **메트릭 비교** 창에서 서로 비교됩니다. 기본적으로 메트릭은 실행 이름으로 표현됩니다.

1. 각 실행에 대한 평균 절대 오차를 시각화하는 그래프의 **&#128393;**(편집) 단추를 선택합니다.
1. **시각화 유형**을 **막대**로 변경합니다.
1. **X축**을 **estimator**로 변경합니다.
1. **바꾸기**를 선택하고 새 그래프를 살펴봅니다.
1. 필요에 따라 **메트릭 비교** 창에서 다른 그래프에 대해 이러한 단계를 반복할 수 있습니다.

로깅된 estimator별 성능 메트릭을 표시하면 어떤 알고리즘이 더 나은 모델을 생성했는지 검토할 수 있습니다.

## 모델 저장

실험 실행에서 학습시킨 기계 학습 모델을 비교한 후 가장 성능이 좋은 모델을 선택할 수 있습니다. 최상의 성능을 보여주는 모델을 사용하려면 모델을 저장하고 이를 사용하여 예측을 생성합니다.

1. 실험 개요에서 **보기** 탭이 선택되어 있는지 확인합니다.
1. **실행 세부 정보**를 선택합니다.
1. 학습 R2 점수가 가장 높은 실행을 선택합니다.
1. **실행을 모델로 저장** 상자에서 **저장**을 선택합니다(오른쪽으로 스크롤해야 이 내용이 보일 수도 있음).
1. 새로 열린 팝업 창에서 **새 모델 만들기**를 선택합니다.
1. **모델** 폴더를 선택합니다.
1. 모델 이름을 `model-diabetes`로 지정하고 **저장**을 선택합니다.
1. 모델을 만들 때 화면 오른쪽 위에 표시되는 알림에서 **ML 모델 보기**를 선택합니다. 창을 새로 고칠 수도 있습니다. 저장된 모델은 **모델 버전** 아래에 링크되어 있습니다.

모델, 실험, 실험 실행이 링크되어 있으므로 모델이 어떻게 학습되었는지 검토할 수 있습니다.

## Notebook 저장 및 Spark 세션 종료

이제 모델 학습 및 평가를 마쳤으므로 의미 있는 이름을 사용해 Notebook을 저장하고 Spark 세션을 종료할 수 있습니다.

1. Notebook으로 돌아가서 Notebook 메뉴 모음에서 ⚙️ **설정** 아이콘을 사용해 Notebook 설정을 봅니다.
2. Notebook **이름**을 **모델 학습 및 비교**로 설정한 다음 설정 창을 닫습니다.
3. Notebook 메뉴에서 **세션 중지**를 선택하여 Spark 세션을 종료합니다.

## 리소스 정리

이 연습에서는 Notebook을 만들고 기계 학습 모델을 학습시켰습니다. scikit-learn을 사용하여 모델 및 MLflow를 학습시켜 성능을 추적했습니다.

모델과 실험을 다 살펴보았다면 이 연습을 위해 만든 작업 영역을 삭제해도 됩니다.

1. 왼쪽 막대에서 작업 영역의 아이콘을 선택하여 포함된 모든 항목을 봅니다.
2. 도구 모음의 **...** 메뉴에서 **작업 영역 설정**을 선택합니다.
3. **일반** 섹션에서 **이 작업 영역 제거**를 선택합니다.
