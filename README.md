# AWS V5 엘라스틱 빈스톡 CI/CD 배포
- EC2 2개
- ALB
- NLB
- IAM
- GithubAction
- RDS
- 엘라스틱IP

# Githubactions script
```java
name: aws-v5
on:
  push:
    branches:
      - main

jobs:
  build:
    runs-on: ubuntu-18.04
    steps:
      - name: Checkout
        uses: actions/checkout@v2
      - name: Set up JDK
        uses: actions/setup-java@v1
        with:
          java-version: 11
      - name: Grant execute permission for gradlew
        run: chmod +x ./gradlew
        shell: bash
      - name: Build with Gradle
        run: ./gradlew clean build
        shell: bash


      - name: Get current time
        uses: 1466587594/get-current-time@v2 
        id: current-time
        with:
          format: YYYY-MM-DDTHH-mm-ss
          utcOffset: "+09:00"
      - name: Show Current Time
        run: echo "CurrentTime=${{steps.current-time.outputs.formattedTime}}"
        shell: bash


      - name: Generate deployment package
        run: |
          mkdir -p deploy 
          cp build/libs/*.jar deploy/application.jar
          cp Procfile deploy/Procfile
          cp -r .ebextensions deploy/.ebextensions
          cd deploy && zip -r deploy.zip .
      - name: Deploy to EB
        uses: einaregilsson/beanstalk-deploy@v20
        with:
          aws_access_key: ${{ secrets.AWS_ACCESS_KEY }}
          aws_secret_key: ${{ secrets.AWS_SECRET_KEY }}
          application_name: aws-v5-beanstalk
          environment_name: Awsv5beanstalk-env
          version_label: aws-v5-${{steps.current-time.outputs.formattedTime}}
          region: ap-northeast-2
          deployment_package: deploy/deploy.zip
          
