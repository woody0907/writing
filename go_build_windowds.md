    go build -ldflags "-H windowsgui -w" -o windows_agent.exe

    if you need build 32 execute file
    first set GOARCH=386
    and then build the programe

## regeist windwos service

1. using sc create command
   this method will lead to 1053 erro when you start the service,
   and i can't solve it until now
2. using nssm tools  
   this is a fantastic tools to resgist your own service and run it 