---
lab:
    title: '랩 03: 개발 환경 설정'
    module: '모듈 2: 디바이스 및 디바이스 통신'
---

# 개발 환경 설정

## 랩 시나리오

Contoso의 개발자 중 한 명으로서, Azure IoT 솔루션을 빌드하기 전에 개발 환경을 설정하는 것이 중요하다는 것을 알고 있습니다. Microsoft는 IoT 솔루션을 개발하고 지원하는 데 사용할 수 있는 여러 가지 도구를 제공하고 있으며, 팀에 어떤 도구를 사용할 것인지에 대해 결정을 내려야 한다는 사실을 알고 계실 것입니다. 팀에서 Azure 클라우드 측과 로컬 작업 환경 모두에서 IoT 솔루션을 개발하는 데 사용할 수 있는 작업 환경을 준비합니다.

논의 후 팀에서 개발 환경에 대해 다음과 같은 높은 수준의 결정을 내렸습니다.

* 운영 체제: Windows 10을 OS로 사용합니다. Windows는 대부분의 팀에서 사용되므로 합리적인 선택이었습니다. Azure IoT 서비스는 Mac OS 및 Linux와 같은 다른 운영 체제를 지원하며, Microsoft는 이러한 대안 중 하나를 선택하는 팀의 구성원에게 도움이 되는 설명서를 제공합니다.
* 일반 코딩 도구: Visual Studio Code 및 Azure CLI를 기본 코딩 도구로 사용합니다. 이 두 도구는 Azure IoT SDK를 활용하는 IoT용 확장 프로그램을 지원합니다.
* IoT Edge 도구: Docker 데스크톱 커뮤니티와 Python은 사용자 지정 IoT Edge 모듈 개발을 지원하는 데 사용됩니다.

이러한 결정을 지원하기 위해 다음과 같은 환경을 설정합니다.

* Windows 10 64비트: Pro, Enterprise 또는 Education(빌드 15063 이상). 다음 포함:
  * 4GB – 8GB 시스템 RAM(더 많을수록 Docker에 더 유리)
  * Windows의 Hyper-V 및 컨테이너 기능을 사용하도록 설정해야 합니다.
  * BIOS 수준의 하드웨어 가상화 지원은 BIOS 설정에서 활성화되어야 합니다.

  > **참고**: 가상 머신에서 개발 환경을 설정할 때 VM 환경은 중첩 가상화를 지원해야 합니다 - [중첩 가상화](https://docs.microsoft.com/ko-kr/virtualization/hyper-v-on-windows/user-guide/nested-virtualization)

* Azure CLI(현재/최신)
* .NET Core 3.1.200 (또는 이후) SDK
* VS Code(최신)
* Python 3.7(3.8 아님)
* Linux 컨테이너에 설정된 Docker 데스크톱 커뮤니티 2.1.0.5(이상)
* Power BI Desktop(데이터 시각화용)
* VS Code 및 Azure CLI용 IoT 확장 프로그램

> **참고**: 위에서 언급한 대부분의 도구를 제공하는 이 과정에 가상 머신이 만들어졌습니다. 아래 지침은 준비된 VM을 사용하거나 PC를 사용하여 로컬로 개발 환경을 설정하는 것을 지원합니다.

## 이 랩에서

이 랩에서는 다음을 다룹니다.

* 이 과정의 랩에서 사용할 기본 도구와 제품을 설치합니다.
* Azure CLI 및 Visual Studio Code에 대한 Azure IoT 확장을 설치합니다.
* 개발 환경 설정 확인

## 랩 지침

### 연습 1: 개발자 도구 및 제품 설치

> **참고**: 이 연습과 연관된 도구 및 제품은 이 과정을 위해 만들어진 가상 머신에 미리 설치되어 있습니다. 계속하기 전에 과정 강사에게 문의하여 호스팅된 랩 VM 환경을 사용하여 랩을 완료하거나 PC에서 로컬로 개발 환경을 설정할지 여부를 확인하세요.

#### 작업 1: .NET Core 설치

.NET Core는 웹 사이트, 서비스 및 콘솔 앱을 빌드하기 위한 .NET의 교차 플랫폼 버전입니다.

1. .NET Core 다운로드 페이지를 열려면 다음 링크를 사용하세요. [.NET 다운로드](https://dotnet.microsoft.com/download)

1. .NET 다운로드 페이지의 .NET Core 아래에서 **.NET Core SDK 다운로드**를 클릭합니다.

    .NET Core SDK는 .NET Core 앱을 빌드하는 데 사용됩니다. 이 과정의 랩에서는 코드 파일 빌드/편집을 사용합니다.

    Windows 컴퓨터에서 .NET Core를 사용하는 앱을 실행하기만 하면 .NET Core Runtime을 설치할 수 있습니다. 또한 .NET Core의 미리 보기 및 레거시 버전은 "모든 .NET Core 다운로드..."에 대한 링크를 사용하여 설치할 수 있습니다.

1. 팝업 메뉴에서 **실행**을 클릭한 다음 화면의 지침에 따라 설치를 완료합니다.

    설치를 완료하는 데에는 1분 미만이 소요됩니다. 다음 구성 요소가 설치됩니다.

    * .NET Core SDK 3.1.100 이상
    * .NET Core Runtime 3.1.100 이상
    * ASP.NET Core Runtime 3.1.100 이상
    * .NET Core Windows Desktop Runtime 3.1.0 이상

    자세한 내용은 다음 리소스를 참조하세요.

    * [.NET Core 설명서](https://aka.ms/dotnet-docs-kor)
    * [.NET Core 종속성 및 요구 사항](https://docs.microsoft.com/ko-kr/dotnet/core/install/dependencies?tabs=netcore31&pivots=os-windows)
    * [SDK 설명서](https://aka.ms/dotnet-sdk-docs-kor)
    * [릴리스 정보](https://aka.ms/netcore3releasenotes)
    * [자습서](https://aka.ms/dotnet-tutorials-kor)

#### 작업 2: Visual Studio Code 설치

Visual Studio Code는 데스크톱에서 실행되는 가벼우면서도 강력한 소스 코드 편집기이며 Windows, macOS 및 Linux에서 사용할 수 있습니다. JavaScript, TypeScript 및 Node.js를 기본적으로 지원하며 다른 언어(예: C++, C#, Java, Python, PHP, Go) 및 실행 시간(예: .NET 및 Unity)에 대한 풍부한 확장 에코시스템을 제공합니다.

1. Visual Studio Code 다운로드 페이지를 열려면 다음 링크를 클릭합니다. [Visual Studio Code 다운로드](https://code.visualstudio.com/Download)

    Mac OS X 및 Linux에 Visual Studio Code를 설치하는 방법은 Visual Studio Code 설정 가이드([여기](https://code.visualstudio.com/docs/setup/setup-overview))에서 찾을 수 있습니다. 이 페이지에는 Windows 설치에 대한 자세한 지침과 팁도 포함되어 있습니다.

1. Visual Studio Code 다운로드 페이지에서 **Windows**를 클릭합니다.

    다운로드를 시작하면 팝업 대화 상자가 열리고 시작 안내가 표시됩니다.

1. 팝업 대화 상자에서 설치 프로세스를 시작하려면 **실행**을 클릭한 다음 화면의 지침을 따릅니다.

    설치 관리자를 다운로드 폴더에 저장하도록 선택한 경우 폴더를 연 다음 VSCodeSetup 실행 파일을 두 번 클릭하여 설치를 완료할 수 있습니다.

    기본적으로 Visual Studio Code는 "C:\Program Files (x86)\Microsoft VS Code" 폴더 위치에 설치됩니다(64비트 컴퓨터의 경우). 이 설정 프로세스는 1분 정도 밖에 걸리지 않습니다.

    > **참고**:  .NET Framework 4.5는 Windows에 설치할 때 Visual Studio Code에 필요합니다. Windows 7을 사용하는 경우 [.NET Framework 4.5](https://www.microsoft.com/ko-kr/download/details.aspx?id=30653)가 설치되어 있는지 확인하세요.

    Visual Studio Code 설치에 대한 자세한 지침은 여기([https://code.visualstudio.com/Docs/editor/setup](https://code.visualstudio.com/Docs/editor/setup))에서 Microsoft Visual Studio Code 설치 지침을 참조하세요.

#### 작업 3: Azure CLI 설치

Azure CLI 2.2는 Azure 관련 작업을 보다 쉽게 스크립팅할 수 있도록 설계된 명령줄 도구입니다. 또한 데이터를 유연하게 쿼리할 수 있으며 비차단 프로세스로 장기 실행 작업을 지원합니다.

1. 브라우저를 연 다음 Azure CLI 2.2 도구 다운로드 페이지로 이동합니다. [Azure CLI 2.2 설치](https://docs.microsoft.com/ko-kr/cli/azure/install-azure-cli?view=azure-cli-latest "Azure CLI 2.2 Install")

    Azure CLI 도구의 최신 버전을 설치해야 합니다. 버전 2.2가 이 "azure-cli-latest" 다운로드 페이지에 나열된 최신 버전이 아닌 경우 최신 버전을 설치합니다.

1. Azure CLI 2.2 설치 페이지에서 OS에 맞는 설치 옵션을 선택한 다음 화면의 지침에 따라 Azure CLI 도구를 설치합니다.

    이 과정의 랩에서 Azure CLI 2.2 도구 사용에 대한 자세한 지침을 제공하지만 지금 자세한 정보를 원할 경우 [Azure CLI 2.2 시작](https://docs.microsoft.com/ko-kr/cli/azure/get-started-with-azure-cli?view=azure-cli-latest)을 참조하세요.

#### 작업 4: Python 3.7 설치

IoT Edge 및 Docker를 지원하기 위해 Python 3.7을 사용합니다. 

1. 웹 브라우저에서 [https://www.python.org/downloads/](https://www.python.org/downloads/)로 이동합니다.

1. "특정 릴리스를 찾고 있습니까?" 아래에서 Python 3.7.6 오른쪽에 있는 **다운로드**를 클릭합니다.

1. Python 3.7.6 페이지에서 페이지의 Files 섹션으로 스크롤합니다.

1. Files 아래에서 운영 체제에 적합한 설치 관리자 파일을 선택합니다.

1. 메시지가 표시되면 설치 관리자를 실행하는 옵션을 선택합니다.

1. Python 3.7.6 설치 대화 상자에서 **Python 3.7을 PATH에 추가**를 클릭합니다.

1. **지금 설치**를 클릭합니다.

1. "설정이 완료되었습니다" 페이지가 나타나면 **경로 길이 제한 사용 안 함**을 클릭합니다.

1. 설치 프로세스를 완료하려면 **닫기**를 클릭합니다.

#### 작업 5: Docker Desktop 설치

IoT Edge 모듈 배포를 다루는 랩에서 Docker Desktop Community 2.1.0.5(또는 이상)를 Linux 컨테이너로 설정하여 사용합니다.

1. 웹 브라우저에서 [https://docs.docker.com/docker-for-windows/install/](https://docs.docker.com/docker-for-windows/install/)로 이동합니다.

    왼쪽 탐색 메뉴는 추가적인 운영 체제 설치에 대한 액세스를 제공합니다.

1. PC가 시스템 요구 사항을 충족하는지 확인합니다.

    Windows 설정을 사용하여 Windows 기능 대화 상자를 열어 Hyper-V 및 컨테이너가 활성화되어 있는지 확인할 수 있습니다.

1. **Docker Hub에서 다운로드**를 클릭합니다.

1. Windows용 Docker Desktop 아래에서 **Windows용 Docker Desktop 가져오기(안정적)**를 클릭합니다.

1. 설치를 시작하려면 **실행**을 클릭합니다.

    Docker Desktop에 대한 설치 대화 상자가 나타나려면 다소 시간이 걸릴 수 있습니다.

1. 설치 성공 메시지가 나타나면 **닫기**를 클릭합니다.

    Docker Desktop은 설치 후 자동으로 시작되지 않습니다. Docker Desktop을 시작하려면 Docker를 검색하고 검색 결과에서 Docker Desktop을 선택합니다. 상태 표시줄의 고래 아이콘이 계속 그대로 있다면 Docker Desktop이 작동 중이며 모든 터미널 창에서 액세스할 수 있습니다.

### 연습 2: 개발 도구 확장 프로그램 설치

Visual Studio Code 및 Azure CLI 도구는 개발자가 솔루션을 보다 효율적으로 만들 수 있도록 도와주는 확장 프로그램을 지원합니다. Microsoft는 IoT SDK를 활용하고 개발 시간을 단축하는 IoT용 확장 프로그램을 개발했습니다.

#### 작업 1: Visual Studio Code 확장 설치

1. Visual Studio Code를 엽니다.

1. Visual Studio Code 창의 왼쪽에서 **확장**을 클릭합니다.

    확장 단추는 위에서 여섯 번째입니다. 단추 위에 마우스 포인터를 가져가 단추 제목을 표시할 수 있습니다.

1. Visual Studio Code 확장 관리자에서 다음 확장을 검색한 다음 설치합니다.

    * Microsoft가 제공하는 [Azure IoT Tools](https://marketplace.visualstudio.com/items?itemName=vsciot-vscode.azure-iot-tools)(`vsciot-vscode.azure-iot-tools`)
    * Microsoft가 제공하는 [Visual Studio Code용 C#](https://marketplace.visualstudio.com/items?itemName=ms-vscode.csharp)(`ms-vscode.csharp`)

#### 작업 2: Azure CLI 확장 설치

1. 새 명령줄/터미널 창을 엽니다.

1. 명령 프롬프트에서 IoT용 Azure CLI 확장을 설치하려면 다음 명령을 입력합니다.

    ```bash
    az extension add --name azure-cli-iot-ext
    ```

#### 작업 3: 개발 환경 설정 확인

개발 환경이 성공적으로 설정되었는지 확인해야 합니다. 이 작업이 완료되면 IoT 솔루션 빌드를 시작할 수 있습니다.

1. 새 명령줄/터미널 창을 엽니다.

1. 현재 설치된 Azure CLI 버전의 버전 정보를 출력하는 다음 명령을 실행하여 **Azure CLI** 설치의 유효성을 검사합니다.

    ```cmd/sh
    az --version
    ```

    `az --version` 명령은 설치한 Azure CLI의 버전 정보(`azure-cli` 버전 번호)를 출력합니다. 또한 이 명령은 IoT 확장을 포함하여 설치된 모든 Azure CLI 모듈의 버전 번호를 출력합니다. 다음과 유사한 출력이 표시됩니다.

    ```cmd/sh
    azure-cli                           2.2.0

    command-modules-nspkg               2.0.3
    core                                2.2.0
    nspkg                               3.0.4
    telemetry                           1.0.4

    Extensions:
    azure-cli-iot-ext                   0.8.9
    ```

1. 현재 설치된 .NET Core SDK 버전의 버전 번호를 출력하는 다음 명령을 실행하여 **NET Core 3.x SDK** 설치의 유효성을 검사합니다.

    ```cmd/sh
    dotnet --version
    ```

1. `dotnet --version` 명령은 현재 설치된 NET Core SDK 버전을 출력합니다. 이 경우 .NET Core 3.1 이상이어야 합니다.

이제 개발 환경이 설정되었습니다!

### 연습 3: 과정 랩 파일 및 대체 도구 설정

이 과정의 여러 랩은 랩 작업의 시작점으로 사용할 수 있는 코드 프로젝트 등 미리 빌드된 리소스에 의존합니다. GitHub 프로젝트를 사용하여 이러한 랩 리소스에 액세스할 수 있도록 해줍니다. 과정 랩을 직접 지원하는 리소스(GitHub 프로젝트에 포함된 리소스) 외에도, 실제 과정 밖에서도 학습 기회를 지원하는 데 사용할 수 있는 도구가 있습니다. 아래 지침은 이러한 두 리소스 유형의 구성을 알려줍니다.

#### 작업 1: 과정 랩 파일 다운로드

Microsoft는 랩 리소스 파일에 대한 액세스를 제공하기 위해 GitHub 리포지토리를 만들었습니다. 이러한 파일을 개발자 환경에 로컬로 두는 것은 경우에 따라 필요하며 많은 경우에 편리합니다. 이 작업에서는 개발 환경 내에서 리포지토리의 내용을 다운로드하고 추출합니다.

1. 웹 브라우저에서 다음 위치로 이동합니다. [https://github.com/MicrosoftLearning/AZ-220-Microsoft-Azure-IoT-Developer](https://github.com/MicrosoftLearning/AZ-220-Microsoft-Azure-IoT-Developer)

1. 페이지 오른쪽에서 **복제 또는 다운로드**를 클릭한 다음 **ZIP 다운로드**를 클릭합니다.

1. ZIP 파일을 개발 환경에 저장하려면 **저장**을 클릭합니다.

1. 파일이 저장되면 **폴더 열기**를 클릭합니다.

1. 저장된 ZIP 파일을 마우스 오른쪽 단추로 클릭한 다음 **압축 풀기**를 클릭합니다.

1.  **찾아보기**를 클릭한 다음 액세스하기 편리한 폴더 위치로 이동합니다. 

1. 파일 압축을 풀려면 **압축 풀기**를 클릭합니다.

    파일의 위치를 기록해 둡니다.

#### 작업 2: Azure PowerShell 모듈 설치

> **참고**: 이 과정의 랩 활동에서는 PowerShell을 사용하지 않지만 PowerShell을 사용하는 참조 문서에 샘플 코드가 표시될 수 있습니다. PowerShell 코드를 실행하려면 다음 지침을 사용하여 설치 단계를 완료할 수 있습니다.

Azure PowerShell은 PowerShell 명령줄에서 직접 Azure 리소스를 관리하기 위한 cmdlet 집합입니다. Azure PowerShell은 쉽게 배우고 시작할 수 있도록 설계되었지만 자동화를 위한 강력한 기능을 제공합니다. .NET Standard로 작성된 Azure PowerShell은 Windows에서는 PowerShell 5.1, 모든 플랫폼에서는 PowerShell 6.x 이상으로 작동합니다.

> **경고**:  Windows용 PowerShell 5.1에 대한 AzureRM 및 Az 모듈을 동시에 설치할 수 없습니다. 시스템에서 AzureRM을 계속 사용해야 하는 경우 PowerShell Core 6.x 이상용 Az 모듈을 설치합니다. 이렇게 하려면 PowerShell Core 6.x 이상을 설치한 다음 PowerShell Core 터미널에서 이러한 지침을 따릅니다.

1. Azure PowerShell 모듈을 현재 사용자를 위해서만 설치할지(권장 방법) 또는 모든 사용자를 위해 설치할지를 결정합니다.

1. 선택한 PowerShell 터미널을 시작합니다. 모든 사용자를 위해 설치하는 경우 **관리자로 실행**을 선택하거나 macOS 또는 Linux에서 **sudo** 명령을 사용하여 관리자 권한 PowerShell 세션을 시작해야 합니다.

1. 현재 사용자에 대해서만 설치하려면 다음 명령을 입력합니다.

    ```powershell
    Install-Module -Name Az -AllowClobber -Scope CurrentUser
    ```

    또는 시스템의 모든 사용자에 대해 설치하려면 다음 명령을 입력합니다.

    ```powershell
    Install-Module -Name Az -AllowClobber -Scope AllUsers
    ```

1. 기본적으로 PowerShell 갤러리는 PowerShellGet의 신뢰할 수 있는 리포지토리로 구성되지 않습니다. PSGallery를 처음 사용하는 경우 다음과 같은 메시지가 표시됩니다.

    ```output
    신뢰할 수 없는 리포지토리

    신뢰할 수 없는 리포지토리에서 모듈을 설치하고 있습니다. 이 리포지토리를 신뢰하는 경우 변경합니다.
    Set-PSRepository cmdlet를 실행하여 InstallationPolicy 값을 변경합니다.

    정말 'PSGallery'에서 모듈을 설치하시겠습니까?
    [Y] 예  [A] 모두 예  [N] 아니요  [L] 모두 아니요  [S] 일시 중단  [?] 도움말(기본값은 "N"):
    ```

1. 설치를 계속하려면 **예** 또는 **모두 예**라고 답변합니다.

    Az 모듈은 Azure PowerShell cmdlet에 대한 롤업 모듈입니다. 이를 설치하면 사용 가능한 모든 Azure Resource Manager 모듈을 다운로드하고 해당 cmdlet을 사용할 수 있습니다.

> **참고**: **Az** 모듈이 이미 설치되어 있는 경우 다음을 사용하여 최신 버전으로 업데이트할 수 있습니다.
> 
> ```powershell
> Update-Module -Name Az
> ```
