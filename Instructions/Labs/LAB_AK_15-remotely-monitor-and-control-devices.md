---
lab:
    title: '랩 15: Azure IoT Hub를 사용하여 원격으로 디바이스 모니터링 및 제어'
    module: '모듈 8: 디바이스 관리'
---

# Azure IoT Hub를 사용하여 원격으로 디바이스 모니터링 및 제어

## 랩 시나리오

Contoso는 수상 경력에 빛나는 치즈를 자랑스럽게 생각하며 전체 제조 공정에서 완벽한 온도와 습도를 유지하기 위해 주의를 기울이고 있지만 숙성 과정에서의 조건은 항상 특별한 주의를 기울였습니다.

최근 몇 년 동안 Contoso는 환경 센서를 사용하여 치즈를 숙성시키는 천연 동굴의 상태를 기록하고 이 데이터를 사용하여 거의 완벽한 환경을 식별했습니다. 가장 성공적인(수상 경력) 위치의 데이터는 치즈 숙성에 가장 이상적인 온도가 섭씨 약 10도 +/- 2.8도임을 알려줍니다. 최대 포화율로 측정되는 이상적인 습도 값은 약 85% +/- 10%입니다.

이러한 이상적인 온도와 습도 값은 대부분의 치즈 유형에 적합합니다. 그러나, 특별히 딱딱하거나 특별히 부드러운 치즈에는 약간의 조정이 필요합니다. 치즈 껍질에 필요한 조건 등 특정 결과를 얻기 위해 숙성 공정의 중요한 시기/단계에서 환경을 조정해야 합니다.

Contoso는 일년 내내 이상적인 조건을 자연스럽게 유지하는 치즈 동굴(특정 지역)을 운영할 수 있을 만큼 운이 좋습니다. 그러나 이러한 곳에서도 숙성하는 동안 환경을 관리하는 것이 중요합니다. 또한, 자연 동굴에는 종종 환경이 약간 다른 여러 공간이 있습니다. 치즈 품종은 특정 요구 사항에 맞는 방(영역)에 배치됩니다. Contoso는 환경 조건을 원하는 한도 내에서 유지하기 위해 온도와 습도를 모두 제어하는 공기 처리/조절 시스템을 사용합니다.

현재, 작업자는 동굴 각 구역의 환경 조건을 모니터링하고, 원하는 온도와 습도를 유지하기 위해 필요한 경우 공기 처리 시스템 설정을 조정하고 있습니다. 운영자는 각 구역을 방문하여 4시간마다 환경 조건을 확인할 수 있습니다. 낮 최고 온도와 밤 최저 온도 사이에 온도가 급격히 변하는 곳에서는 조건이 원하는 한계를 벗어날 수 있습니다.

Contoso로부터 동굴 환경을 제어 범위 내에서 유지하는 자동 시스템을 구현하는 업무를 부여받았습니다.

이 랩에서는 IoT 디바이스를 구현하는 치즈 동굴 모니터링 시스템을 프로토타이핑합니다. 각 디바이스에는 온도 및 습도 센서가 장착되어 있으며 디바이스가 있는 영역의 온도와 습도를 제어하는 공기 처리 시스템에 연결됩니다.

### 간소화된 랩 조건

원격 분석 출력의 빈도는 프로덕션 솔루션에서 중요한 고려 사항입니다. 냉장 디바이스의 온도 센서는 1분에 한 번만 보고해야 하는 반면, 항공기의 가속도 센서는 초당 10회 보고해야 할 수 있습니다. 경우에 따라, 원격 분석을 전송해야 하는 빈도는 현재 조건에 따라 달라집니다. 예를 들어, 치즈 동굴 시나리오의 온도가 밤에 빠르게 떨어지는 경향이 있는 경우, 일몰 2시간 전부터 센서를 더 자주 판독하는 것이 도움이 될 수 있습니다. 물론 원격 분석 빈도를 변경해야 하는 요구 사항은 예측 가능한 패턴의 일부가 될 필요는 없으며, IoT 디바이스 설정을 변경해야 하는 이벤트는 예측할 수 없습니다.

이 랩에서는 간소하게 진행하기 위해 다음과 같은 가정을 합니다.

* 이 디바이스는 몇 초마다 원격 분석(온도 및 습도 값)을 IoT Hub로 전송합니다. 이 빈도는 치즈 동굴의 경우 비현실적이지만, 매 15분이 아닌 자주 변화를 확인해야 할 경우의 랩 환경에는 적합합니다.
* 공기 처리 시스템은 세 가지 상태 즉, 켜기, 끄기 또는 실패 중 하나에 있는 팬입니다.
  * 팬은 꺼진 상태로 초기화됩니다.
  * 팬 전력은 IoT 디바이스에서 직접 메서드를 사용하여 제어(켜기/끄기)됩니다.
  * 디바이스 쌍의 필요한 속성 값은 팬의 필요한 상태를 설정하는 데 사용됩니다. 원하는 속성 값은 팬/디바이스의 기본 설정을 재정의합니다.
  * 팬을 켜고 끄면 온도를 제어할 수 있습니다(팬을 켜면 온도가 낮아집니다).

이 랩의 코딩은 세 부분(원격 분석의 송수신, 직접 메서드를 호출 및 실행, 디바이스 쌍 속성의 설정 및 읽기)으로 나누어져 있습니다.

두 가지 앱(하나는 디바이스에서 원격 분석을 전송하는 용도, 다른 하나는 클라우드에서 실행하는 백 엔드 서비스용)을 작성하여 원격 분석을 수신하는 것부터 시작합니다.

다음 리소스가 만들어집니다.

![랩 15 아키텍처](media/LAB_AK_15-architecture.png)

## 랩 내용

이 랩에서는 다음 활동을 완료할 예정입니다.

* 랩 필수 구성 요소가 충족되는지 확인(필요한 Azure 리소스가 있음)

    * 필요한 경우 스크립트를 실행하여 IoT Hub 만들기
    * 스크립트를 실행하여 이 랩에 필요한 새 디바이스 ID 만들기

* IoT Hub에 디바이스 원격 분석을 보내는 시뮬레이션된 디바이스 앱 만들기
* 원격 분석을 수신 대기하는 백 엔드 서비스 앱 만들기
* IoT 디바이스에 설정을 전달하기 위한 직접 메서드 구현
* IoT 디바이스 속성 관리용 디바이스 쌍 기능 구현

## 랩 지침

### 연습 1: 랩 필수 구성 요소 확인

이 랩에서는 다음과 같은 Azure 리소스를 사용할 수 있다고 가정합니다.

| 리소스 유형 | 리소스 이름 |
| :-- | :-- |
| 리소스 그룹 | rg-az220 |
| IoT Hub | iot-az220-training-{your-id} |
| IoT 디바이스 | sensor-th-0055 |

> **중요**: 설정 스크립트를 실행하여 필요한 디바이스를 만드세요.

누락된 리소스와 새 디바이스를 만들려면 연습 2를 진행하기 전에 아래 설명에 따라 **lab15-setup.azcli** 스크립트를 실행해야 합니다. 스크립트 파일은 개발자 환경 구성(랩 3)의 일부로 로컬로 복제한 GitHub 리포지토리에 포함됩니다.

**lab15-setup.azcli** 스크립트는 **bash** 셸 환경에서 실행되도록 작성됩니다. 이는 Azure Cloud Shell에서 실행할 수 있는 가장 쉬운 방법입니다.

>**참고:** **sensor-th-0055** 디바이스용 연결 문자열이 필요합니다. 이 디바이스가 Azure IoT Hub에 이미 등록된 경우, Azure Cloud Shell에서 다음 명령을 실행하여 연결 문자열을 가져올 수 있습니다."
>
> ```bash
> az iot hub device-identity connection-string show --hub-name iot-az220-training-{your-id} --device-id sensor-th-0055 -o tsv
> ```

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
          * 15-Azure IoT Hub를 사용하여 원격으로 디바이스 모니터링 및 제어
            * Setup

    lab15-setup.azcli 스크립트 파일은 랩 15의 Setup 폴더에 있습니다.

1. **lab15-setup.azcli** 파일을 선택한 다음 **열기**를 클릭합니다.

    파일 업로드가 완료되면 알림이 표시됩니다.

1. Azure Cloud Shell에 올바른 파일이 업로드되었는지 확인하려면 다음 명령을 입력합니다.

    ```bash
    ls
    ```

    `ls` 명령으로 현재 디렉터리의 내용을 나열합니다. lab15-setup.azcli 파일이 나열됩니다.

1. 설치 스크립트가 포함된 이 랩에 대한 디렉터리를 만든 다음 해당 디렉터리로 이동하려면 다음 Bash 명령을 입력합니다.

    ```bash
    mkdir lab15
    mv lab15-setup.azcli lab15
    cd lab15
    ```

1. **lab15-setup.azcli**에 실행 권한이 있는지 확인하려면 다음 명령을 입력합니다.

    ```bash
    chmod +x lab15-setup.azcli
    ```

1. Cloud Shell 도구 모음에서 lab15-setup.azcli 파일에 액세스할 수 있도록 설정하려면 **편집기 열기**(오른쪽에서 두 번째 단추 - **{}**)를 클릭합니다.

1. lab15 폴더를 펼치고 스크립트 파일을 열려면 **파일** 목록에서 **lab15**을 클릭한 다음 **lab15-setup.azcli**를 클릭합니다.

    이제 편집기에서 **lab15-setup.azcli** 파일의 내용을 표시합니다.

1. 편집기에서 `{your-id}` 및 `{your-location}`에 할당된 값을 업데이트합니다.

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
    ./lab15-setup.azcli
    ```

    이 스크립트를 실행하는 데 몇 분이 걸릴 수 있습니다. 각 단계가 완료될 때 출력이 표시됩니다.

    이 스크립트는 먼저 **rg-az220** 리소스 그룹과 **iot-az220-training-{your-id}** IoT Hub를 만듭니다. 이미 있는 경우 해당 메시지가 표시됩니다. 그런 다음 스크립트는 ID가 **sensor-th-0055**인 디바이스를 IoT Hub에 추가하고 디바이스 연결 문자열을 표시합니다.

1. 스크립트가 완료되면 IoT Hub 및 디바이스와 관련된 정보가 표시됩니다.

    스크립트는 다음과 유사한 정보를 표시합니다.

    ```text
    Configuration Data:
    ------------------------------------------------
    iot-az220-training-{your-id} Service connectionstring:
    HostName=iot-az220-training-{your-id}.azure-devices.net;SharedAccessKeyName=iothubowner;SharedAccessKey=nV9WdF3Xk0jYY2Da/pz2i63/3lSeu9tkW831J4aKV2o=

    sensor-th-0055 device connection string:
    HostName=iot-az220-training-{your-id}.azure-devices.net;DeviceId=sensor-th-0055;SharedAccessKey=TzAzgTYbEkLW4nWo51jtgvlKK7CUaAV+YBrc0qj9rD8=

    iot-az220-training-{your-id} eventhub endpoint:
    sb://iothub-ns-iot-az220-training-2610348-5a463f1b56.servicebus.windows.net/

    iot-az220-training-{your-id} eventhub path:
    iot-az220-training-{your-id}

    iot-az220-training-{your-id} eventhub SaS primarykey:
    tGEwDqI+kWoZroH6lKuIFOI7XqyetQHf7xmoSf1t+zQ=
    ```

1. 이 랩의 후반부에서 사용할 수 있도록 스크립트로 표시된 출력을 텍스트 문서에 복사합니다.

    쉽게 찾을 수 있는 위치에 정보를 저장하면 랩을 계속할 수 있습니다.

### 연습 2: 원격 분석을 보내고 받을 코드 작성

이 연습에서는 IoT Hub로 원격 분석을 보내는 시뮬레이션된 디바이스 앱(sensor-th-0055 디바이스용)을 만듭니다.

#### 작업 1: 원격 분석을 생성하는 시뮬레이션된 디바이스 열기

1. **Visual Studio Code**를 엽니다.

1. **파일** 메뉴에서 **폴더 열기**를 클릭합니다.

1. 폴더 열기 대화 상자에서 랩 15 Starter 폴더로 이동합니다.

    _랩 3: 개발 환경 설정_, ZIP 파일을 다운로드하고 콘텐츠를 로컬로 추출하여 랩 리소스를 포함하는 GitHub 리포지토리를 복제했습니다. 추출된 폴더 구조에는 다음 폴더 경로가 포함됩니다.

    * 모든 파일
        * 랩
            * 15-Azure IoT Hub를 사용하여 원격으로 디바이스 모니터링 및 제어
                * Starter
                    * cheesecavedevice
                    * CheeseCaveOperator

1. **cheesecavedevice**를 클릭하고 **폴더 선택**을 클릭합니다.

    Visual Studio Code의 EXPLORER 창에 다음 파일이 나열되어야 합니다.

    * cheesecavedevice.csproj
    * Program.cs

1. 코드 파일을 열려면 **Program.cs**를 클릭합니다.

    이 애플리케이션은 이전 랩의 작업에서 사용했던 시뮬레이션된 디바이스 애플리케이션과 매우 비슷함을 쉽게 확인할 수 있습니다. 이 버전은 대칭 키 인증을 사용하며 원격 분석 및 로깅 메시지를 IoT Hub로 전송합니다. 그리고 이전 랩의 애플리케이션보다 더 복잡한 센서가 구현되어 있습니다.

1. **터미널** 메뉴에서 **새 터미널**을 클릭합니다.

    명령 프롬프트의 일부로 표시된 디렉터리 경로를 확인합니다. 이전 랩 프로젝트의 폴더 구조 내에서 이 프로젝트를 빌드하지 않으려고 합니다.

1. 터미널 명령 프롬프트에서 애플리케이션 빌드를 확인하려면 다음 명령을 입력합니다.

    ```bash
    dotnet build
    ```

    그러면 다음과 같은 출력이 표시됩니다.

    ```text
    > dotnet build
    Microsoft (R) Build Engine version 16.5.0+d4cbfca49 for .NET Core
    Copyright (C) Microsoft Corporation. All rights reserved.

    Restore completed in 39.27 ms for D:\Az220-Code\AllFiles\Labs\15-Remotely monitor and control devices with Azure IoT Hub\Starter\CheeseCaveDevice\CheeseCaveDevice.csproj.
    CheeseCaveDevice -> D:\Az220-Code\AllFiles\Labs\15-Remotely monitor and control devices with Azure IoT Hub\Starter\CheeseCaveDevice\bin\Debug\netcoreapp3.1\CheeseCaveDevice.dll

    Build succeeded.
        0 Warning(s)
        0 Error(s)

    Time Elapsed 00:00:01.16
    ```

다음 작업에서 연결 문자열을 구성하고 애플리케이션을 검토합니다.

#### 작업 2: 연결 구성 및 코드 검토

이 작업에서 빌드하는 시뮬레이션된 디바이스 앱은 온도와 습도를 모니터링하는 IoT 디바이스를 시뮬레이트합니다. 앱은 2초마다 센서 판독값을 시뮬레이트하고 센서 데이터를 전달합니다.

1. **Visual Studio Code**에서 Program.cs 파일이 열려 있는지 확인합니다.

1. 코드 편집기에서 다음 코드 줄을 찾습니다.

    ```csharp
    private readonly static string deviceConnectionString = "<your device connection string>";
    ```

1. **\<디바이스 연결 문자열\>** 은 앞에서 저장한 디바이스 연결 문자열로 바꿉니다.

    IoT Hub로 원격 분석을 보내기 전에 구현해야 하는 변경은 이 항목뿐입니다.

1. **파일** 메뉴에서 **저장**을 클릭합니다.

1. 애플리케이션의 구조를 잠시 검토합니다.

    애플리케이션 구조는 이전 랩에서 사용했던 구조와 비슷합니다.

    * using 문
    * 네임스페이스 정의
      * Program 클래스 - Azure IoT에 연결하여 원격 분석을 전송합니다.
      * CheeseCaveSimulator 클래스 - (EnvironmentSensor 대신 사용됨) 이 클래스는 원격 분석만 생성하는 것이 아니라 실행 중인 치즈 저장고 환경도 시뮬레이트합니다. 이 환경의 상태는 냉각 팬 작동 상태에 따라 달라집니다.
      * ConsoleHelper - 각기 다른 색의 텍스트를 콘솔에 쓰는 코드를 캡슐화하는 클래스입니다.

1. **Main** 메서드를 검토합니다.

    ```csharp
    private static void Main(string[] args)
    {
        ConsoleHelper.WriteColorMessage("Cheese Cave device app.\n", ConsoleColor.Yellow);

        // MQTT 프로토콜을 사용하여 IoT Hub에 연결합니다.
        deviceClient = DeviceClient.CreateFromConnectionString(deviceConnectionString, TransportType.Mqtt);

        // Cheese Cave 시뮬레이터의 인스턴스를 만듭니다.
        cheeseCave = new CheeseCaveSimulator();

        // INSERT 이 주석 아래에 직접 메서드 코드를 등록합니다.

        // INSERT 이 주석 아래에 원하는 속성의 변경된 처리기 코드를 등록합니다.

        SendDeviceToCloudMessagesAsync();
        Console.ReadLine();
    }
    ```

    이전 랩에서와 마찬가지로 이번에도 **Main** 메서드를 사용하여 IoT Hub로의 연결을 설정합니다. 그리고 여기서는 Main 메서드를 사용해 디바이스 쌍 속성 변경 내용을 통합하며, 직접 메서드도 통합합니다.

1. **SendDeviceToCloudMessagesAsync** 메서드를 잠시 살펴봅니다.

    이 메서드는 이전 랩에서 만들었던 버전과 매우 비슷합니다.

1. **CheeseCaveSimulator** 클래스를 살펴봅니다.

   이 클래스는 이전 랩에서 사용했던 **EnvironmentSensor** 클래스의 개선된 버전입니다. 이전 버전과의 가장 큰 차이점은 팬이 포함되었다는 것입니다. 팬이 **On** 상태이면 온도와 습도가 필요한 값과 점차 가까워집니다. 반면 팬이 **Off**(또는 **Failed**) 상태이면 온도와 습도 값은 주변 값과 가까워집니다. 온도를 읽을 때 팬이 **Failed** 상태로 설정되어 있을 가능성은 1%입니다.

#### 작업 3: 원격 분석 전송 코드 테스트

1. Visual Studio Code에서 터미널이 아직 열려 있는지 확인합니다.

1. 시뮬레이션된 디바이스 앱을 실행하려면 터미널 명령 프롬프트에서 다음 명령을 입력합니다.

    ```bash
    dotnet run
    ```

   이 명령은 현재 폴더에서 **Program.cs** 파일을 실행합니다.

1. 터미널로 전송되는 출력을 확인합니다.

    다음과 유사한 콘솔 출력을 빠르게 확인해야 합니다.

    ![콘솔 출력](media/LAB_AK_15-cheesecave-telemetry.png)

    > **참고**:  녹색 텍스트는 실행이 정상적으로 완료되었음을 나타냅니다. 빨간색 텍스트는 문제가 발생했음을 나타냅니다. 위의 이미지와 비슷한 화면이 표시되지 않으면 디바이스 연결 문자열부터 확인합니다.

1. 이 앱을 실행 상태로 둡니다.

    이 랩의 후반부에 IoT Hub로 원격 분석을 보내야 합니다.

### 연습 3: 원격 분석을 받을 두 번째 앱 만들기

이제 IoT Hub로 원격 분석을 보내는 시뮬레이션된 치즈 저장고 디바이스가 있으므로 IoT Hub에 연결하고 해당 원격 분석을 "수신 대기"할 수 있는 백 엔드 앱을 만들어야 합니다. 결국 이 백 엔드 앱은 치즈 동굴의 온도를 자동으로 제어하는 데 사용됩니다.

#### 작업 1: 원격 분석을 수신할 앱 만들기

이 작업에서는 IoT Hub 이벤트 허브 엔드포인트에서 원격 분석을 수신하는 데 사용할 백 엔드 앱 만들기 작업을 시작합니다.

1. Visual Studio Code의 새 인스턴스를 엽니다.

    이미 열려 있는 Visual Studio Code 창에서 시뮬레이션된 디바이스 앱이 실행 중이므로 백 엔드 앱에 대한 Visual Studio Code의 새 인스턴스가 필요합니다.

1. **파일** 메뉴에서 **폴더 열기**를 클릭합니다.

1. **폴더 열기** 대화 상자에서 랩 15 Starter 폴더로 이동합니다.

1. **CheeseCaveOperator**를 클릭하고 **폴더 선택**을 클릭합니다.

    미리 준비되어 있는 CheeseCaveOperator 애플리케이션은 코드 작성 프로세스를 안내하는 데 사용되는 몇 가지 주석과 NuGet 패키지 라이브러리 두 개가 포함되어 있는 간단한 콘솔 애플리케이션입니다. 애플리케이션을 실행하려면 프로젝트에 코드 블록을 추가해야 합니다.

1. **탐색기** 창에서 애플리케이션 프로젝트 파일을 열려면 **CheeseCaveOperator.csproj**를 클릭합니다.

    코드 편집기 창에서 **CheeseCaveOperator.cs** 파일이 열립니다.

1. **CaveDevice.csproj** 파일의 내용을 잠시 검토합니다.

    파일 내용은 다음과 같습니다.

    ```xml
    <Project Sdk="Microsoft.NET.Sdk">

    <PropertyGroup>
        <OutputType>Exe</OutputType>
        <TargetFramework>netcoreapp3.1</TargetFramework>
    </PropertyGroup>

    <ItemGroup>
        <PackageReference Include="Microsoft.Azure.Devices" Version="1.*" />
        <PackageReference Include="Microsoft.Azure.EventHubs" Version="4.*" />
    </ItemGroup>

    </Project>
    ```

    > **참고**: 파일의 패키지 버전 번호가 위에 나와 있는 것과 다를 수도 있는데, 달라도 괜찮습니다.

    프로젝트 파일(.csproj)은 작업 중인 프로젝트 유형을 지정하는 XML 문서입니다. 이 랩의 프로젝트는 **Sdk** 스타일 프로젝트입니다.

    위의 코드에 나와 있듯이 프로젝트 정의에는 **PropertyGroup**과 **ItemGroup**의 2개 섹션이 포함되어 있습니다.

    **PropertyGroup**은 이 프로젝트를 빌드하면 생성되는 출력 유형을 정의합니다. 여기서는 .NET Core 3.1을 대상으로 하는 실행 파일을 작성합니다.

    **ItemGroup**은 애플리케이션에 필요한 외부 라이브러리를 지정합니다. 위 코드에 포함된 참조는 NuGet 패키지 참조이며, 각 패키지 참조에는 패키지 이름과 버전이 지정되어 있습니다.

    > **참고**: 명령 프롬프트(예: Visual Studio Code 터미널 명령 프롬프트)에서 `dotnet add package` 명령을 입력하여 프로젝트 파일에 NuGet 라이브러리(예: 위의 ItemGroup에 나와 있는 라이브러리)를 수동으로 추가할 수 있습니다. `dotnet restore` 명령을 입력하면 모든 종속성이 다운로드됩니다. 예를 들어 위의 라이브러리를 로드하고 코드 프로젝트에서 사용할 수 있도록 설정하려는 경우 다음 명령을 입력하면 됩니다.
    >
    >   dotnet add package Microsoft.Azure.EventHubs
    >   dotnet add package Microsoft.Azure.Devices
    >   dotnet restore
    >
    > **정보**: [여기](https://docs.microsoft.com/ko-kr/nuget/what-is-nuget)서 NuGet에 대해 자세히 알아볼 수 있습니다.

#### 작업 3: 원격 분석 수신기 추가

1. **탐색기** 창에서 **Program.cs**를 클릭합니다.

    **Program.cs** 파일은 다음과 같습니다.

    ```csharp
    // Copyright (c) Microsoft. All rights reserved.
    // MIT 라이선스에 따라 라이선스가 부여되었습니다. 전체 라이선스 정보는 프로젝트 루트의 라이선스 파일을 참조하십시오.

    // INSERT 이 주석 아래에 using 문을 삽입합니다.

    namespace CheeseCaveOperator
    {
        class Program
        {
            // INSERT 이 주석 아래에 변수를 삽입합니다.

            // INSERT 이 주석 아래에 Main 메서드를 삽입합니다.

            // INSERT 이 주석 아래에 ReceiveMessagesFromDeviceAsync 메서드를 삽입합니다.

            // INSERT 이 주석 아래에 InvokeMethod 메서드를 삽입합니다.

            // INSERT 이 주석 아래에 디바이스 쌍 섹션을 삽입합니다.
        }

        internal static class ConsoleHelper
        {
            internal static void WriteColorMessage(string text, ConsoleColor clr)
            {
                Console.ForegroundColor = clr;
                Console.WriteLine(text);
                Console.ResetColor();
            }
            internal static void WriteGreenMessage(string text)
            {
                WriteColorMessage(text, ConsoleColor.Green);
            }

            internal static void WriteRedMessage(string text)
            {
                WriteColorMessage(text, ConsoleColor.Red);
            }
        }
    }
    ```

    이 코드는 Operator 앱 구조를 대략적으로 보여 줍니다.

1. `// INSERT using statements below here` 주석을 찾습니다.

1. 애플리케이션 코드가 사용하도록 할 네임스페이스를 지정하려면 다음 코드를 입력합니다.

    ```csharp
    using System;
    using System.Threading.Tasks;
    using System.Text;
    using System.Collections.Generic;
    using System.Linq;

    using Microsoft.Azure.EventHubs;
    using Microsoft.Azure.Devices;
    using Newtonsoft.Json;
    ```

    위의 코드에서는 **System**을 지정하는 동시에 코드가 사용하도록 할 다른 네임스페이스도 선언했습니다. 구체적으로는 문자열 인코딩용 **System.Text**, 비동기 작업용 **System.Threading.Tasks**, 그리고 앞에서 추가했던 2개 패키지용 네임스페이스를 선언했습니다.

    > **팁**: 코드를 삽입할 때 코드 레이아웃이 적절하지 않은 경우에는 코드 편집기 창을 마우스 오른쪽 단추로 클릭하고 문서 서식을 클릭하면 Visual Studio Code에서 문서 서식을 자동으로 지정합니다. 작업 창을 열고(**F1** 키 누르기) 문서 서식을 입력한 다음 Enter 키를 눌러도 됩니다. Windows에서 이 작업을 수행하는 바로 가기 키는 Shift+Alt+F입니다.

1. `// INSERT variables below here` 주석을 찾습니다.

1. 프로그램에서 사용 중인 변수를 지정하려면 다음 코드를 입력합니다.

    ```csharp
    // 전역 변수입니다.
    // Event Hub 호환 엔드포인트입니다.
    private readonly static string eventHubsCompatibleEndpoint = "<your event hub endpoint>";

    // Event Hub 호환 이름입니다.
    private readonly static string eventHubsCompatiblePath = "<your event hub path>";
    private readonly static string iotHubSasKey = "<your event hub SaS key>";
    private readonly static string iotHubSasKeyName = "service";
    private static EventHubClient eventHubClient;

    // INSERT 이 주석 아래에 서비스 클라이언트 변수를 삽입합니다.

    // INSERT 이 주석 아래에 레지스트리 관리자 변수를 삽입합니다.

    // IoT Hub의 연결 문자열입니다.
    private readonly static string serviceConnectionString = "<your service connection string>";

    private readonly static string deviceId = "sensor-th-0055";
    ```

1. 방금 입력한 코드(및 코드 주석)를 잠시 검토합니다.

    **eventHubsCompatibleEndpoint** 변수를 사용하여 Event Hubs와 호환되는 IoT Hub 기본 제공 서비스 연결 엔드포인트(메시지/이벤트)의 URI를 저장합니다.

    **eventHubsCompatiblePath** 변수에는 이벤트 허브 엔터티의 경로가 포함됩니다.

    **iotHubSasKey** 변수에는 네임스페이스(엔터티)의 키 이름과 해당 공유 액세스 정책 규칙이 포함됩니다.

    **iotHubSasKeyName** 변수에는 네임스페이스(엔터티)의 키와 해당 공유 액세스 정책 규칙이 포함됩니다.

    **eventHubClient** 변수에는 이벤트 허브 클라이언트 인스턴스가 포함됩니다. 이 인스턴스는 IoT Hub에서 메시지를 수신하는 데 사용됩니다.

    **serviceClient** 변수에는 앱에서 IoT Hub로 메시지를 전송(한 다음 IoT Hub에서 대상 디바이스 등으로 메시지를 전송)하는 데 사용되는 서비스 클라이언트 인스턴스가 포함됩니다.

    **serviceConnectionString** 변수에는 Operator 앱이 IoT Hub에 연결하는 데 사용되는 연결 문자열이 포함됩니다.

    **deviceId** 변수에는 **CheeseCaveDevice** 애플리케이션이 사용하는 디바이스 ID가 포함됩니다.

1. 서비스 연결 문자열을 할당하는 데 사용되는 코드 줄 찾기

    ```csharp
    private readonly static string serviceConnectionString = "<your service connection string>";
    ```

1. **\<서비스 연결 문자열\>** 은 이 랩 앞부분에서 저장한 IoT Hub 서비스 연결 문자열로 바꿉니다.

    이 문자열은 연습 1에서 실행했던 lab15-setup.azcli 설정 스크립트에서 생성되어 저장해 두었던 iothubowner 공유 액세스 정책 기본 연결 문자열입니다.

    > **참고**: **서비스** 공유 정책 대신 **iothubowner** 공유 정책을 사용하는 이유가 궁금할 수 있습니다. 대답은 각 정책에 할당된 IoT Hub 사용 권한과 관련이 있습니다. **서비스** 정책에는 **ServiceConnect** 권한이 있으며 일반적으로 백 엔드 클라우드 서비스에서 사용됩니다. 다음과 같은 권한을 부여합니다.
    >
    > * 클라우드 서비스 지향 통신 및 모니터링 중인 엔드포인트에 대한 액세스를 부여합니다.
    > * 디바이스-클라우드 메시지를 받고 클라우드-디바이스 메시지를 보내며 해당 전달 승인을 검색할 권한을 부여합니다.
    > * 파일 업로드에 대한 전달 승인을 검색할 수 있는 권한을 부여합니다.
    > * 태그 및 원하는 속성을 업데이트하고, 보고된 속성을 검색하고, 쿼리를 실행하기 위해 쌍에 액세스할 권한을 부여합니다.
    >
    > **serviceoperator** 애플리케이션이 팬 상태를 전환하는 직접 메서드를 호출하는 랩의 첫 번째 부분의 경우 **서비스** 정책에 충분한 권한이 있습니다. 그러나 이 랩의 뒷부분에서는 디바이스 레지스트리를 쿼리합니다. 이 쿼리에서는 **RegistryManager** 클래스를 사용합니다. **RegistryManager** 클래스를 사용하여 디바이스 레지스트리를 쿼리하려면 IoT Hub에 연결하는 데 사용되는 공유 액세스 정책에 **레지스트리 읽기** 권한이 있어야 하며, 이 권한은 다음과 같습니다.
    >
    > * ID 레지스트리에 대한 읽기 액세스 권한을 부여합니다.
    >
    > **레지스트리 쓰기** 권한이 부여된 **iothubowner** 정책은 **레지스트리 읽기** 권한을 상속하므로 이 랩의 작업을 수행하는 데 사용할 수 있습니다.
    >
    > 프로덕션 시나리오에서 **서비스 연결** 및 **레지스트리 읽기** 권한만 있는 새 공유 액세스 정책을 추가하는 것이 좋습니다.

1. **\<이벤트 허브 엔드포인트\>**, **\<이벤트 허브 경로\>** 및 **\<이벤트 허브 SaS 키\>** 는 이 랩 앞부분에서 저장한 값으로 바꿉니다.

1. `// INSERT Main method below here` 주석을 찾습니다.

1. **Main** 메서드를 구현하려면 다음 코드를 입력합니다.

    ```csharp
    public static void Main(string[] args)
    {
        ConsoleHelper.WriteColorMessage("Cheese Cave Operator\n", ConsoleColor.Yellow);

        // IoT Hub Event Hubs호환 엔드포인트에 연결하기 위해 EventHubClient 인스턴스를 만듭니다.
        var connectionString = new EventHubsConnectionStringBuilder(new Uri(eventHubsCompatibleEndpoint), eventHubsCompatiblePath, iotHubSasKeyName, iotHubSasKey);
        eventHubClient = EventHubClient.CreateFromConnectionString(connectionString.ToString());

        // 허브의 각 파티션에 대해 PartitionReceiver를 만듭니다.
        var runtimeInfo = eventHubClient.GetRuntimeInformationAsync().GetAwaiter().GetResult();
        var d2cPartitions = runtimeInfo.PartitionIds;

        // INSERT 이 주석 아래에 원하는 속성의 변경된 처리기 코드를 등록합니다.

        // INSERT 이 주석 아래에서 서비스 클라이언트 인스턴스를 만듭니다.

        // 메시지를 수신하는 수신기를 만듭니다.
        var tasks = new List<Task>();
        foreach (string partition in d2cPartitions)
        {
            tasks.Add(ReceiveMessagesFromDeviceAsync(partition));
        }

        // 모든 PartitionReceivers가 완료될 때까지 기다립니다.
        Task.WaitAll(tasks.ToArray());
    }
    ```

1. 방금 입력한 코드(및 코드 주석)를 잠시 검토합니다.

    위의 코드에서는 **EventHubsConnectionStringBuilder** 클래스를 사용하여 **EventHubClient** 연결 문자열을 생성합니다. 이 클래스는 여러 값을 연결하여 올바른 형식으로 만드는 도우미 클래스입니다. 이렇게 생성된 연결 문자열을 사용하여 이벤트 허브 엔드포인트에 연결하고 **eventHubClient** 변수에 값을 입력합니다.

    그리고 나면 **eventHubClient**를 사용하여 이벤트 허브의 런타임 정보를 검색합니다. 이 정보에는 다음 항목이 포함됩니다.

    * **CreatedAt** - 이벤트 허브를 만든 날짜/시간
    * **PartitionCount** - 파티션 수(대다수 IoT Hub는 파티션 4개로 구성됨)
    * **PartitionIds** - 파티션 ID가 포함된 문자열 배열
    * **Path** - 이벤트 허브 엔터티 경로

    파티션 ID 배열은 **d2cPartitions** 변수에 저장됩니다. 잠시 후에 이 변수를 사용하여 각 파티션에서 메시지를 수신하는 작업 목록을 만들겠습니다.

    > **정보**: [여기](https://docs.microsoft.com/ko-kr/azure/iot-hub/iot-hub-scaling#partitions)서 파티션의 용도에 대해 자시히 알아볼 수 있습니다.

    디바이스에서 IoT Hub로 전송되는 메시지는 어떤 파티션에서나 처리될 수 있으므로, 앱은 각 파티션에서 메시지를 검색해야 합니다. 다음 코드 섹션에서는 비동기 작업 목록을 만듭니다. 각 작업에서는 특정 파티션의 메시지를 수신합니다. 마지막 코드 줄은 모든 작업이 완료될 때까지 대기하는 부분입니다. 각 작업은 무한 루프에 포함되므로, 이 줄을 추가해야 애플리케이션이 종료되지 않습니다.

1. `// INSERT 이 주석 아래에 ReceiveMessagesFromDeviceAsync 메서드를 삽입합니다.` 주석을 찾습니다.

1. **ReceiveMessagesFromDeviceAsync** 메서드를 구현하려면 다음 코드를 입력합니다.

    ```csharp
    // 비동기적으로 파티션에 대한 PartitionReceiver를 만든 다음 시뮬레이션된 클라이언트에서 보낸 메시지를 읽기 시작합니다.
    private static async Task ReceiveMessagesFromDeviceAsync(string partition)
    {
        // 기본 소비자 그룹을 사용하여 받는 사람을 만듭니다.
        var eventHubReceiver = eventHubClient.CreateReceiver("$Default", partition, EventPosition.FromEnqueuedTime(DateTime.Now));
        Console.WriteLine("Created receiver on partition: " + partition);

        while (true)
        {
            // EventData 확인 - 검색할 항목이 없으면 이 메서드가 시간 초과됩니다.
            var events = await eventHubReceiver.ReceiveAsync(100);

            // 일괄 처리에 데이터가 있는 경우 처리합니다.
            if (events == null) continue;

            foreach (EventData eventData in events)
            {
                string data = Encoding.UTF8.GetString(eventData.Body.Array);

                ConsoleHelper.WriteGreenMessage("Telemetry received: " + data);

                foreach (var prop in eventData.Properties)
                {
                    if (prop.Value.ToString() == "true")
                    {
                        ConsoleHelper.WriteRedMessage(prop.Key);
                    }
                }
                Console.WriteLine();
            }
        }
    }
    ```

    위의 코드에 나와 있는 것처럼, 이 메서드를 입력할 때는 대상 파티션을 정의하는 인수를 포함해야 합니다. 파티션 4개가 지정된 기본 구성에서는 이 메서드가 4번 호출되며, 각 메서드는 각 파티션에 대해 한 번씩 비동기식으로 병렬 실행됩니다.

    이 메서드의 첫 코드 부분은 이벤트 허브 수신기를 만드는 코드입니다. 이 코드는 수신기를 만들 때 **$Default** 소비자 그룹(일반적으로는 사용자 지정 소비자 그룹을 작성함)을 사용함을 지정합니다. 그 다음에는 파티션을 지정하며, 마지막으로 이벤트 파티션 데이터에서 수신을 시작할 위치를 지정합니다. 여기서 수신기는 현재 시간부터 큐에 추가되는 메시지만 수신합니다. 데이터 스트림 시작/종료 위치나 특정 오프셋을 제공할 수 있는 다른 옵션도 있습니다.

    > **정보**: [여기](https://docs.microsoft.com/ko-kr/azure/event-hubs/event-hubs-features#consumer-groups)서 소비자 그룹에 대해 자세히 알아볼 수 있습니다.

    수신기가 작성되면 앱은 무한 루프를 시작하여 이벤트가 수신될 때까지 대기합니다.

    > **참고**: `eventHubReceiver.ReceiveAsync(100)` 코드는 한 번에 수신 가능한 최대 이벤트 수를 지정합니다. 그러나 실제로는 이렇게 많은 이벤트가 수신될 때까지 대기하지 않으며 이벤트 하나 이상이 사용 가능해지는 즉시 결과를 반환합니다. 이벤트가 반환되지 않으면(시간 초과로 인해) 루프는 계속 실행되며 코드는 이벤트가 더 수신될 때까지 대기합니다.

    이벤트가 하나 이상 수신되면 각 이벤트 데이터 본문이 바이트 배열에서 문자열로 변환되어 콘솔에 작성됩니다. 그러면 이벤트 데이터 속성에 대해 코드가 반복 실행되어 값이 true인지를 확인합니다. 현재 시나리오에서는 true 값은 이벤트가 경고임을 나타냅니다. 확인된 경고는 콘솔에 작성됩니다.

1. **파일** 메뉴에서 변경 내용을 Program.cs 파일에 저장하려면 **저장**을 클릭합니다.

#### 작업 3: 원격 분석을 수신하도록 코드 테스트

백 엔드 앱이 시뮬레이트된 디바이스에서 전송되는 원격 분석을 선택하는지 여부를 확인하려면 이 테스트가 중요합니다. 디바이스 앱은 계속 실행되고 원격 분석을 보내고 있습니다.

1. 터미널에서 **CheeseCaveOperator** 백 엔드 앱을 실행하려면 터미널 창을 열고 다음 명령을 입력합니다.

    ```bash
    dotnet run
    ```

   이 명령은 현재 폴더에서 **Program.cs** 파일을 실행합니다.

   > **참고**:  사용되지 않는 변수 **serviceConnectionString** 관련 경고는 무시하면 됩니다. 잠시 후에 해당 변수를 사용하는 코드를 추가하겠습니다.

1. 잠시 시간을 내어 터미널로 출력되는 것을 관찰합니다.

    콘솔 출력을 빠르게 볼 수 있으며 IoT Hub에 성공적으로 연결되면 앱에 원격 분석 메시지 데이터가 거의 즉시 표시됩니다.

    그렇지 않은 경우 IoT Hub 서비스 연결 문자열을 주의 깊게 확인합니다. 이 문자열은 다른 문자열이 아닌 서비스 연결 문자열이어야 합니다.

    ![콘솔 출력](media/LAB_AK_15-cheesecave-telemetry-received.png)

    > **참고**:  녹색 텍스트는 정상 작동을, 빨간색 텍스트는 문제가 발생했음을 의미합니다. 이 이미지와 비슷한 화면이 표시되지 않으면 디바이스 연결 문자열부터 확인합니다.

1. 이 앱을 잠시 더 실행 상태로 둡니다.

1. 두 앱이 모두 실행 중인 상태에서 디바이스 앱에서 전송하는 원격 분석과 동기화된 원격 분석이 Operator 앱에 표시되는지 확인합니다.

    수신되는 원격 분석과 전송되는 원격 분석을 시각적으로 비교합니다.

    * 정확한 데이터 일치가 있나요?
    * 데이터를 전송하는 시간과 수신되는 시간 사이에 많은 지연이 있나요?

1. 원격 분석 데이터를 확인한 후 실행 중인 앱을 중지하고 두 VS Code 인스턴스에서 모두 터미널 창을 닫습니다. 단, Visual Studio Code 창은 닫지 마세요.

이제 디바이스에서 원격 분석을 보내는 앱과 데이터 수신을 확인하는 백 엔드 앱이 있습니다. 다음 연습에서는 제어 측면을 처리하는 단계의 작업을 시작합니다. 즉, 데이터에 문제가 발생하는 경우에 수행할 작업을 지정합니다.

### 연습 4: 직접 메서드를 호출하는 코드 작성

직접 메서드를 호출하기 위한 백 엔드 앱의 호출에는 페이로드의 일부로 여러 매개 변수가 포함될 수 있습니다. 일반적으로 직접 메서드는 디바이스의 기능을 끄고 켜거나 디바이스의 설정을 지정하는 데 사용됩니다.

Contoso 시나리오에서는 치즈 저장고에 있는 팬 작동을 제어하는 직접 메서드를 디바이스에서 구현합니다(팬을 On 또는 Off로 설정하여 온도와 습도 제어 시뮬레이트). Operator 애플리케이션이 IoT Hub와 통신하여 디바이스에서 직접 메서드를 호출합니다.

치즈 저장고 디바이스에 직접 메서드를 실행하라는 명령이 수신되면 여러 가지 오류 조건을 확인해야 합니다. 이 확인 중 하나는 팬이 실패 상태인 경우 간단히 오류에 응답하는 것입니다. 보고할 또 다른 오류 조건은 잘못된 매개 변수가 수신되는 경우입니다. 디바이스를 원격으로 사용할 가능성이 있는 경우 분명한 오류 보고가 중요합니다.

직접 메서드를 사용하려면 백 엔드 앱이 매개 변수를 준비한 후 메서드를 호출할 단일 디바이스를 지정하는 호출을 수행해야 합니다. 백 엔드 앱은 응답을 기다리고 보고합니다.

디바이스 앱은 직접 메서드의 함수형 코드를 포함합니다. 함수 이름은 디바이스의 IoT 클라이언트에 등록됩니다. 이 프로세스를 통해 클라이언트는 호출이 IoT Hub에서 들어올 때 어떤 함수를 실행할지 알 수 있습니다(많은 직접 메서드가 있을 수 있음).

이 연습에서는 치즈 동굴에서 팬을 켜는 것을 시뮬레이션하는 직접 메서드에 대한 코드를 추가하여 디바이스 앱을 업데이트합니다. 그런 다음 백 엔드 서비스 앱에 코드를 추가하여 이 직접 메서드를 호출합니다.

#### 작업 1: 디바이스 앱에서 직접 메서드를 정의하는 코드 추가

1. **cheesecavedevice** 애플리케이션이 표시되어 있는 Visual Studio Code 인스턴스로 돌아갑니다.

    > **참고**: 앱이 계속 실행 중인 경우 터미널 창을 사용하여 앱을 종료합니다(터미널 창 안을 클릭하여 포커스를 설정한 후 **Ctrl+C**를 눌러 실행 중인 애플리케이션 종료).

1. 코드 편집기에서 **Program.cs**가 열려 있는지 확인합니다.

1. `INSERT register direct method code below here` 주석을 찾습니다.

1. 직접 메서드를 등록하려면 다음 코드를 입력합니다.

    ```csharp
    // 직접 메서드 호출에 대한 처리기 만들기
    deviceClient.SetMethodHandlerAsync("SetFanState", SetFanState, null).Wait();
    ```

    **SetFanState** 직접 메서드 처리기도 이 코드를 통해 설정됩니다. 위의 코드에 나와 있는 것처럼, deviceClient의 **SetMethodHandlerAsync** 메서드는 원격 메서드 이름 `"SetFanState"`, 호출할 실제 로컬 메서드 및 사용자 컨텍스트 개체(여기서는 null)를 인수로 사용합니다.

1. `INSERT SetFanState method below here` 주석을 찾습니다.

1. **SetFanState** 직접 메서드를 구현하려면 다음 코드를 입력합니다.

    ```csharp
    // 직접 메서드 호출 처리
    private static Task<MethodResponse> SetFanState(MethodRequest methodRequest, object userContext)
    {
        if (cheeseCave.FanState == StateEnum.Failed)
        {
            // 400 오류 메시지가 있는 직접 메서드 호출을 승인합니다.
            string result = "{\"result\":\"Fan failed\"}";
            ConsoleHelper.WriteRedMessage("Direct method failed: " + result);
            return Task.FromResult(new MethodResponse(Encoding.UTF8.GetBytes(result), 400));
        }
        else
        {
            try
            {
                var data = Encoding.UTF8.GetString(methodRequest.Data);

                // 데이터에서 따옴표를 제거합니다.
                data = data.Replace("\"", "");

                // 페이로드를 구문 분석하고 유효하지 않은 경우 예외를 트리거합니다.
                cheeseCave.UpdateFan((StateEnum)Enum.Parse(typeof(StateEnum), data));
                ConsoleHelper.WriteGreenMessage("Fan set to: " + data);

                // 200 성공 메시지와 함께 직접 메서드 호출을 승인합니다.
                string result = "{\"result\":\"Executed direct method: " + methodRequest.Name + "\"}";
                return Task.FromResult(new MethodResponse(Encoding.UTF8.GetBytes(result), 200));
            }
            catch
            {
                // 400 오류 메시지가 있는 직접 메서드 호출을 승인합니다.
                string result = "{\"result\":\"Invalid parameter\"}";
                ConsoleHelper.WriteRedMessage("Direct method failed: " + result);
                return Task.FromResult(new MethodResponse(Encoding.UTF8.GetBytes(result), 400));
            }
        }
    }
    ```

    이 메서드는 관련 원격 메서드인 **SetFanState**가 IoT Hub를 통해 호출되면 디바이스에서 실행됩니다. 이 메서드는 **MethodRequest** 인스턴스를 수신할 뿐 아니라 직접 메시지 콜백 등록 시에 정의된 **userContext** 개체(여기서는 null)도 수신합니다.

    이 메서드의 첫 코드 줄은 치즈 저장고 팬이 현재 **Failed** 상태인지를 확인합니다. 치즈 저장고 시뮬레이터는 팬이 고장나면 후속 명령도 자동으로 실패한다고 가정합니다. 따라서 **result** 속성이 **Fan Failed**로 설정된 JSON 문자열이 작성됩니다. 그러면 바이트 배열로 인코딩된 결과 문자열과 HTTP 상태 코드를 사용하여 새 **MethodResponse** 개체가 생성됩니다. 위의 코드에서는 상태 코드로 **400**이 사용되었습니다. REST API의 컨텍스트에서 이 코드는 일반 클라이언트 쪽 오류가 발생했다는 의미입니다. **Task\<MethodResponse\>** 를 반환하려면 직접 메서드 콜백이 필요하므로, 새 작업이 작성되어 반환됩니다.

    > **정보**: REST API 내에서 HTTP 상태 코드가 사용되는 방식에 대한 자세한 내용은 [여기](https://restfulapi.net/http-status-codes/)서 확인할 수 있습니다.

    팬 상태가 **Failed**가 아니면 코드가 계속 실행되어 메서드 요청의 일부분으로 전송된 데이터를 처리합니다. **methodRequest.Data** 속성에는 데이터가 바이트 배열 형식으로 포함됩니다. 따라서 이 데이터가 먼저 문자열로 변환됩니다. 이 시나리오에서 필요한 두 값은 다음과 같습니다(따옴표 포함).

    * "On"
    * "Off"

    여기서 수신된 데이터는 다음과 같이 **StateEnum**의 멤버에 매핑된다고 가정합니다.

    ```csharp
    internal enum StateEnum
    {
        Off,
        On,
        Failed
    }
    ```

    데이터를 구문 분석하려면 먼저 따옴표를 제거한 다음 **Enum.Parse** 메서드를 사용하여 일치하는 열거 값을 찾아야 합니다. 일치하는 값을 찾지 못하면(데이터가 정확하게 일치해야 함) 예외가 throw되어 그 뒷부분의 코드에서 catch됩니다. 예외 처리기는 팬 고장 상태용으로 작성되는 것과 비슷한 오류 메서드 응답을 작성하여 반환합니다.

    **StateEnum**에 일치하는 값이 있으면 치즈 저장고 시뮬레이터 **UpdateFan** 메서드가 호출됩니다. 여기서는 해당 메서드가 **FanState** 속성을 제공된 값으로 설정하는 작업만 수행합니다. 실제 구현에서는 이 메서드가 팬과 상호 작용하여 상태를 변경하며, 상태가 정상적으로 변경되었는지를 확인합니다. 그러나 이 시나리오에서는 상태가 정상적으로 변경되었다고 가정하여 적절한 **result** 및 **MethodResponse**가 작성되어 반환됩니다. 그리고 이번에는 작업 성공을 나타내는 HTTP 상태 코드 **200**이 표시됩니다.

1. **파일** 메뉴에서 Program.cs 파일을 저장하려면 **저장**을 클릭합니다.

이제 디바이스 측에서 필요한 코딩을 완료했습니다. 다음으로는 직접 메서드를 호출하는 코드를 백 엔드 Operator 애플리케이션에 추가해야 합니다.

#### 작업 2: 직접 메서드를 호출하는 코드 추가

1. **CheeseCaveOperator** 애플리케이션이 표시되어 있는 Visual Studio Code 인스턴스로 돌아갑니다.

    > **참고**: 앱이 계속 실행 중인 경우 터미널 창을 사용하여 앱을 종료합니다(터미널 창 안을 클릭하여 포커스를 설정한 후 **Ctrl+C**를 눌러 실행 중인 애플리케이션 종료).

1. 코드 편집기에서 **Program.cs**가 열려 있는지 확인합니다.

1. `INSERT service client variable below here` 주석을 찾습니다.

1. 서비스 클라이언트 인스턴스를 저장할 전역 변수를 추가하려면 다음 코드를 입력합니다.

    ```csharp
    private static ServiceClient serviceClient;
    ```

1. `INSERT create service client instance below here` 주석을 찾습니다.

1. 서비스 클라이언트 인스턴스를 만들고 직접 메서드를 호출하는 코드를 추가하려면 다음 코드를 입력합니다.

    ```csharp
    // ServiceClient를 만들어 허브의 서비스 연결 엔드포인트와 통신합니다.
    serviceClient = ServiceClient.CreateFromConnectionString(serviceConnectionString);
    InvokeMethod().GetAwaiter().GetResult();
    ```

1. `INSERT InvokeMethod method below here` 주석을 찾습니다.

1. 직접 메서드를 호출하는 코드를 추가하려면 다음 코드를 입력합니다.

    ```csharp
    // 직접 메서드를 호출하는 핸들입니다.
    private static async Task InvokeMethod()
    {
        try
        {
            var methodInvocation = new CloudToDeviceMethod("SetFanState") { ResponseTimeout = TimeSpan.FromSeconds(30) };
            string payload = JsonConvert.SerializeObject("On");

            methodInvocation.SetPayloadJson(payload);

            // 직접 메서드를 비동기적으로 호출하고 시뮬레이션된 디바이스에서 응답을 가져옵니다.
            var response = await serviceClient.InvokeDeviceMethodAsync(deviceId, methodInvocation);

            if (response.Status == 200)
            {
                ConsoleHelper.WriteGreenMessage("Direct method invoked: " + response.GetPayloadAsJson());
            }
            else
            {
                ConsoleHelper.WriteRedMessage("Direct method failed: " + response.GetPayloadAsJson());
            }
        }
        catch
        {
            ConsoleHelper.WriteRedMessage("Direct method failed: timed-out");
        }
    }
    ```

    이 코드는 디바이스 앱에서 **SetFanState** 직접 메서드를 호출하는 데 사용됩니다.

1. **파일** 메뉴에서 Program.cs 파일을 저장하려면 **저장**을 클릭합니다.

이제 **SetFanState** 직접 메서드를 지원하기 위해 코드를 변경했습니다.

#### 작업 3: 직접 메서드 테스트

직접 메서드를 테스트하려면 올바른 순서로 앱을 시작해야 합니다. 등록되지 않은 직접 메서드를 호출할 수 없습니다!

1. **cheesecavedevice** 디바이스 앱이 표시되어 있는 Visual Studio Code 인스턴스로 전환합니다.

1. **cheesecavedevice** 디바이스 앱을 실행하려면 터미널 창을 열고 `dotnet run` 명령을 입력합니다.

    터미널에 쓰기가 시작되고 원격 분석 메시지가 표시됩니다.

1. **CheeseCaveOperator** 백 엔드 앱이 표시되어 있는 Visual Studio Code 인스턴스로 전환합니다.

1. **CheeseCaveOperator** 백 엔드 앱을 시작하려면 터미널 창을 열고 `dotnet run` 명령을 입력합니다.

    > **참고**:  `Direct method failed: timed-out` 메시지가 표시되면 **CheeseCaveDevice**의 변경 내용을 저장하고 앱을 시작했는지 다시 한 번 확인합니다.

    CheeseCaveOperator 백 엔드 앱은 직접 메서드를 즉시 호출합니다.

    다음과 유사한 출력을 확인합니다.

    ![콘솔 출력](media/LAB_AK_15-cheesecave-direct-method-sent.png)

1. 이제 **cheesecavedevice** 디바이스 앱에 대한 콘솔 출력을 확인합니다. 팬은 켜져 있어야 합니다.

   ![콘솔 출력](media/LAB_AK_15-cheesecave-direct-method-received.png)

이제 원격 디바이스를 성공적으로 모니터링 및 제어하고 있습니다. 클라우드에서 호출할 수 있는 디바이스에서 직접 메서드를 구현했습니다. Contoso 시나리오에서는 직접 메서드를 사용하여 팬을 켜고 동굴의 환경을 원하는 대로 설정합니다. 온도 및 습도 판독값이 계속 낮아져 최종적으로는 경고가 제거됩니다(팬이 고장나지 않는다면).

그런데 치즈 저장고 환경에 대해 원하는 설정을 원격으로 지정하려는 경우도 있습니다. 아마도 숙성 과정의 특정 시점에 치즈 동굴에 대한 특정 목표 온도를 설정하고 싶을 것입니다. 직접 메서드를 사용하여 원하는 설정을 지정하거나(올바른 방법), 이 용도로 설계된 IoT Hub의 다른 기능인 디바이스 쌍을 사용할 수 있습니다. 다음 연습에서는 솔루션 내에서 디바이스 쌍 속성을 구현하는 작업을 수행합니다.

### 연습 5: 디바이스 쌍 기능 구현

디바이스 쌍에는 다음 네 가지 유형의 정보가 포함되어 있습니다.

* **태그**: 디바이스에 표시되지 않는 디바이스의 정보입니다.
* **원하는 속성**: 백 엔드 앱에서 지정한 원하는 설정입니다.
* **보고된 속성**: 보고된 디바이스의 설정 값입니다.
* **디바이스 ID 속성**: 디바이스를 식별하는 읽기 전용 정보입니다.

IoT Hub를 통해 관리되는 디바이스 쌍은 쿼리용으로 설계되었으며 실제 IoT 디바이스와 동기화됩니다. 언제든지 백 엔드 앱에서 디바이스 쌍을 쿼리할 수 있습니다. 이 쿼리는 디바이스에 대한 현재 상태 정보를 반환할 수 있습니다. 디바이스와 쌍이 동기화되기 때문에 이 데이터를 가져오는 것은 디바이스에 대한 호출을 수반하지 않습니다. 디바이스 쌍의 대부분의 기능은 Azure IoT Hub에서 제공되므로 이를 사용하기 위해 많은 코드를 작성할 필요가 없습니다.

디바이스 쌍과 직접 메서드의 기능 사이에는 약간의 중복이 있습니다. 직접 메서드를 사용하여 디바이스 속성을 설정할 수 있는데 이는 직관적인 방식으로 보일 수 있습니다. 그러나 직접 메서드를 사용하려면 백 엔드 앱이 해당 설정을 명시적으로 기록해야 합니다(설정에 액세스해야 하는 경우). 디바이스 쌍을 사용하면 이 정보가 기본적으로 저장 및 유지 관리됩니다.

이 연습에서는 디바이스 앱과 백 엔드 서비스 앱 모두에 일부 코드를 추가하여 디바이스 쌍 동기화가 작업 중인 것을 보여줍니다.

#### 작업 1: 디바이스 쌍을 사용하여 디바이스 속성을 동기화하는 코드 추가

1. **CheeseCaveOperator** 백 엔드 앱을 실행하는 Visual Studio Code 인스턴스로 돌아갑니다.

1. 앱이 계속 실행 중인 경우 터미널에 입력 포커스를 배치하고 **CTRL+C**를 눌러 앱을 종료합니다.

1. **Program.cs**가 열려 있는지 확인합니다.

1. `INSERT registry manager variable below here` 주석을 찾습니다.

1. 레지스트리 관리자 변수를 삽입하려면 다음 코드를 입력합니다.

    ```csharp
    private static RegistryManager registryManager;
    ```

1. `INSERT register desired property changed handler code below here` 주석을 찾습니다.

1. 레지스트리 관리자 인스턴스를 만들고 쌍 속성을 설정하는 기능을 추가하려면 다음 코드를 입력합니다.

    ```csharp
    // 레지스트리 관리자는 Digital Twins에 액세스하는 데 사용됩니다.
    registryManager = RegistryManager.CreateFromConnectionString(serviceConnectionString);
    SetTwinProperties().Wait();
    ```

1. Locate the `INSERT Device twins section below here` comment.

1. 디바이스 쌍의 원하는 속성을 업데이트하는 기능을 추가하려면 다음 코드를 입력합니다.

    ```csharp
    // 디바이스 쌍 섹션입니다.

    private static async Task SetTwinProperties()
    {
        var twin = await registryManager.GetTwinAsync(deviceId);
        var patch =
            @"{
                tags: {
                    customerID: 'Customer1',
                    cheeseCave: 'CheeseCave1'
                },
                properties: {
                    desired: {
                        patchId: 'set values',
                        temperature: '50',
                        humidity: '85'
                    }
                }
            }";
        await registryManager.UpdateTwinAsync(twin.DeviceId, patch, twin.ETag);

        var query = registryManager.CreateQuery(
            "SELECT * FROM devices WHERE tags.cheeseCave = 'CheeseCave1'", 100);
        var twinsInCheeseCave1 = await query.GetNextAsTwinAsync();
        Console.WriteLine("Devices in CheeseCave1: {0}",
            string.Join(", ", twinsInCheeseCave1.Select(t => t.DeviceId)));
    }
    ```

    > **참고**:  **SetTwinProperties** 메서드는 디바이스 쌍에 추가될 태그 및 속성을 정의한 다음 쌍이 업데이트되는 JSON 조각을 만듭니다. 메서드의 다음 부분에서는 **cheeseCave** 태그가 "CheeseCave1"로 설정된 디바이스를 나열하기 위해 쿼리를 수행할 수 있는 방법을 보여줍니다. 이 쿼리는 연결에 **레지스트리 읽기** 권한이 있어야 합니다.

1. **파일** 메뉴에서 Program.cs 파일을 저장하려면 **저장**을 클릭합니다.

#### 작업 2: 디바이스에 대한 디바이스 쌍 설정을 동기화하는 코드 추가

1. **cheesecavedevice** 앱이 표시되어 있는 Visual Studio Code 인스턴스로 돌아갑니다.

1. 앱이 계속 실행 중인 경우 터미널에 입력 포커스를 배치하고 **CTRL+C**를 눌러 앱을 종료합니다.

1. 코드 편집기 창에서 **Program.cs** 파일이 열려 있는지 확인합니다.

1. `INSERT register desired property changed handler code below here` 주석을 찾습니다.

1. 원하는 속성 변경 처리기를 등록하려면 다음 코드를 추가합니다.

    ```csharp
    // 디바이스 쌍이 원하는 초기 속성을 보고하도록 가져옵니다.
    Twin deviceTwin = deviceClient.GetTwinAsync().GetAwaiter().GetResult();
    ConsoleHelper.WriteGreenMessage("Initial twin desired properties: " + deviceTwin.Properties.Desired.ToJson());

    // 디바이스 쌍 업데이트 콜백을 설정합니다.
    deviceClient.SetDesiredPropertyUpdateCallbackAsync(OnDesiredPropertyChanged, null).Wait();
    ```

1. `INSERT OnDesiredPropertyChanged method below here` 주석을 찾습니다.

1. 디바이스 쌍 속성 변경에 응답하는 코드를 추가하려면 다음 코드를 입력합니다.

    ```csharp
    private static async Task OnDesiredPropertyChanged(TwinCollection desiredProperties, object userContext)
    {
        try
        {
            // Cheese Cave Simulator 속성을 업데이트합니다.
            cheeseCave.DesiredHumidity = desiredProperties["humidity"];
            cheeseCave.DesiredTemperature = desiredProperties["temperature"];
            ConsoleHelper.WriteGreenMessage("Setting desired humidity to " + desiredProperties["humidity"]);
            ConsoleHelper.WriteGreenMessage("Setting desired temperature to " + desiredProperties["temperature"]);

            // 속성을 IoT Hub에 다시 보고합니다.
            var reportedProperties = new TwinCollection();
            reportedProperties["fanstate"] = cheeseCave.FanState.ToString();
            reportedProperties["humidity"] = cheeseCave.DesiredHumidity;
            reportedProperties["temperature"] = cheeseCave.DesiredTemperature;
            await deviceClient.UpdateReportedPropertiesAsync(reportedProperties);

            ConsoleHelper.WriteGreenMessage("\nTwin state reported: " + reportedProperties.ToJson());
        }
        catch
        {
            ConsoleHelper.WriteRedMessage("Failed to update device twin");
        }
    }
    ```

    이 코드는 디바이스 쌍에서 원하는 속성이 변경될 때 호출되는 처리기를 정의합니다. 그러면 새 값이 IoT Hub에 다시 보고되어 변경 사항을 확인합니다.

1. **파일** 메뉴에서 Program.cs 파일을 저장하려면 **저장**을 클릭합니다.

    > **참고**:  이제 앱에 디바이스 쌍에 대한 지원을 추가했으며 **desiredHumidity**와 같은 명시적 변수를 포함하는 것을 다시 고려할 수 있습니다. 대신 디바이스 쌍 개체의 변수를 사용할 수 있습니다.

#### 작업 3: 디바이스 쌍 테스트

디바이스 쌍의 원하는 속성 변경을 관리하는 코드를 테스트하려면 앱을 올바른 순서로 시작합니다. 즉, 디바이스 애플리케이션부터 시작한 후에 백 엔드 애플리케이션을 시작합니다.

1. **cheesecavedevice** 디바이스 앱이 표시되어 있는 Visual Studio Code 인스턴스로 전환합니다.

1. **cheesecavedevice** 디바이스 앱을 실행하려면 터미널 창을 열고 `dotnet run` 명령을 입력합니다.

    터미널에 쓰기가 시작되고 원격 분석 메시지가 표시됩니다.

1. **CheeseCaveOperator** 백 엔드 앱이 표시되어 있는 Visual Studio Code 인스턴스로 전환합니다.

1. **CheeseCaveOperator** 백 엔드 앱을 시작하려면 터미널 창을 열고 `dotnet run` 명령을 입력합니다.

1. **cheesecavedevice** 디바이스 앱이 표시되어 있는 Visual Studio Code 인스턴스로 다시 전환합니다.

1. 콘솔 출력을 확인하여 디바이스 쌍이 올바르게 동기화되어 있는지 확인합니다.

    ![콘솔 출력](media/LAB_AK_15-cheesecave-device-twin-received.png)

    팬이 정상 작동하면 빨간색 경고가 해제됩니다(팬이 고장나지 않는다면).

    ![콘솔 출력](media/LAB_AK_15-cheesecave-device-twin-success.png)

1. Visual Studio Code의 두 인스턴스 모두 앱을 중지한 다음 Visual Studio Code 창을 닫습니다.

이 랩에서 구현한 코드는 프로덕션 환경용 품질은 아닙니다. 하지만 직접 메서드와 디바이스 쌍 속성을 조합해 IoT 디바이스를 모니터링하고 제어하는 방법을 기본적으로 확인할 수는 있습니다. 이 구현에서는 백 엔드 서비스 앱을 먼저 실행해야 운영자 제어 메시지가 전송됩니다. 일반적으로 백 엔드 서비스 앱은 운영자가 직접 메서드를 보내거나 필요할 때마다 디바이스 쌍 속성을 설정하기 위해 브라우저 인터페이스가 필요합니다.
