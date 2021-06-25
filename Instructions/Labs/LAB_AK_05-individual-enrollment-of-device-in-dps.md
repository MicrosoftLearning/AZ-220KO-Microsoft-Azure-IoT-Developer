---
lab:
    title: '랩 05: DPS에서 디바이스 개별 등록'
    module: '모듈 3: 대규모로 디바이스 프로비저닝'
---

# DPS에서 디바이스 개별 등록

## 랩 시나리오

Contoso 관리부에서는 기존 자산 모니터링 및 추적 솔루션 업데이트를 추진하고 있습니다. 이 업데이트를 완료하면 Contoso에서는 IoT 디바이스를 사용하여 현재 시스템에서 요구되는 수동 데이터 입력 작업을 줄이고 배송 프로세스 중에 고급 모니터링을 제공하는 IoT 디바이스를 사용하게 될 예정입니다. 이 솔루션은 배송 컨테이너 적재 시 IoT 디바이스를 프로비전하고 컨테이너가 목적지에 도착하면 디바이스 프로비전을 해제하는 기능을 사용합니다. 프로비전 요구 사항을 관리하는 최적 옵션은 IoT Hub DPS(Device Provisioning Service)로 보입니다.

제안된 시스템은 운송 중 선적 컨테이너의 위치, 온도, 압력을 추적하기 위해 통합 센서가 있는 IoT 디바이스를 사용합니다. IoT 디바이스는 Contoso가 치즈를 운반하는 데 사용하는 기존 선적 컨테이너 내에 배치되며 차량 제공 WiFi를 사용하여 Azure IoT Hub에 연결됩니다. 새 시스템은 제품 환경에 대한 지속적인 모니터링을 제공하고 문제가 감지될 때 다양한 알림 시나리오를 가능하게 합니다. IoT Hub로 원격 분석이 전송되는 속도는 구성 가능해야 합니다.

Contoso의 치즈 포장 시설에서 빈 컨테이너가 시스템에 들어오면 새로운 IoT 디바이스를 장착한 다음 포장된 치즈 제품으로 적재합니다. IoT 디바이스는 DPS를 사용하여 IoT Hub에 자동 프로비전됩니다. 컨테이너가 목적지에 도착하면 IoT 디바이스가 검색된 다음 완전히 프로비전 해제(등록 취소)되어야 합니다. 복구된 디바이스는 재활용된 후 이후 배송에서 동일한 자동 프로비전 프로세스에 따라 재사용됩니다.

DPS를 사용하여 디바이스 프로비저닝 및 프로비전 해제 프로세스의 유효성을 검사해야 하는 임무를 받았습니다. 초기 테스트 단계에서는 개별 등록 방법을 사용합니다.

다음 리소스가 만들어집니다.

![랩 5 아키텍처](media/LAB_AK_05-architecture.png)

## 랩 내용

이 랩에서는 먼저 랩 필수 구성 요소를 검토한 후 필요에 따라 스크립트를 실행하여 Azure 구독에 필요한 리소스가 포함되어 있는지를 확인합니다. 그런 다음 DPS에서 새 개별 등록을 작성합니다. 이 등록에서는 대칭 키 증명을 사용하며, 디바이스용 초기 디바이스 트윈 상태(원격 분석 전송 속도)를 지정합니다. 디바이스 등록을 저장한 후에는 등록으로 돌아가 디바이스 증명에 필요한 자동 생성된 기본 키와 보조 키를 가져옵니다. 그 후에는 시뮬레이션된 디바이스를 만들어 IoT Hub에 정상적으로 연결되는지, 그리고 해당 디바이스가 초기 디바이스 트윈 속성을 올바르게 적용하는지 확인합니다. 마지막으로 프로비전 해제 프로세스를 완료합니다. 이 프로세스에서는 디바이스를 등록 취소(DPS와 IoT Hub에서 각각 등록 취소)하여 솔루션에서 디바이스를 안전하게 제거합니다. 랩에 포함된 연습은 다음과 같습니다.

* 랩 필수 구성 요소 확인
* DPS에서 새 개별 등록(대칭 키) 만들기
* 시뮬레이션된 디바이스 구성
* 시뮬레이션된 디바이스 테스트
* 디바이스 프로비전 해제

## 랩 지침

### 연습 1: 랩 필수 구성 요소 확인

이 랩은 다음 Azure 리소스를 사용할 수 있다고 가정합니다.

| 리소스 유형 | 리소스 이름 |
| :-- | :-- |
| 리소스 그룹 | rg-az220 |
| IoT Hub | iot-az220-training-{your-id} |
| Device Provisioning Service | dps-az220-training-{your-id} |

이러한 리소스를 사용할 수 없는 경우 연습 2로 이동하기 전에 아래 설명에 따라 **lab05-setup.azcli** 스크립트를 실행해야 합니다. 스크립트 파일은 개발자 환경 구성(랩 3)의 일부로 로컬로 복제한 GitHub 리포지토리에 포함됩니다.

**lab05-setup.azcli** 스크립트는 **bash** 셸 환경에서 실행되도록 작성됩니다. 이는 Azure Cloud Shell에서 실행할 수 있는 가장 쉬운 방법입니다.

1. 브라우저를 사용하여 [Azure Cloud Shell](https://shell.azure.com/)을 열고 이 과정에 사용 중인 Azure 구독으로 로그인합니다.

    Cloud Shell에 대한 스토리지 설정 관련 메시지가 표시되면 기본값을 수락합니다.

1. Cloud Shell에서 **Bash**를 사용하고 있는지 확인합니다.

    Azure Cloud Shell 페이지의 왼쪽 상단에 있는 드롭다운은 환경을 선택하는 데 사용됩니다. 선택한 드롭다운 값이 **Bash**인지 확인합니다.

1. Cloud Shell 도구 모음에서 **파일 업로드/다운로드**(오른쪽의 네 번째 단추)를 클릭합니다.

1. 드롭다운에서 **업로드**를 클릭합니다.

1. 파일 선택 대화 상자에서 개발 환경을 구성할 때 다운로드한 GitHub 랩 파일의 폴더 위치로 이동합니다.

    _랩 3: 개발 환경 설정_, ZIP 파일을 다운로드하고 콘텐츠를 로컬로 추출하여 랩 리소스를 포함하는 GitHub 리포지토리를 복제했습니다. 추출된 폴더 구조에는 다음 폴더 경로가 포함됩니다.

    * 모든 파일
      * 랩
          * 05-DPS에서 디바이스 개별 등록
            * 설정

    lab05-setup.azcli 스크립트 파일은 랩 5의 Setup 폴더에 있습니다.

1. **lab05-setup.azcli** 파일을 선택한 다음 **열기**를 클릭합니다.

    파일 업로드가 완료되면 알림이 표시됩니다.

1. 올바른 파일을 업로드했는지 확인하려면 다음 명령을 입력합니다.

    ```bash
    ls
    ```

    `ls` 명령으로 현재 디렉터리의 내용을 나열합니다. lab05-setup.azcli 파일이 나열됩니다.

1. 설치 스크립트가 포함된 이 랩에 대한 디렉터리를 만든 다음 해당 디렉터리로 이동하려면 다음 Bash 명령을 입력합니다.

    ```bash
    mkdir lab5
    mv lab05-setup.azcli lab5
    cd lab5
    ```

    이러한 명령은 이 랩의디렉터리를 만들고 **lab05-setup.azcli** 파일을 해당 디렉터리로 이동한 다음 디렉터리를 변경하여 새 디렉터리를 현재 작업 디렉터리로 만듭니다.

1. **lab05-setup.azcli** 스크립트에 실행 권한이 있는지 확인하려면 다음 명령을 입력합니다.

    ```bash
    chmod +x lab05-setup.azcli
    ```

1. Cloud Shell 도구 모음에서 lab05-setup.azcli 파일에 액세스할 수 있도록 설정하려면 **편집기 열기**(오른쪽에서 두 번째 단추 - **{}**)를 클릭합니다.

1. **파일** 목록에서 lab5 폴더를 확장하고 스크립트 파일을 열려면 **lab5**를 클릭한 다음 **lab05-setup.azcli**를 클릭합니다.

    이제 편집기에서 **lab05-setup.azcli** 파일의 내용을 표시합니다.

1. 편집기에서 `{your-id}` 및 `{your-location}` 변수의 값을 업데이트합니다.

    아래 샘플을 예로 들어 보면, `{your-id}`는 이 과정을 시작할 때 만든 고유 ID(예: **cah191211**)로 설정하고 `{your-location}`는 리소스에 적합한 위치로 설정해야 합니다.

    ```bash
    #!/bin/bash

    # 아래 값을 변경하세요!
    YourID="{your-id}"
    Location="{your-location}"
    ```

    > **참고**:  `{your-location}` 변수는 모든 리소스를 배포하는 지역의 짧은 이름으로 설정해야 합니다. 이 명령을 입력하면 사용 가능한 위치 및 짧은 이름(**이름** 열)의 목록을 볼 수 있습니다.

    ```bash
    az account list-locations -o Table

    DisplayName           Latitude    Longitude    Name
    --------------------  ----------  -----------  ------------------
    East Asia             22.267      114.188      eastasia
    Southeast Asia        1.283       103.833      southeastasia
    Central US            41.5908     -93.6208     centralus
    East US               37.3719     -79.8164     eastus
    East US 2             36.6681     -78.3889     eastus2
    ```

1. 파일의 변경 내용을 저장하고 편집기를 닫으려면 편집기 창 오른쪽 위의 **...** 를 클릭한 다음 **편집기 닫기**를 클릭합니다.

    저장하라는 메시지가 표시된 경우 **저장**을 클릭하면 편집기가 닫힙니다.

    > **참고**:  **CTRL+S**를 사용하여 언제든지 저장할 수 있으며 **CTRL+Q**를 사용하여 편집기를 닫을 수 있습니다.

1. 이 랩에 필요한 리소스를 만들려면 다음 명령을 입력합니다.

    ```bash
    ./lab05-setup.azcli
    ```

    이 작업을 실행하려면 몇 분 정도 걸립니다. 각 단계가 완료될 때 출력이 표시됩니다.

    스크립트가 완료되면 랩으로 계속할 준비가 끝납니다.

### 연습 2: DPS에서 새 개별 등록(대칭 키) 만들기

이 연습에서는 _대칭 키 증명_을 사용하여 DPS(Device Provisioning Service)에 있는 디바이스에 새 개별 등록을 만듭니다. 그리고 등록 내에서 초기 디바이스 상태도 구성합니다. 등록을 저장한 후에는 등록으로 돌아가 등록 저장 시에 작성되는 자동 생성 증명 키를 가져옵니다.

#### 작업 1: 등록 만들기

1. 필요한 경우 Azure 계정 자격 증명을 사용하여 [portal.azure.com](https://portal.azure.com)에 로그인합니다.

    Azure 계정이 두 개 이상인 경우 이 과정에 사용할 구독에 연결된 계정으로 로그인해야 합니다.

1. **AZ-220** 대시보드가 로드되고 리소스 타일이 표시됩니다.

    IoT Hub 및 DPS 리소스가 모두 나열되어 있습니다.

1. **rg-az220** 리소스 그룹 타일에서 **dps-az220-training-{your-id}** 를 클릭합니다.

1. 왼쪽 메뉴의 **설정**에서 **등록 관리**를 클릭합니다.

1. **등록 관리** 창 상단에서 **+ 개별 등록 추가**를 클릭합니다.

1. **등록 추가** 블레이드의 **메커니즘** 드롭다운에서 **대칭 키**를 클릭합니다.

    이렇게 하면 대칭 키 인증을 사용하는 증명 방법이 설정됩니다.

1. 메커니즘 설정 바로 아래에 **키 자동 생성** 옵션이 선택되어 있습니다.

    그러면 DPS는 디바이스 등록을 만들 시 **기본 키**와 **보조 키** 값 모두를 자동으로 생성하도록 설정됩니다. 선택적으로 이 옵션의 선택을 취소하면 사용자 지정 키를 수동으로 입력할 수 있습니다.

    > **참고**: 레코드를 저장하고 나면 기본 키 및 보조 키 값이 생성됩니다. 다음 작업에서 이 레코드로 돌아가 값을 가져온 다음, 이 랩 뒷부분에서 실행할 시뮬레이션된 디바이스 앱 내에서 해당 값을 사용합니다.

1. DPS 내에 디바이스를 등록할 때 사용할 등록 ID를 지정하려면 **등록 ID** 필드에 **sensor-thl-1000**을 입력합니다.

    기본적으로 등록 ID는 디바이스가 등록에서 프로비저닝될 때 IoT Hub 디바이스 ID로 사용됩니다. 이러한 값이 달라야 하는 경우 해당 필드에 필수 IoT Hub 디바이스 ID를 입력할 수 있습니다.

1. **IoT Hub 디바이스 ID** 필드를 비워둡니다.

    이 필드를 비워 두면 IoT Hub가 등록 ID를 디바이스 ID로 사용합니다. 선택할 수 없는 필드에 기본 텍스트 값이 표시되어도 걱정하지 마세요. 이 텍스트는 자리 표시자이며 입력한 값으로 취급되지 않습니다.

1. **IoT Edge 디바이스** 필드를 **거짓**으로 설정된 상태로 둡니다.

    새 디바이스는 Edge 디바이스가 아닙니다. IoT Edge 디바이스로 작업하는 것은 이 과정의 뒷부분에서 설명합니다.

1. **허브에 디바이스를 할당할 방법을 선택하세요** 필드를 **균등 가중 배포**로 둡니다.

    등록과 연결된 IoT Hub가 하나만 있으므로 이 설정은 다소 중요하지 않습니다.  여러 분산 허브가 있는 대규모 환경에서 이 설정은 이 디바이스 등록을 받을 IoT Hub를 선택하는 방법을 제어합니다. 지원되는 할당 정책은 네 가지가 있습니다.

    * **최저 대기 시간**: 디바이스에 대한 대기 시간이 가장 짧은 허브에 기반하여 해당 디바이스가 IoT Hub로 프로비저닝됩니다.
    * **균등 가중 분포(기본값)**: 연결된 IoT Hub에서 디바이스를 프로비저닝하는 가능성이 동일합니다. 기본 설정입니다. 디바이스를 단 하나의 IoT Hub에 프로비전하려는 경우 이 설정을 유지할 수 있습니다. 
    * **등록 목록을 통한 정적 구성**: 등록 목록에 지정된 원하는 IoT Hub는 Device Provisioning Service 수준 할당 정책보다 우선 순위가 높습니다.
    * **사용자 지정(Azure Function 사용)**: Device Provisioning Service가 Azure Function 코드를 호출하여 디바이스 및 등록에 대한 모든 관련 정보를 제공합니다. 실행된 함수 코드는 디바이스를 프로비전하는 데 사용된 IoT Hub 정보를 반환합니다.

1. **이 디바이스를 할당할 수 있는 IoT 허브 선택** 드롭다운에는 앞에서 만든 **iot-az220-training-{your-id}** IoT 허브가 지정되어 있습니다.

    이 필드는 디바이스를 할당할 수 있는 IoT Hub를 지정하는 데 사용됩니다.

1. **다시 프로비전닝할 때 디바이스 데이터를 처리하도록 설정하는 방법 선택** 필드가 기본 값인 **데이터 다시 프로비저닝 및 마이그레이션**으로 설정된 상태로 둡니다.

    이 필드는 동일한 디바이스(동일한 등록 ID를 통해 표시된 대로)가 이미 한 번 이상 성공적으로 프로비저닝된 후 나중에 프로비저닝 요청을 제출하는 재프로비저닝 동작에 높은 수준의 제어를 제공합니다. 세 가지 옵션을 사용할 수 있습니다.

    * **데이터 다시 프로비저닝 및 마이그레이션**: 이 정책은 새 등록 항목의 기본값입니다. 이 정책은 등록 항목과 연결된 디바이스가 새 프로비저닝 요청을 제출할 때 조치를 취합니다. 등록 항목 구성에 따라 디바이스가 다른 IoT 허브에 다시 할당될 수 있습니다. 디바이스에서 IoT 허브가 변경되면 초기 IoT 허브에 대한 디바이스 등록이 제거됩니다. 초기 IoT 허브의 모든 디바이스 상태 정보는 새 IoT 허브로 마이그레이션됩니다.
    * **다시 프로비저닝 및 초기 구성으로 다시 설정**: 이 정책은 IoT Hub를 변경하지 않는 초기화에 사용되는 경우가 많습니다. 이 정책은 등록 항목과 연결된 디바이스가 새 프로비저닝 요청을 제출할 때 조치를 취합니다. 등록 항목 구성에 따라 디바이스가 다른 IoT 허브에 다시 할당될 수 있습니다. 디바이스에서 IoT 허브가 변경되면 초기 IoT 허브에 대한 디바이스 등록이 제거됩니다. 디바이스가 프로비전되었을 때 프로비전 서비스 인스턴스 받은 초기 구성 데이터는 새 IoT 허브에 제공됩니다.
    * **다시 프로비전 안 함**: 디바이스가 다른 허브에 다시 할당되지 않습니다. 이 정책은 이전 버전과 호환성을 관리하기 위한 용도로 제공됩니다.

1. 이름이 `telemetryDelay`이고 값이 `"2"`인 속성을 지정하려면 **초기 디바이스 트윈 상태** 필드에서 다음과 같이 JSON 개체를 업데이트합니다.

    최종 JSON은 다음과 같습니다.

    ```json
    {
        "tags": {},
        "properties": {
            "desired": {
                "telemetryDelay": "2"
            }
        }
    }
    ```

    이 필드에는 디바이스의 원하는 속성 초기 구성을 나타내는 JSON 데이터가 포함되어 있습니다. 입력한 데이터는 디바이스에서 센서 원격 분석을 읽고 이벤트를 IoT Hub로 보내는 시간 지연을 설정하는 데 사용됩니다.

1. **입력 사용** 필드를 **사용** 상태로 둡니다.

    일반적으로 새 등록 항목을 사용하도록 설정하고 사용 상태로 유지하려고 합니다.

1. **등록 추가** 블레이드 맨 위에 있는 **저장**을 클릭합니다.

#### 작업 2: 등록 검토 및 인증 키 가져오기

1. **등록 관리** 창에서 개별 디바이스 등록 목록을 보려면 **개별 등록**을 클릭합니다.

    앞에서 설명한 것처럼, 등록 레코드를 사용하여 인증 키를 가져옵니다.

1. **등록 ID**에서 **sensor-thl-1000**을 클릭합니다.

    이 블레이드에서는 방금 만든 개별 등록의 등록 세부 정보를 볼 수 있습니다.

1. **인증 유형** 섹션을 찾습니다.

    등록을 만들 때 인증 유형을 대칭 키로 지정했으므로 기본 키 및 보조 키 값이 자동으로 작성되었습니다. 각 텍스트 상자 오른쪽에는 값을 복사하는 데 사용할 수 있는 단추가 있습니다.

1. 이 디바이스 등록의 **기본 키** 및 **보조 키** 값을 복사한 다음 나중에 참조할 수 있도록 파일에 저장합니다

    이러한 키는 디바이스가 IoT Hub 서비스에 인증을 하는 데 필요한 인증 키입니다.

1. **초기 디바이스 쌍 상태**를 보면 디바이스 쌍의 필요한 상태에 대한 JSON에 `telemetryDelay` 속성 값이 `"2"`로 설정되어 있습니다.

1. **sensor-thl-1000** 개별 등록 블레이드를 닫습니다.

### 연습 3: 시뮬레이션된 디바이스 구성

이 연습에서는 이전 연습에서 만든 개별 등록을 사용하여 Azure IoT에 연결하기 위해 C#으로 작성된 시뮬레이션 디바이스를 구성합니다. 또한 Azure IoT Hub에 있는 디바이스 쌍에 따라 디바이스 구성을 읽고 업데이트하는 시뮬레이션 디바이스에 코드를 추가합니다.

이 연습에서 만든 시뮬레이션된 디바이스는 운송 컨테이너/상자에 있는 IoT 디바이스를 나타내며, 운송하는 동안 Contoso 제품을 모니터링하기 위해 사용합니다. Azure IoT Hub로 전송되는 디바이스의 센서 원격 분석에는 컨테이너의 온도와 습도, 압력, 위도/경도 좌표가 포함됩니다. 이 디바이스는 전체 자산 추적 솔루션의 일부입니다.

> **참고**: 이 시뮬레이션된 디바이스를 만드는 작업이 이전 랩에서 수행했던 디바이스 만들기 작업과 다소 중복된다고 생각될 수도 있습니다. 그러나 이 랩에서 구현하는 증명 메커니즘은 이전 랩에서 수행했던 구현과는 매우 다릅니다. 이전 랩에서는 공유 액세스 키를 사용하여 인증을 했으므로 디바이스를 프로비전할 필요가 없었습니다. 또한 프로비전 관리 이점(예: 디바이스 트윈 활용)도 제공되지 않았으며, 공유 키를 대규모로 배포하고 관리해야 했습니다. 이 랩에서는 Device Provisioning Service를 통해 고유한 디바이스를 프로비전합니다.

#### 작업 1: 시뮬레이션된 디바이스 만들기

1. **dps-az220-training-{your-id}** 블레이드의 왼쪽 메뉴에서 **개요**를 클릭합니다.

1. 블레이드의 오른쪽 상단 영역에서 **ID 범위** 에 할당된 값을 마우스 포인터로 가리키고 **클립보드에 복사**를 클릭합니다.

    이 값을 곧 사용할 것이므로 클립보드를 사용할 수 없는 경우 값을 기록해 둡니다. 대문자 "O"와 숫자 "0"을 구분해야 합니다.

    **ID 범위**는 다음 값과 유사합니다. `0ne0004E52G`

1. **Visual Studio Code**를 엽니다.

1. **파일** 메뉴에서 **폴더 열기**를 클릭하고 랩 5의 Starter 폴더로 이동합니다.

    랩 5 Starter 폴더는 랩 3에서 개발 환경을 설정할 때 다운로드한 랩 리소스 파일의 일부분입니다. 폴더 경로는 다음과 같습니다.

    * 모든 파일
      * 랩
          * 05-DPS에서 디바이스 개별 등록
            * Starter

1. **폴더 열기** 대화 상자에서 **ContainerDevice**를 클릭한 다음 **폴더 선택**을 클릭합니다.
 
    랩 5 Starter 폴더의 하위 폴더인 ContainerDevice 폴더에는 Program.cs 파일과 ContainerDevice.csproj 파일이 포함되어 있습니다.

    > **참고**: Visual Studio Code에서 필요한 자산을 로드하라는 메시지가 표시되면 **예**를 클릭하여 로드합니다.
 
1. **보기** 메뉴에서 **터미널**을 클릭합니다.

    선택한 터미널 셸이 Windows 명령 프롬프트인지 확인합니다.

1. 모든 애플리케이션 NuGet 패키지를 복원하려면 터미널 명령 프롬프트에서 다음 명령을 입력합니다.

    ```cmd/sh
    dotnet restore
    ```

1. Visual Studio Code **탐색기**창에서 **Program.cs**를 클릭합니다.

1. 코드 편집기의 Program 클래스 위쪽에서 **dpsIdScope** 변수를 찾습니다.

1. Device Provisioning Service에서 복사한 ID 범위를 사용하여 **dpsIdScope**에 할당된 값을 업데이트합니다.

    > **참고**: ID 범위 값을 사용할 수 없는 경우 Azure Portal에 있는 DPS 서비스의 개요 블레이드에서 찾을 수 있습니다.

1. **registrationId** 변수를 찾아서 할당된 값을 **sensor-thl-1000**으로 업데이트합니다.

    이 변수는 Device Provisioning Service에서 만든 개별 등록의 **등록 ID** 값을 나타냅니다.

1. 앞에서 저장한 **기본 키** 및 **보조 키** 값을 사용하여 **individualEnrollmentPrimaryKey** 및 **individualEnrollmentSecondaryKey** 변수를 업데이트합니다.

    > **참고**: 이러한 키 값을 사용할 수 없는 경우 다음과 같이 Azure Portal에서 복사할 수 있습니다.
    >
    > **등록 관리** 블레이드를 열고 **개별 등록**을 클릭한 후 **sensor-thl-1000**을 클릭합니다. 값을 복사한 다음 위에 명시된 대로 붙여넣습니다.

#### 작업 2: 프로비전 코드 추가

이 작업에서는 DPS를 통해 디바이스를 프로비전하는 코드를 구현하고, IoT Hub에 연결하는 데 사용할 수 있는 DeviceClient 인스턴스를 만듭니다.

1. **Program.cs** 파일의 코드를 잠시 살펴봅니다. 

    **ContainerDevice** 애플리케이션의 전반적인 레이아웃은 랩 4에서 만들었던 **CaveDevice** 애플리케이션과 비슷합니다. 두 애플리케이션에는 모두 다음 항목이 포함되어 있습니다.

    * using 문
    * 네임스페이스 정의
      * Program 클래스 - Azure IoT에 연결하여 원격 분석을 전송합니다.
      * EnvironmentSensor 클래스 - 센서 데이터를 생성합니다.

1. 코드 편집기에서 `// INSERT Main method below here` 주석을 찾습니다.

1. 시뮬레이션된 디바이스 애플리케이션용 **Main** 메서드를 만들려면 다음 코드를 입력합니다.

    ```csharp
    public static async Task Main(string[] args)
    {

        using (var security = new SecurityProviderSymmetricKey(registrationId,
                                                                individualEnrollmentPrimaryKey,
                                                                individualEnrollmentSecondaryKey))
        using (var transport = new ProvisioningTransportHandlerAmqp(TransportFallbackType.TcpOnly))
        {
            ProvisioningDeviceClient provClient =
                ProvisioningDeviceClient.Create(GlobalDeviceEndpoint, dpsIdScope, security, transport);

            using (deviceClient = await ProvisionDevice(provClient, security))
            {
                await deviceClient.OpenAsync().ConfigureAwait(false);

                // INSERT 이 주석 아래에서 OnDesiredPropertyChanged 이벤트 처리를 설정합니다.

                // INSERT 이 주석 아래에서 디바이스 트윈 속성을 로드합니다.

                // 디바이스 원격 분석 읽기와 전송을 시작합니다.
                Console.WriteLine("Start reading and sending device telemetry...");
                await SendDeviceToCloudMessagesAsync(deviceClient);

                await deviceClient.CloseAsync().ConfigureAwait(false);
            }
        }
    }
    ```

    이 애플리케이션의 Main 메서드는 이전 랩에서 만든 CaveDevice 애플리케이션의 Main 메서드와 비슷한 용도로 사용되지만 약간 더 복잡합니다. CaveDevice 앱에서는 디바이스 연결 문자열을 사용하여 IoT Hub에 직접 연결했습니다. 하지만 이번에는 먼저 디바이스를 프로비전(하거나 후속 연결을 위해 디바이스가 아직 프로비전되어 있는지 확인)한 다음 적절한 IoT Hub 연결 세부 정보를 검색해야 합니다.

    DPS에 연결하려면 **dpsScopeId** 및 **GlobalDeviceEndpoint**(변수에 정의되어 있음)가 필요할 뿐 아니라 다음 항목도 지정해야 합니다.

    * **security** - 등록을 인증하는 데 사용되는 방법입니다. 앞에서 개별 등록이 대칭 키를 사용하도록 구성했으므로 security로는 **SecurityProviderSymmetricKey**를 선택해야 합니다. X.509 및 TPM을 지원하는 공급자의 변형도 있습니다.

    * **transport** - 프로비전된 디바이스에서 사용하는 전송 프로토콜입니다. 여기서는 AMQP 처리기(**ProvisioningTransportHandlerAmqp**)를 선택했습니다. 물론 HTTP 및 MQTT 처리기도 사용 가능합니다.

    **security** 및 **transport** 변수에 값을 입력하고 나면 **ProvisioningDeviceClient** 인스턴스가 작성됩니다. 이 인스턴스를 사용하여 디바이스를 등록하고 **DeviceClient** 및 **ProvisionDevice** 메서드를 만듭니다. 이 두 메서드는 잠시 후에 추가하겠습니다.

    **Main** 메서드의 나머지 부분에서는 **CaveDevice**에서와는 약간 다른 방식으로 디바이스 클라이언트를 사용합니다. 즉, 이번에는 앱이 디바이스 트윈(다음 연습에서 자세히 설명함)을 사용할 수 있도록 디바이스 연결을 명시적으로 연 다음 **SendDeviceToCloudMessagesAsync** 메서드를 호출하여 원격 분석 전송을 시작합니다.

    **SendDeviceToCloudMessagesAsync** 메서드는 **CaveDevice** 애플리케이션에서 작성했던 메서드와 매우 비슷합니다. 이 메서드는 **EnvironmentSensor** 클래스 인스턴스를 만들고(이 클래스도 압력 및 위치 데이터를 반환함) 메시지를 작성한 다음 전송합니다. 메서드 루프 내에서는 고정 지연을 사용하는 대신 **telemetryDelay** 변수를 사용하여 `await Task.Delay(telemetryDelay * 1000);`과 같이 지연을 계산합니다. 시간이 되면 이 클래스를 이전 랩에서 사용했던 클래스와 자세히 비교해 보세요.

    마지막으로 **Main** 메서드로 돌아와 디바이스 클라이언트를 닫습니다.

    > **정보**: [여기](https://docs.microsoft.com/ko-kr/dotnet/api/microsoft.azure.devices.provisioning.client.provisioningdeviceclient?view=azure-dotnet)서 제공되는 **ProvisioningDeviceClient** 설명서에서 기타 관련 클래스를 쉽게 확인할 수 있습니다.

1. `// INSERT ProvisionDevice method below here` 주석을 찾습니다.

1. **ProvisionDevice** 메서드를 만들려면 다음 코드를 입력합니다.

    ```csharp
    private static async Task<DeviceClient> ProvisionDevice(ProvisioningDeviceClient provisioningDeviceClient, SecurityProviderSymmetricKey security)
    {
        var result = await provisioningDeviceClient.RegisterAsync().ConfigureAwait(false);
        Console.WriteLine($"ProvisioningClient AssignedHub: {result.AssignedHub}; DeviceID: {result.DeviceId}");
        if (result.Status != ProvisioningRegistrationStatusType.Assigned)
        {
            throw new Exception($"DeviceRegistrationResult.Status is NOT 'Assigned'");
        }

        var auth = new DeviceAuthenticationWithRegistrySymmetricKey(
            result.DeviceId,
            security.GetPrimaryKey());

        return DeviceClient.Create(result.AssignedHub, auth, TransportType.Amqp);
    }
    ```

    위 코드에 나와 있는 것처럼, 이 메서드는 앞에서 만든 프로비전 디바이스 클라이언트 및 보안 인스턴스를 수신합니다. `provisioningDeviceClient.RegisterAsync()`가 호출되어 **DeviceRegistrationResult**가 반환됩니다. 이 결과에는 **DeviceId**, **AssignedHub**, **Status** 등의 여러 속성이 포함됩니다.

    > **정보**: **DeviceRegistrationResult** 속성에 대한 전체 정보는 [여기](https://docs.microsoft.com/ko-kr/dotnet/api/microsoft.azure.devices.provisioning.client.deviceregistrationresult?view=azure-dotnet)서 확인할 수 있습니다.

    그런 다음 이 메서드는 프로비전 상태가 설정되었는지 확인하며, 디바이스가 *Assigned* 상태가 아니면 예외를 throw합니다. 반환된 수 있는 다른 결과로는 *Unassigned*, *Assigning*, *Failed*, *Disabled* 등이 있습니다.

    * **Program.ProvisionDevice** 메서드에는 DPS를 통해 디바이스를 등록하기 위한 논리가 포함되어 있습니다.
    * **Program.SendDeviceToCloudMessagesAsync** 메서드는 원격 분석을 디바이스-클라우드 메시지로 Azure IoT Hub에 보냅니다.
    * **EnvironmentSensor** 클래스에는 Temperature, Humidity, Pressure, Latitude 및 Longitude에 해당하는 시뮬레이션된 센서 판독값을 생성하는 논리가 있습니다. 

1. **SendDeviceToCloudMessagesAsync** 메서드를 찾습니다.

1. **SendDeviceToCloudMessagesAsync** 메서드 아래쪽에는 `Task.Delay()` 호출이 있습니다.

    `Task.Delay()`는 다음 원격 분석 메시지를 만들고 보내기 전에 `while` 루프를 일정 시간 동안 "일시 중지"하는 데 사용됩니다. **telemetryDelay** 변수는 다음 원격 분석 메시지를 보내기 전에 대기할 시간(초)을 정의하는 데 사용됩니다. Contoso에서는 지연 시간을 구성할 수 있어야 합니다.  

1. **Program** 클래스 위쪽의 **telemetryDelay** 변수 선언을 찾습니다.

    지연의 기본값은 **1**초로 설정되어 있습니다. 다음 단계는 디바이스 쌍 값을 사용하여 지연 시간을 제어하는 코드를 통합하는 것입니다.

#### 작업 3: 디바이스 쌍 속성 통합

디바이스에서 (Azure IoT Hub의) 디바이스 트윈 속성을 사용하려면 디바이스 트윈 속성에 액세스하여 해당 속성을 적용하는 코드를 만들어야 합니다. 여기서는 telemetryDelay 디바이스 트윈의 원하는 속성을 읽도록 시뮬레이션된 디바이스 코드를 업데이트합니다. 그런 다음 코드의 해당 **telemetryDelay** 변수에 속성 값을 할당합니다. 또한, 현재 디바이스에서 구현되는 지연 시간 레코드를 포함하도록 디바이스 트윈의 보고된 속성(IoT Hub에서 유지 관리함)을 업데이트하려고 합니다.

1. Visual Studio Code 편집기에서 **Main** 메서드를 찾습니다.

    디바이스 트윈 속성 통합을 시작하려면 디바이스 트윈 속성 업데이트 시에 시뮬레이션 디바이스가 알림을 받아야 하도록 설정하는 코드를 작성해야 합니다.

    이렇게 하려면 `DeviceClient.SetDesiredPropertyUpdateCallbackAsync` 메서드를 사용하고, `OnDesiredPropertyChanged` 메서드를 만들어 이벤트 처리기를 설정합니다.

1. `// INSERT Setup OnDesiredPropertyChanged Event Handling below here` 주석을 찾습니다.

1. OnDesiredPropertyChanged 이벤트용 DeviceClient를 설정하려면 다음 코드를 입력합니다.

    ```csharp
    await deviceClient.SetDesiredPropertyUpdateCallbackAsync(OnDesiredPropertyChanged, null).ConfigureAwait(false);
    ```

    **SetDesiredPropertyUpdateCallbackAsync** 메서드는 **DesiredPropertyUpdateCallback** 이벤트 처리기가 디바이스 트윈의 원하는 속성 변경 내용을 수신하도록 설정하는 데 사용됩니다. 이 코드는 디바이스 트윈 속성 변경 이벤트가 수신될 때 **OnDesiredPropertyChanged** 메서드를 호출하도록 **deviceClient**를 구성합니다.

    이벤트 처리기를 설정하는 **SetDesiredPropertyUpdateCallbackAsync** 메서드를 포함했으므로, 이제 해당 메서드가 호출하는 **OnDesiredPropertyChanged** 메서드를 만들어야 합니다.

1. `// INSERT OnDesiredPropertyChanged method below here` 주석을 찾습니다.

1. **OnDesiredPropertyChanged** 메서드를 만들려면 다음 코드를 입력합니다.

    ```csharp
    private static async Task OnDesiredPropertyChanged(TwinCollection desiredProperties, object userContext)
    {
        Console.WriteLine("Desired Twin Property Changed:");
        Console.WriteLine($"{desiredProperties.ToJson()}");

        // 필요한 쌍 속성 읽기
        if (desiredProperties.Contains("telemetryDelay"))
        {
            string desiredTelemetryDelay = desiredProperties["telemetryDelay"];
            if (desiredTelemetryDelay != null)
            {
                telemetryDelay = int.Parse(desiredTelemetryDelay);
            }
            // 필요한 telemetryDelay가 null이거나 지정되지 않은 경우 변경하지 마세요.
        }

        // 쌍 속성 보고
        var reportedProperties = new TwinCollection();
        reportedProperties["telemetryDelay"] = telemetryDelay.ToString();
        await deviceClient.UpdateReportedPropertiesAsync(reportedProperties).ConfigureAwait(false);
        Console.WriteLine("Reported Twin Properties:");
        Console.WriteLine($"{reportedProperties.ToJson()}");
    }
    ```

    **OnDesiredPropertyChanged** 이벤트 처리기에서는 **TwinCollection** 형식의 **desiredProperties** 매개 변수를 사용할 수 있습니다.

    **desiredProperties** 매개 변수의 값에 **telemetryDelay**(디바이스 트윈의 원하는 속성)가 포함되어 있는 경우, 코드는 디바이스 트윈 속성의 값을 **telemetryDelay** 변수에 할당합니다. **SendDeviceToCloudMessagesAsync** 메서드에 포함된 **Task.Delay** 호출은 **telemetryDelay** 변수를 사용하여 IoT Hub로 전송되는 메시지 간의 지연 시간을 설정합니다.

    다음 코드 블록은 디바이스 백업의 현재 상태를 Azure IoT Hub에 보고하는 데 사용됩니다. 이 코드는 **DeviceClient.UpdateReportedPropertiesAsync** 메서드를 호출하여 디바이스 속성의 현재 상태를 포함하는 **TwinCollection**을 해당 메서드에 전달합니다. 이는 디바이스가 디바이스 쌍이 원하는 속성 변경 이벤트를 수신하고 이제 그에 따라 구성을 업데이트 했음을 IoT Hub에 다시 보고하는 방법입니다. 이제 필요 속성의 반복이 아닌, 설정된 속성을 보고합니다. 디바이스에서 전송된 보고 속성이 디바이스가 수신한 필요한 상태와 다른 경우, IoT Hub는 디바이스의 상태를 반영하는 정확한 디바이스 쌍을 유지합니다.

    이제 디바이스가 Azure IoT Hub에서 디바이스 쌍이 원하는 속성에 대한 업데이트를 받을 수 있으므로 디바이스가 시작될 때 초기 설정을 구성하도록 코딩해야 합니다. 이를 위해 디바이스가 Azure IoT Hub에서 현재 디바이스 쌍이 원하는 속성을 로드하고 그에 따라 자체적으로 구성해야 합니다.

1. **Main** 메서드에서 `// INSERT Load Device Twin Properties below here` 주석을 찾습니다.

1. 디바이스 트윈의 원하는 속성을 읽고 디바이스 시작 시에 해당 속성과 일치하도록 디바이스를 구성하려면 다음 코드를 입력합니다.

    ```csharp
    var twin = await deviceClient.GetTwinAsync().ConfigureAwait(false);
    await OnDesiredPropertyChanged(twin.Properties.Desired, null);
    ```

    이 코드는 시뮬레이션된 디바이스에 대한 디바이스 쌍을 검색하는 `DeviceTwin.GetTwinAsync` 메서드를 호출합니다. 그런 다음 `Properties.Desired` 속성 개체에 액세스하여 디바이스의 현재 필요한 상태를 검색한 후 **OnDesiredPropertyChanged** 메서드에 전달합니다. 그러면 이 메서드가 시뮬레이션된 디바이스의 **telemetryDelay** 변수를 구성합니다.

    이 코드는 _OnDesiredPropertyChanged_ 이벤트를 처리하기 위해 이미 만든 **OnDesiredPropertyChanged** 메서드를 재사용합니다. 이렇게 하면 디바이스 쌍의 필요한 상태 속성을 읽고 한 곳에서 시작 시 디바이스를 구성하는 코드를 유지하는 데 도움이 됩니다. 결과 코드는 더 간단하고 유지 관리가 더 쉽습니다.

1. Visual Studio Code **파일** 메뉴에서 **저장**을 클릭합니다.

    이제 시뮬레이션된 디바이스에서 Azure IoT Hub의 디바이스 쌍 속성을 사용하여 원격 분석 메시지 간의 지연을 설정합니다.

### 연습 4: 시뮬레이션된 디바이스 테스트

이 연습에서는 시뮬레이션된 디바이스를 실행하고 Azure IoT Hub로 센서 원격 분석을 보내는지 확인합니다. 또한 Azure IoT Hub로 원격 분석이 전송되는 속도도 변경합니다. 구체적으로는 Azure IoT Hub 내에서 시뮬레이션된 디바이스의 telemetryDelay 디바이스 트윈 속성을 업데이트합니다.

#### 작업 1: 디바이스 빌드 및 실행

1. Visual Studio Code에서 코드 프로젝트를 열어 두었는지 확인합니다.

1. **보기** 메뉴에서 **터미널**을 클릭합니다.

1. 터미널 창에서 명령 프롬프트에 `Program.cs` 파일의 디렉터리 경로가 표시되는지 확인합니다.

1. 명령 프롬프트에서 시뮬레이션된 디바이스 애플리케이션을 빌드하고 실행하려면 다음 명령을 입력합니다.

    ```cmd/sh
    dotnet run
    ```

    > **참고**: 시뮬레이션된 디바이스 애플리케이션은 실행되면 먼저 콘솔(터미널 창)에 상태에 대한 몇 가지 세부 정보를 작성합니다.

1. `Desired Twin Property Changed:` 줄 뒤에 있는 JSON 출력에 디바이스의 `telemetryDelay` 필요한 값이 있습니다.

    터미널 창에서 위로 스크롤하여 출력을 확인할 수 있습니다. 다음과 유사하게 나타납니다.

    ```text
    ProvisioningClient AssignedHub: iot-az220-training-{your-id}.azure-devices.net; DeviceID: sensor-thl-1000
    Desired Twin Property Changed:
    {"telemetryDelay":"2","$version":1}
    Reported Twin Properties:
    {"telemetryDelay":"2"}
    Start reading and sending device telemetry...
    ```

1. 시뮬레이션된 디바이스 애플리케이션은 Azure IoT Hub로 원격 분석 이벤트를 보내기 시작합니다.

    원격 분석 이벤트에는 `temperature`, `humidity`, `pressure`, `latitude`, 및 `longitude` 값이 포함되어 있으며 다음과 유사해야 합니다.

    ```text
    11/6/2019 6:38:55 PM > Sending message: {"temperature":25.59094770373355,"humidity":71.17629229611545,"pressure":1019.9274696347665,"latitude":39.82133964767944,"longitude":-98.18181981142438}
    11/6/2019 6:38:57 PM > Sending message: {"temperature":24.68789062681044,"humidity":71.52098010830628,"pressure":1022.6521258267584,"latitude":40.05846882452387,"longitude":-98.08765031156229}
    11/6/2019 6:38:59 PM > Sending message: {"temperature":28.087463226675737,"humidity":74.76071353757787,"pressure":1017.614206096327,"latitude":40.269273772972454,"longitude":-98.28354453319591}
    11/6/2019 6:39:01 PM > Sending message: {"temperature":23.575667940813894,"humidity":77.66409506912534,"pressure":1017.0118147748344,"latitude":40.21020096551372,"longitude":-98.48636739129239}
    ```

    원격 분석 판독값 간의 타임스탬프 차이점을 확인합니다. 원격 분석 메시지 간의 지연은 소스 코드에서는 기본값이 `1`초인 것과 달리 디바이스 쌍으로 구성된 것처럼 `2`초여야 합니다. .

1. 시뮬레이션된 디바이스 앱을 실행 상태로 둡니다.

    다음 활동 중에 디바이스 코드가 예상대로 작동하는지 확인합니다.

#### 작업 2: Azure IoT Hub로 전송된 원격 분석 스트림 확인

이 작업에서는 Azure CLI를 사용하여 시뮬레이션된 디바이스에서 보낸 원격 분석이 Azure IoT Hub에서 수신되고 있는지 확인합니다.

1. 브라우저를 사용하여 [Azure Cloud Shell](https://shell.azure.com/)을 열고 이 과정에 사용 중인 Azure 구독으로 로그인합니다.

1. Azure Cloud Shell에서 다음 명령을 입력합니다.

    ```cmd/sh
    az iot hub monitor-events --hub-name {IoTHubName} --device-id sensor-thl-1000
    ```

    _**{IoTHubName}** 자리 표시자를 Azure IoT Hub의 이름으로 바꿔야 합니다._

1. IoT 허브가 sensor-thl-1000 디바이스에서 원격 분석 메시지를 받고 있습니다.

    시뮬레이션된 디바이스 애플리케이션이 다음 작업을 계속 실행하도록 둡니다.

#### 작업 3: 쌍을 통해 디바이스 구성 변경

시뮬레이션된 디바이스가 실행 중인 상태에서 Azure IoT Hub 내에서 디바이스 트윈의 필요한 상태를 편집하여 `telemetryDelay` 구성을 업데이트할 수 있습니다. 이 작업은 Azure Portal의 Azure IoT Hub에서 디바이스를 구성하여 수행할 수 있습니다.

1. Azure Portal을 열고(아직 열려 있지 않은 경우) **Azure IoT Hub** 서비스로 이동합니다.

1. IoT Hub 블레이드 왼쪽 메뉴의 **탐색기**에서 **IoT 디바이스**를 클릭합니다.

1. **디바이스 ID**에서 **sensor-thl-1000**을 클릭합니다.

    > **중요**: 이 랩에 사용할 디바이스를 선택해야 합니다.

1. **sensor-thl-1000** 디바이스 블레이드에서 블레이드 상단의 **디바이스 트윈**을 클릭합니다.

    **디바이스 쌍** 블레이드는 편집기에 디바이스 쌍에 대한 전체 JSON을 제공합니다. 이렇게 하면 Azure Portal에서 디바이스 쌍 상태를 직접 보거나 편집할 수 있습니다.

1. `properties.desired` 개체의 JSON을 찾습니다.

    여기에는 디바이스의 필요한 상태가 포함됩니다. DPS의 개별 등록에 따라 디바이스가 프로비저닝될 때 구성된 `telemetryDelay` 속성이 이미 있고 `"2"`로 설정되어 있습니다.

1. `telemetryDelay` 원하는 속성에 할당된 값을 업데이트하려면 값을 `"5"`로 변경합니다.

    값에는 따옴표("")가 포함됩니다.

1. **디바이스 트윈** 블레이드 위쪽에서 **저장**을 클릭합니다.

    `OnDesiredPropertyChanged` 이벤트는 시뮬레이션된 디바이스의 코드 내에서 자동으로 트리거되며 디바이스는 디바이스 쌍의 필요한 상태에 대한 변경 내용을 반영하도록 구성을 업데이트합니다.

1. 시뮬레이션된 디바이스 애플리케이션을 실행하는 데 사용하는 Visual Studio Code 창으로 전환합니다.

1. Visual Studio Code에서 터미널 창의 아래쪽으로 스크롤합니다.

1. 디바이스가 디바이스 쌍 속성의 변경 사항을 인식합니다.

    출력에는 새로운 필요한 `telemetryDelay` 속성 값에의 JSON과 함께 `Desired Twin Property Changed`이라는 메시지가 표시됩니다. 디바이스가 디바이스 쌍의 필요한 상태의 새로운 구성을 선택하면, 자동으로 현재 구성된 대로 5초마다 센서 원격 분석을 보내도록 업데이트합니다.

    ```text
    Desired Twin Property Changed:
    {"telemetryDelay":"5","$version":2}
    Reported Twin Properties:
    {"telemetryDelay":"5"}
    4/21/2020 1:20:16 PM > Sending message: {"temperature":34.417625961088405,"humidity":74.12403526442313,"pressure":1023.7792049974805,"latitude":40.172799921919186,"longitude":-98.28591913777421}
    4/21/2020 1:20:22 PM > Sending message: {"temperature":20.963297521678403,"humidity":68.36916032636965,"pressure":1023.7596862048422,"latitude":39.83252821949164,"longitude":-98.31669969393461}
    ```

1. Azure Cloud Shell에서 Azure CLI 명령을 실행하는 브라우저 페이지로 전환합니다.

    `az iot hub monitor-events` 명령을 계속 실행 중인지 확인합니다. 실행 중이 아닌 경우 명령을 다시 시작합니다.

1. Azure IoT Hub로 전송된 원격 분석 이벤트는 5초의 새 간격으로 수신됩니다.

1. **Ctrl-C**를 사용하여 `az` 명령과 시뮬레이션된 디바이스 애플리케이션을 모두 중지합니다.

1. Azure Portal이 표시된 브라우저 창으로 전환합니다.

1. **디바이스 트윈** 블레이드를 닫습니다.

1. 계속해서 Azure Portal의 **sensor-thl-1000** 디바이스 블레이드에서 **디바이스 트윈**을 클릭합니다.

1. `properties.reported` 개체의 JSON을 찾습니다.

    JSON의 이 부분에는 디바이스에서 보고한 상태가 포함됩니다. `telemetryDelay` 속성도 여기에 있으며 `5`로 설정되어 있습니다.  또한 보고된 데이터 값이 마지막으로 업데이트된 시점과 특정 보고된 값이 마지막으로 업데이트된 시기를 표시하는 `$metadata` 값도 있습니다.

1. **디바이스 트윈** 블레이드를 닫습니다.

1. 시뮬레이션된 디바이스 블레이드를 닫은 후 IoT Hub 블레이드를 닫습니다.

### 연습 5: 디바이스 프로비전 해제

Contoso 시나리오에서는 배송 컨테이너가 최종 목적지에 도착하면 IoT 디바이스가 컨테이너에서 제거되어 Contoso 위치로 반환됩니다. Contoso에서 이렇게 반환된 디바이스를 테스트하여 재고에 포함하려면 먼저 디바이스 프로비전을 해제해야 합니다. 그러면 나중에 해당 디바이스를 같은 IoT Hub 또는 다른 지역의 IoT Hub에 프로비전할 수 있습니다. 디바이스 프로비전 해제 완료는 IoT 솔루션 내의 IoT 디바이스 수명 주기에서 중요한 단계입니다.

이 연습에서는 DPS(Device Provisioning Service) 및 Azure IoT Hub에서 디바이스 프로비전을 해제하는 데 필요한 작업을 수행합니다. Azure IoT 솔루션에서 IoT 디바이스를 완전히 프로비전 해제하려면 이 두 서비스에서 모두 디바이스를 제거해야 합니다. 

#### 작업 1: DPS에서 디바이스 등록 취소

1. 필요한 경우 Azure 계정 자격 증명을 사용하여 Azure Portal에 로그인합니다.

    Azure 계정이 두 개 이상인 경우 이 과정에 사용할 구독에 연결된 계정으로 로그인해야 합니다.

1. 리소스 그룹 타일에서 Device Provisioning Service를 열려면 **dps-az220-training-{your-id}** 를 클릭합니다.

1. 왼쪽 메뉴의 **설정**에서 **등록 관리**를 클릭합니다.

1. **등록 관리** 창에서 개별 디바이스 등록 목록을 보려면 **개별 등록**을 클릭합니다.

1. **sensor-thl-1000** 왼쪽의 체크박스를 클릭합니다.

    > **참고**: sensor-thl-1000 개별 디바이스 등록을 열 필요는 없으며 등록을 선택만 하면 됩니다.

1. 블레이드 상단에서 **삭제**를 선택합니다.

    > **참고**: DPS에서 개별 등록을 삭제하면 등록이 영구적으로 제거됩니다. 등록을 일시적으로 사용하지 않으려면 개별 등록의 **등록 세부 정보**에서 **항목 사용** 설정을 **사용 안 함**으로 설정할 수 있습니다.

1. **등록 제거** 프롬프트에서 **예**를 클릭합니다.

    이제 개별 등록이 DPS(Device Provisioning Service)에서 제거됩니다. 프로비전 해제 프로세스를 완료하려면 시뮬레이션된 디바이스의 **디바이스 ID**도 **Azure IoT Hub** 서비스에서 제거해야 합니다.

#### 작업 2: IoT Hub에서 디바이스 등록 취소

1. Azure Portal에서 대시보드로 다시 이동합니다.

1. 리소스 그룹 타일에서 Azure IoT Hub 블레이드를 열려면 **dps-az220-training-{your-id}** 를 클릭합니다.

1. 왼쪽 메뉴의 **탐색기** 아래에서 **IoT 디바이스**를 클릭합니다.

1. **sensor-thl-1000** 왼쪽의 체크박스를 클릭합니다.

    > **중요**: 이 랩에 사용한 시뮬레이션된 디바이스를 나타내는 디바이스를 선택해야 합니다.

1. 블레이드 상단에서 **삭제**를 선택합니다.

1. **선택한 디바이스를 삭제하시겠습니까**라는 프롬프트에서  **예**를 클릭합니다.

    > **참고**:  IoT Hub에서 장치 ID를 삭제하면 디바이스 등록이 영구적으로 제거됩니다. 일시적으로 디바이스가 IoT Hub에 연결되지 않게 하려면 디바이스의 속성 내에서 **IoT Hub에 연결 사용**을 **사용 중지**로 설정할 수 있습니다.

이제 Device Provisioning Service에서 디바이스 등록이 제거되었으며 일치하는 디바이스 ID가 Azure IoT Hub에서 제거되었으므로 시뮬레이션된 디바이스는 솔루션에서 완전히 사용 중지되었습니다.
