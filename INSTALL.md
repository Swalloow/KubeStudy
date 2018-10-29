# AWS 환경에서 KOPS를 이용한 Kubernetes 클러스터 구축

## kops, kubectl, awscli 설치 (Linux)

```
# kops 설치
wget -O kops https://github.com/kubernetes/kops/releases/download/$(curl -s https://api.github.com/repos/kubernetes/kops/releases/latest | grep tag_name | cut -d '"' -f 4)/kops-linux-amd64
chmod +x ./kops
sudo mv ./kops /usr/local/bin/

# kubectl 설치
wget -O kubectl https://storage.googleapis.com/kubernetes-release/release/$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)/bin/linux/amd64/kubectl
chmod +x ./kubectl
sudo mv ./kubectl /usr/local/bin/kubectl

# aws-cli 설치 (amazon linux라면 불필요)
pip install awscli
```

## IAM User 설정

```
# 아래의 권한이 필요
AmazonEC2FullAccess
AmazonRoute53FullAccess
AmazonS3FullAccess
IAMFullAccess
AmazonVPCFullAccess
```

### aws-cli로 IAM 계정 생성

```
aws iam create-group --group-name kops

aws iam attach-group-policy --policy-arn arn:aws:iam::aws:policy/AmazonEC2FullAccess --group-name kops
aws iam attach-group-policy --policy-arn arn:aws:iam::aws:policy/AmazonRoute53FullAccess --group-name kops
aws iam attach-group-policy --policy-arn arn:aws:iam::aws:policy/AmazonS3FullAccess --group-name kops
aws iam attach-group-policy --policy-arn arn:aws:iam::aws:policy/IAMFullAccess --group-name kops
aws iam attach-group-policy --policy-arn arn:aws:iam::aws:policy/AmazonVPCFullAccess --group-name kops

aws iam create-user --user-name kops
aws iam add-user-to-group --user-name kops --group-name kops
aws iam create-access-key --user-name kops

aws configure   # AccessKeyID와 SecretAccessKey 등록
```

## DNS, Cluster State storage 설정

- kops 1.6.2 버전 이상이라면 DNS 설정은 옵션 (gossip-based cluster)
- Cluster Configuration Storage로 S3를 사용 (Bucket 미리 생성해야 함)
- S3 default bucket encryption을 사용할 수 있음
- default encryption 설정이 안되어 있다면 kops에서 AES256 encryption

```
# Create Bucket
aws s3api create-bucket \
    --bucket prefix-example-com-state-store \
    --region ap-northeast-2

# S3 versioning
aws s3api put-bucket-versioning \
    --bucket prefix-example-com-state-store \
    --versioning-configuration Status=Enabled
```

## Kubernetes Cluster 생성
- kops를 통해 생성된 인스턴스는 자동으로 Auto Scaling 그룹에 들어감
- `kops create`: cluster configuration을 생성, SSH-Key가 필요
- `kops edit`: cluster configuation을 수정
- `kops update`: Build 단계, kubernetes component를 모두 설치하고 나면 ready 상태로 전환
- `kops delete`: cluster 제거, --yes (구성요소까지 전부 삭제)
- `kops rolling-update`: downtime이 없는 rolling-update 실행

```
# Environment
export NAME=myfirstcluster.example.com  # DNS가 설정되어 있는 경우
export NAME=myfirstcluster.k8s.local    # DNS가 설정되어 있지 않은 경우
export KOPS_STATE_STORE=s3://prefix-example-com-state-store

# Seoul region
aws ec2 describe-availability-zones --region ap-northeast-2
kops create cluster --zones ap-northeast-2 ${NAME}
kops edit cluster ${NAME}
kops update cluster ${NAME} --yes
kops validate cluster

# Kubectl
kubectl get nodes
kubectl cluster-info
kubectl -n kube-system get po   # system pod

# Dashboard
kops get secrets kube --type secret -oplaintext
kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/master/src/deploy/recommended/kubernetes-dashboard.yaml

# Access https://<kubernetes-master-hostname>/ui
kops get secrets admin --type secret -oplaintext
```

## Advanced
- Network topology를 설정할 수 있음 (--topology public|private)
- Private: VPC내의 private subnet으로 생성
- Public: VPC내의 public subnet으로 생성 (routed to Internet Gateway)
- Multiple zone, HA Master를 구성할 수 있음 (--master-zones=us-east-1b,us-east-1c,us-east-1d)
- Instance Group을 지정 가능 (https://github.com/kubernetes/kops/blob/master/docs/instance_groups.md)
- AMI를 지정가능, CoreOS AMI 추천
- Container Network Interface (CNI) 지정 가능 (https://kubernetes.io/docs/concepts/cluster-administration/networking/)
- Authorization (https://kubernetes.io/docs/reference/access-authn-authz/authorization/)

```
# SSH Key
ssh-keygen -t rsa -f $NAME.key -N ''
export PUBKEY="$NAME.key.pub"

# CoreOS Image
export IMAGE=$(curl -s https://coreos.com/dist/aws/aws-stable.json|sed 's/-/_/g'|jq '.'$REGION'.hvm'|sed 's/_/-/g' | sed 's/\"//g')

# Create Cluster
kops create cluster --kubernetes-version=1.12.1 \
    --ssh-public-key $PUBKEY \
    --networking flannel \
    --api-loadbalancer-type public \
    --admin-access 0.0.0.0/0 \
    --authorization RBAC \
    --zones ap-northeast-2 \
    --master-zones ap-northeast-2 \
    --master-size t2.medium \
    --node-size t2.medium \
    --image $IMAGE \
    --node-count 3 \
    --cloud aws \
    --bastion \
    --name $NAME \
    --yes
```
