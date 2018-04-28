Run Notepad
Run C:\My Documents\Address List.doc
Run C:\My Documents\My Shortcut.lnk
Run www.yahoo.com
Run mailto:someone@somedomain.com

msgbox, 这是我的第一个 AutoHotkey 脚本 \`n 我既关注效率，也尊重版权

#n::Run Notepad          Win+n

^!c::Run calc.exe         ctrl + alt + c

 %A_ProgramFiles%         内置变量

^!s::
Send Sincerely,{Enter}John Smith   发送字符串
return

Send ^c!{tab}^v           ctrl+c alt+tab ctrl+v 一键合成


::btw::by the way         打出btw会代替成by the way

click 1657, 20            左击window_spy 中窗口坐标


myvar = 124               
if myvar>=1
{
   MsgBox 使用文字中使用变量:%myvar%  
}

:=                         赋值符号

MsgBox %clipboard%         当前剪切板内容

;循环RunCount次
Loop %RunCount%
{
    Run C:\Check Server Status.exe
    Sleep 60000  ; 暂停 60 秒.
}


$F1::  ; 把 F1 键设置为热键 ($ 符号会有助于下面 GetKeyState 的 "P" 模式).
Loop  ; 由于没有指定数字, 所以这是个无限循环, 直到遇到内部的 "break" 或 "return".
{
    if not GetKeyState("F1", "P")  ; 如果此状态为 true, 那么用户实际已经释放了 F1 键.
        break  ; 中断循环.
    Click  ; 在当前指针位置点击鼠标左键.
}
return

$F1::
while GetKeyState("F1", "P")  ; 当 F1 键实际被按住时.
{
    Click
}
return


;按住Numpad0在按其他健 但Numpad0会单独无法工作 必须重新定义它的原有功能
Numpad0 & Numpad1::MsgBox You pressed Numpad1 while holding down Numpad0.
Numpad0::send() {Numpad0}  ;让 Numpad0 释放 时产生0


;区分场景
#IfWinActive, ahk_class Notepad
^a::MsgBox 打开记事本情况 其他情况默认
#c::MsgBox 打开记事本情况

#IfWinActive
#c::MsgBox 其他情况

;无论numllock是否开启 热键效果相同
NumpadEnd::


