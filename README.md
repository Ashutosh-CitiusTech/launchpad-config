# Deploying Docker-EE on Azure 

### Docker EE Deployment on Azure Using ARM Template and mirantis launchpad

1. You need to generate SSH-Key pairs using following command (Using PowerShell or Bash)

    ```
    $ mkdir .ssh
    $ ssh-keygen -f .ssh/id_rsa
    ```

2.  Open the Public Key file `.ssh/id_rsa.pub` on terminal

    ```
    $ type .ssh/id_rsa
    ```

3.  Copy the output generated by last command (Select using mouse and RIGHT click to copy)

4.  Paste the copied public-key inside [azuredeploy.paramaters.json](./ARMTemplate/azuredeploy.paramaters.json) inside folder `ARMTemplate` at line #9
    
    ```yml
    "value": " " //Paste your public key in those quotes.
    ```

5.  Use following Powershell commands to create the resource-group and deploy the template:

    > Feel free to change resource-group name and location.

    ```
    $ cd ARMTemplate
    $ New-AzResourceGroup -Name group-t -Location southeastasia
    $ New-AzResourceGroupDeployment -ResourceGroupName group-t -TemplateParameterFile .\azuredeploy.paramaters.json -TemplateFile .\azuredeploy.json
    ```

6.  Just wait for deployment to complete. Once completed, check number of Virtual machines and their status in resource-group `group-t` (or one you used instead of)

    ```
    $ Get-AzVm -ResourceGroupName group-t 
    ## Output should be:
    ResourceGroupName       Name      Location           VmSize OsType                         NIC ProvisioningSta
    -------------------------------------------------------------------
    group-t           MasterNode southeastasia Standard_D2ds_v4  Linux MasterNode-NetworkInterface       Succeeded 
    group-t                Node1 southeastasia Standard_D2ds_v4  Linux      Node1-NetworkInterface       Succeeded 
    group-t                Node2 southeastasia Standard_D2ds_v4  Linux      Node2-NetworkInterface       Succeeded 

    ```

7.  Now, it's time to download the `LaunchPad` cli from [Win EXE](https://github.com/Mirantis/launchpad/releases/download/0.14.0/launchpad-win-x64.exe) or [Linux EXE](https://github.com/Mirantis/launchpad/releases/download/0.14.0/launchpad-linux-x64) link.

8.  Please SSH into MasterNode using Private Key (Use PuTTY or Any other SSH tools)

    ```
    # Test the connectity with Nodes
    $ ping node1
    CTRL+C
    $ ping node2
    CTRL+C
    ```

9.  Open the content of "id_rsa" from local machine. and copy all the contents.
    Use SSH Terminal to paste the contents inside the MasterNode.

    ```
    $ cd ~/.ssh
    $ nano id_rsa
    # PASTE CONTENTS HERE
    # CTRL+O,ENTER, CTRL+X
    $ sudo chown $USER:$USER id_rsa
    $ sudo chmod 600 id_rsa
    $ cd ~
    $ ssh node1
    ## You should be INSIDE the node1
    $ exit
    $ ssh node2
    ## You should be INSIDE the node2
    $ exit
    ```



9.  Use following commands now on master node

    ```
    $ mkdir docker-ee
    $ cd docker-ee
    # Download the launchpad executable
    $ wget https://github.com/Mirantis/launchpad/releases/download/0.14.0/launchpad-linux-x64
    $ chmod +x launchpad-linux-x64
    ## COPY the SSH-PRIVATE-KEY
    $ mkdir .ssh
    $ cp ~/.ssh/id_rsa .ssh/
    $ ls -a .ssh 
    ```

10. Setup the cluster.yml 

    ```
    $ ./launchpad-linux-x64 register
    Enter name: Mahendra Shinde
    Enter EMail: MahendraUnlimited@gmail.com
    Company Name: Synergetics
    Registration Successful.
    ```

11. Create a file `cluster.yml` with following contents:

    ```yml
    apiVersion: launchpad.mirantis.com/v1beta3
    kind: DockerEnterprise
    metadata:
        name: ucp-kube
    spec:
        ucp:
            installFlags:
            - --admin-username=admin
            - --admin-password=passw0rd!
            - --default-node-orchestrator=kubernetes
        hosts:
        ## Public IP address of Manager Node
        ## you can use azure-vm, aws ec2 instance or local virtual-machine
        - address: MasterNode
            role: manager
            ssh:
                user: mahendra
                keyPath: .ssh/id_rsa
            ## Public IP address of worker Node
        - address: node1
            role: worker
            ssh:
                user: mahendra
                keyPath: .ssh/id_rsa
        - address: node2
            role: dtr
            ssh:
                user: mahendra
                keyPath: .ssh/id_rsa
    ```

12. Apply the cluster configuration.

    ```
    $ ./launchpad-linux-x64 apply
    ```