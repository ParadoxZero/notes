## How to make #xaml island container span across win32 windows?

```c++
GetClientRect(hWnd, &rcClient);

SetWindowPos(hWndXamlIsland, nullptr, rcClient.left, rcClient.top, rcClient.right - rcClient.left, rcClient.bottom - rcClient.top, SWP_NOZORDER);
```

