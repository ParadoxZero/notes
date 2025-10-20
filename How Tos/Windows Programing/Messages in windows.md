# Broadcast Window Message

Use the below function to send/broadcast message across windows

```
#include <Windows.h>

PostMessage(HWND hwnd, UNIT uMsg, LPARAM lparam, WPARAM wparam);

HWND_BROADCAST -> Broadcast to all top level windows
```

Example

```
PostMessage(HWND_BROADCAST, DefaultBrightnessChangedMsg, 0, defaultPercentage);
```

# Creating Custom Window Messages

Use below function to get a unique UINT to represent your message. Same UINT is returned for same string for a session.

```
#include <Windows.h>

UINT RegisterWindowMessage(UNIQUE_STR_ID)
```

Example - 

```
#define PPI_MSG_BRIGHTNESS_CHANGED "PPIBrightnessChanged"

UINT msg = RegisterWindowMessageA(PPI_MSG_BRIGHTNESS_CHANGED)
```