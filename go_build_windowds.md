    go build -ldflags "-H windowsgui -w" -o windows_agent.exe

    if you need build 32 execute file
    first set GOARCH=386
    and then build the programe 