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
