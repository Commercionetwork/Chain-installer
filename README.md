# Ansible playbook for deploying a Commercio Network node
The playbook configures a server to be a Commercio Network full node and installs a script that detects whether the node is losing blocks.
If the node loses blocks, a message will be forwarded to a slack channel of your choice.

## Prerequisites
The prerequisites to use this playbook are:
* Have a server on which to install the node
* Have python3 installed (version >= 3.5)
     * To check the version:
         ```bash
         python3 -V
         ```
* Have ansible installed (version >= core 2.9)
     * To check the version:
         ```bash
         ansible --version
         ```
## Usage
To use the playbook you need to proceed as follows:

1. Create and add the `hosts.ini` file to the root directory of this repository and configure it with your target hosts where you want to install your full node. See the [Example hosts file](.hosts.ini).

2. Based on the variables listed in the file `all`  located in `main/group_vars` path, you can change the behavior of the playbook.

     Default values are set for certain variables, if you want to assign  different values for these variables, you can Copy the variable list `all.template` to the file `all` and customize your variables.  
     
     Below is the table of all the variables and their meaning:

     | Variable name | Usage |
     | -- | -- |
     | `chain_id` | Blockchain chain-id (e.g: chain-id for mainnet: `commercio-3` , chain-id for testnet: `commercio-testnet11k`) |
     | `comm_user` | Name of the user that will be created and used by the node service. |
     | `moniker` | Name of the node |
     | `sync_type`| Determines how database synchronization occurs. Options:|
     |  |   • `pull`: Synchronization occurs by doing an rsync from the new node to the node from which it synchronizes.|
     |  |   • `push`: Synchronization occurs by doing an rsync from the node from which you synchronize to the new node.|
     |  |   • `dump`: Synchronization occurs by dumping the dump to disk and unpacking it.|
     |  |   • `statesync`: Synchronization occurs via statesync.|
     |  |   • `none`: Synchronization occurs normally.|
     | `sync_node` | Node IP to sync from in case of `sync_type` = `push` or `pull` |
     | `sync_node_home_folder` | Node home folder to sync from in case of `sync_type` = `push` or `pull` |    
     | `custom_bin_version` | Version of the Commercionetworkd binary to use. |
     | `custom_genesis` | Genesis file URL |
     | `custom_comm_repo` | Path and name of where the Commercio.network binary repository is downloaded |
     | `custom_home_folder` | Home where the `.commercionetwork` folder is installed |
     | `custom_go_path` | Path of installed go |
     | `custom_cosmovisor_version` | Version of Cosmovisor to install |
     | `custom_trust_rpc1` | First RPC to sync from in case of sync_type = statesync `sync_type` = `statesync`. |
     | `custom_trust_rpc2` | Second RPC to sync from in case of sync_type = statesync `sync_type` = `statesync`.  |
     | `external_drive` | Optional parameter to specify the path if the node database should be installed on an external disk |
     | `slack_hook` | Webhook where to send reports about a node losing blocks. **No script will be installed if a webhook isn't set** |
     | `ANSIBLE_HOST_KEY_CHECKING` | Set the variable to `False` to disbale ssh prompt for host key checks (skip the fingerprint) |


- If you want to use a specific cosmovisor version , you can run the following command to list all the cosmovisor versions:

    ```bash
    git -c 'versionsort.suffix=-' ls-remote --tags --sort='v:refname' https://github.com/cosmos/cosmos-sdk.git | grep "cosmovisor" | fgrep -v '{}'
    ```
2. Run the playbook from the root folder of this repo.:
    ```bash
    ansible-playbook -i hosts.ini main/commercio.yml
    ```

### Manual installation of the test height script
If you don't choose to install the test height script via the playbook you can install it manually, here's how:
1. Copy the script from `main/post_install/tasks/test_height.sh` on the server
2. Replace the `{{ slack_hook }}` variable in the file with your own slack webhook
3. Add the script to a crontab using the `crontab -e` command. We suggest running the script every 5 minutes

## Troubleshooting
There may be errors when launching the playbook. These are the most common:

1. During the `install_binary : Run 'make' on target` step there may be a possible failure in compiling the binary due to the `TestProcessTestSuite` test not passing.
The playbook will ignore the error and continue execution. The binary still compiles and will work fine.

2. During the `setup_configuration : Download genesis file` step, a possible failure in the genesis download may occur. This error is caused by Github not responding to the request correctly. In this case the playbook will stop. This error rarely occurs and it will be enough to restart the playbook to complete the configuration.
