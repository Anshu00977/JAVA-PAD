#include <windows.h>
#include <commdlg.h>

#define ID_FILE_NEW 1
#define ID_FILE_OPEN 2
#define ID_FILE_SAVE 3
#define ID_FILE_EXIT 4

LRESULT CALLBACK WindowProc(HWND hwnd, UINT uMsg, WPARAM wParam, LPARAM lParam) {
    static HWND hwndEdit;
    static OPENFILENAME ofn;
    static TCHAR szFile[260];
    static TCHAR szFileTitle[260];
    static HANDLE hFile;
    static DWORD dwBytesRead;
    static TCHAR buffer[1024];
    
    switch (uMsg) {
        case WM_CREATE:
            hwndEdit = CreateWindowEx(
                0, TEXT("EDIT"), NULL,
                WS_CHILD | WS_VISIBLE | WS_VSCROLL | WS_HSCROLL | ES_MULTILINE | ES_AUTOVSCROLL | ES_AUTOHSCROLL,
                0, 0, 0, 0,
                hwnd, NULL, ((LPCREATESTRUCT)lParam)->hInstance, NULL
            );
            
            // Initialize OPENFILENAME structure
            ZeroMemory(&ofn, sizeof(ofn));
            ofn.lStructSize = sizeof(ofn);
            ofn.hwndOwner = hwnd;
            ofn.lpstrFile = szFile;
            ofn.nMaxFile = sizeof(szFile);
            ofn.lpstrFileTitle = szFileTitle;
            ofn.nMaxFileTitle = sizeof(szFileTitle);
            ofn.lpstrFilter = TEXT("Text Files\0*.TXT\0All Files\0*.*\0");
            ofn.nFilterIndex = 1;
            ofn.lpstrFileTitle = NULL;
            ofn.Flags = OFN_PATHMUSTEXIST | OFN_FILEMUSTEXIST | OFN_OVERWRITEPROMPT;
            break;
        
        case WM_COMMAND:
            switch (LOWORD(wParam)) {
                case ID_FILE_NEW: {
                    SetWindowText(hwndEdit, TEXT(""));
                } break;
                
                case ID_FILE_OPEN: {
                    if (GetOpenFileName(&ofn)) {
                        hFile = CreateFile(
                            ofn.lpstrFile,
                            GENERIC_READ,
                            0,
                            NULL,
                            OPEN_EXISTING,
                            FILE_ATTRIBUTE_NORMAL,
                            NULL
                        );
                        if (hFile != INVALID_HANDLE_VALUE) {
                            ReadFile(hFile, buffer, sizeof(buffer) - 1, &dwBytesRead, NULL);
                            buffer[dwBytesRead] = '\0';
                            SetWindowText(hwndEdit, buffer);
                            CloseHandle(hFile);
                        }
                    }
                } break;
                
                case ID_FILE_SAVE: {
                    if (GetSaveFileName(&ofn)) {
                        hFile = CreateFile(
                            ofn.lpstrFile,
                            GENERIC_WRITE,
                            0,
                            NULL,
                            CREATE_ALWAYS,
                            FILE_ATTRIBUTE_NORMAL,
                            NULL
                        );
                        if (hFile != INVALID_HANDLE_VALUE) {
                            GetWindowText(hwndEdit, buffer, sizeof(buffer) - 1);
                            DWORD dwBytesWritten;
                            WriteFile(hFile, buffer, lstrlen(buffer) * sizeof(TCHAR), &dwBytesWritten, NULL);
                            CloseHandle(hFile);
                        }
                    }
                } break;
                
                case ID_FILE_EXIT:
                    PostQuitMessage(0);
                    break;
            }
            break;
        
        case WM_SIZE:
            {
                RECT rcClient;
                GetClientRect(hwnd, &rcClient);
                SetWindowPos(hwndEdit, NULL, 0, 0, rcClient.right, rcClient.bottom, SWP_NOZORDER | SWP_NOACTIVATE);
            }
            break;
        
        case WM_DESTROY:
            PostQuitMessage(0);
            break;
        
        default:
            return DefWindowProc(hwnd, uMsg, wParam, lParam);
    }
    
    return 0;
}

int WINAPI WinMain(HINSTANCE hInstance, HINSTANCE hPrevInstance, LPSTR lpCmdLine, int nCmdShow) {
    const TCHAR CLASS_NAME[] = TEXT("Sample Window Class");
    
    WNDCLASS wc = {0};
    wc.lpfnWndProc = WindowProc;
    wc.hInstance = hInstance;
    wc.lpszClassName = CLASS_NAME;
    
    RegisterClass(&wc);
    
    HWND hwnd = CreateWindowEx(
        0,
        CLASS_NAME,
        TEXT("Basic Notepad"),
        WS_OVERLAPPEDWINDOW,
        CW_USEDEFAULT, CW_USEDEFAULT, CW_USEDEFAULT, CW_USEDEFAULT,
        NULL,
        NULL,
        hInstance,
        NULL
    );
    
    if (hwnd == NULL) {
        return 0;
    }
    
    ShowWindow(hwnd, nCmdShow);
    UpdateWindow(hwnd);
    
    MSG msg;
    while (GetMessage(&msg, NULL, 0, 0)) {
        TranslateMessage(&msg);
        DispatchMessage(&msg);
    }
    
    return 0;
}
