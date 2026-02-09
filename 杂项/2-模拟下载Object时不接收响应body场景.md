# 模拟下载Object时不接收响应body场景


```python
import boto3

s3 = boto3.client('s3',
                  aws_access_key_id='test_access_key',
                  aws_secret_access_key='test_secret_key',
                  endpoint_url='http://127.0.0.1:80'  # 如果使用非AWS S3
                  )

bucket_name = 'test-bucket'
object_key = 'largefile-dat'
start_byte = 0
end_byte = 2815224  # 包含在内

response = s3.get_object(
    Bucket=bucket_name,
    Key=object_key,
    Range=f'bytes={start_byte}-{end_byte}'
)

# 读取返回的数据流
#partial_data = response['Body'].read()

# 将数据写入文件
#with open('local_partial_file.bin', 'wb') as f:
#    f.write(partial_data)
```
