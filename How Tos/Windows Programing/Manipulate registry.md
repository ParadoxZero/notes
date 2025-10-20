# Set Values in Registry

Key can be:

- HKEY_CURRENT_USER
- HKEY_LOCAL_MACHINE

> **Note**: Storing data into  HKEY_LOCAL_MACHINE require the current context having elevated permission.

```c++
#include <Windows.h>

SHSetValue( HKEY Key, LPCWSTR SubKey, LPCWSTR RegistryKeyName, DWORD DataType, LPCVOID Data, DWORD Data_Size )

//example
HRESULT hr = HRESULT_FROM_WIN32(SHSetValue(REG_HIVE_DISPLAY_SETTINGS, REG_KEY_DISPLAY_SETTINGS, REG_KEY_DEFAULT_DISPLAY_BRIGHTNESS, REG_DWORD, &value, sizeof(value)));
```

# Fetching Registry Value

```c++
#include <Windows.h>

SHRegGetDWORD(
    HKEY hkey,
    _In_opt_ PCWSTR pwszSubKey, // nullptr or "" ok
    _In_opt_ PCWSTR pwszValue,  // nullptr or "" ok
    _Out_ DWORD* pdwData
)

// Example:

DWORD percent;

HRESULT hr = SHRegGetDWORD(
	REG_HIVE_DISPLAY_SETTINGS, 
	REG_KEY_DISPLAY_SETTINGS, 
	REG_KEY_DEFAULT_DISPLAY_BRIGHTNESS, 
	&defaultPercent
);
```