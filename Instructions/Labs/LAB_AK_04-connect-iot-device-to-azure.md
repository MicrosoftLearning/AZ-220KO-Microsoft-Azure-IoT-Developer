---
lab:
    title: '랩 04: IoT 디바이스를 Azure에 연결'
    module: '모듈 2: 디바이스 및 디바이스 통신'
---

# IoT 디바이스를 Azure에 연결

## 랩 시나리오

Contoso는 고품질 치즈를 생산하는 곳으로 유명합니다. Contoso는 인기와 판매량이 급격히 성장함에 따라 고객이 기대하는 수준의 치즈 품질을 유지하도록 조치를 취하려고 합니다.

과거에는 각 근무 교대 근무 시 공장 작업자가 온도와 습도 데이터를 수집했습니다. Contoso는 새로운 시설이 온라인으로 전환되면 공장 확장에 따라 모니터링이 증가하고 데이터 수동 수집 프로세스를 조정하지 못할 수도 있다고 걱정하고 있습니다.

Contoso는 IoT 디바이스를 사용하여 온도와 습도를 모니터링하는 자동화 시스템을 도입하기로 결정했습니다. 원격 분석 데이터가 전달되는 속도는 치즈가 환경적으로 민감한 공정을 통과할 때 생산 공정이 제어되도록 조정할 수 있습니다.

본격적으로 구현하기 전에 이 자산 모니터링 솔루션을 평가하기 위해 온도와 습도 센서 등 IoT 디바이스를 IoT Hub에 연결합니다. 이 랩에서는 .NET Core 콘솔 애플리케이션을 사용하여 실제 IoT 디바이스를 시뮬레이션합니다.

다음의 리소스가 만들어집니다.

![랩 4 아키텍처](media/LAB_AK_04-architecture.png)

## 이 랩에서

이 랩에서는 다음 활동을 완료할 예정입니다.

* 랩 필수 구성 요소가 충족되는지 확인(필요한 Azure 리소스가 있음)
* Azure CLI를 사용하여 Azure IoT Hub에 디바이스 ID 등록
* Azure IoT Hub에 연결하도록 시뮬레이션된 (C#으로 이미 빌드되어 작성된) IoT 디바이스 구성
* Azure IoT Hub에 디바이스-클라우드 원격 분석 메시지를 보내려면 시뮬레이션된 디바이스를 실행합니다.
* Azure CLI를 사용하여 Azure IoT Hub에서 디바이스 원격 분석을 수신하고 있는지 확인

## 랩 지침

### 연습 1: 랩 필수 구성 요소 확인

이 랩은 다음 Azure 리소스를 사용할 수 있다고 가정합니다.

| 리소스 종류 | 리소스 이름 |
| :-- | :-- |
| 리소스 그룹 | AZ-220-RG |
| IoT Hub | AZ-220-HUB-_{YOUR-ID}_ |

이러한 리소스를 사용할 수 없는 경우 연습 2로 이동하기 전에 아래 설명에 따라 **lab04-setup.azcli** 스크립트를 실행해야 합니다. 스크립트 파일은 개발자 환경 구성(랩 3)의 일부로 로컬로 복제한 GitHub 리포지토리에 포함됩니다.

> **참고**:  **lab04-setup.azcli** 스크립트는 **bash** 셸 환경에서 실행되도록 작성됩니다. 이는 Azure Cloud Shel에서 실행할 수 있는 가장 쉬운 방법입니다.

1. 브라우저를 사용하여 [Azure Cloud Shell](https://shell.azure.com/)을 열고 이 과정에 사용 중인 Azure 구독으로 로그인합니다.

1. Cloud Shell에 대한 스토리지 설정 관련 메시지가 표시되면 기본값을 수락합니다.

1. Azure Shell에서 **Bash**를 사용 중인지 확인합니다.

    Azure Cloud Shell 페이지의 왼쪽 상단에 있는 드롭다운은 환경을 선택하는 데 사용됩니다. 선택한 드롭다운 값이 **Bash**인지 확인합니다.

1. Azure Shell 도구 모음에서 **파일 업로드/다운로드**(오른쪽의 네 번째 단추)를 클릭합니다.

1. 드롭다운에서 **업로드**를 클릭합니다.

1. 파일 선택 대화 상자에서 개발 환경을 구성할 때 다운로드한 GitHub 랩 파일의 폴더 위치로 이동합니다.

    이 과정의 랩 3 "개발 환경 설정"에서 ZIP 파일을 다운로드하고 내용을 로컬로 추출하여 랩 리소스를 포함하는 GitHub 리포지토리를 복제했습니다. 추출된 폴더 구조에는 다음 폴더 경로가 포함됩니다.

    * 모든 파일
      * 랩
          * 04-IoT 디바이스를 Azure에 연결
            * 설정

    lab04-setup.azcli 스크립트 파일은 랩 4의 Setup 폴더에 있습니다.

1. **lab04-setup.azcli** 파일을 선택한 다음 **열기**를 클릭합니다.

    파일 업로드가 완료되면 알림이 나타납니다.

1. 올바른 파일을 업로드했는지 확인하려면 다음 명령을 입력합니다.

    ```bash
    ls
    ```

    `ls` 명령으로 현재 디렉터리의 내용을 나열합니다. lab04-setup.azcli 파일이 나열됩니다.

1. 설치 스크립트가 포함된 이 랩에 대한 디렉터리를 만든 다음 해당 디렉터리로 이동하려면 다음 Bash 명령을 입력합니다.

    ```bash
    mkdir lab4
    mv lab04-setup.azcli lab4
    cd lab4
    ```

    이러한 명령은 이 랩의디렉터리를 만들고 **lab04-setup.azcli** 파일을 해당 디렉터리로 이동한 다음 디렉터리를 변경하여 새 디렉터리를 현재 작업 디렉터리로 만듭니다.

1. **lab04-setup.azcli**에 실행 권한이 있는지 확인하려면 다음 명령을 입력합니다.

    ```bash
    chmod +x lab04-setup.azcli
    ```

1. Cloud Shell 도구 모음에서 **lab04-setup.azcli** 파일을 편집하려면 **편집기 열기**(오른쪽에서 두 번째 단추 - **{ }**)를 클릭합니다.

1. **Files** 목록에서 lab4 폴더를 확장하려면 **lab4**를 클릭한 다음 **lab04-setup.azcli**를 클릭합니다.

    이제 편집기에서 **lab04-setup.azcli** 파일의 내용을 표시합니다.

1. 편집기에서 `{YOUR-ID}` 및 `{YOUR-LOCATION}` 변수의 값을 업데이트하세요.

    아래 샘플을 예로 들, `{YOUR-ID}`를 이 과정의 시작 시 만든 고유 ID(예: **CAH191211**)로 설정하고 리소스 그룹과 일치하는 위치로 `{YOUR-LOCATION}`을 설정해야 합니다.

    ```bash
    #!/bin/bash

    RGName="AZ-220-RG"
    IoTHubName="AZ-220-HUB-{YOUR-ID}"

    Location="{YOUR-LOCATION}"
    ```

    > **참고**:  `{YOUR-LOCATION}` 변수는 모든 리소스를 배포하는 지역의 짧은 이름으로 설정되어야 합니다. 이 명령을 입력하면 사용 가능한 위치 및 짧은 이름(**이름** 열)의 목록을 볼 수 있습니다.
    >
    > ```bash
    > az account list-locations -o Table
    >
    > 표시이름           위도    경도    이름
    > --------------------  ----------  -----------  ------------------
    > 동아시아             22.267      114.188      eastasia
    > 동남 아시아        1.283       103.833      southeastasia
    > 미국 중부            41.5908     -93.6208     centralus
    > 미국 동부               37.3719     -79.8164     eastus
    > 미국 동부 2             36.6681     -78.3889     eastus2
    > ```

1. 편집기 창의 오른쪽 상단에서 파일에 변경 내용을 저장하고 편집기를 닫고 ...를 클릭한 다음 **편집기 닫기**를 클릭합니다.

    저장하라는 메시지가 표시된 경우 **저장**을 클릭하면 편집기가 닫힙니다.

    > **참고**:  **CTRL+S**를 사용하여 언제든지 저장할 수 있으며 **CTRL+Q**를 사용하여 편집기를 닫을 수 있습니다.

1. 이 랩에 필요한 리소스를 만들려면 다음 명령을 입력합니다.

    ```bash
    ./lab04-setup.azcli
    ```

    이 작업을 실행하려면 몇 분 정도 걸립니다. 각 단계가 완료되면 JSON 출력이 표시됩니다.

스크립트가 완료되면 랩으로 계속할 준비가 끝납니다.

### 연습 2: Azure CLI를 사용하여 Azure IoT Hub 디바이스 ID 만들기

`iot` Azure CLI 모듈에는 `az iot hub device-identity` 명령 그룹의 Azure IoT Hub 내에서 IoT 디바이스를 관리하는 명령이 몇 가지 포함되어 있습니다. 이러한 명령은 스크립트 내에서 또는 명령줄/터미널에서 직접 IoT 디바이스를 관리하는 데 사용할 수 있습니다.

#### 작업 1: 구독 관리

하나의 계정에 구독을 두 개 이상 연결할 수 있으므로 구독을 나열하고 현재 활성 구독을 선택하고 기본 구독을 변경하는 방법을 이해하는 것이 중요합니다.

1. 필요한 경우 Azure 계정 자격 증명을 사용하여 Azure Portal에 로그인합니다.

    둘 이상의 Azure 계정이 있는 경우에는 이 과정에 사용할 구독에 연결된 계정으로 로그인해야 합니다.

1. Azure Portal 상단에서 **Cloud Shell** 아이콘을 클릭하여 Azure Portal 내의 **Azure Cloud Shell**을 엽니다.

1. Cloud Shell에 대한 스토리지 설정 관련 메시지가 표시되면 기본값을 수락합니다.

1. 창이 열리면 Cloud Shell 내의 **Bash** 터미널 옵션을 선택합니다.

1. 사용 가능한 구독을 나열하려면 명령 프롬프트에서 다음 명령을 입력합니다.

    ```bash
    az account list --output table

    Name                      CloudName    SubscriptionId                        State    IsDefault
    ------------------------  -----------  ------------------------------------  -------  -----------
    Subscription1             AzureCloud   aa1122bb-4bd0-462b-8449-a1002aa2233a  Enabled  True
    Subscription2             AzureCloud   aa1122bb-4bd0-462b-8449-a1002aa2233b  Enabled  False
    Azure Pass - Sponsorship  AzureCloud   aa1122bb-4bd0-462b-8449-a1002aa2233c  Enabled  False
    ```

    보시다시피 과정에 사용 중인 **Azure Pass - 스폰서쉽**이 나열되어 있지만 기본 구독으로 설정되어 있지는 않습니다.

1. 현재 활성 구독을 보려면 다음 명령을 입력합니다.

    ```bash
    az account show -o table

    EnvironmentName    IsDefault    Name           State    TenantId
    -----------------  -----------  -------------  -------  ------------------------------------
    AzureCloud         True         Subscription1  Enabled  aa1122bb-4bd0-462b-8449-a1002aa2233a
    ```

1. 현재 세션의 기본 구독을 **Azure Pass - 스폰서쉽**으로 변경하려면 다음 명령을 입력합니다.

    ```bash
    az account set --subscription "Azure Pass - Sponsorship"
    ```

    > **참고**: 구독 **Name** 또는 **SubscriptionId**를 **--subscription** 인수와 함께 사용할 수 있습니다. 이름이 같은 두 개의 구독이 있는 경우 **SubscriptionId**를 **반드시** 사용해야 합니다.

1. 변경 사항을 확인하려면 다음 명령을 입력합니다.

    ```bash
    az account show -o table

    EnvironmentName    IsDefault    Name                      State    TenantId
    -----------------  -----------  ------------------------  -------  ------------------------------------
    AzureCloud         True         Azure Pass - Sponsorship  Enabled  aa1122bb-4bd0-462b-8449-a1002aa2233c
    ```

이 구독은 이제 리소스 등을 만들 때마다 현재 세션에서 사용됩니다.

#### 작업 2: IoT Hub 디바이스 ID 만들기

1. Cloud Shell 내에서 Cloud Shell에 IoT 확장이 설치되어 있는지 확인하려면 다음 명령을 실행합니다.

    ``` sh
    az extension add --name azure-cli-iot-ext
    ```

1. 계속해서 Cloud Shell 내에서 다음 Azure CLI 명령을 실행하여 Azure IoT Hub에 **디바이스 ID**를 만듭니다. 이는 시뮬레이션된 디바이스에 대해 사용됩니다.

    ```sh
    az iot hub device-identity create --hub-name {IoTHubName} --device-id SimulatedDevice1
    ```

    > **참고**:  _{IoTHubName}_ 자리 표시자를 Azure IoT Hub의 이름으로 바꿔야 합니다. IoT Hub 이름을 잊어버린 경우 다음 명령을 입력할 수 있습니다.
    >
    >```sh
    >az iot hub list -o table
    >```

#### 작업 3: 디바이스 연결 문자열 가져오기

1. Cloud Shell 내에서 다음 Azure CLI 명령을 실행하여 IoT Hub에 방금 추가된 디바이스 ID에 대한 _디바이스 연결 문자열_을 가져옵니다. 이 연결 문자열은 시뮬레이션된 디바이스를 Azure IoT Hub에 연결하는 데 사용됩니다.

    ```cmd/sh
    az iot hub device-identity show-connection-string --hub-name {IoTHUbName} --device-id SimulatedDevice1 --output table
    ```

1. 이전 명령에서 출력된 **디바이스 연결 문자열**을 기록합니다. 나중에 사용할 수 있도록 저장해둬야 합니다.

    연결 문자열의 형식은 다음과 같습니다.

    ```text
    HostName={IoTHubName}.azure-devices.net;DeviceId=SimulatedDevice1;SharedAccessKey={SharedAccessKey}
    ```

### 연습 3: 시뮬레이션된 디바이스 구성 및 테스트(C#)

이 연습에서는 이전 연습에서 만든 디바이스 ID 및 공유 액세스 키를 사용하여 Azure IoT Hub에 연결하도록 C#으로 작성된 시뮬레이션된 디바이스를 구성합니다. 그런 다음, 디바이스를 테스트하고 IoT Hub가 예상대로 디바이스에서 원격 분석을 수신하고 있는지 확인합니다.

#### 작업 1: 랩 4 시작 코드 프로젝트 열기

1. Visual Studio Code의 새 인스턴스를 엽니다.

1. 왼쪽 메뉴에서 **탐색기**를 클릭합니다.

    탐색기 창에 파일/폴더 계층 구조가 나열됩니다. Visual Studio Code의 새 인스턴스에는 열린 폴더가 없습니다.

1. 파일 메뉴에서 **폴더 열기**를 클릭합니다.

1. 폴더 열기 대화 상자에서 시작 코드 프로젝트가 포함된 랩 4 폴더로 이동합니다.

    랩 4 시작 프로젝트 폴더에 대한 로컬 경로는 다음과 유사해야 합니다.

    * AZ-220-Microsoft-Azure-IoT-Developer-master
      * 모든 파일
        * 랩
          * 04-IoT 디바이스를 Azure에 연결
            * 시작

    > **참고**: 랩 3에서 개발 환경을 설정할 때 GitHub 프로젝트를 복제했습니다. 리소스 파일을 찾는 데 필요한 경우 과정 강사에게 문의하세요.

1. 폴더를 열려면 **시작**을 클릭한 다음 **폴더 선택**을 클릭합니다.

    이제 Visual Studio Code의 탐색기 창에 두 개의 C# 프로젝트 파일이 나열됩니다.

    * SimulatedDevice.cs
    * SimulatedDevice.csproj

#### 작업 2: 디바이스 연결 문자열 업데이트

1. SimulatedDevice.cs 파일을 열려면 Visual Studio Code 탐색기 창에서 **SimulatedDevice.cs**를 클릭합니다.

1. 편집기 보기에서 `s_connectionString` 변수가 포함된 코드 줄을 찾습니다.

    ```C#
    private readonly static string s_connectionString = "{Your device connection string here}";
    ```

1. 값 자리 표시자 `{Your device connection string here}`을 이전에 복사한 디바이스 연결 문자열로 바꿉니다.

    이렇게 하면 시뮬레이션된 디바이스가 Azure IoT Hub를 인증, 연결 및 통신할 수 있습니다.

    구성되면 변수는 다음과 유사하게 표시됩니다(특정 연결 정보가 포함됨).

    ```csharp
    private readonly static string s_connectionString = "HostName={IoTHubName}.azure-devices.net;DeviceId=SimulatedDevice1;SharedAccessKey={SharedAccessKey}";
    ```

1. **보기** 메뉴에서 **터미널**을 클릭합니다.

    선택한 터미널 셸이 Windows 명령 프롬프트인지 확인합니다.

1. 터미널 보기의 명령 프롬프트에서 다음 명령을 입력합니다.

    ```cmd/sh
    dotnet run
    ```

    이 명령은 시뮬레이션된 디바이스 애플리케이션을 빌드하고 실행합니다. 터미널 위치가 `SimulatedDevice.cs` 파일이 포함된 디렉터리로 설정되어 있는지 확인합니다.

    > **참고**:  명령이 `Malformed Token` 또는 기타 오류 메시지를 출력하는 경우 **디바이스 연결 문자열**이 `s_connectionString` 변수의 값으로 올바르게 구성되어 있는지 확인합니다.

1. 시뮬레이션된 디바이스 애플리케이션이 실행되면 `temperature` 및 `humidity` 값이 포함된 이벤트 메시지를 Azure IoT Hub로 전송합니다.

    터미널 출력은 다음과 유사합니다.

    ```text
    IoT Hub C# 시뮬레이션된 디바이스. Ctrl C를 눌러 종료.

    10/25/2019 6:10:12 PM > Sending message: {"temperature":27.714212817472504,"humidity":63.88147743599558}
    10/25/2019 6:10:13 PM > Sending message: {"temperature":20.017463779085066,"humidity":64.53511070671263}
    10/25/2019 6:10:14 PM > Sending message: {"temperature":20.723927165718717,"humidity":74.07808918230147}
    10/25/2019 6:10:15 PM > Sending message: {"temperature":20.48506045736608,"humidity":71.47250854944461}
    10/25/2019 6:10:16 PM > Sending message: {"temperature":25.027703996760632,"humidity":69.21247714628115}
    10/25/2019 6:10:17 PM > Sending message: {"temperature":29.867399432634656,"humidity":78.19206098010395}
    10/25/2019 6:10:18 PM > Sending message: {"temperature":33.29597232085465,"humidity":62.8990878830194}
    10/25/2019 6:10:19 PM > Sending message: {"temperature":25.77350195766124,"humidity":67.27347029711747}
    ```

    > **참고**: 지금은 시뮬레이션된 디바이스 앱을 실행 상태로 둡니다. 다음 작업은 IoT Hub가 원격 분석 메시지를 받고 있는지 확인하는 것입니다.

#### 작업 3: Azure IoT Hub로 전송된 원격 분석 스트림 확인

이 작업에서는 Azure CLI를 사용하여 시뮬레이션된 디바이스에서 보낸 원격 분석이 Azure IoT Hub에서 수신되고 있는지 확인합니다.

1. 브라우저를 사용하여 [Azure Cloud Shell](https://shell.azure.com/)을 열고 이 과정에 사용 중인 Azure 구독으로 로그인합니다.

1. Azure Cloud Shell에서 다음 명령을 입력합니다.

    ```cmd/sh
    az iot hub monitor-events --hub-name {IoTHubName} --device-id SimulatedDevice1
    ```

    _**IoTHubName** 자리 표시자를 Azure IoT Hub의 이름으로 바꿔야 합니다._

    > **참고**:  Azure CLI 명령을 실행할 때 _"IoT 확장 버전에 필요한 종속성 업데이트"_라는 메시지가 나타나면 `y` 키를 눌러 업데이트를 수락하고 `Enter` 키를 누릅니다. 이렇게 하면 명령이 예상대로 계속될 수 있습니다.

    `--device-id` 매개 변수는 선택 사항이며 이를 통해 단일 디바이스에서 이벤트를 모니터링할 수 있습니다. 매개 변수가 생략된 경우 명령은 지정된 Azure IoT Hub로 전송된 모든 이벤트를 모니터링합니다.

    `az iot hub` Azure CLI 모듈 내의 `monitor-events` 명령은 명령줄/터미널 내에서 Azure IoT Hub로 전송된 디바이스 원격 분석 및 메시지를 모니터링하는 기능을 제공합니다.

1. `az iot hub monitor-events` Azure CLI 명령은 지정된 Azure IoT Hub에 도착하는 이벤트의 JSON 표현을 출력합니다. 

    이 명령을 사용하면 IoT Hub로 전송되는 이벤트를 모니터링할 수 있습니다. 또한 디바이스가 IoT Hub에 연결하고 통신할 수 있는지 확인합니다.

    다음과 유사한 메시지가 표시됩니다.

    ```cmd/sh
    이벤트 모니터 시작, 디바이스에서 필터링: SimulatedDevice1, use ctrl-c to stop...
    {
        "event": {
            "origin": "SimulatedDevice1",
            "payload": "{\"temperature\":25.058683971901743,\"humidity\":67.54816981383979}"
        }
    }
    {
        "event": {
            "origin": "SimulatedDevice1",
            "payload": "{\"temperature\":29.202181296051563,\"humidity\":69.13840303623043}"
        }
    }
    ```

1. IoT Hub가 원격 분석을 받고 있는 것으로 확인되면 Azure Cloud Shell 및 Visual Studio Code 창에서 **Ctrl+C**를 누릅니다.

    Ctrl C는 실행 중인 앱을 중지하는 데 사용됩니다. 불필요한 앱과 작업을 종료해야 한다는 것을 기억하세요.
