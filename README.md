# ansible-concourse
An easy way to deploy and manage a [Concourse CI](http://concourse.ci/) with a cluster of workers vie ansible

## Requirements
* Ubuntu 14.04
* Ansible 2.0 or higher
* PostgreSQL I recommend [ansible postgresql role](https://github.com/ANXS/postgresql)
* Optional SSL termination service I recommend [ansible nginx role](https://github.com/AutomationWithAnsible/ansible-nginx)

## Overview
I am a big fan of concourse CI, not so much bosh. This role will install concourse CI binaries.
The role is in early development, but usable so please submit PRs and PRs.



## Example
You can use test-kitchen to spin a test machine.
```
bundle install
bundle exec kitchen converge simple-vagrant
```
The kitchen machine will have an IP of **192.168.50.150**

Once your done
````
bundle exec kitchen destroy
```

Play example

```yaml
---
- name: Create Single node host
  hosts: ci
  become: True
  vars:
    concourseci_external_url         : "http://192.168.50.150:8080"
    concourseci_basic_auth_username  : "myuser"
    concourseci_basic_auth_password  : "mypass"
  roles:
    - { name: "postgresql", tags: "postgres" }
    - { name: "ansible-go", tags: "go" }
    - { name: "ansible-concourse", tags: "concourse" }
```

```ìni
[concourse-web]
ci
[concourse-worker]
ci
```

## Cluster 1 web and 4 worker example

In order to make a cluster of servers you can easily add the host to groups
```ini
[concourse-web]
ci01
[concourse-worker]
ci01
worker01
worker02
worker03
```

You would also need to generate keys for workers check key section

## Variables
```yaml
---

# Concourse version
concourseci_version                         : "v2.0.1-rc.12"

## Dir structure
concourseci_base_dir                        : "/opt/concourseci"
concourseci_bin_dir                         : "{{ concourseci_base_dir }}/bin"
concourseci_worker_dir                      : "{{ concourseci_base_dir }}/worker"
concourseci_ssh_dir                         : "{{ concourseci_base_dir }}/.ssh"

## Concourse log
concourseci_log_dir                         : "/var/log/concourse"
concourseci_log_worker                      : "{{ concourseci_log_dir }}/concourseci_worker.log"
concourseci_log_web                         : "{{ concourseci_log_dir }}/concourseci_web.log"

## Concourse User
concourseci_user                            : "concourseci"
concourseci_group                           : "concourseci"

## Concourse web
concourseci_bind_ip                         : "0.0.0.0"
concourseci_bind_port                       : "8080"
concourseci_external_url                    : "http://127.0.0.1:8080" #URL used to reach any ATC from the outside world.
# concourseci_oauth_base_url                # URL used as the base of OAuth redirect URIs. If not specified, the external URL is used.
# concourseci_peer_url                      # URL used to reach this ATC from other ATCs in the cluster. (default: http://127.0.0.1:8080)

## Cocnourse resources
# concourseci_resource_checking_interval      # Interval on which to check for new versions of resources. (default: 1m)
# concourseci_old_resource_grace_period       # How long to cache the result of a get step after a newer version of the resource is found. (default: 5m)
# concourseci_resource_cache_cleanup_interval # Interval on which to cleanup old caches of resources. (default: 30s)

## Concourse Container Retention
# concourseci_container_retention_success_duration  # The duration to keep a succeeded step's containers before expiring them. (default: 5m)
# concourseci_container_retention_failure_duration  # The duration to keep a failed step's containers before expiring them. (default: 1h)

## Concuorse authentication
### Basic
# concourseci_basic_auth_username          # Username to use for basic auth.
# concourseci_basic_auth_password          # Password to use for basic auth.
### Github
# concourseci_github_auth_client_id        # Application client ID for enabling GitHub OAuth.
# concourseci_github_auth_client_secret    # Application client secret for enabling GitHub OAuth.
# concourseci_github_auth_organization     # GitHub organization whose members will have access.
# concourseci_github_auth_team             # GitHub team whose members will have access.
# concourseci_github_auth_user             # GitHub user to permit access.
# concourseci_github_auth_auth_url         # Override default endpoint AuthURL for Github Enterprise
# concourseci_github_auth_token_url        # Override default endpoint TokenURL for Github Enterprise
# concourseci_github_auth_api_url          # Override default API endpoint URL for Github Enterprise

# Concourse TSA Config
concourseci_tsa_host                        : "{{ groups[concourseci_web_group][0] }}" # By default we pick the first host in web group
concourseci_tsa_bind_ip                     : "0.0.0.0"
concourseci_tsa_bind_port                   : "2222"
concourseci_tsa_heartbeat_interval          : "30s"
concourseci_tsa_authorization_keys          : "{{ concourseci_ssh_dir }}/tsa_authorization"
# concourseci_peer_url                      : # URL used to reach this ATC from other ATCs in the cluster. (default:http://127.0.0.1:8080)

## Concourse Extra raw options
concourseci_web_extra_options               : ""

## Concourse worker
concourseci_worker_name                     : "{{ inventory_hostname }}"

## Concourse garden linux
concourseci_garden_listen                   : "127.0.0.1:7777"
concourseci_garden_log_level                : "info" # [debug|info|error|fatal]
# concourseci_peer_ip                       # IP used to reach this worker from the ATC nodes. If omitted, the worker will be forwarded through the SSH connection to the TSA.
# concourseci_garden_network_pool           # Network range to use for dynamically allocated container subnets. (default: 10.254.0.0/22)
# concourseci_garden_max_containers         # Maximum number of containers that can be created. (default: )
# concourseci_garden_graph_cleanup_threshold_in_megabytesDirectory # Disk usage of the graph dir at which cleanup should trigger, or -1 to disable graph cleanup. (default: -1)

## PostgreSQL
concourseci_postgresql_user                 : "concourseci"
concourseci_postgresql_pass                 : "conpass"
concourseci_postgresql_host                 : "127.0.0.1"
concourseci_postgresql_db                   : "concourse"
concourseci_postgresql_ssl                  : "disable"
concourseci_postgresql_source               : "postgres://{{ concourseci_postgresql_user }}:{{ concourseci_postgresql_pass }}@{{ concourseci_postgresql_host }}/{{ concourseci_postgresql_db }}?sslmode={{ concourseci_postgresql_ssl }}"

## Ansible Groups
concourseci_web_group                       : "concourse-web"
concourseci_worker_group                    : "concourse-worker"

# ********************* Example Keys (YOU MUST OVERRIDE THEM) *********************
# This keys are used for demo only you should generate your own and store them
# safely i.e. ansible-vault
# **********************************************************************************

# If you dont know how to generate keys check https://www.digitalocean.com/community/tutorials/how-to-set-up-ssh-keys--2
# You would need to Generate a minumum of 2 keys for web
# And one key for each worker node
concourseci_key_session_path               : "{{ concourseci_ssh_dir }}/session_signing"
concourseci_key_session_public             : ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQC6tKHmRtRp0a5SAeqbVy3pJSuoWQfmTIk106Md1bGjELPDkj0A8Z4a5rJZrAR7WqrRmHr2dTL9eKroymtIqxgJdu1RO+SM3uZVV5UFfYrBV0rmp5fP2g/+Wom2RB+zCzPT1TjDnKph8xPqj19P/0FY9rKbU8h6EzEp6Z5DjwKZKvxAF8p9r6wJde4nY+oneIuG1qpxYmLpNvdM3G44vgNeMg20jVywjJVwYDNe8ourqPu8rBauLbSiQI8Uxx6dlJSTsVawrKwHQUPEI9B5LPwUzZ9t/d7k2uJnCig6aJwM8dcyr8tqxlfdfmQiHRlZozail8UzIv65MbVngji5sqoB
concourseci_key_session_private            : |
                                              -----BEGIN RSA PRIVATE KEY-----
                                              MIIEowIBAAKCAQEAurSh5kbUadGuUgHqm1ct6SUrqFkH5kyJNdOjHdWxoxCzw5I9
                                              APGeGuayWawEe1qq0Zh69nUy/Xiq6MprSKsYCXbtUTvkjN7mVVeVBX2KwVdK5qeX
                                              z9oP/lqJtkQfswsz09U4w5yqYfMT6o9fT/9BWPaym1PIehMxKemeQ48CmSr8QBfK
                                              fa+sCXXuJ2PqJ3iLhtaqcWJi6Tb3TNxuOL4DXjINtI1csIyVcGAzXvKLq6j7vKwW
                                              ri20okCPFMcenZSUk7FWsKysB0FDxCPQeSz8FM2fbf3e5NriZwooOmicDPHXMq/L
                                              asZX3X5kIh0ZWaM2opfFMyL+uTG1Z4I4ubKqAQIDAQABAoIBAFWUZoF/Be5bRmQg
                                              rMD3fPvZJeHMrWpKuroJgEM0qG/uP/ftGDlOhwIdrLKdvpAsRxA7rGE751t37B84
                                              aWStyB7OfIk3wtMveLS1qIETwn5M3PBM8bE8awhTx7vcDgurnt4CZjqDnTW4jfB+
                                              N1obzoBQ1B2Okd4i3e4wP3MIIlDCMoTPPd79DfQ6Hz2vd0eFlQcwb2S66oAGTgxi
                                              oG0X0A+o+/GXGGhcuoRfXCR/oaeMtCTAML8UVNT8qktYr+Lfo4JoQR6VroQMStOm
                                              7DvS3yJe7ZZDrQBdNDHVAsIG9/QXEWmiKNv3p1gHm216FQeJV6rzSXGjeE22tE9S
                                              JzmBKAECgYEA6CiFBIMECPzEnBooyrh8tehb5m0F6TeSeYwdgu+WuuBDRMh5Kruu
                                              9ydHE3tYHE1uR2Lng6suoU4Mnzmjv4E6THPTmTlolDQEqv7V24a0e8nWxV/+K7lN
                                              XHrq4BFE5Xa8lkLAHw4tF8Ix6162ooHkaLhhmUWzkGVxAUhL/tbVc/ECgYEAzeEn
                                              cR2NMDsNMR/anJzkjDilhiM5pORtN5O+eBIzpbFUEDZL4LIT7gqzic0kKnMJczr7
                                              0WYUp2U762yrA4U2BqiGyTO2yhcMM5kuDTG+1VTdw3G6rZ0L80jUugW9131VC3tB
                                              zcinIUs8N2hWsbuaRNhTCmlEzfe5UsikRjHgZxECgYEAze1DMCFWrvInI6BAlrDW
                                              TjTxb489MwVMM+yJMN98f/71LEn20GTyaeC5NxqtqU01iLS+TxjEn+gvYf0qtm/W
                                              WoJTKxK1JOCPU24AHF18MmFy1Fi1h+syJ9oQBPjMeA2+cjp7WBCnBvAGf5Tfw34c
                                              MJd8WwxsnqScfFq4ri+53sECgYBGobw6Xn0V0uyPsfH6UQlH4hdHkcYw//1IV/O8
                                              leIKMnA4r6gQioez3xABctO5jIXtdor2KCNl2qFX/4wcRRNn7WFwncFUS9vvx9m4
                                              xRxHbDo410fIUFzNNmtk9ptO1rzal4rX4sMT9Q/Poog7qbUfcWfr5nmogBiggh15
                                              x5rJQQKBgE4khLJKEpLi8ozo/h02R/H4XT1YGNuadyuIULZDyuXVzcUP8R7Xx43n
                                              ITU3tVCvmKizZmC3LZvVPkfDskhI9Yl3X7weBMUDeXxgDeUJNJZXXuDf1CC//Uo9
                                              N1EQdIhtxo4mgHXjF/8L32SqinAJb5ErNXQQwT5k9G22mZkHZY7Y
                                              -----END RSA PRIVATE KEY-----

concourseci_key_tsa_path                    : "{{ concourseci_ssh_dir }}/tsa"
concourseci_key_tsa_public                  : ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQCjddjviqF3BjVnxrledNsKM0wm7bwJwRgnUomLVrwHXjfArEz5yFa2C87IT9CYpIxkZMgmd0Bdtwj3kiNPP0qYpcj/uTqQTE5xLzTiJIUFsgSQwrMt/zd5x44g71qiHF/1KtHdcZq1dW3+5IwBog692HjcytbAxpUEGGpocHs/aoJ5/xn2tx61QOhkr5+PP1Ft7eHu719/pb1czhH8tZwCwNJQs4vzf79Mlgt0ikjJ84o9kOiUGP+Fc0+EjapBg9M2GE6/l86IJzcx/t/uQYCFOdKbg5ukck9NztldaOUeAPkUttPtf2vdjZU+EwSYc3XvhyQlN/QQmZ8tvG3gV9wv
concourseci_key_tsa_private                 : |
                                              -----BEGIN RSA PRIVATE KEY-----
                                              MIIEogIBAAKCAQEAo3XY74qhdwY1Z8a5XnTbCjNMJu28CcEYJ1KJi1a8B143wKxM
                                              +chWtgvOyE/QmKSMZGTIJndAXbcI95IjTz9KmKXI/7k6kExOcS804iSFBbIEkMKz
                                              Lf83eceOIO9aohxf9SrR3XGatXVt/uSMAaIOvdh43MrWwMaVBBhqaHB7P2qCef8Z
                                              9rcetUDoZK+fjz9Rbe3h7u9ff6W9XM4R/LWcAsDSULOL83+/TJYLdIpIyfOKPZDo
                                              lBj/hXNPhI2qQYPTNhhOv5fOiCc3Mf7f7kGAhTnSm4ObpHJPTc7ZXWjlHgD5FLbT
                                              7X9r3Y2VPhMEmHN174ckJTf0EJmfLbxt4FfcLwIDAQABAoIBAFzux2OJIbuV4A8c
                                              QI+fSFlISOdpChtRmPXiSyjZKxXVT0VPsIPijsn5dJsWJbZi9x6s3c5gxkuBoKuA
                                              fmqzxSl8OAaLvOwFNiPLfvmDYc2XJFlZGJ3yGAw4lGnNK243S6cLrT2FNTwtg1gD
                                              gEX9aPwucqi0+duoC1jEuNqf+LJYZykDicw3yHixgas/pKe2yDvsUhyQy2m/g9SW
                                              rpKjppxas7aKQr1GEI4Gz4JY6L78ksdLLFCiXD/pg/DLbyfOoMid8eCUnGbh1rhB
                                              PsKNyk3r/CSWsSlUlrujEqFdc/H8Ej07wVmVduTZddvjE4LcVtFlBzcEZbEofnyx
                                              H8wLv8ECgYEA0F/jBIVcSWLTB00R/Fix7Bo9ICtZ1sXL+hLPm/zVlL/gD+MlAAVB
                                              FimJKqMZa25B1ZUrYWV+Zddtel61ZxTrb86KKqtb0yuIVtPBc2ssVsO9hKL7NJ9i
                                              g6tpR0hOhD46WJxOI9Srjv61f9tP7izlwbKXo6TrdYxM8YdjXlUyMCcCgYEAyNIB
                                              IayYqg+pFoNdqKi3/n7/yGGWvlO0kW9aXkzrwTxT/k3NCHwarGgeSqU+/XVhnAHB
                                              pvsORLAnf++gQNfoxU10nrdhkj6YIdg8OK5rO4n7iNysa4bZi2DrwJt9/mFpNkvY
                                              lD956Lof/J1gPKmcNAwnsxijJE7w3I3rJ5UucLkCgYB5PMEGWV2XqTMlVVc4npZu
                                              y9lyxSZRSuZiSt2WYaYXFQiV1dAqUeRLs8EGGL1qf004qsEBux6uvIgLId2j600M
                                              0XwcVXVoyTRbaHtu3xV+Kgczi+xi8rVL7MilW9GrKdWixtbEDDIBUftiN8Uqy96m
                                              M3X9FbCVxRrjkKVlNmasEwKBgBCMxg0ZZUd2nO+/CcPxi6BMpRXFfR/YVCQ8Mg1d
                                              d3xoVV+616/gUm5s8joinitTNiUeO/Bf9lAQ2GCBxgoyAPvpozfFUyQzRmRbprLh
                                              JPM2LuWbkhYWee0zopov9lU1f+86lvG4vXpBhItUCO9W5wmfCtKGsEM4wj7a70tG
                                              zxn5AoGARTzJJb6nxhqQCAWTq2GOQ1dL3uJvHihDs8+Brtn686Xqkajaw+1+OX2i
                                              ehm8iE8k8Mdv7BqPMXQxlLb954l/ieTYkmwTnTG5Ot2+bx0Q15TGJqKFCWQZRveV
                                              uPTcE+vQzvMV3lJo0CHTlNMo1JgHOO5UsFZ1cBxO7MZXCzChGE8=
                                              -----END RSA PRIVATE KEY-----


concourseci_worker_position                 : "{{ groups[concourseci_worker_group].index(inventory_hostname)| default(0) }}"
concourseci_key_worker_path                 : "{{ concourseci_ssh_dir }}/worker"
concourseci_key_worker_public               : "{{ concourseci_worker_keys[concourseci_worker_position | int ].public}}"
concourseci_key_worker_private              : "{{ concourseci_worker_keys[concourseci_worker_position | int ].private}}"
concourseci_worker_keys                     :
                              - public      : ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQDKW31QIWcCR2Gh8i1fodDPHqviQV5eAW7Zv37Hzs1SKISvYeJ32EQ1mx2UXV8omzJiVojlXIsqkTXIBK6awvXQcRt8HFXwB9LjBfbYOUm+vU6L46HG3p2rBFAynh3NOeXvV1IBMNeuJ/w7v4CNNIkfKfQ34iwpirnX9fwoRV1pIt7c7MnKwZVrq/BwFpGh/GfOKrXLRXUsJAxDA+Mm0q2rvfpcsviINM7V41Lzemany1KVfjMLVe86CKWT0j2WERYejVlhxTXLlz7lHAowyU87dXh4QVHmDgMMSRIWgbMS0/1uAwfpdLMkzBEUhWRgKXDe/NWRk2I+Q77IJa1fnunJ
                                private     : |
                                                -----BEGIN RSA PRIVATE KEY-----
                                                MIIEpQIBAAKCAQEAylt9UCFnAkdhofItX6HQzx6r4kFeXgFu2b9+x87NUiiEr2Hi
                                                d9hENZsdlF1fKJsyYlaI5VyLKpE1yASumsL10HEbfBxV8AfS4wX22DlJvr1Oi+Oh
                                                xt6dqwRQMp4dzTnl71dSATDXrif8O7+AjTSJHyn0N+IsKYq51/X8KEVdaSLe3OzJ
                                                ysGVa6vwcBaRofxnziq1y0V1LCQMQwPjJtKtq736XLL4iDTO1eNS83pmp8tSlX4z
                                                C1XvOgilk9I9lhEWHo1ZYcU1y5c+5RwKMMlPO3V4eEFR5g4DDEkSFoGzEtP9bgMH
                                                6XSzJMwRFIVkYClw3vzVkZNiPkO+yCWtX57pyQIDAQABAoIBAQCP6rWbEcaDFmVX
                                                mjeu9hTd2YCBb+A/l2FROCJg1LGuJucHHOTGO2d3gJRu+mE9LfONgOHnzgOkCJZp
                                                ZPsRUmslDexwPm7YQZg4oftHGKdcIqMEVqauG5GjGXQ4K8AiP3VK3Z2S/zvFvuZj
                                                T/WLd7u2EE6CmDa0bNdzwpzNv1eJ92DGTm7bz71tGbjexuXuIzJVmUq1UVhj6lle
                                                dklzM9RIp0wAaCrKVifNhEdZ4cy6YG0vBaAVbUZfxO9Qnec9V5Ycor9HZ9bsPhub
                                                7H3i5j7eGFH6f01bm2o3bSVwsvSosIiG6uXbNw83RGZhsIIFK1bJ2W4CtP86C1fG
                                                +L2GaZtpAoGBAO9Anc8hsLAZEJ9gYm+abTFbTkNv4f/TPQxSngNbPx/OaDsBtHK0
                                                pQ0piG21wx6eKER0Bsb3p44Qav1G/3NVMwYAPWkoujai6OGt0bAjNCBZe5jzoYHO
                                                cN/PTSNuhfri5Hpp6EqF8m3H6gJT/rMVgEfflorXnfj7WvNwVIh50CynAoGBANiF
                                                t5pHWmvIWJs3feLiJm0o0Jp7IlpwS7vn62qfnoqv9Yze/0vNVscczkCzCbUuayf4
                                                TVgtfOe+AHs+N8u38BHrLzcYf/uRAj6fi9rf8Lhxbjv+jFOhPNttGdP5m+GDjlsW
                                                5D14cNjD/8jKIgecmYSgRTIQmdevfZseQQKhPtQPAoGBAMVVAFQlL3wvUDyD3Oy7
                                                7C/3ZRfOIhNFAWc2hUmzat8q+WEhyNmLEU9H4FTMxABu5jt/j09wWGyeMgBxHKTd
                                                stXSQNSJWP1TZM0u9nJWttmvtHe1CpLr2MFgU/lTYYJKvbQRwhwlWo0dhG8jJEJF
                                                C6c8TQh7SrpfZua+0Zo3DnKlAoGAPYpL8/Kh1Y6c+IjeI9VJPK9kEvQ6gF/4dpDl
                                                TWnOwvZeIUrkXuQe7PrX+HWqpa9qz3J4cT6EiM1tD5pQe3ttJXql8c/p2FOPwsLQ
                                                GkaaAaJjxXOE6OQkCu3IcII6du9QT72C46HO2R1kHuqsn2M4EwUGhcNIJpB/b846
                                                hgfUdqsCgYEAn3EGdd1DNC+ykrCk2P6nlFXvjxdlxCKNzCPWHjwPGE3t2DRIaEWI
                                                0XBHuBy2hbiZPIZpAK3DtPONUjjrcv3gPmhraoz5K7saY5vxyEJFYNCu2nKCJUkp
                                                ZNJ69MjK2HDIBIpqFJ7jnp32Dp8wviHXQ5e1PJQxoaXNyubfOs1Cpa0=
                                                -----END RSA PRIVATE KEY-----

                              - file        : "{{ concourseci_ssh_dir }}/concourseci_worker"
                                public      : ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQDXQFU/VlngUtaW9i6HgkdEfcB6Ak+Ogk/EN96lB6lm6NHvMWL0ggtrxzPcyQ+K6Rri1Vh2zDKenF+ZutqfxfNEmmDUNuHW96djUXEzwLuTYYdoobHNFtV9s2pEix2QFaMxMnvWSIfvKqDvI2Z+zwfzNFKjDweiVsCPw3vAF9vIL6W12zDb3hGN4uJqpz4GCj0K3DR/dxMZVEcE4VQ5ITOusqRKeZTt3QMJI9ZdJF8xg+Bdg/NSDvH7GOcmN5eLEheIx3lWCmhtQvh1iwa+JlDVWQFmxbVqPTzI/8phjOIfEqimg+nBVq157UIfDf77Xj2YyVQSv2inVc4RLZSMQw3p
                                private     : |
                                              -----BEGIN RSA PRIVATE KEY-----
                                              MIIEowIBAAKCAQEA10BVP1ZZ4FLWlvYuh4JHRH3AegJPjoJPxDfepQepZujR7zFi
                                              9IILa8cz3MkPiuka4tVYdswynpxfmbran8XzRJpg1Dbh1venY1FxM8C7k2GHaKGx
                                              zRbVfbNqRIsdkBWjMTJ71kiH7yqg7yNmfs8H8zRSow8HolbAj8N7wBfbyC+ltdsw
                                              294RjeLiaqc+Bgo9Ctw0f3cTGVRHBOFUOSEzrrKkSnmU7d0DCSPWXSRfMYPgXYPz
                                              Ug7x+xjnJjeXixIXiMd5VgpobUL4dYsGviZQ1VkBZsW1aj08yP/KYYziHxKopoPp
                                              wVatee1CHw3++149mMlUEr9op1XOES2UjEMN6QIDAQABAoIBAH42vsWwwGqEqEdE
                                              euwCO/+xLNdd24BYcKVBjU9/OpmZEuAKOVfdmQzNdV+UlYSCQr2XE5Q1D8lpL7VY
                                              lzDwRUCItRY6SBpghMn7y0DpVhOJMHjttu/m37AhL8KZP/Bof5QtYee4B9z5Rfxy
                                              6XqZsrOsjngGLBfIfojNuxZb5wdttX/u7Qp9otnESxifTbn9PfUM5UwhXRncWbsT
                                              MJ0p+aP36aNxwWDKht6bxiBRryvwNbRZX2iu18oxUUWg50uK0M/lo0KK4Svvc5lN
                                              YfBFvum78KckgDX7zVenEOmU9bQfXWgB79oP8IpRP5OyPF2AJjgiKOfR7X+JA7Nq
                                              pfXj48ECgYEA8MjIKz4ILS0ahsaxOIPsc1UyOK6F9v1PrU7ooi8WfLdY4ouPvIl5
                                              BI6zCFL9IdNOlc6Rh+UpfPYndaJz/1cWJyC0diChVdLAR+j6fuEqIKPSiv/Xn+hM
                                              sbsiNn23MoA6C2Jvv1FLez+Shvlj6fF4G1t8MfHwoXZ0yVhuSkJH9o0CgYEA5Np/
                                              k4fA9w/OsbtJu0KtGN0AwhCVmFE/3doE4BVSsmWGznzHcC864S924CHsgznrM3OW
                                              HX7C+PFgbsbtXwqxiaMAaxrh1wBnx28c4wMsNkXCFUds4DkjqDW6IhNH7W9TtuDL
                                              qNoniBH/o18aj0xGF6HJFt6tU7f9iTxJ+tmY280CgYB2HMe0DpXMM1fTzRuZ8XzH
                                              hn9ANrwYUGIJTa/n/tk1DGtZlcRIY9ctWSKRbsQlF5Zw/gd9dfhICCeLGMl18642
                                              O2DKoW8CvoL7w1k9bA5SPIpHDQEku7sDZByARmLbLvNKKltOqf4w0xp5g1RzqbOV
                                              F+dwSJIVYhofunU/kAvk8QKBgGMPcUma6ZwH66BjQXcdVW/9ueZG53oXMV4GkTWu
                                              BS3TZJbczDdzOjlfIkXCaW4kE/shfUknJZ48XVGWKgmJx2+cbwHtkPRP6JwbLJXX
                                              ObwEVg5/7FDiatzU5Mz7K5dLKSFwDLf6NkJgCBffgs+kZHK2RSTxHnWunsBYqG08
                                              4z3BAoGBAItHnGHbnl7cVHJFHd8teLzS+ki+gv+mKwPwSmeimV6zALAKZvj4RIg8
                                              4g6kUWry+NNuiaH6fsDA0FWnT3Kyc59/M/EuKNCR7ci1Gnkunc0IUn78aWtNcxX5
                                              RsCKJUM8l63P0jyUufpTbG6nAP8fMdWCdtDBidFLV2JMPYnWb4aP
                                              -----END RSA PRIVATE KEY-----
```
## Keys

Warning the role comes with default keys. This keys are used for demo only you should generate your own and store them **safely** i.e. ansible-vault

You would need to generate 2 keys for web and one key for each worker node.
An easy way to generate your keys to use a script in ```keys/key.sh```
The script will ask you the number of workers you require and generate to files in ```keys/vars``` you can than copy the content in your group vars  or pass it somehow.

## TODO
* Replace current hacky init.d scripts
* Make test work with travis (huuuh irony)
* Add more tests

## License
MIT


