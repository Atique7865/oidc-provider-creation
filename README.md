
# 🛠️ AWS Load Balancer Controller Install with OIDC + Helm (বাংলায় সম্পূর্ণ গাইড)

---

## 🔰 ধাপ ১: ভ্যারিয়েবল সেট করুন

```bash
cluster_name="ProductionCluster"
region="us-east-1"
```

---

## ✅ ধাপ ২: OIDC Provider তৈরি করুন (IRSA চালুর জন্য)

### 2.1 OIDC Issuer বের করুন

```bash
oidc_issuer=$(aws eks describe-cluster \
  --name "$cluster_name" \
  --region "$region" \
  --query "cluster.identity.oidc.issuer" \
  --output text)

echo "OIDC Issuer URL: $oidc_issuer"
```

---

### 2.2 OIDC Provider অ্যাসোসিয়েট করুন

```bash
eksctl utils associate-iam-oidc-provider \
  --region "$region" \
  --cluster "$cluster_name" \
  --approve
```

---

## ✅ ধাপ ৩: IAM Policy তৈরি করুন

### 3.1 IAM Policy JSON ডাউনলোড করুন

```bash
curl -o iam_policy.json \
  https://raw.githubusercontent.com/kubernetes-sigs/aws-load-balancer-controller/v2.11.0/docs/install/iam_policy.json
```

### 3.2 IAM Policy তৈরি করুন

```bash
aws iam create-policy \
  --policy-name AWSLoadBalancerControllerIAMPolicy \
  --policy-document file://iam_policy.json
```

---

## ✅ ধাপ ৪: IRSA (IAM Role for ServiceAccount) তৈরি করুন

```bash
aws_account_id=$(aws sts get-caller-identity --query Account --output text)

eksctl create iamserviceaccount \
  --cluster "$cluster_name" \
  --region "$region" \
  --namespace kube-system \
  --name aws-load-balancer-controller \
  --attach-policy-arn arn:aws:iam::${aws_account_id}:policy/AWSLoadBalancerControllerIAMPolicy \
  --approve \
  --override-existing-serviceaccounts
```

---

## ✅ ধাপ ৫: cert-manager ইন্সটল করুন

```bash
kubectl apply --validate=false \
  -f https://github.com/jetstack/cert-manager/releases/latest/download/cert-manager.yaml
```

---

## ✅ ধাপ ৬: Helm দিয়ে AWS Load Balancer Controller ইন্সটল করুন

### 6.1 Helm repo অ্যাড করুন

```bash
helm repo add eks https://aws.github.io/eks-charts
helm repo update
```

### 6.2 VPC ID বের করুন

```bash
vpc_id=$(aws eks describe-cluster \
  --name "$cluster_name" \
  --region "$region" \
  --query "cluster.resourcesVpcConfig.vpcId" \
  --output text)

echo "VPC ID: $vpc_id"
```

---

### 6.3 Helm install কমান্ড চালান

```bash
helm install aws-load-balancer-controller eks/aws-load-balancer-controller \
  -n kube-system \
  --set clusterName=$cluster_name \
  --set serviceAccount.create=false \
  --set serviceAccount.name=aws-load-balancer-controller \
  --set region=$region \
  --set vpcId=$vpc_id
```

---

## ✅ ধাপ ৭: ডিপ্লয়মেন্ট যাচাই করুন

```bash
kubectl get deployment -n kube-system aws-load-balancer-controller
```

`READY` status থাকলে ✅ ইন্সটলেশন সফল।

---

## 🎯 উপসংহার

আপনি এখন সফলভাবে:

* OIDC Provider যুক্ত করলেন ✅
* IRSA সহ ServiceAccount তৈরি করলেন ✅
* Helm দিয়ে AWS Load Balancer Controller ইন্সটল করলেন ✅
* Controller এখন Ingress resource দেখে অটোমেটিক ALB/NLB তৈরি করতে পারবে ✅

---

### 🧪 চাইলে এখন আমি একটি Ingress resource তৈরি করে ALB দেখানোর উদাহরণ দিতে পারি। আগ্রহী?

😊
