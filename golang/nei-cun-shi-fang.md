### 内存释放

如果object size&gt;32KB,直接将span返还给mheap的自由链;

如果object size&lt;32KB,查找object对应sizeclass， 归还到mcache自由链;

如果mcache自由链过长或内存过大，将部分span归还到mcentral;

如果某个范围的mspan都已经归还到mcentral，则将这部分mspan归还 到mheap堆;

mheap会定时将释放的内存归还到系统;

#### reference

* golang advance.pdf



