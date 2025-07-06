ধন্যবাদ! চলুন আমরা **"ProductionCluster" নামের EKS ক্লাস্টার** এবং **`us-east-1` রিজিয়নে** একটি **OIDC Provider** তৈরি করার জন্য সমস্ত ধাপগুলো Step-by-Step করে নেই।

---

## ✅ ধাপ ১: প্রয়োজনীয় ভ্যারিয়েবল সেট করুন

```bash
cluster_name="ProductionCluster"
region="us-east-1"
```

---

## ✅ ধাপ ২: ক্লাস্টার থেকে OIDC Issuer ID বের করুন

```bash
oidc_issuer=$(aws eks describe-cluster \
  --name "$cluster_name" \
  --region "$region" \
  --query "cluster.identity.oidc.issuer" \
  --output text)

echo "OIDC Issuer URL: $oidc_issuer"
```

📌 এটি রিটার্ন করবে এমন একটি URL:
`https://oidc.eks.us-east-1.amazonaws.com/id/EXAMPLEOIDCID`

---

## ✅ ধাপ ৩: OIDC Provider অ্যাসোসিয়েট করুন (eksctl দিয়ে)

```bash
eksctl utils associate-iam-oidc-provider \
  --region "$region" \
  --cluster "$cluster_name" \
  --approve
```

🔹 এটি করবে:

* একটি OIDC identity provider তৈরি করবে IAM এ
* এটি EKS ক্লাস্টারের সঙ্গে লিংক করবে

---

## ✅ ধাপ ৪: যাচাই করুন OIDC Provider অ্যাসোসিয়েট হয়েছে কিনা

### 🔹 ৪.১. OIDC ID কেটে বের করুন

```bash
oidc_id=$(echo "$oidc_issuer" | cut -d "/" -f5)
echo "OIDC ID: $oidc_id"
```

### 🔹 ৪.২. IAM থেকে OIDC Provider ARN বের করুন

```bash
oidc_arn=$(aws iam list-open-id-connect-providers \
  --query "OpenIDConnectProviderList[?contains(Arn, '$oidc_id')].Arn" \
  --output text)

echo "OIDC Provider ARN: $oidc_arn"
```

---

## 🧠 এখন কী করবেন?

এখন আপনি এই OIDC provider ARN ব্যবহার করে **IRSA (IAM Role for Service Account)** তৈরি করতে পারবেন, যেখানে:

* Pod-based access control করতে পারবেন
* S3, DynamoDB, SQS ইত্যাদিতে pod level access দিতে পারবেন

---

## ✅ সংক্ষিপ্ত স্ক্রিপ্ট (একসাথে সব)

```bash
#!/bin/bash

cluster_name="ProductionCluster"
region="us-east-1"

echo "[1] Getting OIDC Issuer URL..."
oidc_issuer=$(aws eks describe-cluster \
  --name "$cluster_name" \
  --region "$region" \
  --query "cluster.identity.oidc.issuer" \
  --output text)

echo "[2] Associating OIDC Provider with EKS..."
eksctl utils associate-iam-oidc-provider \
  --region "$region" \
  --cluster "$cluster_name" \
  --approve

echo "[3] Extracting OIDC ID..."
oidc_id=$(echo "$oidc_issuer" | cut -d "/" -f5)

echo "[4] Getting OIDC ARN..."
oidc_arn=$(aws iam list-open-id-connect-providers \
  --query "OpenIDConnectProviderList[?contains(Arn, '$oidc_id')].Arn" \
  --output text)

echo "✅ OIDC Provider created and linked!"
echo "OIDC Issuer: $oidc_issuer"
echo "OIDC ID: $oidc_id"
echo "OIDC ARN: $oidc_arn"
```

---

## 🔚 উপসংহার



