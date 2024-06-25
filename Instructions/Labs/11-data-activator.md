---
lab:
  title: Fabric에서 Data Activator 사용
  module: Get started with Data Activator in Microsoft Fabric
---

# Fabric에서 Data Activator 사용

Microsoft Fabric의 Data Activator는 데이터에서 발생하는 작업에 따라 작업을 수행합니다. Data Activator를 사용하면 데이터를 모니터링하고 데이터 변경 내용에 반응하는 트리거를 만들 수 있습니다.

이 랩을 완료하는 데 약 **30**분이 소요됩니다.

> **참고**: 이 연습을 완료하려면 [Microsoft Fabric 평가판](https://learn.microsoft.com/fabric/get-started/fabric-trial)이 필요합니다.

## 작업 영역 만들기

패브릭에서 데이터를 사용하기 전에 패브릭 평가판을 사용하도록 설정된 작업 영역을 만듭니다.

1. [Microsoft Fabric 홈페이지](https://app.fabric.microsoft.com)에서 **Data Activator**를 선택합니다.
1. 왼쪽 메뉴 모음에서 **작업 영역**을 선택합니다(아이콘은 와 유사함).
1. Fabric 용량이 포함된 라이선스 모드(*평가판*, *프리미엄* 또는 *Fabric*)를 선택하여 원하는 이름으로 새 작업 영역을 만듭니다.
1. 새 작업 영역이 열리면 비어 있어야 합니다.

    ![Fabric의 빈 작업 영역 스크린샷.](./Images/new-workspace.png)

이 랩에서는 Fabric의 Data Activator를 사용하여 *반사*를 만듭니다. Data Activator는 Data Activator의 기능을 탐색하는 데 사용할 수 있는 샘플 데이터 세트를 편리하게 제공합니다. 이 샘플 데이터를 사용하여 일부 실시간 데이터를 분석하고 조건이 충족되면 이메일을 보내는 트리거를 만드는 *반사*를 만듭니다.

> **참고**: Data Activator 샘플 프로세스는 백그라운드에서 임의의 데이터를 생성합니다. 조건과 필터가 복잡할수록 이를 트리거하는 데 더 많은 시간이 걸립니다. 그래프에 데이터가 표시되지 않으면 몇 분 정도 기다린 후 페이지를 새로 고칩니다. 즉, 랩을 계속하기 위해 데이터가 그래프에 표시될 때까지 기다릴 필요가 없습니다.

## 시나리오

이 시나리오에서는 다양한 제품을 판매하고 배송하는 회사의 데이터 분석가라고 가정하겠습니다.  Redmond 시로의 모든 배송 및 판매 데이터에 대한 책임이 있습니다. 배달 예정인 패키지를 모니터링하는 반사를 만들려고 합니다. 배송하는 제품 범주 중 하나는 운송 중 특정 온도에서 냉장 보관해야 하는 의료 처방약입니다. 처방약이 포함된 패키지의 온도가 특정 임계값보다 높거나 낮을 경우 배송 부서에 이메일을 보내는 반사를 만들려고 합니다. 이상적인 온도는 33도에서 41도 사이여야 합니다. 반사 이벤트에는 이미 유사한 트리거가 포함되어 있으므로 Redmond 시로 배송되는 패키지용으로 특별히 하나를 만듭니다. 그럼 시작하겠습니다.

## Reflex 만들기

1. 오른쪽 하단에 있는 아이콘이 Data Activator를 반영하는지 확인하여 Data Activator 홈 화면에 있는지 확인합니다. **반사(미리 보기)** 단추를 선택하여 새로운 반사를 만들겠습니다.

    ![Data Activator 홈 화면의 스크린샷.](./Images/data-activator-home-screen.png)

1. 실제 프로덕션 환경에서는 자체 데이터를 사용합니다. 하지만 이 랩에서는 Data Activator에서 제공하는 샘플 데이터를 사용합니다. **샘플 데이터 사용** 단추를 선택하여 반사 만들기를 완료합니다.

    ![Data Activator 데이터 가져오기 화면의 스크린샷.](./Images/data-activator-get-started.png)

1. 기본적으로 Data Activator는 *Reflex YYYY-MM-DD hh:mm:ss*라는 이름으로 반사를 만듭니다. 작업 영역에 반사가 여러 개 있을 수 있으므로 기본 반사의 이름을 설명이 포함된 상세한 이름으로 변경해야 합니다. 이 예에서는 왼쪽 상단에 있는 현재 반사 이름 옆에 있는 풀다운을 선택하고 이름을 ***Contoso Shipping Reflex***로 변경합니다.

    ![Data Activator 반사 홈 화면의 스크린샷.](./Images/data-activator-reflex-home-screen.png)

이제 반사가 만들어졌으며 여기에 트리거와 작업을 추가할 수 있습니다.

## 반사 홈 화면에 익숙해지기

반사의 홈 화면은 *디자인* 모드와 *데이터* 모드의 두 섹션으로 구분됩니다. 화면 왼쪽 하단의 해당 탭을 선택하여 모드를 선택할 수 있습니다.  *디자인* 모드 탭에서는 트리거, 속성 및 이벤트로 개체를 정의합니다. *데이터* 모드 탭에서는 데이터 원본을 추가하고 반사가 처리하는 데이터를 볼 수 있습니다. 반사를 만들 때 기본적으로 열리는 *디자인* 모드 탭을 살펴보겠습니다.

### 디자인 모드

현재 *디자인* 모드가 아닌 경우 화면 왼쪽 하단에 있는 **디자인** 탭을 선택합니다.

![Data Activator 반사 디자인 모드의 스크린샷.](./Images/data-activator-design-tab.png)

*디자인* 모드에 익숙해지려면 화면의 다양한 섹션, 트리거, 속성 및 이벤트를 선택합니다. 다음 섹션에서 각 섹션을 더 자세히 다룹니다.

### 데이터 모드

현재 *데이터* 모드가 아닌 경우 화면 왼쪽 하단에 있는 **데이터** 탭을 선택합니다. 실제 사례에서는 여기에 EventStreams 및 Power BI 시각적 개체의 고유한 데이터 원본을 추가합니다. 이 랩에서는 Data Activator에서 제공하는 샘플 데이터를 사용합니다. 이 샘플은 패키지 배달 상태를 모니터링하는 3개의 EventStream으로 이미 설정되어 있습니다.

![Data Activator 반사 데이터 모드의 스크린샷.](./Images/data-activator-data-tab.png)

다양한 이벤트를 각각 선택하고 스트림에서 사용되는 데이터를 관찰합니다.

![Data Activator 반사 데이터 모드 이벤트의 스크린샷.](./Images/data-activator-get-data-tab-event-2.png)

이제 반사에 트리거를 추가할 차례입니다. 먼저 새 개체를 만들겠습니다.

##  개체 만들기

실제 시나리오에서는 Data Activator 샘플에 이미 *Package*라는 개체가 포함되어 있으므로 이 반사를 위한 새 개체를 만들 필요가 없을 수도 있습니다. 하지만 이 랩에서는 개체를 만드는 방법을 보여 주기 위해 새 개체를 만듭니다. *Redmond Packages*라는 새 개체를 만들겠습니다.

1. 현재 *데이터* 모드가 아닌 경우 화면 왼쪽 하단에 있는 **데이터** 탭을 선택합니다.

1. ***운송 중 패키지*** 이벤트를 선택합니다. *PackageId*, *Temperature*, *ColdChainType*, *City* 및 *SpecialCare* 열의 값을 주의 깊게 살펴봅니다. 이 열을 사용하여 트리거를 만듭니다.

1. 오른쪽에 *데이터 할당* 대화 상자가 아직 열려 있지 않으면 화면 오른쪽에 있는 **데이터 할당** 단추를 선택합니다.

    ![Data Activator 리플렉스 데이터 모드의 데이터 할당 단추 스크린샷.](./Images/data-activator-data-tab-assign-data-button.png)

1. *데이터 할당* 대화 상자에서 ***새 개체에 할당*** 탭을 선택하고 다음 값을 입력합니다.

    - **개체 이름**: *Redmond Packages*
    - **키 열 할당**: *PackageId*
    - **속성 할당**: *City, ColdChainType, SpecialCare, Temperature*

    ![Data Activator 리플렉스 데이터 모드의 데이터 할당 대화 상자 스크린샷.](./Images/data-activator-data-tab-assign-data.png)

1. **저장**을 선택한 다음 **저장하고 디자인 모드로 이동**을 선택합니다.

1. 이제 *디자인* 모드로 돌아왔습니다. ***Redmond Packages***라는 새 개체가 추가되었습니다. 이 새 개체를 선택하고 *이벤트*를 확장한 다음 **운송 중인 패키지** 이벤트를 선택합니다.

    ![새로운 개체가 포함된 Data Activator 반사 디자인 모드의 스크린샷.](./Images/data-activator-design-tab-new-object.png)

트리거를 만들 시간입니다.

## 트리거 만들기

트리거에서 수행하려는 작업을 검토하겠습니다. *처방약이 포함된 패키지의 온도가 특정 임계값보다 높거나 낮을 경우 배송 부서에 이메일을 보내는 반사를 만들려고 합니다. 이상적인 온도는 33도에서 41도 사이입니다. 반사 이벤트에는 이미 유사한 트리거가 포함되어 있으므로 Redmond 시로 배송되는 패키지용으로 특별히 하나를 만듭니다*.

1. **Redmond Packages** 개체의 *운송 중인 패키지* 이벤트 내 상단 메뉴에서 **새 트리거** 단추를 선택합니다. 기본 이름이 *제목 없음*인 새 트리거가 만들어집니다. 트리거를 더 잘 정의하려면 이름을 ***의약품 온도가 범위를 벗어남***으로 변경합니다.

    ![Data Activator 반사 디자인 새 트리거 만들기 스크린샷.](./Images/data-activator-trigger-new.png)

1. 반사를 트리거하는 속성 또는 이벤트 열을 선택할 시간입니다. 개체를 만들 때 여러 속성을 만들었으므로 **기존 속성** 단추를 선택하고 ***Temperature*** 속성을 선택합니다. 

    ![Data Activator 반사 디자인 속성 선택 스크린샷.](./Images/data-activator-trigger-select-property.png)

    이 속성을 선택하면 샘플 과거 온도 값이 포함된 그래프가 반환됩니다.

    ![과거 값의 Data Activator 속성 그래프 스크린샷.](./Images/data-activator-trigger-property-sample-graph.png)

1. 이제 이 속성에서 트리거하려는 조건 형식을 결정해야 합니다. 이 경우 온도가 41도 이상 또는 33도 이하일 때 반사를 트리거하려고 합니다. 숫자 범위를 찾고 있으므로 **숫자** 단추를 선택하고 **범위 종료** 조건을 선택합니다.

    ![Data Activator 반사 디자인 조건 형식 선택 스크린샷.](./Images/data-activator-trigger-select-condition-type.png)

1. 이제 조건에 대한 값을 입력해야 합니다. 범위 값으로 ***33*** 및 ***41***을 입력합니다. *숫자 범위 종료* 조건을 선택했으므로 온도가 *33*도 미만이거나 *41*도 이상일 때 트리거가 실행됩니다.

    ![Data Activator 반사 디자인 조건 값 입력 스크린샷.](./Images/data-activator-trigger-select-condition-define.png)

1. 지금까지 트리거를 실행하려는 속성과 조건을 정의했지만 여전히 필요한 모든 매개 변수가 포함되어 있지 않습니다. **Redmond** *시*와 *특별 진료* 유형의 **의약품**에 대해서만 트리거가 실행되는지 확인해야 합니다. 해당 조건에 대해 몇 가지 필터를 추가하겠습니다.  **필터 추가** 단추를 선택하고 속성을 ***City***로 설정하고 관계를 ***같음***으로 설정한 다음 값으로 ***Redmond***를 입력합니다. 그런 다음 ***SpecialCare*** 속성이 있는 새 필터를 추가하고 이를 ***같음***으로 설정한 다음 값으로 ***Medicine***을 입력합니다.

    ![Data Activator 반사 디자인 추가 필터의 스크린샷.](./Images/data-activator-trigger-select-condition-add-filter.png)

1. 의약품이 냉장보관되는지 확인하기 위해 필터를 하나 더 추가하겠습니다. **필터 추가** 단추를 선택하고 ***ColdChainType*** 속성을 설정한 후 ***같음***으로 설정하고 값으로 ***Refrigerated***를 입력합니다.

    ![Data Activator 반사 디자인 추가 필터의 스크린샷.](./Images/data-activator-trigger-select-condition-add-filter-additional.png)

1. 거의 완료되었습니다! 트리거가 실행될 때 수행할 작업을 정의하기만 하면 됩니다. 이 경우 배송 부서에 이메일을 보내려고 합니다. **이메일** 단추를 선택합니다.

    ![Data Activator 작업 추가 스크린샷.](./Images/data-activator-trigger-select-action.png)

1. 이메일 작업에 다음 값을 입력합니다.

    - **전송 위치**: 현재 사용자 계정은 기본적으로 선택되어야 하며 이는 이 랩에 적합합니다.
    - **제목**: *허용 온도 범위를 벗어난 Redmond 의료 패키지*
    - **헤드라인**: *온도가 너무 높거나 너무 낮음*
    - **추가 정보**: 확인란 목록에서 *Temperature* 속성을 선택합니다.

    ![Data Activator 작업 정의 스크린샷.](./Images/data-activator-trigger-define-action.png)

1. 상단 메뉴에서 **저장**을 선택한 다음 **시작**을 선택합니다.

이제 Data Activator에서 트리거를 만들고 시작했습니다.

## 트리거 업데이트 및 중지

이 트리거의 유일한 문제는 트리거가 온도가 포함된 이메일을 보내는 동안 패키지의 *PackageId*를 보내지 않았다는 것입니다. 계속해서 *PackageId*를 포함하도록 트리거를 업데이트하겠습니다.

1. **Redmond Packages** 개체에서 **운송 중인 패키지** 이벤트를 선택하고 상단 메뉴에서 **새 속성**을 선택합니다.

    ![Data Activator 개체 이벤트 선택 스크린샷.](./Images/data-activator-trigger-select-event.png)

1. *운송 중인 패키지* 이벤트에서 열을 선택하여 **PackageId** 속성을 추가하겠습니다. 속성 이름을 *Untitled*에서 *PackageId*로 변경해야 합니다.

    ![Data Activator 속성 만들기 스크린샷.](./Images/data-activator-trigger-create-new-property.png)

1. 트리거 작업을 업데이트하겠습니다. **의약품 온도가 범위를 벗어남** 트리거를 선택하고 하단의 **Act** 섹션으로 스크롤한 다음 **추가 정보**를 선택하고 **PackageId** 속성을 추가합니다. 아직 **저장** 단추를 선택하지 마세요.

    ![Data Activator 트리거할 속성 추가 스크린샷.](./Images/data-activator-trigger-add-property-existing-trigger.png)

1. 트리거를 업데이트했으므로 올바른 작업은 트리거를 업데이트하고 저장하지 않아야 합니다. 그러나 이 랩에서는 반대로 수행하고 **업데이트** 단추 대신 **저장** 단추를 선택하여 어떤 일이 일어나는지 확인합니다. *업데이트* 단추를 선택해야 하는 이유는 트리거를 *업데이트*하도록 선택하면 트리거가 저장되고 현재 실행 중인 트리거가 새 조건으로 업데이트되기 때문입니다. *저장* 단추만 선택하면 현재 실행 중인 트리거는 트리거 업데이트를 선택할 때까지 새 조건을 적용하지 않습니다. 계속해서 **저장** 단추를 선택하겠습니다.

1. *업데이트* 대신 *저장*을 선택했기 때문에 *사용 가능한 속성 업데이트가 있습니다. 트리거에 최신 변경 내용이 적용되도록 지금 업데이트합니다.* 화면 상단에 나타납니다. 메시지에는 추가로 *업데이트* 단추가 있습니다. 계속해서 **업데이트** 단추를 선택하겠습니다.

    ![Data Activator 트리거 업데이트 스크린샷.](./Images/data-activator-trigger-updated.png)

1. 상단 메뉴에서 **중지** 단추를 선택하여 트리거를 중지합니다.

## 리소스 정리

이 연습에서는 Data Activator의 트리거를 사용하여 반사를 만들었습니다. 이제 Data Activator 인터페이스와 반사 및 개체, 트리거 및 속성을 만드는 방법에 익숙해졌을 것입니다.

Data Activator 반사 탐색을 마쳤으면 이 연습을 위해 만든 작업 영역을 삭제할 수 있습니다.

1. 왼쪽 막대에서 작업 영역의 아이콘을 선택하여 포함된 모든 항목을 봅니다.
2. 도구 모음의 **...** 메뉴에서 **작업 영역 설정**을 선택합니다.
3. **일반** 섹션에서 **이 작업 영역 제거**를 선택합니다.
