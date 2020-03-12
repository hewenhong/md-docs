## kubesphere 删除namespace 一直处于Terminating 状态



1. 首先查看一直处于删除状态的namespace

   ```
   [root@uat-master01 ~]# kubectl get ns
   
   NAME              STATUS    AGE
   
   test              Terminating  42m
   
   ```

   

2. 将namespace 配置导出为json 文件。

   ```
   kubectl get namespace agriculture -o json > tmp.json
   ```

3. 修改tmp.json 中的字段

   删除以下字段：

   ```
       "spec": {
           "finalizers": [
               "kubernetes"
           ]
       },
   ```

4. 导出kubesphere 证书

   ```
   export client=$(grep client-cert ~/.kube/config |cut -d" " -f 6)
   echo $client
   export key=$(grep client-key-data ~/.kube/config |cut -d " " -f 6)
   echo $key
   export auth=$(grep certificate-authority-data ~/.kube/config |cut -d " " -f 6)
   echo $auth
   
   echo $client | base64 -d - > ./client.pem
   echo $key | base64 -d - > ./client-key.pem
   echo $auth | base64 -d - > ./ca.pem
   ```

   

5. 将json 文件使用curl api 方式导入。

   ```
   curl --cert ./client.pem --key ./client-key.pem --cacert ./ca.pem -k -H "Content-Type: application/json" -X PUT --data-binary @tmp.json 
   
   https://88.4.38.171:6443/api/v1/namespaces/test/finalize
   {
     "kind": "Namespace",
     "apiVersion": "v1",
     "metadata": {
       "name": "test",
       "selfLink": "/api/v1/namespaces/test/finalize",
       "uid": "94f4ef44-a90a-452a-960b-af81b2837397",
       "resourceVersion": "51296678",
       "creationTimestamp": "2020-03-11T08:09:38Z",
       "deletionTimestamp": "2020-03-11T08:09:46Z",
       "labels": {
         "kubesphere.io/workspace": "business"
       },
       "annotations": {
         "kubesphere.io/creator": "zhanghailiang",
         "openpitrix_runtime": "runtime-BDnAL7RY6rJO"
       },
       "ownerReferences": [
         {
           "apiVersion": "tenant.kubesphere.io/v1alpha1",
           "kind": "Workspace",
           "name": "business",
           "uid": "89e3c6dd-7df7-4d76-87d4-dd3044f92d21",
           "controller": true,
           "blockOwnerDeletion": true
         }
       ]
     },
     "spec": {
       
     },
     "status": {
       "phase": "Terminating"
     }
   }
   ```



6. 再次查看ns，是否已经删除。

   ```
   kubectl get ns |grep test
   ```

   

