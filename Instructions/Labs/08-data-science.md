---
lab:
  title: 고객 변동을 예측하는 분류 모델 학습
  module: Get started with data science in Microsoft Fabric
---

# Notebook을 사용하여 Microsoft Fabric에서 모델 학습

이 랩에서는 Microsoft Fabric을 사용하여 Notebook을 만들고 기계 학습 모델을 학습하여 고객 변동을 예측합니다. Scikit-Learn을 사용하여 모델 및 MLflow를 학습하여 성능을 추적합니다. 고객 이탈은 많은 회사에서 직면하는 중요한 비즈니스 문제이며, 변동 가능성이 있는 고객을 예측하면 회사가 고객을 유지하고 수익을 늘리는 데 도움이 될 수 있습니다. 이 랩을 완료하면 기계 학습 및 모델 추적에 대한 실습 경험을 얻고 Microsoft Fabric을 사용하여 프로젝트에 대한 Notebook을 만드는 방법을 알아봅니다.

이 랩을 완료하는 데 약 **45**분이 걸립니다.

> **참고**: 이 연습을 완료하려면 Microsoft Fabric 라이선스가 필요합니다. 무료 [Fabric](https://learn.microsoft.com/fabric/get-started/fabric-trial) 평가판 라이선스를 사용하도록 설정하는 방법에 대한 자세한 내용은 Fabric 시작을 참조하세요. 이 작업을 수행하려면 Microsoft *학교* 또는 *회사* 계정이 필요합니다. 없는 경우 [Microsoft Office 365 E3 이상의 평가판에 등록](https://www.microsoft.com/microsoft-365/business/compare-more-office-365-for-business-plans)할 수 있습니다.

## 작업 영역 만들기

패브릭에서 데이터를 사용하기 전에 패브릭 평가판을 사용하도록 설정된 작업 영역을 만듭니다.

1. Microsoft Fabric에 로그인[하고 Power BI**를 선택합니다**.`https://app.fabric.microsoft.com`](https://app.fabric.microsoft.com)
2. 왼쪽 메뉴 모음에서 **작업 영역**을 선택합니다(아이콘은 와 유사함).
3. 선택한 이름으로 새 작업 영역을 만들고 패브릭 용량(*평가판*, *프리미엄* 또는 *패브릭*)이 포함된 라이선스 모드를 선택합니다.
4. 새 작업 영역이 열리면 다음과 같이 비어 있어야 합니다.

    ![Power BI의 빈 작업 영역 스크린샷](./Images/new-workspace.png)

## Lakehouse 만들기 및 파일 업로드

이제 작업 영역이 있으므로 포털에서 데이터 과학* 환경으로 전환*하고 분석하려는 데이터 파일에 대한 데이터 레이크하우스를 만들어야 합니다.

1. Power BI 포털의 왼쪽 아래에서 Power BI 아이콘을 **선택하고 데이터 엔지니어 환경**으로 **전환**합니다.
1. **데이터 엔지니어링** 홈페이지에서 원하는 이름의 새 **레이크하우스**를 만듭니다.

    1분 정도 지나면 테이블**이나 **파일이** 없는 **새 레이크하우스가 만들어집니다. 분석을 위해 데이터 레이크하우스에 일부 데이터를 수집해야 합니다. 이 작업을 수행하는 방법에는 여러 가지가 있지만, 이 연습에서는 로컬 컴퓨터(또는 해당하는 경우 랩 VM)의 텍스트 파일 폴더를 다운로드하여 추출한 다음 레이크하우스에 업로드하기만 하면 됩니다.

1. 이 연습[https://raw.githubusercontent.com/MicrosoftLearning/dp-data/main/churn.csv](https://raw.githubusercontent.com/MicrosoftLearning/dp-data/main/churn.csv)의 `churn.csv` CSV 파일을 다운로드하여 저장합니다.


1. 레이크하우스가 포함된 웹 브라우저 탭으로 돌아가서 Lake 보기** 창의 **파일** 노드에 **대한 **...** 메뉴에서 파일** 업로드** 및 **업로드를 선택한 **다음, 로컬 컴퓨터(또는 해당하는 경우 랩 VM)의 churn.csv** 파일을 Lakehouse에 업로드**합니다.
6. 파일이 업로드된 후 파일을** 확장하고 **CSV 파일이 업로드되었는지 확인합니다.

## Notebook 만들기

모델을 학습하려면 Notebook*을 *만들 수 있습니다. Notebook은 여러 언어로 코드를 작성하고 실험*으로 *실행할 수 있는 대화형 환경을 제공합니다.

1. Power BI 포털의 왼쪽 아래에서 데이터 엔지니어링 아이콘을 **선택하고 데이터 과학** 환경으로 **전환**합니다.

1. **데이터 과학** 홈페이지에서 새 **Notebook**을 만듭니다.

    몇 초 후에 단일 *셀* 이 포함된 새 Notebook이 열립니다. Notebook은 코드 또는 markdown *(서식이 지정된 텍스트)을 포함*할 수 있는 하나 이상의 셀로 구성*됩니다.*

1. 첫 번째 셀(현재 코드* 셀)을 *선택한 다음 오른쪽 위에 있는 동적 도구 모음에서 M&#8595;** 단추를 사용하여 **셀을 markdown* 셀로 *변환합니다.

    셀이 markdown 셀로 변경되면 포함된 텍스트가 렌더링됩니다.

1. **&#128393;**(편집) 단추를 사용하여 셀을 편집 모드로 전환한 다음 콘텐츠를 삭제하고 다음 텍스트를 입력합니다.

    ```text
   # Train a machine learning model and track with MLflow

   Use the code in this notebook to train and track models.
    ``` 

## 데이터 프레임에 데이터 로드

이제 코드를 실행하여 데이터를 준비하고 모델을 학습할 준비가 되었습니다. 데이터를 사용하려면 데이터 프레임을* 사용합니다*. Spark의 데이터 프레임은 Python의 Pandas 데이터 프레임과 유사하며 행 및 열에서 데이터를 사용하기 위한 일반적인 구조를 제공합니다.

1. **레이크하우스** 추가 창에서 추가**를 선택하여 **레이크하우스를 추가합니다.
1. 기존 레이크하우스**를 선택하고 **추가**를 선택합니다**.
1. 이전 섹션에서 만든 레이크하우스를 선택합니다.
1. **CSV 파일이 Notebook 편집기 옆에 나열되도록 파일** 폴더를 확장합니다.
1. churn.csv ...** 메뉴에서 데이터** > **Pandas 로드를** 선택합니다.******** 다음 코드를 포함하는 새 코드 셀을 Notebook에 추가해야 합니다.

    ```python
   import pandas as pd
   # Load data into pandas DataFrame from "/lakehouse/default/" + "Files/churn.csv"
   df = pd.read_csv("/lakehouse/default/" + "Files/churn.csv")
   display(df)
    ```

    > **팁**: 아이콘을 사용하여 **<<** 왼쪽에 있는 파일이 포함된 창을 숨길 수 있습니다. 이렇게 하면 전자 필기장을 집중하게 됩니다.

1. 사용 하 여 **는 &#9655; 셀 왼쪽의 셀** 단추를 실행하여 실행합니다.

    > **참고**: 이 세션에서 Spark 코드를 처음 실행했으므로 Spark 풀을 시작해야 합니다. 즉, 세션의 첫 번째 실행을 완료하는 데 1분 정도 걸릴 수 있습니다. 후속 실행은 더 빨라질 것입니다.

1. 셀 명령이 완료되면 셀 아래의 출력을 검토합니다. 이 출력은 다음과 유사합니다.

    |색인|CustomerID|years_with_company|total_day_calls|total_eve_calls|total_night_calls|total_intl_calls|average_call_minutes|total_customer_service_calls|연령|변동|
    | -- | -- | -- | -- | -- | -- | -- | -- | -- | -- | -- |
    |1|1000038|0|117|88|32|607|43.90625678|0.810828179|34|0|
    |2|1000183|1|164|102|22|40|49.82223317|0.294453889|35|0|
    |3|1000326|3|116|43|45|207|29.83377967|1.344657937|57|1|
    |4|1000340|0|92|24|11|37|31.61998183|0.124931779|34|0|
    | ... | ... | ... | ... | ... | ... | ... | ... | ... | ... | ... |

    출력은 churn.csv 파일에서 고객 데이터의 행과 열을 보여 줍니다.

## 기계 학습 모델 학습

이제 데이터를 로드했으므로 이를 사용하여 기계 학습 모델을 학습시키고 고객 변동을 예측할 수 있습니다. Scikit-Learn 라이브러리를 사용하여 모델을 학습시키고 MLflow를 사용하여 모델을 추적합니다. 

1. **셀 출력 아래의 + 코드** 아이콘을 사용하여 Notebook에 새 코드 셀을 추가하고 그 안에 다음 코드를 입력합니다.

    ```python
   from sklearn.model_selection import train_test_split

   print("Splitting data...")
   X, y = df[['years_with_company','total_day_calls','total_eve_calls','total_night_calls','total_intl_calls','average_call_minutes','total_customer_service_calls','age']].values, df['churn'].values
   
   X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.30, random_state=0)
    ```

1. 추가한 코드 셀을 실행하고 데이터 세트에서 'CustomerID'를 생략하고 데이터를 학습 및 테스트 데이터 세트로 분할합니다.
1. Notebook에 다른 새 코드 셀을 추가하고, 여기에 다음 코드를 입력하고, 실행합니다.
    
    ```python
   import mlflow
   experiment_name = "experiment-churn"
   mlflow.set_experiment(experiment_name)
    ```
    
    이 코드는 라는 `experiment-churn`MLflow 실험을 만듭니다. 모델은 이 실험에서 추적됩니다.

1. Notebook에 다른 새 코드 셀을 추가하고, 여기에 다음 코드를 입력하고, 실행합니다.

    ```python
   from sklearn.linear_model import LogisticRegression
   
   with mlflow.start_run():
       mlflow.autolog()

       model = LogisticRegression(C=1/0.1, solver="liblinear").fit(X_train, y_train)

       mlflow.log_param("estimator", "LogisticRegression")
    ```
    
    이 코드는 로지스틱 회귀를 사용하여 분류 모델을 학습합니다. 매개 변수, 메트릭 및 아티팩트가 MLflow를 사용하여 자동으로 기록됩니다. 또한 값`LogisticRegression`과 함께 호출`estimator`된 매개 변수를 로깅합니다.

1. Notebook에 다른 새 코드 셀을 추가하고, 여기에 다음 코드를 입력하고, 실행합니다.

    ```python
   from sklearn.tree import DecisionTreeClassifier
   
   with mlflow.start_run():
       mlflow.autolog()

       model = DecisionTreeClassifier().fit(X_train, y_train)
   
       mlflow.log_param("estimator", "DecisionTreeClassifier")
    ```

    이 코드는 의사 결정 트리 분류자를 사용하여 분류 모델을 학습합니다. 매개 변수, 메트릭 및 아티팩트가 MLflow를 사용하여 자동으로 기록됩니다. 또한 값`DecisionTreeClassifier`과 함께 호출`estimator`된 매개 변수를 로깅합니다.

## MLflow를 사용하여 실험 검색 및 보기

MLflow를 사용하여 모델을 학습하고 추적한 경우 MLflow 라이브러리를 사용하여 실험 및 세부 정보를 검색할 수 있습니다.

1. 모든 실험을 나열하려면 다음 코드를 사용합니다.

    ```python
   import mlflow
   experiments = mlflow.search_experiments()
   for exp in experiments:
       print(exp.name)
    ```

1. 특정 실험을 검색하려면 해당 이름으로 가져올 수 있습니다.

    ```python
   experiment_name = "experiment-churn"
   exp = mlflow.get_experiment_by_name(experiment_name)
   print(exp)
    ```

1. 실험 이름을 사용하여 해당 실험의 모든 작업을 검색할 수 있습니다.

    ```python
   mlflow.search_runs(exp.experiment_id)
    ```

1. 작업 실행 및 출력을 보다 쉽게 비교하려면 검색을 구성하여 결과를 정렬할 수 있습니다. 예를 들어 다음 셀은 결과를 `start_time`순서대로 정렬하고 최대 `2` 결과만 표시합니다. 

    ```python
   mlflow.search_runs(exp.experiment_id, order_by=["start_time DESC"], max_results=2)
    ```

1. 마지막으로 여러 모델의 평가 메트릭을 나란히 그리면 모델을 쉽게 비교할 수 있습니다.

    ```python
   import matplotlib.pyplot as plt
   
   df_results = mlflow.search_runs(exp.experiment_id, order_by=["start_time DESC"], max_results=2)[["metrics.training_accuracy_score", "params.estimator"]]
   
   fig, ax = plt.subplots()
   ax.bar(df_results["params.estimator"], df_results["metrics.training_accuracy_score"])
   ax.set_xlabel("Estimator")
   ax.set_ylabel("Accuracy")
   ax.set_title("Accuracy by Estimator")
   for i, v in enumerate(df_results["metrics.training_accuracy_score"]):
       ax.text(i, v, str(round(v, 2)), ha='center', va='bottom', fontweight='bold')
   plt.show()
    ```

    출력은 다음 이미지와 유사해야 합니다.

    ![표시된 평가 메트릭의 스크린샷.](./Images/plotted-metrics.png)

## 실험 살펴보기

Microsoft Fabric은 모든 실험을 추적하고 시각적으로 탐색할 수 있도록 합니다.

1. 왼쪽 창에서 작업 영역으로 이동합니다.
1. `experiment-churn` 목록에서 실험을 선택합니다.

    > **팁:** 기록된 실험 실행이 표시되지 않으면 페이지를 새로 고칩니다.

1. **보기** 탭을 선택합니다.
1. 실행 목록을** 선택합니다**. 
1. 각 상자를 검사 두 개의 최신 실행을 선택합니다.
    따라서 메트릭 비교 창에서 두 개의 마지막 실행이 **서로 비교** 됩니다. 기본적으로 메트릭은 실행 이름으로 그려집니다. 
1. **각 실행에 대한 정확도를 시각화하는 그래프의 &#128393;**(편집) 단추를 선택합니다. 
1. **시각화 유형을** .로 `bar`변경합니다. 
1. X축**을 **.로 변경합니다`estimator`. 
1. 바꾸기**를 선택하고 **새 그래프를 탐색합니다.

기록된 추정기당 정확도를 그려 더 나은 모델을 생성한 알고리즘을 검토할 수 있습니다.

## 모델 저장

실험 실행에서 학습한 기계 학습 모델을 비교한 후 가장 성능이 좋은 모델을 선택할 수 있습니다. 최상의 성능을 발휘하는 모델을 사용하려면 모델을 저장하고 이를 사용하여 예측을 생성합니다.

1. 실험 개요에서 보기** 탭이 **선택되어 있는지 확인합니다.
1. 실행 세부 정보를** 선택합니다**.
1. 가장 높은 정확도로 실행을 선택합니다. 
1. 모델**로 저장 상자에서 저장**을 **선택합니다**.
1. 새로 연 팝업 창에서 새 모델** 만들기를 선택합니다**.
1. 모델 `model-churn`이름을 지정하고 만들기**를 선택합니다**. 
1. 모델을** 만들 때 화면 오른쪽 위에 표시되는 알림에서 모델 보기를 선택합니다**. 창을 새로 고칠 수도 있습니다. 저장된 모델은 등록된 버전**에서 **연결됩니다. 

모델, 실험 및 실험 실행이 연결되므로 모델을 학습하는 방법을 검토할 수 있습니다. 

## Notebook 저장 및 Spark 세션 종료

이제 모델 학습 및 평가를 마쳤으므로 의미 있는 이름으로 Notebook을 저장하고 Spark 세션을 종료할 수 있습니다.

1. 전자 필기장 메뉴 모음에서 설정** 아이콘을 사용하여 ⚙️ **전자 필기장 설정을 봅니다.
2. **모델을** 학습 및 비교하도록 **Notebook의 이름을** 설정한 다음 설정 창을 닫습니다.
3. Notebook 메뉴에서 세션 중지를 선택하여 **Spark 세션을** 종료합니다.

## 리소스 정리

이 연습에서는 Notebook을 만들고 기계 학습 모델을 학습시켰습니다. Scikit-Learn을 사용하여 모델 및 MLflow를 학습하여 성능을 추적했습니다.

모델 및 실험 탐색을 완료한 경우 이 연습에 대해 만든 작업 영역을 삭제할 수 있습니다.

1. 왼쪽 막대에서 작업 영역의 아이콘을 선택하여 포함된 모든 항목을 봅니다.
2. 도구 모음의 **...** 메뉴에서 **작업 영역 설정**을 선택합니다.
3. **기타** 섹션에서 **이 작업 영역 제거**를 선택합니다.
