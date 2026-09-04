# Fudge

Fudge is a package manager for Windows. It can find packages in a remote catalog, store them locally, and run scripts by name.

## Quick start

Run commands from the directory that contains `fudge.exe`:

```powershell
.\fudge search example
.\fudge install example-app
.\fudge run example-app
```

`install` stores a package locally without running it. `run` runs a stored package. If the package is not installed, `run` downloads it first and then runs it.

## Create a package

Create a script and publish it with:

```powershell
.\fudge create .\sample.ps1 sample-app
```

This command:

1. Saves the package to local storage.
2. Sends the package name, type, and file contents to FudgeServer.
3. Updates the remote package catalog.
4. Prints a private update key for the package.

Package names must be unique. Creating another package with an existing name is rejected.

After publishing, the package can be installed on another computer:

```powershell
.\fudge install sample-app
.\fudge run sample-app
```

## Commands

### `install`

Download a package to local storage without running it:

```powershell
.\fudge install <package-name>
```

Packages are stored in:

```text
%LOCALAPPDATA%\Fudge\packages
```

### `run`

Run a package by name:

```powershell
.\fudge run <package-name>
```

If the package is not installed, Fudge downloads it automatically before running it. PowerShell output, CMD output, and script errors are forwarded to the Fudge console.

### `search`

Find a package by name or description:

```powershell
.\fudge search <query>
```

Search includes the remote catalog and packages already stored locally.

### `report`

Send a report about a package. You can provide up to two reasons separated by `;;`:

```powershell
.\fudge report sample-app "Does not start;;Incorrect output"
```

### `help`

Show help:

```powershell
.\fudge --help
```

### `update`

Update a package using its private update key and a new script file:

```powershell
.\fudge update <update-key> <script-path>
```

The key is checked by the server. The client keeps a copy locally for the owner, while the public package catalog never exposes it.

### `delete`

Delete a package from the remote catalog:

```powershell
.\fudge delete <update-key> <package-name>
```

Fudge asks for `Are you sure? Y/N:` before sending the delete request. The server validates the key before removing the package.
After a successful deletion, the local package cache is removed as well.

## Supported files

Fudge supports these script types:

- `.ps1`
- `.ps2`
- `.bat`
- `.cmd`
- `.psd1`
- `.psm1`

PowerShell scripts run with `-NoProfile` and `-ExecutionPolicy Bypass`. Batch files run through `cmd.exe`.

## Storage format

Local packages are stored in `.fge` containers with random names, for example:

```text
%LOCALAPPDATA%\Fudge\packages\a13f7c2e9b4d.fge
```

Each container contains one line:

```text
.ps1 <obfuscated package contents>
```

When running a package, Fudge temporarily extracts its contents, runs them through the required shell, and deletes the temporary file afterward.

## Remote catalog

By default, Fudge uses the URL from the GitHub `nowurl` file. Override it with an environment variable:

```powershell
$env:FUDGE_REMOTE_URL = "https://example.com/manifest"
```

You can also save the URL in:

```text
%USERPROFILE%\.fudge\nowurl
```

The catalog contains a `Packages` array. Each package uses these fields:

```json
{
  "Name": "sample-app",
  "FileType": ".ps1",
  "Command": "Write-Host 'Hello from Fudge'",
  "Description": "Example package"
}
```

The `Command` field contains the package file contents. The `DownloadUrl` field is not used.
Update keys are stored in the server's private catalog data and are never returned by public catalog requests.

## Common errors

### `Package was not found`

Check the package name with:

```powershell
.\fudge search <part-of-name>
```

### `can't fetch file`

Check the remote catalog and the value of `FUDGE_REMOTE_URL`.

### The package runs but produces no output

Make sure the script contains an output command, such as `Write-Host` for PowerShell or `echo` for CMD.
