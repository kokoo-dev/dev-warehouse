## SpringBoot + S3 파일 업로드 설정 예제


#### 버전 정보
 - SpringBoot: 2.5.2
<br/><br/>


### Dependency 설정
> ex) build.gradle
~~~gradle
dependencies {
    implementation 'org.springframework.cloud:spring-cloud-starter-aws:2.2.5.RELEASE'
}
~~~
<br/>

### yml에 aws 설정 작성
> ex) application.yml
~~~yml
cloud:
  aws:
    credentials:
      accessKey: {AWS_IAM_ACCESS_KEY}
      secretKey: {AWS_IAM_SECRET_KEY}
    s3:
      bucket: {AWS_S3_BUCKET_NAME}
    region:
      static: {REGION_NAME}
    stack:
      auto: false
~~~

<br/>

### 설정 빈 등록
> ex) S3Config.java
~~~java
@Configuration
public class S3Config{
  @Value("${cloud.aws.credentials.accessKey}")
  private String accessKey;
  
  @Value("${cloud.aws.credentials.secretKey}")
  private String secretKey;
    
  @Value("${cloud.aws.s3.bucket}")
  private String bucket;
      
  @Value("${cloud.aws.region.static}")
  private String region;
  
  @Bean
  public BasicAWSCredentials awsCredentialsProvider(){
      BasicAWSCredentials basicAWSCredentials = new BasicAWSCredentials(accessKey, secretKey);
      return basicAWSCredentials;
  }
  
  @Bean
  public AmazonS3 amazonS3(){
      AmazonS33 s3Builder = AmazonS3ClientBuilder.standard()
                .withRegion(region)
                .withCredentials(new AWSStaticCredentialsProvider(aWSStaticCredentialsProvider()))
                .build();
                
     return s3Builder;
  }
}
~~~
<br/>

### 업로드 로직 Service
> ex)S3Service
~~~java
@Service
@RequiredArgsConstructor
public class S3Service{
    private final AmazonS3 amazonS3;
    
    @Value("${cloud.aws.s3.bucket}")
    private String bucket;
    
    public String upload(MultipartFile uploadFile, String filePath, String saveFileName) throws IOException{
        ObjectMetadata objectMetadata = new ObjectMetadata();
        byte[] bytes = IOUtils.toByteArray(uploadFile.getInputStream());
        objectMetadata.setContentLength(bytes.length);
        
        ByteArrayInputStream byteArrayInputStream = new ByteArrayInputStream(bytes);
        
        String fileName = filePath + "/" + saveFileName;
        
        amazonS3.putObject(new PutObjectRequest(bucket, fileName, byteArrayInputStream, objectMetadata)
                .withCannedAcl(CannedAccessControlList.PublicRead));
                
        return amazonS3.getUrl(bucket, fileName).toString();
    }
}
~~~
