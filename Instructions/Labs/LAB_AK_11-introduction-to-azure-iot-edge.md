---
lab:
    title: '랩 11: Azure IoT Edge 소개'
    module: '모듈 6: Azure IoT Edge 배포 프로세스'
---

# Azure IoT Edge 소개

## 랩 시나리오

Contoso는 글로벌 시장에서 현지 소비자를 장려하기 위해 현지 장인들과 협력하여 전 세계의 새로운 지역에서 치즈를 생산하고 있습니다.

각 지역은 지역 치즈를 생산하는 데 사용되는 혼합 및 가공 기계가 설치된 여러 생산 라인을 지원합니다. 현재 시설에는 각 컴퓨터에 연결된 IoT 디바이스가 있습니다. 이러한 디바이스는 센서 데이터를 Azure로 스트리밍하며, 모든 데이터는 클라우드에서 처리됩니다.

많은 양의 데이터가 수집되고 일부 컴퓨터에서 신속히 응답해야 하기 때문에 Contoso는 IoT Edge 게이트웨이 디바이스를 사용하여 일부 인텔리전스를 Edge로 가져와서 즉시 처리하려고 합니다. 데이터의 일부는 여전히 클라우드로 전송됩니다. 또한, 데이터 인텔리전스를 IoT Edge로 가져오면 로컬 네트워크가 불량하더라도 데이터를 처리하고 신속하게 대응할 수 있습니다.

Azure IoT Edge 솔루션의 프로토타입을 생성하는 업무를 맡았습니다. 먼저 온도를 모니터링하는 IoT Edge 디바이스를 설정합니다(치즈 가공 기계 중 하나에 연결된 디바이스를 시뮬레이션). 그런 다음, 평균 온도를 계산하고 프로세스 제어 값을 초과하는 경우 경고 알림을 생성하는 디바이스에 Stream Analytics 모듈을 배포합니다.

다음 리소스가 만들어집니다.

![랩 11 아키텍처](media/LAB_AK_11-architecture.png)

## 랩 내용

이 랩에서는 다음 활동을 완료할 예정입니다.

* 랩 필수 구성 요소가 충족되는지 확인(필요한 Azure 리소스가 있음)
* Azure IoT Edge 사용 Linux VM 배포
* Azure CLI를 사용하여 IoT Hub에서 IoT Edge 디바이스 ID 만들기
* IoT Edge 디바이스를 IoT Hub에 연결
* 에지 디바이스에 에지 모듈 추가
* Azure Stream Analytics를 IoT Edge 모듈로 배포

## 랩 지침

### 연습 1: 랩 필수 구성 요소 확인

이 랩은 다음 Azure 리소스를 사용할 수 있다고 가정합니다.

| 리소스 유형 | 리소스 이름 |
| :-- | :-- |
| 리소스 그룹 | rg-az220 |
| IoT Hub | iot-az220-training-{your-id} |

이러한 리소스를 사용할 수 없는 경우 연습 2로 이동하기 전에 아래 설명에 따라 **lab11-setup.azcli** 스크립트를 실행해야 합니다. 스크립트 파일은 개발자 환경 구성(랩 3)의 일부로 로컬로 복제한 GitHub 리포지토리에 포함됩니다.

> **참고**:  **lab11-setup.azcli** 스크립트는 **bash** 셸 환경에서 실행되도록 작성됩니다. 이는 Azure Cloud Shell에서 실행할 수 있는 가장 쉬운 방법입니다.

1. 브라우저를 사용하여 [Azure Cloud Shell](https://shell.azure.com/)을 열고 이 과정에 사용 중인 Azure 구독으로 로그인합니다.

1. Cloud Shell에 대한 스토리지 설정 관련 메시지가 표시되면 기본값을 수락합니다.

1. Cloud Shell에서 **Bash**를 사용하고 있는지 확인합니다.

    Azure Cloud Shell 페이지의 왼쪽 상단에 있는 드롭다운은 환경을 선택하는 데 사용됩니다. 선택한 드롭다운 값이 **Bash**인지 확인합니다.

1. Cloud Shell 도구 모음에서 **파일 업로드/다운로드**(오른쪽의 네 번째 단추)를 클릭합니다.

1. 드롭다운에서 **업로드**를 클릭합니다.

1. 파일 선택 대화 상자에서 개발 환경을 구성할 때 다운로드한 GitHub 랩 파일의 폴더 위치로 이동합니다.

    이 과정의 랩 3 "개발 환경 설정"에서 ZIP 파일을 다운로드하고 내용을 로컬로 추출하여 랩 리소스를 포함하는 GitHub 리포지토리를 복제했습니다. 추출된 폴더 구조에는 다음 폴더 경로가 포함됩니다.

    * 모든 파일
      * 랩
          * 11-Azure IoT Edge 소개
            * Setup

    lab11-setup.azcli 스크립트 파일은 랩 11의 Setup 폴더에 있습니다.

1. **lab11-setup.azcli** 파일을 선택한 다음 **열기**를 클릭합니다.

    파일 업로드가 완료되면 알림이 표시됩니다.

1. 올바른 파일을 업로드했는지 확인하려면 다음 명령을 입력합니다.

    ```bash
    ls
    ```

    `ls` 명령으로 현재 디렉터리의 내용을 나열합니다. lab11-setup.azcli 파일이 나열됩니다.

1. 설치 스크립트가 포함된 이 랩에 대한 디렉터리를 만든 다음 해당 디렉터리로 이동하려면 다음 Bash 명령을 입력합니다.

    ```bash
    mkdir lab11
    mv lab11-setup.azcli lab11
    cd lab11
    ```

    이러한 명령은 이 랩의디렉터리를 만들고 **lab11-setup.azcli** 파일을 해당 디렉터리로 이동한 다음 디렉터리를 변경하여 새 디렉터리를 현재 작업 디렉터리로 만듭니다.

1. **lab11-setup.azcli**에 실행 권한이 있는지 확인하려면 다음 명령을 입력합니다.

    ```bash
    chmod +x lab11-setup.azcli
    ```

1. Cloud Shell 도구 모음에서 lab11-setup.azcli 파일에 액세스할 수 있도록 설정하려면 **편집기 열기**(오른쪽에서 두 번째 단추 - **{}**)를 클릭합니다.

1. **파일** 목록에서 lab11 폴더를 확장하고 스크립트 파일을 열려면 **lab11**를 클릭한 다음 **lab11-setup.azcli**를 클릭합니다.

    이제 편집기에서 **lab11-setup.azcli** 파일의 내용을 표시합니다.

1. 편집기에서 `{your-id}` 및 `{your-location}` 변수의 값을 업데이트합니다.

    아래 샘플을 예로 들면 `{your-id}`는 이 과정을 시작할 때 만든 고유 ID(**cah191211**)로 설정해야 하며 `{your-location}`는 리소스 그룹과 일치하는 위치로 설정해야 합니다.

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
    ./lab11-setup.azcli
    ```

    이 작업을 실행하려면 몇 분 정도 걸립니다. 각 단계가 완료될 때 출력이 표시됩니다.

스크립트가 완료되면 랩으로 계속할 준비가 끝납니다.

### 연습 2: Linux VM 배포

이 연습에서는 Ubuntu Server VM을 배포합니다.

1. 필요한 경우 Azure 계정 자격 증명을 사용하여 Azure Portal에 로그인합니다.

    Azure 계정이 두 개 이상인 경우 이 과정에 사용할 구독에 연결된 계정으로 로그인해야 합니다.

1. **리소스, 서비스 및 문서 검색** 필드에 **가상 머신**을 입력합니다.

1. 검색 결과의 **서비스**에서 **가상 머신**을 클릭합니다.

1. **가상 머신** 페이지에서 **+ 추가**를 클릭하고 **가상 머신**을 클릭합니다.

1. **가상 머신 만들기** 블레이드의 **구독** 드롭다운에서 이 과정에 사용 중인 Azure 구독을 선택합니다.

1. **리소스 그룹**의 오른쪽에 있는 **새로 만들기**를 클릭합니다.

    > **참고**: 이 과정에서 만드는 모든 가상 머신 리소스를 쉽게 추적/관리하기 위해 여기서는 리소스 그룹 하나를 사용합니다. 각 VM용으로 별도의 리소스 그룹을 만들라는 지침이 나올 수도 있습니다. 프로덕션 환경에서는 각 VM용으로 개별 리소스 그룹을 만들면 VM에 추가하는 리소스를 쉽게 관리할 수 있습니다. 하지만 이 과정에서는 VM을 단순한 방식으로 사용하므로 각 VM용으로 개별 리소스 그룹을 만들 필요도 없고 만들 수도 없습니다.

1. 새 리소스 그룹 팝업에서 **이름** 아래에 **rg-az220vm**를 입력하고 **확인**을 클릭합니다.

1. **가상 머신 이름** 텍스트 상자에 **vm-az220-training-edge0001-{your-id}**를 입력합니다.

1. **지역** 드롭다운에서 Azure IoT Hub가 프로비전된 지역을 선택합니다.

1. **가용성 옵션**은 **인프라 중복성 필요 없음**으로 남겨 둡니다.

1. **이미지** 필드에서 **Ubuntu Server 18.04 LTS - Gen1** 이미지를 선택합니다.

1. **Azure Spot 인스턴스** 필드는 선택하지 않은 상태로 유지합니다.

1. **크기**의 오른쪽에서 **크기 변경**을 클릭합니다.

1. **VM 크기 선택** 블레이드의 **VM 크기**에서 **Standard_B1ms**를 클릭한 다음 **선택**을 클릭합니다.

    Standard_B1ms가 표시되지 않으면 **모든 필터 지우기** 링크를 사용하여 이 크기가 목록에서 사용 가능하게 해야 할 수도 있습니다.

    > **참고**:  일부 VM 크기는 일부 지역에서 사용할 수 없습니다. 이후 단계에서 VM 크기를 선택할 수 없는 경우 다른 지역을 시도해 보세요. 예를 들어 **미국 서부**에 사용 가능한 크기가 없는 경우 **미국 서부 2**를 사용해 보세요.

1. **관리자 계정**에서 **인증 유형**의 오른쪽에 있는 **암호**를 클릭합니다.

1. VM 관리자 계정은 **사용자 이름**, **암호** 및 **암호 확인** 필드에 값을 입력하세요.

    > **중요:** 이러한 값 없이는 VM에 연결할 수 없으니 잊어버리지 마세요.

1. **인바운드 포트 규칙**은 VM에 대한 인바운드 **SSH** 액세스를 사용하도록 구성됩니다.

    이 규칙은 VM을 원격으로 구성 및 관리하는 데 사용됩니다.

1. **검토 + 만들기**를 클릭합니다.

1. **유효성 검사 통과** 메시지가 블레이드 상단에 표시될 때까지 기다린 다음 **만들기**를 클릭합니다.

    > **참고**:  배포가 완료되기까지 최대 5분이 걸릴 수 있습니다. 배포 중에 다음 연습을 할 수 있습니다.

### 연습 3: Azure CLI를 사용하여 IoT Hub에서 IoT Edge 디바이스 ID 만들기

이 연습에서는 Azure CLI를 사용하여 Azure IoT Hub에 새 IoT Edge 디바이스 ID를 만듭니다.

1. 필요한 경우 Azure 계정 자격 증명을 사용하여 Azure Portal에 로그인합니다.

    Azure 계정이 두 개 이상인 경우 이 과정에 사용할 구독에 연결된 계정으로 로그인해야 합니다.

1. Azure Portal 도구 모음에서 Azure Cloud Shell을 열려면 **Cloud Shell**을 클릭합니다.

    왼쪽 탐색 메뉴가 아니라 Azure Portal 도구 모음의 Cloud Shell 단추에 명령 프롬프트와 유사한 모양의 아이콘이 있습니다.

1. **Bash** 환경 옵션을 사용 중인지 확인합니다.

    Cloud Shell의 좌측 상단 모서리에 있는 환경 드롭다운에서 Bash를 선택해야 합니다.

1. 명령 프롬프트의 IoT Hub에서 IoT Edge 디바이스 ID를 만들려면 다음 명령을 입력합니다.

    ```bash
    az iot hub device-identity create --hub-name iot-az220-training-{your-id} --device-id sensor-th-0067 --edge-enabled
    ```

    `{your-id}` 자리 표시자는 이 과정을 시작할 때 만든 사용자 ID 값으로 바꿔야 합니다.

    > **참고**: Azure Portal에서 IoT Hub를 사용하여 이 IoT Edge 디바이스를 만들 수도 있습니다. **IoT Hub** -> **IoT Edge** -> **IoT Edge 디바이스 추가**.

1. 명령으로 만든 출력을 리뷰합니다.

    출력에는 IoT Edge 디바이스에 대해 만든 **디바이스 ID**의 정보가 포함되어 있습니다. 예를 들어 자동 생성된 키를 포함하는 `symmetricKey` 인증이 기본값이며, `iotEdge` 기능은 지정된 `--edge-enabled` 매개 변수가 표시하는 것처럼 `true`로 설정된 것을 확인할 수 있습니다.

    ```json
    {
        "authentication": {
            "symmetricKey": {
                "primaryKey": "jftBfeefPsXgrd87UcotVKJ88kBl5Zjk1oWmMwwxlME=",
                "secondaryKey": "vbegAag/mTJReQjNvuEM9HEe1zpGPnGI2j6DJ7nECxo="
            },
            "type": "sas",
            "x509Thumbprint": {
                "primaryThumbprint": null,
                "secondaryThumbprint": null
            }
        },
        "capabilities": {
            "iotEdge": true
        },
        "cloudToDeviceMessageCount": 0,
        "connectionState": "Disconnected",
        "connectionStateUpdatedTime": "0001-01-01T00:00:00",
        "deviceId": "sensor-th-0067",
        "deviceScope": "ms-azure-iot-edge://sensor-th-0067-637093398936580016",
        "etag": "OTg0MjI1NzE1",
        "generationId": "637093398936580016",
        "lastActivityTime": "0001-01-01T00:00:00",
        "status": "enabled",
        "statusReason": null,
        "statusUpdatedTime": "0001-01-01T00:00:00"
    }
    ```

1. IoT Edge 디바이스의 **연결 문자열**을 표시하려면 다음 명령어를 입력하세요.

    ```bash
    az iot hub device-identity connection-string show --device-id sensor-th-0067 --hub-name iot-az220-training-{your-id}
    ```

    `{your-id}` 자리 표시자는 이 과정을 시작할 때 만든 사용자 ID 값으로 바꿔야 합니다.

1. 명령의 JSON 출력에서 `connectionString`의 값을 복사한 다음 나중에 참조할 수 있도록 저장해두세요.

    이 연결 문자열은 IoT Edge 디바이스를 IoT Hub에 연결하도록 구성하는 데 사용합니다.

    ```json
        {
          "connectionString": "HostName={IoTHubName}.azure-devices.net;DeviceId=sensor-th-0067;SharedAccessKey=jftBfeefPsXgrd87UcotVKJ88kBl5Zjk1oWmMwwxlME="
        }
    ```

    > **참고**:  IoT Edge 디바이스 연결 문자열은 **IoT Hub** -> **IoT Edge** -> **Edge 디바이스** -> **연결 문자열(기본 키)**로 이동하여 Azure Portal에서 액세스할 수도 있습니다.

### 연습 4: Iot Edge를 설치하고 IoT Hub에 IoT Edge 디바이스 연결

이 연습에서는 IoT Edge 디바이스를 Azure IoT Hub에 연결합니다.

#### 작업 1: VM에 연결

1. IoT Edge 가상 머신이 성공적으로 배포되었는지 확인합니다.

    Azure Portal에서 알림 창을 확인할 수 있습니다.

1. Azure Portal 메뉴에서 **리소스 그룹**을 클릭합니다.

1. **리소스 그룹** 블레이드에서 rg-az220vm 리소스 그룹을 찾습니다.

1. 블레이드 오른쪽의 **rg-az220vm** 옆에 있는 **클릭하여 상황에 맞는 메뉴 열기** (줄임표 아이콘 - **...**) 를 클릭합니다.

1. 상황에 맞는 메뉴에서 **대시보드에 고정**을 클릭한 후 대시보드로 다시 이동합니다.

    만약 리소스에 쉽게 액세스 하는데 도움이 된다면 대시보드를 **편집**하여 타일을 다시 정렬해도 됩니다.

1. **rg-az220vm** 리소스 그룹 타일에서 IoT Edge 가상 머신을 열려면 **vm-az220-training-edge0001-{your-id}** 를 클릭합니다.

1. **개요** 창 상단에서 **연결**을 클릭한 다음 **SSH**를 클릭합니다.

1. **연결** 창에서 **4. 아래의 예제 명령을 실행하여 VM에 연결**의 예제 명령을 복사합니다.

    이 명령은 VM 및 관리자 사용자 이름의 IP 주소가 포함된 가상 머신에 연결하는 데 사용할 수 있는 샘플 SSH 명령입니다. 명령의 서식은 `ssh username@52.170.205.79`와 유사하게 지정되어야 합니다.

    > **참고**: 샘플 명령에 `-i <private key path>`가 포함된 경우 텍스트 편집기를 사용하여 명령의 해당 부분을 삭제한 다음 업데이트된 명령을 클립보드에 복사합니다.

1. Cloud Shell이 아직 열려 있지 않으면 **Cloud Shell**을 클릭합니다.

1. Cloud Shell 명령 프롬프트에서 텍스트 편집기에 업데이트한 `ssh` 명령을 붙여넣은 다음 **Enter 키**를 누릅니다.

1. **계속 연결하시겠습니까?** 라는 메시지가 표시되면 `예`를 입력한 다음 **Enter 키**를 누릅니다.

    해당 메시지는 VM에 대한 연결 보안에 사용된 인증서가 자체 서명된 인증서이므로 보안 확인입니다. 이 메시지에 대한 대답은 이후 연결을 위해 기억되며, 이 메시지는 첫 번째 연결에서만 표시됩니다.

1. 암호를 입력하라는 메시지가 표시되면 VM이 프로비전될 때 만든 관리자 암호를 입력합니다.

1. 연결이 끝나면 다음과 유사한 Linux VM 이름을 표시하도록 터미널 명령 프롬프트가 변경됩니다.

    ```bash
    username@vm-az220-training-edge0001-{your-id}:~$
    ```

    이는 어떤 VM에 연결되었는지 알려줍니다.

    > **중요:** 연결하면 아마도 뛰어난 Edge VM OS 업데이트가 있다는 안내를 받을 것입니다.  랩 목적상 이것을 무시하고 있지만 생산 중에는 항상 Edge 디바이스를 최신 상태로 유지해야 합니다.

#### 작업 2: 패키지 관리자에 Microsoft 설치 패키지 추가

1. Microsoft 설치 패키지에 액세스하도록 VM을 구성하려면 다음 명령을 실행합니다.

    ```bash
    curl https://packages.microsoft.com/config/ubuntu/18.04/multiarch/prod.list > ./microsoft-prod.list
    ```

1. 다운로드한 패키지 목록을 패키지 관리자에 추가하려면 다음 명령을 실행합니다.

    ```bash
    sudo cp ./microsoft-prod.list /etc/apt/sources.list.d/
    ```

1. 패키지를 설치하려면 Microsoft GPG 공개 키를 설치해야 합니다. 다음 명령을 실행합니다.

    ```bash
    curl https://packages.microsoft.com/keys/microsoft.asc | gpg --dearmor > microsoft.gpg
    sudo cp ./microsoft.gpg /etc/apt/trusted.gpg.d/
    ```

    > **중요**: Azure IoT Edge 소프트웨어 패키지에는 각 패키지 **usr/share/doc/{패키지 이름}** 또는 **LICENSE** 디렉터리에 포함되어 있는 사용 조건이 적용됩니다. 패키지를 사용하기 전에 사용 조건을 읽어보세요. 패키지를 설치 및 사용하면 이러한 사용 조건에 동의하게 됩니다. 사용 조건에 동의하지 않는 경우 해당 패키지를 사용하지 마세요.

#### 작업 3: 컨테이너 엔진 설치

Azure IoT Edge는 OCI 호환 컨테이너 런타임을 사용합니다. 프로덕션 시나리오에서는 Moby 엔진을 사용하는 것이 좋습니다. Azure IoT Edge에서 공식적으로 지원되는 컨테이너 엔진은 Moby 엔진뿐입니다. Docker CE/EE 컨테이너 이미지는 Moby 런타임과 호환 가능합니다.

1. 디바이스에서 패키지 목록을 업데이트하려면 다음 명령을 실행합니다.

    ```bash
    sudo apt-get update
    ```

    이 명령을 실행하려면 몇 분 정도 걸립니다.

1. **Moby** 엔진을 설치하려면 다음 명령을 실행합니다.

    ```bash
    sudo apt-get install moby-engine
    ```

    계속 진행할지를 묻는 프롬프트가 표시되면 **Y**를 입력합니다. 설치는 몇 분 정도 걸릴 수 있습니다.

#### 작업 4: IoT Edge 설치

IoT Edge 보안 디먼은 IoT Edge 디바이스에서 보안 표준을 제공하고 유지 관리합니다. 디먼은 부팅할 때마다 시작되며, 나머지 IoT Edge 런타임을 시작하여 디바이스를 부트스트랩합니다.

1. 일반적으로는 새 패키지를 설치하기 전에 패키지 목록을 업데이트하는 것이 좋습니다. 그러나 이 연습에서는 이전 작업에서 패키지를 업데이트했습니다. 디바이스에서 패키지 목록을 업데이트하려면 다음 명령을 실행합니다.

    ```bash
    sudo apt-get update
    ```

1. 사용 가능한 **IoT Edge 런타임** 버전의 목록을 표시하려면 다음 명령을 실행합니다.

    ```bash
    apt list -a iotedge
    ```

    > **팁**: 이전 버전 런타임을 설치해야 하는 경우 이 명령을 실행하면 유용합니다.

1. 최신 버전의 **IoT Edge 런타임**을 설치하려면 다음 명령을 실행합니다.

    ```bash
    sudo apt-get install iotedge
    ```

    계속 진행할지를 묻는 프롬프트가 표시되면 **Y**를 입력합니다. 설치는 몇 분 정도 걸릴 수 있습니다.

    > **팁**: `apt list -a iotedge` 명령 출력에 표시되었던 **1.0.9-1** 등의 이전 버전을 설치하려는 경우에는 다음 명령을 사용합니다.
    > ```bash
    > sudo apt-get install iotedge=1.0.9-1 libiothsm-std=1.0.9-1
    > ```

1. Azure IoT Edge 런타임이 VM에 설치되어 있는지 확인하려면 다음 명령을 실행합니다.

    ```bash
    iotedge version
    ```

    이 명령은 현재 가상 머신에 설치된 Azure IoT Edge 런타임 버전을 출력합니다.

#### 작업 5: 연결 문자열 구성

이 작업에서는 앞에서 만든 연결 문자열을 사용하여 대칭 키 인증을 통해 VM을 연결하는 단계를 설명합니다.

1. Azure IoT Edge를 구성할 수 있도록 하려면 다음 명령을 입력합니다.

    ```bash
    sudo chmod a+w /etc/iotedge/config.yaml
    ```

    Azure IoT Edge를 구성하려면 IoT Edge 디바이스의 인증서 및 키 파일 전체 경로를 포함하도록 **etc/iotedge/config.yaml** 구성 파일을 수정해야 합니다. 파일을 편집하려면 먼저 **config.yaml** 파일이 읽기 전용이 아닌지 확인해야 합니다. 위의 명령은 **config.yaml** 파일을 쓰기 가능으로 설정합니다.

1. vi/vim 편집기 내에서 **config.yaml** 파일을 열려면 다음 명령을 입력합니다.

    ```bash
    sudo vi /etc/iotedge/config.yaml
    ```

    > **참고**: **code**, **nano**, **emacs** 등의 다른 편집기를 사용해도 됩니다.

1. vi/vim 편집기에서 **연결 문자열을 사용하는 수동 프로비전 구성** 섹션이 표시될 때까지 파일 내에서 아래쪽으로 스크롤합니다.

    > **참고**:  다음은 **config.yaml** 파일을 편집할 때 **vi**를 사용하기 위한 몇 가지 팁입니다.
    > * **i** 키를 눌러 편집기를 삽입 모드로 전환하면 변경 작업을 수행할 수 있습니다.
    > * **Esc** 키를 누르면 삽입 모드를 중지하고 일반 모드로 돌아갑니다.
    > * 저장 후 종료하려면 **x**를 입력하고 **Enter** 키를 누릅니다.
    > * 파일을 저장하려면 **w**를 입력하고 **Enter** 키를 누릅니다.
    > * vi를 종료하려면 **quit**을 입력하고 **Enter** 키를 누릅니다.
    >
    > 저장하거나 종료하기 전에 반드시 삽입 모드를 중지해야 합니다.

1. 파일 프로비전 구성을 찾은 다음 '연결 문자열을 사용하는 수동 프로비전 구성'의 주석 처리가 제거되어 있지 않으면 맨 앞의 **'# '**(우물정자와 공백) 문자를 제거하여 주석 처리를 제거합니다. 그런 다음 `<ADD DEVICE CONNECTION STRING HERE>`를 앞에서 복사한 IoT Edge 디바이스용 연결 문자열로 바꿉니다.

    ```yaml
    # 연결 문자열을 사용하는 수동 프로비전 구성
    provisioning:
      source: "manual"
      device_connection_string: "<ADD DEVICE CONNECTION STRING HERE>"
      dynamic_reprovisioning: false
    ```

    > **중요**: YAML은 공백을 중요한 문자로 처리합니다. 즉, 위와 같이 입력한 코드 줄에서 **provisioning:** 앞에 선행 공백이 없어야 하며 **source:**, **device_connection_string:** 및 **dynamic_reprovisioning:** 앞에는 선행 공백 2개가 있어야 합니다.

1. 변경 내용을 저장하고 편집기를 종료하려면 **Esc** 키를 누르고 **x**를 입력한 다음 **Enter** 키를 누릅니다.

1. 변경 내용을 적용하려면 다음 명령을 사용하여 IoT Edge 디먼을 다시 시작해야 합니다.

    ```bash
    sudo systemctl restart iotedge
    ```

1. IoT Edge 디먼이 실행되고 있는지 확인하려면 다음 명령을 입력합니다.

    ```bash
    sudo systemctl status iotedge
    ```

    이 명령이 실행되면 여러 콘텐츠 줄이 표시됩니다. 그 중에서 처음 세 줄이 서비스 실행 여부를 나타냅니다. 실행 중인 서비스의 경우 출력은 다음과 같습니다.

    ```bash
    ● iotedge.service - Azure IoT Edge daemon
       Loaded: loaded (/lib/systemd/system/iotedge.service; enabled; vendor preset: enabled)
       Active: active (running) since Fri 2021-03-19 18:06:16 UTC; 1min 0s ago
    ```

1. IoT Edge 런타임이 연결되었는지 확인하려면 다음 명령을 실행합니다.

    ```bash
    sudo iotedge check
    ```

    그러면 여러 검사가 실행되어 결과가 표시됩니다. 이 랩에서 **Configuration checks** 경고/오류는 무시하면 됩니다. **Connectivity checks**는 성공하며, 다음과 같은 결과가 표시됩니다.

    ```bash
    Connectivity checks
    -------------------
    √ host can connect to and perform TLS handshake with IoT Hub AMQP port - OK
    √ host can connect to and perform TLS handshake with IoT Hub HTTPS / WebSockets port - OK
    √ host can connect to and perform TLS handshake with IoT Hub MQTT port - OK
    √ container on the default network can connect to IoT Hub AMQP port - OK
    √ container on the default network can connect to IoT Hub HTTPS / WebSockets port - OK
    √ container on the default network can connect to IoT Hub MQTT port - OK
    √ container on the IoT Edge module network can connect to IoT Hub AMQP port - OK
    √ container on the IoT Edge module network can connect to IoT Hub HTTPS / WebSockets port - OK
    √ container on the IoT Edge module network can connect to IoT Hub MQTT port - OK
    ```

    연결이 실패하면 **config.yaml**의 연결 문자열 값을 다시 확인합니다.

### 연습 5: Edge 디바이스에 Edge 모듈 추가

이 연습에서는 시뮬레이션된 온도 센서를 사용자 지정 IoT Edge 모듈로 추가하고 이를 배포하여 IoT Edge 디바이스에서 실행합니다.

1. 필요한 경우 Azure 계정 자격 증명을 사용하여 Azure Portal에 로그인합니다.

    Azure 계정이 두 개 이상인 경우 이 과정에 사용할 구독에 연결된 계정으로 로그인해야 합니다.

1. 리소스 그룹 타일에서 IoT Hub를 열려면 **iot-az220-training-{your-id}** 를 클릭합니다.

1. **IoT Hub** 블레이드 왼쪽의 **자동 장치 관리**에서 **IoT Edge**를 클릭합니다.

1. IoT Edge 디바이스 목록에서 **sensor-thl-0067**을 클릭합니다.

1. **sensor-th-0067** 블레이드의 **모듈** 탭에는 현재 디바이스용으로 구성된 모듈 목록이 표시됩니다.

    현재 IoT Edge 디바이스는 IoT Edge 런타임의 일부인 Edge Agent(`$edgeAgent`) 및 Edge Hub(`$edgeHub`) 모듈로만 구성됩니다.

1. **sensor-th-0067** 블레이드 상단에서 **모듈 설정**을 클릭합니다.

1. **디바이스에서 모듈 설정: sensor-th-0067** 블레이드에서 **IoT Edge 모듈** 섹션을 찾습니다.

1. **IoT Edge 모듈**에서 **추가**를 클릭한 다음 **IoT Edge 모듈**을 클릭합니다.

1. **IoT Edge 모듈 추가** 창의 **IoT Edge 모듈 이름**에 **tempsensor**를 입력합니다.

    사용자 정의 모듈의 이름을 "tempsensor"로 지정할 것입니다.

1. **이미지 URI**에 **asaedgedockerhubtest/asa-edge-test-module:simulated-temperature-sensor**를 입력합니다.

    > **참고**: 이 이미지는 이 테스트 시나리오를 지원하기 위해 제품 그룹에서 제공하는 Docker Hub에 게시된 이미지입니다.

1. 선택한 탭을 변경하려면 **모듈 쌍 설정**을 클릭합니다.

1. 모듈 쌍에 원하는 속성을 지정하려면 다음 JSON을 입력합니다.

    ```json
    {
        "EnableProtobufSerializer": false,
        "EventGeneratingSettings": {
            "IntervalMilliSec": 500,
            "PercentageChange": 2,
            "SpikeFactor": 2,
            "StartValue": 20,
            "SpikeFrequency": 20
        }
    }
    ```

    이 JSON은 모듈 쌍을 원하는 속성으로 설정하여 Edge 모듈을 구성합니다.

1. 블레이드 하단의 **추가**를 클릭하세요.

1. **디바이스에서 모듈 설정: sensor-th-0067** 블레이드의 하단에서 **다음: 경로 >** 를 클릭합니다.

1. 기본 라우팅이 이미 구성되어 있는 것을 알 수 있습니다.

    * 이름: **라우팅**
    * 값: `FROM /messages/* INTO $upstream`

    이 라우팅은 IoT Edge 디바이스의 모든 모듈에서 IoT Hub로 모든 메시지를 전송합니다.

1. **검토 + 만들기**를 클릭합니다.

1. 잠시 시간을 내어 **배포**에서 표시되는 배포 매니페스트를 검토합니다.

    보시다시피 IoT Edge 디바이스의 배포 매니페스트는 JSON으로 서식이 지정되어 있어 읽기가 매우 쉽습니다.

    `properties.desired` 섹션에는 IoT Edge 디바이스에 배포될 IoT Edge 모듈을 선언하는 `modules` 섹션이 있습니다. 여기에는 Container Registry 자격 증명을 포함한 모든 모듈의 이미지 URI가 포함됩니다.

    ```json
    {
        "modulesContent": {
            "$edgeAgent": {
                "properties.desired": {
                    "modules": {
                        "tempsensor": {
                            "settings": {
                                "image": "asaedgedockerhubtest/asa-edge-test-module:simulated-temperature-sensor",
                                "createOptions": ""
                            },
                            "type": "docker",
                            "status": "running",
                            "restartPolicy": "always",
                            "version": "1.0"
                       },
    ```

    JSON의 아래쪽에는 Edge 허브의 원하는 속성이 포함된 **$edgeHub** 섹션이 있습니다. 이 섹션에는 모듈 간 및 IoT Hub로 이벤트를 라우팅하기 위한 라우팅 구성도 포함되어 있습니다.

    ```json
        "$edgeHub": {
            "properties.desired": {
                "routes": {
                  "route": "FROM /messages/* INTO $upstream"
                },
                "schemaVersion": "1.1",
                "storeAndForwardConfiguration": {
                    "timeToLiveSecs": 7200
                }
            }
        },
    ```

    JSON의 하단은 **tempsensor** 모듈 섹션으로, `properties.desired` 섹션에 원하는 Edge 모듈 구성의 속성이 포함되어 있습니다.

    ```json
                },
                "tempsensor": {
                    "properties.desired": {
                        "EnableProtobufSerializer": false,
                        "EventGeneratingSettings": {
                            "IntervalMilliSec": 500,
                            "PercentageChange": 2,
                            "SpikeFactor": 2,
                            "StartValue": 20,
                            "SpikeFrequency": 20
                        }
                    }
                }
            }
        }
    ```

1. 블레이드 하단에서 디바이스 모듈 설정을 완료하려면 **만들기**를 클릭합니다.

1. **sensor-th-0067** 블레이드의 **모듈**에서 **tempsensor** 센서가 나열되어 있음을 알 수 있습니다.

    > **참고**: 처음으로 나열된 모듈을 확인하려면 **새로 고침**을 클릭해야 할 수도 있습니다.

    **tempsensor**의 런타임 상태가 보고되지 않는 것을 알 수 있습니다.

1. 블레이드 상단의 **새로 고침**을 클릭합니다.

1. **tempsensor** 모듈의 **런타임 상태**가 이제 **실행 중**으로 설정된 것을 알 수 있습니다.

    값이 아직 보고되지 않으면 잠시 기다렸다가 블레이드를 다시 새로 고칩니다.

1. Cloud Shell 세션을 엽니다(아직 열려 있지 않은 경우).

    `vm-az220-training-edge0001-{your-id}` 가상 머신에 더 이상 연결되어 있지 않은 경우 이전과 같이 **SSH**를 사용하여 연결합니다.

1. Cloud Shell 명령 프롬프트에서 현재 IoT Edge 디바이스에서 실행 중인 모듈을 나열하려면 다음 명령을 입력합니다.

    ```bash
    iotedge list
    ```

1. 명령의 출력은 다음과 유사합니다.

    ```bash
    demouser@vm-az220-training-edge0001-{your-id}:~$ iotedge list
    NAME             STATUS           DESCRIPTION      CONFIG
    edgeHub          running          Up a minute      mcr.microsoft.com/azureiotedge-hub:1.1
    edgeAgent        running          Up 26 minutes    mcr.microsoft.com/azureiotedge-agent:1.1
    tempsensor       running          Up 34 seconds    asaedgedockerhubtest/asa-edge-test-module:simulated-temperature-sensor
    ```

    `tempsensor`가 실행 중인 모듈 중 하나로 나열된 것을 알 수 있습니다.

1. 모듈 로그를 보려면 다음 명령을 입력하세요.

    ```bash
    iotedge logs tempsensor
    ```

    해당 명령 출력 시 다음과 비슷합니다.

    ```bash
    demouser@vm-az220-training-edge0001-{your-id}:~$ iotedge logs tempsensor
    11/14/2019 18:05:02 - Send Json Event : {"machine":{"temperature":41.199999999999925,"pressure":1.0182182583425192},"ambient":{"temperature":21.460937846433808,"humidity":25},"timeCreated":"2019-11-14T18:05:02.8765526Z"}
    11/14/2019 18:05:03 - Send Json Event : {"machine":{"temperature":41.599999999999923,"pressure":1.0185790159334602},"ambient":{"temperature":20.51992724976499,"humidity":26},"timeCreated":"2019-11-14T18:05:03.3789786Z"}
    11/14/2019 18:05:03 - Send Json Event : {"machine":{"temperature":41.999999999999922,"pressure":1.0189397735244012},"ambient":{"temperature":20.715225311096397,"humidity":26},"timeCreated":"2019-11-14T18:05:03.8811372Z"}
    ```

    `iotedge logs` 명령을 사용하여 모든 Edge 모듈의 모듈 로그를 볼 수 있습니다.

1. 시뮬레이션된 온도 센서 모듈은 500개의 메시지를 보낸 후 중지됩니다. 다음 명령을 실행하여 재시작할 수 있습니다.

    ```bash
    iotedge restart tempsensor
    ```

    지금 모듈을 재시작할 필요는 없지만 만일 나중에 원격 분석 전송이 중지된다면 Cloud Shell로 돌아가고 Edge VM으로 SSH한 뒤 이 명령을 실행하여 재설정합니다. 재설정되면 모듈이 원격 분석을 다시 전송하기 시작합니다.

### 연습 6: Azure Stream Analytics를 IoT Edge 모듈로 배포

이제 tempSensor 모듈이 IoT Edge 디바이스에서 배포되어 실행되고 있으므로 IoT Edge 디바이스에서 메시지를 처리할 수 있는 Stream Analytics 모듈을 추가하여 IoT Hub로 전송할 수 있습니다.

#### 작업 1: Azure Storage 계정 만들기

1. 필요한 경우 Azure 계정 자격 증명을 사용하여 Azure Portal에 로그인합니다.

    Azure 계정이 두 개 이상인 경우 이 과정에 사용할 구독에 연결된 계정으로 로그인해야 합니다.

1. Azure Portal 메뉴에서 **+ 리소스 만들기**를 클릭합니다.

1. **새** 블레이드의 검색 텍스트 상자에 **저장소**를 입력하고 **Enter 키**를 누릅니다.

1. **Marketplace** 블레이드에서 **스토리지 계정**을 클릭합니다.

1. **스토리지 계정** 블레이드에서 **만들기**를 클릭합니다.

1. **스토리지 계정 만들기** 블레이드의 구독 드롭다운에 이 과정에 사용 중인 구독이 표시되어 있는지 확인합니다.

1. **리소스 그룹** 드롭다운에서 **rg-az220**을 클릭합니다.

1. **스토리지 계정 이름** 텍스트 상자에 **az220store{your-id}** 를 입력합니다.

    > **참고**: 이 필드의 {your-id}는 소문자로 입력해야 하며 하이픈 또는 밑줄 문자는 입력할 수 없습니다.

1. **위치** 필드를 Azure IoT Hub에 사용되는 것과 동일한 Azure 지역으로 설정합니다.

1. **복제** 필드를 **LRS(로컬 중복 저장소)** 로 설정합니다.

1. 다른 모든 설정은 그대로 둡니다.

1. 블레이드 하단의 **검토 + 생성**을 클릭하세요.

1. **유효성 검사 통과** 메시지가 표시될 때까지 기다린 다음 **만들기**를 클릭합니다.

    배포를 완료하는 데 몇 분 정도 걸릴 수 있으며, 생성되는 동안 계속 Stream Analytics 리소스를 만들 수 있습니다.

#### 작업 2: Azure Stream Analytics 작업 만들기

1. Azure Portal 메뉴에서 **+ 리소스 만들기**를 클릭합니다.

1. **신규** 블레이드의 **Azure Marketplace**에서 **사물 인터넷**을 클릭한 다음 **Stream Analytics 작업**을 클릭합니다.

    > **참고**:
    > **모두 보기**를 클릭하여 **Stream Analytics 작업**이 목록에 표시될 수도 있습니다.

1. **새 Stream Analytics 작업** 블레이드의 **작업 이름** 필드에 **asa-az220-training-{your-id}** 를 입력합니다.

1. **리소스 그룹** 드롭다운에서 **rg-az220**을 클릭합니다.

1. **위치** 드롭다운에서 스토리지 계정 및 Azure IoT Hub에 사용되는 것과 동일한 Azure 지역을 선택합니다.

1. **호스팅 환경** 필드를 **Edge**로 설정합니다.

    이렇게 하면 Stream Analytics 작업이 온-프레미스 IoT 게이트웨이 Edge 디바이스에 배포됩니다.

1. 블레이드 하단의 **만들기**를 클릭합니다.

    이 리소스를 배포하려면 몇 분 정도 걸립니다.

#### 작업 3: Azure Stream Analytics 작업 구성

1. **배포가 완료되었습니다**라는 메시지가 표시되면 **리소스로 이동**을 클릭합니다.

    이제 새 Stream Analytics 작업의 개요 창에 있어야 합니다.

1. 왼쪽 탐색 메뉴의 **작업 토폴로지**에서 **입력**을 클릭합니다.

1. **입력** 창에서 **스트림 입력 추가**를 클릭한 다음 **Edge 허브**를 클릭합니다.

1. **Edge 허브** 창에서 **입력 별칭** 필드에 **온도**를 입력합니다.

1. **이벤트 serialization 형식** 드롭다운에서 **JSON**이 선택되어 있는지 확인합니다.

    Stream Analytics가 메시지 형식을 이해해야 합니다. JSON은 표준 형식입니다.

1. **인코딩** 드롭다운에서 **UTF-8**이 선택되어 있는지 확인합니다.

    > **참고**:  UTF-8은 이 문서가 작성되는 시점에서 유일하게 지원되는 JSON 인코딩입니다.

1. **이벤트 압축 형식** 드롭다운에서 **없음**을 선택했는지 확인합니다.

    이 랩에서는 압축을 사용하지 않습니다. GZip 및 Deflate 형식도 서비스에서 지원됩니다.

1. 창 하단에서 **저장**을 클릭합니다.

1. 왼쪽 탐색 메뉴의 **작업 토폴로지**에서 **출력**을 클릭합니다.

1. **출력** 창에서 **+ 추가**를 클릭한 다음 **Edge 허브**를 클릭합니다.

1. **Edge 허브** 창의 **출력 별칭** 필드에 **경고**를 입력합니다.

1. **이벤트 serialization 형식** 드롭다운에서 **JSON**이 선택되어 있는지 확인합니다.

    Stream Analytics가 메시지 형식을 이해해야 합니다. JSON이 표준 형식이지만 CSV 또한 서비스에서 지원됩니다.

1. **형식** 드롭다운에서 **줄로 구분됨**이 선택되어 있는지 확인합니다.

1. **인코딩** 드롭다운에서 **UTF-8**이 선택되어 있는지 확인합니다.

    > **참고**:  UTF-8은 이 문서가 작성되는 시점에서 유일하게 지원되는 JSON 인코딩입니다.

1. 창 하단에서 **저장**을 클릭합니다.

1. 왼쪽 탐색 메뉴의 **작업 토폴로지**에서 **쿼리**를 클릭합니다.

1. **쿼리** 창에서 기본 쿼리를 다음으로 바꿉니다.

    ```sql
    SELECT
        'reset' AS command
    INTO
        alert
    FROM
        temperature TIMESTAMP BY timeCreated
    GROUP BY TumblingWindow(second,15)
    HAVING Avg(machine.temperature) > 25
    ```

    이 쿼리는 `temperature` 입력으로 들어오는 이벤트를 살펴보고 15초의 Tumbling Windows별로 그룹화한 다음 해당 그룹내의 평균 온도 값이 25보다 큰지 확인합니다. 평균이 25보다 크면 이벤트와 함께 `alert` 출력에 `reset` 값으로 설정된 `command` 속성을 보냅니다.

    `TumblingWindow` 함수에 대한 자세한 내용은 다음 링크를 참조하세요. [https://docs.microsoft.com/ko-kr/stream-analytics-query/tumbling-window-azure-stream-analytics](https://docs.microsoft.com/ko-kr/stream-analytics-query/tumbling-window-azure-stream-analytics)

1. 쿼리 편집기 상단의 **쿼리 저장**을 클릭합니다.

#### 작업 4: 스토리지 계정 설정 구성

IoT Edge 디바이스에 배포할 Stream Analytics 작업을 준비하려면 Azure Blob Storage 컨테이너와 연결해야 합니다. 작업이 배포되면 작업 정의가 저장소 컨테이너로 내보내 집니다.

1. **Stream Analytics 작업** 블레이드에서 **구성**의 왼쪽 탐색 메뉴에서 **스토리지 계정 설정**을 클릭합니다.

1. **스토리지 계정 설정** 창에서 **저장소 계정 추가**를 클릭합니다.

1. **스토리지 계정 설정**에서 **구독에서 저장소 계정 선택**이 선택되어 있는지 확인하세요.

1. **스토리지 계정** 드롭다운에서 **az220store{your-id}** 스토리지 계정이 선택되어 있는지 확인합니다.

1. **컨테이너**에서 **새로 만들기**를 클릭한 다음 컨테이너의 이름을 **jobdefinition**으로 입력합니다.

1. 창 상단의 **저장**을 클릭합니다.

    변경 내용을 저장한다는 메시지가 표시되면 **예**를 클릭합니다.

#### 작업 5: Stream Analytics 작업 배포

1. Azure Portal에서 **iot-az220-training-{your-id}** IoT Hub 리소스로 이동합니다.

1. 왼쪽 탐색 메뉴의 **자동 장치 관리**에서 **IoT Edge**를 클릭합니다.

1. **디바이스 ID**에서 **sensor-th-0067**을 클릭합니다.

1. **sensor-th-0067** 창 상단에서 **모듈 설정**을 클릭합니다.

1. **디바이스에서 모듈 설정: sensor-th-0067** 창에서 **IoT Edge 모듈** 섹션을 찾습니다.

1. **IoT Edge 모듈**에서 **추가**를 클릭한 다음 **Azure Stream Analytics 모듈**을 클릭합니다.

1. **Edge 배포** 창의 **구독**에서 이 과정에 사용 중인 구독이 선택되어 있는지 확인합니다.

1. **Edge 작업** 드롭다운에서 **asa-az220-training-{your-id}** Steam Analytics 작업이 선택되었는지 확인합니다.

    > **참고**:  작업은 이미 선택되어 있지만 **저장** 단추는 비활성화되어 있습니다. **Edge 작업** 드롭다운을 다시 열고 **asa-az220-training-{your-id}** 작업을 다시 선택합니다. 그러면 **저장** 단추가 활성화됩니다.

1. 창 하단에서 **저장**을 클릭합니다.

    배포하는데 몇 분 정도 걸릴 수 있습니다.

1. Edge 패키지가 성공적으로 게시되면 새 ASA 모듈이 **IoT Edge 모듈** 섹션 아래에 나열됩니다.

1. **IoT Edge 모듈**에서 **asa-az220-training-{your-id}** 를 클릭합니다.

    방금 Edge 디바이스에 추가 된 Steam Analytics 모듈입니다.

1. **IoT Edge 모듈 업데이트** 창에서 **이미지 URI**는 표준 Azure Stream Analytics 이미지를 가리킨다는 점을 알 수 있습니다.

    ```text
    mcr.microsoft.com/azure-stream-analytics/azureiotedge:1.0.8
    ```

    IoT Edge 디바이스에 배포되는 모든 ASA 작업에 사용되는 것과 동일한 이미지입니다.

    > **참고**:  구성된 **이미지 URI** 끝에 있는 버전 번호는 Stream Analytics 모듈을 만들었을 때의 최신 버전을 반영합니다. 이 단원을 작성할 당시 버전은 `1.0.8`이었습니다.

1. 모든 값을 기본값으로 두고 **IoT Edge 사용자 지정 모듈** 창을 닫습니다.

1. **디바이스에서 모듈 설정: sensor-th-0067** 창에서 **다음: 경로 >** 를 클릭합니다.

    기존 라우팅이 표시되는 것을 알 수 있습니다.

1. 기본 라우팅을 정의된 다음 세 라우팅으로 바꾸세요.

    * Route 1
        * NAME: **telemetryToCloud**
        * 값: `FROM /messages/modules/tempsensor/* INTO $upstream`
    * Route 2
        * 이름: **alertsToReset**
        * 값: `FROM /messages/modules/asa-az220-training-{your-id}/* INTO BrokeredEndpoint("/modules/tempsensor/inputs/control")`
    * Route 3
        * 이름: **telemetryToAsa**
        * 값: `FROM /messages/modules/tempsensor/* INTO BrokeredEndpoint("/modules/asa-az220-training-{your-id}/inputs/temperature")`

    > **참고**: `asa-az220-training-{your-id}` 자리 표시자는 Azure Stream Analytics 작업 모듈의 이름으로 바꿔야 합니다. **이전**을 클릭하여 모듈 목록과 이름을 본 다음 **다음**을 클릭하여 이 단계로 돌아올 수 있습니다.

    정의되는 경로는 다음과 같습니다.

    * **telemetryToCloud** 경로는 `tempsensor` 모듈 출력에서 Azure IoT Hub로 모든 메시지를 보냅니다.
    * **alertsToReset** 경로는 Stream Analytics 모듈 출력에서 **tempsensor** 모듈의 입력으로 모든 경고 메시지를 보냅니다.
    * **telemetryToAsa** 경로는 `tempsensor` 모듈 출력에서 Stream Analytics 모듈 입력으로 모든 메시지를 보냅니다.

1. **디바이스에서 모듈 설정: sensor-th-0067** 블레이드 하단에서 **검토 + 만들기**를 클릭합니다.

1. **검토 + 만들기** 탭에서 **배포 매니페스트** JSON이 이제 Stream Analytics 모듈 및 방금 구성된 라우팅 정의로 업데이트된 것을 알 수 있습니다.

1. `tempsensor` 시뮬레이션된 온도 센서 모듈에 대한 JSON 구성을 확인합니다.

    ```json
    "tempsensor": {
        "settings": {
            "image": "asaedgedockerhubtest/asa-edge-test-module:simulated-temperature-sensor",
            "createOptions": ""
        },
        "type": "docker",
        "version": "1.0",
        "status": "running",
        "restartPolicy": "always"
    },
    ```

1. 이전에 구성된 라우팅의 JSON 구성과 JSON 배포 정의의 구성을 확인하세요.

    ```json
    "$edgeHub": {
        "properties.desired": {
            "routes": {
                "telemetryToCloud": "FROM /messages/modules/tempsensor/* INTO $upstream",
                "alertsToReset": "FROM /messages/modules/asa-az220-training-CP122619/* INTO BrokeredEndpoint(\\\"/modules/tempsensor/inputs/control\\\")",
                "telemetryToAsa": "FROM /messages/modules/tempsensor/* INTO BrokeredEndpoint(\\\"/modules/asa-az220-training-CP122619/inputs/temperature\\\")"
            },
            "schemaVersion": "1.0",
            "storeAndForwardConfiguration": {
                "timeToLiveSecs": 7200
            }
        }
    },
    ```

1. 블레이드 하단의 **만들기**를 클릭합니다.

#### 작업 6: 데이터 보기

1. **SSH**를 통해 **IoT Edge 디바이스**로 연결했던 **Cloud Shell** 세션으로 돌아갑니다.

    닫히거나 시간이 만료된 경우 다시 연결합니다. `SSH` 명령을 실행하고 이전과 같이 로그인합니다.

1. 명령 프롬프트에서 디바이스에 배포된 모듈 목록을 보려면 다음 명령을 입력합니다.

    ```bash
    iotedge list
    ```

    새로운 Stream Analytics 모듈을 IoT Edge 디바이스에 배포하는 데 1분 정도 걸릴 수 있습니다. 배포가 된 이후에는, 이 명령을 통해 출력된 목록에 표시됩니다.

    ```bash
    demouser@vm-az220-training-edge0001-{your-id}:~$ iotedge list
    NAME               STATUS           DESCRIPTION      CONFIG
    asa-az220-training-CP1119  running          Up a minute      mcr.microsoft.com/azure-stream-analytics/azureiotedge:1.0.5
    edgeAgent          running          Up 6 hours       mcr.microsoft.com/azureiotedge-agent:1.0
    edgeHub            running          Up 4 hours       mcr.microsoft.com/azureiotedge-hub:1.0
    tempsensor         running          Up 4 hours       asaedgedockerhubtest/asa-edge-test-module:simulated-temperature-sensor
    ```

    > **참고**:  Stream Analytics 모듈이 목록에 나타나지 않으면 1~2분 정도 기다린 후 다시 시도하세요. IoT Edge 디바이스에서 모듈 배포를 업데이트하는 데 1분 정도 걸릴 수 있습니다.

1. 명령 프롬프트에서 `tempsensor` 모듈로 인해 Edge 디바이스에서 전송되는 원격 분석을 보려면 다음 명령을 입력하세요.

    ```bash
    iotedge logs tempsensor
    ```

1. 잠시 시간을 내어 출력을 관찰하세요.

    **tempsensor**에서 전송되는 온도 원격 분석을 보면서 `machine.temperature`가 평균 `25`가 넘는 온도에 도달하면 Stream Analytics 작업에서 **재설정** 명령이 전송되는 것을 확인하세요. 이 작업은 Stream Analytics 작업 쿼리에서 구성된 작업입니다.

    이 이벤트의 출력은 다음과 유사합니다.

    ```bash
    11/14/2019 22:26:44 - Send Json Event : {"machine":{"temperature":231.599999999999959,"pressure":1.0095600761599359},"ambient":{"temperature":21.430643635304012,"humidity":24},"timeCreated":"2019-11-14T22:26:44.7904425Z"}
    11/14/2019 22:26:45 - Send Json Event : {"machine":{"temperature":531.999999999999957,"pressure":1.0099208337508767},"ambient":{"temperature":20.569532965342297,"humidity":25},"timeCreated":"2019-11-14T22:26:45.2901801Z"}
    Received message
    Received message Body: [{"command":"reset"}]
    Received message MetaData: {"MessageId":null,"To":null,"ExpiryTimeUtc":"0001-01-01T00:00:00","CorrelationId":null,"SequenceNumber":0,"LockToken":"e0e778b5-60ff-4e5d-93a4-ba5295b995941","EnqueuedTimeUtc":"0001-01-01T00:00:00","DeliveryCount":0,"UserId":null,"MessageSchema":null,"CreationTimeUtc":"0001-01-01T00:00:00","ContentType":"application/json","InputName":"control","ConnectionDeviceId":"sensor-th-0067","ConnectionModuleId":"asa-az220-training-CP1119","ContentEncoding":"utf-8","Properties":{},"BodyStream":{"CanRead":true,"CanSeek":false,"CanWrite":false,"CanTimeout":false}}
    Resetting temperature sensor..
    11/14/2019 22:26:45 - Send Json Event : {"machine":{"temperature":320.4,"pressure":0.99945886361358849},"ambient":{"temperature":20.940019742324957,"humidity":26},"timeCreated":"2019-11-14T22:26:45.7931201Z"}
    ```
