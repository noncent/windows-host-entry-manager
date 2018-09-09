# Windows-Host-Entry-Manager
Window Host Manager is windows batch file to add entries in host file


```batch
@ECHO OFF
:: BatchGotAdmin
:-------------------------------------
REM  -- Check for permissions
>nul 2>&1 "%SYSTEMROOT%\system32\cacls.exe" "%SYSTEMROOT%\system32\config\system"

REM -- If error flag set, we do not have admin.
if '%errorlevel%' NEQ '0' (
    echo Requesting administrative privileges...
    goto UACPrompt
) else ( goto gotAdmin )

:UACPrompt
    echo Set UAC = CreateObject^("Shell.Application"^) > "%temp%\getadmin.vbs"
    set params = %*:"="
    echo UAC.ShellExecute "cmd.exe", "/c %~s0 %params%", "", "runas", 1 >> "%temp%\getadmin.vbs"

    "%temp%\getadmin.vbs"
    del "%temp%\getadmin.vbs"
    exit /B

:gotAdmin
    pushd "%CD%"
    CD /D "%~dp0"
:--------------------------------------

:LOOP
ECHO ++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++
ECHO +                                                            +
ECHO + Windows HOST File Editor                                   +                                                           +
ECHO + EXIT: PRESS CTRL+C ANY TIME TO EXIT                        +
ECHO +                                                            +
ECHO ++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++
ECHO.
SET Choice=
SET /P Choice="Do you want to modify HOSTS file ? (Y/N)"
ECHO.

IF NOT '%Choice%'=='' SET Choice=%Choice:~0,1%

ECHO.
IF /I '%Choice%'=='Y' GOTO ACCEPTED
IF /I '%Choice%'=='N' GOTO REJECTED
ECHO Please type Y (for Yes) or N (for No) to proceed!
ECHO.
GOTO Loop


:REJECTED
ECHO Your HOSTS file was left unchanged>>%systemroot%\Temp\hostFileUpdate.log
ECHO Finished.
GOTO END


:ACCEPTED
setlocal enabledelayedexpansion
::Take User Input
set /p IP="A) - Enter IP ADDRESS [Right Click To Paste From Clipboard]: "
set /p VHOST="B) - Enter HOST NAME [Right Click To Paste From Clipboard]: "
::Create your list of host domains
set LIST=(%VHOST%)
::Set the ip of the domains you set in the list above
set %VHOST%=%IP%
:: deletes the parentheses from LIST
set _list=%LIST:~1,-1%
::ECHO %WINDIR%\System32\drivers\etc\hosts > tmp.txt

for  %%G in (%_list%) do (
    set  _name=%%G
    set  _value=!%%G!
    SET NEWLINE=^& echo.
	ECHO.
    ECHO Carrying out requested modifications to your HOSTS file
    ::strip out this specific line and store in tmp file
    type %WINDIR%\System32\drivers\etc\hosts | findstr /v !_name! > tmp.txt
    ::re-add the line to it
	ECHO.>>tmp.txt
    ECHO %NEWLINE%# ++++++++++++++++++ BEGINNING ++++++++++++++++++ >> tmp.txt
	ECHO %NEWLINE%^!_value! !_name!>>tmp.txt
    ECHO %NEWLINE%# ++++++++++++++++++ -ENDING- ++++++++++++++++++ >> tmp.txt
	ECHO.>>tmp.txt
    ::overwrite host file
    copy /b/v/y tmp.txt %WINDIR%\System32\drivers\etc\hosts
    del tmp.txt
)

ECHO.

ipconfig /flushdns

ECHO.
ECHO Finished, you may close this window now.
ECHO You should now open Chrome and go to "chrome://net-internals/#dns" (without quotes)
ECHO then click the "clear host cache" button
GOTO END

:END
ECHO.
ping -n %IP% > nul

ECHO.

:Browser
SET Choice=
SET /P Choice="Do you want to open your host website in Chrome browser ? (Y/N)"
ECHO.

ECHO.

IF NOT '%Choice%'=='' SET Choice=%Choice:~0,1%

ECHO.
IF /I '%Choice%'=='Y' GOTO GOCHROME
IF /I '%Choice%'=='N' GOTO GOHOME
ECHO Please type Y (for Yes) or N (for No) to proceed!
ECHO.
GOTO Browser


:GOHOME
EXIT
:END

:GOCHROME
start chrome %VHOST%
:END
```
