**work in progress**

For the Liquid Metal parts of the demo environment we are using Equinx Metal Service. You can login [here](https://console.equinix.com/login). 
You will need to register for an account if you don't have an account yet.

We do have the "WeaveWorks Liquid Metal Demos" organization to host our demo environments lm hosts :

![Screenshot from 2022-11-01 12-46-06](https://user-images.githubusercontent.com/2788194/199225734-62ec3b4a-e71a-42aa-a743-3bd5280c2e7a.png)

Each demo environment gets a separate Project. 

* Liquid Metal Demo1
* Liquid Metal Demo2
* Liquid Metal Demo3

Not all environments might be up and running all the time.

A running environment has usually 3 servers. The dhcp-nat is the gateway and hosts the dhcp server as well. And we start with two additional hosts for the bare metal service : 

![Screenshot from 2022-11-01 12-48-35](https://user-images.githubusercontent.com/2788194/199226237-f8aff112-fd52-460b-9c74-8905ed54f34d.png)

## Setup & Management of the Equinix Metal environments

There is an EC2 instance "equinix-manager" running in eu-central-1 on the weaveworks-cx 482649550366 account. You can access the machine with the shared ssh demo key :
```
ssh -i ~/git/cx-presales/lutzdemo/shared-ssh-key/id_rsa_automatic ubuntu@18.193.21.18 
```

On the machine is a ~/git directory with a LIQUID subdir. 
```
$ ls ~/git/LIQUID/
demo1  demo2  demo3
```

Each of these subdirectories holds a customized copy of the original terraform repo to 
