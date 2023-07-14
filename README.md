# question

Mình đang sử dụng kubernet với circleci để triển khai docker. Khi mình chạy lệnh "./bin/kubectl rollout status Deploys/prj5-deploy --timeout=2700s" thì nó chạy khoảng 20 phút và báo lỗi như dưới:

        #!/bin/sh -eo pipefail
        cd .circleci/ansible
        cat inventory.txt
        ansible-playbook -i inventory.txt deploy-application.yml

        [web]
        54.237.82.69
        54.226.214.135

        PLAY [Deploy application] ******************************************************

        TASK [build] *******************************************************************
        changed: [54.237.82.69]

        TASK [if successful] ***********************************************************
        Too long with no output (exceeded 40m0s): context deadline exceeded

>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>

Sometimes it's like this:
        [web]
        54.210.76.165
        52.91.189.108

        PLAY [Deploy application] ******************************************************

        TASK [build] *******************************************************************
        changed: [54.210.76.165]

        TASK [if successful] ***********************************************************
        fatal: [54.210.76.165]: FAILED! => {"changed": true, "cmd": "./bin/kubectl rollout status deployments/****-deploy --timeout=3600s", "delta": "0:21:34.424507", "end": "2023-07-13 12:55:14.013552", "msg": "non-zero return code", "rc": 1, "start": "2023-07-13 12:33:39.589045", "stderr": "error: deployment \"****-deploy\" exceeded its progress deadline", "stderr_lines": ["error: deployment \"****-deploy\" exceeded its progress deadline"], "stdout": "Waiting for deployment \"****-deploy\" rollout to finish: 1 out of 4 new replicas have been updated...\nWaiting for deployment \"****-deploy\" rollout to finish: 1 out of 4 new replicas have been updated...\nWaiting for deployment \"****-deploy\" rollout to finish: 1 out of 4 new replicas have been updated...\nWaiting for deployment \"****-deploy\" rollout to finish: 2 out of 4 new replicas have been updated...\nWaiting for deployment \"****-deploy\" rollout to finish: 2 out of 4 new replicas have been updated...\nWaiting for deployment \"****-deploy\" rollout to finish: 2 out of 4 new replicas have been updated...\nWaiting for deployment \"****-deploy\" rollout to finish: 2 out of 4 new replicas have been updated...\nWaiting for deployment \"****-deploy\" rollout to finish: 2 out of 4 new replicas have been updated...\nWaiting for deployment \"****-deploy\" rollout to finish: 2 out of 4 new replicas have been updated...\nWaiting for deployment \"****-deploy\" rollout to finish: 2 out of 4 new replicas have been updated...\nWaiting for deployment \"****-deploy\" rollout to finish: 2 out of 4 new replicas have been updated...\nWaiting for deployment \"****-deploy\" rollout to finish: 2 out of 4 new replicas have been updated...\nWaiting for deployment \"****-deploy\" rollout to finish: 2 out of 4 new replicas have been updated...\nWaiting for deployment \"****-deploy\" rollout to finish: 2 out of 4 new replicas have been updated...\nWaiting for deployment \"****-deploy\" rollout to finish: 2 out of 4 new replicas have been updated...\nWaiting for deployment \"****-deploy\" rollout to finish: 2 out of 4 new replicas have been updated...\nWaiting for deployment \"****-deploy\" rollout to finish: 2 out of 4 new replicas have been updated...", "stdout_lines": ["Waiting for deployment \"****-deploy\" rollout to finish: 1 out of 4 new replicas have been updated...", "Waiting for deployment \"****-deploy\" rollout to finish: 1 out of 4 new replicas have been updated...", "Waiting for deployment \"****-deploy\" rollout to finish: 1 out of 4 new replicas have been updated...", "Waiting for deployment \"****-deploy\" rollout to finish: 2 out of 4 new replicas have been updated...", "Waiting for deployment \"****-deploy\" rollout to finish: 2 out of 4 new replicas have been updated...", "Waiting for deployment \"****-deploy\" rollout to finish: 2 out of 4 new replicas have been updated...", "Waiting for deployment \"****-deploy\" rollout to finish: 2 out of 4 new replicas have been updated...", "Waiting for deployment \"****-deploy\" rollout to finish: 2 out of 4 new replicas have been updated...", "Waiting for deployment \"****-deploy\" rollout to finish: 2 out of 4 new replicas have been updated...", "Waiting for deployment \"****-deploy\" rollout to finish: 2 out of 4 new replicas have been updated...", "Waiting for deployment \"****-deploy\" rollout to finish: 2 out of 4 new replicas have been updated...", "Waiting for deployment \"****-deploy\" rollout to finish: 2 out of 4 new replicas have been updated...", "Waiting for deployment \"****-deploy\" rollout to finish: 2 out of 4 new replicas have been updated...", "Waiting for deployment \"****-deploy\" rollout to finish: 2 out of 4 new replicas have been updated...", "Waiting for deployment \"****-deploy\" rollout to finish: 2 out of 4 new replicas have been updated...", "Waiting for deployment \"****-deploy\" rollout to finish: 2 out of 4 new replicas have been updated...", "Waiting for deployment \"****-deploy\" rollout to finish: 2 out of 4 new replicas have been updated..."]}

        PLAY RECAP *********************************************************************
        54.210.76.165              : ok=1    changed=1    unreachable=0    failed=1    skipped=0    rescued=0    ignored=0   

        Exited with code exit status 2
        CircleCI received exit code 2

>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>

Here is the deploy script: deploy-application.yml:
    - name: "Deploy application"
    hosts: web[0]
    user: ubuntu
    gather_facts: false
    become: yes
    tasks:
        - name: build
        shell: "./bin/kubectl set image deployments/prj5-deploy prj5-app=shalltearbloodfallen01/prj5-deploy:master"
        args:
            chdir: $HOME  

        - name: if successful
        shell: "./bin/kubectl rollout status deployments/prj5-deploy --timeout=2700s"
        args:
            chdir: $HOME

>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>

prj5-deploy:
    apiVersion: apps/v1
    kind: Deployment
    metadata:
    name: prj5-deploy
    labels:
        app: prj5-app
    spec:
    paused: false
    replicas: 4
    selector:
        matchLabels:
        app: prj5-app
    strategy:
        type: RollingUpdate
        rollingUpdate:
            maxSurge: 0
            maxUnavailable: 1
    template:
        metadata:
        labels:
            app: prj5-app
        spec:
        containers:
            - name: prj5-app
            image: shalltearbloodfallen01/prj5-deploy
            ports:
                - containerPort: 80

>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>

config.yml
    deploy-docker:
        docker:
        - image: python:3.7-alpine3.11
        steps:
        - checkout
        - add_ssh_keys:
            fingerprints: ["xxxxxxxxxxxxxxxxxxxxxxxxxx"]
        - attach_workspace:
            at: ~/
        - run:
            name: Install dependencies
            command: |
                apk add --update ansible
        - run:
            name: deploy image docker
            no_output_timeout: 40m
            command: |
                cd .circleci/ansible
                cat inventory.txt
                ansible-playbook -i inventory.txt deploy-application.yml