# minio

## 1. 什么是minio？

minio是S3替身，完全兼容S3，开源，可本地部署

## 2. minio常用API

### 如何部署minio？

采用docker部署比较方便，docker下载镜像, 并用docker启动服务， 具体操作参考[MINIO快速入门](https://docs.min.io/cn/minio-docker-quickstart-guide.html)：
```shell script
docker pull minio/minio
docker run -p 9000:9000 --name minio1 \
  -e "MINIO_ACCESS_KEY=AKIAIOSFODNN7EXAMPLE" \
  -e "MINIO_SECRET_KEY=wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY" \
  -v /mnt/data:/data \
  -v /mnt/config:/root/.minio \
  minio/minio server /data
```

### a. 如何连接minio？

```python
from minio import Minio
minio_client2 = Minio('play.min.io',
access_key='Q3AM3UQ867SPQQA43P2F',
secret_key='zuf+tfteSlswRu7BJ86wekitnifILbZam1KYY3TG',
secure=True)
```

### minio 支持的主要API

bucket operations:

    make_bucket(bucket_name, location): 创建一个名为 bucket_name 的存储桶
    list_buckets(): 列出所有的存储桶
    bucket_exists(bucket_name): 判断存储桶是否存在
    remove_bucket(bucket_name): 删除存储桶
    list_objects(bucket_name, prefix=None, recursive=False, include_version=False): 列出存储桶中所有对象，如果指定了prefix，则返回以prefix为前缀的所有对象，recursive表示是否递归查找，include_version表示是否要包含版本信息
    list_objects_v2(bucket_name, prefix=None, recursive=False, start_after=None, include_user_meta=False, include_version=False): 列出存储桶中所有对象，如果指定了prefix，则返回以prefix为前缀的所有对象，recursive表示是否递归查找，include_user_meta表示是否要返回meta信息，include_version表示是否要包含版本信息
    list_incomplete_uploads(bucket_name, prefix='', recursive=False): 列出所有未完整上传的对象
    enable_bucket_versioning(bucket_name): 对某个存储桶启用版本控制
    disable_bucket_versioning(bucket_name):对存储桶暂停版本控制

object operations:

    get_object(self, bucket_name, object_name, request_headers=None, sse=None, version_id=None, extra_query_params=None): 返回指定存储桶中指定key的对象，以及是否要指定版本ID，返回一个 response 对象，里面的 data 属性表示实际存的数据，以二进制字符串表示。
    get_partial_object(bucket_name, object_name, offset=0, length=0, request_headers=None, sse=None, version_id=None, extra_query_params=None): 根据偏移量获取对象数据。
    select_object_content(bucket_name, object_name, opts): 从对象的数据中过滤数据。
    fget_object(bucket_name, object_name, file_path, request_headers=None, sse=None, version_id=None, extra_query_params=None): 将对象作为文件下载。
    copy_object(bucket_name, object_name, object_source, conditions=None, source_sse=None, sse=None, metadata=None): 将服务器中的一个对象拷贝出另一个对象
    put_object(bucket_name, object_name, data, length, content_type='application/octet-stream', metadata=None, sse=None, progress=None, part_size=DEFAULT_PART_SIZE): 将数据从流上传到存储桶中的对象。
    fput_object(bucket_name, object_name, file_path, content_type='application/octet-stream', metadata=None, sse=None, progress=None, part_size=DEFAULT_PART_SIZE): 通过文件上传对象
    stat_object(bucket_name, object_name, sse=None, version_id=None, extra_query_params=None): 获取对象信息和对象元数据。
    remove_object(bucket_name, object_name, version_id=None): 删除对象
    remove_objects(bucket_name, objects_iter): 批量删除对象
    remove_incomplete_upload(bucket_name, object_name): 删除未完整上传的对象
    
Bucket policy/notification/encryption operations:

    get_bucket_policy(bucket_name):
    set_bucket_policy(bucket_name, policy):
    delete_bucket_policy(bucket_name):
    set_bucket_notification(bucket_name, notification): 设置当前桶的通知配置
    remove_all_bucket_notification(bucket_name): 删除存储桶的设置
    