# TIL 

## 어떤 공부를 했는지 or 문제가 있었는지
  - AWSKRUG - Nextjs + CDN + ECS 세미나 참석
  - FE 아티클 번역 리뷰

## 내가 시도해본 것들

## 어떻게 해결했는지

## 무엇을 새롭게 알았는지
  - AWS Fargate 서버리스 환경을 사용하여 SSR, 이미지 최적화 서버(in Next)를 구성이 가능하다는 것을 알게 됨.
  - ECS, ECR과 EC2의 용도 차이를 명확히 알게 되었으며, aws 내부에서 CodeCommit 기능을 통해 CodePipeline을 설정할 수 있다는 점을 알게됨.
    이 케이스는 github actions에서도 동일하게 사용할 수 있으나, aws 플랫폼 자체에서 개별적으로 관리하고 사용이 가능하다는 점.
  - Blue/Green 배포를 통해 무중단 배포 또는 롤링 배포를 선택해서 가능하다는점. 그리고 Blue/Green 배포 사용시 UI 인터페이스만으로 롤백 또는 terminate를 쉽게 사용 가능하다는점.
  - EC2를 사용하여 S3 버킷 활용으로 CDN 서버를 관리가 가능하며 이때, JS와 CSS, static 이미지 파일을 CDN으로 관리하여 애플리케이션 서버의 부하를 줄여줄수 있다는 점.



