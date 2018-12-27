# ansible jenkins

## cloudformation + ansible

---

* ansible-galaxy jenkins (roles 폴더에서 진행)
* roles/jenkins/tasks/main.yml
```
---
# tasks file for jenkins
# yum을 통한 RPM 패키지를 설치하는것으로 GPG키를 가져오는 것으로 구성되 있으며 해당 부분은 앤서블의이런 종류의 키를 관리하기 위한 모듈로 적용
- name: Import Jenkins GPG key
  rpm_key:
    state: present
    key: http://pkg.jenkins-ci.org/redhat/jenkins-ci.org.key
# yum 저장소 설정에서 yum 저장소를 임포트하는 설정
# 기본 저장소에 대한 사용을 하지않으며 이에 대한 부분은 타사 저장소를 통해서 중요시스템
#라이브러리가 업데이트 되는 걱을 막기 위해 실행하는 일반적인 방법이다
- name: Add Jenkins repository
  yum_repository:
    name: jenkins
    description: jenkins repository
    baseurl: http://pkg.jenkins.io/redhat
    enabled: no
    gpgcheck: yes
# 젠킨스 저장소의 사용은 기본적으로 비활성화 되어있는 부분을 enablerepo 설정으로 활성화해 yum 명령어가 실행될수 있도록 설정
- name: Install Jenkins
  yum:
    name: jenkins-2.45
    enablerepo: jenkins
    state: present
# 서비스 모듈을 이용한 jenkins 실행
- name: Start Jenkins
  service:
    name: jenkins
    enabled: yes
    state: started
```
* 플레이북을 이용한 nodejs 도 동시에 설치 ansible root 폴더에서
jenkins.yml
```
--- 
- hosts: "{{ target | default('localhost') }}" 
  become: yes 
  roles: 
    - jenkins 
    - nodejs 
```
* 위의 역활에서 jenkins / nodejs 를 추가해서 해당되는 역활들이 동작하도록 진행.
> 해당 부분에서 플레이북에서 진행되는 부부과 역활의 종속 모듈로 진행하게 하는 부분에 대한 차이점을 숙지 해야한다.
* 위에 업데이트 된 파일을 별도의 브런치로 commit 시키고 push 까지 완료된 상태에서 master 로 전환하여 pull 받는다,
```
git checkout -b jenkins
git add jenkins.yml roles/jenkins
git commit -m "Adding a jenkins playbook and role"
git push
# 전환
git checkout master
git pull
```

위위 내용을 참조한 템플릿화 -> cloudformation 참조