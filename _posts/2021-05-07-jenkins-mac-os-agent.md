---
layout: post
current: post
cover:  assets/images/jenkins-mac-os-agent-cover.png
navigation: True
title: Jenkins macOS Agent
date: 2021-05-07 10:47:00
tags: devlog
class: post-template
subclass: 'post'
author: seyoon
---

# mac OS Agent

Master (Jenkins Server)

Agent (mac OS Agent)

1. Agent에서 jenkins 유저 생성 + 관리자 권한 부여

    ![2021-05-07-jenkins-mac-os-agent(1).png](assets/images/2021-05-07-jenkins-mac-os-agent(1).png)

2. Master에서 /var/lib/jenkins/.ssh 경로에 SSH Key 생성

    ```bash
    ssh-keygen -f /var/lib/jenkins/.ssh/jenkins_agent_key

    # known_hosts 파일 없을 경우
    # touch /var/lib/jenkins/.ssh/known_hosts
    ssh-keyscan jenkins_agent_key.pub >> known_hosts
    ```

3. Agent에서 파일 및 경로, 권한 셋팅

    ```bash
    touch ~/.ssh/authorized_keys

    chown -R jenkins ~/.ssh
    chmod 600 ~/.ssh/authorized_keys
    chmod 700 ~/.ssh
    ```

### 여기서부터는 Jenkins Dashboard에서

1. Manage Jenkins - Manage Credentials - Add Credentials
    - [http://192.8.203.103:9001/credentials/store/system/domain/_/newCredentials](http://192.8.203.103:9001/credentials/store/system/domain/_/newCredentials)
2. SSH Username with private key
    - ID : 해당 Credential을 구분할 ID
    - Description: 해당 Credential에 대한 설명
    - Username: SSH Client (=Agent)에 접속할 때 쓸 username
    - Private Key: Enter directly 라디오 버튼 누르면 뜨는 텍스트 필드에 Private Key의 값 입력. 추출 방법은 아래에 명령어 참고.
    - Passphrase: SSH 생성 시 입력한 비밀번호

    ![2021-05-07-jenkins-mac-os-agent(2).png](assets/images/2021-05-07-jenkins-mac-os-agent(2).png)

    ```bash
    more /var/lib/jenkins/.ssh/jenkins_agent_key
    ```

3. Manage Jenkins - Manage Nodes and Clouds - New Node
    - Name, Description: 새 Agent의 이름과 설명
    - Remote root directory: 위에서 jenkins 라는 사용자를 만들었다면 /Users/jenkins가 root directory가 된다 (macOS 기준)
    - Labels: Pipeline에서 이 Agent를 지정할 때 사용할 라벨명
    - Usage: 작업 노드가 지정되지 않은 작업이고 해당 에이전트에 여유가 있을 경우 이 에이전트를 활용할지, 명시적으로 라벨을 통해 지정되었을 경우에만 활용할지
    - Launch agents via SSH: 위에서 생성한 SSH Credential을 이용해 접속
        - Host: 연결할 Agent의 URL 이나 IP 주소
        - Credentials: 위에서 생성한 Credential 선택
        - Host Key Verification Strategy: 위에서 Master의 known_hosts 파일 작업을 해주었으므로 Known hosts file Verification Strategy 선택

    ![2021-05-07-jenkins-mac-os-agent(3).png](assets/images/2021-05-07-jenkins-mac-os-agent(3).png)

    ![2021-05-07-jenkins-mac-os-agent(4).png](assets/images/2021-05-07-jenkins-mac-os-agent(4).png)

모두 입력 후에 Node Launch 했을 때 아래와 같은 로그가 찍히면 연결 성공.

![2021-05-07-jenkins-mac-os-agent(5).png](assets/images/2021-05-07-jenkins-mac-os-agent(5).png)

![2021-05-07-jenkins-mac-os-agent(6).png](assets/images/2021-05-07-jenkins-mac-os-agent(6).png)
