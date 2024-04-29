### Reproduction Instructions

0. Install virtualenv and ssh pass

   ```
   pip3 install virtualenv
   sudo apt install sshpass
   ```

1. Create Virtual Environment

   ```
   virtualenv -p python3 .venv
   ```

2. Enter Virtual Environment

   ```
   source .venv/bin/activate
   ```

3. Install python requirments

   ```
   pip3 install -r requirements.txt
   ```

5. Start test infrastructure

    ```
    docker run -dt --name target-server \
        -v /sys/fs/cgroup:/sys/fs/cgroup:ro \
        --privileged \
        --cgroupns=host \
        --rm \
        geerlingguy/docker-debian12-ansible:latest;
    ```

6. Setup Target Server

    ```
    docker exec target-server apt update;
    docker exec target-server apt install -y openssh-server python3 curl;
    docker exec target-server systemctl enable sshd;
    docker exec target-server systemctl start sshd;

    # Add test user
    docker exec -i target-server useradd --create-home test;

    # Set test user password
    printf 'test123456\ntest123456' | docker exec -i target-server passwd test;
    ```

7. Run playbook with normal strategy

   ```
   ANSIBLE_REMOTE_TMP=/tmp/.ansible-normal-${USER}/tmp \
    ansible-playbook playbook.yml \
    -i ",$(docker inspect -f \
        '{{range.NetworkSettings.Networks}}{{.IPAddress}}{{end}}' \
        target-server \
    )" \
    --extra-vars=strategy=linear \
    --extra-vars=ansible_user=test \
    --extra-vars=ansible_password=test123456
    ```

    ```
    ANSIBLE_REMOTE_TMP=/tmp/.ansible-normal-${USER}/tmp \
    ansible-playbook playbook.yml \
        -i ",localhost" \
        -c local \
        --extra-vars=strategy=linear \
        -vv
    ```

8. Run playbook with mitogen strategy

   ```
   ANSIBLE_REMOTE_TMP=/tmp/.ansible-mito-${USER}/tmp \
   ansible-playbook playbook.yml \
    -i ",$(docker inspect -f \
        '{{range.NetworkSettings.Networks}}{{.IPAddress}}{{end}}' \
        target-server \
    )" \
    --extra-vars=strategy=mitogen_linear \
    --extra-vars=ansible_user=test \
    --extra-vars=ansible_password=test123456
    ```

    ```
    ANSIBLE_REMOTE_TMP=/tmp/.ansible-mito-${USER}/tmp \
    ansible-playbook playbook.yml \
        -i ",localhost" \
        -c local \
        --extra-vars=strategy=mitogen_linear \
        -vv
    ```

9. Install fix
    ```
    pip3 install -r requirements-fix.txt
    ```

10. Run playbook with mitogen strategy

    ```
    ANSIBLE_REMOTE_TMP=/tmp/.ansible-mito-${USER}/tmp \
    ansible-playbook playbook.yml \
        -i ",$(docker inspect -f \
            '{{range.NetworkSettings.Networks}}{{.IPAddress}}{{end}}' \
            target-server \
        )" \
        --extra-vars=strategy=mitogen_linear \
        --extra-vars=ansible_user=test \
        --extra-vars=ansible_password=test123456 \
        -vv
    ```

    ```
    ANSIBLE_REMOTE_TMP=/tmp/.ansible-mito-${USER}/tmp \
    ansible-playbook playbook.yml \
        -i ",localhost" \
        -c local \
        --extra-vars=strategy=mitogen_linear \
        -vv
    ```
