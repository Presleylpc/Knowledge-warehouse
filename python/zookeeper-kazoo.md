## 安装相关库:
```
pip install kazoo
```

## 使用
```
#!/usr/bin/python
# -*- coding:utf-8 -*-
 
from kazoo.client import KazooClient
import sys
reload(sys)
sys.setdefaultencoding('utf-8')
 
 
class PyZookeeper:
    def __init__(self, zk):
        self.zk = zk
        self.zkconn = KazooClient(hosts=self.zk)
        self.zkconn.start()
 
    def get_node(self, param):
        result = self.zkconn.get(param)
        return result
 
    def get_children(self, param):
        result = self.zkconn.get_children(param)
        return result
 
    def create_node(self, node, value):
        self.zkconn.create(node, value)
 
    def reconnection(self):
        self.zkconn = KazooClient(hosts=self.zk)
        self.zkconn.start()
 
    def close(self):
        self.zkconn.stop()
        self.zkconn.close()
 
 
if __name__ == "__main__":
    zookeeper = 'pycdhnode2:2181,pycdhnode3:2181,pycdhnode4:2181'
    topic = '/brokers/topics'
    # topic = '/'
    ZK = PyZookeeper(zk=zookeeper)
    res = ZK.get_children(topic)
    print res
    for i in res:
        print i
    ZK.close()
```
