```
lsof -i :9403 => Ko ra gì do ko có quyền  
sudo lsof -i :9403 => Ra PID  
ls -l /proc/12345/cwd 2835738
```

```
anual-refactor-robot-workflow-agent-registry+* ± sudo ls -l /proc/12345/cwd 2835738
ls: cannot access '2835738': No such file or directory
lrwxrwxrwx 1 root root 0 Nov 12 10:51 /proc/12345/cwd -> /run/containerd/io.containerd.runtime.v2.task/moby/540ee8ab4de1b2ec6d5b92dbb40a22a2ab340fef4565a3ba30745f09d97dc234
```

```
find /path/to/project -name "docker-compose*.yml" -o -name "docker-compose*.yaml" -exec grep -l "9403" {} \;
```


---

Đây là hướng dẫn từng bước để kiểm tra **port export** của các container đang chạy và không chạy trên Docker:

---
### 1.1 Dùng `docker ps` cho container đang chạy.
    
### 1.2 Dùng `docker ps -a` để liệt kê tất cả.
    
### 1.3 Dùng `docker inspect <container_id_or_name>` để kiểm tra port của từng container (kể cả đã stop).