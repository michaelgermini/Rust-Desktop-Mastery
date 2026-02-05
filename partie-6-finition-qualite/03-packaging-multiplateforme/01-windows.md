# 27.1 Windows

## MSI avec cargo-wix

```toml
# Cargo.toml
[package]
name = "mon-app"
version = "1.0.0"

[package.metadata.wix]
# Configuration WiX
```

```bash
# Installer cargo-wix
cargo install cargo-wix

# Initialiser la configuration WiX
cargo wix init

# Créer l'installer MSI
cargo wix

# Le résultat est dans target/wix/mon-app-1.0.0-x86_64.msi
```

```xml
<!-- wix/main.wxs - Configuration WiX -->
<?xml version="1.0" encoding="UTF-8"?>
<Wix xmlns="http://schemas.microsoft.com/wix/2006/wi">
    <Product Name="Mon Application" 
             Manufacturer="Votre Société"
             Version="1.0.0"
             UpgradeCode="GUID-HERE"
             Language="1033">
        
        <Package InstallerVersion="500" 
                 Compressed="yes" 
                 InstallScope="perUser" />
        
        <MajorUpgrade DowngradeErrorMessage="Une version plus récente est installée." />
        
        <MediaTemplate />
        
        <Feature Id="ProductFeature" Title="Mon Application" Level="1">
            <ComponentGroupRef Id="ProductComponents" />
        </Feature>
    </Product>
    
    <Fragment>
        <Directory Id="TARGETDIR" Name="SourceDir">
            <Directory Id="ProgramFilesFolder">
                <Directory Id="INSTALLFOLDER" Name="Mon Application" />
            </Directory>
        </Directory>
    </Fragment>
    
    <Fragment>
        <ComponentGroup Id="ProductComponents" Directory="INSTALLFOLDER">
            <Component Id="ApplicationFiles">
                <File Id="MonApp.exe" Source="target/release/mon-app.exe" />
            </Component>
        </ComponentGroup>
    </Fragment>
</Wix>
```

## NSIS

```nsis
; installer.nsi
!define APP_NAME "Mon Application"
!define APP_VERSION "1.0.0"
!define APP_PUBLISHER "Votre Société"

Name "${APP_NAME}"
OutFile "MonApp-Setup.exe"
InstallDir "$PROGRAMFILES\${APP_NAME}"
RequestExecutionLevel admin

Section "Install"
    SetOutPath "$INSTDIR"
    File "target\release\mon-app.exe"
    
    ; Créer un raccourci
    CreateShortcut "$DESKTOP\${APP_NAME}.lnk" "$INSTDIR\mon-app.exe"
    CreateShortcut "$SMPROGRAMS\${APP_NAME}.lnk" "$INSTDIR\mon-app.exe"
    
    ; Ajouter au PATH (optionnel)
    ; EnvSet::EnvSet "PATH" "$INSTDIR"
SectionEnd
```

## Signature de code

```bash
# Signer l'exécutable avec un certificat de code signing
signtool sign /f certificate.pfx /p password /t http://timestamp.digicert.com mon-app.exe

# Vérifier la signature
signtool verify /pa mon-app.exe
```

## Résumé

- **MSI** : Format standard Windows avec cargo-wix
- **NSIS** : Alternative plus flexible
- **Signature** : Certificat de code signing pour éviter les avertissements
- **InstallScope** : perUser ou perMachine
