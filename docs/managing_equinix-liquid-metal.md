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

Each of these subdirectories holds a customized copy of the [original terraform repo](https://github.com/weaveworks/team-quick-silver) to setup the environments. We do have a clone of this with modifications [https://github.com/weavegitops/liquid-metal-tf](https://github.com/weavegitops/liquid-metal-tf) . But this is currently not reflecting the modifications for the three different environments.

**TODO** Store all changes in Git

To configure, roll out our change an environment, you need to configure the terraform.tfvars.json.

Please note that the ts_auth_key has timed out and needs a refresh. 

```
ubuntu@ip-10-0-5-13:~$ cd git/LIQUID/demo3/liquid-metal-tf/terraform
ubuntu@ip-10-0-5-13:~/git/LIQUID/demo3/liquid-metal-tf$ cat terraform.tfvars.json 
{
  "org_id": "00bb4bef-bde5-4e65-afb7-9e94b2906fae",
  "metal_auth_token": "EY4BatRevVbts5YPjdamZ3MNsjuKB1tT",
  "ts_auth_key": "tskey-kVFPp13CNTRL-9p3og4gsgWY7AZvTGHidAX",
  "public_key": "ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABgQDDsOgtZ7ZY/p04GtWLHHFyri5N07/UNawkGl0Z6AX4Q0xO6t8Zx5mh3N5H5qCLwF98GCUmtOTUUwWxM8LHRJ7TrHP3uqQ7I+ZJ/hmp2qkUmsmgWE/3Ng/LyesixqdLXvPW0uhlbD97miLE89G3ZhXGFocbWbPAm2duhqhQ4z/vNuGRcuSNHjGO/a/7eFiXcJ8eP1goHwwTgg+EUZJwjhFTqyVwuNkOG6HhhLeyToKh6wXfsJGopWa7OBBm061xBq9Ct/Ayp5h42YJvYHuYQPvmMFg9dS/jodz1PTFPDlPb0IRpy7iBZKncFfS0BFz3I+EcsHKw83RxBevDrPHkdiH1LjJyyk+lzkeO1JHu+NkQUqu4RkqVjz9FDByu7wk6pYgXvpqzRBuSfDQU0FGQNFU/8QiFlFchVyC09yioDG4xkBQfwYj0DEgwJTRu4w7FEsTplwtnTYTRlvUD1IkX6fmopdAtaHVnrO4hIH8KTLLkx5ZfIK9UKSKlDNFlfGSm3QU= lutz@dellbook",
  "private_key_path": "/home/ubuntu/git/cx-presales/lutzdemo/shared-ssh-key/id_rsa_automatic",
  "project_name": "Liquid-Metal-Demo3",
  "metro": "MD",
  "server_type": "c2.medium.x86"

}

ubuntu@ip-10-0-5-13:~/git/LIQUID/demo3/liquid-metal-tf/terraform$ terraform plan
...
No changes. Your infrastructure matches the configuration.
```
