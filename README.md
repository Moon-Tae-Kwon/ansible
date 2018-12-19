# ansible

## 이후 아래의 모든 작업에 대한 설명 및 옵션등을 상세 확인하여 TIL형태가 아닌 blog형태로 진행 예정

---
* 작성자 환경 ansible 2.7.5
* AWS instance 관리를 위한 앤서블 깃 공식 저장소에서 파이썬 스크립트 다운로드 (https://github.com/ansible/ansible/blob/devel/contrib/inventory/ec2.py)
* vim ec2.ini
```
[ec2]
regions = all
regions_exclude = us-gov-west-1,cn-north-1
destination_variable = public_dns_name
vpc_destination_variable = ip_address
route53 = False
cache_path = ~/.ansible/tmp
cache_max_age = 300
rds = False
```
```loacl터미널
./ec2.py
```
* 위와 같이 AWS에서 운영되는 여러 정보를 확인할수 있다.

* 간단한 설정을 통한 AWS instnces 확인. / vim ansible.cfg
```
[defaults]
inventory      = ./ec2.py
remote_user    = ec2-user
become         = True
become_method  = sudo
become_user    = root
nocows         = 1
```
* ping 모듈을 이용한 접속 테스트
```
ansible --private-key ~/.ssh/EffectiveDevOpsAWS.pem ec2 -m ping
```
* instances에 접속하여 명령어 실행해보기
```
ansible --private-key ~/.ssh/EffectiveDevOpsAWS.pem '54.180.99.*' -a 'df -h'
```

---
> 중요 포인트 플레이북 사용시 무조건 Roles 사용 권장
* 앞전에 진행 했던 아래의 명령어를 ansible 로 변환해 보기
```
sudo yum install --enablerepo=epel -y nodejs
wget https://raw.githubusercontent.com/Moon-Tae-Kwon/TIL/master/node/helloworld.js -O /home/ec2-user/helloworld.js
wget https://raw.githubusercontent.com/Moon-Tae-Kwon/TIL/master/node/helloworld.conf -O /etc/init/helloworld.conf
sudo start helloworld
```
* ansible-galaxy 명령어를 이용한 역활 생성 (https://galaxy.ansible.com)
```
ansible-galaxy init nodejs
ansible-galaxy init helloworld
```
* nodejs/tasks/main.yml
```
---
# tasks file for nodejs

- name: Installing node and npm
  yum:
    name: "{{ item }}"
    enablerepo: epel
    state: installed
  with_items:
    - nodejs
    - npm
```
* helloworld/files (해당 폴더로 helloworld.js 및 helloworld.conf 파일을 복사한다. (해당 경로의 파일들은 원격호스트에 복사가 된다))
* helloworld/tasks/main.yml
```
---
# tasks file for helloworld

# copy 모듈을 이용한 복사 / notify=helloworld.js 파일이 변경되면 동작
- name: Copying the application file
  copy:
    src: helloworld.js
    dest: /home/ec2-user/
    owner: ec2-user
    group: ec2-user
    mode: 0644
  notify: restart helloworld

- name: Copying the upstart file
  copy:
    src: helloworld.conf
    dest: /etc/init/helloworld.conf
    owner: root
    group: root
    mode: 0644
# service 모듈을 이용한 진행
- name: Starting the HelloWorld node service
  service:
    name: helloworld
    state: started
```
* helloworld/handlers/main.yml
(해당 파일을 통해서 앤서블이 task의 notify 매개변수를 호출함으로써 helloworld를 재시작하는 방법을 알게 될 것이다)
```
---
# handlers file for helloworld

- name: restart helloworld
  service:
    name: helloworld
    state: restarted
```
* helloworld/meta/main.yml (해당 파일에서 dependencies를 수정함으로 응용프로그램이 시작하기 전에 시스템에 nodejs를 미리 설치하도록 편집할 것이다.)
```
dependencies:
  - nodejs

  # List your role dependencies here, one per line.
  # Be sure to remove the '[]' above if you add dependencies
  # to this list.
  ```
* 플레이북 파일 작성하기 roles 와 동일한 위치에서 진행. helloworld.yml
```
---
- hosts: "{{ target | default('localhost') }}"
  become: yes
  roles:
    - helloworld
```
* 플레이북 실행 (1)
모든 EC2 인스턴스를 대상으로 앤서블을 실행 --list-hosts 옵션은 앤서블이 호스트 범주와 일치하는 호스트 목록을 반환한다.
```
ansible-playbook helloworld.yml --private-key ~/.ssh/EffectiveDevOpsAWS.pem -e target=ec2 --list-hosts
```
* 플레이북 실행 (2) --check 옵션의 경우에는 플레이북 실행 시 어떤 것이 변경되는지 확인 할수 있는 옵션이다.
```
ansible-playbook helloworld.yml --private-key ~/.ssh/EffectiveDevOpsAWS.pem -e target=54.180.99.86 --check
```