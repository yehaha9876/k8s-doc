相关文档
https://blog.csdn.net/yuan1125/article/details/50435345
https://blog.csdn.net/xizaihui/article/details/53178897

生成根证书
```
openssl req \
    -newkey rsa:4096 -nodes -sha256 -keyout ca.key \
    -x509 -days 365 -out ca.crt
```

生成网站证书的csr
```
openssl req \
    -newkey rsa:4096 -nodes -sha256 -keyout reg.gzky.com.key \
    -out reg.gzky.com.csr
```
注意：这里的CN必须是你的域名

生成证书
```
echo subjectAltName = IP:192.168.3.77 > extfile.cnf //如果需要ip访问
openssl x509 -req -days 365 -in reg.gzky.com.csr -CA ca.crt -CAkey ca.key -CAcreateserial -out reg.gzky.com.crt -extfile extfile.cnf
```

验证证书
```
openssl verify -CAfile ca.crt reg.gzky.com.crt
```

```
openssl s_client -connect <server name>:443
```
相关文档
https://stackoverflow.com/questions/43354403/verify-return-code-21-unable-to-verify-the-first-certificate

其他
  
openssl req -new -key apiserver.key -out apiserver.csr -subj "/CN=kube-apiserver" -config openssl.cnf
openssl x509 -req -in apiserver.csr -CA ca.crt -CAkey ca.key -CAcreateserial -out apiserver.crt -days 365 -extensions v3_req -extfile openssl.cnf
