
https://github.com/kbandla/ImmunityDebugger
https://github.com/corelan/mona

```
C:\Program Files (x86)\Immunity Inc\Immunity Debugger\PyCommands
```

```cmd
BCDEDIT /SET {CURRENT} NX ALWAYSOFF
```

https://gist.github.com/s4vitar/**b88fefd5d9fbbdcc5f30729f7e06826e

Gatekeeper

## Comandos mona

Configurar espacio de trabajo

```
!mona config -set workingfolder C:\Users\fmol\Documents\%p
```

```
!mona bytearray 
```

```bash
!mona compare -f C:\Users\fmol\Documents\gatekeeper\bytearray.bin -a 009B19E4
```

```
!mona modules
```

```
!mona find -s "\xff\xe4" -m gatekeeper.exe
```



