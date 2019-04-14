### WinKey + Alt + UpArrow to center window:
https://superuser.com/questions/156351/keyboard-shortcut-in-windows-7-to-center-window

- Download and install AutoHotKey
- Right lick on desktop > New > AutoHotKey Script. Calle it "CenterWindow.ahk"
- Edit (with notepad and save
```
#NoEnv  ; Recommended for performance and compatibility with future AutoHotkey releases.
; #Warn  ; Enable warnings to assist with detecting common errors.
SendMode Input  ; Recommended for new scripts due to its superior speed and reliability.
SetWorkingDir %A_ScriptDir%  ; Ensures a consistent starting directory.

#!Up::CenterActiveWindow() ; if win+alt+↑ is pressed

CenterActiveWindow()
{
    SysGet, WA_, MonitorWorkArea ; get the actual work area. That is, screen size w/o the taskbar.
    A_ScreenWidthWA := WA_Right - WA_Left
    A_ScreenHeightWA := WA_Bottom - WA_Top
    windowWidth := A_ScreenWidthWA * 0.5 ; desired width
    windowHeight := A_ScreenHeightWA ; desired height
    WinGetTitle, windowName, A
    WinMove, %windowName%, , A_ScreenWidthWA / 2 - (windowWidth / 2), 0, windowWidth, windowHeight
}
```
- Double click to run it
- Try it with WinKey + Alt + UpArrow
