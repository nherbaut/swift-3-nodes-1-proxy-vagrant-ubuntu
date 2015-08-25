# Goal

this vagrant file inspired by thornelabs.net [1] allows you to setup a swiftstack 3 node 1 proxy environement unduer Ubuntu 14.04 with the openstack swift Kilo Release.

# how to

* clone this repo
* install vagrant and virtualbox
* do 
~~~
vagrant up
~~~
* wait for a few minutes
* you should have a running swift environement

# testing

to test if your installation works correctly, create a file called creds in a folder, then paste the following text inside
~~~
export ST_AUTH=http://192.168.236.60:8080/auth/v1.0
export ST_USER=admin:admin
export ST_KEY=admin
~~~

then do

~~~
source ./creds
~~~

and finally, run your swift commands as an admin

~~~
#swift stat
>        Account: AUTH_admin
>     Containers: 0
>        Objects: 0
>          Bytes: 0
>   Content-Type: text/plain; charset=utf-8
>   X-Timestamp: 1440524395.72660
>   X-Trans-Id: txdf51fb0c5cf94b1f8cca4-0055dca86b
>   X-Put-Timestamp: 1440524395.72660
~~~

you must have the python swift client for it to work.


# References

[1] http://thornelabs.net/2014/07/14/install-a-stand-alone-multi-node-openstack-swift-cluster-with-virtualbox-or-vmware-fusion-and-vagrant.html
