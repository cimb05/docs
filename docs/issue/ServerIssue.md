# 后端问题汇总


 - nginx 附件太大无法上传

```nginx configuration
http {
    include       mime.types;
    default_type  application/octet-stream;
    client_max_body_size 10m;
}
```