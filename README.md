Hey all,

Many people know zabbix and its capabilities, but many people i spoke with don't know about the option the allow you to create your own items.

This option also known by the name: UserParameter.

Why do we need this anyway?
Sometimes, Zabbix can't monitor exactly what we want, or we would like to have specific monitoring for our services \ products. And using this feature we will be able to create item for anything that we can provide using scripts or commands.

In this article, I will show you what are all the steps needed in order to start using this feature. and will show you an example of how to use it in order to get alerts for expired certificates in IIS.

First things first, The configuration.
We have 2 ways to let zabbix know that it has UserParameter he should know about.

In the actual config file.
All you need to do basiaclly, is to insert the UserParameter in th e configuration file.

So, how does it look?

UserParameter=test.userparameter,echo hello
This line allow you to create a new item called test.userparameter, that will run the command "echo 1" each time it checks the item, and return the value to zabbix.

In a new file.
Another way, nicer, is to create a new file only for user parameters, and include the file in the "Include" section in the configuration file.

### Option: Include
#    You may include individual files in the configuration file.
#
# Mandatory: no
# Default:
# Include=

# Include=c:\zabbix\zabbix_agentd.userparams.conf
Now all you have to do is to paste the UserParameter there, instead of inside the configuration file, which you probably want to keep as generic as possible.

Let's create the item.
When we type a new UserParameter we should give it a unique key name.

In the former example the key name is "test.userparameter", so let's create a new item.



In this example, what matters is that the key name will be the same as the key name in the conf file, and that the data type will match the type to be returned.

Now, for an actual use case
I used this solution when I needed to create an item that will alert me when there's a certificate installed on the server which is going to be expired soon.

Step 1 - The UserParameter:
UserParameter=cert.daystoexpire,powershell.exe -Command "Invoke-Command { Dir Cert:\LocalMachine\My } | foreach { $_ | Select @{Label=\"Days\";Expression={($_.NotAfter - (Get-Date)).Days}} } | foreach { $_.Days } | sort-object -Descending | Select -last 1"
In this case, i decided to put the command in the conf file, but of course you can use a script and it will look something like this:

UserParameter=cert.daystoexpire,powershell ./CertScript.ps1
Step 2 - The Item:


As you can see, the key name is the same as the key name in the config file, and the data type match the data type the line returns.

Step 3 - The Alert:
Now I need to create the alert because items alone are good, but not great.

The item will collect each day the amount of days that are left until the earliest expiration time of a certificate in our server.

This is why we would like to receive an alert when this number is bellow 45.




Conclusion
So that's it, using those 3 little steps and some scripting skills you can monitor ANYTHING you want.

Don't forget that updating a conf file requires an agent restart!

Have fun.
