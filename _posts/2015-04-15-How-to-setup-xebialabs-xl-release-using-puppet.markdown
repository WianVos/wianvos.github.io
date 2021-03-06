---
layout: post
title:  "How to setup Xebialabs XL Release using puppet"
date:   2015-04-15 14:14:00
categories: Puppet XL Release tech
---
I finally got around to finishing my Puppet module for Xebialabs Xl-Release. 
And so now it's time to share and do a little blogumentation :-)

Now I'm not really going to go into the inner workings of XL Release or Puppet because there is another time and place for that. But I would like to take the time and explain how to setup XL Release using my puppet module. 

### Basic Usage

First off where gonna install the module. Now i've published the module on the [Puppetlabs forge](https://forge.puppetlabs.com/xebialabs/xlrelease) under the XebiaLabs account but of course it is also available on [Github](https://github.com/xebialabs/puppet-xlrelease).
Now if you are going to use the module I would recommend you always use the one on GitHub because that is going to be the latest and greatest ... or not (suit yourself).

Now installing the module from the forge is fairly simple.
On the commandline of a functional Puppet Master just do: 

```
puppet module install xebialabs-xlrelease
```

and there you go: module installed.

Now we need to wire it up and put it in a manifest for it to be useful and install an instance of XL Release on a machine. 
For this example I'm gonna pretend I have a machine called xl-release.sandbox.xebialabs (which I don't actually have of course :-)) and a puppet master. 

On your puppet master add the following to your site.pp (or whatever manifest file is used to hold your node definitions)


    node 'xl-release.sandbox.xebialabs' {
    
        class{xlrelease:
            install_java => true,
            install_type: 'download'
            xlr_download_user: '<your download user>'
            xlr_download_password: '<your download password'
            xlr_licsource: 'https://dist.xebialabs.com/customer/licenses/download/xl-release-license.lic'
         }
    }

Now to use this piece of code you do need a XL Release download account. This can be obtained from [XebiaLabs](http://www.xebialabs.com). Once you get this you can substitute the user and password into the above example and puppet will be able to install the software directly from the XebiaLabs download site. It will also install a license file (related to your download account).

The install_java parameter tells the module to install java (which is going to be version 1.7.0 from the openjdk project).
 If you want to install and use your own version of java just set this parameter to false and specify a java_home pointing to a valid java 1.7.0 jdk by using the java_home parameter.

If you now invoke a puppet run on XL-Release.sandbox.xebialabs and all is well you should end up with a working installation of XL Release.
 The instance will be listening on port 5516 which should be accessible through http://xl-release.sandbox.xebialabs:5516/ (username and password where left default here being admin/xebialabs)

But this is not all the module can do....

### Further Configuration

After you install XL Release you are going to want to hook up all kinds of stuff to it. And the puppet module is going to help you there of course. 
The module comes equipped with two resources (type/providers) which you can leverage to setup configuration inside XL Release which makes integration with for instance XL Deploy, Git repositories and Jenkins a piece of cake.

To hook up a XebiaLabs XL Deploy server to your just created instance of XL Release, pop this bit of code in the manifest used to provision your XL Deploy server. 

    xlrelease_xld_server{'xldeploy1':
             properties => { 'url' => "http://${fqdn}:4516/<context_root>",
                             'username' => 'your xld user',
                             'password' => 'your xld password' 
                            }
             rest_url => 'http://user:password@xl-release.sandbox.xebialab:5516/'
        } 

Now when you start a puppet run on your XL Deploy server it will contact the XL Release server and hook itself into to XL Release configuration. 

The same goes for a Jenkins server or a git repository
    
Adding this code to a manifest used to provision a Jenkins server will result in the server tying itself into to XL Release instance configuration. 
Once the puppet run on the Jenkins server is done it is ready to go from an XL Release perspective. 

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

And add this to a manifest to setup a git repository connection in XL Release.

    xlrelease_config_item{'your_git_repo':
        type => 'git.Repository',
        rest_url => 'http://user:password@xl-release.sandbox.xebialab:5516/',
        properties => { username: "git user name"
                        title: "title in xlr"
                        password: "git user password"
                        url: 'http://url.to.git/repo'
                       }
    }

This allows you to setup XL Release and completely wire it from your puppet manifests, resulting in a ready to go infrastructure.
 Now take notice of the properties parameter. The values that are in the hash are dictated by the type of configuration item that's created in XL Release. 
 If you crap these up the Puppet run will fail, but it won't be my fault :-) Keep that in mind before you open an issue on GitHub. 

Enjoy.

### Wrap up 

Now this module does more than described here but it's fairly well documented on github so please see that for more information. 

Also the module is not done yet, there will be new features added in the near future. 
Obvious stuff that is missing and will be added soon:

* Ldap integration
* Plugin installation
* Repo to database integration
* Windows support? (looking for help here) 

Feel free to use the module and submit issues and feature requests. 

Even better: if you want to contribute just send in a pull request (don't forget to update the test set if u do) and I'll be happy to accommodate . 
