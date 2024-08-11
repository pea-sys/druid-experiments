# Docker で druid を実行する

```
mkdir druid-docker
cd druid-docker

sudo curl -O https://raw.githubusercontent.com/apache/druid/30.0.0/distribution/docker/docker-co
mpose.yml
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100  2981  100  2981    0     0   5589      0 --:--:-- --:--:-- --:--:--  5582
sudo curl -O https://raw.githubusercontent.com/apache/druid/30.0.0/distribution/docker/envi
ronment
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100  2416  100  2416    0     0   3607      0 --:--:-- --:--:-- --:--:--  3605

masami@masami-L /u/s/druid-docker> sudo docker compose up -d
WARN[0000] /usr/src/druid-docker/docker-compose.yml: the attribute `version` is obsolete, it will be ignored, please remove it to avoid potential confusion
[+] Running 72/7
 ✔ historical Pulled                                                                                                        182.0s
 ✔ zookeeper Pulled                                                                                                          71.4s
 ✔ middlemanager Pulled                                                                                                     182.0s
 ✔ router Pulled                                                                                                            182.0s
 ✔ postgres Pulled                                                                                                          136.4s
 ✔ broker Pulled                                                                                                            182.0s
 ✔ coordinator Pulled                                                                                                       182.0s
[+] Running 15/15
 ✔ Network druid-docker_default           Created                                                                             0.1s
 ✔ Volume "druid-docker_broker_var"       Created                                                                             0.0s
 ✔ Volume "druid-docker_historical_var"   Created                                                                             0.0s
 ✔ Volume "druid-docker_middle_var"       Created                                                                             0.0s
 ✔ Volume "druid-docker_router_var"       Created                                                                             0.0s
 ✔ Volume "druid-docker_metadata_data"    Created                                                                             0.0s
 ✔ Volume "druid-docker_druid_shared"     Created                                                                             0.0s
 ✔ Volume "druid-docker_coordinator_var"  Created                                                                             0.0s
 ✔ Container zookeeper                    Started                                                                             3.0s
 ✔ Container postgres                     Started                                                                             3.0s
 ✔ Container coordinator                  Started                                                                             1.3s
 ✔ Container middlemanager                Started                                                                             2.9s
 ✔ Container broker                       Started                                                                             2.4s
 ✔ Container router                       Started                                                                             2.4s
 ✔ Container historical                   Started                                                                             2.5s
```

http://{hostname}:8888/にアクセスすると GUI インターフェースが表示されます。

![1](https://github.com/user-attachments/assets/fb4348b1-41e7-48a6-bca7-2b02767e5839)
