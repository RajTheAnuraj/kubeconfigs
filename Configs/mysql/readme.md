When the container starts the engine will try to fix the user permissions on /var/lib/mysql path.  
Since the container is not running under root context, this will fail.  
The solution I found is to create a user on all the nodes with explicit user id (so that we can specify the user id in the manifest) and use that user id in the security context of container inthe manifest.  

Here I created user id 4000 using following commands

I also created a group id to add any such future users to that group

```
groupadd appadmins -g4000
useradd sqluser -u 4000 -g 4000 -m -s /bin/bash
```

Once this is done on all machines you can use the user id 4000 in the security context  as below

spec:  
      containers:  
        - image: mysql:5.6  
          `securityContext:`   
            `runAsUser: 4000`  
            `allowPrivilegeEscalation: false`  
          name: mysql  
          env:  
            - name: MYSQL_ROOT_PASSWORD  
              value: password  
          ports:  
            - containerPort: 3306  
          volumeMounts:   
            - name: mysql-persistent-storage  
              mountPath: /var/lib/mysql  
