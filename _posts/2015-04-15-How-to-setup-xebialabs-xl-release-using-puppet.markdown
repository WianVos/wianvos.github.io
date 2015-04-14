---
layout: post
title:  "How to setup Xebialabs Xl-Release using puppet"
date:   2015-04-15 14:14:00
categories: Puppet Xl-Release tech
---
I finally got arount to finishing my puppet module for xebialabs Xl-Release. 
And now it's time to share and do a little blogumentation :-)

Now i'm not really going to go into the inner workings of Xl-Release because there is another time and place for that. But i would like to take the time and explain how to setup Xl-Release using my puppet module. 

Firts-off where gonna install the module. Now i've published the module on the [puppetlabs forge](https://forge.puppetlabs.com/xebialabs/xlrelease) under the Xebialabs acount but ofcourse it is also available on [Github](https://github.com/xebialabs/puppet-xlrelease). Now if u are going to use the module i would recommend you always use the one on github because that is going to be the latest and greatest ... or not (suite yourself).

Now installing the module from the forge is fairly simple, just do: 

```
puppet module install xebialabs-xlrelease
```

and there you go: module installed.

Now we need to wire it up and put it in a manifest for it to be usefull and install an instance of Xl-Release on a machine. 
For this example i'm gonna pretend i have a machine called Xl-Release.sandbox.xebialabs (wich i do actually have ofcourse :-)) and a puppet master. 

On your puppet master add the following to your site.pp (or whatever manifest file you think it should go into)


    node 'xl-release.sandbox.xebialabs' {
    
        class{xlrelease:
            install_java => true,
            install_type: 'download'
            xlr_download_user: '<your download user>'
            xlr_download_password: '<your download password'
            xlr_licsource: 'https://dist.xebialabs.com/customer/licenses/download/xl-release-license.lic'
         }
    }

Now to use this piece of code you do need a Xl-Release download account. This can be obtained from [www.xebialabs.com](http://www.xebialabs.com). Once you get this then you can substitute in the download user and password into the above example and puppet will be able to install the software directly from the xebialabs download site. 
It will also install a license file (related to your download account).
The install_java parameter tells the module to install java (which is going to be version openjdk 1.7.0) if you want to install and use your own version of java just set this parameter to false and specify a java_home pointing to a valid java 1.7.0 jdk by using the java_home parameter.

If u now invoke a puppet run on Xl-Release.sandbox.xebialabs and all is well you should end up with a working installation of Xl-Release listening on port 5516 which should be accessible through http://Xl-Release.sandbox.xebialabs:5516/ . (username and password where left default here being admin/xebialabs)

But this is not all the module can do....

After you install Xl-Release your probably are going to want to hook up all kinds of stuff to it. And it is going to help you there ofcourse. 
The module comes equipped with a couple of resources (type/providers) which you can leverage to setup resources inside Xl-Release which makes integration with for instance xl-deploy, git repositories and jenkis a piece of cake.

To hook up a xebialabs Xl-Deploy server to your just created instance of Xl-Release pop this bit of code in the manifest used to install your xl-deploy server. 

    xlrelease_xld_server{'xldeploy1':
             properties => { 'url' => "http://${fqdn}:4516/<context_root>",
                             'username' => 'your xld user',
                             'password' => 'your xld password' 
                            }
             rest_url => 'http://user:password@xl-release.sandbox.xebialab:5516/'
        } 

Now when you start a puppet run on your Xl-Deploy server it will contact the xl-release server and hook itself into to Xl-Release configuration. 

The same goes for a jenkins server or a git repository
    
Adding this to a jenkins server manifest  will result in the jenkins server tying itself into to Xl-Release configuration ready to go. 

    xlrelease_config_item{'jenkins_default':
        type => 'jenkins.Server',
        rest_url => 'http://user:password@xl-release.sandbox.xebialab:5516/',
        properties => { username: "jenkins user name"
                        title: "title in xlr"
                        proxyHost: <optional proxy host .. null for no proxy>
                        proxyPort: <optional proxy port .. null for no port>
                        password: "jenkins user password"
                        url: 'http://your_jenkins_host_goes_here:and_the_port_here'
                        }
    }

And add this to a manifest to setup a git repository connection in Xl-Release.

    xlrelease_config_item{'your_git_repo':
        type => 'git.Repository',
        rest_url => 'http://user:password@xl-release.sandbox.xebialab:5516/',
        properties => { username: "git user name"
                        title: "title in xlr"
                        password: "git user password"
                        url: 'http://url.to.git/repo'
                       }
    }

This allows you to setup Xl-Release and completely wire it from your puppet manifests, resulting in a ready to go infrastructure.
 Now take notice of the properties parameter. The values that are in the hash are dictated by the type of configuration item that's created in Xl-Release. 
 If u crap these up the Puppet run will fail, but it won't be my fault :-) Keep that in mind before you open an issue on GitHub. 

Enjoy.

### wrap up 

Now this module does more than described here but it's fairly well documented on github so please see that for more information. 

Also the module is not done yet, there will be new features added in the near future. 
Obvious stuff that is missing and will be added soon:

* Ldap integration
* Plugin installation
* Repo to database integration

Feel free to use the module and submit issues and feature requests. 

Even better .. if u want to contribute just send in a pull request (don't forget to update the test set if u do) and i'll be happy to accomodate . 


