Clipped from: [https://stackoverflow.com/questions/2251578/how-do-i-get-the-selected-text-from-the-focused-window-using-native-win32-api](https://stackoverflow.com/questions/2251578/how-do-i-get-the-selected-text-from-the-focused-window-using-native-win32-api)

TL;DR: Yes, there is a way to do this using plain win32 system APIs, but it's difficult to implement correctly.

WM_COPY and WM_GETTEXT may work, but not in all cases. They depend on the receiving window handling the request correctly - and in many cases it will not. Let me run through one possible way of doing this. It may not be as simple as you were hoping, but what is in the adventure filled world of win32 programming? Ready? Ok. Let's go.

First we need to get the HWND id of the target window. There are many ways of doing this. One such approach is the one you mentioned above: get the foreground window and then the window with focus, etc. However, there is one huge gotcha that many people forget. After you get the foreground window you must AttachThreadInput to get the window with focus. Otherwise GetFocus() will simply return NULL.

There is a much easier way. Simply (miss)use the GUITREADINFO functions. It's much safer, as it avoids all the hidden dangers associated with attaching your input thread with another program.

```
LPGUITHREADINFO lpgui = NULL;  
HWND target_window = NULL;  
  
if( GetGUIThreadInfo( NULL, lpgui ) )  
    target_window = lpgui->hwndFocus;  
else  
{  
    // You can get more information on why the function failed by calling  
    // the win32 function, GetLastError().  
}  
```
 

Sending the keystrokes to copy the text is a bit more involved...

We're going to use SendInput instead of keybd_event because it's faster, and, most importantly, cannot be messed up by concurrent user input, or other programs simulating keystrokes.

This does mean that the program will be required to run on Windows XP or later, though, so, sorry if your running 98!

```
// We're sending two keys CONTROL and 'V'. Since keydown and keyup are two  
// seperate messages, we multiply that number by two.  
int key_count = 4;  
  
INPUT* input = new INPUT[key_count];  
for( int i = 0; i < key_count; i++ )  
{  
    input[i].dwFlags = 0;  
    input[i].type = INPUT_KEYBOARD;  
}  
  
input[0].wVK = VK_CONTROL;  
input[0].wScan = MapVirtualKey( VK_CONTROL, MAPVK_VK_TO_VSC );  
input[1].wVK = 0x56 // Virtual key code for 'v'  
input[1].wScan = MapVirtualKey( 0x56, MAPVK_VK_TO_VSC );  
input[2].dwFlags = KEYEVENTF_KEYUP;  
input[2].wVK = input[0].wVK;  
input[2].wScan = input[0].wScan;  
input[3].dwFlags = KEYEVENTF_KEYUP;  
input[3].wVK = input[1].wVK;  
input[3].wScan = input[1].wScan;  
  
if( !SendInput( key_count, (LPINPUT)input, sizeof(INPUT) ) )  
{  
    // You can get more information on why this function failed by calling  
    // the win32 function, GetLastError().  
}  
 
```

There. That wasn't so bad, was it?

Now we just have to take a peek at what's in the clipboard. This isn't as simple as you would first think. The "clipboard" can actually hold multiple representations of the same thing. The application that is active when you copy to the clipboard has control over what exactly to place in the clipboard.

When you copy text from Microsoft Office, for example, it places RTF data into the clipboard, alongside a plain-text representation of the same text. That way you can paste it into wordpad and notepad. Wordpad would use the rich-text format, while notepad would use the plain-text format.

For this simple example, though, let's assume we're only interested in plaintext.

```
if( OpenClipboard(NULL) )  
{  
    // Optionally you may want to change CF_TEXT below to CF_UNICODE.  
    // Play around with it, and check out all the standard formats at:  
    // [http://msdn.microsoft.com/en-us/library/ms649013(VS.85).aspx](http://msdn.microsoft.com/en-us/library/ms649013\(VS.85\).aspx)    HGLOBAL hglb = GetClipboardData( CF_TEXT );  
    LPSTR lpstr = GlobalLock(hglb);  
  
    // Copy lpstr, then do whatever you want with the copy.  
  
    GlobalUnlock(hglb);  
    CloseClipboard();  
}  
else  
{  
    // You know the drill by now. Check GetLastError() to find out what  
    // went wrong. :)  
} 
```

And there you have it! Just make sure you copy lpstr to some variable you want to use, don't use lpstr directly, since we have to cede control of the contents of the clipboard before we close it.

Win32 programming can be quite daunting at first, but after a while... it's still daunting.

Cheers!