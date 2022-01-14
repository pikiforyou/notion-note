# GSUtil CLI를 이용한 rsync 작업하기

## INTRO, 

GCF 대신 직접 google cloud sdk CLI를 이용해 작업했다.  
작업의 시작은 GCP의 스토리지를, AWS 버킷으로 이전하는 내용이었다. 현 회사의 구조는 GCP/AWS를 둘다 사용하고 있는데 기존 GCP에 묶여 있는 각 구별로 되어있는 상점이미지, 리뷰이미지등을 S3버킷으로 옮기기로 했다. 이 과정에서 API구현을 통해 커버하자고 해서 처음에는 GCF(Google Cloud Function, lamda와 비슷하다)를 작성해두었었다. 그러나 용량, 내용을 점검한 결과 결국은 CLI을 통해 직접적으로 파일을 옮기기로 결정이 되었다.  
  
다만 팀장님은 직접적으로 로컬을 이용하여 [ 1)구글에서 파일을 로컬에 받는다. 2)AWS S3에 파일을 올린다 ]라는 방식을 취하셨는데, 이 다운받고 올리는 과정이 오래 걸리기도 하고 비효율적이라고 느꼈다(용량도 얼마없는데...!)  
그래서 조금 찾아봤더니 AWS->GS(Google Storage)로 바로 전송시키는 방법이 있었다. 그렇다면 반대도 될거라고 생각해서 진행한 내역들의 기록이다.    


⚡️ 결국 이 글은 (중요)  
로컬에 다운받고 올리는 방법이 아닌, google credential과 amazone s3 credential을 설정한 후, gsutil을 이용하여 google storage(gs)에서 직접적으로 s3(s3)로 mtime정보를 이용해 싱크를 맞춰주는 작업이다. 반대로 S3에서 s3 CLI를 통해 google storage와 싱크를 맞춰줘도 상관없다.

해당사항은 최대 ~8MB/s 정도의 속도가 나오며, 컴퓨팅엔진(Google compute Engine)을 이용해서 설정하면 ~100MB/s 가 나온다고 한다.  
Python 으로 진행한다.  
<strong>** ✨ 가 붙는건 꼭 필수로 작성해줘야한다.</strong>
  
---

1. **AWS CLI 설치**

```python
pip3 install awscli
or pip3 install awscli --upgrade --user

#설치확인
aws --version
which aws
```

1. **AWS CLI를 위한 AWS Configure 설정**

```python
aws configure
------
✨AWS Access key ID ? -> 입력
✨AWS Secret Access Key ? -> 입력
✨Default region name ? -> ap-northeast-2 입력(리전이 틀리다면 리전명)
Default output format ? -> json형식으로 하고싶다면 json, 그냥 엔터로 넘겨도됨
```

1. **Google CLI에서 AWS인증정보를 참고하기 위한 자격증명 파일 설정** 
    
    (참고 : [https://docs.aws.amazon.com/ko_kr/cli/latest/userguide/cli-configure-files.html](https://docs.aws.amazon.com/ko_kr/cli/latest/userguide/cli-configure-files.html))
    
    *여러번 시도하면서 개인적으로 설정한 내역이라, rsync호출시 certificate error가 뜨지않는다면 aws_access_key_id / aws_secret_access_key 부분만 설정해줘도됨.
    

```python
vi ~/.aws/credentials
```

```python
----------------- INSERT ------------------
[default]
✨aws_access_key_id = {키아이디}
✨aws_secret_access_key = {엑세스키}
region = ap-northeast-2 {리전}
aws_global_region = ap-northeast-2 {리전}
s3_region_name = ap-northeast-2 
s3_region_endpoint = s3.ap-northeast-2.amazonaws.com

[s3]
calling_format = boto.s3.connection.OrdinaryCallingFormat
```

- aws_access_key_id/key 의 경우 **AWS IAM**을 이용해 관리자권한으로 받은 계정이 있다면, 그 아이디의 값을 적어도 된다
- certificate error중 엔드포인트 지정에 대한 에러가 뜬다면 region ~ endpoint 부분은 반드시 써줘야 내가 원하는 가상호스팅형식의 버킷에 제대로 요청이 들어간다
- [s3] calling_format의 경우는 기존 경로스타일 버킷으로 불러주기위한 설정이다
    


1. **Google내 설정** 

```python
gsutil version -l  or  gsutil ver -l
```

```python
#확인해야할 부분 
multiprocessing available : True
pass cloud sdk credentials to gsutil : False
config path(s) : /Users/~local name~/.boto, /Users/~local name~/.aws/credentials
complied crcmod : True
```

- ✨multiprocessing available : false 일 경우 True 로 변경
- ✨config path(s) 부분에  `.boto`, `.aws/credentials` 파일외에 다른 경로가 있는경우  set를 통해  pass cloud sdk credentials to gsutil 부분을 False로 변경
- ✨complied crcmod : False인경우 crcmod 체크섬 사용설정
    
    (참고: [https://cloud.google.com/storage/docs/gsutil/addlhelp/CRC32CandInstallingcrcmod](https://cloud.google.com/storage/docs/gsutil/addlhelp/CRC32CandInstallingcrcmod))
    
    ```python
    pip3 install -U crcmod
    ```
    

1. **.boto 파일 설정**

```python
vi ~/.boto
```

```python
#수정해야할 섹션, 부분만 작성
[Credentials]
✨aws_access_key_id = 적어주세요 
✨aws_secret_access_key = 적어주세요
✨s3_host = s3.ap-northeast-2.amazonaws.com
✨# use_client_certificate = False   ->주석

[Boto]
✨https_validate_certificates = False

[GSUtil]
✨parallel_process_count = 1
**속도를 올리기위해 청크로 나눌때 사용하는 변수. 용량은 상황에 맞게 설정할것 
parallel_thread_count = 100
parallel_composite_upload_threshold = 6M
parallel_composite_upload_component_size = 2M

[s3]  -> 추가
✨calling_format = boto.s3.connection.OrdinaryCallingFormat
```

*분할, 병렬에 대해서는 원하는 형태로 상황에 맞게 바꿔서 작업하시면 됩니다.
**설정끝! 이제 작업하면된다**


```python
gsutil -m rsync -r gs://seouldatepop/shop s3://cdn.datepop.co.kr/image/shop
```

변수사항도 많고, 제대로 정리된 문서가 없어 설정 하면서 진짜 너무 힘들었으므로.. 혹시나 또 할 사람이 있을까봐 남겨 놓긴 합니다만...   
언제나 그렇듯 최신의 상황을 꼭 찾아보시고 해주세요.  
2021.11.25
   
---

*참고문서
Google rsync 에 관하여 : [https://cloud.google.com/storage/docs/gsutil/commands/rsync](https://cloud.google.com/storage/docs/gsutil/commands/rsync)
스크립트를 작성해서 할생각이라면 : [https://googleapis.dev/python/storage/latest/client.html](https://googleapis.dev/python/storage/latest/client.html)
아마존 S3 : [https://docs.aws.amazon.com/ko_kr/AmazonS3/latest/userguide/Welcome.html](https://docs.aws.amazon.com/ko_kr/AmazonS3/latest/userguide/Welcome.html)
S3 버킷이 가상호스팅경로인경우 읽어볼 글 : [https://aws.amazon.com/ko/blogs/aws/amazon-s3-path-deprecation-plan-the-rest-of-the-story/](https://aws.amazon.com/ko/blogs/aws/amazon-s3-path-deprecation-plan-the-rest-of-the-story/)
구글컴퓨팅엔진을 이용하고자한다면: [https://cloud.google.com/compute/](https://cloud.google.com/compute/)
