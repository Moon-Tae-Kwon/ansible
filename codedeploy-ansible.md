# codedeploy ansible

## Code Example

---

* codedeploy 라이브러리 등록
```
mkdir library
touch aws_codedeploy # ansible/library/aws_codedeploy
vim ansible.cfg # library = library 내용 추가
```
* cd roles && ansible-galaxy init codedeploy
* codedeploy 설정 && vim codedeploy/tasks/main.tml # 내용 수정
```
---
# tasks file for codedeploy
- name: Installs and starts the AWS CodeDeploy Agent
  aws_codedeploy: 
    enabled: yes  
```
* vim ansible/modeserver.yml # 신규 생성
```
--- 
- hosts: "{{ target | default('localhost') }}" 
  become: yes 
  roles: 
    - nodejs 
    - codedeploy 
# 중요: 경쟁 조건(race condition)을 피하기 위해서는 코드 디플로이를 시작하기 전에
# 응용프로그램에 대한 모든 종속 파일을 설치하는 것이 중요
```
* cloudformation 설정 파일복사 및 수정이후 템플릿화 (TIL/aws/cloudformation)
```
cp jenkins-cf-template.py nodeserver-cf-template.py
```
* 변경 내용
1. ApplicationName = "nodeserver"
2. ApplicationPort = "3000"
3. 기존의 jenkins code 변경
```기존
t.add_resource(IAMPolicy(
    "Policy",
    PolicyName="AllowCodePipeline",
    PolicyDocument=Policy(
        Statement=[
            Statement(
                Effect=Allow,
                Action=[Action("codepipeline", "*")],
                Resource=["*"]
            )
        ]
    ),
    Roles=[Ref("Role")]
))
```
```변경
t.add_resource(IAMPolicy(
    "Policy",
    PolicyName="AllowS3",
    PolicyDocument=Policy(
        Statement=[
            Statement(
                Effect=Allow,
                Action=[Action("s3", "*")],
                Resource=["*"])
        ]
    ),
    Roles=[Ref("Role")]
))
```
* 이후 git push 및 python nodeserver-cf-template.py > nodeserver-cf.template
* 이후 cloudformation stack 실행. # 실행되는 --stack-name 은 helloworld-staging 로 설정
* 이후 [codedepoly 설정](https://github.com/Moon-Tae-Kwon/TIL/tree/master/aws/codedeploy/run-codedeploy.md)