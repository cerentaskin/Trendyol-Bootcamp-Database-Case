# Trendyol-Bootcamp-Database-Case
Bootcamp Database Case Çalışması
## Case 1 
Bu senaryoda, göreceğiniz komutlar Centos 7 makine üzerinde çalıştırılmıştır. Temel yapılandırmalar yapıldıktan sonra docker kurulumu tamamlanır. 
```
systemctl enable docker
systemctl start docker
curl -L "https://github.com/docker/compose/releases/download/1.23.2/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
chmod +x /usr/local/bin/docker-compose 
```
`vim docker-compose.yml` ile dosya oluşturulur ve aşağıdaki gibi düzenlenir: 
```
couchbase1:
  image: couchbase/server
  ports:
        - '8080:8091'
  volumes:
    - ~/couchbase/node1:/opt/couchbase/var

couchbase2:
  image: couchbase/server
  ports:
        - '8081:8091'
  volumes:
    - ~/couchbase/node2:/opt/couchbase/var
```

ardından 
`docker-compose -f docker-compose.yml up ` komutuyla containerlar ayağa kaldırılır. 
```
[root@localhost ~]# docker ps
CONTAINER ID   IMAGE              COMMAND                  CREATED       STATUS       PORTS                                                                                                   NAMES
3dc9ce31f82c   couchbase/server   "/entrypoint.sh couc…"   3 hours ago   Up 3 hours   8092-8096/tcp, 11207/tcp, 11210-11211/tcp, 18091-18096/tcp, 0.0.0.0:8081->8091/tcp, :::8081->8091/tcp   root_couchbase2_1
d103c6b1c798   couchbase/server   "/entrypoint.sh couc…"   3 hours ago   Up 3 hours   8092-8096/tcp, 11207/tcp, 11210-11211/tcp, 18091-18096/tcp, 0.0.0.0:8080->8091/tcp, :::8080->8091/tcp   root_couchbase1_1
```
http://192.168.79.130:8081 adresine gittiğimizde couchbase serverın cluster oluşturma ekranı açıldı. 


![image](https://user-images.githubusercontent.com/33395649/116463505-6db1b400-a873-11eb-8a0a-f218a653b78f.png)

 
Bu sayfada “Setup New Cluster” seçeneğine tıklanır ve database ismi, kullanıcı adı ve parolalar yapılandırılır. 

`docker inspect -f '{{range.NetworkSettings.Networks}}{{.IPAddress}}{{end}}' container idsi`
komutuyla Docker IP’leri öğrenilir. 
```

[root@localhost ~]# docker inspect -f '{{range.NetworkSettings.Networks}}{{.IPAddress}}{{end}}' d103c6b1c798
172.17.0.2
[root@localhost ~]# docker inspect -f '{{range.NetworkSettings.Networks}}{{.IPAddress}}{{end}}' 3dc9ce31f82c
172.17.0.3
```

Sonrasında http://192.168.79.130:8080’e gidilir. Burada “Join Existing Cluster” seçeneğine tıklanır.  

![image](https://user-images.githubusercontent.com/33395649/116463530-75715880-a873-11eb-978a-2f67b1d8b05e.png)

 
Yukarıdaki gibi ayarlar yapılır.  


Sonrasında aşağıda gördüğünüz sekmeden bucket oluşturulur.   
 
![image](https://user-images.githubusercontent.com/33395649/116463551-7bffd000-a873-11eb-8413-c0f2f34c665c.png)


![image](https://user-images.githubusercontent.com/33395649/116463576-8326de00-a873-11eb-9d22-b8a2944a1cbb.png)




Ardından “Servers” sekmesine gelinir ve rebalance butonuna tıklanır. 

![image](https://user-images.githubusercontent.com/33395649/116463589-88842880-a873-11eb-9fef-18b495529231.png)


 



## CASE 2: 
`yum install python-requests` ile python-requests modülü yüklenir. 
Sonrasında `vim check.py`  ile dosya oluşturulup aşağıdaki gibi düzenlenir:  
```
#Importing libraries that we will use in getting information.
import requests
from requests.auth import HTTPBasicAuth
import json

#We are defining a class to store information about our clusters.
class Case:
    def __init__(self, clusterName, balanced, nodes, ramTotal ):
        self.clusterName = clusterName
        self.balanced = balanced
        self.nodes = nodes
        self.ramTotal = ramTotal

#This class has purpose of authentication.
class Auth:
    def __init__(self, ip, port, username, password):
        self.ip=ip
        self.port=port
        self.username=username
        self.password=password


    def getCluster(self):
        url="http://"+self.ip+":"+self.port+"/pools/default" #This api gives general information about cluster pools, we are going to use it in this example.
        print(url)
        response = requests.get(url, auth=HTTPBasicAuth(self.username, self.password))
        clusters = response.json()
        nodelist=[]
        for i in clusters["nodes"]:
             nodelist.append(i['hostname'])
        return Case( clusters ["clusterName"], clusters["balanced"], nodelist, clusters["storageTotals"]["ram"]["total"])


user=Auth("192.168.79.130", "8081", "Administrator", "root123")
cluster=user.getCluster()
print(cluster.clusterName,cluster.balanced,cluster.nodes,cluster.ramTotal)
```
```
[root@localhost ~]# python check.py
http://192.168.79.130:8081/pools/default
(u'cerendb', True, [u'172.17.0.2:8091', u'172.17.0.3:8091'], 5929361408)
```

http://192.168.79.130:8081/pools/default adresine gittiğimizde şu şekilde görünüyor. JSON editor kullanarak daha okunabilir hale getirebiliriz. 

![image](https://user-images.githubusercontent.com/33395649/116463601-8fab3680-a873-11eb-9e36-d87c6dd5e653.png)

![image](https://user-images.githubusercontent.com/33395649/116463619-95088100-a873-11eb-9ce7-6283d8e6be8d.png)
 
