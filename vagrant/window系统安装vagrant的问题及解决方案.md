# window系统安装vagrant的问题及解决方案

**window下安装遇到的问题**：

 - **中文编码问题**

    `Bringing machine 'default' up with 'virtualbox' provider...
==> default: Importing base box 'CentOS 7.0'...
C:/HashiCorp/Vagrant/embedded/gems/gems/childprocess-0.6.3/lib/childprocess/windows/process_builder.rb:43:in `join': **incompatible character encodings: GBK and UTF-8** (Encoding::CompatibilityError)
        from C:/HashiCorp/Vagrant/embedded/gems/gems/childprocess-0.6.3/lib/childprocess/windows/process_builder.rb:43:in `create_command_pointer'
        from C:/HashiCorp/Vagrant/embedded/gems/gems/childprocess-0.6.3/lib/childprocess/windows/process_builder.rb:27:in `start'
        from C:/HashiCorp/Vagrant/embedded/gems/gems/childprocess-0.6.3/lib/childprocess/windows/process.rb:70:in `launch_process'`
    
    **解决方案**：此为中文编码问题，即vagrant运行环境为C://users//中文名导致。在window命令行执行如下命令`set VAGRANT_HOME=C:\HashiCorp\Vagrant`，将vagrant运行环境打到英文目录下即可。
    
 - **virtualBox启动失败**

    ```
    `Bringing machine 'default' up with 'virtualbox' provider...
    ==> default: Importing base box 'CentOS 7.0'...
    There was an error while executing `VBoxManage`, a CLI used by Vagrant
    for controlling VirtualBox. The command and stderr is shown below.
    
    Command: ["import", "\\\\?\\C:\\HashiCorp\\Vagrant\\boxes\\CentOS 7.0\\0\\virtualbox\\box.ovf", "--vsys", "0", "--vmname", "centos-7.0-x86_64_1519369787941_82455", "--vsys", "0", "--unit", "9", "--disk", "C:\\Users\\\u53F6\u94D6\u9633\\VirtualBox VMs\\centos-7.0-x86_64_1519369787941_82455\\box-disk1.vmdk"]
    
    Stderr: 0%...10%...20%...30%...40%...50%...60%...70%...80%...90%...100%
    Interpreting \\?\C:\HashiCorp\Vagrant\boxes\CentOS 7.0\0\virtualbox\box.ovf...
    OK.
    0%...
    Progress state: VBOX_E_IPRT_ERROR
    VBoxManage.exe: error: Appliance import failed
    ```

    **解决方案**：打开Virtual Box，在VirtualBox首选项上更改默认的VM文件夹，改为vagrant运行环境即可。


​        

