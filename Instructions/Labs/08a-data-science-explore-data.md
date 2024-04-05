---
lab:
  title: Microsoft Fabric의 Notebooks를 사용하여 데이터 과학에 대한 데이터 살펴보기
  module: Explore data for data science with notebooks in Microsoft Fabric
---

# Microsoft Fabric의 Notebooks를 사용하여 데이터 과학에 대한 데이터 살펴보기

이 랩에서는 데이터 탐색을 위해 Notebook을 사용합니다. Notebook은 대화형으로 데이터를 탐색하고 분석하는 데 사용하는 강력한 도구입니다. 이 연습에서는 데이터 세트를 탐색하고, 요약 통계를 생성하고, 시각화를 만들어 데이터를 더 잘 이해할 수 있도록 Notebook을 만들고 사용하는 방법을 알아봅니다. 이 랩을 마치면 Notebook을 사용하여 데이터를 탐색하고 분석하는 방법을 확실하게 이해할 수 있습니다.

이 랩을 완료하는 데 약 **30**분이 소요됩니다.

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

모델을 학습하기 위해 *Notebook*을 만들 수 있습니다. Notebook은 여러 언어로 코드를 *실험*으로 작성하고 실행할 수 있는 대화형 환경을 제공합니다.

1. **Synapse 데이터 과학** 홈페이지에서 새 **Notebook**을 만듭니다.

    몇 초 후에 단일 *셀*이 포함된 새 Notebook이 열립니다. Notebook은 *코드* 또는 *markdown*(서식이 지정된 텍스트)을 포함할 수 있는 하나 이상의 셀로 구성됩니다.

1. 첫 번째 셀(현재 *코드* 셀)을 선택한 다음 오른쪽 상단에 있는 동적 도구 모음에서 **M&#8595;** 단추를 눌러 셀을 *markdown* 셀로 변환합니다.

    셀이 markdown 셀로 변경되면 포함된 텍스트가 렌더링됩니다.

1. 필요한 경우 **&#128393;**(편집) 단추를 사용하여 셀을 편집 모드로 전환한 다음 내용을 삭제하고 다음 텍스트를 입력합니다.

    ```text
   # Perform data exploration for data science

   Use the code in this notebook to perform data exploration for data science.
    ```

## 데이터 프레임에 데이터 로드

이제 데이터를 가져오는 코드를 실행할 준비가 되었습니다. Azure Open Datasets의 [**당뇨병 데이터 세트**](https://learn.microsoft.com/azure/open-datasets/dataset-diabetes?tabs=azureml-opendatasets?azure-portal=true)를 작업합니다. 데이터를 로드한 후에는 데이터를 Pandas 데이터 프레임으로 변환합니다. 이는 데이터를 행과 열로 작업하는 일반적인 구조입니다.

1. Notebook에서 최신 셀 아래의 **+ 코드** 아이콘을 사용하여 Notebook에 새 코드 셀을 추가합니다.

    > **팁**: **+ 코드** 아이콘을 보려면 마우스를 현재 셀의 출력 바로 아래 왼쪽으로 이동합니다. 아니면 메뉴 모음의 **편집** 탭에서 **+ 코드 셀 추가**를 선택합니다.

1. 데이터 프레임에 데이터 세트를 로드하려면 다음 코드를 입력합니다.

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

    출력은 당뇨병 데이터 세트의 행과 열을 보여줍니다. 데이터는 당뇨병 환자의 나이, 성, 체질량 지수, 평균 혈압 및 여섯 가지 혈액 혈청 측정이라는 10개의 기준 변수와 관심 반응(기준선 1년 후 질병 진행을 정량적으로 측정한 값)으로 구성되며, 이는 **Y**로 표시됩니다.

1. 데이터는 Spark 데이터 프레임으로 로드됩니다. Scikit-learn은 입력 데이터 세트가 Pandas 데이터 프레임일 것으로 예상합니다. 아래 코드를 실행하여 데이터 세트를 Pandas 데이터 프레임으로 변환합니다.

    ```python
   df = df.toPandas()
   df.head()
    ```

## 데이터 모양 확인

이제 데이터를 로드했으므로 행과 열의 수, 데이터 형식, 누락 값과 같은 데이터 세트의 구조를 확인할 수 있습니다.

1. 셀 출력 아래의 **+ 코드** 아이콘을 사용하여 Notebook에 새 코드 셀을 추가하고 그 안에 다음 코드를 입력합니다.

    ```python
   # Display the number of rows and columns in the dataset
   print("Number of rows:", df.shape[0])
   print("Number of columns:", df.shape[1])

   # Display the data types of each column
   print("\nData types of columns:")
   print(df.dtypes)
    ```

    데이터 세트에는 **행 442개**와 **열 11개**가 포함됩니다. 이는 데이터 세트에 442개의 샘플과 11개의 기능 또는 변수가 있다는 뜻입니다. **SEX** 변수는 범주 또는 문자열 데이터를 포함할 가능성이 높습니다.

## 누락된 데이터 확인

1. 셀 출력 아래의 **+ 코드** 아이콘을 사용하여 Notebook에 새 코드 셀을 추가하고 그 안에 다음 코드를 입력합니다.

    ```python
   missing_values = df.isnull().sum()
   print("\nMissing values per column:")
   print(missing_values)
    ```

    코드는 누락된 값을 확인합니다. 데이터 세트에 누락된 데이터가 없는지 확인합니다.

## 숫자 변수에 대한 기술 통계 생성

이제 숫자 변수의 분포를 이해하기 위한 기술 통계를 생성하겠습니다.

1. 셀 출력 아래의 **+ 코드** 아이콘을 사용하여 Notebook에 새 코드 셀을 추가하고 다음 코드를 입력합니다.

    ```python
   df.describe()
    ```

    평균 연령은 약 48.5세이며 표준 편차는 13.1세입니다. 가장 어린 사람은 19세이고 최고령자는 79세입니다. 평균 BMI는 약 26.4로, [WHO 표준](https://www.who.int/health-topics/obesity#tab=tab_1)에 따라 **과체중** 범주에 속합니다. 최소 BMI는 18이고 최댓값은 42.2입니다.

## 데이터 분포 그리기

BMI 기능을 확인하고, 그 특성을 더 잘 이해할 수 있도록 분포를 그림으로 나타내겠습니다.

1. Notebook에 새 코드 셀을 추가합니다. 그런 다음 이 셀에 다음 코드를 입력하고 실행합니다.

    ```python
   import matplotlib.pyplot as plt
   import seaborn as sns
   import numpy as np
    
   # Calculate the mean, median of the BMI variable
   mean = df['BMI'].mean()
   median = df['BMI'].median()
   
   # Histogram of the BMI variable
   plt.figure(figsize=(8, 6))
   plt.hist(df['BMI'], bins=20, color='skyblue', edgecolor='black')
   plt.title('BMI Distribution')
   plt.xlabel('BMI')
   plt.ylabel('Frequency')
    
   # Add lines for the mean and median
   plt.axvline(mean, color='red', linestyle='dashed', linewidth=2, label='Mean')
   plt.axvline(median, color='green', linestyle='dashed', linewidth=2, label='Median')
    
   # Add a legend
   plt.legend()
   plt.show()
    ```

    이 그래프에서 데이터 세트의 BMI 범위와 분포를 관찰할 수 있습니다. 예를 들어 대부분의 BMI는 23.2~29.2에 속하며 데이터가 오른쪽으로 치우쳐 있습니다.

## 다변량 분석 수행

데이터 내의 패턴과 관계를 밝히기 위해 산점도와 상자 그림과 같은 시각화를 생성해 보겠습니다.

1. 셀 출력 아래의 **+ 코드** 아이콘을 사용하여 Notebook에 새 코드 셀을 추가하고 다음 코드를 입력합니다.

    ```python
   import matplotlib.pyplot as plt
   import seaborn as sns

   # Scatter plot of BMI vs. Target variable
   plt.figure(figsize=(8, 6))
   sns.scatterplot(x='BMI', y='Y', data=df)
   plt.title('BMI vs. Target variable')
   plt.xlabel('BMI')
   plt.ylabel('Target')
   plt.show()
    ```

    BMI가 증가함에 따라 대상 변수도 증가하여 이 두 변수 간 양의 선형 관계가 나타나는 것을 볼 수 있습니다.

1. Notebook에 새 코드 셀을 추가합니다. 그런 다음 이 셀에 다음 코드를 입력하고 실행합니다.

    ```python
   import seaborn as sns
   import matplotlib.pyplot as plt
    
   fig, ax = plt.subplots(figsize=(7, 5))
    
   # Replace numeric values with labels
   df['SEX'] = df['SEX'].replace({1: 'Male', 2: 'Female'})
    
   sns.boxplot(x='SEX', y='BP', data=df, ax=ax)
   ax.set_title('Blood pressure across Gender')
   plt.tight_layout()
   plt.show()
    ```

    이러한 관찰은 남성 환자와 여성 환자의 혈압 프로필에 차이가 있음을 시사합니다. 평균적으로 여성 환자는 남성 환자보다 혈압이 높습니다.

1. 데이터를 집계하면 더 쉽게 시각화하고 분석하도록 관리할 수 있습니다. Notebook에 새 코드 셀을 추가합니다. 그런 다음 이 셀에 다음 코드를 입력하고 실행합니다.

    ```python
   import matplotlib.pyplot as plt
   import seaborn as sns
    
   # Calculate average BP and BMI by SEX
   avg_values = df.groupby('SEX')[['BP', 'BMI']].mean()
    
   # Bar chart of the average BP and BMI by SEX
   ax = avg_values.plot(kind='bar', figsize=(15, 6), edgecolor='black')
    
   # Add title and labels
   plt.title('Avg. Blood Pressure and BMI by Gender')
   plt.xlabel('Gender')
   plt.ylabel('Average')
    
   # Display actual numbers on the bar chart
   for p in ax.patches:
       ax.annotate(format(p.get_height(), '.2f'), 
                   (p.get_x() + p.get_width() / 2., p.get_height()), 
                   ha = 'center', va = 'center', 
                   xytext = (0, 10), 
                   textcoords = 'offset points')
    
   plt.show()
    ```

    이 그래프는 평균 혈압이 남성 환자보다 여성 환자가 더 높다는 것을 보여줍니다. 또한 평균 체질량 지수(BMI)가 남성보다 여성이 약간 더 높은 것으로 나타납니다.

1. Notebook에 새 코드 셀을 추가합니다. 그런 다음 이 셀에 다음 코드를 입력하고 실행합니다.

    ```python
   import matplotlib.pyplot as plt
   import seaborn as sns
    
   plt.figure(figsize=(10, 6))
   sns.lineplot(x='AGE', y='BMI', data=df, errorbar=None)
   plt.title('BMI over Age')
   plt.xlabel('Age')
   plt.ylabel('BMI')
   plt.show()
    ```

    19~30세 연령대의 평균 BMI 값이 가장 낮고, 65~79세 연령대의 평균 BMI가 가장 높습니다. 또한 대부분 연령대의 평균 BMI가 과체중 범위에 속한다는 것을 알 수 있습니다.

## 상관 관계 분석

서로 다른 기능 간의 상관 관계를 따져 관계의 양상과 종속성을 파악해 보겠습니다.

1. 셀 출력 아래의 **+ 코드** 아이콘을 사용하여 Notebook에 새 코드 셀을 추가하고 다음 코드를 입력합니다.

    ```python
   df.corr(numeric_only=True)
    ```

1. 열 지도는 변수 쌍 간 관계의 강도와 방향을 빠르게 시각화하는 데 유용한 도구입니다. 열 지도는 강한 양의 상관 관계 또는 강한 음의 상관 관계를 강조 표시하고, 상관 관계가 없는 쌍을 식별할 수 있습니다. 열 지도를 만들려면 Notebook에 새 코드 셀을 추가하고 다음 코드를 입력합니다.

    ```python
   plt.figure(figsize=(15, 7))
   sns.heatmap(df.corr(numeric_only=True), annot=True, vmin=-1, vmax=1, cmap="Blues")
    ```

    S1 변수와 S2 변수는 **0.89**의 강한 양의 상관 관계를 보이며, 같은 방향으로 움직인다는 것을 나타냅니다. S1이 증가하면 S2도 증가하는 경향이 있으며, 그 반대도 마찬가지입니다. 또한 S3과 S4는 **-0.73**의 강한 음의 상관 관계가 있습니다. 이는 S3이 증가하면 S4는 감소하는 경향이 있음을 뜻합니다.

## Notebook 저장 및 Spark 세션 종료

이제 데이터 탐색을 마쳤으므로 Notebook을 의미가 있는 이름으로 저장하고 Spark 세션을 종료할 수 있습니다.

1. 전자 필기장 메뉴 모음에서 ⚙️ **설정** 아이콘을 사용하여 전자 필기장 설정을 봅니다.
2. Notebook **이름**을 **데이터 과학용 데이터 탐색**으로 설정한 다음 설정 창을 닫습니다.
3. Notebook 메뉴에서 **세션 중지**를 선택하여 Spark 세션을 종료합니다.

## 리소스 정리

이 연습에서는 데이터 탐색을 위해 Notebook을 만들고 사용했습니다. 또한 데이터의 패턴과 관계를 더 잘 이해하기 위해 요약 통계를 계산하고 시각화를 만드는 코드를 실행해 보았습니다.

모델과 실험을 다 살펴보았다면 이 연습에서 만든 작업 영역을 삭제해도 됩니다.

1. 왼쪽 막대에서 작업 영역의 아이콘을 선택하여 포함된 모든 항목을 봅니다.
2. 도구 모음의 **...** 메뉴에서 **작업 영역 설정**을 선택합니다.
3. **기타** 섹션에서 **이 작업 영역 제거**를 선택합니다.
