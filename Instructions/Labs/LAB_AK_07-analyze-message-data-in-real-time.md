---
lab:
    title: '랩 07: 디바이스 메시지 라우팅'
    module: '모듈 4: 메시지 처리 및 분석'
---

# 디바이스 메시지 라우팅

## 랩 시나리오

Contoso 경영진은 DPS를 사용하여 자동 디바이스 등록을 구현하는 데 깊은 인상을 받았습니다. 이제 제품 패키징 및 배송과 관련된 비즈니스별 문제에 대한 IoT 솔루션을 탐색할 것을 요청하고 있습니다.

Contoso 치즈 제조 사업의 핵심 구성 요소는 고객에게 치즈의 패키징 및 배송입니다. 비용 효율성을 극대화하기 위해 Contoso는 온-프레미스 포장 시설을 운영하고 있습니다. 워크플로는 간단합니다 - 패키지는 배송을 위해 조립된 다음 패키지를 가져와 우체통에 떨어뜨리는 컨베이어 벨트 시스템에 배치됩니다. 성공의 척도는 주어진 기간 동안 컨베이어 벨트 시스템을 떠나는 패키지 수입니다(일반적으로 작업 시프트).

컨베이어 벨트 시스템은 이 과정에서 중요한 연결고리이며, 패키지가 올바르게 배송되도록 시각적으로 모니터링됩니다. 이 시스템에는 중지, 느림 및 빠름의 세 가지 운영자 제어 속도가 있습니다. 당연히 느린 속도로 전송되는 패키지 의 수는 빠른 속도보다 적습니다. 그러나 컨베이어 벨트 시스템의 진동 수준도 속도가 느립니다. 또한 진동 수준이 높으면 시스템의 마모가 가속화되고 컨베이어에서 패키지가 떨어질 수 있습니다. 진동이 과도해지면 컨베이어 벨트를 정지하여 검사를 통해 더 심각한 고장을 방지해야 합니다.

솔루션의 주요 목표는 심각한 시스템 손상이 발생하기 전에 문제가 있음을 감지하는 데 사용할 수 있는 진동 수준에 기반한 예방 유지 보수 형태를 구현하는 것입니다. 

> **참고**: **예방적 유지 보수**(예방적 유지 보수 또는 예측적 유지 보수라고도 함)는 장비가 정상적으로 작동하는 동안 수행되도록 유지 보수 활동을 예약하는 장비 유지 보수 프로그램입니다. 이 방법의 목적은 비싼 대가를 지불하는 중단 사태를 초래하는 예기치 않은 고장을 방지하는 것입니다.

비정상적인 진동 수준을 감지하는 것이 항상 쉬운 것은 아닙니다. 이러한 이유로 진동 수준 및 데이터 이상을 측정하는 데 도움이 되는 Azure IoT 솔루션을 찾고 있습니다. 진동 센서는 다양한 위치에서 컨베이어 벨트에 부착되며, IoT 디바이스를 사용하여 원격 분석을 IoT hub로 전송합니다. IoT Hub는 Azure Stream Analytics 및 기본 제공 ML(Machine Learning) 모델을 사용하여 진동 이상 징후를 실시간으로 경고합니다. 또한 나중에 추가로 분석할 수 있도록 모든 원격 분석 데이터를 보관할 계획입니다.

단일 IoT 디바이스에서 시뮬레이션된 원격 분석을 사용하여 솔루션을 프로토타입화하기로 결정했습니다.

진동 데이터를 사실적으로 시뮬레이션하려면 엔지니어 중 한 명과 협력하여 진동의 원인이 무엇인지 조금 알아봅니다. 전체 진동 수준에 기여하는 진동의 종류는 여러 가지가 있는 것으로 나타났습니다. 예를 들어, "강제 진동"은 파손된 가이드 휠이나 컨베이어 벨트에 부적절하게 놓여진 특히 무거운 하중에 의해 발생할 수 있습니다. 또한 시스템 설계 제한(예: 속도 또는 무게)을 초과할 때 발생할 수 있는 "진동 증가"도 있습니다. 약간의 도움만 있으면 수용 가능한 진동 데이터 표시를 생성하고 이상 징후를 발생시키는 시뮬레이션된 IoT 디바이스의 코드를 개발할 수 있습니다.

다음의 리소스가 만들어집니다.

![랩 7 아키텍처](media/LAB_AK_07-architecture.png)

## 이 랩에서

이 랩에서는 다음 활동을 완료할 예정입니다.

* 랩 필수 구성 요소가 충족되는지 확인(필요한 Azure 리소스가 있음)
* Azure CLI를 사용하여 Azure IoT Hub 및 장치 ID 만들기
* Visual Studio Code를 사용하여 IoT Hub로 디바이스 원격 분석을 전송하는 C# 앱 만들기
* Azure Portal을 사용하여 Blob Storage를 통해 메시지 라우팅 만들기
* Azure Portal을 사용하여 Azure Stream Analytics 작업까지의 두 번째 메시지 라우팅 만들기

## 랩 지침

### 연습 1: 랩 필수 구성 요소 확인

이 랩에서는 다음과 같은 Azure 리소스를 사용할 수 있다고 가정합니다.

| 리소스 종류 | 리소스 이름 |
| :-- | :-- |
| 리소스 그룹 | AZ-220-RG |
| IoT Hub | AZ-220-HUB-_{YOUR-ID}_ |
| 디바이스 ID | VibrationSensorId |

이러한 리소스를 사용할 수 없는 경우 연습 2로 이동하기 전에 아래 설명에 따라 **lab07-setup.azcli** 스크립트를 실행해야 합니다. 스크립트 파일은 개발자 환경 구성(랩 3)의 일부로 로컬로 복제한 GitHub 리포지토리에 포함됩니다.

**lab07-setup.azcli** 스크립트는 **bash** 셸 환경에서 실행되도록 작성됩니다. Azure Cloud Shell에서 실행하는 것이 가장 쉽습니다.

1. 브라우저를 사용하여 [Azure Shell](https://shell.azure.com/)을 열고 이 과정에 사용 중인 Azure 구독으로 로그인합니다.

    Cloud Shell에 대한 저장소 설정 관련 메시지가 표시되면 기본값을 수락합니다.

1. Azure Cloud Shell에서 **Bash**를 사용하고 있는지 확인합니다.

    Azure Cloud Shell 페이지의 왼쪽 상단에 있는 드롭다운으로 환경을 선택할 수 있습니다. 선택한 드롭다운 값이 **Bash**인지 확인합니다.

1. Azure Shell 도구 모음에서 **파일 업로드/다운로드**(오른쪽에서 네 번째 단추)를 클릭합니다.

1. 드롭다운에서 **업로드**를 클릭합니다.

1. 파일 선택 대화 상자에서 개발 환경을 구성할 때 다운로드한 GitHub 랩 파일의 폴더 위치로 이동합니다.

    _랩 3: 개발 환경 설정_, ZIP 파일을 다운로드하고 콘텐츠를 로컬로 추출하여 랩 리소스를 포함하는 GitHub 리포지토리를 복제했습니다. 추출된 폴더 구조는 다음 폴더 경로를 포함합니다.

    * Allfiles
      * 랩
          * 07-디바이스 메시지 라우팅
            * 설정

    lab07-setup.azcli 스크립트 파일은 랩 7의 설치 폴더에 있습니다.

1. **lab07-setup.azcli** 파일을 선택한 다음 **열기**를 클릭합니다.

    파일 업로드가 완료되면 알림이 나타납니다.

1. Azure Cloud Shell에 올바른 파일이 업로드되었는지 확인하려면 다음 명령을 입력합니다.

    ```bash
    ls
    ```

    `ls` 명령으로 현재 디렉터리의 내용을 나열합니다. lab07-setup.azcli 파일이 나열되어 있어야 합니다.

1. 설치 스크립트가 포함된 이 랩에 대한 디렉터리를 만든 다음 해당 디렉터리로 이동하려면 다음 Bash 명령을 입력합니다.

    ```bash
    mkdir lab7
    mv lab07-setup.azcli lab7
    cd lab7
    ```

1. **lab07-setup.azcli**에 실행 권한이 있는지 확인하려면 다음 명령을 입력합니다.

    ```bash
    chmod +x lab07-setup.azcli
    ```

1. Cloud Shell 도구 모음에서 lab07-setup.azcli 파일을 편집하려면 **편집기 열기**(오른쪽에서 두 번째 단추 - **{ }**)를 클릭합니다.

1. lab7 폴더를 확장하고 스크립트 파일을 열려면 **파일** 목록에서 **lab7**을 클릭한 다음 **lab07-setup.azcli**를 클릭합니다.

    이제 편집기에서 **lab07-setup.azcli** 파일의 내용을 표시합니다.

1. 편집기에서 `{YOUR-ID}` 및 `{YOUR-LOCATION}`의 할당된 값을 업데이트합니다.

    아래 샘플을 예로 들어, `{YOUR-ID}`를 이 과정을 시작할 때 만든 고유 ID(예: **CAH191211**)로 설정하고 `{YOUR-LOCATION}`를 리소스에 적합한 위치로 설정해야 합니다.

    ```bash
    #!/bin/bash

    YourID="{YOUR-ID}"
    RGName="AZ-220-RG"
    IoTHubName="AZ-220-HUB-$YourID"

    Location="{YOUR-LOCATION}"
    ```

    > **참고**:  `{YOUR-LOCATION}` 변수는 해당 지역의 짧은 이름으로 설정되어야 합니다. 이 명령을 입력하면 사용 가능한 지역 목록과 이 지역의 짧은 이름(**이름** 열)을 볼 수 있습니다.
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

1. 파일의 변경 내용을 저장하고 편집기를 닫으려면 편집기 창의 오른쪽 상단에서 ...를 클릭한 다음 **편집기 닫기**를 클릭합니다.

    저장하라는 메시지가 표시된 경우 **저장**을 클릭하면 편집기가 닫힙니다.

    > **참고**:  **CTRL+S**를 사용하여 언제든지 저장할 수 있으며 **CTRL+Q**를 사용하여 편집기를 닫을 수 있습니다.

1. 이 랩에 필요한 리소스를 만들려면 다음 명령을 입력합니다.

    ```bash
    ./lab07-setup.azcli
    ```

    이 스크립트를 실행하는 데 몇 분이 걸릴 수 있습니다. 각 단계가 완료되면 JSON 출력이 표시됩니다.

    스크립트는 먼저 **AZ-220-RG**라는 리소스 그룹과 **AZ-220-HUB-{YourID}** 라는 IoT Hub를 만듭니다. 이미 있는 경우 해당 메시지가 표시됩니다. 그런 다음 스크립트는 **VibrationSensorId**의 ID가 있는 디바이스를 IoT hub에 추가하고 디바이스 연결 문자열을 표시합니다.

1. 스크립트가 완료되면 디바이스의 연결 문자열이 표시됩니다.

    연결 문자열은 "HostName="으로 시작합니다.

1. 연결 문자열을 텍스트 문서에 복사하고 **VibrationSensorId**디바이스용임을 기억합니다.

    연결 문자열을 쉽게 찾을 수 있는 위치에 저장하셨다면, 랩을 계속할 준비가 되었습니다.

### 연습 2: 진동 원격 분석용 코드 작성

컨베이어 벨트를 모니터링하는 핵심은 진동 원격 분석의 출력입니다. 진동은 일반적으로 가속도(m/s&#x00B2;)로 측정되지만, 때로는 중력(1g = 9.81m/s&#x00B2;)으로도 측정됩니다. 진동에는 3가지 유형이 있습니다.

* 자연 진동은 구조물이 진동하는 빈도입니다.
* 자유 진동은 구조물에 충격이 가해졌을 때 발생하지만 간섭 없이 진동하도록 내버려 둡니다.
* 강제 진동: 구조물이 약간의 압력을 받았을 때 발생합니다.

강제 진동은 컨베이어 벨트에 위험한 진동입니다. 이 진동은 낮은 수준에서 시작하더라도 구조물이 조기에 내려 앉도록 할 수 있습니다. 컨베이어 벨트 작동 시 자유 진동이 발생하는 경우는 적습니다. 아시다시피, 대부분의 기계에는 자연 진동이 있습니다.

작성할 코드 샘플은 다양한 속도(정지, 느림, 빠름)로 작동되는 컨베이어 벨트를 시뮬레이션합니다. 벨트가 빨리 작동할수록 더 많은 패키지가 배송되지만 진동의 영향을 더 많이 받게 됩니다. 무작위로 사인파를 토대로 자연 진동을 추가합니다. 변칙 검색 시스템은 이 사인파의 급상승이나 급하락을 변칙으로 잘못 식별할 수 있습니다. 그런 다음, 두 가지 형태의 강제 진동을 추가합니다. 첫 번째는 진동이 주기적으로 증가하는 효과가 있습니다(아래 이미지 참조). 두 번째로 증가 진동에서는 사인 파가 추가되고 처음에는 작지만 점점 증가합니다.

이 프로토타입 단계에서는 컨베이어 벨트에 하나의 센서 디바이스(시뮬레이션된 IoT 디바이스)만 있다고 가정합니다. 센서는 진동 데이터를 통신하는 것 외에도 다른 데이터(전달된 패키지, 주변 온도 및 유사한 메트릭)도 전송합니다. 이 랩의 경우 추가 값이 스토리지 보관으로 전송됩니다.

이 연습 동안 이 랩의 거의 모든 코딩이 완성됩니다. Visual Studio Code를 사용하여 C#으로 시뮬레이터 코드를 작성합니다. 나중에 랩에서 소량의 SQL 코딩을 완료합니다.

이 연습에서는 다음을 수행합니다.

* 컨베이어 벨트 시뮬레이터 구축
* 이전 단원에서 만든 IoT Hub로 원격 분석 메시지 보내기

#### 작업 1: 원격 분석을 보낼 앱 만들기

1. Visual Studio Code를 연 다음 C# 확장이 설치되어 있는지 확인합니다.

    이 과정의 랩 3에서 개발 환경을 설정했지만 디바이스 앱 빌드를 시작하기 전에 빠르게 다시 확인하는 것이 좋습니다. 

    Visual Studio Code에서 C#을 사용하려면 [.NET Core](https://dotnet.microsoft.com/download)와 [C# 확장](https://marketplace.visualstudio.com/items?itemName=ms-vscode.csharp) 모두 설치해야 합니다. 상단에서 다섯 번째 단추를 클릭해서 왼쪽 도구 모음을 사용하여 Visual Studio Code 확장 창을 열 수 있습니다.

1. **터미널** 메뉴에서 **새 터미널**을 클릭합니다.

    명령 프롬프트의 일부로 표시된 디렉터리 경로를 확인합니다. 이전 랩 프로젝트의 폴더 구조 내에서 이 프로젝트를 빌드하지 않으려고 합니다.
  
1. 터미널 명령 프롬프트에서 "vibrationdevice"라는 디렉터리를 만들고 현재 디렉터리를 해당 디렉터리로 변경하려면 다음 명령을 입력합니다.

   ```bash
   mkdir vibrationdevice
   cd vibrationdevice
   ```

1. 새 .NET 콘솔 애플리케이션을 만들려면 다음 명령을 입력합니다.

    ```bash
    dotnet new console
    ```

    이 명령은 프로젝트 파일과 함께 폴더에 **Program.cs** 파일을 만듭니다.

1. 디바이스 앱에 필요한 코드 라이브러리를 설치하려면 다음 명령을 입력합니다.

    ```bash
    dotnet add package Microsoft.Azure.Devices.Client
    dotnet add package Newtonsoft.Json
    ```

    다음 작업에서 시뮬레이션된 디바이스 앱을 빌드하고 테스트합니다.

#### 작업 2: 원격 분석 전송에 코드 추가

이 작업에서 빌드하는 시뮬레이션된 디바이스 앱은 컨베이어 벨트를 모니터링하는 IoT 디바이스를 시뮬레이션합니다. 앱은 2초마다 센서 판독 값을 시뮬레이션하고 진동 센서 데이터를 보고합니다.

1. Visual Studio Code **파일** 메뉴에서 **폴더 열기**를 클릭합니다.

    터미널 명령 프롬프트에 나열된 폴더 경로를 사용하여 프로젝트 폴더를 찾습니다.
  
1. 폴더 열기 대화 상자에서 터미널 명령 프롬프트에 표시되는 디렉터리 경로로 이동하고 **vibrationdevice**를 클릭한 다음 **폴더 선택**을 클릭합니다.

    필요한 자산을 로드하라는 메시지가 나타나면 **예**를 클릭합니다.

    이제 Visual Studio Code 탐색기 창을 열어야 합니다. 그렇지 않은 경우 왼쪽 도구 모음을 사용하여 탐색기 창을 엽니다. 도구 모음 단추 위에 마우스 포인터를 올려놓아 단추 이름을 표시할 수 있습니다.

1. 탐색기 창에서 **Program.cs**를 클릭합니다.

1. 코드 편집기 보기에서 Program.cs 파일의 기본 내용을 삭제합니다.

    기본 내용은 이전 작업에서 `dotnet new console` 명령을 실행했을 때 만들어졌습니다.

1. 시뮬레이션된 디바이스에 대한 코드를 만들려면 다음 코드를 빈 Program.cs 파일에 붙여넣습니다.

    ```csharp
    // Copyright (c) Microsoft. All rights reserved.
    // MIT 라이선스에 따라 라이선스가 부여되었습니다. 전체 라이선스 정보는 프로젝트 루트의 LICENSE 파일을 참조하세요.

    using System;
    using Microsoft.Azure.Devices.Client;
    using Newtonsoft.Json;
    using System.Text;
    using System.Threading.Tasks;

    namespace vibration_device
    {
        class SimulatedDevice
        {
            // 전역 원격 분석.
            private const int intervalInMilliseconds = 2000;                                // 대기 함수에 필요한 시간 간격입니다.
            private static readonly int intervalInSeconds = intervalInMilliseconds / 1000;  // 시간 간격(초)입니다.

            // 전역 컨베이어 벨트.
            enum SpeedEnum
            {
                stopped,
                slow,
                fast
            }
            private static int packageCount = 0;                                        // 컨베이어 벨트를 떠나는 패키지 수입니다.
            private static SpeedEnum beltSpeed = SpeedEnum.stopped;                     // 컨베이어 벨트의 초기 상태.
            private static readonly double slowPackagePerSecond = 1;                   // 느린 속도/초당 완성된 패키지
            private static readonly double fastPackagePerSecond = 2;                   // 빠른 속도/초당 완성된 패키지
            private static double beltStoppedSeconds = 0;                               // 벨트가 멈춘 시간입니다.
            private static double temperature = 60;                                     // 시설의 주변 온도.
            private static double seconds = 0;                                          // 컨베이어 벨트가 실행되는 시간.

            // 진동 전역.
            private static double forcedSeconds= 0;                                    // 강제 진동이 시작된 이후의 시간.
            private static double increasingSeconds = 0;                                // 진동이 증가하기 시작한 이후의 시간.
            private static double naturalConstant;                                      // 자연 진동의 심각도를 식별하는 상수.
            private static double forcedConstant = 0;                                   // 강제 적용의 심각도를 식별하는 상수.
            private static double increasingConstant = 0;                               // 진동 증가의 심각성을 식별하는 상수.

            // IoT Hub 전역 변수.
            private static DeviceClient s_deviceClient;

            // IoT Hub를 사용하여 디바이스를 인증하는 디바이스 연결 문자열입니다.
            private readonly static string s_deviceConnectionString = "<your device connection string>";

            private static void colorMessage(string text, ConsoleColor clr)
            {
                Console.ForegroundColor = clr;
                Console.WriteLine(text);
                Console.ResetColor();
            }
            private static void greenMessage(string text)
            {
                colorMessage(text, ConsoleColor.Green);
            }

            private static void redMessage(string text)
            {
                colorMessage(text, ConsoleColor.Red);
            }

            // 시뮬레이션된 원격 분석을 보내는 비동기 메서드입니다.
            private static async void SendDeviceToCloudMessagesAsync(Random rand)
            {
                // 컨베이어 벨트의 진동 원격 분석을 시뮬레이션합니다.
                double vibration;

                while (true)
                {
                    // 벨트 속도를 무작위로 조정합니다.
                    switch (beltSpeed)
                    {
                        case SpeedEnum.fast:
                            if (rand.NextDouble() < 0.01)
                            {
                                beltSpeed = SpeedEnum.stopped;
                            }
                            if (rand.NextDouble() > 0.95)
                            {
                                beltSpeed = SpeedEnum.slow;
                            }
                            break;

                        case SpeedEnum.slow:
                            if (rand.NextDouble() < 0.01)
                            {
                                beltSpeed = SpeedEnum.stopped;
                            }
                            if (rand.NextDouble() > 0.95)
                            {
                                beltSpeed = SpeedEnum.fast;
                            }
                            break;

                        case SpeedEnum.stopped:
                            if (rand.NextDouble() > 0.75)
                            {
                                beltSpeed = SpeedEnum.slow;
                            }
                            break;
                    }

                    // 진동 수준을 설정합니다.
                    if (beltSpeed == SpeedEnum.stopped)
                    {
                        // 벨트가 멈춘 경우 모든 진동이 멈춥니다.
                        forcedConstant = 0;
                        increasingConstant = 0;
                        vibration = 0;

                        // 경고를 보내야 하는 경우를 대비하여 벨트가 중지된 시간을 기록합니다.
                        beltStoppedSeconds += intervalInSeconds;
                    }
                    else
                    {
                        // 컨베이어 벨트가 작동되고 있습니다.
                        beltStoppedSeconds = 0;

                        // 원치 않는 진동에서 임의의 시작을 확인합니다.

                        // 강제 진동을 확인합니다.
                        if (forcedConstant == 0)
                        {
                            if (rand.NextDouble() < 0.1)
                            {
                                // 강제 진동이 시작됩니다.
                                forcedConstant = 1 + 6 * rand.NextDouble();             // 1에서 7 사이의 숫자입니다.
                                if (beltSpeed == SpeedEnum.slow)
                                    forcedConstant /= 2;                                // 느린 속도의 경우 진동이 적습니다.
                                forcedSeconds = 0;
                                redMessage($"Forced vibration starting with severity: {Math.Round(forcedConstant, 2)}");
                            }
                        }
                        else
                        {
                            if (rand.NextDouble() > 0.99)
                            {
                                forcedConstant = 0;
                                greenMessage("Forced vibration stopped");
                            }
                            else
                            {
                                redMessage($"Forced vibration: {Math.Round(forcedConstant, 1)} 시작 위치: {DateTime.Now.ToShortTimeString()}");
                            }
                        }

                        // 증가 진동을 확인합니다.
                        if (increasingConstant == 0)
                        {
                            if (rand.NextDouble() < 0.05)
                            {
                                // 진동이 증가합니다.
                                increasingConstant = 100 + 100 * rand.NextDouble();     // 100~200 사이의 숫자.
                                if (beltSpeed == SpeedEnum.slow)
                                    increasingConstant *= 2;                            // 속도가 느린 경우 기간이 더 길어집니다.
                                increasingSeconds = 0;
                                redMessage($"Increasing vibration starting with severity: {Math.Round(increasingConstant, 2)}");
                            }
                        }
                        else
                        {
                            if (rand.NextDouble() > 0.99)
                            {
                                increasingConstant = 0;
                                greenMessage("Increasing vibration stopped");
                            }
                            else
                            {
                                redMessage($"Increasing vibration: {Math.Round(increasingConstant, 1)} 시작 위치: {DateTime.Now.ToShortTimeString()}");
                            }
                        }

                        // 자연 진동부터 시작하여 진동을 적용합니다.
                        vibration = naturalConstant * Math.Sin(seconds);

                        if (forcedConstant > 0)
                        {
                            // 강제 진동을 추가합니다.
                            vibration += forcedConstant * Math.Sin(0.75 * forcedSeconds) * Math.Sin(10 * forcedSeconds);
                            forcedSeconds += intervalInSeconds;
                        }

                        if (increasingConstant > 0)
                        {
                            // 증가 진동을 추가합니다.
                            vibration += (increasingSeconds / increasingConstant) * Math.Sin(increasingSeconds);
                            increasingSeconds += intervalInSeconds;
                        }
                    }

                    // 컨베이어 벨트 앱이 시작된 이후 시간을 늘립니다.
                    seconds += intervalInSeconds;

                    // 여정을 완료한 패키지를 계산합니다.
                    switch (beltSpeed)
                    {
                        case SpeedEnum.fast:
                            packageCount += (int)(fastPackagesPerSecond * intervalInSeconds);
                            break;

                        case SpeedEnum.slow:
                            packageCount += (int)(slowPackagesPerSecond * intervalInSeconds);
                            break;

                        case SpeedEnum.stopped:
                            // 패키지가 없습니다!
                            break;
                    }

                    // 주변 온도가 임의로 변합니다.
                    temperature += rand.NextDouble() - 0.5d;

                    // 두 개의 메시지를 만듭니다.
                    // 1. 진동 원격 분석만 Azure Stream Analytics로 라우팅됩니다.
                    // 2. Azure 스토리지 계정으로 라우팅되는 로깅 정보입니다.

                    // 원격 분석 JSON 메시지를 만듭니다.
                    var telemetryDataPoint = new
                    {
                        vibration = Math.Round(vibration, 2),
                    };
                    var telemetryMessageString = JsonConvert.SerializeObject(telemetryDataPoint);
                    var telemetryMessage = new Message(Encoding.ASCII.GetBytes(telemetryMessageString));

                    // 메시지에 사용자 지정 애플리케이션 속성을 추가합니다. 메시지를 라우팅하는 데 사용됩니다.
                    telemetryMessage.Properties.Add("sensorID", "VSTel");

                    // 벨트가 5초 이상 멈춘 경우 경고를 보냅니다.
                    telemetryMessage.Properties.Add("beltAlert", (beltStoppedSeconds > 5) ? "true", "false");

                    Console.WriteLine($"Telemetry data: {telemetryMessageString}");

                    // 원격 분석 메시지를 보냅니다.
                    await s_deviceClient.SendEventAsync(telemetryMessage);
                    greenMessage($"Telemetry sent {DateTime.Now.ToShortTimeString()}");

                    // 로깅 JSON 메시지를 만듭니다.
                    var loggingDataPoint = new
                    {
                        vibration = Math.Round(vibration, 2),
                        packages = packageCount,
                        speed = beltSpeed.ToString(),
                        temp = Math.Round(temperature, 2),
                    };
                    var loggingMessageString = JsonConvert.SerializeObject(loggingDataPoint);
                    var loggingMessage = new Message(Encoding.ASCII.GetBytes(loggingMessageString));

                    // 메시지에 사용자 지정 애플리케이션 속성을 추가합니다. 메시지를 라우팅하는 데 사용됩니다.
                    loggingMessage.Properties.Add("sensorID", "VSLog");

                    // 벨트가 5초 이상 멈춘 경우 경고를 보냅니다.
                    loggingMessage.Properties.Add("beltAlert", (beltStoppedSeconds > 5) ? "true", "false");

                    Console.WriteLine($"Log data: {loggingMessageString}");

                    // 로깅 메시지를 보냅니다.
                    await s_deviceClient.SendEventAsync(loggingMessage);
                    greenMessage("Log data sent\n");

                    await Task.Delay(intervalInMilliseconds);
                }
            }

            private static void Main(string[] args)
            {
                Random rand = new Random();
                colorMessage("Vibration sensor device app.\n", ConsoleColor.Yellow);

                // MQTT 프로토콜을 사용하여 IoT Hub에 연결합니다.
                s_deviceClient = DeviceClient.CreateFromConnectionString(s_deviceConnectionString, TransportType.Mqtt);

                // 일반 진동 수준에 대해 상수로 2에서 4 사이의 숫자를 만듭니다.
                naturalConstant = 2 + 2 * rand.NextDouble();

                SendDeviceToCloudMessagesAsync(rand);
                Console.ReadLine();
            }
        }
    }
    ```

1. 코드를 검토하는 데 몇 분 정도 걸립니다.

    > **중요:** 몇 분 정도 시간을 내어 코드의 주석을 읽습니다. IoT 메시지를 학습하는 데 가장 중요한 코드 섹션은 "두 메시지 만들기:" 라는 설명으로 시작합니다. 컨베이어 벨트 진동 수준을 정의하는 데 사용되는 수학(이 랩 시작 부분의 시나리오 설명에서 설명)이 코드에 어떻게 작용했는지에도 관심이 있을 수 있습니다.

1. `<your device connection string>`(줄 44)을 이전 연습 중에 저장한 디바이스 연결 문자열로 바꿉니다.

    > **참고**: 이 코드에 필요한 변경 사항은 이것뿐입니다.

1. **Program.cs** 파일을 저장합니다.

    > **참고**:  해당 코드는 랩 7의 `/Starter` 폴더에서도 사용할 수 있습니다. 시작 폴더에서 해당 코드를 사용하도록 선택한 경우 `<your device connection string>`을 바꿔야 합니다.

#### 작업 3: 원격 분석 전송 코드 테스트

1. 터미널에서 앱을 실행하려면 다음 명령을 입력합니다.

    ```bash
    dotnet run
    ```

   이 명령은 현재 폴더에서 **Program.cs** 파일을 실행합니다.

1. 콘솔 출력은 다음과 유사하게 표시되어야 합니다.

    ```
    진동 센서 디바이스 앱.

    원격 분석 데이터: {"vibration":0.0}
    오전 10:29에 원격 분석 전송
    로그 데이터: {"vibration":0.0,"packages":0,"speed":"stopped","temp":60.22}
    로그 데이터 전송함

    원격 분석 데이터: {"vibration":0.0}
    오전 10:29에 원격 분석 전송
    로그 데이터: {"vibration":0.0,"packages":0,"speed":"stopped","temp":59.78}
    로그 데이터 전송함
    ```

    > **참고**:  터미널 창에서 녹색 텍스트는 제대로 작동 중임을 표시하는 데 사용되며 빨간색 텍스트는 문제가 발생했음을 표시하는 데 사용됩니다. 오류 메시지를 수신하면 디바이스 연결 문자열을 확인하여 시작합니다.

1. 잠시 동안 원격 분석을 보고 예상된 범위에서 진동을 주는지 확인합니다.

1. 다음 작업을 위해 이 앱이 실행되도록 둡니다.

    다음 작업을 계속하지 않으면 터미널 창에 **Ctrl-C**를 입력하여 앱을 중지할 수 있습니다. 나중에 `dotnet run` 명령을 사용하여 다시 시작할 수 있습니다.

#### 작업 4: IoT Hub에서 원격 분석을 수신하고 있는지 확인

이 작업에서는 Azure Portal을 사용하여 IoT Hub가 원격 분석을 받고 있는지 확인합니다.

1. [Azure Portal](https://portal.azure.com)을 엽니다.

1. 대시보드의 **AZ-220-RG** 리소스 그룹 타일에서 **AZ-220-HUB-_{YourID}_** 를 클릭합니다.

1. **개요** 창에서 아래로 스크롤하여 메트릭 타일을 봅니다.

1. **마지막 데이터 표시**에서 시간 범위를 1시간으로 변경합니다. 

    **클라우드 메시지에 대한 디바이스** 타일은 현재 작업을 일부 표시해야 합니다. 작업이 표시되지 않으면 약간의 대기 시간이 있기 때문에 잠시 기다립니다.

    디바이스가 원격 분석을 만들고 허브에서 수신하면 다음 단계는 메시지를 올바른 엔드포인트로 라우팅하는 것입니다.

### 연습 3: Azure Blob Storage에 대한 메시지 라우팅 만들기

진동 모니터링 시스템의 아키텍처는 데이터를 보관하기 위한 스토리지 위치와 보다 즉각적인 분석을 위한 위치라는 두 개의 목적지로 데이터를 전송해야 합니다. Azure IoT는 *메시지 라우팅*을 통해 정확한 서비스로 데이터를 전달하는 훌륭한 방법을 제공합니다.

이 시나리오에서는 다음 두 가지 경로를 만들어야 합니다.

* 첫 번째 라우팅은 데이터 보관을 위한 스토리지입니다.
* 두 번째 라우팅은 변칙 검색을 위한 이벤트 허브로 이동합니다.

메시지 라우팅은 한 번에 하나씩 만들고 테스트하는 것이 가장 좋기 때문에 이 연습에서는 스토리지 라우팅에 중점을 두겠습니다. 이 라우팅을 "로깅" 라우팅이라고 하며, Azure 리소스를 만들기 위한 몇 가지 단계를 자세히 살펴보는 작업이 포함됩니다. 이 라우팅을 만드는 데 필요한 모든 기능은 Azure Portal에 있습니다.

스토리지 라우팅을 단순하게 유지하고 Azure Blob Storage를 사용합니다(Data Lake Storage도 사용할 수 있음). 메시지 라우팅의 핵심 기능은 수신 데이터의 필터링입니다. SQL로 작성된 필터는 특정 조건이 충족될 때만 라우팅 아래로 출력됩니다.

데이터를 필터링하는 가장 쉬운 방법 중 하나는 메시지 속성에 있으므로 코드에 이 두 줄을 추가했습니다.

```csharp
...
telemetryMessage.Properties.Add("sensorID", "VSTel");
...
loggingMessage.Properties.Add("sensorID", "VSLog");
```

메시지 라우팅에 포함된 SQL 쿼리는 `sensorID` 값을 테스트할 수 있습니다.

이 연습에서는 로깅 라우팅을 만들고 테스트합니다.

#### 작업 1: 로깅 메시지를 Azure Storage로 라우팅

1. [Azure Portal](https://portal.azure.com/)에서 IoT Hub의 **개요** 창이 열려 있는지 확인합니다.

1. 왼쪽 메뉴의 **메시지**에서 **메시지 라우팅**을 클릭합니다.

1. **메시지 라우팅** 창에서 **라우팅** 탭이 선택되어 있는지 확인합니다.

1. 첫 번째 라우팅을 추가하려면 **추가**를 클릭합니다.

    이제 **라우팅 추가** 블레이드가 표시되어야 합니다.

1. **라우팅 추가** 블레이드에서 **이름** 아래에 `vibrationLoggingRoute`를 입력합니다.

1. **엔드포인트** 오른쪽에서 **엔드포인트 추가**를 클릭한 다음 드롭다운 목록에서 **스토리지**를 클릭합니다.

    **스토리지 엔드포인트 추가** 블레이드가 표시됩니다.

1. **엔드포인트 이름** 아래에 `vibrationLogEndpoint`를 입력합니다.

1. 스토리지를 만들고 컨테이너를 선택하려면 **컨테이너 선택**을 클릭합니다.

    Azure 구독에 이미 있는 스토리지 계정 목록이 나열됩니다. 이 시점에서 기존 스토리지 계정 및 컨테이너를 선택할 수 있지만 이 랩에서는 새 스토리지 계정과 컨테이너를 만듭니다.

1. 새 스토리지 계정을 만들려면 **스토리지 계정**을 클릭합니다.

    **스토리지 만들기** 창이 표시됩니다.

1. **이름** 아래에 있는 **스토리지 만들기** 창에 **vibrationstore**를 입력한 다음 귀하의 이니셜과 오늘 날짜를 다음과 같이 추가합니다. **vibrationstorecah191211**

    > **참고**:  이 필드에는 3~24자 사이의 고유한 소문자와 숫자만 포함될 수 있습니다.

1. **계정 종류** 목록에서 **StorageV2(범용 V2)** 를 선택합니다.

1. **성능**에서 **표준**이 선택되었는지 확인합니다.

    이렇게 하면 전체 성능을 낮추면서 비용을 절감할 수 있습니다.

1. **복제**에서 **LRS(로컬 중복 저장소)** 가 선택되어 있는지 확인합니다.

    이렇게 하면 재해 복구에 대한 위험 완화 비용을 절감할 수 있습니다. 프로덕션에서 솔루션은 보다 강력한 복제 전략이 필요할 수 있습니다.

1. **위치**에서 이 과정의 랩에 사용 중인 지역을 선택합니다.

1. 스토리지 계정을 만들려면 **확인**을 클릭합니다.

1. 요청의 유효성을 검사하고 스토리지 계정 배포가 완료될 때까지 기다립니다.

    유효성 검사 및 생성에는 1~2분 정도 걸릴 수 있습니다.

    완료되면 **스토리지 계정 만들기** 창이 닫히고 **스토리지 계정** 블레이드가 표시됩니다. 방금 만든 스토리지 계정을 표시하려면 스토리지 계정 블레이드를 업데이트해야 합니다.

1. **vibrationstore**를 검색한 다음 방금 만든 스토리지 계정을 선택합니다.

   **컨테이너** 블레이드가 표시되어야 합니다. 새 스토리지 계정이기 때문에 나열할 컨테이너가 없습니다.

1. 컨테이너를 만들려면 **컨테이너**를 클릭합니다.

    **새 컨테이너** 팝업이 표시됩니다.

1. **새 컨테이너** 팝업에서 **이름** 아래에 **vibrationcontainer**를 입력합니다.

   다시 말하지만 소문자와 숫자만 허용됩니다.

1. **공용 액세스 수준**에서 **프라이빗(익명 액세스 없음)** 이 선택되었는지 확인합니다.

1. 컨테이너를 만들려면 **확인**을 클릭합니다.

    잠시 후 컨테이너에 대한 **임대 상태**가 **사용 가능**으로 표시됩니다.

1. 솔루션용 컨테이너를 선택하려면 **vibrationcontainer**를 클릭한 다음 **선택**을 클릭합니다.

    **스토리지 엔드포인트 추가** 창으로 돌아가집니다. 방금 만든 스토리지 계정 및 컨테이너에 대한 URL로 **Azure Storage 컨테이너**가 설정되었습니다.

1. **일괄 처리 빈도** 및 **청크 크기 창**을 기본값인 **100**으로 둡니다.

1. **인코딩**에서, 두 가지 옵션이 있으며 **AVRO**가 선택되어 있음을 알 수 있습니다.

    > **참고**:  기본적으로 IoT Hub는 메시지 본문 속성과 메시지 속성이 모두 있는 Avro 형식으로 콘텐츠를 씁니다. Avro 형식은 다른 엔드포인트에 사용되지 않습니다. Avro 형식은 데이터 및 메시지 보존에 적합하지만 데이터를 쿼리하는 데 사용하는 것은 어렵습니다. 이에 비해 JSON 또는 CSV 형식은 데이터를 쿼리하는 데 사용하는 것이 훨씬 쉽습니다. 이제 IoT Hub는 AVRO뿐만 아니라 JSON의 Blob Storage에 데이터를 기록하는 것을 지원합니다.

1. **파일 이름 형식** 필드에 지정된 값을 검사합니다.

    **파일 이름 형식** 필드는 스토리지의 파일에 데이터를 쓰는 데 사용되는 패턴을 지정합니다. 다양한 토큰은 파일이 생성될 때 값으로 대체됩니다.

1. 엔드포인트를 만들려면 창 아래쪽에서 **만들기**를 클릭합니다.

    유효성 검사 및 만들기에는 몇 분 정도 걸릴 수 있습니다. 완료되면 **라우팅 추가** 블레이드로 돌아가집니다.

1. **라우팅 추가** 블레이드의 **데이터 원본**에서 **디바이스 원격 분석 메시지**가 선택되어 있는지 확인합니다.

1. **라우팅 활성화**에서 **사용**이 선택되어 있는지 확인합니다.

1. **라우팅 쿼리**에서 **참**을 아래의 쿼리로 바꿉니다.

    ```sql
    sensorID = "VSLog"
    ```

    이렇게 하면 `sensorID = "VSLog"`인 경우에만 메시지가 이 라우팅을 따릅니다.

1. 이 라우팅을 저장하려면 **저장**을 클릭합니다.

    성공 메시지를 기다립니다. 완료되면 라우팅이 **메시지 라우팅** 블레이드에 나열되어야 합니다.

1. Azure Portal 대시보드로 돌아갑니다.

다음 단계에서는 로깅 라우팅이 작동하는지 확인합니다.

### 연습 4: 로깅 라우팅 Azure Stream Analytics 작업

로깅 라우팅이 예상대로 작동하는지 확인하기 위해 Azure Portal의 Storage Explorer를 사용하여 유효성을 검사할 수 있는 Blob Storage로 로깅 메시지를 라우팅하는 Stream Analytics 작업을 만듭니다.

이렇게 하면 라우팅에 다음 설정이 포함되어 있는지 확인할 수 있습니다.

* **이름** - vibrationLoggingRoute
* **데이터 원본** - DeviceMessages
* **라우팅 쿼리** - sensorID = "VSLog"
* **엔드포인트** - vibrationLogEndpoint
* **사용** - 참

> **참고**: 이 랩에서 데이터를 스토리지로 라우팅한 다음 Azure Stream Analytics를 통해 데이터를 스토리지로 보내는 것이 이상하게 보일 수 있습니다. 프로덕션 시나리오에서는 두 경로 모두 장기적으로 사용할 수 없습니다. 대신 여기에서 만드는 두 번째 경로가 존재하지 않을 가능성이 높습니다. 지금, 랩 환경에서 라우팅이 예상대로 작동하는지 확인하고 Azure Stream Analytics의 간단한 구현을 표시하는 방법을 사용하고 있습니다.

#### 작업 1: Stream Analytics 작업 만들기

1. Azure Portal 메뉴에서 **리소스 만들기**를 클릭합니다.

1. **새** 블레이드의 **Marketplace 검색** 텍스트 상자에 **Stream Analytics 작업**을 입력한 다음 **Stream Analytics 작업**을 클릭합니다.

1. **Stream Analytics 작업** 블레이드에서 **만들기**를 클릭합니다.

    **새 Stream Analytics 작업** 창이 표시됩니다.

1. **새 Stream Analytics 작업** 창에서 **이름** 아래에 `vibrationJob`을 입력합니다.

1. **구독**에서 랩에 사용 중인 구독을 선택합니다.

1. **리소스 그룹**에서 **AZ-220-RG**를 선택합니다.

1. **위치**에서 모든 랩 작업에 사용 중인 지역을 선택합니다.

1. **호스팅 환경**에서 **클라우드**를 선택합니다.

    Edge 호스팅은 과정의 후반부에서 설명합니다.

1. **스트리밍 단위**에서 수를 **3**에서 **1**로 줄입니다.

    이 랩에서는 3 단위가 필요하지 않으며 이로 인해 비용을 줄일 수 있습니다,

1. Stream Analytics 작업을 만들려면 **만들기**를 클릭합니다.

1. **배포 성공** 메시지를 기다린 다음 새 리소스를 엽니다.

    > **팁:** 새 리소스로 이동하는 메시지를 놓치거나 언제든지 리소스를 찾아야 하는 경우 **홈/모든 리소스**를 선택합니다. 리소스 목록에 표시할 리소스 이름을 충분히 입력합니다.

1. 잠시 후 새 Stream Analytics 작업을 검토합니다.

    입력이나 출력이 없는 비어 있는 작업과 스켈레톤 쿼리가 있습니다. 다음 단계에서는 이러한 항목을 채웁니다.

1. 입력을 만들려면 왼쪽 탐색의 **작업 토폴로지** 아래에서 **입력**을 클릭합니다.

    **입력** 창이 표시됩니다.

1. **입력** 창에서 **스트림 입력 추가**를 클릭하고 드롭다운 목록에서 **IoT Hub**를 선택합니다.

    **새 입력** 창이 표시됩니다.

1. **새 입력** 창에서 **별칭 입력** 아래에 `vibrationInput`을 입력합니다.

1. **구독에서 IoT Hub 선택**을 선택해야 합니다.

1. **구독**에서, 이전에 IoT Hub를 만드는 데 사용한 구독이 선택되어 있는지 확인합니다.

1. **IoT Hub**에서, 과정 랩의 시작 부분에서 만든 IoT Hub를 선택합니다(**AZ-220-HUB-_{YourID}_**).

1. **엔드포인트**에서 **메시지**가 선택되어 있는지 확인합니다.

1. **공유 액세스 정책 이름**에서 **iothubowner**가 선택되어 있는지 확인합니다.

    > **참고**:  **공유 액세스 정책 키**가 채워졌으며 읽기 전용입니다.

1. **소비자 그룹**에서 **$Default**가 선택되어 있는지 확인합니다.

1. **이벤트 Serialization 형식**에서 **JSON**이 선택되어 있는지 확인합니다.

1. **인코딩**에서 **UTF-8**이 선택되어 있는지 확인합니다.

    일부 필드를 보려면 아래로 스크롤해야 할 수도 있습니다.

1. **이벤트 압축 유형**에서 **없음**이 선택되어 있는지 확인합니다.

1. 새 입력을 저장하려면 **저장**을 클릭한 다음 입력이 만들어질 때까지 기다립니다.

    새 입력을 표시하도록 **입력** 목록을 업데이트해야 합니다.

1. 출력을 만들려면 왼쪽 탐색의 **작업 토폴로지** 아래에서 **출력**을 클릭합니다.

    **출력** 창이 표시됩니다.

1. **출력** 창에서 **추가**를 클릭한 다음 드롭다운 목록에서 **Blob Storage/Data Lake Storage Gen2**를 선택합니다.

    **새 출력** 창이 표시됩니다.

1. **새 출력** 창에서 **출력 별칭** 아래에 `vibrationOutput`을 입력합니다.

1. **구독에서 스토리지 선택**이 선택되어 있는지 확인합니다.

1. **구독**에서 이 랩에 사용 중인 구독을 선택합니다.

1. **스토리지 계정**에서 이전에 만든 스토리지 계정(**vibrationstore**및 이니셜과 날짜 포함)을 선택합니다.

    > **참고**:  **스토리지 계정 키**가 자동으로 채워졌으며 읽기 전용입니다.

1. **컨테이너**에서 **기존 사용**이 선택되어 있고 드롭다운 목록에서 **vibrationcontainer**가 선택되어 있는지 확인합니다.

1. **경로 패턴**을 비어 있는 상태로 둡니다.

1. **날짜 형식** 및 **시간 형식**을 기본값으로 둡니다.

1. **이벤트 Serialization 형식**에서 **JSON**이 선택되어 있는지 확인합니다.

1. **인코딩**에서 **UTF-8**이 선택되어 있는지 확인합니다.

1. **형식**에서 **줄로 구분됨**이 선택되어 있는지 확인합니다.

    > **참고**:  이 설정에서 각 레코드를 각 줄에 있는 JSON 개체로 저장하고 전체적으로 수행하면 유효하지 않은 JSON 레코드인 파일이 됩니다. 다른 옵션인 **배열**은 전체 문서에 각 레코드가 배열의 항목인 JSON 배열로 서식이 지정되도록 합니다. 이렇게 하면 전체 파일을 유효한 JSON으로 구문 분석할 수 있습니다.

1. **최소 행**을 비어 있는 상태로 둡니다.

1. **최대 시간**에서 **시간**과 **분**을 비어 있는 상태로 둡니다.

1. **인증 모드**에서 **연결 문자열**이 선택되어 있는지 확인합니다.

1. 출력을 만들려면 **저장**을 클릭한 다음 출력이 만들어질 때까지 기다립니다.

    **출력** 목록이 새 출력으로 업데이트됩니다.

1. 쿼리를 편집하려면 왼쪽 탐색의 **작업 토폴로지** 아래에서 **쿼리**를 클릭합니다.

1. 쿼리 편집 창에서 기존 쿼리를 아래의 쿼리로 바꿉니다.

    ```sql
    SELECT
        *
    INTO
        vibrationOutput
    FROM
        vibrationInput
    ```

1. 편집 창 바로 위의 **쿼리 저장**을 클릭합니다.

1. 왼쪽 탐색에서 **개요**를 클릭합니다.

#### 작업 2: 로깅 경로 테스트

이제 재미있는 부분입니다. 디바이스 앱이 생성하는 원격 분석이 라우팅을 따라 스토리지 컨테이너로 작동하나요?

1. Visual Studio Code에서 만든 디바이스 앱이 계속 실행 중인지 확인합니다. 

    그렇지 않은 경우 `dotnet run`을 사용하여 Visual Studio Code 터미널에서 실행합니다.

1. Stream Analytics 작업의 **개요** 창에서 **시작**을 클릭합니다.

1. **작업 시작** 창에서 **작업 출력 시작 시간**을 **지금**으로 설정한 다음 **시작**을 클릭합니다.

    작업이 시작하는 데 몇 분 정도 걸릴 수 있습니다.

1. Azure Portal 메뉴에서 **대시보드**를 클릭합니다.

1. 리소스 그룹 타일에서 **vibrationstore**(이니셜 및 날짜 포함) 스토리지 계정을 선택합니다.

    스토리지 계정이 표시되지 않으면 리소스 그룹 타일 상단의 **새로 고침** 단추를 사용합니다.

1. 스토리지 계정의 **개요** 창에서 **모니터링** 섹션이 보일 때까지 아래로 스크롤합니다.

1. **모니터링**에서 **다음 기간의 데이터 표시:**에 인접한 시간 범위를**1시간**으로 변경합니다.

    차트에서 활동을 확인해야 합니다.

1. 왼쪽 탐색 메뉴에서 **Storage Explorer(미리 보기)** 를 클릭합니다.

    Storage Explorer를 사용하여 모든 데이터가 스토리지 계정으로 이동 중임을 다시 한 번 확신할 수 있습니다. 

    > **참고**:  Storage Explorer는 현재 미리 보기 모드이므로 정확한 작업 모드가 변경될 수 있습니다.

1. **Storage Explorer(미리 보기)** 의 **BLOB 컨테이너** 아래에서 **vibrationcontainer**를 클릭합니다.

    데이터를 보려면 폴더 계층 구조를 탐색해야 합니다. 첫 번째 폴더는 IoT Hub의 이름으로 지정되고, 다음 폴더는 파티션, 연도, 월, 일 및 시간 순으로 지정됩니다. 

1. 오른쪽 창의 **이름** 아래에서 IoT Hub의 폴더를 두 번 클릭한 다음, 가장 최근 시간 폴더를 열 때까지 두 번 클릭하면서 계층 구조 아래로 이동합니다.

    시간 폴더 내에 생성된 시간(분)의 이름을 한 파일이 표시됩니다.

1. Azure Streaming Analytics 작업을 중지하려면 포털 대시보드로 돌아가서 **vibrationJob**을 선택합니다.

1. **Stream Analytics 작업** 페이지에서 **중지**를 클릭한 다음 **예**를 클릭합니다.

    디바이스 앱, 허브, 아래로 라우팅 및 스토리지 컨테이너에 대한 활동을 추적했습니다. 대단한 발전입니다! 데이터 시각화를 간략하게 살펴볼 때 다음 모듈에서 이 시나리오 Stream Analytics을 계속 진행하겠습니다.

1. Visual Studio Code 창으로 전환합니다.

1. 터미널 명령 프롬프트에서 디바이스 시뮬레이터 앱을 종료하려면 **CTRL-C**를 누릅니다.

> **중요**: 이 과정의 데이터 시각화 모듈을 완료할 때까지 이러한 리소스를 제거하지 마세요.
