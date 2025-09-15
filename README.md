# Velociraptor MCP
Velociraptor MCP is a POC Model Context Protocol bridge for exposing LLMs to MCP clients.

Initial version has several Windows orientated triage tools deployed. Best use is querying usecase to target machine name.

e.g 

`can you give me all network connections on MACHINENAME and look for suspicious processes?`

`can you tell me which artifacts target the USN journal`

## Note: When IP address of server has changed, have to change the following:
1. Server config file (velociraptor.config.yaml)
2. Generate a new client config file
``` ./velociraptor-v0.75.1-rc1-linux-amd64 --config velociraptor.config.yaml config client > client.config.yaml ```
3. Repackage the new instance of velociraptor and perform installation on client
``` ./velociraptor-v0.75.1-rc1-linux-amd64 config repack --exe velociraptor-v0.74.5-windows-amd64.exe client.config.yaml repackaged_velociraptor.exe ```
4. Modify the server IP address in api_client.yaml (in same folder as velociraptor config files)

## Launching in WSL - Ubuntu22.04
### 1. Launch WSL with Ubuntu-22.04 in cmd prompt

```
wsl -d Ubuntu-22.04
cd Desktop/Velociraptor/veloBuild
 ./velociraptor-v0.75.1-rc1-linux-amd64 --config velociraptor.config.yaml frontend -v
```

## Building Velociraptor Binary from Source with potential plugins [WSL Ubuntu-22.04]
1. Clone repository:
   ```bash
   git clone https://github.com/Velocidex/velociraptor.git
   ```
2. Place `<pluginName>.go` in `velociraptor/vql/common`
3. Navigate to project:
   ```bash
   cd velociraptor
   cd gui/velociraptor
   ```
4. Install dependencies:
   ```bash
   npm install
   ```
5. Ensure GUI is build:
   ```
   make build
   ```
6. Return to root directory:
   ```bash
   cd ../..
   ```
7. Build Linux binary:
   ```bash
   make linux
   ```
8. Find compiled binary in `velociraptor/output` (e.g., `velociraptor-v0.75.1-rc1-linux-amd64`)
9. Pipe output to ../veloBuild directory
    ```
    ./output.sh
    ```
---

## Server and Client Setup (WSL Ubuntu 22.04)
1. Install Ubuntu-22.04 on WSL
2. Generate config:
   ```bash
   ./velociraptor-v0.75.1-rc1-linux-amd64 config generate > velociraptor.config.yaml
   ```
3. Update server IP in config(note that IP may change occassionally unless static):
   ```bash
   nano velociraptor.config.yaml  # Replace all localhost/127.0.0.1 with server IP
   ```
4. Add administrator user:
   ```bash
   ./velociraptor-v0.75.1-rc1-linux-amd64 --config velociraptor.config.yaml user add admin --role administrator
   ```
   Credentials: `admin`/`123456`
5. Create client config:
   ```bash
   ./velociraptor-v0.75.1-rc1-linux-amd64 --config velociraptor.config.yaml config client > client.config.yaml
   ```
6. Download Windows executable:
   ```bash
   wget https://github.com/Velocidex/velociraptor/releases/download/v0.74/velociraptor-v0.74.5-windows-amd64.exe
   ```
7. Repackage for Windows:
   ```bash
   ./velociraptor-v0.75.1-rc1-linux-amd64 config repack --exe velociraptor-v0.74.5-windows-amd64.exe client.config.yaml repackaged_velociraptor.exe
   ```
8. Copy `repackaged_velociraptor.exe` to Windows client machine
9. Install as Windows service (on client machine):
   ```powershell
   .\repackaged_velociraptor.exe service install
   ```
   Verify service in Windows Services Manager
10. Start server:
    ```bash
    ./velociraptor-v0.75.1-rc1-linux-amd64 --config velociraptor.config.yaml frontend -v
    ```
    Server URL: `<IP addr of server>:8889`


## Installation
### 1. Setup an API account
https://docs.velociraptor.app/docs/server_automation/server_api/

Generate an api config file:

`./velociraptor-v0.75.1-rc1-linux-amd64 --config velociraptor.config.yaml config api_client --name api --role administrator,api api_client.yaml`

### 2. Clone mcp-velociraptor repo and test API 

- copy api_client.yaml to preferred config location and ensure configuration correct (pointing to appropriate IP address).
- modify test_api.py to appropriate location.
- Run test_api.py to confirm working
- Modify mcp_velociraptor_bridge.py to correct API config

### 3. Connect to Claude desktop or MCP client of choice

The easiest configuration is to run your venv python directly calling mcp_velociraptor_bridge.
```{
  "mcpServers": {
    "velociraptor": {
      "command": "/path/to/venv/bin/python",
      "args": [
        "/path/to/mcp_velociraptor_bridge.py"
      ]
    }
  }
}
```

![image](https://github.com/user-attachments/assets/3e810f03-ca74-4757-b5dc-89d4e8f8aef6)

#### 3.1 Fixes to enable MCP-Velociraptor to work as intended
There were some input validation errors. FastMCP builds a Pydantic schema from the type hints in the code in mcp_velociraptor_bridge.py, so when a tool returns a list or dict but the schema says string, we will get the “Input should be a valid string … input_type=list/dict” error.
<img width="897" height="436" alt="image" src="https://github.com/user-attachments/assets/4e907958-6b15-41c2-a6aa-838cf60a51cd" />


#### Current fix: Change all relevant return types to List[Dict[str, Any]] in mcp_velociraptor_bridge.py and velociraptor_api.py (works)


### 3. Caveats

Due to the nature of DFIR, results depend on amount of data returned, model use and context window.

I have included a function to find artifacts and dynamically create collections but had mixed results.
I have been pleasantly surprised with some results and disappointed when running other collections that cause lots of rows.

Please let me know how you go and feel free to add PR!


`can you give me all network connections on MACHINENAME and look for suspicious processes?`
<img alt="image" src="https://github.com/user-attachments/assets/cc19ccde-f8fa-40d5-8b4d-82215777dc6b" />
<img alt="image" src="https://github.com/user-attachments/assets/734ce6d0-6c66-49cf-a0f7-8236f7435be3" />
<img alt="image" src="https://github.com/user-attachments/assets/b6593321-1089-4f00-8011-5ef08cf80d88" />

`can you tell me which artifacts target the USN journal`
<img alt="image" src="https://github.com/user-attachments/assets/b9f93b1c-4a08-437d-b25a-ff82bdd2ab8c" />

