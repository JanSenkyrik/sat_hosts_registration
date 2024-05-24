## REGISTERING HOSTS AND SETTING UP HOST INTEGRATION
You must register hosts that have not been provisioned through Satellite to be able to manage them with Satellite. You can register hosts through Satellite Server or Capsule Server.

Note that the entitlement-based subscription model is deprecated and will be removed in a future release. Red Hat recommends that you use the access-based subscription model of Simple Content Access instead.

### SUPPORTED CLIENTS IN REGISTRATION
Satellite supports the following operating systems and architectures for registration.

#### Supported Host Operating Systems
The hosts can use the following operating systems:
- Red Hat Enterprise Linux 9, 8, 7
- Red Hat Enterprise Linux 6 with the ELS Add-On

#### Supported Host Architectures
The hosts can use the following architectures:
- i386
- x86_64
- s390x
- ppc_64

### REGISTRATION METHODS
You can use the following methods to register hosts to Satellite:

#### Global registration
You generate a curl command from Satellite and run this command from an unlimited number of hosts to register them using provisioning templates over the Satellite API. By using this method, you can also deploy Satellite SSH keys to hosts during registration to Satellite to enable hosts for remote execution jobs. By using this method, you can also configure hosts with Red Hat Insights during registration to Satellite.

#### (Deprecated) Katello CA Consumer
You download and install the consumer RPM from satellite.example.com/pub/katello-ca-consumer-latest.noarch.rpm on the host and then run subscription-manager.

#### (Deprecated) Bootstrap script
You download the bootstrap script from satellite.example.com/pub/bootstrap.py on the host and then run the script.

### REGISTERING HOSTS BY USING GLOBAL REGISTRATION
You can register a host to Satellite by generating a curl command on Satellite and running this command on hosts. This method uses two provisioning templates: Global Registration template and Linux host_init_config default template. That gives you complete control over the host registration process.

You can also customize the default templates if you need greater flexibility.

#### Global parameters for registration
You can configure the following global parameters by navigating to Configure > Global Parameters:

- The host_registration_insights parameter is used in the insights snippet. If the parameter is set to true, the registration installs and enables the Red Hat Insights client on the host. If the parameter is set to false, it prevents Satellite and the Red Hat Insights client from uploading Inventory reports to your Red Hat Hybrid Cloud Console. The default value is true. When overriding the parameter value, set the parameter type to boolean.

- The host_packages parameter is for installing packages on the host.

- The host_registration_remote_execution parameter is used in the remote_execution_ssh_keys snippet. If it is set to true, the registration enables remote execution on the host. The default value is true.

- The remote_execution_ssh_keys, remote_execution_ssh_user,
remote_execution_create_user, and remote_execution_effective_user_method parameters are used in the remote_execution_ssh_keys snippet. For more details, see the snippet.

You can navigate to snippets in the Satellite web UI through Hosts > Templates > Provisioning Templates.

#### Registering a host
You can register a host by using registration templates and set up various integration features and host tools during the registration process.

##### Prerequisites
- Your user account has a role assigned that has the create_hosts permission.
 
- You must have root privileges on the host that you want to register.

- Satellite Server, any Capsule Servers, and all hosts must be synchronized with the same NTP server, and have a time synchronization tool enabled and running.

- An activation key must be available for the host.

- Optional: If you want to register hosts to Red Hat Insights, you must synchronize the rhel-8-for-x86_64-baseos-rpms and rhel-8-for-x86_64-appstream-rpms repositories and make them available in the activation key that you use. This is required to install the insights-client package on hosts.

- If you want to use Capsule Servers instead of your Satellite Server, ensure that you have configured your Capsule Servers accordingly. If your Satellite Server or Capsule Server is behind an HTTP proxy, configure the Subscription Manager on your host to use the HTTP proxy for connection.
 
#### Procedure
1. In the Satellite web UI, navigate to Hosts > Register Host.

2. Optional: Select a different Organization.

3. Optional: Select a different Location.

4. Optional: From the Host Group list, select the host group to associate the hosts with. Fields that inherit value from Host group: Operating system, Activation Keys and Lifecycle environment.

5. Optional: From the Operating system list, select the operating system of hosts that you want to register.

6. Optional: From the Capsule list, select the Capsule to register hosts through. NOTE: A Capsule behind a load balancer takes precedence over a Capsule selected in the Satellite web UI as the host’s content source.

7. In the Activation Keys field, enter one or more activation keys to assign to hosts.

8. Optional: Select the Insecure option, if you want to make the first call insecure. During this first call, hosts download the CA file from Satellite. Hosts will use this CA file to connect to Satellite with all future calls making them secure. Red Hat recommends that you avoid insecure calls.  
If an attacker, located in the network between Satellite and a host, fetches the CA file from the first insecure call, the attacker will be able to access the content of the API calls to and from the registered host and the JSON Web Tokens (JWT). Therefore, if you have chosen to deploy SSH keys during registration, the attacker will be able to access the host using the SSH key.  
Instead, you can manually copy and install the CA file on each host before registering the host. To do this, find where Satellite stores the CA file by navigating to Administer > Settings > Authentication and locating the value of the SSL CA file setting.  
Copy the CA file to the /etc/pki/ca-trust/source/anchors/ directory on hosts and enter the following commands:  
~~~
# update-ca-trust enable
# update-ca-trust
~~~
Then register the hosts with a secure curl command, such as:
~~~
# curl -sS https://satellite.example.com/register ...
~~~
The following is an example of the curl command with the --insecure option:
~~~
# curl -sS --insecure https://satellite.example.com/register ...
~~~

9. Select the Advanced tab.

10. Optional: From the Setup REX list, select whether you want to deploy Satellite SSH keys to hosts or not. If set to Yes, public SSH keys will be installed on the registered host. The inherited value is based on the host_registration_remote_execution parameter. It can be inherited, for example from a host group, an operating system, or an organization. When overridden, the selected value will be stored on host parameter level.

11. Optional: From the Setup Insights list, select whether you want to install insights-client and register the hosts to Insights. The Insights tool is available for Red Hat Enterprise Linux only. It has no effect on other operating systems.
You must enable the following repositories on a registered machine:
- Red Hat Enterprise Linux 6: rhel-6-server-rpms
- Red Hat Enterprise Linux 7: rhel-7-server-rpms
- Red Hat Enterprise Linux 8: rhel-8-for-x86_64-appstream-rpms  
The insights-client package is installed by default on Red Hat Enterprise Linux 8 except in environments whereby Red Hat Enterprise Linux 8 was deployed with "Minimal Install"
option.

12. Optional: In the Install packages field, list the packages (separated with spaces) that you want to install on the host upon registration. This can be set by the host_packages parameter.

13. Optional: Select the Update packages option to update all packages on the host upon registration. This can be set by the host_update_packages parameter.

14. Optional: In the Repository field, enter a repository to be added before the registration is performed. For example, it can be useful to make the subscription-manager package available for the purpose of the registration. For Red Hat family distributions, enter the URL of the repository, for example http://rpm.example.com/.

15. Optional: In the Repository GPG key URL field, specify the public key to verify the signatures of GPG-signed packages. It needs to be specified in the ASCII form with the GPG public key
header.

16. Optional: In the Token lifetime (hours) field, change the validity duration of the JSON Web Token (JWT) that Satellite uses for authentication. The duration of this token defines how long the generated curl command works. You can set the duration to 0 – 999 999 hours or unlimited.  
Note that Satellite applies the permissions of the user who generates the curl command to authorization of hosts. If the user loses or gains additional permissions, the permissions of the JWT change too. Therefore, do not delete, block, or change permissions of the user during the token duration.  
The scope of the JWTs is limited to the registration endpoints only and cannot be used anywhere else.

17. Optional: In the Remote Execution Interface field, enter the identifier of a network interface that hosts must use for the SSH connection. If you keep this field blank, Satellite uses the default network interface.

18. Optional: From the REX pull mode list, select whether you want to deploy Satellite remote execution pull client.  
If set to Yes, the remote execution pull client is installed on the registered host. The inherited value is based on the host_registration_remote_execution_pull parameter. It can be inherited, for example from a host group, an operating system, or an organization. When overridden, the selected value is stored on the host parameter level.  
The registered host must have access to the Red Hat Satellite Client 6 repository.

19. Optional: Select the Ignore errors option if you want to ignore subscription manager errors.

20. Optional: Select the Force option if you want to remove any katello-ca-consumer rpms before
registration and run subscription-manager with the --force argument.

21. Click Generate.

22. Copy the generated curl command.

23. On the host that you want to register, run the curl command as root.

#### Customizing the registration templates
You can customize the registration process by editing the provisioning templates. Note that all default templates in Satellite are locked. If you want to customize the registration templates, you must clone the default templates and edit the clones.  

The registration process uses the following provisioning templates:
- The Global Registration template contains steps for registering hosts to Satellite. This template renders when hosts access the /register Satellite API endpoint.

- The Linux host_init_config default template contains steps for initial configuration of hosts after they are registered.

##### Procedure
1. Navigate to Hosts > Templates > Provisioning Templates.

2. Search for the template you want to edit.

3. In the row of the required template, click Clone.

4. Edit the template as needed. For more information, see Appendix A, Template writing reference.

5. Click Submit.

6. Navigate to Administer > Settings > Provisioning.

7. Change the following settings as needed:
- Point the Default Global registration template setting to your custom global registration template,
- Point the Default 'Host initial configuration' template setting to your custom initial configuration template.