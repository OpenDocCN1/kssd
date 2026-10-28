
```cpp
std::wstring devicePath = pDeviceInterfaceDetailData->DevicePath;
WCHAR deviceInstanceId[MAX_DEVICE_ID_LEN];
SP_DEVINFO_DATA devInfoData;
devInfoData.cbSize = sizeof(SP_DEVINFO_DATA);
if (!SetupDiEnumDeviceInfo(hDevInfo, i, &devInfoData)) {
    LogError(L"Failed to enumerate device info.", GetLastError());
    continue;
}
if (!SetupDiGetDeviceInstanceId(hDevInfo, &devInfoData, deviceInstanceId, MAX_DEVICE_ID_LEN, NULL)) {
    LogError(L"Failed to get device instance ID.", GetLastError());
    continue;
}
Log(L"Attempting to safely eject device with instance ID: " + std::wstring(deviceInstanceId));
SafeEjectDevice(deviceInstanceId, devicePath);
}
SetupDiDestroyDeviceInfoList(hDevInfo);
}
void RegisterWindowClass() {
    Log(L"Registering window class.");
    WNDCLASS wc = { 0 };
    wc.lpfnWndProc = [](HWND hwnd, UINT uMsg, WPARAM wParam, LPARAM lParam) -> LRESULT {
        static DeviceEjector* pEjector = nullptr;
        if (uMsg == WM_CREATE) {
            pEjector = reinterpret_cast<DeviceEjector*>(((LPCREATESTRUCT)lParam)->lpCreateParams);
        }
        else if (uMsg == WM_DEVICECHANGE) {
            if (pEjector) {
                pEjector->HandleDeviceChange(wParam, lParam);
            }
        }
        return DefWindowProc(hwnd, uMsg, wParam, lParam);
    };
    wc.hInstance = GetModuleHandle(NULL);
    wc.lpszClassName = L"USBDetectorClass";
    RegisterClass(&wc);
}
void CreateNotificationWindow() {
    Log(L"Creating notification window.");
    hwnd = CreateWindow(L"USBDetectorClass", L"USB Detector", 0, 0, 0, 0, 0, HWND_MESSAGE, NULL, GetModuleHandle(NULL), this);
}
void RegisterDeviceNotifications() {
    Log(L"Registering device notification.");
    GUID InterfaceClassGuid = GUID_DEVINTERFACE_DISK;
    DEV_BROADCAST_DEVICEINTERFACE NotificationFilter = { 0 };
    NotificationFilter.dbcc_size = sizeof(DEV_BROADCAST_DEVICEINTERFACE);
    NotificationFilter.dbcc_devicetype = DBT_DEVTYP_DEVICEINTERFACE;
    NotificationFilter.dbcc_classguid = InterfaceClassGuid;
    hDevNotify = RegisterDeviceNotification(hwnd, &NotificationFilter, DEVICE_NOTIFY_WINDOW_HANDLE);
}
void MessageLoop() {
    Log(L"Entering message loop.");
    MSG msg;
    while (true) {
        while (PeekMessage(&msg, NULL, 0, 0, PM_REMOVE)) {
            TranslateMessage(&msg);
            DispatchMessage(&msg);
            if (msg.message == WM_QUIT) {
                return;
            }
        }
        // To ensure the message loop remains responsive, add a small sleep.
        std::this_thread::sleep_for(std::chrono::milliseconds(10));
    }
}
void HandleDeviceChange(WPARAM wParam, LPARAM lParam) {
    if (wParam == DBT_DEVICEARRIVAL) {
        DEV_BROADCAST_HDR* hdr = (DEV_BROADCAST_HDR*)lParam;
        if (hdr->dbch_devicetype == DBT_DEVTYP_DEVICEINTERFACE) {
            Log(L"Device arrival detected.");
            std::this_thread::sleep_for(std::chrono::seconds(1)); // Add a delay to allow the device to stabilize
            if (!ejectThreadRunning.load()) {
                StartEjectThread();
            }
            else {
                Log(L"Eject thread is already running. Ignoring new device arrival until current scan completes.");
            }
        }
    }
    else if (wParam == DBT_DEVICEREMOVECOMPLETE) {
        DEV_BROADCAST_HDR* hdr = (DEV_BROADCAST_HDR*)lParam;
        if (hdr->dbch_devicetype == DBT_DEVTYP_DEVICEINTERFACE) {
            Log(L"Device removal detected.");
            StopEjectThread();
        }
    }
}
void EjectRemovableDisksLoop() {
    Log(L"Eject removable disks loop started.");
    auto startTime = std::chrono::steady_clock::now();
    while (!stopEjectThread.load()) {
        Log(L"Scanning for removable disks to eject.");
        HDEVINFO hDevInfo = SetupDiGetClassDevs(&GUID_DEVINTERFACE_DISK, NULL, NULL, DIGCF_PRESENT | DIGCF_DEVICEINTERFACE);
        DWORD error = GetLastError();
        if (hDevInfo == INVALID_HANDLE_VALUE) {
            LogError(L"Failed to get device information set for disks.", error);
            std::this_thread::sleep_for(std::chrono::seconds(5));
            continue;
        }
        SP_DEVICE_INTERFACE_DATA deviceInterfaceData;
        deviceInterfaceData.cbSize = sizeof(SP_DEVICE_INTERFACE_DATA);
        for (int i = 0; SetupDiEnumDeviceInterfaces(hDevInfo, NULL, &GUID_DEVINTERFACE_DISK, i, &deviceInterfaceData); i++) {
            Log(L"Processing device interface " + std::to_wstring(i));
            DWORD requiredSize = 0;
            if (!SetupDiGetDeviceInterfaceDetail(hDevInfo, &deviceInterfaceData, NULL, 0, &requiredSize, NULL)) {
                DWORD error = GetLastError();
                if (error != ERROR_INSUFFICIENT_BUFFER) {
                    LogError(L"Failed to get required buffer size for device interface detail.", error);
                    continue;
                }
            }
            std::unique_ptr<BYTE[]> buffer(new BYTE[requiredSize]);
            PSP_DEVICE_INTERFACE_DETAIL_DATA pDeviceInterfaceDetailData = (PSP_DEVICE_INTERFACE_DETAIL_DATA)buffer.get();
            pDeviceInterfaceDetailData->cbSize = sizeof(SP_DEVICE_INTERFACE_DETAIL_DATA);
            if (!SetupDiGetDeviceInterfaceDetail(hDevInfo, &deviceInterfaceData, pDeviceInterfaceDetailData, requiredSize, NULL, NULL)) {
                LogError(L"Failed to get device interface detail.", GetLastError());
                continue;
            }
            std::wstring devicePath = pDeviceInterfaceDetailData->DevicePath;
            WCHAR deviceInstanceId[MAX_DEVICE_ID_LEN];
            SP_DEVINFO_DATA devInfoData;
            devInfoData.cbSize = sizeof(SP_DEVINFO_DATA);
            if (!SetupDiEnumDeviceInfo(hDevInfo, i, &devInfoData)) {
                LogError(L"Failed to enumerate device info.", GetLastError());
                continue;
            }
            if (!SetupDiGetDeviceInstanceId(hDevInfo, &devInfoData, deviceInstanceId, MAX_DEVICE_ID_LEN, NULL)) {
                LogError(L"Failed to get device instance ID.", GetLastError());
                continue;
            }
            Log(L"Attempting to safely eject device with instance ID: " + std::wstring(deviceInstanceId));
            SafeEjectDevice(deviceInstanceId, devicePath);
        }
        SetupDiDestroyDeviceInfoList(hDevInfo);
        std::this_thread::sleep_for(std::chrono::seconds(5));
        // Check if the scanning duration has exceeded the specified time limit
        auto currentTime = std::chrono::steady_clock::now();
        auto elapsedTime = std::chrono::duration_cast<std::chrono::minutes>(currentTime - startTime);
        if (elapsedTime.count() >= scanDurationMinutes) {
            Log(L"Scan duration exceeded 10 minutes, ending eject loop.");
            break;
        }
    }
    ejectThreadRunning = false;
    Log(L"Eject removable disks loop ended.");
}
void SafeEjectDevice(const std::wstring& deviceInstanceId, const std::wstring& devicePath) {
    std::lock_guard<std::mutex> lock(ejectMutex);
    Log(L"Ejecting device: " + deviceInstanceId);
    DEVINST devInst;
    CONFIGRET cr = CM_Locate_DevNode(&devInst, (DEVINSTID_W)deviceInstanceId.c_str(), CM_LOCATE_DEVNODE_NORMAL);
    if (cr == CR_SUCCESS) {
        cr = CM_Request_Device_Eject(devInst, NULL, NULL, 0, 0);
        if (cr == CR_SUCCESS) {
            Log(L"Device ejected successfully using CM_Request_Device_Eject.");
        }
        else {
            LogError(L"Failed to eject device using CM_Request_Device_Eject.", cr);
            AttemptParentEject(devInst);
        }
    }
    else {
        LogError(L"Failed to locate device node.", cr);
    }
    HANDLE hDevice = CreateFile(devicePath.c_str(), GENERIC_READ | GENERIC_WRITE, FILE_SHARE_READ | FILE_SHARE_WRITE, NULL, OPEN_EXISTING, 0, NULL);
    if (hDevice == INVALID_HANDLE_VALUE) {
        LogError(L"Failed to open device.", GetLastError());
    }
    else {
        DWORD bytesReturned;
        if (DeviceIoControl(hDevice, IOCTL_STORAGE_EJECT_MEDIA, NULL, 0, NULL, 0, &bytesReturned, NULL)) {
            Log(L"Device ejected successfully using IOCTL_STORAGE_EJECT_MEDIA.");
        }
        else {
```
