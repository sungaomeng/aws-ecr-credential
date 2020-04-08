# AWS ECR Private Registry Rotation

**本项目仅使用中国区域AWS(宁夏&北京)**

**This project is only used for AWS China Region**

## 背景

在非 AWS 环境中运行 Kubernets 的时候，需要从 AWS ECR 私有仓库中拉取镜像。
本项目用来在非AWS Kubernets集群中方便的使用 AWS ECR。

## 问题

1、由于 Kubernets 不支持使用 `credHelpers` `credsStore` 等 Docker 的扩展认证方案，所以不能够简单的通过使用 [amazon-ecr-credential-helper](https://github.com/awslabs/amazon-ecr-credential-helper) 来解决。

2、由于 AWS ECR 私有仓库密码是一天过期，所以直接使用 imagePullSecrets 会导致在有效期过后 yaml 不能正常安装。

## 解

通过在 Kubernets 中创建一个通用的类型为 `kubernetes.io/dockerconfigjson` 的 secret 对象，然后所有部署脚本添加 `imagePullSecrets` 来实现。

具体步骤为：

* 创建一个名为 `aws-registry` 的保密字典。
* 创建一个 Kubernets Jobs 来每间隔几个小时来通过调用 `kubetcl get ecr xxx` 的方式刷新 `aws-registry`。

## Getting Started

1、克隆项目到本地 `git clone xxx`

2、打包 helm

```sh
helm package ./aws-ecr-credential/ -d ./package/
helm repo index ./package/
```
3、安装 helm

```sh
helm install --name aws-ecr-credential aws-ecr-credential
  --namespace default \
  --set targetNamespace=default
  --set-string aws.account=<aws account nubmer> \
  --set aws.region=<aws region> \
  --set aws.accessKeyId=<base64> \
  --set aws.secretAccessKey=<base64>
```

4、在部署中使用

通过添加 `imagePullSecrets aws-registry` 来实现，示例如下：

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: xxx
  namespace: default
spec:
  containers:
    - name: xxx
      image: xxx:latest
  imagePullSecrets:       <-- 新增
    - name: aws-registry  <-- 新增
```

## 说明

本项目参考并修改自：[architectminds/helm-charts](https://github.com/architectminds/helm-charts)