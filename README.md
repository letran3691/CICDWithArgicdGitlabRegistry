### install gitlab on redhat9

    dnf -y install curl vim policycoreutils python3-policycoreutils git
    
    cat > /etc/yum.repos.d/gitlab_gitlab-ce.repo >> END
    [gitlab_gitlab-ce]
    name=gitlab_gitlab-ce
    baseurl=https://packages.gitlab.com/gitlab/gitlab-ce/el/8/$basearch
    repo_gpgcheck=1
    gpgcheck=1
    enabled=1
    gpgkey=https://packages.gitlab.com/gitlab/gitlab-ce/gpgkey
           https://packages.gitlab.com/gitlab/gitlab-ce/gpgkey/gitlab-gitlab-ce-3D645A26AB9FBD22.pub.gpg
    sslverify=1
    sslcacert=/etc/pki/tls/certs/ca-bundle.crt
    metadata_expire=300
    
    [gitlab_gitlab-ce-source]
    name=gitlab_gitlab-ce-source
    baseurl=https://packages.gitlab.com/gitlab/gitlab-ce/el/8/SRPMS
    repo_gpgcheck=1
    gpgcheck=1
    enabled=1
    gpgkey=https://packages.gitlab.com/gitlab/gitlab-ce/gpgkey
           https://packages.gitlab.com/gitlab/gitlab-ce/gpgkey/gitlab-gitlab-ce-3D645A26AB9FBD22.pub.gpg
    sslverify=1
    sslcacert=/etc/pki/tls/certs/ca-bundle.crt
    metadata_expire=300
    
    END
    
    dnf install gitlab-ce -y
    
![](images/2024-03-15_10h53_47.png)

#### config gitlab and registry
    vim /etc/gitlab/gitlab.rb

change external_url 'https://gitlab.example.com' to your domain

change registry_external_url 'https://registry.example.com' to your domain

change letsencrypt['enable'] = nill to letsencrypt['enable'] = false

#### config ssl for gitlab and registry
copy your domains cert to  /etc/gitlab/ssl
![img.png](images/img.png)

Note: set certs name them same your domain

    cd /etc/gitlab/ssl; chmod 400  *.key
    cd /etc/gitlab/ssl; chmod 644  *.crt
    gitlab-ctl reconfigure
#### check status git

    gitlab-ctl status

#### get root password gitla

    cat /etc/gitlab/initial_root_password

#### login to gitlab with domain your setup

![img.png](images/img1.png)

#### install gitlab-runner and docker on server gitlab

    curl -LJO "https://s3.dualstack.us-east-1.amazonaws.com/gitlab-runner-downloads/latest/rpm/gitlab-runner_amd64-fips.rpm"
    rpm -ivh gitlab-runner_amd64-fips.rpm

    systemctl enable --now gitlab-runner.service

    yum install -y yum-utils
    yum-config-manager --add-repo https://download.docker.com/linux/rhel/docker-ce.repo
    yum install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
    systemctl enable --now docker

### docker login
    docker login -u user -p passwd registry.mbageas.life/code-repo




#### Install docker on all cluster nodes
    
    yum install -y yum-utils
    yum-config-manager --add-repo https://download.docker.com/linux/rhel/docker-ce.repo
    yum install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
    systemctl enable --now docker

#### Test docker login all cluster nodes

    docker login -u user -p passwd registry.mbageas.life/code-repo
    

#### deploy Argocd
    
    kubectl create ns argocd
    kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml

note: change argocd server to mode NodePort
    
    kubectl edit service/argocd-server -n argocd
![img_1.png](manifest-repo-main/img_1.png)

#### get argocd admin password

    kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d

#### login argocd with user/passs 

![img_1.png](images/img_2.png)

#### config ssh to gitlab repo

on gitlab server ssh-key
    
    ssh-keygen -t rsa

![](images/2024-03-15_12h51_31.gif)

#### create gitlab repo contain gitlab-ci and dockerfile

![img_1.png](images/img_1.png)

allow push

![img_3.png](images/img_3.png)



#### create gitlab repo contain helm chart to deploy k8s

Create a Variables
![](images/2024-03-15_13h34_43.gif)


#### create secret for docker registry 

kubectl create secret docker-registry regcred-loginapp --docker-server=registry.mbageas.life --docker-username=k8s --docker-password=gldt-ZhQzbMVN1pgdUMCySfdL -n default

Note: in repo helm chart have secret file, maybe edit it

#### connect argocd to gitlab repo helm chart

![](images/2024-03-15_14h11_18.gif)

#### Argocd create application

![](images/2024-03-15_14h30_26.gif)


Note summary

change:  
    APP_NAME
    IMAGE_TAG
    git clone
    cd
    sed -i
    git add

Add private key to code-repo

