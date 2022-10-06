# Steps to deploy a Maui application

## Author: Stanley Munson 
*Using Microsoft documentation, linked below*

## Date: 2022-Oct-05

---

## Create a self-signed certificate:

Use the following commands in PowerShell to create a new self-signed certificate and see if it was added to the cert store. Make sure to browse to the directory where the project lives. Copy the thumbprint from either command.

### Create

```
New-SelfSignedCertificate -Type Custom `
                          -Subject "CN=skm3.net" `
                          -KeyUsage DigitalSignature `
                          -FriendlyName "Temp self-signed cert for dev purposes" `
                          -CertStoreLocation "Cert:\CurrentUser\My" `
                          -TextExtension @("2.5.29.37={text}1.3.6.1.5.5.7.3.3", "2.5.29.19={text}")
```

### View

`Get-ChildItem "Cert:\CurrentUser\My" | Format-Table Subject, FriendlyName, Thumbprint`

---

## Modifying the csproj file for deployment:

Add the following XML to the Maui csproj file. Use the thumbprint copied from the last section in the `<PackageCertificateThumbprint>` when copying.

```
<PropertyGroup Condition="$([MSBuild]::GetTargetPlatformIdentifier('$(TargetFramework)')) == 'windows' and '$(Configuration)' == 'Release'">
    <AppxPackageSigningEnabled>true</AppxPackageSigningEnabled>
    <PackageCertificateThumbprint>F635AAC844C9F2AA5DD2419417A401F00338F529</PackageCertificateThumbprint>
</PropertyGroup>
<PropertyGroup Condition="$([MSBuild]::GetTargetPlatformIdentifier('$(TargetFramework)')) == 'windows' and '$(RuntimeIdentifierOverride)' != ''">
    <RuntimeIdentifier>$(RuntimeIdentifierOverride)</RuntimeIdentifier>
</PropertyGroup>
```

---

## Publish dotnet application from VS2022:

Use the following command from the VS command prompt to publish using dotnet. Be sure to replace the "-f" parameter with the net6.0-windows10{version} found in the csproj. Look for the `<TargetFrameworks>` tags to get the version used by the application:

`dotnet publish -f net6.0-windows10.0.19041.0 -c Release /p:RuntimeIdentifierOverride=win10-x64`

Successfully publishing the app will result in the MSIX file being placed in the bin folder of the project; specifically: bin\Release\net6.0-windows10.0.19041.0\win10-x64\AppPackages\\{appname}\\.

---

## Installing the application:

You have to trust the self-signed certificate used in the MSIX file before you can install the application. Follow these steps to trust the cert:

- Right-click on the .msix file and choose Properties.
- Select the Digital Signatures tab.
- Choose the certificate then press Details.
- Select View Certificate.
- Select Install Certificate...
- Choose Local Machine then select Next.
- In the Certificate Import Wizard window, select Place all certificates in the following store.
- Select Browse... and then choose the Trusted People store. Select OK to close the dialog.
- Select Next and then Finish. You should see a dialog that says: The import was successful.
- Select OK on any window opened as part of this process, to close them all.

--- 

## Closing Remarks:

With the self-signed certificate installed, running the MSIX file should result in a successful prompt to install the application. Select the install button if you would like to install the app

For more details and links to additional steps, visit the [Microsoft page regarding windows deployment](https://learn.microsoft.com/en-us/dotnet/maui/windows/deployment/overview)
